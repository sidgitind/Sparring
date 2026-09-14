## Context

Flowdesk's ticket detail view currently renders the raw message thread only.
There is no existing summarization capability and no prior integration with
an LLM provider anywhere in the ticket read/write path.

Assumptions made in this design (no existing Flowdesk architecture docs were
available to ground these — flag/correct if wrong):
- Flowdesk has a **Ticket Service** owning ticket and message-thread data,
  reachable via an internal API/event bus, with a ticket detail endpoint the
  frontend calls on ticket-open.
- There is an internal event bus / message queue already in use for other
  ticket lifecycle events (assignment, status change, etc.) that a new
  consumer can subscribe to.
- No LLM provider integration exists yet anywhere in Flowdesk; this is the
  first one, so its contract sets precedent for future AI features.

Constraints from specs/Input_Spec.md:
- Summary must appear automatically on ticket open (AC1) — this puts LLM
  latency on or near the ticket-open critical path unless we deliberately
  design around it.
- Summary must regenerate when new messages arrive after the last summary
  (AC4) — this is an event-driven, unbounded-frequency trigger (a thread can
  receive many messages in quick succession).
- Attachments/screenshots, multi-language, and summary editing are
  explicitly out of scope (v1).

## Goals / Non-Goals

**Goals:**
- Generate and surface a summary without adding a hard synchronous
  dependency to ticket-open (a summarization failure or slow LLM response
  must never block or degrade the ticket detail page itself).
- Keep summary generation, storage, and feedback isolated in a new module so
  Ticket Service ownership and its existing regression surface are untouched.
- Make regeneration behavior deterministic and cost-bounded under bursty
  message activity, not naive "regenerate on every message."
- Make the AC2 accuracy threshold (90% against a human-reviewed set)
  operationally measurable — offline, versioned, repeatable — not just a
  one-time launch check.

**Non-Goals:**
- Summarizing attachments, screenshots, or non-text content (spec: out of
  scope).
- Multi-language summary generation (spec: out of scope).
- Engineer editing of summary content (spec: out of scope).
- Manual/on-demand "regenerate now" affordance for the engineer — not an AC;
  treated as a future enhancement, not this change (see Risks).
- Making summaries searchable/indexed — not requested; would expand the
  regression surface into search infrastructure.

## Decisions

### Module ownership and boundaries

Introduce a new **Summary Service**, owned independently of Ticket Service:

```
                event: message.added
                event: ticket.opened
Ticket Service --------------------------> Summary Service ----> LLM Provider
      ^                                          |
      | ticket detail (unchanged)                | summary read/write,
      | read path                                | feedback read/write
      |                                          v
   Frontend  <----------------------------  Summary API
   (ticket detail view)   GET /summaries/{ticket_id}
                           POST /summaries/{summary_id}/feedback
```

- Ticket Service keeps its existing read path unchanged: it does not call the
  LLM and does not block on Summary Service. It only emits `ticket.opened`
  and `message.added` events it already has the data for.
- Summary Service is the only module that talks to the LLM provider, owns
  the summary and feedback tables, and exposes a read API the frontend polls
  or fetches alongside (not instead of) the ticket detail call.
- Frontend renders the ticket detail page from Ticket Service's response
  immediately; the summary panel is a separate, independently-loading region
  that shows a loading state and fills in when Summary Service responds (or
  shows a degraded state on failure). This is what keeps AC1 ("generates
  automatically when opened") from becoming a blocking dependency.
- Rationale: coupling summary generation into the ticket-open request path
  would make every ticket-open latency-bound by LLM response time and
  failure-bound by LLM availability — an unacceptable blast radius increase
  on Flowdesk's core read path for a v1, non-critical feature.

### Frontend-facing API contracts

Added after QA NFQ review, which found neither endpoint had a defined
timeout-and-failure-shape pair — the same non-negotiable already applied
to the LLM provider contract below, just not carried through to where the
frontend actually consumes it.

**`GET /summaries/{ticket_id}`** (backs AC1, AC4, AC7, AC8)
- Timeout: 20s (AC8), already defined.
- Response: always `200` with a body carrying the state explicitly —
  `{status: "pending" | "complete" | "stale" | "failed" | "off", content,
  generated_at, model_version}` — since `pending`, `failed`, and `off` are
  expected application states, not transport errors, they don't get HTTP
  error codes of their own.
- Failure shape: a non-2xx response, or no response within the 20s
  timeout, is treated identically — both resolve into the existing
  failure display rule (stale-but-visible if a summary was already
  visible client-side, otherwise "Summary unavailable").

**`POST /summaries/{summary_id}/feedback`** (backs AC3, AC9)
- Request body: `{vote: "up" | "down"}`.
- Timeout: 5s — shorter than the summary fetch, since this is a small
  write with no LLM dependency in its path.
- Response: `204` on success. `404` with `{error: "summary_not_found"}`
  if `summary_id` doesn't exist. Any other failure (5xx, network error, or
  exceeding the 5s timeout) is treated per AC9's existing behavior: one
  silent retry, then revert the selection and show the non-blocking
  message — no new response shape needed for that path, since AC9 already
  defines what the user sees.

### External service contract — LLM provider

- **Timeout:** 8s hard timeout per generation call. Chosen to keep p99
  summary latency bounded without needing the frontend to hold a spinner
  indefinitely; if exceeded, the call is treated as a failure (see below).
- **Failure shape:** on timeout, non-2xx response, or malformed/empty
  content, the summary is marked `status: failed`. No partial or
  unvalidated content is ever written, and there is no retry-now
  affordance in the UI (out of scope per Non-Goals). What the panel shows
  on failure depends on whether a prior summary exists — see the unified
  failure display rule under State write policy below. (This replaces an
  earlier, self-contradictory framing caught in Architect review, where
  this bullet said "Summary unavailable" unconditionally while the
  write-policy bullet said the prior summary stays visible — both cannot
  be true at once.)
- **Retry policy:** one retry with exponential backoff (base 1s) on
  transient failures only (5xx, timeout, connection reset). No retry on 4xx
  (indicates a malformed request on our side — logged as a bug signal, not
  retried). This caps worst-case latency at roughly 2x timeout and avoids
  hammering the provider during an outage.
- **State write policy:** a summary row is written only after a complete,
  schema-validated response is received. Generation is tracked through an
  explicit state machine (`pending -> complete | failed`) so a crash or
  timeout mid-call never leaves a half-written or ambiguous row.
  **Unified failure display rule:** if a prior summary exists, a failed
  regeneration leaves it visible with a "summary may be out of date"
  indicator; "Summary unavailable" is shown only when there is no prior
  summary to fall back to (first-generation failure). This is strictly
  better for the engineer than replacing working content with an error.
- **Concurrency / rate limiting:** requests to the LLM provider are
  rate-limited per-tenant at the Summary Service boundary (token bucket) to
  prevent one high-traffic tenant's message bursts from exhausting shared
  provider quota and starving other tenants' summary generation.

### Data model changes

New tables, owned by Summary Service (not Ticket Service's schema):

- `ticket_summary`: `id`, `ticket_id`, `version` (monotonic per ticket),
  `content`, `status` (`pending|complete|failed`), `source_high_water_mark`
  (id/timestamp of the latest message included in this summary — this is
  the field AC4's staleness check is computed against), `model_version`,
  `generated_at`, `consecutive_failure_count` (reset to 0 on any successful
  generation or on `message.added`; drives the AC7 retry cap). A unique
  index on `(ticket_id, source_high_water_mark)` is recommended to enforce
  the idempotency check described under "External service contract — event
  bus" at the database level, not just in application logic.
- `ticket_summary_feedback`: `id`, `summary_id`, `engineer_id`, `vote`
  (`up|down`), `created_at`, `updated_at`. Unique on `(summary_id,
  engineer_id)` — an engineer's vote is upserted, not appended, so re-voting
  changes rather than duplicates their feedback.

No changes to Ticket Service's schema. Ticket Service is only a producer of
events already derivable from its existing data (message-added,
ticket-opened) — it does not need to know summaries exist.

### Regeneration semantics (AC4)

Naively regenerating on every `message.added` event is rejected: a burst of
N messages in quick succession (common in active threads) would trigger N
LLM calls, most of them wasted, and each visible reader might see the
summary panel flicker mid-burst.

Decision: **lazy regeneration with a staleness check**, not eager
regeneration on every message.
- `message.added` events update a lightweight "thread is stale" marker
  (comparing new message id/timestamp against the current summary's
  `source_high_water_mark`) — no LLM call happens at this point.
- Generation is triggered on the *next* `ticket.opened` event if the thread
  is marked stale, using the message state at that time (so a burst of 10
  messages before anyone reopens the ticket costs one LLM call, not ten).
- Trade-off made explicit and confirmed (resolved during QA Functional
  review, after being flagged as open since Architect review): an engineer
  who is already viewing an open ticket when new messages arrive will not
  see the summary auto-update live; it updates on next open/reload.
  AC4 now states this explicitly rather than leaving it to design.md alone.

### Failure retry and frontend timeout (AC7, AC8)

Added after UI Designer review, which found "Summary unavailable" had no
stated recovery path and the loading state had no user-facing bound:

- **Automatic retry on reopen (AC7):** a `failed` status is itself treated
  as a trigger for the next `ticket.opened` event, not just new messages —
  reopening the ticket is the recovery path, with no retry button needed.
  A `consecutive_failure_count` field on `ticket_summary` caps this at 3:
  after 3 consecutive failures, auto-retry on open stops until a
  `message.added` event resets the count (new content is a reasonable
  signal that a prior permanent failure — e.g. content that reliably
  triggers a 4xx — may no longer apply). This bounds worst-case repeated
  LLM calls on a ticket that fails for a structural reason, while still
  giving engineers who reopen a ticket a real chance at a working summary.
- **Frontend timeout (AC8):** the frontend's own request to Summary
  Service is bounded at 20s — a buffer above the backend's ~16s worst case
  (8s LLM timeout + 1 retry) to account for the Summary Service round-trip
  itself. If exceeded, the loading state resolves into the same failure
  display rule already defined (stale-but-visible, or "Summary
  unavailable" if no prior summary) — no new UI state is needed, just a
  bound on how long "loading" can persist.

### Feedback submission behavior (AC9)

Added after UI Designer review, which found the thumbs up/down write path
had no failure handling at all:

- Optimistic UI: clicking a thumbs up/down button highlights it
  immediately: the request fires in the background rather than blocking
  on a round-trip, since this is a low-stakes, non-primary interaction.
- One silent retry on transient failure, matching the same retry pattern
  used for the LLM contract, against the 5s timeout defined under
  "Frontend-facing API contracts" below. If it still fails, the selection
  reverts to its prior state (unvoted, or the previous vote) and a brief,
  non-blocking message indicates the vote wasn't saved. The reverted state
  is itself the recovery path — the engineer can simply click again.

### External service contract — event bus (internal)

Added after Architect review: Summary Service's correctness depends
entirely on what the internal event bus guarantees, so this is treated as
a contracted dependency rather than an implementation detail.

- **Delivery guarantee:** assumed **at-least-once** (the common case for
  most bus implementations) unless Flowdesk's actual bus guarantees
  otherwise — confirm and correct this assumption before implementation.
- **Idempotency:** both consumers are designed to be safe under duplicate
  delivery rather than relying on exactly-once semantics.
  - `message.added` -> marking a thread stale is a plain overwrite
    (`stale = true`), which is idempotent regardless of duplicate delivery.
  - `ticket.opened` -> before calling the LLM, check for an existing
    `ticket_summary` row with the same `(ticket_id, source_high_water_mark)`
    in `pending` or `complete` state; if one exists, skip the call. This
    also makes two engineers opening the same stale ticket at nearly the
    same instant safe — the second request finds the first's row instead
    of triggering a second LLM call.

### Threshold monitoring and kill switch (AC6)

AC6 (added after PM review) requires detecting a thumbs-down-rate or
accuracy breach and flagging/disabling the feature. Added after Architect
review, which found the AC had no mechanism behind it:

- **Threshold monitoring:** a scheduled job in Summary Service (e.g.
  nightly) computes, over AC6's trailing window, (a) thumbs-down rate from
  `ticket_summary_feedback` and (b) accuracy on a sampled set of production
  summaries via the same offline eval harness used for `model_version`
  regression checks (see Accuracy measurement below). A breach of either
  writes an alert record and emits it through Flowdesk's existing
  alerting/notification channel (assumption — confirm this exists; if not,
  this needs its own decision before implementation).
- **Kill switch, not auto-disable:** AC6 says the feature "may be
  disabled," not "is disabled" — read as a human decision made after being
  alerted, not an automatic shutoff; an automatic disable triggered by a
  noisy sample window is its own failure mode, capable of killing a
  working feature on a bad sample. Implemented as a single boolean config
  Summary Service checks before generating: when off, the summary panel
  attempts no generation and the frontend shows nothing — a third display
  state, distinct from both "Summary unavailable" and a stale-but-visible
  summary, signaling the feature is off rather than broken. A human flips
  the switch after triage; the monitoring job only alerts, it never
  disables on its own.

### Accuracy measurement (AC2)

AC2 ("90% accuracy against a human-reviewed test set") is a model-quality
requirement, not something the architecture can satisfy directly — but the
design commits to making it measurable and regression-checkable:
- An offline evaluation harness runs the same generation path (same prompt
  construction, same `model_version`) against a fixed, human-labeled golden
  set, independent of production traffic.
- Every summary row records `model_version`, so a provider/prompt change
  can be evaluated against the golden set *before* rollout, and production
  summaries stay traceable to the model version that produced them if a
  regression is later reported via the thumbs-down signal.
- The accuracy rubric (what counts as a "correct" summary) is now defined
  in AC2 of specs/Input_Spec.md — resolved during QA Functional review,
  after PM and QA Functional both independently flagged it as untestable
  without one. Not restated here; this design only needs to know the
  eval harness scores against that rubric, not what the rubric says.

### Regression surface

- **Touched:** ticket detail page (new panel, additively rendered;
  existing content unchanged), new Summary Service and its two tables, one
  new outbound integration (LLM provider), Ticket Service gains two event
  emissions (no schema or endpoint changes).
- **Explicitly not touched:** ticket list/queue views, ticket search and
  indexing, notification system, ticket assignment/routing, existing
  message-thread storage and rendering, any existing Ticket Service API
  contracts or response shapes.

## Risks / Trade-offs

- **Staleness window (from lazy regeneration):** an engineer keeping a
  ticket open across incoming messages sees a summary that reflects the
  thread state as of last open, not live. Confirmed as the intended
  behavior (resolved during QA Functional review, now explicit in AC4) —
  recorded here as an accepted trade-off, not an open question. If
  real-time freshness turns out to matter more in practice, it would need
  new infrastructure (push/websocket delivery) not currently in scope.
- **Long-thread cost/accuracy tension:** very long threads may need
  truncation or map-reduce summarization to stay within provider context
  limits; either approach can affect the AC2 accuracy threshold, and neither
  is specified in specs/. Flagged, not resolved, here.
- **Blast radius depends on frontend discipline:** the "summary panel loads
  independently of ticket detail" boundary is an architectural intent, not
  something the backend design can enforce by itself — if the frontend
  implementation makes ticket-open await the summary call, the isolation
  this design relies on is defeated. Needs an explicit contract/test, not
  just documentation.
- **No on-demand regenerate affordance:** treating manual regeneration as
  out of scope means an engineer who spots a stale summary has no recourse
  in v1 beyond reloading the ticket. Acceptable for v1 per current ACs, but
  worth the PM confirming this is intentional rather than an oversight.
- **First LLM integration in Flowdesk:** the timeout/retry/rate-limit
  contract above has no precedent to follow or diverge from; if Flowdesk
  later adds more AI features, this contract becomes the de facto template,
  for better or worse.

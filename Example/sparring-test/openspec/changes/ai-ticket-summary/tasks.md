<!-- Precheck note: sparring_pm.md, sparring_architect.md, sparring_ui.md,
     sparring_qa_functional.md, and sparring_qa_nfq.md each still contain
     their original "VERDICT: BLOCK" header — none were rewritten after
     their items were resolved (see sparring_brief.md's process note).
     Verified via sparring_brief.md (PIPELINE PASS) that every BLOCK item
     across all five rounds was resolved in specs/Input_Spec.md and
     design.md before proceeding. No persona was skipped. -->

## 1. Setup

- [ ] 1.1 Scaffold Summary Service as a new module, independent of Ticket
      Service's codebase/deploy unit (design.md: Module ownership).
- [ ] 1.2 Create `ticket_summary` table: `id`, `ticket_id`, `version`,
      `content`, `status` (`pending|complete|failed`),
      `source_high_water_mark`, `model_version`, `generated_at`,
      `consecutive_failure_count`; unique index on `(ticket_id,
      source_high_water_mark)`.
- [ ] 1.3 Create `ticket_summary_feedback` table: `id`, `summary_id`,
      `engineer_id`, `vote`, `created_at`, `updated_at`; unique index on
      `(summary_id, engineer_id)`.
- [ ] 1.4 Add `ticket.opened` and `message.added` event emissions to
      Ticket Service (no schema or endpoint changes to Ticket Service
      itself).
- [ ] 1.5 Provision LLM provider credentials/config for Summary Service
      (first LLM integration in Flowdesk — no existing pattern to follow).
- [ ] 1.6 Add the kill-switch boolean config Summary Service checks
      before generating (AC6).

## 2. Core Implementation

- [ ] 2.1 Implement the `ticket.opened` consumer: skip entirely if the
      kill switch has disabled the feature; otherwise check for an
      existing `ticket_summary` row at the current
      `source_high_water_mark` before calling the LLM (AC1, AC4).
- [ ] 2.2 Implement the `message.added` consumer: idempotent overwrite
      marking the thread stale; reset `consecutive_failure_count` to 0
      (AC4, AC7).
- [ ] 2.3 Implement the LLM provider call: 8s timeout, one retry
      (1s backoff) on 5xx/timeout/connection-reset only, no retry on 4xx;
      schema-validate the response before any write.
- [ ] 2.4 Implement per-tenant rate limiting (token bucket) in front of
      the LLM call.
- [ ] 2.5 Implement `GET /summaries/{ticket_id}`: always `200`, body
      `{status, content, generated_at, model_version}` covering
      `pending|complete|stale|failed|off` (AC1, AC4, AC7, AC8).
- [ ] 2.6 Implement `POST /summaries/{summary_id}/feedback`: upsert on
      `(summary_id, engineer_id)`, 5s timeout, `204` on success, `404`
      with `{error: "summary_not_found"}` if unknown (AC3, AC9).
- [ ] 2.7 Implement the frontend summary panel as an independently
      loading region: loading indicator, 20s request timeout, success
      (fresh), success (stale-with-indicator), "Summary unavailable" (no
      prior summary), and "off" (kill switch, shows nothing) states
      (AC1, AC8).
- [ ] 2.8 Implement the frontend thumbs up/down control: optimistic
      highlight on click, request fires in background (AC3).

## 3. Error Handling

- [ ] 3.1 Implement the unified failure display rule: a failed
      regeneration leaves a prior summary visible with a "may be out of
      date" indicator; "Summary unavailable" only when no prior summary
      exists.
- [ ] 3.2 Implement the consecutive-failure cap: after the 3rd
      consecutive failed attempt, stop auto-retry on ticket-open until a
      `message.added` event resets the counter (AC7).
- [ ] 3.3 Implement idempotent handling of duplicate `message.added` /
      `ticket.opened` delivery via the `(ticket_id,
      source_high_water_mark)` dedupe key (assumes at-least-once bus
      delivery — confirm against Flowdesk's actual bus guarantee).
- [ ] 3.4 Implement the feedback-submission failure path: one silent
      retry, then revert the selection and show the non-blocking
      "wasn't saved" message (AC9).

## 4. Instrumentation

- [ ] 4.1 Build the scheduled job (e.g. nightly) computing thumbs-down
      rate and sampled accuracy over AC6's rolling window; emit an alert
      on breach through Flowdesk's existing alerting channel (confirm
      this channel exists before implementation).
- [ ] 4.2 Build the offline evaluation harness: runs the generation path
      against a fixed, human-labeled golden set per `model_version`,
      independent of production traffic (AC2).
- [ ] 4.3 Confirm `model_version` is recorded on every summary row and
      wired into the eval harness for regression traceability.
- [ ] 4.4 *(blocked on an open decision — see sparring_brief.md Tier 2)*
      Add monitoring on the feedback-submission failure **rate** itself,
      distinct from feedback content already covered by 4.1.
- [ ] 4.5 *(blocked on an open decision — see sparring_brief.md Tier 2)*
      Define and instrument an operational SLO (e.g. p95 generation
      latency, availability rate) for Summary Service, distinct from
      AC5's business-outcome metric.

## 5. Tests

- [ ] 5.1 AC1: opening a ticket with no summary triggers generation;
      opening one with a fresh summary does not.
- [ ] 5.2 AC2: golden-set evaluation scores >= 90% under the defined
      rubric (no factual errors + states current status/open item,
      2-reviewer agreement, 3rd-reviewer tiebreak).
- [ ] 5.3 AC3 / AC9: vote persists, re-vote changes rather than
      duplicates it, and a simulated submission failure reverts the UI
      selection with the non-blocking message.
- [ ] 5.4 AC4: a message added while a ticket is open does not
      regenerate the summary live; reopening after new messages does.
- [ ] 5.5 AC5: the time-to-first-action measurement pipeline itself
      captures data correctly (the 40%/30-day target is a post-launch
      production measurement, not something to assert in a pre-launch
      test).
- [ ] 5.6 AC6: a simulated threshold breach raises an alert within the
      stated window; manually enabling the kill switch stops generation
      and the frontend shows the "off" state.
- [ ] 5.7 AC7: 3 consecutive simulated failures stop auto-retry on
      subsequent opens; a new message resets the counter.
- [ ] 5.8 AC8: a simulated 20s+ delay from Summary Service resolves the
      loading state into the failure display.
- [ ] 5.9 Idempotency: duplicate `message.added` / `ticket.opened`
      delivery produces no duplicate LLM calls or duplicate summary rows.
- [ ] 5.10 Regression: ticket detail page render path, ticket list/queue
      views, and existing Ticket Service API contracts are unaffected.

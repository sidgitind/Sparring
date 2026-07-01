# SPEC — Feature: AI Ticket Summary (v4 — post-full pipeline)

> **Example disclaimer:** Flowdesk is a fictional product created for this worked example.
> Any resemblance to real products is coincidental.

> This is the CONTRACT-GRADE spec — what the agent builds from.
> Every section and acceptance criterion in this document traces to a
> specific finding in Delta_Spec.md. Nothing was added speculatively.
> The finding reference is noted inline for traceability.

---

## Background

Support engineers have flagged context-switching as a major time sink.
Tickets often contain long thread histories. Engineers spend significant
time reading through threads before they can respond or take action.

We want to reduce time-to-first-action by surfacing an AI-generated
summary at the top of every ticket — with explicit design for the
cases where the summary is incomplete, stale, or operating under
conditions of non-trivial risk.

---

## Section 1 — Customer Definition
*(Delta Finding 1 — Amazon PM)*

**Primary user:** A support engineer making a consequential decision
on a ticket with non-trivial thread complexity — escalations,
multi-day threads, threads with status changes, threads with
multiple participants.

**Secondary user:** A support engineer triaging a routine,
low-complexity ticket (short thread, single issue, no status change).

The feature must serve both users correctly. Critically: it must not
optimise for the secondary user at the cost of the primary user.
The primary user is the one whose error is expensive.

---

## Section 2 — Accountability Model
*(Delta Finding 3 — Amazon PM)*

Engineers remain responsible for decisions and actions taken on
the basis of a summary. The product is responsible for surfacing
when a summary is operating in higher-risk territory.

This trade-off is explicit, not implicit. It is the design
contract between the product and the engineering team using it.
The concrete mechanisms that make this true are defined in
Section 6 (Risk Indicator) and Section 7 (Source Navigation).

---

## Section 3 — Acceptance Criteria

### AC1 — Summary generation
*(Delta Finding 1 — Amazon PM; Finding 18 — QA NFQ)*

- Summary generates automatically when an engineer opens a ticket
- p95 generation time from ticket open to summary render: < 1500ms
  (rolling 7-day measurement window)
- If generation exceeds 4000ms, the panel enters failure state
  (see AC3)

### AC2 — Summary accuracy and completeness
*(Delta Finding 2 — Amazon PM; Finding 14 — QA Functional)*

- Summary accuracy meets a minimum threshold of 90% against a
  human-reviewed test set (aggregate measure, unchanged from v1)
- In addition: structural completeness heuristics are computed
  per-ticket at generation time and surfaced to the engineer
  when thresholds are exceeded (see Section 6 — Risk Indicator)
- Heuristics are based on verifiable input signals only:
  - Message count (threshold: > 10 messages)
  - Time span covered (threshold: > 4 hours between first and
    last message)
  - Detected status or resolution change mid-thread
    (keyword-based heuristic, not semantic analysis)
  - Participant count (threshold: > 3 distinct participants)
- These heuristics do NOT represent the AI assessing its own
  correctness. They represent structural conditions under which
  summaries are more likely to be incomplete. This distinction
  must be preserved in all copy and documentation.

### AC3 — States
*(Delta Finding 11 — UI Designer; Finding 8 — Architect;
Finding 15 — QA Functional)*

The summary panel must handle four states explicitly:

**Loading state**
- Neutral loading indicator while summary is generating
- Full thread remains visible and accessible during generation
- No interaction blocked

**Success state**
- Summary rendered
- Risk indicator rendered if any heuristic threshold is exceeded
  (see Section 6)
- Thumbs up / thumbs down feedback toggle present

**Stale state**
*(Delta Finding 10 — Architect)*
- Triggered when new messages arrive after the last summary
  was generated
- Existing summary immediately marked: "May be outdated —
  [N] new message(s) since this summary"
- Banner appears above summary text, not below
- Regeneration begins automatically; summary updates on completion

**Failure state**
- Triggered when summarization service call exceeds 4000ms timeout
  or returns an error
- Summary panel collapses cleanly
- Full thread remains the primary view, fully accessible
- No error dialog, no blocked interaction, no spinner persisting
- Testable assertion: ticket page renders fully functional with
  no summary panel and no degraded interaction on service failure

### AC4 — Regeneration
*(Delta Finding 10 — Architect; Finding 20 — QA NFQ)*

- Summary regenerates when new messages are added to the thread
- Regeneration is debounced: a new message resets a short window
  (500ms); only one regeneration fires per window
- Last completed regeneration wins
- No partial or interleaved summary state is possible
- Stale-state banner appears immediately on new message arrival,
  before regeneration completes (see AC3)

### AC5 — Feedback
*(Delta Finding 6 — Google PM)*

- Thumbs up / thumbs down toggle present on every summary
  (unchanged from v1)
- Feedback is supplemented by instrumentation events (see
  Section 8 — Instrumentation) — the toggle alone is not the
  primary signal for feature health

---

## Section 4 — Scope and Rollout
*(Delta Finding 4 — Amazon PM; Finding 7 — Google PM)*

**Phase 1:** Tickets with low thread complexity — defined as:
fewer than 5 messages, single participant, no status change
detected. Summary generated with no risk indicator.

**Phase 2:** All remaining tickets. Summary generated with
risk indicator where applicable.

Phasing is a trust decision, not a performance decision.
Architect Finding 9 confirmed heuristic computation is cheap
and synchronous — there is no latency reason to delay Phase 2.
The phase boundary exists to establish trust with the simpler
case before applying the feature to the consequential one.

**Rollback trigger:** If full-thread expand rate (see Section 8)
climbs above 60% on any complexity tier within a rolling 7-day
window, that tier rolls back to no-summary display until the
signal is understood.

---

## Section 5 — Summarization Service Contract
*(Delta Finding 8 — Architect; Finding 17 — QA NFQ pre-flight)*

- Timeout: 4000ms per call
- On timeout: trigger failure state (AC3)
- On transient error: retry once, then trigger failure state
- On success: render summary panel with heuristic flags computed
  and cached alongside the summary
- Failure shape: ticket page renders fully, summary panel absent,
  no error surfaced to engineer

---

## Section 6 — Risk Indicator
*(Delta Finding 9 — Architect; Finding 12 — UI Designer;
Finding 3 — Amazon PM)*

When any structural completeness heuristic threshold is exceeded
(AC2), a risk indicator renders above the summary text.

**Visual treatment:**
- Labeled banner, not an icon or color change alone
- Banner text names the specific heuristic that triggered it
  (e.g., "Long thread, status changed — review full thread
  before acting")
- Prominent enough to be seen before the summary is read

**Purpose statement (for engineering and QA reference):**
The risk indicator is the product's side of the accountability
model defined in Section 2. Without visible and actionable
risk signaling, the accountability claim is aspirational.
The indicator must be impossible to miss in the moment it
fires — which is the highest-stakes moment in the workflow.

---

## Section 7 — Source Navigation
*(Delta Finding 13 — UI Designer)*

When a risk indicator is present, a "jump to relevant message"
link appears within the banner.

The link takes the engineer directly to the message location
in the thread that triggered the heuristic — the status change
message, the message that pushed the thread past the time
threshold, or the first message from a new participant.

**Purpose:** A risk flag without an efficient path to resolution
creates anxiety without action. The engineer must be able to
verify the flagged condition in fewer steps than reading the
full thread — otherwise the summary feature is net-negative
for the consequential case.

---

## Section 8 — Success Metrics and Counter-Metrics
*(Delta Finding 5 — Google PM; Finding 6 — Google PM)*

**Primary metric:**
Time-to-first-action per ticket (engineer opens ticket to
first action taken), measured as a rolling 7-day p50 and p95.
Target: measurable reduction versus pre-feature baseline
within 30 days of Phase 1 launch.

**Counter-metric:**
*(This is the metric the team must NOT sacrifice)*
Rate of post-resolution escalations and reopens on tickets
where a summary was viewed, compared against a matched
cohort where the engineer expanded the full thread before
acting. Tracked from Phase 1 launch. Any statistically
significant increase in escalation rate on the summary-viewed
cohort is a rollback signal regardless of primary metric
performance.

**Trust signal:**
Full-thread expand rate (see Section 9). High expand rate
indicates engineers are reading both summary and thread —
either because they do not trust the summary or because
the risk indicator is triggering correctly. These are
different signals and must be distinguished in analysis.

---

## Section 9 — Instrumentation
*(Delta Finding 6 — Google PM; Finding 19 — QA NFQ)*

**Product analytics events (from Phase 1 launch):**

| Event | Properties |
|---|---|
| summary_viewed | ticket_id, summary_version, complexity_tier, risk_indicator_shown |
| full_thread_expanded | ticket_id, summary_version, time_since_summary_view |
| action_taken | ticket_id, summary_version_at_action_time, risk_indicator_shown |
| feedback_submitted | ticket_id, summary_version, thumbs_up / thumbs_down |
| risk_indicator_shown | ticket_id, heuristics_triggered (array) |

**Operational monitoring (from Phase 1 launch):**

Summarization service error rate:
- Warning: > 2% of calls failing in any 5-minute window
- Critical: > 5% of calls failing in any 5-minute window

Summarization service latency:
- Warning: p95 > 1200ms (rolling 5-minute window)
- Critical: p95 > 1500ms (rolling 5-minute window)

Both alerts must be in place before Phase 1 launches.
Silent production degradation is not acceptable on a feature
that is load-bearing to support team throughput.

---

## Section 10 — Out of Scope

- Summarization of attachments or screenshots
- Multi-language summary generation
- Summary editing by the engineer
- Semantic analysis of thread content for completeness
  assessment (structural heuristics only — see AC2)

---

## Section 11 — Regression Surface
*(Delta Finding 16 — QA Functional)*

This feature touches the following existing components.
All must be regression-tested before Phase 1 launch:

- Ticket detail view (new panel insertion point)
- Thread rendering component (regeneration trigger on
  new message — must not affect thread display or
  scroll position)
- Existing ticket-state logic (mark as read, ticket
  status changes — must not be affected by summary
  regeneration or stale-state detection)

---

## Persona Bias Acknowledgments

Every finding in this spec was generated by a Sparring persona
and evaluated by the PM before acceptance. The following biases
were noted and managed:

**Amazon PM:** Will demand a named customer narrative even on
features where the customer is already understood. The customer
definition in Section 1 satisfies this requirement.
The persona did not require further escalation.

**Google PM:** Will request instrumentation depth that a small
team cannot sustain. Section 9 reflects what this team can
realistically operate, not the full instrumentation the Google
PM persona proposed. Three proposed events were deferred to
Phase 2.

**Architect:** Will flag every async/sync boundary regardless
of actual risk. The heuristic computation decision (synchronous,
cached) was validated as low-risk before acceptance.

**UI Designer:** Will default to minimal visual treatment on
warning states. The risk indicator visual spec (Section 6)
overrides the persona's initial recommendation — the banner
treatment was specified explicitly to prevent the indicator
from being missable.

**QA Functional:** Will ask for regression coverage beyond
what the team can deliver in one sprint. Section 11 is scoped
to what is realistic for Phase 1. Full regression coverage
is a Phase 2 commitment.

**QA NFQ:** Will ask for SLOs and monitoring on every feature
regardless of criticality. All NFQ requirements in this spec
were accepted without override — this feature's load-bearing
nature to support team throughput justifies the full ask.

---

## Open Items Before Engineering Starts

- [ ] Legal review: does the audit trail in Section 9
      (action_taken event linked to summary_version) create
      any data retention or privacy obligations?
- [ ] Confirm summarization service provider and validate
      4000ms timeout is achievable at p95 under production load
- [ ] Define "status change" keyword list for heuristic
      detection — must be reviewed by support team lead
      before Phase 1 launch
- [ ] Confirm rollback mechanism: who is authorised to trigger
      a tier rollback, and what is the response time SLA?

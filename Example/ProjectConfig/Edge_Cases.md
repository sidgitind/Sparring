# EDGE_CASES.md — Flowdesk

> Discovered failures with extracted rules.
> Every entry here has a corresponding rule in ARCHITECTURE.md.
> Rules here are the evidence. ARCHITECTURE.md is the law.

---

## EDGE-001

**Discovery date:** Post-launch — week two  
**Discovered in:** Feature 2 — Third-party webhook integration  
**Surface:** Production — detected via support team escalation

**Behavior observed:**
- Flowdesk integrated with a third-party project management tool
  via outbound webhook on ticket status change
- Engineers updated ticket status; UI showed success immediately
  (optimistic write)
- Webhook call was made after the UI update, with no timeout defined
- Under third-party API slowdown, the webhook call hung at Node.js
  default (no timeout — indefinite wait)
- Ticket page became unresponsive while the hung call blocked the
  event loop on the route handler
- Engineers on a high-volume support floor reported tickets "freezing"
  intermittently — pattern was load-dependent and hard to reproduce
  in testing
- Secondary failure: on webhook timeout the state had already been
  written optimistically to the UI. The downstream system had not
  received the update. Engineer saw "success"; third-party tool
  showed the old status.

**Root cause:**
Webhook call made directly from a route handler with no timeout,
no failure response shape, and an optimistic state write that
committed before external confirmation.

**Resolution applied:**
- All external service calls moved to /integrations module
- Explicit 3000ms timeout on all outbound webhook calls
- On timeout or error: surface error to user, revert UI to pre-action
  state, do not commit state change
- Retry once on transient error (5xx); surface error on second failure
- Route handlers never call external services directly

**Rule extracted:**
Every external service call must define an explicit timeout, failure
response shape, and retry strategy. System state is never written
before external service confirmation — no optimistic writes on
external calls. All external calls routed through /integrations only.
See Architecture.md — Hard Boundaries Rule 3, External Service Contracts.

---

## EDGE-002

**Discovery date:** Post-launch — week one  
**Discovered in:** Feature 3 — Bulk ticket assignment  
**Surface:** Production — detected via QA regression report and
engineer complaints

**Behavior observed:**
- Bulk ticket assignment feature allowed team leads to reassign
  multiple tickets simultaneously
- Feature was built as a utility function that wrote ticket-state
  (assigned_to, status) directly to /db, bypassing /tickets module
- On concurrent bulk assignments (two team leads running bulk assign
  simultaneously), ticket state became inconsistent:
  - Some tickets showed the first assignee, some the second,
    with no deterministic outcome
  - Mark-as-read status was reset on some tickets that had already
    been read, because the bulk write did not check existing state
    before overwriting
- Additionally: a status badge on the ticket list view was computed
  by an async background job triggered after the bulk assignment
  completed. Engineers saw the ticket list re-render with old badges
  immediately after assignment, then badges updated 2-3 seconds later
  in a second render — causing confusion about whether the assignment
  had taken effect

**Root cause:**
Two independent failures in the same feature:
1. Ticket-state writes outside /tickets module — no coordination
   with existing state logic, produced inconsistent concurrent writes
2. Status badge computation triggered as async job after the primary
   write — race condition between the ticket list rendering and the
   badge computation completing, producing a visibly inconsistent
   intermediate state

**Resolution applied:**
- All ticket-state writes routed through /tickets module only
- /tickets module enforces write ordering and checks existing state
  before overwriting on concurrent requests
- Status badge computation moved to synchronous, in-process calculation
  at render time — no async job, no race condition
- Regression test suite added for ticket-state logic covering
  concurrent write scenarios

**Rule extracted:**
Ticket-state logic lives in /tickets only — no other module writes
ticket state directly. Any computation that must appear in the same
render cycle as a UI element it accompanies must be synchronous and
in-process — never async or queued. An async companion job produces
a visible intermediate state that users act on before it resolves.
See Architecture.md — Hard Boundaries Rule 4 and Rule 5,
Known Regression Surfaces.

---

## EDGE-003

**Discovery date:** Post-launch — week three  
**Discovered in:** Feature 3 — Bulk ticket assignment  
**Surface:** Production — detected via engineer complaints,
confirmed by session recording review

**Behavior observed:**
- After bulk ticket assignment shipped, engineers reported that
  the ticket detail view occasionally showed stale data after
  navigating to a ticket that had just been reassigned
- Investigation: the ticket detail view was caching the GET response
  for the ticket record for 3 minutes via a global API cache setting
- Engineers who opened a ticket within the cache window saw the
  pre-assignment state — old assignee, old status
- Some engineers took action on tickets (adding notes, updating status)
  based on the stale view, not knowing the ticket had already been
  reassigned and was being worked by another engineer
- Duplicate work and conflicting ticket updates followed

**Root cause:**
Spec for bulk ticket assignment did not define cache invalidation
behaviour for the ticket detail view. The global API cache setting
was inherited silently. No spec instruction to bust the ticket cache
on successful assignment write.

**Resolution applied:**
- Ticket detail GET endpoint marked Cache-Control: no-store
- On any successful ticket-state write: frontend invalidates cached
  ticket record and re-fetches before rendering confirmation
- Inline confirmation only shown after re-fetch confirms the written
  value matches what was returned
- Ticket detail view added to known regression surface list —
  any new feature that writes ticket state must verify ticket detail
  view renders live data, not cached data

**Rule extracted:**
Any feature that writes state and then reads it back must specify
cache invalidation behaviour explicitly in the spec. Inheriting a
global cache setting is not a safe default — it is an unexamined
assumption. The source of truth for a view must be stated: what
triggers a re-fetch, and what confirmation is shown only after the
re-fetch confirms the write. See Architecture.md — Known Regression
Surfaces.

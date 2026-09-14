# ARCHITECTURE.md — Flowdesk

> This file is read by every persona before reviewing any spec.
> It is the architectural constitution. Violating it requires explicit human override.

---

## Product Context

Flowdesk is a B2B ticket management tool used by support engineering teams.
Support engineers open tickets, read thread histories, and take action on
customer issues. Tickets can contain long thread histories with multiple
participants, status changes, and attached context.

---

## Module Map

```
/src
├── /api                Route handlers only. No business logic.
├── /services           Business logic. Calls DB and external services.
├── /db                 All database queries. No raw SQL outside this folder.
├── /utils              Pure functions. No side effects. No DB calls.
├── /tickets            Ticket reads, thread rendering, ticket-state logic.
├── /integrations       All third-party and external service calls.
│                       Each integration has its own subfolder.
│                       No external calls outside this module.
└── /middleware         Express middleware. Auth guards live here.
```

---

## Hard Boundaries

```
Rule 1: Route handlers (/api) never call the database directly.
        They call /services. Services call /db.
        Reason: Keeps business logic testable in isolation.

Rule 2: No raw SQL outside /db. Ever.
        Reason: SQL injection prevention. Query visibility for auditing.
        Testability — /db can be mocked in isolation.

Rule 3: All external service calls live in /integrations only.
        /services calls /integrations. Route handlers do not call
        external services directly.
        Reason: Single audit surface for all outbound calls.
        Timeouts, retries, and failure handling are defined and
        debugged in one place.
        Source: EDGE-001 — third-party webhook integration hung the
        ticket page under load when called directly from a route handler
        with no timeout.

Rule 4: Ticket-state logic lives in /tickets only.
        No other module writes ticket state directly.
        Reason: Ticket state (open, closed, assigned, status changes)
        must not be affected by unrelated feature logic running
        alongside it. Coupling creates untestable state and
        introduces regression risk on every new feature that
        touches the ticket view.
        Source: EDGE-002 — a bulk-action feature wrote ticket state
        from a utility function outside /tickets, producing inconsistent
        state on concurrent updates.

Rule 5: Any computation that must appear in the same render cycle
        as a UI element it accompanies must be synchronous and
        in-process — never async or queued.
        Reason: An async job creates a race condition between the
        primary element rendering and its companion signal appearing.
        Users see the element without the signal, act on incomplete
        information, and the signal arrives after the damage is done.
        Source: EDGE-002 — status badge on ticket list rendered
        before the async job that computed it had completed,
        causing engineers to see stale status on active tickets.

Rule 6: No new top-level folders without explicit human instruction.
        Reason: Folder proliferation is the first sign of architectural drift.
```

---

## Current Tech Decisions (locked — do not change without instruction)

```
Framework:          Express (not Fastify, not Koa)
Database client:    pg library (not Prisma, not Sequelize)
Auth:               JWT — HS256 algorithm
External services:  All calls routed through /integrations.
                    Each integration subfolder owns its timeout,
                    retry strategy, and failure response shape.
                    No provider-specific logic outside its subfolder.
```

---

## External Service Contracts

> Extracted from EDGE_CASES.md. Every external call must comply.
> Non-compliance is a blocking issue in QA NFQ persona review.

Every external service call must define:

1. **Timeout** — explicit ms value, never rely on language or library default
2. **Failure response shape** — what the user sees and what state is written
   on timeout or error
3. **Retry strategy** — explicit retry logic OR explicit "no retry"
4. **State write policy** — system state is never written before external
   service confirmation. No optimistic writes on external calls.

**Current service timeouts:**
```
PostgreSQL queries:      2000ms timeout
Third-party webhooks:    3000ms timeout (source: EDGE-001)
```

New integrations must define their own timeout in their subfolder
before the first call is made. There is no global default — the
absence of an explicit timeout is a spec gap, not a safe fallback.

**State write policy (global):**
```
System state is written ONLY after external service confirmation.
On external service failure: surface the error to the user,
revert any optimistic UI state, do not commit the state change.
```

Source: EDGE-001 — webhook integration written optimistically;
on failure the UI showed success while the downstream system had
not received the update.

---

## Known Regression Surfaces

Features that have caused regression in the past. Any new feature
that touches these areas must include explicit regression testing
before launch.

```
Ticket detail view:     High regression surface. Multiple features
                        have introduced rendering bugs by inserting
                        new panels or modifying the thread component
                        without testing adjacent functionality.
                        Source: EDGE-002, EDGE-003.

Ticket-state logic:     Mark as read, status change, assignment —
                        all have been broken by features that wrote
                        state outside /tickets.
                        Source: EDGE-002.

Thread rendering:       Scroll position, message ordering, and
                        new-message detection have all produced
                        bugs when touched by unrelated features.
                        Source: EDGE-003.
```

---

## What Does Not Exist Yet

```
- AI-generated content of any kind (no summarization, no suggestions,
  no auto-complete)
- Per-engineer feature preferences or opt-out controls
- Audit log beyond standard application logging
- Frequency or scheduling controls on any feature
- Multi-language content generation
```

Do not build these until specced. Do not reference them in code as stubs.

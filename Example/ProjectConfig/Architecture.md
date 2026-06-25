# ARCHITECTURE.md — Flowdesk

> This file is read by every persona before reviewing any spec.
> It is the architectural constitution. Violating it requires explicit human override.

---

## Module Map

```
/src
├── /api                Route handlers only. No business logic.
├── /services           Business logic. Calls DB and external services.
├── /db                 All database queries. No raw SQL outside this folder.
├── /utils              Pure functions. No side effects. No DB calls.
├── /notifications      Notification dispatch — email via SendGrid, in-app via WebSocket.
├── /preferences        User notification preference reads and writes.
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

Rule 3: Notification dispatch logic lives in /notifications only.
        /services calls /notifications. Route handlers do not dispatch directly.
        Reason: Single audit surface for all outbound communication.
        Deliverability failures are debugged in one place.

Rule 4: Preference reads and writes live in /preferences only.
        /services calls /preferences. No direct DB calls from /notifications.
        Reason: Preference logic changes independently of dispatch logic.
        Coupling them creates untestable state.

Rule 5: No new top-level folders without explicit human instruction.
        Reason: Folder proliferation is the first sign of architectural drift.
```

---

## Current Tech Decisions (locked — do not change without instruction)

```
Framework:        Express (not Fastify, not Koa)
Database client:  pg library (not Prisma, not Sequelize)
Auth:             JWT — HS256 algorithm
Email:            SendGrid only — no fallback provider
Unsubscribe link: Signed token — HMAC-SHA256, user-specific, single-use
Token storage:    Signed tokens stored in /db — invalidated on first use
```

---

## External Service Contracts

> Extracted from EDGE_CASES.md. Every external call must comply.
> Non-compliance is a blocking issue in QA NFQ persona review.

Every external service call must define:

1. **Timeout** — explicit ms value, never rely on default
2. **Failure response shape** — exact HTTP status and body on failure
3. **Retry strategy** — explicit retry logic OR explicit "no retry"
4. **State write policy** — does user/system state get written before or after confirmation?

**Current service timeouts:**
```
SendGrid API:       3000ms timeout
PostgreSQL queries: 2000ms timeout
```

**State write policy (global):**
```
Preference state is written ONLY after database confirmation.
No optimistic preference writes — the toggle must not move before the write succeeds.
```

Source: EDGE-001 (silent SendGrid failure discovered in testing)

---

## What Does Not Exist Yet

```
- Notification frequency controls (do not disturb, scheduling)
- Per-device push notification settings
- Product update / weekly digest / in-app alert toggles (deferred to v2)
- Preference history or audit log
```

Do not build these until specced. Do not reference them in code as stubs.

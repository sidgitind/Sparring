# ARCHITECTURE.md — Flowdesk

> This file is read by every agent and every persona before touching this project.
> It is the architectural constitution. Violating it requires explicit human override.

---

## Module Map

```
/src
├── /auth               All authentication logic. Nowhere else.
├── /api                Route handlers only. No business logic.
├── /services           Business logic. Calls DB and external services.
├── /db                 All database queries. No raw SQL outside this folder.
├── /utils              Pure functions. No side effects. No DB calls.
├── /websocket          WebSocket server and connection management.
├── /notifications      Notification dispatch — in-app and email.
└── /middleware         Express middleware. Auth guards live here.
```

---

## Hard Boundaries

```
Rule 1: Route handlers (/api) never call the database directly.
        They call /services. Services call /db.
        Reason: Keeps business logic testable in isolation. Route handlers change
        frequently; business logic should not be coupled to HTTP concerns.

Rule 2: Auth logic never lives in /utils or /middleware.
        /middleware calls /auth. It does not contain auth logic.
        Reason: Auth logic in utils is untraceable. Central /auth module means
        one audit surface, one place to fix a vulnerability.

Rule 3: No new top-level folders without explicit human instruction.
        Reason: Folder proliferation is the first sign of architectural drift.
        New folders should be decisions, not defaults.

Rule 4: No raw SQL outside /db. Ever.
        Reason: SQL injection prevention. Query visibility for auditing.
        Testability — /db can be mocked in isolation.

Rule 5: WebSocket logic never lives in /api or /services.
        It lives in /websocket only.
        Reason: WebSocket lifecycle (connect/disconnect/reconnect) is fundamentally
        different from request/response. Mixing them creates untestable state.
```

---

## Current Tech Decisions (locked — do not change without instruction)

```
Framework:        Express (not Fastify, not Koa)
Database client:  pg library (not Prisma, not Sequelize)
Auth:             JWT — HS256 algorithm
Token storage:    Access token: memory (frontend) / Refresh token: httpOnly cookie
Email:            SendGrid only — no fallback provider
WebSocket:        ws library (not Socket.io)
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
Google OAuth API:   5000ms timeout
SendGrid API:       3000ms timeout
PostgreSQL queries: 2000ms timeout
```

**State write policy (global):**
```
Auth state is written ONLY after full external service confirmation.
No optimistic auth writes. Ever.
```

Source: EDGE-001 (Google OAuth delay discovered in testing)

---

## What Does Not Exist Yet

```
- Refresh token rotation logic
- Password reset flow
- Admin workspace management
- Billing integration
- Notification preference management
```

Do not build these until specced. Do not reference them in code as stubs.

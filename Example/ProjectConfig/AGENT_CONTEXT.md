# AGENT_CONTEXT.md — Flowdesk

> Updated at the end of every session.
> Pasted at the start of every new session before any instruction.
> This is the agent's memory between sessions.

---

## Current Feature In Progress

Feature 1 — Google OAuth Login
Spec location: /Example/00_input_spec.md

---

## System State Right Now

```
Auth module:          Not started
Dashboard:            Not started
Notification system:  Not started
Database schema:      Defined, not implemented
API contracts:        Not locked — frontend not started
```

---

## What Is Stable (do not change without instruction)

```
- Tech stack decisions in ARCHITECTURE.md
- Module folder structure
- Token storage strategy (access: memory, refresh: httpOnly cookie)
- External service timeout values
```

---

## Last Session Summary

Session 1 — Repo structure and mockup project created.
Files created:
- /Example/ProjectConfig/PROJECT.md
- /Example/ProjectConfig/ARCHITECTURE.md
- /Example/ProjectConfig/EDGE_CASES.md
- /Example/00_input_spec.md

Decisions made:
- Flowdesk chosen as mockup project
- Feature 1 (Google OAuth Login) chosen as worked example feature
- Three edge cases pre-seeded in EDGE_CASES.md to demonstrate rule extraction

Not done yet:
- Persona files (Sessions 2, 3, 4)
- Worked example pipeline run (Session 5)
- Compatibility matrix, starter kits (Session 6)
- README, HOW-IT-WORKS (Session 7)

---

## Known Decisions Made (and why)

| Decision | Reason |
|---|---|
| HS256 for JWT | No multi-service setup — RS256 overhead not justified yet |
| httpOnly cookie for refresh token | XSS protection — localStorage is not acceptable |
| pg library not ORM | Keeps query visibility explicit — team preference |
| No refresh token rotation in scope | Deferred — not in Feature 1 spec |

---

## Do Not Touch

```
- ARCHITECTURE.md module map (locked)
- Token storage decisions
- External service timeout values
```

---

## Open Questions (unresolved)

1. Should auto-created accounts (new Google users) require email verification?
   Currently: spec says create automatically — no verification step defined.
   Risk: persona pipeline will flag this. Decision deferred to PM persona review.

2. What happens if user revokes Google OAuth access post-login?
   Currently: not addressed in spec.
   Risk: QA NFQ will flag this. Resolution pending persona review.

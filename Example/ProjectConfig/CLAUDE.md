# CLAUDE.md — Flowdesk

> Read this file first. Every session. No exceptions.

---

## What This Project Is

Flowdesk is a fictional B2B SaaS project used as the worked example
for the Sparring persona library.
Full context: /Example/ProjectConfig/PROJECT.md

---

## Before You Write Any Code

1. Read AGENT_CONTEXT.md — current state of the project
2. Read ARCHITECTURE.md — module boundaries and hard rules
3. Read the relevant SPEC file — what you are building
4. Read EDGE_CASES.md — what has already broken and why

If any of these files are missing or empty: stop and tell me.
Do not proceed on assumptions.

---

## Before You Create Any New File or Function

1. Check ARCHITECTURE.md — does this belong in an existing module?
2. Check AGENT_CONTEXT.md SYSTEM_MAP section — does this already exist?
3. If creating a new module: flag it to me before proceeding

---

## Before You Touch Any External Service Integration

Read ARCHITECTURE.md section "External Service Contracts".
Verify the spec explicitly defines:
- Timeout value (ms)
- Failure response shape
- Retry strategy or explicit no-retry
- State write policy on failure

If any of these are missing in the spec: STOP.
Tell me what is missing. Do not invent values.

---

## Standing Instructions — End of Every Session

After completing the task, before closing:

1. Update AGENT_CONTEXT.md:
   - What was completed
   - Files changed
   - Decisions made and why
   - What is not done yet
   - Any new open questions

2. If a bug was fixed: update BUGS_[feature].md status to FIXED

3. If a new function or module was created: add it to SYSTEM_MAP
   in AGENT_CONTEXT.md

Do not ask me if you should do this. Just do it.

---

## Constraints That Are Never Negotiable

- Auth state is written only after full external service confirmation
- No raw SQL outside /db
- No business logic in /api route handlers
- No auth logic in /utils
- httpOnly cookie for refresh token — this is not up for debate

---

## One Task Per Session

If I give you a task that is too large for one session,
tell me how you would break it down and ask me which part to start with.
Do not attempt to build the entire feature in one session.

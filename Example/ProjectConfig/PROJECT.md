# Mockup Project: Flowdesk

> This is a fictional project created for the Sparring library worked examples.
> It is intentionally realistic — complex enough to stress every persona,
> simple enough to follow without domain expertise.

---

## What It Is

**Flowdesk** is a B2B SaaS team productivity tool for small and mid-sized teams (5–50 people).

It lets teams:
- Create and assign tasks with due dates and priority levels
- Authenticate via Google OAuth (primary) or email/password (fallback)
- View a personal dashboard showing their tasks, team activity, and deadlines
- Receive in-app and email notifications for assignments, mentions, and due date reminders

Think: a lightweight Asana, without the enterprise complexity.

---

## Users

| Persona | Role | Primary Need |
|---|---|---|
| Team Member | Creates and completes tasks | See what I need to do today |
| Team Lead | Assigns tasks, tracks progress | See what my team is working on |
| Admin | Manages workspace, billing | Control access and settings |

---

## Tech Stack (mockup — fictional but realistic)

```
Frontend:     React 18, TypeScript, Tailwind CSS
Backend:      Node.js, Express
Database:     PostgreSQL
Auth:         Google OAuth 2.0 + JWT (access + refresh tokens)
Notifications: SendGrid (email), in-app via WebSocket
Hosting:      AWS (EC2 + RDS + SES)
```

---

## Three Features In Scope (for worked examples)

### Feature 1 — Google OAuth Login (Session 5 worked example)
The authentication flow. Primary entry point for all users.
External dependency: Google OAuth 2.0 API.
This is the worked example feature — chosen because it has external service dependency,
user state implications, UI requirements, and non-trivial architecture decisions.

### Feature 2 — Personal Dashboard
User's home screen after login.
Shows: my tasks (filtered by due date), team activity feed, overdue alerts.
Depends on Feature 1 (auth) being complete.

### Feature 3 — Notification System
In-app notifications via WebSocket + email via SendGrid.
Most complex feature — two external services, real-time requirements,
user preference management.

---

## Known Constraints

- Google OAuth is the primary login — email/password is fallback only
- JWT access token: 15 minute expiry
- JWT refresh token: 7 day expiry, stored in httpOnly cookie
- WebSocket connections must survive server restarts gracefully
- SendGrid is the only approved email provider — no fallback email service
- All user data is tenant-scoped — no cross-workspace data leakage permitted

---

## What This Project Is Designed To Test

| Persona | What Flowdesk surfaces |
|---|---|
| PM | Unclear success metrics on auth flow, missing edge case on OAuth failure |
| Architect | JWT storage decisions, refresh token strategy, WebSocket state |
| UI Designer | OAuth button standards (Google brand guidelines), loading states, error messages |
| QA Functional | Happy path vs OAuth failure vs network timeout paths |
| QA NFQ | Google API timeout, SendGrid failure, WebSocket reconnection, token expiry under load |

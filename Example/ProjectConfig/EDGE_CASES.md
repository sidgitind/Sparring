# EDGE_CASES.md — Flowdesk

> Discovered failures with extracted rules.
> Every entry here has a corresponding rule in ARCHITECTURE.md.
> Rules here are the evidence. ARCHITECTURE.md is the law.

---

## EDGE-001

**Discovery date:** Pre-launch testing  
**Discovered in:** Feature 1 — Google OAuth Login  
**Surface:** QA NFQ testing phase

**Behavior observed:**
- Auth middleware used Node.js default HTTP timeout (~120s)
- Google OAuth API occasionally delayed 4–6s under load
- No timeout defined on the call
- On delay: frontend showed blank loading state with no feedback
- On eventual failure: returned 500 (unhandled promise rejection)
- User session state was partially written before Google confirmed — caused ghost sessions

**Root cause:**
No explicit timeout defined on external service call.
No failure response shape specified in spec.
No state write policy — auth state written optimistically.

**Resolution applied:**
- Explicit 5000ms timeout on all Google OAuth API calls
- On timeout: return 503 with `{ error: 'auth_service_unavailable', retryAfter: 30 }`
- Auth state written only after full Google confirmation
- Frontend shows explicit error message with retry option after 5s

**Rule extracted:**
All external service calls must define timeout, failure shape, retry strategy, and state write policy.
See ARCHITECTURE.md — "External Service Contracts"

---

## EDGE-002

**Discovery date:** Pre-launch testing  
**Discovered in:** Feature 1 — Google OAuth Login  
**Surface:** QA Functional testing phase

**Behavior observed:**
- OAuth callback handler did not validate `state` parameter
- CSRF attack vector: attacker could substitute their OAuth code for victim's
- No state parameter generated on OAuth initiation
- No state parameter validated on callback

**Root cause:**
Spec did not mention CSRF protection on OAuth flow.
Agent implemented happy path only.

**Resolution applied:**
- Generate cryptographic `state` parameter on OAuth initiation
- Store in server-side session (not cookie, not localStorage)
- Validate on callback — mismatch returns 403
- State parameter single-use — invalidated after use

**Rule extracted:**
All OAuth flows must implement state parameter CSRF protection.
Added to QA Functional persona as mandatory check on any auth spec.

---

## EDGE-003

**Discovery date:** Pre-launch testing  
**Discovered in:** Feature 1 — Google OAuth Login  
**Surface:** QA NFQ testing — token expiry scenarios

**Behavior observed:**
- Access token (15min expiry) expired during active user session
- API calls started returning 401
- Frontend had no refresh logic — showed raw 401 error to user
- User had to manually log out and log back in
- On some browsers: refresh token cookie was blocked by third-party cookie policy

**Root cause:**
Spec defined token expiry values but did not specify:
- How the frontend handles 401 mid-session
- What happens when refresh token is unavailable
- Browser cookie policy implications of httpOnly cookie on subdomain

**Resolution applied:**
- Frontend intercepts 401 → attempts silent refresh via `/auth/refresh`
- On successful refresh: retry original request transparently
- On failed refresh (expired/missing refresh token): redirect to login with `session_expired` query param
- Cookie set with `SameSite=Strict` and explicit domain — not subdomain wildcard

**Rule extracted:**
Auth specs must define the full token lifecycle:
initiation → expiry → refresh → refresh failure → re-authentication.
Silence on any step = incomplete spec = BLOCK from Architect persona.

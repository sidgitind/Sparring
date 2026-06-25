# EDGE_CASES.md — Flowdesk

> Discovered failures with extracted rules.
> Every entry here has a corresponding rule in ARCHITECTURE.md.
> Rules here are the evidence. ARCHITECTURE.md is the law.

---

## EDGE-001

**Discovery date:** Pre-launch testing  
**Discovered in:** Feature 3 — Notification Preferences  
**Surface:** QA NFQ testing phase

**Behavior observed:**
- User toggled marketing email preference off
- Preference write to DB succeeded
- SendGrid suppression list update was called after DB write
- SendGrid returned 429 (rate limit) on the suppression API
- Error was caught and logged — but not surfaced to the user
- DB preference showed "unsubscribed"; SendGrid still had the user active
- User received another marketing email the next day
- Support ticket raised: "I unsubscribed but still got an email"

**Root cause:**
SendGrid suppression list update was treated as a fire-and-forget call.
No timeout defined. No failure response shape in spec.
No retry strategy. Silent failure on 429.

**Resolution applied:**
- Explicit 3000ms timeout on all SendGrid API calls
- On failure: return error to user, revert toggle to previous state
- Error message: "Couldn't save your preference. Try again."
- Retry once on 429 after 2 seconds — if second attempt fails, surface error
- Preference DB write and SendGrid update treated as a two-step operation:
  DB write held in pending state until SendGrid confirms, then committed

**Rule extracted:**
All external service calls must define timeout, failure response shape,
and retry strategy. Preference state is not committed until all
external service updates confirm. See ARCHITECTURE.md — "External Service Contracts"
and "State write policy".

---

## EDGE-002

**Discovery date:** Pre-launch testing  
**Discovered in:** Feature 3 — Notification Preferences  
**Surface:** QA Functional testing phase

**Behavior observed:**
- Unsubscribe link in marketing email used a non-expiring token
- Token contained encoded user ID only — no signature validation
- Attacker constructed a valid-looking unsubscribe URL for another user
  by encoding a different user ID
- No validation on the server — unsubscribe processed silently
- Affected user stopped receiving marketing emails without their knowledge

**Root cause:**
Spec defined an unsubscribe link but did not specify token structure,
signature validation, or single-use enforcement.
Agent implemented the simplest possible token: base64-encoded user ID.

**Resolution applied:**
- Unsubscribe token is HMAC-SHA256 signed with server secret
- Token is user-specific and single-use — invalidated in DB after first use
- On reuse: return 410 Gone — "This unsubscribe link has already been used."
- Token expiry: 30 days — after which user must use Settings page

**Rule extracted:**
All one-click action links (unsubscribe, confirmation, password reset)
must use signed, single-use tokens. Token structure and validation
behaviour must be explicitly defined in the spec before build.
Added to QA Functional persona as mandatory check on any
one-click action flow.

---

## EDGE-003

**Discovery date:** Post-launch monitoring  
**Discovered in:** Feature 3 — Notification Preferences  
**Surface:** Production — caught by support tickets

**Behavior observed:**
- User saved preferences successfully
- User navigated away and returned to Settings > Notifications
- Toggle showed original state (opted in) not saved state (opted out)
- On investigation: DB write had succeeded; UI was reading a cached
  API response, not the live DB value
- Frontend was caching the preferences GET response for 5 minutes
- User saw stale state; assumed their change had not saved
- Some users toggled again — re-opting themselves in

**Root cause:**
Spec did not define cache invalidation behaviour on the preferences GET endpoint.
Frontend caching was inherited from a global API caching setting.
No spec instruction to bust cache on successful preference write.

**Resolution applied:**
- Preferences GET endpoint marked Cache-Control: no-store
- On successful preference write: frontend invalidates local cached value
  and re-fetches before rendering the settings page
- Inline confirmation ("Saved") only shown after re-fetch confirms
  the value matches what was written

**Rule extracted:**
Any feature that writes state and then reads it back must specify
cache invalidation behaviour explicitly. "Settings persist after
page refresh" is not a sufficient acceptance criterion — it must
specify what the source of truth is and when the UI re-fetches it.

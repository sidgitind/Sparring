# SPEC — Feature: Notification Preferences (v4 — post-full-pipeline review)

> **Example disclaimer:** Flowdesk is a fictional product created for this worked example.
> Any resemblance to real products is coincidental. The email dispatch service referenced
> in this spec uses "SendGrid" as a placeholder name — substitute your actual email
> provider. No real product or company is implied or endorsed.

> This is the output after running all five personas: Amazon PM, Google PM,
> Architect, UI Designer, QA Functional, and QA NFQ — all in **detailed mode**.
> Detailed mode produces a full structured review across every dimension.
> Summary mode (the default) produces verdict + top 2 blockers + conditional items only.
> This example uses detailed mode deliberately — it shows the full depth of what each persona catches.
> Every addition is traceable to a specific persona finding.
> See DELTA.md for the full change map.

---

## Problem This Solves

73% of notification-related support tickets are specifically about
marketing emails. Users report not knowing they were subscribed,
or not being able to find the unsubscribe option.
47 marketing email support tickets per month (3-month average baseline).

This is a discoverability and control problem — not a notification
volume problem across all types.

---

## Customer

Existing user who receives marketing emails and has not opened
one in the past 60 days.

Not in scope for v1:
- Users who want to reduce notification frequency (different problem)
- New users who have never set preferences (no established behaviour yet)
- Users on free tier (different email cadence — separate decision)

---

## What We Are Building (Scoped)

Marketing email unsubscribe — v1 scope only.

The full 5-toggle preference centre is deferred.
Rationale: support ticket data shows 73% of complaints are marketing
emails specifically. Building 5 toggles solves 27% of the problem
at 5x the cost. Start with the one that matters. Measure. Expand.

---

## Legal Review

Marketing email unsubscribe has regulatory implications.
CAN-SPAM (US) and GDPR (EU) require:
- Unsubscribe must be honoured within 10 business days
  (we will process instantly)
- A physical mailing address in every marketing email
- No requirement to provide a reason for unsubscribing

Legal reviewed: confirmed.
Owner: [Legal contact name]

---

## Acceptance Criteria

### Unsubscribe Flow

1. Every marketing email contains an unsubscribe link in the footer.
   Link text: "Unsubscribe from marketing emails".
   Testable requirements: not hidden; not styled as body text;
   minimum font size 12px; contrast ratio meeting WCAG AA against
   email background; not nested inside a collapsed section.
   Reason: "prominent" is not binary-testable — these criteria are.

2. Clicking the unsubscribe link opens a confirmation page.

   Loading state: between link click and page render, display a
   neutral loading indicator. Do not show success state until
   server confirms processing.

   Success state: "You've been unsubscribed. We won't send you
   marketing emails."
   Not: "Preference updated successfully." (system language — rejected)

   Error states (confirmation page):
   - Expired token (> 30 days old): HTTP 410.
     Page: "This unsubscribe link has expired. Go to
     Settings > Notifications to manage your preferences."
     Link to Settings provided.
   - Already-used token: HTTP 410.
     Page: "You've already been unsubscribed. You won't receive
     further marketing emails."
   - Invalid signature: HTTP 403.
     Page: "This link is not valid. Go to
     Settings > Notifications to manage your preferences."
     Link to Settings provided.
   - Server failure: HTTP 503.
     Page: "Something went wrong. Please try again or go to
     Settings > Notifications to unsubscribe."
     Retry button + link to Settings provided.

   Recovery path: all error states provide a direct link to
   Settings > Notifications as the fallback action.

3. Unsubscribe is processed within 60 seconds of link click.
   User does not receive another marketing email after that window.

   Email dispatch service contract:
   - Timeout: 3000ms explicit (not service default). Source: ARCHITECTURE.md.
   - Failure response: HTTP 503, body:
     { error: "email_service_unavailable", retryAfter: 30 }
   - Retry strategy: retry once after 2 seconds on 429.
     If second attempt fails: surface server failure error state (AC2).
   - State write policy: DB preference write is held until email
     service confirms suppression update. No optimistic write.
     Source: EDGE-001.
   - Silent failure prevention: any email service failure must
     surface an error to the user. No silent swallowing of 4xx/5xx.

   Cache invalidation: preferences GET endpoint must return
   Cache-Control: no-store. On successful unsubscribe, any cached
   preference value is invalidated before confirmation page renders.
   Source: EDGE-003.

4. Unsubscribed user can resubscribe from Settings > Notifications
   > Marketing emails. Toggle is off. They can turn it on.

5. Unsubscribe link is authenticated via signed token.
   Token is user-specific and single-use.
   Token signing algorithm: HMAC-SHA256 with server secret.
   Token expiry: 30 days from email send date.
   Tokens stored in /db and invalidated on first use. Source: EDGE-002.
   Reason: prevents malicious actors from unsubscribing other users
   by guessing or reusing links.

   Token violation responses:
   - Reused token: HTTP 410. See error state in AC2.
   - Expired token: HTTP 410. See error state in AC2.
   - Invalid signature: HTTP 403. See error state in AC2.

### Settings Page

6. Settings > Notifications shows a Marketing emails toggle.
   Default state reflects current subscription status accurately.

7. Toggle change saves automatically (no Save button required).
   User sees inline confirmation: "Saved" (fades after 2 seconds).

8. Toggle shows a loading state during save (spinner, disabled).
   If save fails: toggle reverts to previous state.
   Error message: "Couldn't save your preference. Try again."

### States Required

Unsubscribe confirmation page:
- Loading state: neutral loading indicator between click and page render
- Success state: defined in AC2
- Error states (4): expired token, already-used token, invalid signature,
  server failure — all defined in AC2 with message content and recovery path
- Empty state: not applicable

Settings toggle:
- Default state: reflects live DB value (Cache-Control: no-store on GET)
- Loading state: spinner, toggle disabled — timeout 5000ms;
  if exceeded, resolve to error state
- Success state: inline "Saved" confirmation, fades after 2 seconds
- Error state: toggle reverts to previous state.
  Error message: "Couldn't save your preference. Try again."
- Empty state: not applicable (toggle always has a state)

### Non-Functional Requirements

Email dispatch service (all calls from this feature):
- Timeout: 3000ms explicit
- Failure response shape: HTTP 503, { error: "email_service_unavailable", retryAfter: 30 }
- Retry: once after 2 seconds on 429; surface error to user on second failure
- State write policy: no DB write before service confirmation

Cache:
- Preferences GET endpoint: Cache-Control: no-store
- Post-write: invalidate cached preference value before rendering updated state
  Source: EDGE-003

Token:
- Algorithm: HMAC-SHA256
- Expiry: 30 days
- Storage: /db, invalidated on first use
- Boundary behaviour: expired and reused tokens return HTTP 410;
  invalid signature returns HTTP 403

Concurrency:
- Last write wins for simultaneous toggle saves from the same user
- Final state must match the last confirmed DB write
- No silent merge or conflict — UI reflects confirmed state only

Operational SLO (critical path — paying customers apply):
- Toggle save: p95 response time < 500ms
- Unsubscribe confirmation page render: p95 < 2000ms
- Measurement: rolling 7-day window post-launch

Regression surface:
- Marketing email templates (new footer link — test all active templates)
- /preferences module (new — full integration test required)
- /notifications module (affected by suppression logic)
- Settings > Notifications page (existing route, new toggle behaviour)

---

## Success Metric

Primary: Marketing email support tickets < 28 per month
Baseline: 47 per month (3-month average)
Measurement window: 30 days post-launch
Owner: [PM name]

Counter-metric: 30-day retention of users who unsubscribe
vs matched cohort of 60-day non-openers who do not unsubscribe.
Rationale: marketing emails may be a re-engagement mechanism.
If unsubscribing accelerates churn, we need to know within 30 days.
This does not block launch — it must be instrumented before launch.

Secondary counter-metric: Re-subscribe rate within 7 days.
If > 15%: unsubscribe confirmation UX may be causing
accidental unsubscribes. Investigate before expanding the feature.

---

## Instrumentation (Required Before Launch)

Product analytics events:
- Unsubscribe link click event
- Unsubscribe confirmation page load (including which error state, if any)
- Settings toggle change (direction: on→off, off→on)
- Save success / save failure
- Re-subscribe event

Operational monitoring (required before launch):
- Email service error rate alert: threshold > 5% of suppression calls
  failing within any 5-minute window
- Toggle save error rate alert: threshold > 2% failure rate
- Latency alert: toggle save p95 > 1000ms (warning), > 2000ms (critical)

All product analytics events must be firing and verified in staging.
All operational alerts must be configured and tested before launch.
No launch without both confirmed.

---

## Rollout Plan

Not A/B testable for unsubscribe — all users must have access to
unsubscribe. Creating a control group without unsubscribe would be
a legal and trust violation.

Candidate for A/B test: prominent vs subtle unsubscribe link placement.
Both groups retain full ability to unsubscribe. Ethical.
This is optional for v1 — flagged for consideration.

Rollout: 100% on launch. Feature flag for rollback if needed.

---

## Open Decisions (Human Must Resolve)

1. Should the unsubscribe confirmation page offer a frequency
   option? ("Too many? Get one email per month instead.")
   This may reduce full unsubscribes while solving the frustration.
   Decision deferred — add to v2 consideration if re-subscribe
   rate is high.

2. Does the revenue / growth team know this is being built?
   Marketing email opt-outs will reduce marketing email open rates.
   Email team dashboard will show a drop — this is expected.
   Pre-communication required before launch.
   Owner: [PM name] to notify before sprint end.

---

## Out of Scope (with reasons)

- Product update emails: 8% of support tickets. Deferred to v2.
- Weekly digest: 6% of support tickets. Deferred to v2.
- In-app alerts: 4% of support tickets. Deferred to v2.
- Mobile push: 9% of support tickets. Deferred to v2.
- Notification frequency controls: different problem. Separate spec.
- Do not disturb scheduling: different problem. Separate spec.
- Per-device settings: different problem. Separate spec.

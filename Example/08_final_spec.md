# SPEC — Feature: Notification Preferences (v3 — post-persona review)

> This is the output after running the Amazon PM and Google PM personas.
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

1. Every marketing email contains a prominent unsubscribe link
   in the footer. Link text: "Unsubscribe from marketing emails".
   Not hidden. Not styled as body text.

2. Clicking the unsubscribe link opens a confirmation page.
   Page displays: "You’ve been unsubscribed. We won’t send you
   marketing emails."
   Not: "Preference updated successfully." (system language — rejected)

3. Unsubscribe is processed within 60 seconds of link click.
   User does not receive another marketing email after that window.

4. Unsubscribed user can resubscribe from Settings > Notifications
   > Marketing emails. Toggle is off. They can turn it on.

5. Unsubscribe link is authenticated via signed token.
   Token is user-specific and single-use.
   Reason: prevents malicious actors from unsubscribing other users
   by guessing or reusing links.

### Settings Page

6. Settings > Notifications shows a Marketing emails toggle.
   Default state reflects current subscription status accurately.

7. Toggle change saves automatically (no Save button required).
   User sees inline confirmation: "Saved" (fades after 2 seconds).

8. Toggle shows a loading state during save (spinner, disabled).
   If save fails: toggle reverts to previous state.
   Error message: "Couldn’t save your preference. Try again."

### States Required

For both the unsubscribe confirmation page and the settings toggle:
- Default state: defined above
- Loading state: defined above
- Success state: defined above
- Error state: defined above
- Empty state: not applicable (toggle always has a state)

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

- Unsubscribe link click event
- Unsubscribe confirmation page load
- Settings toggle change (direction: on→off, off→on)
- Save success / save failure
- Re-subscribe event

All events must be firing and verified in staging before launch.
No launch without confirmed instrumentation.

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

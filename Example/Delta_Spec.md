# DELTA.md — What the Pipeline Changed and Why

> **Example disclaimer:** Flowdesk is a fictional product created for this worked example.
> Any resemblance to real products is coincidental. "SendGrid" is used as a placeholder
> name for an email dispatch service. No real product or company is implied or endorsed.

> This file is the proof of the Sparring library's value.
> It maps every addition in the final spec back to:
> — which persona caught it
> — what the agent would have built without it
> — what class of problem it prevents
>
> **Pipeline mode used in this example: detailed.**
> Detailed mode produces a full structured review — every AC audited, every assumption logged.
> Summary mode (the default) produces verdict + top 2 blockers + conditional items.
> This example uses detailed mode so you can see every finding and trace it to its source.
> Run summary mode first. Switch to detailed when you need to understand a specific block.
>
> A competent, time-pressured PM wrote the input spec. It is not careless.
> It is realistic.

---

## Summary

| Version | ACs | Sections | Scope | Personas run |
|---|---|---|---|---|
| v1 (thin spec) | 6 | 3 | 5 notification types | — |
| v4 (post-full pipeline) | 17 | 12 | 1 notification type (fully contracted) | All five personas |

The final spec has fewer acceptance criteria but more integrity.
It solves a precisely defined problem instead of a vaguely described one.
Every change is traceable to a specific persona finding.
None were invented. None are bureaucratic overhead.

---

## Change Log — By Persona

---

### Amazon PM — 5 findings

**Finding 1: The customer was not defined**

v1 said: "Users have been complaining about too many notifications."

Amazon PM asked: which users? All users, or a specific segment?
A user who signed up yesterday and a user who has not opened a
marketing email in 60 days have completely different problems.
Building one feature for both solves neither well.

What was added: Customer definition — "existing user who receives
marketing emails and has not opened one in 60 days."

What the agent would have built without it: a preference page for
all users, with equal prominence given to all notification types.
The user most likely to complain would have had to navigate to Settings,
find the right toggle, and remember to turn it off. The easier path
— the unsubscribe link in the email itself — would not exist.

Class of problem prevented: Feature built for the wrong user.
The loudest complaint solved last.

---

**Finding 2: The problem was misdiagnosed**

v1 said: "too many notifications" — implying volume was the issue.

Amazon PM asked: what does the support ticket data actually say?
Not what the PM assumes — what do the tickets say?

The PM checked. 73% of tickets were specifically about marketing emails.
Users did not say "too many notifications."
They said "I don't know how I got on this list" and
"I can't find where to unsubscribe."

This is a discoverability problem, not a volume problem.
The solution is a prominent unsubscribe path, not a preference centre.

What was added: Support ticket data (73%, 47/month baseline),
problem reframe from volume to discoverability.

What the agent would have built without it: 5 toggles for 5
notification types. The user who wants to unsubscribe from marketing
emails still has to find Settings > Notifications, understand
the toggle model, and make 5 individual decisions.
The problem — "I can't find where to unsubscribe" — persists.

Class of problem prevented: Right feature, wrong solution.
Spec addressed the symptom, not the cause.

---

**Finding 3: Scope was not justified**

v1 scoped 5 notification types without stating why all 5 were needed.

Amazon PM applied the frugality check:
if 73% of complaints are about one notification type,
building 5 toggles solves 27% of the problem at 5x the cost.

What was added: Scope rationale and deferral of 4 notification types
with explicit ticket percentages for each (product updates 8%,
weekly digest 6%, in-app alerts 4%, mobile push 9%).

What the agent would have built without it: 5 toggles. 2 sprints.
The feature launches. Support tickets drop by roughly 27%
(the non-marketing complaints that are now addressed).
The PM reports partial success. The 73% remains unsolved.

Class of problem prevented: Scope waste. Building the less important
80% of a feature before validating the critical 20%.

---

**Finding 4: No success metric defined**

v1 had no measurement definition of any kind.

Amazon PM asked: how will you know this worked?
What is the metric, what is the baseline, what is the target,
and who owns measuring it?

Without a baseline, any result is uninterpretable.
A drop from 47 to 40 tickets looks like success.
A drop from 47 to 40 with a new unsubscribe feature that 3% of
users found looks like a problem worth investigating.
You cannot tell the difference without the baseline.

What was added: Primary metric (support tickets < 28/month),
baseline (47/month), measurement window (30 days), named owner.

What the agent would have built without it: A feature with no
measurement. It launches. Support ticket volume either goes down
or it doesn't. Nobody can prove the feature caused the change.
The PM cannot make the case for v2 investment.

Class of problem prevented: Unmeasurable feature.
Success and failure look identical after launch.

---

**Finding 5: Legal implication not flagged**

Marketing email unsubscribe has CAN-SPAM and GDPR implications.
v1 did not mention legal review.

Amazon PM classified this as an irreversible decision:
once you build a public unsubscribe mechanism, users will rely on it.
Changing its behaviour later has legal and trust implications.
Irreversible decisions require explicit sign-off.

What was added: Legal review section, confirmed compliance
requirements (honoured within 10 business days, we process instantly),
named legal owner.

What the agent would have built without it: An unsubscribe mechanism
with no legal review. In regulated markets (EU, US), this is
a compliance gap. Not a theoretical risk — a real one.

Class of problem prevented: Compliance failure discovered after launch.
The most expensive place to find a legal gap.

---

### Google PM — 4 findings

**Finding 6: Counter-metric not defined**

Google PM asked: what is the metric you must NOT sacrifice
while improving support ticket volume?

Marketing emails, even unwanted ones, are a re-engagement mechanism.
A user who unsubscribes has one fewer touchpoint with the product.
Two competing hypotheses exist — unsubscribing increases trust
(better retention) or unsubscribing removes re-engagement
(worse retention). Both are plausible. Neither is proven.

What was added: Counter-metric — 30-day retention of users who
unsubscribe vs matched cohort of 60-day non-openers who do not.
Also: re-subscribe rate within 7 days as a secondary counter.

What the agent would have built without it: A feature optimised
for one metric (support tickets) with no visibility into its
effect on retention. The feature could be simultaneously
reducing complaints and accelerating churn. Invisible without
the counter-metric.

Class of problem prevented: Metric optimisation that damages a
more important metric. The dashboard looks green. The business
gets worse.

---

**Finding 7: Instrumentation not specified**

Google PM: a feature that launches without instrumentation
produces data that cannot be acted on.

Funnels defined after launch are retrospective and incomplete.
Instrumentation defined before launch is prospective and precise.

What was added: Instrumentation section with 5 required events,
confirmation requirement in staging before launch.

What the agent would have built without it: A feature with
no tracking. The PM checks the metric at 30 days.
Support tickets dropped from 47 to 38. Is that the feature?
Or did a marketing campaign pause at the same time?
With instrumentation: you can trace the causal path.
Without it: correlation only.

Class of problem prevented: Unmeasurable feature.
Launch data that cannot answer the causal question.

---

**Finding 8: Rollout plan not defined**

Google PM asked: can this be A/B tested?
If not, why not, and what is the alternative?

For unsubscribe: A/B testing is not ethical. Explained in spec.
But link placement prominence IS testable. Flagged.

What was added: Explicit A/B test justification and alternative
(feature flag for rollback, optional prominence A/B test).

What the agent would have built without it: A 100% rollout
with no rollback mechanism. If the unsubscribe confirmation UX
causes accidental unsubscribes, you discover this from
customer complaints — not from a metric. Rollback requires
a new deployment, not a flag toggle.

Class of problem prevented: No rollback path on a legal feature.
An accidental unsubscribe on a compliance-critical mechanism
is a trust and legal risk.

---

**Finding 9: Cross-team impact not communicated**

Google PM asked: does the email team know this is being built?

Marketing email opt-outs reduce open rates and CTR on the
email dashboard. The email team will see a drop and investigate.
If they do not know the feature launched, they will treat it
as a deliverability problem and take incorrect action.

What was added: Open decision requiring PM to notify email/growth
team before launch. Named owner.

What the agent would have built without it: A feature that
causes a false alarm in another team's dashboard.
The email team pauses campaigns or contacts the ESP to investigate
a deliverability issue that is not a deliverability issue.
Invisible coordination failure.

Class of problem prevented: Cross-team alert fatigue and
incorrect remediation action triggered by expected metric movement.

---

## What the Agent Would Have Built From v1

```
From the original 5 acceptance criteria:

✓ A settings page in the correct location
✓ Five toggles, one per notification type
✓ Settings that persist after refresh
✓ A confirmation message after saving
✓ Immediate effect on toggle change
✗ No prominent unsubscribe link in email footer
  (the place 73% of complaining users look first)
✗ No problem statement or customer definition
  (agent has no basis to flag scope drift)
✗ No success metric or baseline
  (feature launches, nobody can prove it worked)
✗ No legal review of email unsubscribe compliance
  (CAN-SPAM and GDPR compliance gap)
✗ No counter-metric on retention
  (feature could reduce complaints and increase churn simultaneously)
✗ No instrumentation
  (causal connection between feature and metric is unprovable)
✗ No rollback plan
  (accidental unsubscribes require a new deployment to fix)
✗ No cross-team notification
  (email team triggers false deliverability investigation)
```

The agent would have shipped a working feature.
Five toggles. Correctly implemented. Tests passing.

The feature would have solved roughly 27% of the problem
at full 5-feature build cost.
The 73% would have remained.
The PM would have had no data to prove what happened.
A legal gap would have been live in production.
Another team would have spent time investigating a non-problem.

None of this would have been visible in code review.
All of it was visible in the spec review.

---

## What Each Persona Uniquely Caught

| Persona | Unique contribution | Class of problem prevented |
|---|---|---|
| Amazon PM | Wrong customer, misdiagnosed problem, unjustified scope, missing metric, legal gap | Built wrong thing, unmeasurable, compliance risk |
| Google PM | Missing counter-metric, no instrumentation, no rollback, cross-team impact | Metric gaming, causal blindness, coordination failure |

No persona duplicated the other's primary findings.
Both were required. Neither alone was sufficient.

---

## The Uncomfortable Implication

The PM who wrote v1 is not a bad PM.
They had support ticket data. They identified a real problem.
They scoped a reasonable solution. They wrote clear acceptance criteria.

They missed the customer definition, the root cause diagnosis,
the legal implication, the success metric, the counter-metric,
the instrumentation plan, and the cross-team coordination requirement.

Not because they are inadequate.
Because no single human holds all of these lenses simultaneously
while also managing a sprint, attending stakeholder meetings,
and writing the next three specs in the backlog.

That is not a skills problem.
It is a cognitive load problem.
And it has a structural solution.

---

### Google Architect — 6 findings

**Finding 10: SendGrid timeout not cited in spec**

The spec implied a SendGrid suppression call in AC3 but contained no timeout
value. A new agent reading the spec alone had no timeout to implement and would
fall back to the Node.js HTTP default (~120 seconds).

What was added: Email dispatch service contract in AC3 — explicit 3000ms timeout
cited from ARCHITECTURE.md.

What the agent would have built without it: A suppression call with no timeout.
Under any SendGrid degradation, users would wait up to 2 minutes for a page
that either renders or crashes with an unhandled promise rejection.

Class of problem prevented: EDGE-001 repeated. Same gap, different feature.

---

**Finding 11: SendGrid failure response shape absent**

AC3 defined a 60-second user-facing SLA but gave no failure response shape for
what the system returns when SendGrid fails. The agent would have returned HTTP 500
with no body.

What was added: Explicit failure shape — HTTP 503, { error: "email_service_unavailable",
retryAfter: 30 } — plus retry strategy and silent failure prevention.

What the agent would have built without it: A 500 error with no message and no
recovery path. User sees a blank error page with no instruction.

Class of problem prevented: Unrecoverable user state on external service failure.

---

**Finding 12: Cache invalidation not specified (EDGE-003 rule violated)**

AC6 required the toggle to reflect current subscription status accurately. AC7
wrote new state. The spec contained no Cache-Control instruction. Per EDGE-003,
any feature that writes then reads state must specify cache invalidation.

What was added: Cache-Control: no-store on preferences GET endpoint. Post-write
cache invalidation before rendering updated state.

What the agent would have built without it: A global API cache setting would have
applied to the preferences GET. Users would see stale state for up to 5 minutes
after saving — the exact pattern of EDGE-003.

Class of problem prevented: Stale UI state post-save. Support ticket: "I changed
my setting but it reverted."

---

**Finding 13: Token signing algorithm unspecified — irreversible decision**

AC5 said "signed token" without specifying the algorithm. Choosing an algorithm
after tokens have been issued to users is irreversible — all outstanding tokens
are invalidated on algorithm change.

What was added: HMAC-SHA256 specified explicitly in AC5.

What the agent would have built without it: An algorithm chosen by default
(often HS256 or a library default). Undocumented. Difficult to rotate.

Class of problem prevented: Undocumented irreversible architectural decision.

---

**Finding 14: No retry strategy in spec**

The spec was silent on what happens when SendGrid returns 429. The agent would
have either swallowed the error or crashed.

What was added: Retry once after 2 seconds on 429. Surface error to user if
second attempt fails.

Class of problem prevented: Silent failure on rate limiting. User unsubscribes,
gets confirmation, still receives email.

---

**Finding 15: No degraded mode for SendGrid unavailability**

If SendGrid is completely down when a user clicks unsubscribe, the spec gave
no instruction. The agent would have returned an unhandled error.

What was added: Server failure error state in AC2 with message content and
recovery path (link to Settings).

Class of problem prevented: Dead end on critical user action during service outage.

---

### UI Designer — 4 findings

**Finding 16: Unsubscribe confirmation page error states missing**

The "States Required" section claimed error state was "defined above" for the
confirmation page. It was not. AC2 only defined the success state. Three failure
modes had no message content: expired token, already-used token, server failure.

What was added: Four error states in AC2, each with message text and a recovery
path linking to Settings > Notifications.

What the agent would have built without it: A generic error page with no message
and no path forward. Users with expired links would have no way to unsubscribe.

Class of problem prevented: Dead-end error page on a legally required user action.

---

**Finding 17: No recovery path for token failure modes**

AC5 stated single-use but gave no UX for what happens on violation. A user
clicking an expired or already-used link would hit an undefined state.

What was added: Recovery path in every error state — direct link to
Settings > Notifications as the fallback unsubscribe action.

Class of problem prevented: User unable to complete a legally required action.

---

**Finding 18: No loading state for unsubscribe link processing**

AC8 specified a loading state for the toggle save. No equivalent existed for
the period between unsubscribe link click and confirmation page render. The server
must validate the token and call SendGrid — this takes variable time.

What was added: Neutral loading indicator specified in AC2 between click and render.

What the agent would have built without it: Immediate page transition with no
feedback. On slow responses, user clicks again (double-submission) or navigates away.

Class of problem prevented: Double-submission on slow external service call.

---

**Finding 19: Toggle loading state has no timeout**

AC8 specified spinner during save but no timeout. If the save call never
completes, the spinner never resolves.

What was added: 5000ms timeout on toggle loading state; resolves to error state
if exceeded.

Class of problem prevented: Infinite spinner. User cannot retry a failed action.

---

### QA Functional — 6 findings

**Finding 20: AC1 "prominent" is not binary-testable**

"Prominent unsubscribe link" cannot produce a pass/fail verdict. Prominence is
subjective. A QA engineer cannot write a test for it without criteria.

What was added: Testable criteria replacing "prominent" — minimum font size 12px,
WCAG AA contrast ratio, not nested inside a collapsed section.

What the agent would have built without it: An unsubscribe link that technically
exists but may be invisible in practice — which was the original user complaint.

Class of problem prevented: AC passes QA, user complaint persists.

---

**Finding 21: No negative path for SendGrid failure during unsubscribe flow**

AC8 covered toggle save failure. No equivalent negative path existed for the
unsubscribe confirmation page when SendGrid fails during processing.

What was added: Server failure error state in AC2 (surfaced from Architect and
UI Designer findings). Negative path is now specified.

Class of problem prevented: Unhandled failure on the primary user journey.

---

**Finding 22: Token reuse response not specified**

AC5 said "single-use" — the rule was stated but the violation response was absent.
The agent needs: what HTTP status, what page state, what message?

What was added: HTTP 410 for reused token, "already-used" error state in AC2.

What the agent would have built without it: An implementation of single-use with
an invented violation response — likely a generic 403 with no user-facing message.

Class of problem prevented: Confusing or absent user feedback on link reuse.

---

**Finding 23: Token expiry boundary not specified**

AC5 stated single-use but no expiry window. ARCHITECTURE.md implied 30 days
but the spec was silent. A user clicking a 31-day-old link would hit an
undefined state.

What was added: 30-day expiry in AC5, expired token error state in AC2.

Class of problem prevented: Undefined behaviour at a predictable boundary.

---

**Finding 24: Regression surface not named**

The spec did not list what existing functionality this feature affects.
The marketing email template, /preferences module, /notifications module, and
Settings route are all touched. None were named.

What was added: Regression surface section in Non-Functional Requirements.

Class of problem prevented: Regression bugs in untested adjacent features.

---

**Finding 25: System state after unsubscribe flow failure not explicit**

EDGE-001 rule says DB write is held until email service confirms. The spec did
not state this for the unsubscribe flow explicitly. An agent reading the spec
could write the DB preference before calling SendGrid.

What was added: State write policy in AC3 email dispatch service contract —
explicit "DB preference write held until email service confirms."

Class of problem prevented: EDGE-001 repeated — DB says unsubscribed, SendGrid
still active, user receives email.

---

### QA NFQ — 7 findings

**Findings 26–28: Timeout, failure shape, silent failure (same gaps as Architect)**

QA NFQ independently flagged the same SendGrid contract gaps. This is the
pipeline working correctly — Architect checks that contracts exist, NFQ
runs the pre-flight check that confirms their values are in the spec.
Both must pass independently.

What was added: Already documented in Architect findings 10–12.
NFQ pre-flight check now passes.

---

**Finding 29: Signed token expiry boundary not defined**

AC5 stated single-use but no expiry. NFQ non-negotiable: token boundary
behaviour must be defined on any feature with signed tokens.

What was added: 30-day expiry in AC5 (same as QA Functional finding 23,
caught independently by NFQ from a resilience angle rather than a functional one).

Class of problem prevented: Unpredictable behaviour at a scheduled boundary.

---

**Finding 30: No operational SLO for toggle save**

AC3 contained a user-facing 60-second SLA for unsubscribe processing.
No operational SLO existed for the Settings toggle save path. Flowdesk has
paying customers — SLO block applies on critical paths.

What was added: p95 < 500ms for toggle save, p95 < 2000ms for confirmation
page render. Rolling 7-day measurement window.

Class of problem prevented: No measurable reliability target on a feature
with paying customers. Success and degradation are indistinguishable.

---

**Finding 31: No monitoring or alerting plan**

The instrumentation section contained product analytics events only.
No operational monitoring existed for email service error rate or
toggle save latency.

What was added: Three operational alerts — email service error rate threshold,
toggle save error rate threshold, latency warning and critical thresholds.

Class of problem prevented: Silent production degradation. Feature works in
testing, fails for 8% of users in production, discovered via support tickets.

---

**Finding 32: Concurrency not addressed**

A user with Settings open in two browser tabs simultaneously could submit
conflicting toggle states. Spec was silent on which write wins.

What was added: Last-write-wins policy, final state must match confirmed DB write,
no silent merge.

Class of problem prevented: Inconsistent preference state between tabs or sessions.

---

## What Each Persona Uniquely Caught

| Persona | Primary contribution | Class of problem prevented |
|---|---|---|
| Amazon PM | Wrong customer, misdiagnosed problem, unjustified scope, missing metric, legal gap | Built wrong thing, unmeasurable, compliance risk |
| Google PM | Missing counter-metric, no instrumentation, no rollback, cross-team impact | Metric gaming, causal blindness, coordination failure |
| Architect | External service contracts, cache invalidation, irreversible token decision | EDGE-001/003 patterns repeated, undocumented architecture |
| UI Designer | Error states with message content, recovery paths, loading timeouts | Dead-end error pages, infinite spinner, double-submission |
| QA Functional | Binary ACs, negative paths, token boundary, regression surface | Untestable spec, unhandled failure paths, regression gaps |
| QA NFQ | Pre-flight contract check, SLO, monitoring, concurrency | Silent degradation, no reliability baseline, race conditions |

No persona duplicated another's primary contribution.
All six were required. No single persona alone was sufficient.

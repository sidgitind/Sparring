# DELTA.md — What the Pipeline Changed and Why

> This file is the proof of the Sparring library's value.
> It maps every addition in the final spec back to:
> — which persona caught it
> — what the agent would have built without it
> — what class of problem it prevents
>
> Note: this worked example runs only the PM personas.
> The gaps caught here are PM-level gaps — thinking gaps, not technical ones.
> A mid-level PM wrote the input spec. It is not careless.
> It is what a competent, time-pressured PM produces.

---

## Summary

| Version | ACs | Sections | Scope |
|---|---|---|---|
| v1 (thin spec) | 5 | 3 | 5 notification types |
| v3 (post-persona) | 8 | 9 | 1 notification type (scoped down) |

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

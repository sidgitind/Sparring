# DELTA.md — What the Pipeline Changed and Why

> **Example disclaimer:** Flowdesk is a fictional product created for this worked example.
> Any resemblance to real products is coincidental. No real product or company is
> implied or endorsed.

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
| v1 (thin spec) | 4 | 3 | All tickets, all engineers | — |
| v4 (post-full pipeline) | 15 | 11 | All tickets, risk-tiered by thread complexity | All five personas |

The final spec has more acceptance criteria and a fundamentally different
trust model. It does not solve a different problem — it solves the same
problem honestly, including the part where the AI is wrong sometimes.
Every change is traceable to a specific persona finding.
None were invented. None are bureaucratic overhead.

---

## Change Log — By Persona

---

### Amazon PM — 4 findings

**Finding 1: The customer was not defined**

v1 said: "Support engineers have flagged context-switching as a major time sink."

Amazon PM asked: which engineer, in which moment? An engineer triaging
a routine password-reset ticket and an engineer working a complex
multi-day escalation have completely different relationships to a
summary. The first can tolerate an imperfect summary — the cost of
being wrong is low. The second cannot — the cost of being wrong is
a customer escalation and a post-mortem.

What was added: Customer definition — "engineer making a consequential
decision on a ticket with non-trivial thread complexity," distinguished
explicitly from the low-stakes routine case.

What the agent would have built without it: a single summary experience
applied uniformly regardless of stakes. The routine case (where it works
fine) and the escalation case (where an omission is expensive) would
receive identical treatment. The feature would optimize for the common
case and fail silently on the consequential one.

Class of problem prevented: Feature built for the average case.
The highest-stakes user experience left unexamined.

---

**Finding 2: Failure was not defined**

v1 defined a 90% accuracy threshold against a test set. It did not
define what failure looks like in production, for a real engineer,
on a real decision.

Amazon PM asked: 90% accurate against what measure? A summary that
correctly compresses 90% of a thread's content but omits the one
status change that matters is not a 10%-failure. It is a complete
failure for that specific decision. Aggregate accuracy and
decision-relevant completeness are different things, and the spec
only measured the first.

What was added: A second, decision-relevant standard — defined in
Finding 14 (QA Functional) as structural completeness heuristics —
sitting alongside the aggregate accuracy metric, not replacing it.

What the agent would have built without it: A summary feature that
passes its own accuracy test while still failing the user in exactly
the moments that matter most. The metric would say green. The outcome
would not be.

Class of problem prevented: Metric that measures the wrong kind of correctness.

---

**Finding 3: No accountability model**

v1 was silent on what happens when an engineer acts on a summary
that turns out to be wrong.

Amazon PM classified this as an irreversible-decision risk: once
engineers begin trusting summaries as a substitute for reading
threads, that behavior is hard to reverse — and the first bad
outcome will either erode trust permanently or get rationalized
away, both of which are worse than addressing it upfront.

What was added: An explicit accountability section — engineers remain
responsible for actions taken, but the product is responsible for
surfacing when a summary is operating in higher-risk territory
(see Finding 9, Architect, and Findings 12–13, UI Designer, for the
concrete mechanisms that make this true rather than aspirational).
The spec states this trade-off explicitly instead of leaving it implicit.

What the agent would have built without it: A feature where
responsibility is undefined until the first escalation forces the
question in a post-mortem — the most expensive possible place to
discover the answer.

Class of problem prevented: Undefined accountability discovered
reactively, under pressure, after damage is done.

---

**Finding 4: No scope rationale for uniform rollout**

v1 applied the same summary behavior to every ticket without
justifying why a one-size-fits-all rollout was the right scope.

Amazon PM applied the frugality and risk-tiering check: if the
cost of a wrong summary scales with thread complexity, a uniform
rollout spends the same engineering and trust budget on low-risk
and high-risk tickets alike.

What was added: Scope rationale — tiered behavior by thread
complexity (see Final Spec), starting narrower than v1's "every
ticket" framing.

What the agent would have built without it: A summary panel
identical in every ticket regardless of risk, with no differentiated
treatment for the cases most likely to cause harm.

Class of problem prevented: Uniform treatment of non-uniform risk.

---

### Google PM — 3 findings

**Finding 5: Counter-metric not defined**

Google PM asked: what is the metric you must NOT sacrifice while
improving time-to-first-action?

Time saved per ticket is the obvious win metric. But decision
quality — whether the engineer's action was correct given the full
thread context — is a different thing entirely, and it is possible
to improve the first while quietly damaging the second.

What was added: Counter-metric — rate of post-resolution escalations
or reopens on tickets where a summary was viewed, compared against a
matched cohort where the engineer read the full thread. Tracked
from week one, not introduced retroactively after a problem surfaces.

What the agent would have built without it: A feature optimized for
speed with no visibility into whether the speed was coming at the
cost of correct decisions. Week-one data would have looked identical
to the actual incident's lead-up.

Class of problem prevented: Metric optimization that damages a more
important, unmeasured metric.

---

**Finding 6: Instrumentation not specified**

v1 specified a thumbs up / thumbs down toggle but no other
instrumentation.

Google PM: thumbs feedback measures perceived helpfulness in the
moment, not whether the summary was actually complete, and not
whether the engineer read the full thread anyway. None of these
are visible from a binary toggle.

What was added: Instrumentation section — summary view event, full-thread
expand event (whether the engineer opened the full thread after
viewing the summary), action-taken event linked to which version of
the summary was visible at the time.

What the agent would have built without it: A feature with a single
soft feedback signal and no way to detect the actual failure pattern —
engineers quietly reading both the summary and the full thread,
doubling their work while appearing to use the feature successfully.

Class of problem prevented: Invisible adoption decay. The exact
pattern that occurred in week three of the real scenario — detectable
only with the right instrumentation in place from launch.

---

**Finding 7: Rollout and rollback plan not defined**

Google PM asked: can trust in this feature be monitored in a way
that allows rollback before a post-mortem forces the question?

What was added: Phased rollout by ticket complexity tier (low-risk
tickets first), with a defined trust signal — full-thread expand
rate — as a rollback trigger if it climbs past a threshold,
indicating engineers no longer trust the summary on that tier.

What the agent would have built without it: A 100% rollout with
no early warning signal. Trust erosion would be discovered only
after an incident, not from a leading indicator.

Class of problem prevented: No rollback path on a feature whose
failure mode is silent until it isn't.

*See Cross-References section: this finding's phasing rationale is
checked against Architect Finding 9 below.*

---

### Google Architect — 3 findings

**Finding 8: Summarization service contract undefined**

The spec implied a call to a summarization service (internal model
or third-party LLM API) but specified no timeout, no failure shape,
and no behavior when generation fails or exceeds latency budget.

What was added: Explicit service contract — 4000ms timeout, failure
response shape (ticket renders with thread visible, no summary panel,
no error blocking the engineer), retry-once policy on transient failures.

What the agent would have built without it: A summary call with no
timeout, defaulting to a language or library default. Under model
degradation or API slowness, the ticket page would hang or render
a blank panel with no explanation.

Class of problem prevented: Blocked or broken ticket page on
summarization service degradation — on a tool support engineers
depend on continuously.

---

**Finding 9: Structural risk heuristics — where they run and what they cost**

Finding 3 (Amazon PM) and Finding 14 (QA Functional) require
structural completeness heuristics — message count, time span,
detected status change — to be computed and surfaced to the engineer.
The spec did not say where this computation happens or what it costs.

What was added: Heuristics computed synchronously at summary
generation time (cheap — counting messages and checking for status
keywords adds negligible latency compared to the generation call
itself), cached alongside the summary, invalidated on regeneration.

What the agent would have built without it: An ambiguous instruction
that an agent might implement as a separate async job, introducing a
race condition between the summary rendering and the risk flag
appearing — exactly the kind of inconsistent state EDGE-003 already
warns against in this project's architecture context.

Class of problem prevented: Race condition between summary and
its own risk indicator.

*See Cross-References section: this finding's "cheap and synchronous"
conclusion is what makes Google PM Finding 7's phasing a trust decision
rather than a performance one — the two findings converge on the same
rollout shape for different reasons.*

---

**Finding 10: Regeneration trigger creates a stale-summary window**

AC4 stated the summary regenerates if new messages are added after
the last generation. It did not specify what the engineer sees
between the new message arriving and regeneration completing.

What was added: Explicit stale-state handling — if a new message
arrives, the existing summary is immediately marked "may be
outdated — N new message(s) since this summary" rather than silently
left as-is or blanked.

What the agent would have built without it: An engineer reading a
summary that looks current but reflects a thread state that no
longer exists, with no signal that anything has changed.

Class of problem prevented: Silently stale summary presented as current.

---

### UI Designer — 3 findings

**Finding 11: No states defined beyond the success case**

The spec described summary generation and the thumbs feedback toggle.
It did not define what the engineer sees during generation, on
generation failure, or on the stale-summary case introduced in
Finding 10.

What was added: Four states — loading (neutral indicator while
generating), success (summary with risk indicator per Finding 9),
stale (banner per Finding 10), and failure (panel collapses cleanly,
full thread remains the primary view, no error blocking the page —
consistent with the failure shape Architect defined in Finding 8).

What the agent would have built without it: Only the success case
rendered intentionally. Every other state left to whatever the
agent inferred — likely a blank panel or an unhandled loading
spinner with no resolution.

Class of problem prevented: Unhandled states on a panel engineers
rely on dozens of times a day.

---

**Finding 12: Risk indicator has no defined visual treatment**

Finding 9 introduced a structural risk signal. The spec did not
say how it appears, how prominent it is, or whether it could be
missed.

What was added: Explicit visual spec — risk indicator appears as a
labeled banner above the summary text, not a subtle icon or color
change alone, with the specific heuristic that triggered it named
in the banner text (e.g., "Long thread, status changed — review
full thread before acting").

What the agent would have built without it: A small icon or badge
easily missed in exactly the high-stakes moment it exists to flag —
repeating the original incident's failure mode inside the fix meant
to prevent it.

Class of problem prevented: A safety signal that is technically
present but practically invisible.

*See Cross-References section: this finding and Finding 13 are what
give Amazon PM Finding 3's accountability claim concrete mechanism.*

---

**Finding 13: No path from summary to source**

Even with a risk indicator, the spec gave no way for the engineer
to quickly verify the flagged content against the original thread.

What was added: A "jump to relevant message" link embedded in the
risk banner, taking the engineer directly to the thread location
the heuristic flagged (e.g., the message where a status change was
detected) rather than requiring a full re-read.

What the agent would have built without it: A warning with no
efficient way to act on it — the engineer would have to read the
entire thread anyway, defeating the purpose of the summary feature
in exactly the cases that matter most.

Class of problem prevented: A risk flag that creates anxiety without
giving the engineer an efficient way to resolve it.

---

### QA Functional — 3 findings

**Finding 14: "Accuracy" was not binary-testable in the way that matters**

AC2 specified 90% accuracy against a human-reviewed test set. This is
testable in aggregate but says nothing about per-ticket completeness —
exactly the gap that caused the original incident.

QA Functional asked: what is "summary confidence" or "completeness"
actually measuring, and can it be tested per-ticket, not just in
aggregate? A confidence score implies the model assesses its own
completeness — it does not.

What was added: Structural completeness heuristics — message count,
time span covered, detected status/resolution change mid-thread,
participant count — as testable, verifiable signals about the input,
explicitly not framed as the AI assessing its own correctness.
(Builds directly on Amazon PM Finding 3's identification of the need;
see Cross-References section.)

What the agent would have built without it: Either no per-ticket
signal at all, or — worse — an invented "confidence score" that
sounds rigorous but measures token-level generation certainty,
which does not correlate with whether something important was
omitted.

Class of problem prevented: A false-confidence feature that looks
like a safety mechanism and is not one.

---

**Finding 15: No negative path for summarization service failure**

No negative path existed for what the engineer experiences when
the summarization service fails entirely (covered architecturally
in Finding 8, but not specified as a testable user-facing path).

What was added: Failure state in AC2 (see UI Designer Finding 11) —
testable assertion: ticket page renders fully functional with thread
visible and no summary panel, no error dialog, no blocked interaction.

Class of problem prevented: Unhandled failure on the primary page
support engineers work from continuously.

---

**Finding 16: Regression surface not named**

The spec did not list what existing functionality this feature
touches. The ticket detail view, the thread rendering component, and
any existing "mark as read" or ticket-state logic are all adjacent.

What was added: Regression surface section — ticket detail view
(new panel insertion point), thread component (regeneration trigger
on new message), existing ticket-state logic (must not be affected
by summary regeneration).

Class of problem prevented: Regression bugs in adjacent, untested
functionality.

---

### QA NFQ — 4 findings

**Finding 17: Summarization service contract — pre-flight check (cross-reference)**

QA NFQ independently flagged the same summarization service contract
gap as the Architect persona. This is the pipeline working correctly —
Architect checks that a contract exists, NFQ runs the pre-flight check
confirming the values are actually in the spec. Both must pass
independently.

What was added: Already documented in Architect Finding 8.
NFQ pre-flight check now passes.

---

**Finding 18: No operational SLO for summary generation**

AC1 stated summaries "generate automatically when a ticket is
opened" with no latency target. Flowdesk has paying customers whose
support teams depend on ticket load time.

What was added: p95 < 1500ms for summary generation from ticket
open to render (with the 4000ms service timeout from Finding 8 as
the outer bound before failure state triggers). Rolling 7-day
measurement window.

Class of problem prevented: No measurable reliability target on a
feature load-bearing to support team throughput.

---

**Finding 19: No monitoring or alerting plan**

The instrumentation section (Finding 6, Google PM) contained product
analytics only. No operational monitoring existed for summarization
service error rate or generation latency.

What was added: Two operational alerts — summarization service error
rate threshold (> 5% of calls failing in any 5-minute window) and
generation latency threshold (p95 > 1200ms warning, > 1500ms critical).

Class of problem prevented: Silent production degradation — summaries
quietly failing or slowing for a subset of tickets, discovered only
when engineers complain rather than from a monitored signal.

---

**Finding 20: Concurrent regeneration not addressed**

Two new messages arriving in close succession could each trigger a
regeneration request (AC4), creating a race condition over which
summary version is stored and displayed.

What was added: Regeneration debounced — a new message resets a
short window; only one regeneration fires per window. Last completed
regeneration wins. No partial or interleaved summary state.

Class of problem prevented: Inconsistent or flickering summary state
on active, fast-moving threads.

---

## Cross-References — Where Findings Touch the Same Mechanism

Several findings across different personas operate on the same
underlying mechanism. None of these are contradictions. They are
flagged explicitly here so the relationship is visible rather than
implicit — the reader should not have to notice on their own that
two personas are talking about the same thing from different angles.

---

**Structural completeness heuristics — Findings 3, 9, 14**

Three personas touch the same four-item heuristic list (message
count, time span, detected status change, participant count) from
three different angles:

- Amazon PM (Finding 3) establishes *why* a signal is needed at all —
  the accountability gap.
- QA Functional (Finding 14) defines *what* the signal actually
  measures, and explicitly rejects the alternative framing
  ("confidence score") as unverifiable.
- Architect (Finding 9) defines *where* the signal is computed and
  what it costs.

These build on each other in the order the pipeline ran. QA Functional's
definition would not exist without Amazon PM's finding establishing
the need. Architect's placement decision would not exist without QA
Functional's definition of what needs computing. This is the pipeline
working as designed — one mechanism, three lenses, no duplication of
each other's actual contribution.

---

**Shared accountability between engineer and product — Findings 3, 12, 13**

Amazon PM's Finding 3 states that engineers remain accountable for
actions taken, but that the product shares responsibility for
surfacing risk. That claim is abstract until UI Designer's Finding 12
(visual prominence of the risk indicator) and Finding 13 (jump-to-source
link) give it concrete mechanism. Finding 3 references Finding 9 in
its original write-up but should be read as also resting on Findings
12 and 13 — without the indicator being visible and actionable, the
"shared responsibility" claim in Finding 3 is aspirational rather than
real. Flagged here rather than rewriting Finding 3, since the
relationship is sequential (PM defines the principle, UI Designer
makes it true) rather than duplicative.

---

**Rollout phasing — Finding 7, considered against Finding 9**

Google PM's Finding 7 phases rollout by ticket complexity tier,
framed as a trust concern — start where the cost of being wrong is
lowest. Architect's Finding 9 establishes that the heuristics
computation is cheap and synchronous, meaning there is no
*performance* reason to phase the rollout.

These two findings agree on the rollout shape but for different
reasons, and that is worth naming rather than leaving implicit. In
this run, performance and trust both point toward the same phased
approach, so there is no visible tension. Worth stating plainly: had
the heuristics been expensive to compute, Architect might have
recommended phasing for a different reason than Google PM, or
recommended an async/cached approach that changes the UI Designer's
loading-state design (Finding 11). The Delta should make clear when
two personas converge on the same answer for different reasons,
because that convergence is not guaranteed in every spec — it happened
to hold here.

---

**Service failure handling — Findings 8, 11, 15 — confirmed non-contradictory**

Architect (Finding 8) defines the service contract and failure shape.
UI Designer (Finding 11) defines what the engineer sees when that
failure occurs. QA Functional (Finding 15) defines the testable
negative-path assertion for the same failure. All three describe the
same event from contract, experience, and test perspectives
respectively, and were checked against each other for consistency:
Architect's failure response triggers UI Designer's failure state,
which is the exact condition QA Functional's negative path asserts
against. No gap, no contradiction, no persona working from a stale
version of another's finding.

---

## What the Agent Would Have Built From v1

```
From the original 4 acceptance criteria:

✓ A summary panel that generates on ticket open
✓ An accuracy threshold measured against a test set
✓ A thumbs up / thumbs down feedback toggle
✓ Regeneration on new messages
✗ No distinction between routine and consequential tickets
  (uniform treatment of non-uniform risk)
✗ No accountability model
  (undefined until the first escalation forces the question)
✗ No counter-metric on decision quality
  (adoption could look healthy while trust quietly erodes)
✗ No instrumentation beyond a binary feedback toggle
  (the exact week-three failure pattern would be invisible)
✗ No service contract for the summarization call
  (timeout, failure shape, and retry behavior all undefined)
✗ No stale-state handling on regeneration
  (engineer could read a summary reflecting a thread state that no longer exists)
✗ No states beyond the success case
  (loading, failure, and stale states left to agent inference)
✗ No honest completeness signal
  (risk of an invented "confidence score" that sounds rigorous but isn't)
✗ No operational SLO or monitoring
  (silent degradation indistinguishable from normal operation)
✗ No concurrency handling on regeneration
  (race condition on fast-moving threads)
```

The agent would have shipped a working feature.
Four acceptance criteria. Correctly implemented. Tests passing.

The feature would have looked successful in week one.
Adoption would have climbed. Nobody would have flagged a problem —
because nothing in the spec was designed to detect the problem that
was coming. By week three, the cost would have shown up as a
customer escalation, a post-mortem, and a quiet, permanent drop in
engineer trust that no metric in the original spec was built to see.

None of this would have been visible in code review.
All of it was visible in the spec review.

---

## What Each Persona Uniquely Caught

| Persona | Primary contribution | Class of problem prevented |
|---|---|---|
| Amazon PM | Wrong customer (average case vs. consequential case), wrong failure definition, no accountability model, unjustified uniform scope | Built for the easy case, false-positive metric, reactive accountability |
| Google PM | Missing counter-metric, missing instrumentation, no rollback trigger | Metric gaming, invisible trust decay, no early warning |
| Architect | Service contract, heuristic computation placement, stale-summary window | Race conditions, blocked page on service failure, silently outdated output |
| UI Designer | Full state set, risk indicator visibility, source-jump path | Invisible safety signal, unhandled states, warning with no resolution path |
| QA Functional | Honest completeness signal (not fake confidence), failure negative path, regression surface | False-confidence feature, unhandled failure, regression gaps |
| QA NFQ | Pre-flight contract check, SLO, monitoring, concurrency | Silent degradation, no reliability baseline, race conditions |

No persona duplicated another's primary contribution.
All six were required. No single persona alone was sufficient.

---

## The Uncomfortable Implication

The PM who wrote v1 is not a bad PM.
They identified a real time-sink. They scoped a reasonable feature.
They set an accuracy bar and added a feedback mechanism.

They missed the customer definition at the point of highest stakes,
the difference between aggregate accuracy and decision-relevant
completeness, the accountability question, the counter-metric, the
instrumentation needed to detect silent trust decay, and the
service-level contract that keeps the feature from breaking the
page it lives on.

Not because they are inadequate.
Because no single human holds all of these lenses simultaneously
while also managing a sprint, attending stakeholder meetings, and
writing the next three specs in the backlog.

That is not a skills problem.
It is a cognitive load problem.
And it has a structural solution.

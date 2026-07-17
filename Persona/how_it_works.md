# HOW-IT-WORKS.md
> The Sparring pipeline — sequence, persona selection, tensions, and the human gate.
> Read this before running your first pipeline.
> If a persona flags something unexpected, read this to understand why.

---

## The Pipeline

Sparring runs five personas in sequence. Each persona has one cognitive job.
Each persona's output is the next persona's input.

```
[PM: Amazon or Google] → [Architect] → [UI Designer] → [QA Functional] → [QA NFQ] → [@Synthesis]
```

```
PM defines the problem correctly
        ↓
Architect validates the solution fits the system
        ↓
UI Designer ensures every user state is specified
        ↓
QA Functional ensures every path is testable
        ↓
QA NFQ ensures the spec survives reality
        ↓
@Synthesis produces the SPARRING BRIEF
```

The sequence is not arbitrary. Running personas out of order degrades output quality.
Specific consequences are documented below under "Sequence Consequences."

**The human edits the spec. The persona never does.**

Each persona flags blocking items and conditional items. You resolve blocking items
before moving to the next persona. Conditional items are your judgment call.
A spec that passes every persona without a single override is a red flag —
it means the personas found nothing to say, which means they were not read carefully.

---

## The SPARRING BRIEF — Three-Tier Output

After all personas complete, invoke @Synthesis:

```
"@Synthesis — produce the SPARRING BRIEF from today's pipeline run on [spec ID]."
```

@Synthesis reads the Tier 1 snippets each persona produced, deduplicates convergent
findings, ranks by surprise value, and produces a three-tier output:

```
Tier 1 — VERDICT (30 seconds)
  Overall verdict + top 3 findings, one line each.
  Convergent findings named. Read always.

Tier 2 — ACTION INDEX (one line per item)
  Every blocking item as one line, grouped by owner.
  PM | Architect | Legal | Admin | AMBIENT?
  No restatement of Tier 1. Read when acting.

Tier 3 — FULL REVIEWS
  Complete persona output, unchanged.
  Read when challenging a finding.
```

Nothing is removed. Reading order is controlled.

**Finding tags:**

```
⚡ NEW      — Not in SPARRING_CONTEXT.md. Likely a real gap. Act on it.
◎ AMBIENT? — May be org knowledge. Confirm before acting.
               Add to SPARRING_CONTEXT.md if it IS org knowledge.
               Treat as a real gap if it is NOT.
~ KNOWN    — Already in SPARRING_FINDINGS.md or flagged in the spec.
               Not new. Surfaces for completeness only.
```

**The AMBIENT? tag is the context file building itself.** After every first pipeline
run on a new product area, work through the AMBIENT? items in Tier 2 — each one
tells you exactly which section of SPARRING_CONTEXT.md to add it to. Ten minutes
of work. The next run will be materially cleaner.

See SPARRING_SYNTHESIS.md in this folder for the full @Synthesis instruction.

---

## Cross-Reference Convention — When Findings Touch the Same Mechanism

This is a pipeline rule, not a persona preference.

As the pipeline runs, later personas read the findings of earlier personas before
producing their own output. When a persona identifies a finding that touches the
same mechanism as an earlier finding — even from a different angle — it must
state that relationship explicitly rather than presenting the finding as new.

**Why this matters:**

A single mechanism can appear in multiple persona outputs. The Amazon PM may
identify that a trust signal is needed (accountability gap). QA Functional may
define what that signal actually measures (structural heuristics). The Architect
may define where it runs and what it costs (synchronous, cached). Three findings,
one mechanism, three different contributions. This is the pipeline working correctly.

What breaks this is when the relationship between those findings is implicit —
when the reader has to notice on their own that three personas are talking about
the same thing. The cross-reference convention makes that relationship visible.

**The rule:**

Before a persona finalises its findings, it checks SPARRING_FINDINGS.md
(or prior findings in the same session) for any finding that touches the
same mechanism it is about to flag. If one exists:

- State the relationship explicitly: "This finding builds on / extends /
  confirms / was anticipated by [Persona] Finding [N]."
- Do not re-flag what the earlier persona already caught — add the new angle.
- If the findings appear to contradict, flag the contradiction explicitly
  rather than silently resolving it.

**Three relationship types to name:**

| Relationship | What it means | How to flag it |
|---|---|---|
| Sequential | Later persona defines a different layer of the same mechanism | "Builds on Finding N — [earlier persona] identified the need, this finding defines [the what / where / how]." |
| Convergent | Two personas reach the same conclusion for different reasons | "Converges with Finding N — same outcome, different rationale. Both hold independently." |
| Contradictory | Two findings point toward incompatible solutions | "Conflicts with Finding N — [describe the tension]. Human resolution required before proceeding." |

**Convergent findings are not duplicates — name why both hold:**

When two personas converge on the same recommendation (e.g., phased rollout),
the Delta should state whether they converge for the same reason or different
reasons. Convergence for different reasons is not guaranteed to hold in every
spec — it happened to hold in this run. Making that explicit protects future
readers who might assume the recommendation is more robust than it is.

**The Delta_Spec cross-reference section:**

Every worked example in this library includes a Cross-References section in
Delta_Spec.md that maps all findings touching the same mechanism, names the
relationship type, and confirms whether any contradictions were resolved or
remain open. This section exists so the reader does not have to trace
relationships manually across twenty findings.

When you run the pipeline on your own spec, you do not need to produce a
formal cross-reference section unless you are writing a Delta for publication
or handoff. What you do need is to flag the relationship inline in each
finding as it arises. That is the minimum the convention requires.

---

## Persona Selection Guide

### PM Persona: Amazon vs Google

| Choose Amazon PM when | Choose Google PM when |
|---|---|
| Problem space is new or poorly understood | Problem space is well-understood by team |
| Spec involves irreversible decisions (auth strategy, data model) | Feature is exploratory — learning, not delivering |
| Stakeholder alignment needs documented customer narrative | Team has strong existing customer knowledge |
| Feature has external dependencies or compliance implications | Speed of iteration matters more than upfront completeness |
| First time building in this domain | Third or fourth iteration on a known pattern |

**Can you run both?** Yes, on high-stakes features. Run Amazon PM first,
then Google PM. Amazon PM defines the customer problem. Google PM ensures
the feature is measurable and instrumented. They address different layers.
Combined review time is approximately 2x a single PM review —
worth it for Type 1 decisions, excessive for Type 2.

**Resolution when they conflict:**
Amazon PM will block on missing customer narrative.
Google PM will block on missing success metric.
Neither overrides the other. Both blocking items must be resolved.
The human decides the depth of customer narrative required —
Google PM's comfort with implicit knowledge vs Amazon PM's demand
for explicit documentation is a judgment call based on team context.

---

## Persona Pair Tensions

### Tension 1: Architect ↔ QA NFQ (constructive — do not collapse)

**What they share:** Both flag external service contract gaps.
Architect checks: does the contract exist? (module, timeout, failure shape)
QA NFQ checks: does the contract have the right values? (5000ms, not "default")

**Why the redundancy is intentional:**
Architect may pass a spec that has timeout = "5000ms as per ARCHITECTURE.md"
without the value in the spec itself.
QA NFQ will catch this — the agent needs the value in the spec,
not just a reference to another file.

**Resolution:** Both can flag the same gap from different angles. Let them.
The human resolves once. The spec improves once. No duplication in the fix.
Per the cross-reference convention above: QA NFQ should name this as a
pre-flight confirmation of the Architect's finding, not a new independent block.

---

### Tension 2: UI Designer ↔ QA Functional (constructive — do not collapse)

**What they share:** Both flag missing error state coverage.
UI Designer flags: error state not specified — user has no message.
QA Functional flags: error AC not binary — cannot write a test.

**Why the redundancy is intentional:**
UI Designer catches presentation gaps (what does the user see?).
QA Functional catches testability gaps (can I write a pass/fail test for this?).
A message can be specified but still not testable.
A message can be testable but poorly worded.
Different concerns, same spec section.

**Resolution:** Apply both sets of recommendations. They address different
dimensions of the same AC. Not competing — complementing.
Per the cross-reference convention: each should name the other's finding
and confirm the relationship is complementary, not duplicative.

---

### Tension 3: Amazon PM ↔ Architect (watch for over-specification)

**What creates conflict:**
Amazon PM's demand for precise customer definition and problem statement
can produce a spec so narrative-heavy that the Architect is reviewing
prose, not architecture.

Architect needs: module ownership, external service contracts, failure modes.
Amazon PM may add: several paragraphs of customer context that are
valuable for alignment but not for architecture review.

**Resolution:** After PM review and human edits, the Architect reads
the spec for architecture-relevant content only. The Architect persona
is not required to comment on customer narrative sections it cannot
assess. If the narrative sections displace technical sections in a spec
that has a page limit (some teams impose one), flag it. Otherwise: no conflict.

---

### Tension 4: Google PM ↔ QA NFQ (watch for SLO over-commitment)

**What creates conflict:**
Google PM will add success metrics. QA NFQ will assess whether those
metrics are achievable given the architecture.

A Google PM may add: "p95 response time < 200ms."
QA NFQ will flag: "The external service p95 latency is 300–500ms.
This SLO is not achievable for this flow as specified."

This is not a conflict. This is the pipeline working correctly.
The PM set an aspirational target. The NFQ check reveals it is
mathematically impossible. Human resolves.

**Resolution:** Revise SLO to what is actually achievable, or change
the architecture (caching, pre-processing, etc.) to make it achievable.
Do not leave an impossible SLO in the spec — the agent cannot build
to a target that the underlying dependencies prevent.
Per the cross-reference convention: QA NFQ should name the Google PM
finding it is responding to and frame this as a contradictory relationship
requiring human resolution — not a new independent block.

---

### Tension 5: UI Designer ↔ Architect (escalate, do not resolve locally)

**What creates conflict:**
UI Designer may spec an interaction that implies backend complexity.
Example: "Show real-time field validation as user types."
This implies: debounced API calls on keypress, or client-side
validation rules pushed from backend.
Neither is a UI decision — both are architecture decisions.

**Resolution:** UI Designer flags the implication. Does not resolve it.
Routes back to Architect. Human decides which approach to take.
Then Architect reviews the resulting technical approach.
UI Designer never makes backend architectural decisions.
Architect never overrides UI interaction specifications.

---

## Sequence Consequences — What Breaks When Order Changes

### Running QA NFQ before QA Functional
**Consequence:** QA NFQ reviews resilience on paths that QA Functional
has not yet verified are fully specified. NFQ validates timeout handling
on a failure path that may not even have a binary AC yet.
Result: incomplete resilience spec because the functional spec
beneath it was incomplete.
**Fix:** Always run QA Functional before QA NFQ.

### Running UI Designer before Architect
**Consequence:** UI Designer may spec loading states and error messages
for failure modes the Architect has not yet reviewed.
Some of those failure modes may be removed or restructured by the Architect.
Result: UI work on flows that change in the next review.
**Fix:** Always run Architect before UI Designer.

### Running any QA persona before both PM and Architect
**Consequence:** QA is writing test cases for an incomplete problem definition
built on an unreviewed architecture.
The problem definition may change (PM review) and the architecture
may change (Architect review). Both would invalidate QA findings.
**Fix:** PM and Architect always precede QA.

### Skipping PM entirely
**Consequence:** No success metric exists. Agent builds to functional
completion but has no definition of done beyond "it works."
Post-launch: no way to know if the feature achieved its purpose.
**Fix:** PM gate is not optional. Google PM is faster than Amazon PM
for well-understood features — but some PM review always runs.

### Skipping UI Designer
**Consequence:** Error states are specified as HTTP codes but not as
user messages. Loading states are absent.
Agent builds technically correct code that produces blank pages
and machine-readable error codes for users.
**Fix:** UI Designer gate is not optional for user-facing features.
It may be skipped for pure backend/API features with no user-facing output.

---

## Pre-Tested Combinations by Project Type

### Combination A: Consumer App (mobile-first)
```
Google PM → Architect (SCALE_MODE=proportional) → UI Designer
(COPY_MODE=strict) → QA Functional (SECURITY_MODE=auth-only)
→ QA NFQ (SCALE_MODE=proportional)
```
Notes:
- Google PM: move fast, instrument well
- UI Designer strict copy mode: consumer apps live or die on microcopy
- QA Functional auth-only security: standard auth, not enterprise SSO

### Combination B: SaaS B2B
```
Amazon PM → Architect (SCALE_MODE=proportional) → UI Designer
(COPY_MODE=standard) → QA Functional (SECURITY_MODE=auth-only)
→ QA NFQ (SCALE_MODE=proportional)
```
Notes:
- Amazon PM: B2B specs benefit from customer narrative (multiple buyer roles)
- UI Designer standard copy: B2B users tolerate more technical language
- Proportional NFQ: paying customers, so SLO blocks apply on critical paths

### Combination C: Startup MVP
```
Google PM → Architect (SCALE_MODE=proportional) → QA Functional
(SECURITY_MODE=auth-only) → QA NFQ (SCALE_MODE=proportional)
```
Notes:
- UI Designer is optional for MVP backend features. Include for
  user-facing features, skip for internal APIs.
- Google PM: bias toward instrumentation over narrative
- NFQ proportional: SLO WARNs not BLOCKs at MVP stage

### Combination D: Enterprise / High Compliance
```
Amazon PM → Architect (SCALE_MODE=strict) → UI Designer
(COPY_MODE=strict) → QA Functional (SECURITY_MODE=full)
→ QA NFQ (SCALE_MODE=strict)
```
Notes:
- Amazon PM: compliance features need precise customer definition
- Strict everywhere: full SRE rigour, full security testing, exact copy
- Full security mode: OWASP Top 10 on all features, not just auth flows
- This combination produces the longest reviews and the most blocking items.
  It is appropriate when the cost of a production failure is high.

---

## Quick Reference: What Each Persona Will NOT Do

| Persona | Will not do |
|---|---|
| Amazon PM | Review technical decisions, assess UI quality, validate failure modes |
| Google PM | Require customer narrative, assess architecture, validate failure modes |
| Architect | Review UI, write the solution, make product decisions |
| UI Designer | Make backend decisions, perform WCAG audit, write exact copy |
| QA Functional | Assess resilience or performance, make architectural recommendations |
| QA NFQ | Review functional correctness, assess UI, make product decisions |

Any persona that starts doing work outside this boundary is a signal
that the spec has an unusual structure. Flag it. Do not normalise it.

---

## The Human Gate — When to Override a Persona

Every persona has a bias section. That section exists precisely for this.

**Override criteria:**
1. The persona's blocking item is driven by its documented bias
2. The specific context makes the concern inapplicable
3. The human can state why in one sentence

**How to override:**
Add to spec changelog: "[Persona] blocked on [item]. Override: [one-sentence reason]."
Document it. Do not silently ignore it.

**What not to override:**
- ARCHITECTURE.md violations (these are project-specific hard rules)
- EDGE_CASES.md-sourced findings (these are real bugs from this project)
- Security checklist items on auth flows (these are OWASP requirements)

Overriding these without documented reason is not an override —
it is a gap that will surface in production.

# Sparring

A library of AI personas that review your spec before your agent builds from it.

Your agent builds exactly what the spec says.  
If the spec is incomplete, the agent builds something incomplete — correctly.  
Sparring catches the gaps before the build starts.

---

---

# Part 1 — The Library

What Sparring is, how it works, what it is not, and how to run it on your own spec.

---

## The problem it solves

The spec is the product manager's artifact. And product management is not a universal discipline. It is shaped by company culture, by what the organization measures, by the specific product being built, and by the individual PM's experience and blind spots. Two PMs writing a spec for the same feature will produce different documents — not because one is wrong, but because each brings a different set of lenses and leaves different gaps.

This is where spec-level hallucination lives - unknown assumptions not missing facts, unasked questions not wrong answers.

The Sparring library is built to surface them.

In a non-AI world, an architect building the high level design based on the spec might catch the unsaid assumptions. An expert reviewing the spec to build the Low level design flags might flag the missing Non functional asks. A human developer will sometimes catch a spec gap during mid-implementation, the act of writing code makes the missing piece visible. They flag it and spec if fixed.

An AI agent does not do this. It builds what the spec says.

An agent does not hesitate. It reads "generates a summary" and it generates a summary — confidently,
completely, and exactly as specified, including everything the spec forgot to specify. No pause at
the ambiguous line. No comment asking what happens if the service times out. The gap that a human
might have stumbled on gets built, tested against its own flawed premise, and shipped — faster than
before, with less visible struggle along the way. The spec's blind spots don't get caught later.
They get executed more efficiently.

This is what Sparring interrogates before any of that happens: five review lenses, run sequentially,
against the spec and the design, before a task list is generated and before an agent starts building.
Each lens has one job and non-negotiables it will not pass regardless of what else looks fine.
The human resolves what gets flagged. The agent does not touch the spec. Nothing gets built until
the gate clears.


---

## How it works

Five personas plus a synthesis step. Each enforces one layer of spec quality.

```
PM (Amazon or Google)  →  Is the problem correctly defined? Is success measurable?
Architect              →  Does this fit the system? Are contracts with external services defined?
UI Designer            →  Are all user-facing states specified? What does failure look like?
QA Functional          →  Is every path testable? Are negative paths covered?
QA NFQ                 →  Will it survive reality? Timeouts, SLOs, failure visibility.
@Synthesis             →  Aggregates all findings. Produces the SPARRING BRIEF.
```

Each persona reads your spec. It flags blocking items and conditional items. You resolve blocking items before moving to the next persona. Conditional items are your judgment call.

**The human edits the spec. The persona never does.**

The sequence is not arbitrary — each persona's output is the next persona's input. Running them out of order degrades the result. The full explanation is in [`/Persona/how_it_works.md`](Persona/how_it_works.md).

---

## Two ways to use Sparring

### Journey 1 — On-Demand (while drafting)

Invoke any single persona on any specific decision while you are writing or refining a spec. Works in CLI, IDE, or chat. No file setup required. No pipeline to run.

Use this when:
- You are mid-draft and want one lens on a decision you are about to make
- You want a quick challenge on a UI change, an architectural assumption, or a scope boundary — before the spec is complete
- You are working in chat (Gemini, Claude.ai, or any chat interface) and want a persona to interrogate a specific section

What the persona does:
- Scans the conversation for any prior Sparring output automatically — no copy-paste required
- States what it found before reviewing — correct it if wrong
- Derives project context from the conversation if no config files exist
- Labels output: **Journey: On-Demand — Directional**

What it is not:
- Not a formal gate. Not a blocking verdict.
- Thinking support while you draft — not a sign-off before build.
- Does not create or require SPARRING_FINDINGS.md

---

### Journey 2 — Full Pipeline (before build)

Run all personas sequentially on a complete spec before handing it to an agent. Each persona reads prior findings. SPARRING_FINDINGS.md accumulates. Cross-reference convention activates fully.

Use this when:
- Your spec is considered complete and you are ready to hand it to an agent
- You want a formal multi-lens review with a blocking verdict
- You need an audit trail — for a design review, stakeholder handoff, or your own confidence before build

What the persona does:
- Reads SPARRING_FINDINGS.md from prior runs
- Produces a formal FINDINGS PAYLOAD
- Cross-references findings from other personas by relationship type
- Labels output: **Journey: Full Pipeline — Formal Gate**

What it is not:
- Not recommended in chat mode — SPARRING_FINDINGS.md cannot persist between chat sessions automatically. Use CLI or IDE for full pipeline runs.

---

| | Journey 1 — On-Demand | Journey 2 — Full Pipeline |
|---|---|---|
| When | While drafting | Spec complete, before build |
| Personas | Any single persona | All personas, sequentially |
| Setup required | None | PROJECT CONFIG + context files |
| Prior findings | Scanned from conversation automatically | Read from SPARRING_FINDINGS.md |
| Output type | Directional — thinking support | Formal gate — blocking verdict |
| Output mode | Summary (default) or Detailed — your choice | Summary (default) or Detailed — your choice |
| SPARRING_FINDINGS.md | Not required | Required across sessions |
| Works in chat | Yes | Not recommended |
| Works in CLI / IDE | Yes | Yes |

---

---

## Output modes

Every persona supports two output modes. Set this in the PROJECT CONFIG block inside each persona file, or state it explicitly when you invoke the persona.

**Summary mode (default)**

One verdict line, top two blocking items, and any conditional items requiring human judgment. Designed for the first pass — get the signal fast, decide what to fix, move on.

```
Example invocation: "Run @AmazonPM in summary mode on this spec."
```

**Detailed mode**

Full structured review across every dimension the persona covers — every AC audited, every assumption logged, every gap traced to its source. Use this when a persona flags a block you don't fully understand, or when you need the full audit trail before a design review or stakeholder handoff.

```
Example invocation: "Run @AmazonPM in detailed mode on this spec."
```

The default is summary. If the output feels thin, switch to detailed. If the output overwhelms, you are already in detailed mode on a spec with structural gaps — the length is the spec's report card, not the library's.

**Three-tier output (full pipeline runs)**

When running the full pipeline, invoke @Synthesis after all personas complete. It produces a SPARRING BRIEF — three tiers with non-overlapping jobs:

| Tier | What it is | When to read it |
|---|---|---|
| Tier 1 — SPARRING BRIEF | Verdict + top 3 findings, one line each. Convergent findings named. | Always — read this first. Takes 30 seconds. |
| Tier 2 — Action Index | Every blocking item as one line, grouped by owner (PM / Architect / Legal / Admin). No restatement of Tier 1. | When acting on findings. |
| Tier 3 — Full Reviews | Complete persona output, unchanged. | When challenging a finding or building an audit trail. |

The full output is not reduced. The entry point is controlled.

```
Example invocation: "@Synthesis — produce the SPARRING BRIEF from today's pipeline run on [spec ID]."
```

**Finding tags in the Brief:**

| Tag | Meaning | What to do |
|---|---|---|
| ⚡ NEW | Not in SPARRING_CONTEXT.md. Likely a real gap. | Act on it. |
| ◎ AMBIENT? | May be org knowledge — persona could not confirm. | Add to SPARRING_CONTEXT.md if it IS org knowledge. Treat as a real gap if it is NOT. |
| ~ KNOWN | Already in SPARRING_FINDINGS.md or flagged in the spec. | Not new — surfaces for completeness only. |

---

---

## Running a single persona

You do not have to run the full pipeline. Invoke any single persona by name at any time — either as part of Journey 1 (On-Demand, while drafting) or as a targeted pass within Journey 2 (Full Pipeline).

```
On-Demand, summary (default):
"Review this UI change as @UIDesigner. Journey: On-Demand."

On-Demand, detailed:
"Review this UI change as @UIDesigner. Journey: On-Demand. Output: detailed."

Full Pipeline, summary (default):
"Run @AmazonPM on this spec. Journey: Full Pipeline."

Full Pipeline, detailed:
"Run @AmazonPM on this spec. Journey: Full Pipeline. Output: detailed."
```

**When to run a single persona**

- You are mid-draft and want one lens on a specific decision (Journey 1)
- You already know which layer is weak and want a targeted review (Journey 2)
- You want a focused answer fast — one persona on a near-final spec produces an actionable gap list without the full pipeline overhead
- You want to chain personas one at a time across sessions — each persona reads SPARRING_FINDINGS.md from the prior run (Journey 2)

**In Journey 1 (On-Demand):** The persona scans the conversation for prior Sparring output automatically. No copy-paste required. State your assumed context in the invocation if no config files exist — the persona will confirm what it understood before reviewing.

**In Journey 2 (Full Pipeline):** When a single persona runs, its findings are saved as a FINDINGS PAYLOAD block. Copy this into SPARRING_FINDINGS.md. The next persona reads that file before reviewing — building on prior findings rather than duplicating them.

**The trade-off**

| | Full pipeline | Single persona |
|---|---|---|
| Coverage | All five lenses in one pass | One lens only |
| Output | Three-tier SPARRING BRIEF via @Synthesis | Review + findings payload |
| Cross-persona carry-over | Handled internally in one session | Via SPARRING_FINDINGS.md across sessions |
| Best for | Complete spec approaching build | Targeted review or iterative drafting |

**Run order still matters when chaining.** The recommended sequence is PM → Architect → UI Designer → QA Functional → QA NFQ → @Synthesis. Running out of order is allowed — the persona will note missing prior findings and proceed. Use your judgment.

---

---

## SPARRING_FINDINGS.md — saving output between runs

When you run a single persona, the output includes a `FINDINGS PAYLOAD` block at the end of the review. Copy this block into a file called `SPARRING_FINDINGS.md` in your project root.

The next persona you invoke will read this file before reviewing your spec. It uses the prior findings to avoid re-flagging already-known blockers and to build on what the previous persona established.

**The findings payload contains:**
- Spec identifier (the name or ID you gave the spec — see below)
- Date of the run
- Persona that produced it
- Verdict
- Blocking items list
- Flagged assumptions

**If you run the full pipeline in a single session**, findings carry over internally — you do not need `SPARRING_FINDINGS.md`. The file is only needed when runs happen across separate sessions or separate conversations.

---

---

## SPARRING_CONTEXT.md — reducing noise over time

Every persona checks this file before flagging any finding. It answers one question: **what does your org already know that should not appear as a gap in a spec?**

Without it, personas flag role definitions, settled decisions, and ambient org knowledge as gaps — because they cannot tell the difference between a real gap and something everyone on your team already knows. The result is signal mixed with noise.

With it, personas suppress findings for anything documented in the file and flag everything else as either `⚡ NEW` (real gap) or `◎ AMBIENT?` (may be org knowledge — confirm before acting).

**The file has five sections:**

```
Section 1 — Terminology & role definitions
            Any term that appears in your specs without being defined there.
            Example: "Agent: a contact centre agent — not an AI agent"

Section 2 — Decisions already made
            Settled architectural or product decisions that do not need re-justification.
            Example: "English-only transcription for MVP — validated with customers"

Section 3 — Known ambient knowledge
            Things your entire org knows that never appear in specs.
            Example: "All voice interactions use SIP unless stated otherwise"

Section 4 — Out of scope for this project
            Anything explicitly excluded — personas will not flag its absence as a gap.

Section 5 — Customer context
            Named enterprise customers with known requirements and commitment references.
```

**The AMBIENT? items in the SPARRING BRIEF are the context file building itself.**

After your first pipeline run, every `◎ AMBIENT?` item tells you exactly which section to add it to. Work through those items once — ten minutes — and the next run on any spec in the same product area will be materially cleaner.

The library improves with use. Populate `SPARRING_CONTEXT.md` after every first run on a new product area.

---

---

## Output header — date, spec identifier, and traceability

Every persona output begins with a standard header:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SPARRING REVIEW
Spec:     [your spec identifier]
Date:     [date of run]
Persona:  [persona name]
Mode:     summary | detailed
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Spec identifier** — use something that uniquely identifies this spec and stays stable across runs. A Jira ticket key, a story ID, a short canonical name — whatever your team uses. The identifier links the findings payload to the spec it reviewed. If you run Sparring on the same spec twice (after a significant revision), the identifier plus date gives you a clear before/after trail.

There is no enforced format. The onus is on you to use an identifier that will make sense to you in three months when you are looking at a `SPARRING_FINDINGS.md` with four entries in it.

---

---

## What level does this work at

Sparring works at the **feature or epic level** — one spec, one pipeline run.

This matters because the value the library produces is proportional to the specificity of the spec you give it. A feature spec with acceptance criteria gives every persona something concrete to interrogate. An initiative brief or a product-level vision document does not — the Architect has no module boundaries to check, the QA personas have no paths to verify, the UI Designer has no states to audit. You will get generic output on a generic input.

The right trigger for running Sparring is the moment a feature spec is written and before it is handed to an agent or an engineer. Not at roadmap planning. Not at initiative kickoff. At the spec.

**What Sparring is not:**

- **Not a product strategy tool.** The Amazon PM persona will ask "who is the customer?" — but it cannot tell you whether your answer is correct. It enforces that the answer exists in the spec. Whether the answer is right is upstream of this library entirely. That work belongs in discovery.

- **Not a substitute for a human architecture review.** The Architect persona catches spec-level gaps — missing contracts, undefined failure modes, module boundary violations implied by the spec. It does not replace a technical design review with engineers who know the codebase.

- **Not a one-time pass.** Specs change. If a feature spec changes significantly after a persona has reviewed it, the affected personas should run again. The pipeline is valid against the spec version it reviewed — not against whatever the spec became afterward.

- **Not a tool for reviewing an entire product at once.** The temptation when you first see this library is to point it at everything. One spec, one pipeline run. That is the unit of work.

---

---

## Run it on your own spec

**Step 1 — Pick a starter kit**

Three pre-configured persona stacks matched to project type. Each kit specifies which personas to run, in what order, and how to configure the behavior switches.

| Kit | Use when |
|---|---|
| [`consumer_app.md`](StarterKit/consumer_app.md) | Mobile-first or web consumer product. This is the kit used in the worked example. |
| [`saas_b2b.md`](StarterKit/saas_b2b.md) | Multi-role SaaS with paying customers and external integrations. |
| [`startup_mvp.md`](StarterKit/startup_mvp.md) | Pre-revenue validation build where speed matters more than completeness. |

Not sure which fits, or what to do when two personas disagree? → [`Persona/how_it_works.md`](Persona/how_it_works.md)

**Step 2 — Create your project context files**

Each persona reads three files before reviewing your spec.

`SPARRING_CONTEXT.md` — org knowledge, role definitions, settled decisions, out-of-scope items, and customer context. Create this in your project root alongside `SPARRING_FINDINGS.md`. It can start empty — the `◎ AMBIENT?` items from your first pipeline run will tell you exactly what to add. The template is at the root of this library.

`Architecture.md` — your module map, tech decisions, and external service contracts (timeouts, failure shapes, state write policy). If you already have architecture docs, link them. If not, the worked example at [`/Example/ProjectConfig/Architecture.md`](Example/ProjectConfig/Architecture.md) shows the structure.

`Edge_Cases.md` — bugs already discovered on this project, with the rule extracted from each. Starting a new project: create it empty. It grows as the project does.

**Complex projects:** If your architecture lives across multiple files, list all relevant paths in the PROJECT CONFIG block inside each persona file. A persona pointed at one file out of five will miss what's in the other four.

**Step 3 — Fill in the PROJECT CONFIG block**

Each persona file has a PROJECT CONFIG section. Add your product context, stack, and the file paths from Step 2. This takes about five minutes and connects the persona to your project — without it, the persona reviews in a vacuum.

**Step 4 — Run the pipeline**

Download the persona files from `/Persona` and add them to your project. Work through the pipeline one persona at a time: read the output, resolve blocking items, update the spec, move to the next. After all personas complete, invoke @Synthesis to produce the SPARRING BRIEF.

```
Works with: Claude Code, Cursor, Windsurf, or any agent that reads files.
No API. No subscription. No dependency.
```

> **Browser or chat interface (Claude.ai, Gemini, or any chat)?** Two options depending on your goal:
>
> **Journey 1 (On-Demand):** Start your spec drafting conversation normally. When you want a persona to review a specific decision, invoke it by name with "Journey: On-Demand." The persona scans the conversation for prior Sparring output automatically. No file setup required.
>
> **Journey 2 (Full Pipeline):** Open a new conversation. Drop the persona file in as your first message, then your spec. The persona reads both and reviews. Resolve blocking items, update your spec, open a new conversation for the next persona — paste the FINDINGS PAYLOAD from the prior run into context before invoking the next one. After the final persona, run @Synthesis with all prior findings.

> **Existing spec-driven pipeline?** Copy the `/Persona` folder into your project. Add each persona as a review step before the build step. Add @Synthesis as the final review step. That is the full integration — one folder, one new stage in your pipeline.

---

---

## What's in the library

```
/Persona
  /PM_Persona
    amazon_pm_working_backwards.md     Customer definition, problem statement, success metrics
    google_pm_data_driven.md           OKR connection, instrumentation, iteration readiness
    apple_narrative.md                 Narrative coherence, experience integrity, product story
    meta_movefast.md                   Speed-to-learning, instrumentation, reversibility
    enterprise_b2b.md                  Multi-stakeholder definition, compliance surface, buyer logic
  /Architect_Persona
    google_distributed_architect.md    Module boundaries, external service contracts, blast radius
  /Designer_Persona
    ui_designer_interaction_clarity.md Four States Rule, error messages, design system compliance
  /QA_Persona
    qa_functional_path_coverage.md     Binary ACs, negative paths, edge case coverage
    qa_nfq_resilience.md               Full non-functional review — SLOs, resilience, failure visibility
  SPARRING_SYNTHESIS.md                Pipeline synthesis step — produces the SPARRING BRIEF
  how_it_works.md                      Pipeline sequence, persona selection, tensions, human gate

/StarterKit
  consumer_app.md                      Pre-configured stack for consumer products
  saas_b2b.md                          Pre-configured stack for B2B SaaS (used in example)
  startup_mvp.md                       Pre-configured stack for pre-revenue builds

/Example
  Input_Spec.md                        The thin 4-AC spec entering the pipeline
  /sparring-test                       The live OpenSpec + Sparring pipeline run
    /openspec/changes/ai-ticket-summary/
      /specs
        ai-ticket-summary.md           The resolved spec after all 14 blocking items
      design.md                        Technical design — architecture context for personas
      sparring_pm.md                   PM persona findings — 2 blocks, 3 conditionals
      sparring_architect.md            Architect findings — 3 blocks, 2 conditionals
      sparring_ui.md                   UI Designer findings — 3 blocks, 4 conditionals
      sparring_qa_functional.md        QA Functional findings — 4 blocks, 5 conditionals
      sparring_qa_nfq.md               QA NFQ findings — 2 blocks, 3 conditionals
      sparring_brief.md                Synthesis — PIPELINE PASS, Tier 1/2/3 output
  /ProjectConfig
    Architecture.md                    Example module map, service contracts, hard rules
    Edge_Cases.md                      Example discovered bugs with extracted rules
    Edge_Cases.md                      Example discovered bugs with extracted rules

SPARRING_CONTEXT.md                    Org knowledge template — terminology, decisions, ambient
                                       knowledge, out-of-scope items, customer context.
                                       Reduces false positives across all personas.
Persona_Template.md                    Base structure — copy to create new personas
```

---

---

## The non-negotiable floor

The QA NFQ persona has a full non-functional review — SLOs, blast radius, resilience, monitoring, token lifecycle, recovery time. All of it is configurable by project scale.

Five things are not configurable. The NFQ persona runs these as a pre-flight check before it reads anything else. If any of the five fail, the spec does not proceed regardless of what else it contains:

```
1. Every external service call has an explicit timeout — not a language default
2. Every external service call has a defined failure response shape
3. No auth state written before external service confirmation
4. Token expiry boundary behaviour defined on any feature that touches a JWT
5. Failures are loud — not silent
```

These five are the minimum that makes a spec buildable. Everything above them scales with your product's maturity. These do not.

An MVP with no SLO commitments is reasonable. An MVP where the agent uses Node's default 120-second HTTP timeout on a Google OAuth call — because the spec said nothing — is not an MVP. It is a blank loading screen waiting to happen.

---

---

## The bias sections

Every persona documents where it over-indexes.

An Amazon PM will demand a customer narrative on a feature the whole team already understands. A Google Architect will flag distributed systems concerns on a feature that serves two hundred users. A QA NFQ persona will surface timeout risks on a feature with no external calls.

This is not a defect. It is the point.

The bias section tells you when to push back. A spec that passes every persona without a single override is a red flag — it means the personas found nothing to say, which means they were not read carefully enough.

The value is in the friction.

The tasks.md that the pipeline produces at the end is not a generic implementation checklist.
It is the accumulated resolution of every blocking item made executable — idempotency keys,
failure counters, timeout values, evaluation harnesses. Tasks that could not exist until the
spec was good enough to generate them from.

---

---

## Relationship to Superpower

[Superpower](https://github.com/SuperpowerCorp/superpower) enforces the coding phase.
Sparring enforces the specification phase.

```
Thin spec
    ↓  Sparring — spec quality gate
Contract-grade spec
    ↓  Superpower — process layer
Code that does what was specified
```

Use Sparring before Superpower.
Sparring without Superpower still works.
Superpower without Sparring misses the upstream problem.

---

---

## Use with OpenSpec

If you use [OpenSpec](https://github.com/Fission-AI/OpenSpec) for spec-driven development, Sparring ships a community schema that inserts the adversarial review gate directly into your OpenSpec workflow between `design` and `tasks`.

```
proposal → specs → design → [SPARRING GATE] → tasks → apply
```

Install instructions and full usage guide: [`/openspec-schema`](openspec-schema/README.md)

```bash
# Copy schema into your project
cp -r openspec-schema/openspec/schemas/sparring /your-project/openspec/schemas/sparring

# Add to openspec/config.yaml
schema: sparring
sparring_library_path: /absolute/path/to/Sparring
```

Works alongside [superpowers-bridge](https://github.com/JiangWay/openspec-schemas): Sparring gates the spec, Superpowers executes the build. Use both for the full spec-to-code pipeline.

---

---

# Part 2 — Walkthrough: What Happened When We Ran It

We ran the full Sparring pipeline via OpenSpec against a real feature spec, using Claude Code.
This section is the journey — not documentation, a narrative of what the pipeline actually caught,
what the human decided at each step, and what came out the other side.

---

## The Feature

AI ticket summary for Flowdesk — support engineers get an AI-generated summary at the top of every ticket to reduce time-to-first-action. A fictional B2B SaaS feature, written under real sprint pressure.

**The feature:** AI ticket summary for Flowdesk — support engineers get an AI-generated summary at the top of every ticket to reduce time-to-first-action. A fictional B2B SaaS feature, written under real sprint pressure, with second-order gaps that no single person would catch alone.

**The spec entering the pipeline:** [`/Example/Input_Spec.md`](Example/Input_Spec.md) — four ACs. Realistic. Not careless.

**The pipeline output — five persona reviews, one synthesis:**

| File | Persona | Verdict |
|---|---|---|
| [`sparring_pm.md`](Example/sparring-test/openspec/changes/ai-ticket-summary/sparring_pm.md) | Amazon PM | BLOCK — 2 blocks, 3 conditionals |
| [`sparring_architect.md`](Example/sparring-test/openspec/changes/ai-ticket-summary/sparring_architect.md) | Architect | BLOCK — 3 blocks, 2 conditionals |
| [`sparring_ui.md`](Example/sparring-test/openspec/changes/ai-ticket-summary/sparring_ui.md) | UI Designer | BLOCK — 3 blocks, 4 conditionals |
| [`sparring_qa_functional.md`](Example/sparring-test/openspec/changes/ai-ticket-summary/sparring_qa_functional.md) | QA Functional | BLOCK — 4 blocks, 5 conditionals |
| [`sparring_qa_nfq.md`](Example/sparring-test/openspec/changes/ai-ticket-summary/sparring_qa_nfq.md) | QA NFQ | BLOCK — 2 blocks, 3 conditionals |
| [`sparring_brief.md`](Example/sparring-test/openspec/changes/ai-ticket-summary/sparring_brief.md) | Synthesis | **PIPELINE PASS** |

**Start here:** Read `sparring_brief.md` first — 30 seconds, Tier 1 verdict, top findings, what changed.
Then open individual persona files for the full evidence.

**The spec after all 14 blocking items resolved:** [`specs/ai-ticket-summary.md`](Example/sparring-test/openspec/changes/ai-ticket-summary/specs/ai-ticket-summary.md) — nine ACs, failure condition, accuracy rubric, event bus contract, idempotency strategy, frontend loading timeout, feedback error state, and defined failure response shapes for both endpoints.

The example also includes [`/Example/ProjectConfig/`](Example/ProjectConfig/) — the context files personas read before reviewing:
- `Architecture.md` — module map, tech decisions, service contracts
- `Edge_Cases.md` — discovered bugs with rules extracted from each

> **Note:** The example ran without a SPARRING_CONTEXT.md — which is why each persona applied `◎ AMBIENT?` tags broadly. Creating this file for your own project reduces false-positive flags on domain knowledge the personas cannot verify independently.

**What the human did between personas:**

The pipeline is not autonomous. After each persona ran, the human read the findings, decided
on each conditional item (accept / defer / reject), resolved the blocking items in the spec,
and confirmed before the next persona ran. That human gate is not a formality — it is the
mechanism. The PM persona blocked on a missing failure condition. The human decided what the
failure condition should be and wrote it into the spec. The Architect persona then read the
new spec — and found the PM's addition had no mechanism behind it. The human resolved that too.

Each round's resolution is the next round's raw material. The pipeline cannot produce that
without a human reading and deciding at every step.

**What happened to design.md:**

The agent read the persona files during design.md creation and preemptively satisfied several
non-negotiables — an 8s LLM timeout, a state machine for generation status, a debounce strategy
for event-triggered regeneration. These never became blocking items because the gaps were closed
before the personas ran. The design.md in the example shows what an agent produces when it has
read the Architect and NFQ personas before writing.

**What tasks.md shows:**

[`tasks.md`](Example/sparring-test/openspec/changes/ai-ticket-summary/tasks.md) was generated
after the pipeline cleared. Read it alongside `sparring_brief.md` — every non-trivial task
traces to a persona finding. The idempotency key, the failure counter, the unified failure
display rule, the evaluation harness — all visible in the spec before the build started.
Two tasks are explicitly flagged as blocked on open decisions the human had not yet made.
That is the audit trail working. showing where multiple personas touched the same mechanism from different angles.

The example also includes [`/Example/ProjectConfig/`](Example/ProjectConfig/) — the project context files every persona reads before reviewing your spec:
- `Architecture.md` — the Flowdesk module map, tech decisions, and service contracts.
- `Edge_Cases.md` — three real bugs from the Flowdesk project, each with the rule extracted.

Note: the Flowdesk example reflects the pre-v1.2 output format — persona reviews only, no SPARRING BRIEF. The three-tier output and SPARRING_CONTEXT.md were introduced in v1.2. The example remains valid as a demonstration of the persona review layer. A v1.2 example showing the full pipeline including @Synthesis is in progress.

---

---

## Contributing

If you run the pipeline on a real feature and catch a class of bug none of the personas flagged — open a PR. Document the gap, the class of problem, and the persona or rule that should catch it next time.

If a `◎ AMBIENT?` finding appears repeatedly across different projects and teams — it is no longer ambient knowledge. It is a real gap the library should catch by default. Flag it as a candidate for promotion to a formal persona rule.

See [`Persona_Template.md`](Persona/Persona_Template.md) to build a new persona.

---

*Built on the principle that the most expensive bugs are written into specs, not code.*

# Sparring

A library of AI personas that review your spec before your agent builds from it.

Your agent builds exactly what the spec says.  
If the spec is incomplete, the agent builds something incomplete — correctly.  
Sparring catches the gaps before the build starts.

---

## The problem it solves

A competent developer wrote a six-AC spec for a notification preferences feature.  
It entered the Sparring pipeline and exited with 21 ACs.

The delta contained: unrecoverable error states, missing loading states, no unsubscribe compliance requirement, and no success metric. None of these were careless omissions. They were the gaps a single person cannot hold simultaneously — the PM lens, the architecture lens, the UX lens, the QA lens — while also writing the spec.

Sparring runs those lenses sequentially. Each persona has one job. None of them build anything.

---

## How it works

Five personas. Each enforces one layer of spec quality.

```
PM (Amazon or Google)  →  Is the problem correctly defined? Is success measurable?
Architect              →  Does this fit the system? Are contracts with external services defined?
UI Designer            →  Are all user-facing states specified? What does failure look like?
QA Functional          →  Is every path testable? Are negative paths covered?
QA NFQ                 →  Will it survive reality? Timeouts, SLOs, failure visibility.
```

Each persona reads your spec. It flags blocking items and conditional items. You resolve blocking items before moving to the next persona. Conditional items are your judgment call.

**The human edits the spec. The persona never does.**

The sequence is not arbitrary — each persona's output is the next persona's input. Running them out of order degrades the result. The full explanation is in [`/Persona/how_it_works.md`](Persona/how_it_works.md).

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

## Walk through the example first

Before you run this on your own spec, read the worked example. It takes ten minutes and shows you exactly what to expect.

**The feature:** Notification preferences — users control which notifications they receive.  
**The spec entering the pipeline:** [`/Example/Input_Spec.md`](Example/Input_Spec.md) — six ACs, written by a competent PM under normal sprint conditions.  
**The spec exiting the pipeline:** [`/Example/Final_Spec.md`](Example/Final_Spec.md) — twenty-one ACs.  
**Every change explained:** [`/Example/Delta_Spec.md`](Example/Delta_Spec.md) — each addition traced to the persona that caught it and the class of production bug it prevents.

The example also includes [`/Example/ProjectConfig/`](Example/ProjectConfig/) — the two project context files every persona reads before reviewing your spec:
- `Architecture.md` — the Flowdesk module map, tech decisions, and service contracts. Shows you what a filled-in architecture context looks like.
- `Edge_Cases.md` — three real bugs from the Flowdesk project, each with the rule extracted. Shows you how past failures inform future spec reviews.

You will create equivalent files for your own project. Step 2 explains how.

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

**Step 2 — Point personas to your project context**

Each persona reads two files before reviewing your spec. You either link existing files or create them if they don't exist yet.

`Architecture.md` — your module map, tech decisions, and external service contracts (timeouts, failure shapes, state write policy). If your project already has architecture documentation — even spread across multiple files — list all the relevant paths. The persona reads everything you point it to. The worked example at [`/Example/ProjectConfig/Architecture.md`](Example/ProjectConfig/Architecture.md) shows the structure if you're starting from scratch.

`Edge_Cases.md` — bugs already discovered on this project, with the rule extracted from each. If you have existing post-mortems, incident reports, or a known issues list, link those. Starting a new project: create it empty. It grows as the project does.

**Complex projects:** If your architecture lives across multiple files — frontend, backend, API contracts, data model, service-level docs in a monorepo — list all relevant paths in the PROJECT CONFIG block. The persona reviews against whatever context you give it. A persona pointed at one file out of five will miss what's in the other four.

**Step 3 — Fill in the PROJECT CONFIG block**

Each persona file has a PROJECT CONFIG section at the top. Add your product context, stack, and the file paths from Step 2. This takes about five minutes and is what connects the persona to your project — without it, the persona reviews in a vacuum.

**Step 4 — Run the pipeline**

Download the persona files from `/Persona` and add them to your project. Your agent reads each persona file alongside your spec — the persona already knows where to find your `Architecture.md` and `Edge_Cases.md` because you set those paths in Step 3. Work through the pipeline one persona at a time: read the output, resolve blocking items, update the spec, move to the next.

```
Works with: Claude Code, Cursor, Windsurf, or any agent that reads files.
No API. No subscription. No dependency.
```

> **Browser (Claude.ai)?** Open a new conversation. Drop the persona file in as your first message, then your spec. The persona reads both and reviews. Resolve the blocking items, update your spec, open a new conversation for the next persona.

> **Existing spec-driven pipeline?** Copy the `/Persona` folder into your project. Add each persona as a review step before the build step. That is the full integration — one folder, one new stage in your pipeline.

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
  how_it_works.md                      Pipeline sequence, persona selection, tensions, human gate

/StarterKit
  consumer_app.md                      Pre-configured stack for consumer products (used in example)
  saas_b2b.md                          Pre-configured stack for B2B SaaS
  startup_mvp.md                       Pre-configured stack for pre-revenue builds

/Example
  Input_Spec.md                     The thin spec entering the pipeline
  Final_Spec.md                     The contract-grade spec exiting the pipeline
  Delta_Spec.md                          Every change traced to persona and bug class prevented
  /ProjectConfig
    Architecture.md                    Example module map, service contracts, hard rules
    Edge_Cases.md                      Example discovered bugs with extracted rules

Persona_Template.md                    Base structure — copy to create new personas
```

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

## The bias sections

Every persona documents where it over-indexes.

An Amazon PM will demand a customer narrative on a feature the whole team already understands. A Google Architect will flag distributed systems concerns on a feature that serves two hundred users. A QA NFQ persona will surface timeout risks on a feature with no external calls.

This is not a defect. It is the point.

The bias section tells you when to push back. A spec that passes every persona without a single override is a red flag — it means the personas found nothing to say, which means they were not read carefully enough.

The value is in the friction.

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

## Contributing

If you run the pipeline on a real feature and catch a class of bug none of the personas flagged — open a PR. Document the gap, the class of problem, and the persona or rule that should catch it next time.

See [`Persona_Template.md`](Persona/Persona_Template.md) to build a new persona.

---

*Built on the principle that the most expensive bugs are written into specs, not code.*

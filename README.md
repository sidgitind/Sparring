# Sparring

**The spec quality layer for AI-assisted development.**

Superpower tells your agent *when* to build.  
Sparring tells your agent *what standard to build to*.

They are orthogonal. They stack.

---

## The problem

Process discipline on a bad spec produces well-structured wrong software.

Your agent follows the workflow correctly.  
It builds exactly what the spec says.  
The spec was incomplete.  
The bug surfaces in testing — or in production.

The gap is not in how the agent builds. It is in **what it builds against.**

---

## What this library is

Five personas. Each one a constraint enforcer, not a stylistic advisor.

```
Amazon PM / Google PM    →   Is the problem correctly defined?
Google Architect         →   Does this fit the architecture?
UI Designer              →   Are all user states specified?
QA Functional            →   Are all paths testable?
QA Non-Functional        →   Will it survive reality?
```

A thin spec enters. A contract-grade spec exits.

Every persona has:
- A fixed cognitive contribution — what this role always asks
- Non-negotiables — what it blocks on
- A **bias section** — where it over-indexes, so you stay calibrated
- A project config block — your stack, your thresholds, your rules

The human edits the spec. The persona never does.

---

## Proof

The worked example runs a realistic Google OAuth login spec through the full pipeline.

**Before:** 6 acceptance criteria. No success metric. No error messages. No CSRF protection.  
**After:** 21 acceptance criteria. Instrumented. Recoverable. Secure.

Every addition is traced to the persona that caught it and the class of bug it prevents.

→ [`/Example/09_DELTA.md`](Example/09_DELTA.md)

---

## Start in 10 minutes

```bash
git clone https://github.com/[your-handle]/sparring
```

1. Pick a starter kit → [`/StarterKit/`](StarterKit/)
2. Copy the persona files it references into your project
3. Fill in the `PROJECT CONFIG` block in each persona (~5 min)
4. Write your spec
5. Paste persona + spec into your agent. Read the output. Resolve blocking items. Move to next persona.

**Works with:** Claude Code, Cursor, Windsurf, or any agent that reads files.  
**No API. No subscription. No dependency.**

> **Using Claude.ai in a browser?** Open a new conversation. Paste the entire persona file as your first message. Then paste your spec and send. That is the full workflow.

> **Using Claude Code or Cursor?** Drop the persona file into your project root and reference it in your instruction to the agent.

---

## What's in the library

```
/Persona
  /PM_Persona
    amazon-pm-working-backwards.md       Customer definition, problem statement, success metrics
    google-pm-data-driven.md             OKR connection, instrumentation, iteration readiness
  /Architect_Persona
    google-distributed-architect.md      Module boundaries, external service contracts, blast radius
  /Designer_Persona
    ui-designer-interaction-clarity.md   Four States Rule, error messages, design system compliance
  /QA_Persona
    qa-functional-path-coverage.md       Binary ACs, negative paths, OAuth security checklist
    qa-nfq-resilience.md                 Minimum NFQ Contract, SLOs, failure visibility

/StarterKit
    consumer-app.md                      Mobile-first / consumer products
    saas-b2b.md                          Multi-role SaaS with paying customers
    startup-mvp.md                       Pre-revenue validation builds
    COMPATIBILITY-MATRIX.md              Persona tensions, sequence rules, override criteria

/Example
    00_input_spec.md                     The thin spec that enters the pipeline
    08_final_spec.md                     The contract-grade spec that exits
    09_DELTA.md                          Every change traced to persona + bug class prevented
    /ProjectConfig                       Example ARCHITECTURE.md, EDGE_CASES.md, CLAUDE.md

PERSONA-TEMPLATE.md                      Base structure — copy to create new personas
HOW-IT-WORKS.md                          The orthogonal positioning explained in full
```

---

## The bias sections

Every persona documents where it over-indexes.

A Google Architect will flag distributed systems concerns on a two-person startup feature.  
An Amazon PM will demand customer narrative on a feature everyone already understands.  
A QA NFQ persona will find timeout risks in features with no external calls.

This is not a defect. It is the point.

The bias section tells you when to push back. The human gate is not a formality — it is the intelligence layer.

A spec that passes every persona without a single override is a red flag.  
The value is in the friction. Not in eliminating it.

---

## The Minimum NFQ Contract

Regardless of project size, these five things cannot be skipped:

```
1. Every external service call has an explicit timeout (not language default)
2. Every external service call has a defined failure response shape
3. No auth state written before external service confirmation
4. Token expiry boundary behaviour defined
5. Failures are loud — not silent
```

These apply to a 10-user MVP exactly as much as a 10,000-user product.  
The only thing that scales at MVP stage is the SLO target requirement.  
The failure hygiene does not.

---

## Relationship to Superpower

[Superpower](https://github.com/SuperpowerCorp/superpower) enforces the coding phase.  
Sparring enforces the specification phase.

```
Thin spec
    ↓  Sparring — spec quality layer
Contract-grade spec
    ↓  Superpower — process layer
Code that does what was specified
```

Use Sparring before Superpower.  
Sparring without Superpower still works.  
Superpower without Sparring misses the upstream problem.

---

## Contributing

The library is designed to grow. New personas, new starter kits, new worked examples.

If you run the pipeline on a real feature and catch something new — a class of bug
none of the personas caught — open a PR. Document the gap, the class of problem,
and the persona or rule that should catch it next time.

See [`PERSONA-TEMPLATE.md`](PERSONA-TEMPLATE.md) to build a new persona.

---

*Built on the principle that the most expensive bugs are written into specs, not code.*

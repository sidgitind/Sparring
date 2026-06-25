# PERSONA TEMPLATE
> Copy this file to create a new persona.
> Every section marked [FILL] must be completed before the persona is usable.
> Every section marked [CONFIGURE] is filled per project by the user.
> Do not delete any section — empty sections are flagged as errors by the pipeline.

---

## IDENTITY

**Name:** [FILL — descriptive name, not company name]
**Role in pipeline:** [FILL — what stage this persona reviews]
**Cognitive function:** [FILL — one sentence: what thinking this persona contributes]
**Veto authority:** [FILL — what this persona can BLOCK vs WARN]
**Address as:** @[handle]

---

## UNIVERSAL THINKING PATTERNS

> These never change between projects.
> They represent the core cognitive contribution of this role.

### What I always ask (regardless of project):
1. [FILL]
2. [FILL]
3. [FILL]
4. [FILL]
5. [FILL]

### What only I ask (unique cognitive contribution):
> Questions no other persona in the pipeline would ask.
- [FILL]
- [FILL]
- [FILL]

### Where I over-index (built-in bias — read this):
> This section is not optional. Every role has a bias.
> Knowing the bias is how the human gate stays calibrated.
[FILL — what this persona will over-flag, and when to push back on it]

---

## NON-NEGOTIABLES

> These are BLOCK conditions — the persona will not pass a spec that violates them.
> Distinguish from WARN conditions — things flagged but not blocking.

### BLOCK (spec cannot proceed without resolution):
- [FILL]
- [FILL]

### WARN (flagged for human decision, not blocking):
- [FILL]
- [FILL]

---

## HANDOFF PROTOCOL

### What I receive:
- Spec file (markdown)
- ARCHITECTURE.md (read before review)
- EDGE_CASES.md (read before review)
- Previous persona output (if sequential pipeline)
- SPARRING_FINDINGS.md (if running as single persona across sessions — read before review)

### What I produce:
- Structured review output (see OUTPUT TEMPLATE below)
- FINDINGS PAYLOAD (when running as single persona — appended to output, copy into SPARRING_FINDINGS.md)
- List of unresolved items for human decision

### What I do NOT do:
- [FILL — explicit exclusions prevent scope creep]
- I do not rewrite the spec. I flag gaps and return to the human.
- I do not make architectural decisions outside my domain.

---

## PROJECT CONFIG

> [CONFIGURE] — fill in per project before running this persona.
> If this section is empty: persona returns BLOCK with message
> "PROJECT CONFIG not configured — persona cannot run."

### Read before reviewing any spec:
- [ ] ARCHITECTURE.md
- [ ] EDGE_CASES.md
- [ ] AGENT_CONTEXT.md
- [ ] SPARRING_FINDINGS.md (if exists — prior persona findings)

### Project-specific inputs:
```
[CONFIGURE — project-specific thresholds, design system pointer,
stack details, or any other inputs this persona needs]
```

### Behavior switches:
```
STRICTNESS:   high / medium / low        [CONFIGURE — default: high]
OUTPUT:       detailed / summary          [CONFIGURE — default: summary]
GATE_MODE:    block / warn               [CONFIGURE — default: block]
```

---

## OUTPUT TEMPLATE

> The output header is mandatory on every run, regardless of mode.
> In summary mode: header + verdict + top 2 blockers + conditional items + findings payload.
> In detailed mode: header + full structured review + findings payload.
> The FINDINGS PAYLOAD is only appended when running as a single persona.
> Full pipeline runs in one session do not need it — findings carry over internally.

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SPARRING REVIEW
Spec:     [spec identifier provided by user]
Date:     [date of run]
Persona:  [persona name]
Mode:     summary | detailed
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[SECTION 1 NAME]
Status: PASS | FAIL | PARTIAL
Finding: [specific finding]
Evidence: [where in the spec — quote the exact line or absence]
Required action: [what must change]

[SECTION 2 NAME]
...

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
VERDICT: PASS | BLOCK | CONDITIONAL

Reason: [one sentence]
Blocking items: [numbered list — must be resolved before proceeding]
Conditional items: [numbered list — human decision required]
Passed items: [brief summary]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

── FINDINGS PAYLOAD ── (copy into SPARRING_FINDINGS.md if running single persona)

Spec:       [spec identifier]
Date:       [date of run]
Persona:    [persona name]
Verdict:    PASS | BLOCK | CONDITIONAL

Blocking items:
1. [item]
2. [item]

Flagged assumptions:
1. [assumption] — validated: yes / no / unknown
2. [assumption] — validated: yes / no / unknown

Prior findings read: yes (SPARRING_FINDINGS.md) | no (first persona in sequence)

── END FINDINGS PAYLOAD ──
```

---

## CONFLICT PROFILE

> Documents known conflicts with other personas in the pipeline.
> Used by the compatibility matrix.

### Conflicts with:
[FILL — which personas this one frequently disagrees with and why]

### Complements:
[FILL — which personas this one reinforces]

### Resolution when conflict occurs:
[FILL — how to resolve disagreements between this persona and others]

---

## CHANGE LOG
v1.0 — [date] — initial

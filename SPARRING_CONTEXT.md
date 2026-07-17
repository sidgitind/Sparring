# SPARRING_CONTEXT.md
> Project-specific context for the Sparring pipeline.
> Place this file in your project root alongside SPARRING_FINDINGS.md.
> Every persona reads this file before reviewing your spec.
> Fill in what your org already knows so personas don't flag it as missing.
> Leave sections blank if not applicable — do not delete them.
> Last updated: [date]

---

## HOW PERSONAS USE THIS FILE

Before flagging any gap as AMBIENT? (possibly org knowledge), each persona
checks this file for matching terms, definitions, or decisions.

- If a term or decision is found here: persona uses it silently. No flag.
- If a term is NOT found here: persona flags with AMBIENT? and tells you
  exactly which section to add it to. That is the handheld prompt.

This file does not replace ARCHITECTURE.md or EDGE_CASES.md.
It answers a different question: what does your org already know
that should not appear as a gap in a spec?

---

## SECTION 1 — TERMINOLOGY & ROLE DEFINITIONS

> Add any term that appears in your specs without being defined there.
> Format: Term: [your definition]
> The persona will use this definition and suppress the "undefined" flag.

[ADD YOUR TERMS HERE]

---

## SECTION 2 — DECISIONS ALREADY MADE

> Add decisions that are settled at the org, product, or architecture level
> and do not need to be re-justified in every spec.
> Format: Decision: [what was decided] | Reason: [one sentence] | Date: [when]
> The persona will not re-flag these as missing justification.

[ADD YOUR DECISIONS HERE]

---

## SECTION 3 — KNOWN AMBIENT KNOWLEDGE

> Add things your entire org knows that never appear in specs
> because everyone assumes everyone knows them.
> Format: [statement of ambient knowledge]
> The persona will treat these as confirmed context, not gaps.

[ADD YOUR AMBIENT KNOWLEDGE HERE]

---

## SECTION 4 — OUT OF SCOPE FOR THIS PROJECT

> List anything explicitly excluded from scope.
> The persona will not flag absence of these as gaps.
> Format: [what is excluded] | Reason: [one sentence]

[ADD YOUR OUT-OF-SCOPE ITEMS HERE]

---

## SECTION 5 — CUSTOMER CONTEXT

> If your org has named enterprise customers with known requirements,
> add them here so personas don't flag "customer undefined."
> Format: Customer: [name] | Key requirement: [one sentence] | Commitment ref: [ID if applicable]

[ADD YOUR CUSTOMER CONTEXT HERE]

---

## WHAT TO DO WHEN A PERSONA SAYS "NOT FOUND IN SPARRING_CONTEXT.md"

The persona will flag: "Term / decision not found in SPARRING_CONTEXT.md —
add to Section [N] if this is org knowledge."

Your options:
1. It IS org knowledge → add it to the relevant section above. On the next
   run, the persona will find it and suppress the flag.
2. It is NOT org knowledge → the gap is real. Add it to the spec.
3. You are unsure → treat it as a real gap. Document it. Org knowledge
   that lives only in people's heads is a risk, not a strength.

---

## CHANGE LOG
[date] — file created

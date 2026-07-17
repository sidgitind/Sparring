# SPARRING_SYNTHESIS — Pipeline Synthesis Step
**Version:** 1.1
**Role in pipeline:** Final step — runs after all personas complete
**Address as:** @Synthesis
**When to run:** After all selected personas have completed their reviews
**Input:** All persona Tier 1 snippets from the current session

---

## WHAT THIS STEP DOES

Four jobs. In order:
1. **Aggregate** — collect all Tier 1 snippets from the session
2. **Deduplicate** — merge findings that multiple personas flagged for the same mechanism; name the relationship, keep one finding, note which personas converged
3. **Rank** — surface highest surprise value first (NEW before AMBIENT?; convergent before single-persona)
4. **Index** — list every action as one line, grouped by owner

**What this step does NOT do:**
- Does not add new findings
- Does not override persona verdicts
- Does not restate findings from Tier 1 in Tier 2
- Does not make product decisions

---

## TAGS

```
⚡ NEW      — Not in SPARRING_CONTEXT.md. Likely a real gap. Act on it.
◎ AMBIENT? — May be org knowledge. Confirm before acting.
              Add to SPARRING_CONTEXT.md if it IS org knowledge.
              Treat as a real gap if it is NOT.
~ KNOWN    — Already in SPARRING_FINDINGS.md or flagged in the spec.
              Not new. Surfaces for completeness only.
```

---

## OUTPUT TEMPLATE

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SPARRING BRIEF — [Spec ID] — [Date]
Personas: [list]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

TIER 1 — VERDICT (30 seconds)
──────────────────────────────
Overall: BLOCK | CONDITIONAL | PASS
Reason:  [one sentence — the single most critical unresolved item]

[TAG]  [Finding — one line]  [BLOCK|WARN]  @[Persona]
[TAG]  [Finding — one line]  [BLOCK|WARN]  @[Persona]
[TAG]  [Finding — one line]  [BLOCK|WARN]  @[Persona]  ← max 3

Converges: [TAG] [Finding] — @PersonaA + @PersonaB — [one phrase: same gap, different angle]
           [or: none]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

TIER 2 — ACTION INDEX (one line per item)
──────────────────────────────────────────
PM          [TAG]  [action — one line]  → @[Persona]
PM          [TAG]  [action — one line]  → @[Persona]
Architect   [TAG]  [action — one line]  → @[Persona]
Architect   [TAG]  [action — one line]  → @[Persona]
Legal       [TAG]  [action — one line]  → @[Persona]
Admin       [TAG]  [action — one line]  → @[Persona]
AMBIENT?    [◎]   [confirm or add to SPARRING_CONTEXT.md Section N]  → @[Persona]

For detail on any item: see Tier 3 → [Persona] review → [section name]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

TIER 3 — FULL REVIEWS
──────────────────────
[Full persona reviews in pipeline order — read when challenging a finding]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Tier 1** — discovery. Max 3 findings. Read always.
**Tier 2** — action. One line per item. Read when acting. No restatement of Tier 1.
**Tier 3** — evidence. Full persona reviews. Read when challenging a finding.

Nothing is removed. Reading order is controlled.

---

## TIER 2 FORMATTING RULES

- One line per action. No sub-bullets. No explanation.
- If the reader needs reasoning: Tier 3.
- AMBIENT? items always last, always include the Section number.
- Owner labels: PM | Architect | Legal | Admin | AMBIENT?
- If an action has no clear single owner: split into two lines with two owners.

---

## HOW TO INVOKE

```
"@Synthesis — produce the SPARRING BRIEF from today's pipeline run on [spec ID]."
```

If running personas across sessions, paste SPARRING_FINDINGS.md content first.

---

## WORKED EXAMPLE

Input: AW-57850, three personas — @AmazonPM (BLOCK), @EnterprisePM (BLOCK), @GoogleArch (BLOCK).

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SPARRING BRIEF — AW-57850 — July 17, 2026
Personas: @AmazonPM · @EnterprisePM · @GoogleArch
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

TIER 1 — VERDICT (30 seconds)
──────────────────────────────
Overall: BLOCK
Reason:  External service failure behaviour undefined; no success metrics;
         breaking changes have no migration plan.

⚡ NEW  No failure behaviour for ANI, CRM, AI activation, transcription  [BLOCK]  @GoogleArch
⚡ NEW  Breaking changes to ANI/MasterID/recording — no migration plan  [BLOCK]  @EnterprisePM
⚡ NEW  No success metrics — ISOS year-1 has no measurable threshold  [BLOCK]  @AmazonPM

Converges: ⚡ NEW  Missing metrics / missing SLA — @AmazonPM + @EnterprisePM — same gap,
           product angle vs. commercial angle

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

TIER 2 — ACTION INDEX (one line per item)
──────────────────────────────────────────
PM          ⚡  Define success metrics for transfer continuity, AI activation, ACW delivery, conference persistence  → @AmazonPM
PM          ⚡  Resolve all QQ items — assign owner + date; priority: UC-T1 CaseID, UC-T13 elevation  → @AmazonPM
PM          ⚡  Add ISOS year-1 contractual SLA as explicit engineering constraint  → @EnterprisePM
PM          ⚡  Define migration plan + customer notice path for breaking ANI/MasterID/recording changes  → @EnterprisePM
Architect   ⚡  Define external service contracts: timeout, failure shape, state write for ANI/CRM/AI/transcription  → @GoogleArch
Architect   ⚡  Define conference persistence mechanism — owner, failure mode, active conference degradation  → @GoogleArch
Architect   ⚡  Define state write order for ownership transfers — confirmed-write or optimistic, not implicit  → @GoogleArch
Architect   ⚡  Define phased rollout plan with go/no-go criteria — ANI change is platform-wide  → @GoogleArch
Architect   ⚡  Resolve module ownership: ANI layer, MasterID writes, AI activation trigger, recording consolidation  → @GoogleArch
Legal       ⚡  Assign owner + deadline for PSAP recording jurisdiction compliance  → @EnterprisePM
Admin       ⚡  Document admin launch config requirements: transfer rules, cross-BU permissions, ACW config  → @EnterprisePM
AMBIENT?    ◎  Single "agent" persona — explicit decision or assumption? → Section 3 if intentional  → @AmazonPM
AMBIENT?    ◎  Services Australia description blank — add to Section 5 or fill in spec  → @AmazonPM
AMBIENT?    ◎  English-only transcription — validated with ISOS/GM? → Section 2 if confirmed  → @AmazonPM
AMBIENT?    ◎  "8 party" limit — platform constraint or product choice? → Section 2 if settled  → @AmazonPM
AMBIENT?    ◎  ROI baselines (handle time, CSAT delta) — exist internally? → Section 3 if yes  → @EnterprisePM
AMBIENT?    ◎  Standard 30-day change notice process — documented? → Section 3 if yes  → @EnterprisePM

For detail on any item: see Tier 3 → relevant persona review → relevant section

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

TIER 3 — FULL REVIEWS
──────────────────────
[Full persona reviews in pipeline order]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## CHANGE LOG
v1.0 — July 2026 — initial build
v1.1 — Tier 2 collapsed to one-line action index; removed narrative restatement;
        Tier 3 pointer added to Tier 2; output now discovery → action → evidence
        with no duplication between tiers

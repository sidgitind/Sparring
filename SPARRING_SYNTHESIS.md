# SPARRING_SYNTHESIS — Pipeline Synthesis Step
**Version:** 1.0
**Role in pipeline:** Final step — runs after all personas complete
**Cognitive function:** Aggregates Tier 1 snippets from all persona reviews,
deduplicates, groups by owner, and produces the three-tier output the PM reads first
**Address as:** @Synthesis
**When to run:** After all selected personas have completed their reviews
**Input:** All persona Tier 1 snippets from the current session

---

## WHAT THIS STEP DOES

Every persona produces a full review (Tier 3) and a short Tier 1 snippet.
This step reads all Tier 1 snippets, does four things, and produces
the SPARRING BRIEF that a time-pressed PM can read in 30 seconds.

**The four jobs:**
1. Aggregate — collect all Tier 1 snippets from the session
2. Deduplicate — merge findings that multiple personas flagged for
   the same mechanism (cross-reference convention: name the relationship,
   keep one finding, note which personas converged)
3. Rank — surface the two findings with the highest surprise value first
   (NEW findings before AMBIENT? findings; higher confidence before lower)
4. Group — sort all blocking items by who needs to act, not by which
   persona flagged them

**What this step does NOT do:**
- It does not add new findings. It only synthesises what the personas found.
- It does not override persona verdicts.
- It does not rewrite persona output.
- It does not make product decisions.

---

## SURPRISE VALUE RANKING

When ranking findings for Tier 1, use this priority order:

1. `⚡ NEW` — finding that has no match in SPARRING_CONTEXT.md and
   is unlikely to be org knowledge. High surprise value. Always surfaces first.

2. `⚡ NEW | CRITICAL` — NEW finding that also has a BLOCK verdict
   from two or more personas (convergent block). Highest priority.

3. `◎ AMBIENT?` — finding the persona suspects may be org knowledge
   but could not confirm from SPARRING_CONTEXT.md. Medium surprise value.
   Surfaces after NEW items.

4. `~ KNOWN` — finding already documented in SPARRING_FINDINGS.md
   from a prior session, or explicitly marked as a known gap in the spec.
   Lowest surprise value. Surfaces last in Tier 1.

**The rule for Tier 1:** Show maximum 3 findings. If there are more than 3,
show the top 2 NEW findings and the top 1 AMBIENT? finding.
The rest appear in Tier 2 (Action Queue). Nothing is lost — it is deferred
to the section the reader goes to when acting, not when discovering.

---

## TAG DEFINITIONS (apply consistently across all tiers)

```
⚡ NEW       — Not in SPARRING_CONTEXT.md. Likely a real gap.
               Persona had no org context to suppress this finding.
               High probability of being actionable.

◎ AMBIENT?  — Not confirmed in SPARRING_CONTEXT.md but may be org knowledge.
               Persona is flagging it because it could not verify.
               Add to SPARRING_CONTEXT.md if it IS org knowledge.
               Treat as a real gap if it is NOT.

~ KNOWN     — Already in SPARRING_FINDINGS.md from a prior session,
               or explicitly marked as a known gap in the spec itself.
               Not a new discovery. Surfaces for completeness only.
```

---

## OUTPUT TEMPLATE

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SPARRING BRIEF
Spec:       [spec identifier]
Date:       [date]
Personas:   [list of personas that ran]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

╔══════════════════════════════════════╗
║  TIER 1 — READ THIS. TAKES 30 SEC.  ║
╚══════════════════════════════════════╝

Overall verdict: BLOCK | CONDITIONAL | PASS
[One sentence: why, and what the most critical unresolved item is]

Top findings (max 3 — highest surprise value first):

[TAG] [Finding in one line] [BLOCK | WARN] — @[Persona]
[TAG] [Finding in one line] [BLOCK | WARN] — @[Persona]
[TAG] [Finding in one line] [BLOCK | WARN] — @[Persona]

Convergent findings (multiple personas, same mechanism):
[If none: "None — all findings are persona-unique"]
[If any: "[TAG] [Finding] — Converges: @[PersonaA] + @[PersonaB] — same gap, different angle"]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

╔══════════════════════════════════════╗
║  TIER 2 — ACTION QUEUE               ║
║  Read when acting. Grouped by owner. ║
╚══════════════════════════════════════╝

[Group by who needs to act — not by which persona flagged it]
[Each item: tag + finding + recommendation + confidence + persona source]

── PM / PRODUCT OWNER ──────────────────
[TAG] [Finding]
  Recommendation: [what to add or resolve in the spec]
  Confidence: high | medium | low
  Source: @[Persona]
  [If AMBIENT?: "Add to SPARRING_CONTEXT.md Section [N] if org knowledge"]

── ARCHITECT / TECH LEAD ───────────────
[TAG] [Finding]
  Recommendation: [what to define architecturally]
  Confidence: high | medium | low
  Source: @[Persona]

── LEGAL / COMPLIANCE ──────────────────
[TAG] [Finding]
  Recommendation: [what needs legal review]
  Confidence: high | medium | low
  Source: @[Persona]

── ADMIN / OPERATIONS ──────────────────
[TAG] [Finding]
  Recommendation: [what needs admin config or ops decision]
  Confidence: high | medium | low
  Source: @[Persona]

── DEFERRED (AMBIENT? — confirm before acting) ─
[TAG] [Finding]
  Why deferred: [persona could not find this in SPARRING_CONTEXT.md]
  If org knowledge: add to SPARRING_CONTEXT.md Section [N]
  If not org knowledge: treat as PM | Architect | Legal item above
  Source: @[Persona]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

╔══════════════════════════════════════╗
║  TIER 3 — FULL REVIEWS               ║
║  Read when challenging a finding.    ║
╚══════════════════════════════════════╝

[Full persona reviews appear below in pipeline order]
[Each review is unchanged from the persona's original output]
[This section exists so the reader can trace any Tier 1 or Tier 2
 finding back to its full evidence and reasoning]

[Paste or reference full persona outputs here]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## HOW TO INVOKE

After all personas have run in a session, say:

> "Run @Synthesis on the findings from this session."

Or at the end of a full pipeline run:

> "@Synthesis — produce the SPARRING BRIEF from today's pipeline run on [spec ID]."

If running personas across sessions, paste the SPARRING_FINDINGS.md content
before invoking @Synthesis so it has the full finding set.

---

## WORKED EXAMPLE (Tier 1 only)

Input: three personas ran — Amazon PM (BLOCK), Enterprise B2B (BLOCK), Google Architect (BLOCK).

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SPARRING BRIEF
Spec:       AW-57850 — Agent Transfers, Consults, and Conferences
Date:       July 16, 2026
Personas:   @AmazonPM · @EnterprisePM · @GoogleArch
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

╔══════════════════════════════════════╗
║  TIER 1 — READ THIS. TAKES 30 SEC.  ║
╚══════════════════════════════════════╝

Overall verdict: BLOCK
No success metrics exist. External service failure behaviour is undefined.
Five unresolved QQ items in use cases will be decided by engineering.

⚡ NEW | CRITICAL  No external service failure behaviour defined for
                   ANI propagation, CRM fetch, AI activation, transcription
                   [BLOCK] — @GoogleArch

⚡ NEW             No success metrics — ISOS year-1 contract has no
                   measurable delivery threshold in the spec [BLOCK]
                   — @AmazonPM + @EnterprisePM (convergent)

◎ AMBIENT?         "Agent" undefined in spec — may be org knowledge
                   (contact centre agent, not AI agent) [WARN]
                   — @AmazonPM | Add to SPARRING_CONTEXT.md Section 1

Convergent findings:
⚡ NEW  Missing success metrics — @AmazonPM (product angle: no definition
        of done) + @EnterprisePM (renewal angle: champion cannot quantify
        improvement at renewal). Same gap, different rationale. Both hold.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## CHANGE LOG
v1.0 — July 2026 — initial build

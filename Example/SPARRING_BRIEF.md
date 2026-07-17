# SPARRING_BRIEF — AI Ticket Summary (Flowdesk)
> This is the @Synthesis output for the worked example.
> It shows what the three-tier output looks like after a full pipeline run.
> The full persona reviews (Tier 3) are in Delta_Spec.md — every finding
> traced to its source, its class of problem, and its cross-references.
> This file is Tier 1 + Tier 2. Read this first. Go to Delta_Spec.md
> only when you want to understand or challenge a specific finding.

---

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SPARRING BRIEF — Flowdesk: AI Ticket Summary — [pipeline run date]
Personas: @AmazonPM · @GooglePM · @GoogleArch · @UIDesigner · @QAFunctional · @QANFQ
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

TIER 1 — VERDICT (30 seconds)
──────────────────────────────
Overall: BLOCK
Reason:  No customer definition at the point of highest stakes; no
         counter-metric; summarization service has no contract.

⚡ NEW  No counter-metric — time saved could improve while decision quality silently decays  [BLOCK]  @GooglePM
⚡ NEW  Summarization service call has no timeout, failure shape, or retry defined  [BLOCK]  @GoogleArch
⚡ NEW  Customer undefined at consequence level — routine ticket ≠ escalation ticket  [BLOCK]  @AmazonPM

Converges: ⚡ NEW  Completeness signal — @AmazonPM (accountability gap) + @QAFunctional
           (structural heuristics, not fake confidence score) — same gap, principle vs.
           testable mechanism
           ⚡ NEW  Service failure — @GoogleArch (contract) + @UIDesigner (failure state)
           + @QANFQ (pre-flight check) — same event, three lenses, all required

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

TIER 2 — ACTION INDEX
──────────────────────
PM          ⚡  Define customer at consequence level — routine engineer vs. escalation engineer  → @AmazonPM
PM          ⚡  Add accountability model — product surfaces risk, engineer owns decision  → @AmazonPM
PM          ⚡  Add counter-metric: post-resolution escalation rate on summary-viewed tickets  → @GooglePM
PM          ⚡  Add instrumentation: summary_viewed, full_thread_expanded, action_taken events  → @GooglePM
PM          ⚡  Define phased rollout by complexity tier + rollback trigger on expand rate  → @GooglePM
Architect   ⚡  Define summarization service contract: 4000ms timeout, failure shape, retry-once  → @GoogleArch
Architect   ⚡  Place heuristic computation synchronously at generation time — not async job  → @GoogleArch
Architect   ⚡  Define stale-state handling — mark summary on new message arrival before regen  → @GoogleArch
UI Designer ⚡  Define all four panel states: loading, success, stale, failure  → @UIDesigner
UI Designer ⚡  Risk indicator must be a labeled banner above summary — not icon or color only  → @UIDesigner
UI Designer ⚡  Add jump-to-source link in risk banner — engineer must reach flagged message in < 3 clicks  → @UIDesigner
QA          ⚡  Replace "accuracy threshold" with structural heuristics — testable per-ticket, not aggregate only  → @QAFunctional
QA          ⚡  Define failure negative path: ticket renders fully with no summary panel, no blocked interaction  → @QAFunctional
QA          ⚡  Name regression surface: ticket detail view, thread component, ticket-state logic  → @QAFunctional
QA NFQ      ⚡  Add p95 < 1500ms SLO on summary generation — rolling 7-day window  → @QANFQ
QA NFQ      ⚡  Add operational monitoring: error rate + latency alerts before Phase 1 launch  → @QANFQ
QA NFQ      ⚡  Debounce regeneration — one fire per window, last completed wins, no interleaved state  → @QANFQ

For detail on any item: Delta_Spec.md → relevant persona section → finding number

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

TIER 3 — FULL REVIEWS
──────────────────────
See Delta_Spec.md — every finding traced to persona, class of problem
prevented, and cross-references where multiple personas touch the
same mechanism.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## What this output tells you

**Tier 1** gives you the three highest-surprise findings and names the convergent ones.
Two convergences appear — completeness signal and service failure — because three
personas independently touched the same mechanisms. The convergences are the most
important findings: they confirm the gap is real, not a single persona's over-index.

**Tier 2** is a 17-line action checklist. Every item is one line. Every item has an owner
and a source. You can assign these directly without reading anything else.

**The AMBIENT? section is empty** in this example — because SPARRING_CONTEXT.md
was populated after the first run (see `/Example/ProjectConfig/SPARRING_CONTEXT.md`).
On your first run, before the context file exists, several of these items would have
appeared as `◎ AMBIENT?` with a pointer to which section to add them to. That is
the mechanism that makes subsequent runs cleaner.

---

## The one thing this output cannot show you

The delta between a spec that passes and one that blocks.

The thin spec (Input_Spec.md) had four acceptance criteria.
All four were correct. The agent would have built them correctly.
The SPARRING BRIEF shows you the 17 things that were also needed —
none of which were wrong to omit from a normal PM's first draft,
and all of which would have been discovered later, at higher cost.

That is the point of running the pipeline before the build, not after.

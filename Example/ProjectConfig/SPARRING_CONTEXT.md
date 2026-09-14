# SPARRING_CONTEXT.md — Flowdesk
> Project-specific context for the Sparring pipeline.
> This is the EXAMPLE version — pre-populated to show what a
> filled-in context file looks like after the first pipeline run.
> Copy the template from the library root and replace this content
> with your own project's context.
> Last updated: after full pipeline run on AI Ticket Summary feature

---

## HOW PERSONAS USE THIS FILE

Before flagging any gap as AMBIENT? (possibly org knowledge), each persona
checks this file for matching terms, definitions, or decisions.

- If a term or decision is found here: persona uses it silently. No flag.
- If a term is NOT found here: persona flags with AMBIENT? and tells you
  exactly which section to add it to. That is the handheld prompt.

---

## SECTION 1 — TERMINOLOGY & ROLE DEFINITIONS

Support engineer: a Flowdesk user who opens, reads, and takes action on
customer support tickets. Not a software engineer. The primary user of
every feature in this product.

Team lead: a support engineer with additional permissions for bulk actions
and queue management. Not a manager — still works tickets daily.

Thread: the full message history attached to a ticket, including all
participant messages, status changes, and system events.

Complexity tier: a classification of tickets by structural properties —
message count, time span, participant count, detected status changes.
Low-complexity = fewer than 5 messages, single participant, no status change.
All other tickets = standard complexity.

---

## SECTION 2 — DECISIONS ALREADY MADE

Decision: JWT HS256 algorithm for auth | Reason: locked at project start |
Date: project inception

Decision: pg library for database client, not Prisma or Sequelize |
Reason: locked at project start | Date: project inception

Decision: Express framework, not Fastify or Koa |
Reason: locked at project start | Date: project inception

Decision: Summarization service timeout = 4000ms |
Reason: validated as achievable at p95 under production load with
selected provider | Date: post-pipeline run, AI Ticket Summary feature

Decision: Structural completeness heuristics (not a model confidence score)
as the completeness signal | Reason: model confidence scores measure
token-level certainty, not content completeness — rejected as false
signal | Date: AI Ticket Summary pipeline run, QA Functional Finding 14

Decision: Heuristic computation is synchronous and in-process at summary
generation time | Reason: cheap — negligible latency vs generation call;
avoids race condition with async job | Date: AI Ticket Summary pipeline
run, Architect Finding 9

Decision: Phase 1 rollout = low-complexity tickets only |
Reason: trust decision, not performance — establish confidence on
low-stakes tickets before applying to consequential ones |
Date: AI Ticket Summary pipeline run, Google PM Finding 7

---

## SECTION 3 — KNOWN AMBIENT KNOWLEDGE

All engineers referenced in specs are support engineers using Flowdesk —
not software engineers, not AI agents.

Ticket detail view is the highest-traffic page in the product. Any new
panel or component inserted into this view is a high-regression-risk change
and must include explicit regression testing.

The product has paying customers on annual contracts. SLOs on critical
user paths are not optional — see QA NFQ non-negotiable floor.

Support teams work high-volume queues. Any feature that slows down the
ticket detail view — even by 200ms — has a compounding effect across
dozens of tickets per engineer per day.

The full thread is always the source of truth. Any AI-generated content
is supplementary. No feature should make the full thread harder to access.

---

## SECTION 4 — OUT OF SCOPE FOR THIS PROJECT

Summarization of attachments or screenshots | Reason: requires multimodal
model capability not currently available in this stack

Multi-language summary generation | Reason: deferred to future phase —
current customer base is English-language only

Summary editing by the engineer | Reason: creates accountability ambiguity —
if an engineer edits a summary, who is responsible for its accuracy?
Deferred until accountability model is established

Semantic analysis of thread content for completeness assessment |
Reason: structural heuristics only — semantic analysis is a separate
capability investment. See QA Functional Finding 14.

Per-engineer feature preferences or opt-out controls | Reason: not yet
built — see Architecture.md "What Does Not Exist Yet"

Audit log beyond standard application logging | Reason: not yet built —
legal review of action_taken event in instrumentation spec is an open
item before Phase 2

---

## SECTION 5 — CUSTOMER CONTEXT

No named enterprise customers with specific contractual requirements
at this stage — Flowdesk is a self-serve B2B SaaS product.
All customers are on standard annual contracts.

Primary buyer persona: Head of Support / VP Customer Success at a
mid-market SaaS company (100–2000 employees). Buys Flowdesk to
reduce support ticket resolution time and improve engineer throughput.
Renewal metric: ticket resolution time, engineer satisfaction score.

---

## WHAT TO DO WHEN A PERSONA SAYS "NOT FOUND IN SPARRING_CONTEXT.md"

Your options:
1. It IS org knowledge → add it to the relevant section above.
2. It is NOT org knowledge → the gap is real. Add it to the spec.
3. You are unsure → treat it as a real gap. Document it. Org knowledge
   that lives only in people's heads is a risk, not a strength.

---

## CHANGE LOG
Initial population — after AI Ticket Summary full pipeline run

# SPEC — Feature: AI Ticket Summary (v1 — pre-persona review)

> This is the THIN spec — what a mid-level PM writes under normal
> sprint conditions. It is not careless writing. It is realistic.
> The worked example shows what the personas catch and why it matters.

---

## Background

Support engineers have flagged context-switching as a major
time sink. Tickets often contain long thread histories. Engineers
spend significant time reading through threads before they can
respond or take action.

We want to reduce time-to-first-action by surfacing an AI-generated
summary at the top of every ticket.

---

## What We Are Building

An AI summary panel that appears at the top of every ticket in
Flowdesk, generated automatically when the ticket is opened.

---

## Acceptance Criteria

1. A summary is displayed automatically when an engineer opens a ticket;
   generation is triggered only if no summary exists for that ticket or
   the existing summary is stale (per AC4)
2. Summary accuracy meets a minimum threshold of 90% against a
   human-reviewed test set, where a summary is rated "accurate" if it
   (a) contains no factual errors relative to the ticket thread, and
   (b) states the ticket's current status and its most recent unresolved
   question or action item, if any. Accuracy is the percentage of
   test-set summaries rated accurate by at least 2 independent human
   reviewers, with a third reviewer breaking any disagreement
3. Engineer can mark a summary as helpful or not helpful
   (thumbs up / thumbs down)
4. Summary regenerates on the next ticket-open event after new messages
   have been added to the thread since the last summary was generated
   (regeneration is not live while a ticket is already open — an engineer
   viewing a ticket when new messages arrive sees the update on next open,
   not immediately)

---

## Out of Scope

- Summarization of attachments or screenshots
- Multi-language summary generation
- Summary editing by the engineer

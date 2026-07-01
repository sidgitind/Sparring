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

1. Summary generates automatically when an engineer opens a ticket
2. Summary accuracy meets a minimum threshold of 90% against a
   human-reviewed test set
3. Engineer can mark a summary as helpful or not helpful
   (thumbs up / thumbs down)
4. Summary regenerates if new messages are added to the thread
   after the last summary was generated

---

## Out of Scope

- Summarization of attachments or screenshots
- Multi-language summary generation
- Summary editing by the engineer

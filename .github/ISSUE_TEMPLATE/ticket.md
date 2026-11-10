---
name: Ticket
about: One ticket = one PR. Filled in by the Analyst in the Requirements phase.
title: "TXX <short summary>"
labels: []
---

**Sprint N · <sprint name> (<dates>)** · Stories: [US-0X](https://claude.ai/artifact/17UhCg3NV7bjarzmBECKJB#us-0x) · Depends on: #<issue>

## Goal
<!-- One or two sentences: what this ticket delivers and why. -->

## User story
Full stories and criteria: [plan · User stories](https://claude.ai/artifact/17UhCg3NV7bjarzmBECKJB#stories)

**[US-0X](https://claude.ai/artifact/17UhCg3NV7bjarzmBECKJB#us-0x) <title>** — As a user, …

## Acceptance criteria
<!-- Each one must be testable. The Reviewer checks the PR against this list. -->
- [ ] …

## Tests
<!-- Which tests prove the criteria: pytest, Vitest, Playwright. -->

## Out of scope
- …

## Definition of done
- [ ] Requirements — criteria testable, labelled `ready`
- [ ] Design — Planner's plan comment on this issue (fits one PR, < 300 changed lines)
- [ ] Build — PR from `feat/<issue>-<slug>` into `sprint/N`, linked to this issue
- [ ] Test — CI green
- [ ] Review — no blocking findings (max 3 fix rounds)

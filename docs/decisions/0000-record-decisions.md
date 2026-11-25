# ADR-0000: Record every decision as a short Markdown record

**Status:** Accepted · **Date:** 2026-11-25 · **Decided by:** owner (project brief: "document every good decision in MD files")

## Context

The project will run for a long time in small steps, with one person and AI assistants doing the work. Early choices (message bus, firmware, radios, model sizes) are easy to forget and expensive to rediscover, and a new working session starts with no memory of why things are the way they are.

## Decision

Every decision that shapes the stack, the hardware or the way of working is written as a short Markdown record in `docs/decisions/`:

- One file per decision, numbered in order: `NNNN-short-title.md`.
- Each record states the context, the decision, the alternatives considered, the consequences, and the condition under which it should be revisited.
- Each record is listed in [README.md](README.md) in this folder.
- Records are not rewritten when a decision changes. A new record supersedes the old one, and the old record's status line is updated to point to it.

Statuses:

| Status | Meaning |
|---|---|
| Proposed | Suggested, with reasons, but not yet confirmed by the owner |
| Accepted | Confirmed by the owner |
| Superseded by ADR-NNNN | Replaced by a later record |
| Dropped | Considered and rejected; kept so the question is not reopened by accident |

## Consequences

- Anyone, human or AI, can reconstruct why the system looks the way it does from the index alone.
- The cost is about one page per decision.
- Research that backs a decision lives in `reports/` and `research_notes/`. Records link to it and do not repeat it.

## Revisit when

The number of records makes the index hard to scan. Group it by area before changing the practice.

# ADR-0001: Record architecture decisions

**Status:** accepted · **Date:** {{INSTALL_DATE}}

## Context

Decisions that shape this project's architecture are made in conversations — human or agent —
whose memory is transient. Without a durable record, the same questions get re-litigated and
the reasoning behind the current design is lost.

## Decision

We record every significant architecture decision as an ADR in `docs/adr/`, numbered
sequentially, using `docs/adr/template.md`. Agents working in this repository are required to
write an ADR whenever they make or change a decision that another session would need to know
(see AGENTS.md §11, Durable Memory).

## Consequences

- The reasoning behind the design survives sessions, agents, and reboots.
- Superseded decisions are marked as such, never deleted — the history is the value.
- Feature-level decisions still belong in Spec Kit artifacts; ADRs are for decisions that
  outlive a single feature.

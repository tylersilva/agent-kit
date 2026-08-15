---
name: brief
description: Produce a technical brief for delegating a task (AGENTS.md §8)
argument-hint: the objective to brief
---

Produce a technical brief for the objective named in the request.

Follow the ritual in `docs/agents/rituals/brief.md`: fill every section of the AGENTS.md §8
format from the current context. Requirements must be verifiable; VERIFICATION must name actual
commands; DO NOT must state boundaries explicitly. Write `UNKNOWN — <what would resolve it>`
for anything not known rather than guessing, and flag any UNKNOWN that should go to the
researcher role before dispatch.

Output the completed brief ready to hand to the implementer role. Do not begin implementing —
the human decides when to dispatch.

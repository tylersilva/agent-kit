---
description: Fresh-context audit gate for the artifact just produced (agent-kit)
---

# Audit Gate (agent-kit)

The preceding stage produced a document artifact — a spec or plan under `.specify/`. Before
the next lifecycle stage consumes it, it must pass a fresh-context audit: **its author cannot
close its own quality gate** (AGENTS.md §10), and any self-assessed checklist it contains is a
claim under audit, not a result.

1. Identify the artifact just produced or updated (the current feature's `spec.md` or
   `plan.md`).
2. Dispatch the reviewer role in a **fresh context** — the native reviewer subagent where the
   host supports one, otherwise a new focused session given `docs/agents/reviewer.md`. Provide
   only the artifact path and the sources it names — nothing from this conversation.
3. Use the document-artifact charge from `docs/agents/rituals/audit.md`: verify the evidence
   and sources; re-verify every self-assessed checklist item against the artifact's actual
   text (closed only by cited evidence); check traceability, bidirectional consistency, and
   closeable criteria (AGENTS.md §13); check internal consistency across sections.
4. Relay the VERDICT unsoftened, and record it in `docs/reviews/` (dated file).
5. **A Defective verdict blocks the next stage.** Fix the findings and re-run this gate before
   proceeding — do not carry a defective artifact forward.

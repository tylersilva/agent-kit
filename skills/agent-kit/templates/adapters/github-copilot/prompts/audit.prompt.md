---
name: audit
description: Run an independent fresh-context audit of completed work (AGENTS.md §10)
agent: reviewer
argument-hint: what to audit (feature, branch, or claim)
---

Run the independent audit ritual from `docs/agents/rituals/audit.md` on the target named in the
request.

Follow the reviewer role card (`docs/agents/reviewer.md`): assume the previous implementation
may be wrong, independently reproduce the reported behavior, re-run the tests and the smoke
test yourself, and hunt for hidden assumptions, edge cases, and missing tests.

Report VERDICT / EVIDENCE / FINDINGS / MISSING TESTS, unsoftened, and append significant
findings to `docs/reviews/` (dated file), stating the path.

# Ritual: Independent Audit

Use when completed work needs independent verification, or whenever something feels wrong.
Never audit inside the conversation that produced the work — questionable context contaminates.

## Procedure

1. Identify the target: the feature, change, branch, or claim to audit.
2. Choose the tier (AGENTS.md §16): a routine post-work audit runs on the reviewer's default
   (mid) model; escalate to a **strong** model — per-dispatch override where the host supports
   it — when the audit exists because something already feels wrong, the change is
   architectural or security-adjacent, or a prior audit is disputed. If a strong-tier audit is
   itself disputed, or the stakes are extreme (security-critical, pre-release), escalate once
   more to the **apex** frontier-class model.
3. Open a **fresh context** (a new session, or dispatch the reviewer role where the host
   supports subagents). Provide only: the target, the relevant brief / Spec Kit artifacts, and
   the reviewer role card (`docs/agents/reviewer.md`). Not the implementation conversation.
4. Give the auditor this charge, filling in the target:

```text
You are auditing: <TARGET>

Assume the previous implementation may be wrong.
Independently reproduce the reported behavior.
Inspect the implementation from a different point of view.
Look for hidden assumptions, edge cases, missing tests, race conditions, and incorrect claims.
Do not rely on the previous agent's assertion that the feature works.

Re-run the tests and the smoke test yourself (AGENTS.md §3, §5). Report using the reviewer
role card's format: VERDICT / EVIDENCE / FINDINGS / MISSING TESTS.
```

5. Relay the verdict **unsoftened**. A "defective" verdict is a result, not an embarrassment.
6. Persist significant findings to `docs/reviews/` and report the path.

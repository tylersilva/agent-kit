# Ritual: Independent Audit

Use before the next lifecycle stage consumes any meaningful artifact — implementation code,
and equally specs, plans, constitutions, and ADRs (AGENTS.md §10's definition) — and whenever
something feels wrong. Never audit inside the conversation that produced the work —
questionable context contaminates, and an author cannot close its own quality gate.

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
4. Give the auditor the matching charge, filling in the target.

**For code / implementation work:**

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

**For a document artifact (spec, plan, constitution, ADR):**

```text
You are auditing the document artifact: <TARGET>

Assume the artifact may be wrong. Its author cannot close its own quality gate (AGENTS.md
§10): treat any self-assessed checklist — including an all-pass one — as a claim under audit,
not a starting point.

Verify, against the artifact's actual text:
1. EVIDENCE — the sources it derives from support what it asserts.
2. SELF-ASSESSMENT — re-verify every checklist item; an item is closed only by cited evidence
   (a quote or file reference), never by assertion.
3. TRACEABILITY — every MUST/SHOULD requirement traces to a source or is declared in
   Assumptions; undeclared invention is a defect (AGENTS.md §13).
4. BIDIRECTIONAL CONSISTENCY — every acceptance-scenario behavior has a functional
   requirement, and every requirement has a scenario or criterion that would catch its
   violation (AGENTS.md §13).
5. CLOSEABLE CRITERIA — every success criterion is measurable by this artifact's feature
   alone; cross-feature needs are declared dependencies (AGENTS.md §13).
6. INTERNAL CONSISTENCY — stories vs. requirements vs. criteria vs. edge cases agree.

Report using the reviewer role card's format: VERDICT / EVIDENCE / FINDINGS / MISSING TESTS
(for a document, MISSING TESTS = the checks or criteria the artifact should contain and does
not).
```

5. Relay the verdict **unsoftened**. A "defective" verdict is a result, not an embarrassment.
6. Persist significant findings to `docs/reviews/` and report the path.

# The agent-kit Framework

The portable core of a persistent multi-agent development system. The full architecture —
including the host-side runtime it belongs to — is in [docs/architecture.md](docs/architecture.md);
this page is what travels into every project.

## Ownership boundaries

The framework's central rule is to never build the same machinery twice:

| Concern | Owner |
|---|---|
| Behavior: delegation, verification, memory, gates | **AGENTS.md** (generated per project) |
| Specialist roles | **Role cards** in `docs/agents/` (+ native renderings per host) |
| Feature workflow: specify → clarify → plan → tasks → analyze → implement | **Spec Kit** (`.specify/`) |
| Executing the work | **The agent host** (Claude Code, Copilot, …) |
| Durable record: history, branches, PRs, review | **Git / GitHub** |

## The roster

The **main session is the Brain**: it plans, produces technical briefs, delegates, and verifies
— it never blindly accepts a worker's claims. Five specialist roles support it:

| Role | Tier | One-line charter |
|---|---|---|
| implementer | mid ▾ cheap for mechanical | One brief, one branch, one objective; evidence or it didn't happen |
| researcher | mid | Replace guessing with labeled findings: fact / inference / unknown / recommendation |
| reviewer | mid ▸ strong on suspicion | Fresh-context auditor; assumes the implementation may be wrong |
| security | strong | Vulnerabilities, secrets, dependency risk; never merges, never touches prod |
| devops | mid ▾ cheap for triage | CI/CD and infra config as reviewable diffs; nothing production-affecting unaided |

Later specialists (GitHub operator, dependency monitor, repo health, …) are documented as
recipes in the skill's `references/extensions.md` — added only when recurring workload proves
the need.

## Lifecycle of a feature

```text
Request → Brain → technical brief → dedicated branch/worktree → implementer
        → Spec Kit flow (specify → … → implement) → tests → smoke test
        → fresh-context audit (reviewer) [+ security when relevant]
        → PR → human approval → merge
```

## The portable principles

1. One interface, many specialists — you normally talk to the Brain.
2. Spec Kit owns the feature workflow; never recreate it.
3. Git is mandatory; the repository is the durable memory.
4. One task = one focused session; fresh contexts are a feature.
5. Worktrees/branches isolate parallel work — never share a working tree casually.
6. Context is a budget: workers get a brief, never a transcript.
7. Never trust an unsupported success claim; evidence or it didn't happen.
8. When something feels wrong, audit from a fresh context.
9. Research instead of guessing; label findings by evidence tier.
10. Smoke-test applications end to end; unit tests passing ≠ done.
11. Route models by job, cheapest-first: strong for high-leverage reasoning and security, mid
    for bounded execution, cheap for mechanical work — and when a dispatch fails or is
    disputed, escalate one tier up with the same brief, never retry sideways.
12. Human approval stays explicit for consequential operations.
13. Event-driven beats constant polling: agents wake on a trigger, work, and stop — never wake
    a model to ask whether anything needs doing.

## Host support

| Host | Gets |
|---|---|
| Claude Code | AGENTS.md (via CLAUDE.md import) + real subagents + **enforced** permission gates + `/brief`, `/audit` commands |
| GitHub Copilot | AGENTS.md (native) + custom agents (`.github/agents/`) + prompt files (VS Code-family surfaces) |
| Codex, Cursor, Gemini CLI, others | AGENTS.md (native) + role cards adopted in fresh sessions |

On hosts without enforcement, the approval gates are binding conventions written into
AGENTS.md — weaker teeth, same rules.

## Deliberately out of scope

The runtime side of the source architecture — supervisor, session persistence (tmux/systemd),
private networking, usage-aware scheduling, event routing, web UI — is host infrastructure, not
project structure. It lives in [docs/architecture.md](docs/architecture.md) for when that layer
gets built.

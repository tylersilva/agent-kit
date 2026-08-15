# agent-kit

A repeatable, agent-agnostic multi-agent working structure for any project — distributed as a
single [Agent Skill](https://agentskills.io) via the GitHub CLI.

One skill, invoked in any repository, installs the portable parts of a persistent multi-agent
system: an **AGENTS.md** operating contract (the "Brain" behavior: plan, delegate, verify),
five specialist **role cards** (implementer, researcher, reviewer, security, devops),
**verification rituals** (technical briefs, fresh-context audits), **approval gates**,
**durable-memory scaffolding** (ADRs, runbooks), and **Spec Kit** initialization for the
feature workflow. It auto-detects what it can from your repo and interviews you for the rest.

Works the same whether the project is driven by Claude Code, GitHub Copilot, Codex, Cursor,
Gemini CLI, or any other AGENTS.md-aware agent. Claude Code and GitHub Copilot additionally get
native renderings (real subagents, enforced permission gates, slash commands / custom agents).
See [FRAMEWORK.md](FRAMEWORK.md) for the principles and
[docs/architecture.md](docs/architecture.md) for the full source architecture.

## Install (once per machine)

Requires GitHub CLI ≥ 2.97 (`gh skill` is in preview).

```bash
# for Claude Code
gh skill install tylersilva/agent-kit agent-kit --agent claude-code --scope user

# for GitHub Copilot
gh skill install tylersilva/agent-kit agent-kit --agent github-copilot --scope user
```

(`--scope project` installs into just the current repo instead.)

## Use (per project)

In any repository, tell your agent:

> set up agent-kit in this project

The skill runs preflight checks, detects your stack and commands, asks a short batched
interview (purpose, smoke test, autonomy level, boundaries, target hosts, Spec Kit), then
generates everything and reports what it created. Re-running it later enters **update mode**:
it refreshes only its managed blocks and never touches content you wrote.

## Update the kit

```bash
gh skill update --all
```

## Repository layout

```
skills/agent-kit/          the installable skill (SKILL.md + templates + references)
FRAMEWORK.md               the distilled framework principles
docs/architecture.md       the full source architecture document
```

## Publishing (maintainer)

```bash
gh skill publish --dry-run     # validate
gh skill publish --tag vX.Y.Z  # release
```

## License

MIT

# Host Adapter Mapping

What each selected host gets, beyond the universal core. Facts below were verified August 2026;
where a host's format is marked *verify*, check its current docs before generating.

## Universal core (every host, always)

| Template | Destination in project |
|---|---|
| `core/AGENTS.md` | `AGENTS.md` |
| `core/roles/*.md` | `docs/agents/<role>.md` |
| `core/rituals/*.md` | `docs/agents/rituals/<ritual>.md` |
| `core/docs/adr-0001…` / `adr-template.md` | `docs/adr/0001-record-architecture-decisions.md`, `docs/adr/template.md` |
| `core/docs/runbooks-README.md` / `runbook-smoke-test.md` | `docs/runbooks/README.md`, `docs/runbooks/smoke-test.md` |
| `core/agent-kit.json` | `.agent-kit.json` |

AGENTS.md is read natively by GitHub Copilot (cloud agent, CLI, VS Code), Codex, Cursor,
Gemini CLI, and other AGENTS.md-aware hosts.

## claude-code

Claude Code does NOT read AGENTS.md natively — the overlay imports it.

| Template | Destination |
|---|---|
| `adapters/claude-code/CLAUDE.md` | `CLAUDE.md` (contains `@AGENTS.md` import — verified syntax; relative to the file; max 4 hops) |
| `adapters/claude-code/agents/*.md` | `.claude/agents/<role>.md` |
| `adapters/claude-code/settings.json` | `.claude/settings.json` (deep-merge per `merging.md`) |
| `adapters/claude-code/commands/*.md` | `.claude/commands/audit.md`, `brief.md` |

Frontmatter facts (verified): `tools` is a comma-separated string; `model` accepts aliases
(sonnet/opus/haiku/fable/inherit) and full model IDs; `isolation: worktree` is valid and runs
the subagent in an isolated git worktree. `{{ISOLATION_LINE}}` in the implementer template
becomes `isolation: worktree` when worktrees were chosen, otherwise remove the line entirely.

Model routing mechanics (verified August 2026): subagent model resolution is
`CLAUDE_CODE_SUBAGENT_MODEL` env var → **per-dispatch model parameter** → frontmatter `model:`
→ main session's model. Per-dispatch override is what makes AGENTS.md §16's task-based ladder
real on this host (caveat: as of v2.1.211 an override persists across a subagent resume).
Honesty caveat for generated docs: subscription-quota consumption by model choice is NOT
documented — API-billed cost scales with model, but do not promise Pro/Max quota savings. The
one documented model-specific billing behavior: **Fable 5** (`fable` alias, the apex tier) may
bill to usage credits instead of drawing on the plan's included limits, depending on seat tier
— which is why AGENTS.md §16 treats apex as a deliberate per-dispatch decision, never a role
default. Fable availability depends on the user's plan; where absent, the apex tier resolves
to the strongest model the plan offers.

## github-copilot

Copilot reads AGENTS.md natively on all surfaces, so no context-file overlay is needed. It also
reads a root `CLAUDE.md` if present — the two adapters coexist; keep instructions
non-conflicting.

| Template | Destination |
|---|---|
| `adapters/github-copilot/agents/*.agent.md` | `.github/agents/<role>.agent.md` |
| `adapters/github-copilot/prompts/*.prompt.md` | `.github/prompts/audit.prompt.md`, `brief.prompt.md` |

Format facts (verified August 2026): `.agent.md` is the canonical custom-agent extension
(read by Copilot cloud agent, Copilot CLI, and VS Code); safe cross-surface frontmatter is
`name` + `description` — `tools` takes an array here (NOT a comma string) and its vocabulary
varies by surface, so the kit omits it and enforces restrictions in body prose. Prompt files
are a VS Code / Visual Studio / JetBrains feature (public preview); the cloud agent and CLI
ignore them — the rituals remain reachable via `docs/agents/rituals/` everywhere.

## Other hosts (cursor, codex, gemini-cli, …)

Universal core only. AGENTS.md's §7 tells the host to adopt role cards in fresh sessions when
it has no native subagents. Add a dedicated adapter directory later if a host earns one — the
core does not change.

## Spec Kit integration values

`specify init --here --integration <value> --force` — value per primary host: `claude` for
claude-code, `copilot` for github-copilot. Run init once with the PRIMARY host's integration.
*Verify at run time* with `specify init --help`: whether `--integration` accepts multiple
values (unconfirmed), and current flag names — this CLI has already renamed `--ai` →
`--integration`. After init, detect which command style landed
(`.claude/skills/speckit-*/SKILL.md` → dash style `/speckit-specify`;
`.claude/commands/speckit.*.md` → dot style `/speckit.specify`; for copilot,
`.github/prompts/speckit.*` or agents equivalents) and write that style into AGENTS.md §13.

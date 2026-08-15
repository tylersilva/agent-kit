<!-- agent-kit:begin:claude-overlay -->
@AGENTS.md

## Claude Code specifics

AGENTS.md above is the operating contract. On this host it is backed by real machinery:

- **Subagents.** The role cards in `docs/agents/` are installed as native subagents in
  `.claude/agents/` ({{INSTALLED_ROLES}}). Delegate to them instead of adopting roles inline:
  the implementer for briefs, the researcher for unknowns, the reviewer for audits — the
  reviewer always runs as a subagent so its context is genuinely fresh.
- **Commands.** `/brief <objective>` produces a technical brief (AGENTS.md §8);
  `/audit <target>` dispatches the fresh-context audit (AGENTS.md §10).
- **Approval gates.** AGENTS.md §14 is enforced here via `.claude/settings.json` permissions —
  a denied action means the gate is working, not that you should route around it.
- **Worktrees.** {{WORKTREE_NOTE}}
- **Model routing.** Per-dispatch model override is supported on this host — apply AGENTS.md
  §16's cheap-first ladder when dispatching subagents. The frontmatter models are the floor,
  not the law: mechanical briefs go down a tier, architectural/security/disputed work goes up.
<!-- agent-kit:end:claude-overlay -->

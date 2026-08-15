# Interview Guide

Ask only what detection (see `detection.md`) could not establish. Present detected values for
confirmation, never re-ask them from scratch. Batch questions into at most 4 rounds. Where the
host supports a structured question tool, use it; otherwise ask in plain chat, one batch at a
time.

## Round 1 — Identity

1. **Purpose** — "In one or two sentences, what is this project?" (free text)
2. **Type** — Web app / API or service / CLI tool / Library / Monorepo / Other.
   Shapes the smoke-test framing and examples.

## Round 2 — Commands & verification

3. **Command confirmation** — show the detected build/test/run/lint table and detected coding
   conventions (formatter/linter, style observations): "Correct?" → Correct / corrections
   (free text). For greenfield repos with nothing to detect, ask what the intended stack is.
4. **Smoke test** — "What single command or user path proves this actually works end-to-end?"
   Offer any detected candidate (e2e script, compose up + curl). Accept
   "none yet" → generate the runbook with a clearly-marked TODO procedure instead of inventing
   one.

## Round 3 — Autonomy & roles

5. **Autonomy level**:
   - *Conservative* — agent asks before commit and push.
   - *Standard* (default) — automatic through branch/commit/push/draft PR; merge, deploy, and
     destructive actions gated.
   - *Autonomous* — everything except merge, deploy, destructive actions, and credential
     changes.
6. **Roles** — implementer, researcher, reviewer, security, devops are ALL installed by
   default. Ask only for opt-outs: "Skip security or devops for this project?" (skip both only
   if the project genuinely has no dependencies/CI surface).
7. **Worktrees** — "Use isolated git worktrees for implementation tasks?" Yes (default for
   projects expecting parallel work) / No, plain branches.

## Round 4 — Boundaries & platform

8. **Branch convention** — confirm detected default branch; propose `feature/` + `fix/`
   prefixes unless history shows an existing convention.
9. **Deployment** — "Does this project deploy anywhere? How? And what must an agent never touch
   without approval?" Feeds AGENTS.md §6, the devops role, and settings `ask` entries.
   "No deployment" is a fine answer.
10. **Hard NOs** — "Anything an agent must never do in this repo?" Free text; feeds AGENTS.md
    §14 and deny rules.
11. **Target host(s)** — confirm detected hosts, multi-select: claude-code / github-copilot /
    other (universal core only). A repo may serve several.
12. **Spec Kit** — "Initialize Spec Kit now? (`specify init --here` merges files into this
    repo.)" Yes latest / Yes pinned to a version / Skip for now (record `deferred` in the
    manifest). If `uvx` is unavailable, state that and record `deferred` with the install hint.

## Brownfield extra

13. If AGENTS.md or CLAUDE.md already exists: "Keep your existing file and add the agent-kit
    sections as managed blocks (recommended), or rebuild it guided?" Default: keep + append.

## Update mode

Re-present the manifest's stored answers as defaults; ask only "anything you want to change?"
and re-open just those questions.

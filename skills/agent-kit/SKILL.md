---
name: agent-kit
description: Bootstrap or update an agent-agnostic multi-agent working structure in the current project. Use when the user says "set up agent-kit", "initialize the multi-agent framework", "bootstrap this repo for agents", or asks to install AGENTS.md, agent role definitions, approval gates, and Spec Kit scaffolding.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, AskUserQuestion, WebFetch
license: MIT
---

# agent-kit: Project Bootstrap

You are setting up (or updating) the agent-kit framework in the **current project**: an
AGENTS.md operating contract, specialist role definitions, verification rituals, approval
gates, durable-memory scaffolding, and Spec Kit initialization.

Everything you need ships with this skill: templates under `templates/`, execution guides under
`references/`. Read a reference file when its phase begins — do not improvise where a reference
has an answer. These instructions are host-neutral: where they say "ask the user", use your
host's structured question tool if you have one, plain chat if not.

Work through the six phases in order.

## Phase 0 — Mode

Check for `.agent-kit.json` (also accept legacy `.claude/agent-kit.json`) at the project root.

- **Exists → update mode.** Read it: kit version, stored answers, generated-file list. Tell the
  user what is installed, present stored answers as defaults, and ask what they want to change.
  Then skip to Phase 4, regenerating only kit-managed files/blocks per `references/merging.md`.
- **Absent → install mode.** If the repo already has meaningful content (source files, an
  existing AGENTS.md/CLAUDE.md, `.claude/` or `.github/` customization), it is **brownfield**:
  read `references/merging.md` now and apply its rules throughout. Otherwise greenfield.

## Phase 1 — Preflight

Check, report as a short checklist, and fix only with the user's consent:

1. **Git repository** — required. If absent, offer `git init -b main`; if declined, stop (the
   framework is built on Git).
2. **Clean tree** — recommend committing pending changes first so the kit's output is one
   reviewable diff. Proceed anyway if the user prefers.
3. **`gh` authenticated** — only needed for the optional repo-creation offer in Phase 6;
   degrade gracefully.
4. **`uvx` available** — only needed for Spec Kit init. If missing, note the install hint from
   `references/detection.md` and mark the Spec Kit step deferred.
5. **Inventory** — existing AGENTS.md, CLAUDE.md, `.claude/`, `.github/` customization,
   `.specify/`, `docs/`.

## Phase 2 — Auto-detection

Follow `references/detection.md`. Establish: stack, build/test/run/lint commands, coding
conventions, default branch and remote, branch conventions, CI presence, monorepo shape, and
likely host(s). Produce a compact "Detected" summary. Detection replaces questions — the
interview only confirms.

## Phase 3 — Interview

Follow `references/interview.md` exactly: at most 4 batched rounds, skipping everything
detection answered. You must end this phase holding values for every template token (listed in
Phase 4) plus: roles to install, worktrees yes/no, autonomy level, target host(s), and the
Spec Kit decision.

## Phase 4 — Generation

Read `references/hosts.md` for the template → destination mapping. Generate **core first, then
each selected host adapter**, applying `references/merging.md` wherever a destination exists.

Substitution: replace every `{{TOKEN}}` in copied templates. The tokens:

`{{PROJECT_NAME}}` `{{PROJECT_PURPOSE}}` `{{PROJECT_TYPE}}` `{{TECH_STACK}}`
`{{ARCHITECTURE_SUMMARY}}` `{{REPO_STRUCTURE}}` (a short tree of top-level dirs with one-line
annotations) `{{BUILD_CMD}}` `{{TEST_CMD}}` `{{RUN_CMD}}` `{{LINT_CMD}}` (use `none` where a
command genuinely does not exist) `{{CODING_CONVENTIONS}}` `{{SMOKE_TEST}}` (or the TODO text
if none yet) `{{DEPLOYMENT_NOTES}}` `{{DEPLOY_NEVER_TOUCH}}` (use `nothing — no deployment` when
n/a) `{{DEFAULT_BRANCH}}` `{{BRANCH_PREFIX}}` `{{AUTONOMY_LEVEL}}` `{{HARD_NOS}}`
`{{KNOWN_PITFALLS}}` (seed from the interview; `none recorded yet` if empty)
`{{SPECKIT_COMMAND_STYLE}}` (filled in Phase 5; write `pending Spec Kit init` if deferred)
`{{KIT_VERSION}}` (this skill's version — read it from the source-tracking metadata in this
file's frontmatter if present, else `dev`) `{{INSTALL_DATE}}` (today, YYYY-MM-DD)
`{{INSTALLED_ROLES}}` (comma list) `{{WORKTREE_RULE}}` (` + worktree` if worktrees enabled,
else empty) `{{WORKTREE_NOTE}}` (one sentence: enabled — the implementer subagent runs in an
isolated worktree; or disabled — plain branches) `{{ISOLATION_LINE}}` (see hosts.md)
`{{STACK_ALLOW_ENTRIES}}` `{{DEPLOY_ASK_ENTRIES}}` (see merging.md) `{{HOSTS_JSON}}`
`{{ROLES_JSON}}` `{{WORKTREES_BOOL}}` `{{SPECKIT_STATUS}}` `{{FILES_JSON}}` (manifest values;
FILES_JSON lists every file you actually wrote, each with `"managed": true`).

Skip role files the user opted out of, and remove opted-out roles from AGENTS.md §7's table.

**JSON escaping rule:** when a token lands inside a `.json` template's string value, escape the
value for JSON (newlines become `\n`, quotes become `\"`). Multi-line answers are the common
trap — the coding-conventions and smoke-test answers often span lines.

Write the manifest `.agent-kit.json` **last**, so it records exactly what happened.

**Self-check (mandatory):** search every generated file for `{{` — zero matches allowed; parse
every generated `.json` file; confirm each file in the manifest exists.

## Phase 5 — Spec Kit

Skip if the user chose to defer, or if `.specify/` already exists (then only detect the command
style, below). Otherwise:

1. Require a committed tree and the user's explicit go-ahead — init merges files into the
   working tree.
2. First run `uvx --from git+https://github.com/github/spec-kit.git specify init --help` and
   read it: this CLI changes fast (`--ai` has already become `--integration`). Adapt the
   command to what the help actually says.
3. Run init for the primary host, typically:
   `uvx --from git+https://github.com/github/spec-kit.git specify init --here --integration <claude|copilot> --force`
   (append the user's pinned version to the git URL as `@<tag>` if they chose pinning).
4. **Detect the command style** that landed (dash `speckit-*` skills vs dot `speckit.*`
   commands, per `references/hosts.md`) and substitute it into AGENTS.md §13's
   `{{SPECKIT_COMMAND_STYLE}}`.
5. Draft constitution input from the interview (purpose, conventions, verification rules,
   approval gates) into a short note for the user, and tell them to run the Spec Kit
   constitution command as their next session's first step — the constitution is Spec Kit's
   artifact; feed it, don't forge it.

## Phase 6 — Verify & report

1. Re-run the Phase 4 self-check across everything, including Phase 5 substitutions.
2. If no git remote exists and `gh` is authenticated, offer:
   `gh repo create <project-name> --private --source . --push` (private by default — the
   user's call).
3. Offer to commit everything as `chore: initialize agent-kit`.
4. Report: files created / updated / skipped (with reasons), detected vs. answered values, the
   Spec Kit status, and next steps — restart the agent host so new agents/settings load; run
   the Spec Kit constitution command; try the brief and audit rituals on something small.

Throughout: never claim a step succeeded without having seen it succeed — this kit's own
verification rules apply to you while you install it.

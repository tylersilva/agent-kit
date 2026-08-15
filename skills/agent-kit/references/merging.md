# Brownfield Merge Rules

The prime directive: **never destroy content the user wrote.** When in doubt, append and tell
the user what to reconcile.

## Managed blocks

Every kit-generated section in AGENTS.md and CLAUDE.md is wrapped in
`<!-- agent-kit:begin:<section> -->` / `<!-- agent-kit:end:<section> -->` markers.

- **Generate**: if the file does not exist, write it whole. If it exists without markers,
  append the kit's blocks after the existing content, then flag any obvious contradictions
  (e.g. the existing file names different test commands) for the user to reconcile — do not
  silently resolve them.
- **Update**: replace only the content between matching markers. Everything outside markers is
  untouched, byte for byte. A block the user deleted stays deleted (removing a block is an
  opt-out); note it in the report instead of resurrecting it.

## AGENTS.md / CLAUDE.md specifics

- Existing `CLAUDE.md` with real content + claude-code selected: keep the user's content, add
  the overlay block (with `@AGENTS.md` import) at the top. If their CLAUDE.md duplicates what
  moved into AGENTS.md, point out the duplication; let the user trim.
- Existing `AGENTS.md` from another tool: treat as user content; append kit blocks below it.

## .claude/settings.json deep-merge

- Parse both. If the existing file fails to parse, stop and report — never overwrite a file you
  could not read.
- `permissions.allow/ask/deny`: set-union the arrays. **On conflict (same rule in different
  lists), the existing file's placement wins.** Preserve all keys the kit does not manage.
- Write back with stable formatting; confirm the result parses before moving on.

## Autonomy variants (settings template adjustments)

The template ships the **Standard** baseline. Adjust at generation time:

- **Conservative**: move `Bash(git add:*)`, `Bash(git commit:*)`, `Bash(git push:*)`,
  `Bash(gh pr create:*)` from `allow` to `ask`.
- **Autonomous**: as Standard (merge/deploy/destructive stay gated — that is the §24 floor,
  regardless of autonomy level).
- `{{STACK_ALLOW_ENTRIES}}`: JSON string entries for the detected commands, e.g.
  `"Bash(npm test:*)", "Bash(npm run build:*)", "Bash(npm run lint:*)"`. Never allowlist a
  command you did not detect or the user did not confirm.
- `{{DEPLOY_ASK_ENTRIES}}`: entries for deploy commands named in the interview (prefix each
  with a comma to keep the JSON valid), or empty string if none.

## Other collisions

- `.claude/agents/<role>.md`, `.github/agents/<role>.agent.md`, commands, prompts, docs seeds:
  if the exact path exists and is not kit-managed per the manifest, keep the existing file,
  write nothing, and report the skip. If kit-managed, regenerate.
- `docs/adr/` numbering: if ADRs already exist, do not add `0001` — install only `template.md`
  and reference the existing convention.

## Never

- Never edit `.specify/` content — that is Spec Kit's territory.
- Never touch files outside the manifest's file list plus the explicit destinations above.
- Never run `specify init` on a dirty tree, and never without the user's explicit go-ahead.

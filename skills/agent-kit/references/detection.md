# Detection Recipes

Run these before the interview. Everything detected here is *confirmed*, not re-asked. Prefer
reading files over guessing; a detection you are not sure of is presented as a candidate, not a
fact.

## Stack & commands

| Evidence | Stack | Commands to extract |
|---|---|---|
| `package.json` | Node/JS/TS | `scripts.build/test/dev/start/lint`; note the package manager from lockfile (`package-lock.json`→npm, `pnpm-lock.yaml`→pnpm, `yarn.lock`→yarn, `bun.lockb`→bun) |
| `pyproject.toml` / `setup.py` | Python | tool sections (poetry/uv/hatch); `pytest` presence; `ruff`/`black` config |
| `go.mod` | Go | `go build ./...`, `go test ./...`; Makefile targets override |
| `Cargo.toml` | Rust | `cargo build/test/clippy` |
| `Gemfile` | Ruby | `bundle exec rake`, rspec presence |
| `pom.xml` / `build.gradle*` | JVM | `mvn verify` / `./gradlew build test` |
| `Makefile` / `justfile` | any | list targets; `make test`, `make build` etc. take precedence as the human-blessed interface |
| `docker-compose*.yml` / `Dockerfile` | container | compose up as run/smoke candidate |

Multiple manifests in subdirectories → monorepo: capture the layout in the architecture
summary and note per-package commands.

## Coding conventions

Look for: formatter/linter configs (`.editorconfig`, `.prettierrc*`, `eslint.config.*`,
`ruff.toml`/`[tool.ruff]`, `.golangci.yml`, `rustfmt.toml`), language version pins, and obvious
style in existing source (naming, test file layout). Summarize in 2–4 bullets; the formatter's
word is final.

## Git

- Default branch: `git symbolic-ref refs/remotes/origin/HEAD` (fallback: current branch).
- Remote: `git remote get-url origin` (absence → offer repo creation in the final phase).
- Branch convention: `git branch -a` — existing `feature/`, `fix/`, `chore/` prefixes are the
  convention; otherwise propose `feature/` + `fix/`.
- Dirty tree: `git status --porcelain` — recommend committing before generation.

## CI

`.github/workflows/*.yml` → CI exists; note which workflows run tests, whether anything smoke
tests. Feeds the devops role and the smoke-test runbook's CI-gate section.

## Host detection

| Evidence | Host |
|---|---|
| `.claude/` dir or `CLAUDE.md` | claude-code |
| `.github/copilot-instructions.md`, `.github/agents/`, `.github/prompts/` | github-copilot |
| `.cursor/` | cursor (universal core) |
| `.agents/` shared dir | multi-agent setup already present (universal core; do not disturb) |
| none of the above | ask in the interview |

## Existing agent-kit state

- `.agent-kit.json` at repo root → update mode.
- Existing `AGENTS.md`/`CLAUDE.md` without markers → brownfield merge per `merging.md`.

## Prerequisites

- `git rev-parse --is-inside-work-tree` — hard requirement (offer `git init -b main`).
- `gh auth status` — needed only for repo-creation offer; degrade gracefully.
- `uvx --version` — needed only for Spec Kit init; if absent, report the install hint
  (`curl -LsSf https://astral.sh/uv/install.sh | sh` or `brew install uv`) and defer that step.

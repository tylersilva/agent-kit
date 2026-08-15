# Extension Roles (not installed by default)

The architecture document (§35) names later-stage specialists to add **only when a recurring
workload justifies them**. Recipe: copy `docs/agents/reviewer.md`'s structure, fill the §6
fields below, add the role to AGENTS.md §7's delegation table, and (per host) render it like
the existing five.

All extension roles run on the **cheap tier** (haiku-class, AGENTS.md §16) — their work is
high-volume and mechanical; anything that turns out to need judgment escalates up the ladder.

| Role | Purpose | Model tier | Activation trigger | Notes |
|---|---|---|---|---|
| GitHub operator | Issue/PR triage, labels, release notes | cheap | New issue, stale PR | Read-mostly `gh` work; merge stays human-gated |
| Dependency monitor | Track updates & advisories | cheap | Schedule, advisory published | Reports to `docs/security/findings.md`; PRs for safe bumps |
| Repository health | Dead code, flaky tests, TODO rot | cheap | Schedule | Output = prioritized issue list, not drive-by fixes |
| Documentation drift | Docs vs. reality divergence | cheap | Feature merged | Compares AGENTS.md/runbooks/README against actual behavior |
| Release | Changelog, versioning, release PRs | cheap | Release requested | Release publishing is approval-gated |
| Data | Analysis, migrations support | cheap | Data task in a brief | Gets the implementer's rules plus data-safety boundaries |

**Activation rule (AGENTS.md §16):** wake on the trigger, do the work, stop. Never leave a
specialist polling — a specialist with nothing to do consumes nothing. Resist adding a role
before the workload exists — the document's own rule (§41.19) is to add complexity only when
actual workload proves the need.

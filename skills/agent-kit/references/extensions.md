# Extension Roles (not installed by default)

The architecture document (§35) names later-stage specialists to add **only when a recurring
workload justifies them**. Recipe: copy `docs/agents/reviewer.md`'s structure, fill the §6
fields below, add the role to AGENTS.md §7's delegation table, and (per host) render it like
the existing five.

| Role | Purpose | Triggers | Notes |
|---|---|---|---|
| GitHub operator | Issue/PR triage, labels, release notes | New issues, stale PRs | Read-mostly `gh` work; merge stays human-gated |
| Dependency monitor | Track updates & advisories | Scheduled, advisory published | Reports to `docs/security/findings.md`; PRs for safe bumps |
| Repository health | Dead code, flaky tests, TODO rot | Scheduled | Output = prioritized issue list, not drive-by fixes |
| Documentation drift | Docs vs. reality divergence | After feature merges | Compares AGENTS.md/runbooks/README against actual behavior |
| Release | Changelog, versioning, release PRs | Release request | Release publishing is approval-gated |
| Data | Analysis, migrations support | Data task in a brief | Gets the implementer's rules plus data-safety boundaries |

Keep them quiet: a specialist with nothing useful to say should say nothing. Resist adding a
role before the workload exists — the document's own rule (§41.19) is to add complexity only
when actual workload proves the need.

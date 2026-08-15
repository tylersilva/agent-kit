# Runbook: Smoke Test

The standing end-to-end check for {{PROJECT_NAME}}. Run it after any application-level change —
unit tests passing is not completion (AGENTS.md §5).

## Procedure

Start from the user's actual entry point and exercise the meaningful path through the system:

{{SMOKE_TEST}}

## Pass criteria

The full path completes with the expected result, using output you observed — not inferred.
Record the evidence (command output, response body, screenshot) in the task's report.

## CI gate

Where practical, this smoke test should run in CI so it is an explicit completion gate rather
than an optional suggestion. If it is not yet in CI, treat wiring it in as standing follow-up
work for the devops role.

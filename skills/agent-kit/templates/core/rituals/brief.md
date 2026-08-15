# Ritual: Technical Brief

Use before delegating any implementation work. The worker gets this brief and the artifacts it
names — never the planning conversation.

## Procedure

1. From the current planning context, distill the task into the format below.
2. Fill every section. Anything not known is written as `UNKNOWN — <what would resolve it>`
   rather than guessed; resolve UNKNOWNs via the researcher before dispatch if they block work.
3. Requirements must be verifiable; VERIFICATION must name the actual commands/checks.
4. DO NOT must state the boundaries explicitly — workers honor what is written, not what is
   assumed.

## Format

```text
OBJECTIVE
<one sentence: what done looks like>

PROJECT
<project name>

BRANCH / WORKTREE
<dedicated branch for this task>

SPEC KIT
<the active feature artifacts in .specify/ to follow, or "none">

REQUIREMENTS
- <specific, verifiable requirement>
- <...>

VERIFICATION
- <test command(s) to run>
- <smoke check to perform (AGENTS.md §5)>

DO NOT
- Merge the PR.
- Change production configuration.
- <task-specific boundaries>
```

5. Dispatch to the implementer role (`docs/agents/implementer.md`) with the brief.
6. When the report returns, verify the EVIDENCE section before accepting any claim in it.

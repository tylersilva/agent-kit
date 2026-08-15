# Persistent Multi-Agent Claude Code System — Updated Implementation Plan

## 1. Executive Summary

Build a private, persistent, home-hosted AI runtime in which a single **Persona/Brain** is the primary interface, a small number of long-lived specialists provide focused capabilities, and short-lived Claude Code workers execute individual pieces of work.

The key architectural decision is to **not build a second software-development workflow engine**. For application/software work, Spec Kit is the project-level workflow engine. The custom runtime is responsible for agent lifecycle, routing, resource management, permissions, usage-aware scheduling, event handling, and remote access. Claude Code is the execution environment. Git/GitHub provide durable versioned project state.

The target is therefore not "22 Claude sessions." The target is a **personal AI operating environment** that can dynamically create or activate the right worker for a task, keep focused contexts small, run work in isolated Git worktrees, automatically pause when Claude capacity is exhausted, and resume work from durable project state.

---

# 2. Core Architectural Boundary

Use this separation throughout the design:

```text
YOU
 │
 ▼
Persona / Brain
 │
 ├── Research request ─────────────► Research specialist
 │
 ├── Infrastructure request ───────► DevOps specialist
 │
 ├── Security request ─────────────► Security specialist
 │
 └── Software request ─────────────► Coding worker
                                         │
                                         ▼
                                      Spec Kit
                                         │
                              ┌──────────┼──────────┐
                              ▼          ▼          ▼
                           Specify     Plan       Tasks
                                                     │
                                                     ▼
                                                 Implement
                                                     │
                                          Review / Verify
                                                     │
                                                     ▼
                                                     PR
```

### Your runtime owns

- Agent registry
- Agent lifecycle
- Starting/stopping workers
- Routing work to specialists
- Model selection
- Claude usage-aware scheduling
- Permissions and approval policy
- Git worktree allocation
- Event handling
- Runtime state
- Logs and observability
- SSH/Tailscale access
- Optional web UI

### Spec Kit owns

- Specification-driven development
- Project development artifacts
- Requirements/clarification
- Planning
- Task decomposition
- Implementation workflows
- Workflow orchestration
- Human gates inside development workflows
- Loops
- Fan-out/fan-in
- Workflow pause/resume
- Feature-sized sub-agent delegation where supported

Spec Kit's current workflow engine supports commands, prompts, shell steps, human checkpoints, conditional logic, loops, fan-out/fan-in, and persistent pause/resume state. Its workflow runs store state under `.specify/workflows/runs/<run_id>/`. citeturn920865search0turn920865search2

### Claude Code owns

- Actual model interaction
- Shell/file/tool execution
- Code changes
- Test execution
- Repository inspection
- Interactive task work

### Git/GitHub own

- Version history
- Branches
- Worktrees
- PRs
- Review history
- CI results
- Recovery/collaboration surface

---

# 3. Host Environment

## Recommended host: Unraid → Ubuntu VM

Use the existing Unraid server instead of paying for a VPS.

```text
Unraid
└── Ubuntu Agent VM
    ├── Tailscale
    ├── Agent Supervisor
    ├── Agent Registry
    ├── Scheduler
    ├── Event Router
    ├── Usage Manager
    ├── Claude Code
    ├── tmux
    ├── Git / GitHub CLI
    ├── Project repositories
    └── Logs / runtime state
```

### Why a VM

- Persistent uptime
- Isolation from the Unraid host
- Independent CPU/RAM allocation
- Easy snapshots/backups
- No VPS bill
- Local access to repositories and tooling
- Tailscale support

Do not run the agent system directly as root on the Unraid host. Keep it inside a dedicated Ubuntu VM and use a dedicated non-root user for the agent runtime.

---

# 4. Remote Access

Use Tailscale as the private network layer.

```text
Phone / Android
       │
Laptop / Desktop
       │
       ▼
    Tailscale
       │
       ▼
 Ubuntu Agent VM
       │
       ├── SSH
       ├── tmux
       └── Agent API
```

Do not expose SSH or an agent-control API directly to the public internet unless there is a deliberate reason to do so.

The baseline workflow should remain:

```bash
ssh agent@ai-server
tmux ls
tmux attach -t persona
```

This makes the system usable even if the custom web interface fails.

---

# 5. Directory Structure

Use a separation between runtime, agents, projects, and runtime state.

```text
/opt/ai/
│
├── system/
│   ├── supervisor/
│   ├── router/
│   ├── scheduler/
│   ├── events/
│   └── web/
│
├── agents/
│   ├── persona/
│   ├── security/
│   ├── devops/
│   ├── researcher/
│   ├── reviewer/
│   └── ...
│
├── projects/
│   ├── project-a/
│   ├── project-b/
│   └── ...
│
├── state/
│   ├── system/
│   └── agents/
│
├── logs/
│
└── config/
```

A project itself should look more like:

```text
project-a/
├── .git/
├── .github/
├── .specify/
├── .claude/
├── CLAUDE.md
├── docs/
├── architecture/
├── adr/
├── src/
└── tests/
```

The exact layout can vary. The separation of concerns should not.

For software development, **do not create a parallel `/tasks/pending/running/completed` hierarchy as the second source of truth**. Spec Kit artifacts and workflow runs already provide durable workflow state. Use runtime state only for runtime concerns such as process identity, worker state, worktree ownership, and scheduling.

---

# 6. Agent Definitions

Every persistent specialist should have a formal definition rather than being merely a tmux session.

Each definition should include:

```text
Identity
Purpose
Responsibilities
Allowed tools
Project/workspace scope
Permissions
Default model
Escalation model
Triggers
Autonomy level
Reporting format
Persistent runtime state
```

Example:

```text
agents/security/
├── CLAUDE.md
├── config.yaml
└── state/
    ├── current.md
    ├── findings.md
    └── history.md
```

Example configuration:

```yaml
agent:
  name: security
  role: security-engineer

permissions:
  repository_read: true
  repository_write: true
  create_branch: true
  create_pr: true
  merge_pr: false
  production_access: false

autonomy:
  level: 2

model:
  default: efficient
  escalation: strong

triggers:
  - pull_request
  - vulnerability
  - scheduled_scan
  - manual
```

---

# 7. Persistent Specialists vs Ephemeral Workers

The original idea of maintaining 22 permanent Claude sessions should not be the default.

Use two categories.

## Persistent specialists

Keep these alive because their role benefits from long-lived context or recurring responsibilities.

Examples:

- Persona / Brain
- Security
- DevOps / Infrastructure
- Research
- Review / QA
- GitHub operator
- General operator

## Ephemeral workers

Launch these only when there is actual work.

Examples:

- Feature implementation worker
- Bug-fix worker
- Test worker
- Documentation worker
- Refactor worker
- Migration worker
- Data-analysis worker

Spec Kit can delegate focused implementation tasks to sub-agents when the coding agent supports that mode. It also supports bounded implementation runs and decomposition of large features into smaller specs. citeturn920865search3turn920865search6

Therefore, **22 is a possible concurrency ceiling, not a target number of permanent Claude processes**.

---

# 8. The Persona / Brain

The Persona is the single interface you normally interact with.

It should:

- Understand your request
- Identify what kind of work it represents
- Inspect relevant project state
- Choose an appropriate specialist
- Produce a clear technical brief
- Dispatch the work
- Monitor the result
- Challenge unsupported success claims
- Escalate uncertainty to research
- Request an independent audit when something looks wrong
- Return decisions, evidence, and next actions to you

The brain should generally be the **highest-reasoning model** available to you, while workers can use a more efficient model when the task permits.

Conceptually:

```text
YOU
 │
 ▼
Persona / Brain
 │
 ├── Coding ──────► Coding worker ──────► Spec Kit
 ├── Research ────► Research specialist
 ├── Security ────► Security specialist
 └── Operations ──► DevOps specialist
```

The brain should not send the entire conversation to the worker. It should produce a concise technical brief containing objective, project, worktree, relevant artifacts, constraints, and required verification.

---

# 9. Worker Session Model: One Task = One Focused Chat

Adopt the rule:

**One meaningful task = one focused Claude session.**

Do not build a single never-ending coding conversation for an entire project.

A worker session should have:

```text
One objective
One worktree
One branch
One focused context
One completion criterion
```

When the work is complete, start a fresh context for the next independent task.

The supplied practice of keeping a task below roughly 500k tokens is a useful operational heuristic, not a guaranteed model-quality boundary. The important idea is to prevent enormous histories from becoming the default development mechanism.

Spec Kit directly supports this approach by allowing implementation to be limited to a bounded set of tasks or phases, delegated to sub-agents, or decomposed into smaller specs when needed. citeturn920865search3

Do not treat auto-compaction as equivalent to a clean task boundary. Prefer durable artifacts plus fresh task contexts.

---

# 10. Git Is Mandatory

Every serious project should use Git from the first meaningful change.

Recommended flow:

```text
Create project
   ↓
Initialize Git
   ↓
Create private GitHub repository
   ↓
Push immediately
   ↓
Work on branches/worktrees
   ↓
Commit frequently
   ↓
Push branches
   ↓
PR / review
   ↓
Merge
```

A private GitHub repository should be the default when confidentiality is a concern.

The point is not only collaboration. Git provides recovery and an externally durable record if the Ubuntu VM, local filesystem, or Claude session fails.

---

# 11. Worktrees Are the Default for Parallel Work

Never have multiple independent workers casually edit the same working tree.

Use worktrees:

```text
project/
├── main/
├── .worktrees/
│   ├── feature-auth/
│   ├── bug-login-timeout/
│   ├── docs-api/
│   └── migration-db/
└── .git/
```

The normal unit should be:

```text
1 task
1 branch
1 worktree
1 Claude session
```

This allows parallel implementation, independent review, safe rollback, and clean PR boundaries.

A supervisor should create and dispose of worktrees as part of worker lifecycle.

---

# 12. Project Context: CLAUDE.md and Rules

The project repository should contain the durable operating instructions for Claude Code.

## Root CLAUDE.md

Create a high-quality `CLAUDE.md` describing:

```text
Project purpose
Architecture
Repository structure
Technology choices
How to build
How to test
How to run
How to deploy
Coding conventions
Security rules
Git / branch / worktree rules
Verification requirements
Smoke-test procedure
Known pitfalls
Documentation expectations
Durable-memory rules
```

## Rules

Use project/user rules where supported for stable behavioral constraints.

Include rules such as:

```text
Never guess when the repository or environment can be inspected.
Always measure and verify.
Always use Git.
Use a dedicated branch/worktree for meaningful changes.
Keep task context focused.
Run the relevant tests.
Perform a full end-to-end smoke check for application changes.
Do not claim success without evidence.
Record important discoveries in durable documentation.
Ask for approval before consequential actions.
```

The main brain's instructions should also make clear that its responsibility is delegation and verification, not blindly accepting worker claims.

---

# 13. Durable Memory: Never Trust "Remember This"

Treat conversational memory as transient.

When something matters later, require the agent to write it into a file and report the exact path.

Examples:

```text
docs/
architecture/
adr/
runbooks/
project-notes/
```

A good rule is:

> If this information would matter in another session, another agent, or after a reboot, write it to a durable project artifact.

For software projects, Spec Kit's own artifacts are the natural place for feature-level decisions and workflow state. Other persistent information belongs in project documentation, ADRs, runbooks, or agent-specific state.

---

# 14. Spec Kit: Project-Level Development Workflow

Spec Kit should be the standard development workflow for projects that use it.

A normal feature should look like:

```text
Human request
      ↓
Persona
      ↓
Coding worker
      ↓
/speckit.specify
      ↓
/speckit.clarify
      ↓
/speckit.plan
      ↓
/speckit.tasks
      ↓
/speckit.analyze
      ↓
/speckit.implement
      ↓
Tests
      ↓
Full smoke test
      ↓
Independent audit / review
      ↓
PR
```

The exact invocation varies by coding-agent integration, but the workflow concept remains the same. Spec Kit's current quickstart includes project-level principles/constitution and the specify → plan → tasks → implement lifecycle. citeturn920865search4

## Important architectural rule

Do not make the supervisor understand feature-level implementation states such as:

```text
specifying
planning
implementing T003
reviewing requirement 4
```

That belongs to Spec Kit.

The supervisor only needs to know things such as:

```text
agent active
agent idle
workflow active
workflow paused
workflow completed
process failed
Claude unavailable
```

---

# 15. Spec Kit Parallelism and Large Features

Before creating additional permanent agents, use Spec Kit's native options for parallel development.

### Option A — Scope implementation

Run only a bounded group of tasks or one phase.

### Option B — Delegate parallel tasks

Where the coding agent supports sub-agents, delegate independent tasks.

### Option C — Combine both

Bound the work and delegate parallel subtasks.

### Option D — Spec of Specs

If the entire feature is still too large, first produce a roadmap of independently specified sub-features and execute those separately.

Spec Kit explicitly documents all four strategies. citeturn920865search3turn920865search6

This should dramatically reduce the need for 22 permanent coding sessions.

---

# 16. Brain-to-Worker Delegation

The main brain chat should be the high-context planning surface.

Workers should start in a waiting state such as:

```text
wait for task
```

The brain should then send a focused technical brief.

Example:

```text
OBJECTIVE
Implement OAuth callback handling.

PROJECT
project-a

WORKTREE
feature/oauth-callback

SPEC KIT
Use the active Spec Kit feature artifacts in .specify/.

REQUIREMENTS
- Preserve existing session behavior.
- Add callback validation.
- Add tests for invalid state.

VERIFICATION
- Unit tests
- Integration test
- Full application smoke check

DO NOT
- Change production configuration.
- Merge the PR.
```

The worker does not need the brain's conversation transcript.

---

# 17. Verification and Anti-Bullshit Controls

Treat Claude's statements as claims until backed by evidence.

The runtime and project rules should enforce:

```text
Do not say "it works" without performing the relevant check.
Do not say "I checked everything" without defining what was checked.
Do not infer environment state from memory.
Do not guess when the repository or environment can be inspected.
Prefer measurements, commands, tests, logs, and reproducible evidence.
```

## Different-point-of-view audit

When you suspect something is wrong, do not endlessly extend the same conversation.

Start a separate, fresh audit context and explicitly instruct it to challenge the result.

```text
Assume the previous implementation may be wrong.
Independently reproduce the reported behavior.
Inspect the implementation from a different point of view.
Look for hidden assumptions, edge cases, missing tests, race conditions, and incorrect claims.
Do not rely on the previous agent's assertion that the feature works.
```

This keeps questionable context from contaminating the implementation conversation.

---

# 18. Application Smoke Testing

For applications, make end-to-end smoke testing a standing rule.

The smoke test should start from the user's actual entry point and exercise the meaningful path through the system.

Examples:

```text
Browser → frontend → API → database → response

or

CLI → service → external dependency → result
```

The smoke test should be reproducible and documented in the project.

Where practical, put it into CI and/or a Spec Kit workflow so it becomes an explicit completion gate rather than an optional suggestion.

Do not declare a feature complete merely because unit tests pass.

---

# 19. Research Escalation

When a worker says it does not know, cannot do something, or proposes a path that seems dubious, the brain should request research instead of accepting the claim.

```text
Worker uncertainty
      ↓
Separate research task
      ↓
Trusted sources
      ↓
Evidence-backed finding
      ↓
Return to implementation
```

For technical questions, prefer:

- Official vendor documentation
- Official specifications
- Source repositories
- Primary technical references

Explicitly require the researcher to distinguish:

```text
Confirmed fact
Inference
Unknown
Recommendation
```

Keep exploratory research in a separate context when it is likely to generate competing hypotheses.

---

# 20. Model Routing

Use strong reasoning capacity for orchestration and difficult reasoning, and efficient models for repetitive execution.

Example policy:

```text
Brain / Persona       → strongest available model
Architecture          → strong model when needed
Security analysis     → strong model when needed
Research synthesis    → strong model when needed
Implementation        → efficient worker model
Routine docs          → efficient worker model
Triage                → efficient worker model
Monitoring            → efficient worker model
```

If the current Claude environment makes a Sonnet-class worker materially cheaper or gives it more practical usage than a stronger model, prefer the stronger model for the brain and the efficient model for workers when task complexity allows it.

Do not hard-code model names into the architecture. Model names and availability can change; model classes and routing policies should be configurable.

---

# 21. Usage-Aware Scheduling

Treat Claude capacity as a finite resource.

The supervisor should maintain a conceptual budget:

```text
Claude capacity
───────────────
Available: 72%
Reserved: 15%
Usable:   57%
```

Priority:

```text
P0 — urgent production/operational issue
P1 — important active feature or bug
P2 — normal development
P3 — background research/maintenance
```

Example behavior:

```text
100–50% → normal operation
50–25%  → reduce background work
25–10%  → prioritize P0/P1
<10%    → do not launch background work
limit   → stop launching Claude work
reset   → resume eligible work
```

The goal is not to bypass provider limits. The goal is to schedule intelligently around them.

---

# 22. Pause and Resume Around Claude Limits

Separate runtime process state from development workflow state.

If a Claude process becomes unavailable:

```text
Claude session unavailable
       ↓
Supervisor records process state
       ↓
Workflow/project state remains durable
       ↓
Capacity returns
       ↓
Restart worker
       ↓
Resume relevant workflow
```

For Spec Kit workflows, use Spec Kit's persisted state and resume support rather than inventing a second feature-checkpoint system. Spec Kit can resume a paused or failed workflow from the exact step where it stopped. citeturn920865search0

This is the cleanest way to achieve your original goal of "pause everything when I hit the session limit and pick it up automatically after reset."

---

# 23. Event System

Prefer events over constant polling.

Potential events:

```text
GitHub PR opened
GitHub issue created
GitHub Actions failed
New vulnerability detected
Scheduled maintenance window
Deployment failed
Worker failed
Claude capacity threshold crossed
Claude capacity reset
Spec Kit workflow completed
Spec Kit workflow paused
Human instruction received
```

Example:

```text
GitHub PR
    ↓
Event Router
    ├── Reviewer
    ├── Security
    └── Verification worker
```

Do not wake a language model every few seconds simply to ask whether anything needs doing.

---

# 24. Human Approval Gates

Separate safe automation from consequential operations.

## Normally automatic

- Read repositories
- Inspect logs
- Run tests
- Create branches
- Create worktrees
- Commit changes
- Push private branches
- Create draft PRs
- Create GitHub issues
- Run Spec Kit workflows
- Run safe smoke tests

## Approval required by default

- Merge into protected branches
- Production deployments
- Destructive infrastructure actions
- Credential changes
- Firewall/network changes
- Deletion/destruction
- External public communication
- Other irreversible actions

Spec Kit also provides explicit human gate steps, so project-specific workflow approvals can live in Spec Kit instead of a custom workflow engine. citeturn920865search1

The outer supervisor still needs permission boundaries because Spec Kit shell steps execute with local privileges and do not provide a runtime capability sandbox. citeturn920865search1

---

# 25. GitHub Integration

GitHub should be the external collaboration and recovery surface.

Agents should be able to:

- Read issues
- Create issues
- Read PRs
- Create branches
- Push commits
- Open PRs
- Review PRs
- Read Actions results
- Inspect workflow failures
- Update labels
- Generate release notes

Recommended feature path:

```text
Human request / GitHub issue
            ↓
         Persona
            ↓
      Coding worker
            ↓
         Worktree
            ↓
        Spec Kit
            ↓
       Implementation
            ↓
    Review / security / tests
            ↓
      Full smoke check
            ↓
           PR
            ↓
        Human gate
            ↓
          Merge
```

---

# 26. Shared Knowledge vs Agent Memory

Use three layers.

## Runtime-wide knowledge

```text
Agent roles
Security policy
Model policy
Usage policy
Host configuration
Approval policy
```

## Project-wide knowledge

```text
CLAUDE.md
Rules
README
Architecture docs
ADRs
Runbooks
Development commands
Smoke-test instructions
```

## Feature/task knowledge

```text
.specify/
spec.md
plan.md
tasks.md
workflow state
PR
verification results
```

Do not inject all three layers into every worker. Select only what the worker needs.

---

# 27. Context Management

Context is both a quality constraint and a cost constraint.

A worker should receive:

```text
Task objective
+
Relevant Spec Kit artifacts
+
Relevant files
+
Relevant project rules
+
Relevant verification instructions
+
Small amount of prior context when necessary
```

Avoid automatically injecting:

```text
Entire repository history
Entire brain conversation
All worker conversations
Unfiltered logs
Unrelated research
```

The brain should send a technical brief, not a transcript dump.

When the implementation context gets too large, first split the work into bounded runs or use Spec Kit's documented parallel/decomposition strategies. citeturn920865search3

---

# 28. Runtime Supervisor

The supervisor is the main piece of custom infrastructure.

It should manage **runtime concerns only**.

## Process lifecycle

- Start agent
- Stop agent
- Restart agent
- Detect crashes
- Detect inactivity
- Attach/detach tmux
- Allocate worker sessions
- Allocate worktrees
- Clean up finished worker processes

## Routing

- Select persistent specialist
- Launch ephemeral worker
- Provide technical brief
- Select model class
- Collect result metadata

## Resource management

- CPU
- RAM
- Concurrency
- Claude capacity
- Scheduling priorities

## Security

- Enforce runtime permissions
- Prevent unauthorized production actions
- Protect secrets
- Require approval when configured

## Observability

- Agent state
- Process state
- Project
- Worktree
- Model
- Last activity
- Logs
- Usage state

The supervisor should **not** implement its own version of `spec.md`, `plan.md`, `tasks.md`, workflow gates, or Spec Kit workflow state.

---

# 29. tmux

Use tmux as the initial persistent process/session layer.

Example:

```text
tmux
├── persona
├── security
├── devops
├── researcher
├── reviewer
└── general
```

Persistent workers can sit in:

```text
wait for task
```

until activated by the brain/supervisor.

However, the architecture should represent:

```text
Agent
Process
Project
Worktree
Session
State
```

not:

```text
tmux pane
```

This allows migration to systemd services, containers, or another process model later.

---

# 30. Web Interface

Do not build the web interface first.

Start with:

```text
SSH + tmux
```

Once the runtime is reliable, expose it through a browser.

The UI should show:

```text
AGENTS

● PERSONA       ACTIVE       Planning OAuth feature
● SECURITY      WAITING      -
● DEVOPS        IDLE         -
● REVIEWER      WORKING      PR #42
● CODER         WORKING      feature/oauth-callback
```

For each active worker, show:

- Agent
- Project
- Worktree
- Current objective
- Status
- Model
- Last activity
- CPU/RAM
- Usage state
- Logs

For software tasks, show the associated Spec Kit workflow/run status rather than maintaining a second task-state representation.

Useful controls:

```text
Send instruction
Pause
Resume
Restart
Approve
Reject
View logs
Attach to session
```

---

# 31. Mobile Access

The first-class fallback remains:

```text
Android
  ↓
Tailscale
  ↓
SSH
  ↓
Ubuntu
  ↓
tmux
```

The web UI is a convenience layer, not a dependency.

---

# 32. Security Model

Treat the environment as an automation platform with meaningful privileges.

Use:

- Dedicated Ubuntu VM
- Dedicated non-root agent user
- Minimal sudo privileges
- Tailscale-only remote access
- SSH keys/passkeys where appropriate
- No public agent API by default
- No plaintext secrets in Git or prompts
- Secret-management mechanism
- Minimum-scope GitHub tokens
- Separate development/production credentials
- Explicit production approval
- VM snapshots/backups
- Audit logs

Do not give every agent root.

Do not give every agent production credentials.

Do not allow arbitrary workers to modify the supervisor or its security policy without a controlled approval path.

Spec Kit workflows require additional scrutiny because shell steps execute local commands with the user's privileges and do not constitute a sandbox. Review third-party workflow definitions before installing them. citeturn920865search1

---

# 33. Failure Handling and Recovery

The supervisor should not be the source of truth for project work.

Runtime state should cover:

```text
Agent registry
Process IDs
Session IDs
Worktree allocations
Resource state
Usage state
Event offsets
Logs
```

Project recovery should come from:

```text
Git/GitHub
Spec Kit artifacts
Spec Kit workflow state
Project documentation
```

After a reboot:

```text
Ubuntu reboots
      ↓
systemd
      ↓
Supervisor
      ├── restore persistent specialists
      ├── inspect worker processes
      ├── inspect active worktrees
      ├── inspect Spec Kit workflow runs
      └── resume eligible work
```

Use systemd to start the supervisor and other core services automatically.

---

# 34. Observability

Maintain a central runtime status view.

At minimum:

```text
Agent
Status
Project
Worktree
Workflow run
Model
Started
Last activity
CPU
Memory
Usage state
```

Example:

```text
PERSONA       ACTIVE       project-a        -
CODER         WORKING      project-a        feature/oauth
REVIEWER      WAITING      -                -
SECURITY      WORKING      project-a        PR #42
DEVOPS        IDLE         -                -
RESEARCH      WORKING      project-b        research/auth
```

Logs should be retained independently of tmux output.

Spec Kit's workflow CLI can expose workflow status in machine-readable form, which makes it suitable for consumption by a future supervisor or web UI. citeturn920865search0turn920865search1

---

# 35. Recommended Initial Agent Set

Start with two:

```text
1. Persona / Brain
2. Coding worker
```

Then add:

```text
3. Research
4. Review / QA
5. DevOps
6. Security
```

Only add more persistent specialists when a recurring workload justifies them.

Potential later specialists:

```text
GitHub operator
Dependency monitor
Repository health
Documentation drift
Release
Product analysis
Data
Infrastructure
General operator
```

Implementation workers can remain ephemeral.

---

# 36. Development Phases

## Phase 1 — Host

Set up:

```text
Unraid
└── Ubuntu VM
    ├── Tailscale
    ├── Git
    ├── GitHub CLI
    ├── Claude Code
    └── tmux
```

Verify:

- SSH
- Tailscale
- tmux
- Claude Code
- Git
- GitHub authentication
- Persistent storage

Do not build the supervisor yet.

## Phase 2 — One Real Project

Choose a single real project.

Establish:

```text
GitHub repository
CLAUDE.md
Rules
Git branching/worktree policy
Smoke test
Development commands
Durable documentation rules
Spec Kit initialization
Project principles / constitution
```

Spec Kit's current recommended process includes establishing project-level principles before feature work. citeturn920865search4

## Phase 3 — Brain + Worker

Create:

```text
persona
coder
```

Keep the worker waiting for a task.

Give the brain the task of producing a technical brief and delegating it.

Test:

```text
Brain
 ↓
New worktree
 ↓
Coder worker
 ↓
Spec Kit
 ↓
Implementation
 ↓
Tests
 ↓
Smoke test
 ↓
PR
```

## Phase 4 — Add Audit + Research

Add:

```text
reviewer
researcher
```

Test:

- Separate-context audit
- Trusted-source research
- Worker verification
- Fresh-chat debugging

## Phase 5 — Runtime Supervisor

Implement only:

```text
start worker
stop worker
restart worker
allocate worktree
inspect status
route work
collect logs
```

Do not build a duplicate software task engine.

## Phase 6 — Model / Usage Management

Add:

- Model routing
- Worker concurrency limits
- Usage monitoring
- Background-work throttling
- Pause on capacity exhaustion
- Resume of eligible work

## Phase 7 — GitHub Events

Connect:

```text
GitHub
 ↓
Event Router
 ↓
Persona / specialist
 ↓
Spec Kit workflow
```

Start with PRs, issues, and failed Actions runs.

## Phase 8 — Autonomous Specialists

Add scheduled/event-driven specialists such as:

```text
Security
Dependency monitoring
Repository health
Documentation drift
Infrastructure monitoring
```

They should remain quiet when there is nothing useful to do.

## Phase 9 — Parallel Worker Scaling

Increase worker concurrency only after the model-routing, worktree, verification, and usage controls are reliable.

Aim for enough parallelism to improve throughput; do not target 22 permanent Claude processes just because 22 is possible.

## Phase 10 — Web UI

Only after the CLI/tmux runtime is stable should you build the browser interface.

The UI should consume supervisor state plus Spec Kit workflow state, not introduce another source of truth.

---

# 37. Example End-to-End Feature

Suppose you tell the Persona:

> Add OAuth authentication to Project X.

The intended behavior is:

```text
YOU
 │
 ▼
PERSONA
 │
 ├── Inspect repository
 ├── Inspect current Spec Kit state
 └── Assign coding worker
          │
          ▼
      Git worktree
          │
          ▼
      Claude Code
          │
          ▼
       Spec Kit
          │
          ├── Specify
          ├── Clarify
          ├── Plan
          ├── Tasks
          └── Analyze
                  │
                  ▼
               Implement
                  │
              ┌───┼───┐
              ▼   ▼   ▼
            Task Task Task
              │   │   │
              └───┼───┘
                  ▼
            Tests + Audit
                  │
                  ▼
           Full smoke test
                  │
                  ▼
                  PR
                  │
       ┌──────────┼──────────┐
       ▼          ▼          ▼
   Reviewer     Security   Verification
       └──────────┼──────────┘
                  ▼
             Human approval
                  ▼
                Merge
```

If the feature becomes too large for a clean run, use bounded implementation, sub-agent delegation, or Spec Kit's spec-of-specs decomposition rather than growing the same conversation indefinitely. citeturn920865search3turn920865search6

---

# 38. Final Target Architecture

```text
                         YOU
                    /      |      \
                 Phone   Laptop    Web
                    \      |      /
                     Tailscale
                         │
                         ▼
                  Ubuntu Agent VM
                         │
             ┌───────────┴───────────┐
             │                       │
        Agent API               SSH / tmux
             │                       │
             ▼                       │
       ┌─────────────┐               │
       │ Supervisor  │◄──────────────┘
       └──────┬──────┘
              │
       ┌──────┼──────┐
       ▼      ▼      ▼
    Agents  Events  Scheduler
       │      │      │
       └──────┼──────┘
              ▼
       Persona / Router
              │
      ┌───────┼────────┐
      ▼       ▼        ▼
    Coder  Research   DevOps
      │       │        │
      ▼       ▼        ▼
 Claude Code / specialists
      │
      ▼
    Spec Kit
      │
      ├── specify
      ├── clarify
      ├── plan
      ├── tasks
      ├── implement
      ├── workflows
      └── review / converge
      │
      ▼
   Git worktree
      │
      ▼
    GitHub
```

---

# 39. Operating Rules for Claude Code

These should become project/system standards.

1. **Always use Git.**
2. **Push important work to GitHub promptly.** Use private repositories when confidentiality matters.
3. **Always use a dedicated branch/worktree for meaningful parallel work.**
4. **One task = one focused Claude session.** Avoid endlessly growing conversations.
5. **Keep task context bounded.** Treat ~500k tokens as a practical ceiling heuristic, not a hard law.
6. **Keep a high-quality CLAUDE.md and rules.**
7. **Never trust an agent's "everything works" claim without evidence.**
8. **When something feels wrong, perform an independent audit in fresh context.**
9. **When uncertain, research rather than guess.** Prefer trusted primary sources.
10. **Make full end-to-end smoke testing a standing rule for applications.**
11. **Use the strongest practical model for the brain and efficient models for workers when appropriate.**
12. **Do not rely on conversational memory for durable facts. Write important knowledge to files.**
13. **Measure and inspect rather than reason from assumptions.**
14. **Keep worker prompts focused and give them a clear technical brief.**
15. **Treat project artifacts and Git history as the durable record.**
16. **Do not let multiple workers edit the same working tree casually.**
17. **Use Spec Kit for feature-level software-development workflow rather than building a duplicate workflow engine.**
18. **Use runtime orchestration for agent lifecycle, not for replacing Spec Kit.**

---

# 40. What This Architecture Eliminates

Compared with the original 22-agent design, this version deliberately avoids building several redundant systems.

### No custom feature specification engine

Spec Kit handles it.

### No custom implementation task state machine

Spec Kit's feature artifacts and workflow state handle it.

### No custom workflow-loop engine

Spec Kit workflows already provide loops, conditional logic, fan-out/fan-in, and gates. citeturn920865search0

### No custom feature checkpoint format

Use Spec Kit workflow state plus Git/project artifacts.

### No requirement for 22 permanent Claude processes

Use persistent specialists plus ephemeral workers and Spec Kit sub-agent delegation.

### No dependence on one giant conversation

Use focused task sessions, worktrees, and durable artifacts.

The custom runtime therefore stays relatively small and focused.

---

# 41. Core Design Principles

1. **One interface, many specialists.** You normally talk to the Persona.
2. **Spec Kit owns software workflow.** Do not recreate it.
3. **Git is mandatory.** Version everything.
4. **Worktrees isolate parallel work.** One task should normally have one worktree.
5. **One task = one focused session.** Fresh contexts are a feature, not a failure.
6. **Context is a budget.** Give workers only what they need.
7. **Never trust unsupported success claims.** Verify everything important.
8. **Audit from a fresh context when something looks wrong.**
9. **Research instead of guessing.**
10. **Smoke-test applications end to end.**
11. **Use strong models for high-leverage reasoning and efficient models for execution.**
12. **Usage limits are scheduling constraints.** Pause and resume instead of trying to bypass them.
13. **Persistent specialists, ephemeral implementation workers.**
14. **Event-driven beats constant polling.**
15. **Human approval remains explicit for consequential operations.**
16. **tmux is a runtime convenience, not the architecture.**
17. **The repository is durable memory.**
18. **Home infrastructure is sufficient.** Unraid + Ubuntu + Tailscale can replace the VPS for this use case.
19. **Start with one brain and one worker.** Add complexity only when actual workload proves the need.
20. **22 agents is a possible operating scale, not a required design.**

The resulting system is a **personal AI runtime**: the Persona is the control surface, the supervisor is the runtime layer, persistent specialists handle recurring responsibilities, ephemeral workers execute focused tasks, Spec Kit manages software-development workflow, Claude Code performs the work, and Git/GitHub provide durable project state.

# Track: process configuration

Status: **DRAFT v2**, 2026-09-24. Specification: [`process-configuration.md`](process-configuration.md). Roadmap: `wiki/roadmap/ecosystem-roadmap.md`, process-configuration line (it replaces the earlier playbooks track PB0–PB4 of `skill-project-management/.specs/drafts/playbooks-track.md`). First refuter run: [`../reviews/2026-09-24-refuter-v0.md`](../reviews/2026-09-24-refuter-v0.md).

## Why

The board's structure — element types, statuses, the producer–reviewer loop, the roles — is Go code inside task-board. The operators decided on 2026-09-24 that extracting it into an external, project-level configuration handled by its own module is the first priority: it lets anyone describe their own process, it makes the board usable beyond software, and the next board can reuse the module instead of rewriting it. The current board stays in maintenance; new capabilities arrive as modules connected through thin seams.

## Milestones

| # | Delivers | Done when |
| --- | --- | --- |
| PC0 | draft specification, JSON Schema, examples (this repository); the refuter's first run on v1 and this v2 revision | merged as a draft; every refuter finding is resolved or listed as open (spec §18) |
| PC1 | the module: schema, parser, resolver for `extends`, `uses` and slots, `playbook validate` (library and CLI, portable and machine checks); a thin seam in task-board that loads `default.playbook.json`; new types from `Playbook.json` run on the generic engine; a structured `assignment.role` | task-board behaves identically on the default playbook, or every difference is documented and accepted; a new type declared in `Playbook.json` runs end to end |
| Demos | right after PC1: `dev-cycle`, `bug-hunt` → `dev-cycle`, `scrum` over `dev-cycle`; the cafe once `task-board tick` and signed events exist (PC3) | each demo runs from a poorly specified input to a terminal state under its orchestrator |
| PC2 | role skills; the `curator-playbook-dev-cycle` package (orchestrator plus role skills, template in the orchestrator's data) on Skillfile schema 2; task-board resolves roles from the project overlay `.roles`, then installed role skills, then `~/.roles` read-only for one release with a warning; `setup.sh` stops copying `~/.roles` | a clean machine gets every role from one Skillfile selector; this realizes roadmap milestone M1 |
| PC3 | delivery-module seams (Change Requests, worktrees, integration, validation as guards, actions and invariants); `task-board tick`; signed events and approvals; hand-off through waggle offers | the delivery invariants hold on every delivery workflow; the cafe demo runs its SLA, schedule and payment event |

PC4 of the first plan (a Curator package kind for playbooks) is dropped: a playbook is a collection of skills.

## Demo playbooks

Each demo takes a deliberately rough input and runs it through its playbook under one orchestrator. Chains show one playbook's output becoming another's input.

1. `dev-cycle` — today's delivery process, cleaned up, with the delivery invariants.
2. `bug-hunt` → `dev-cycle` — a proven finding is offered to the fixing process.
3. `scrum` over `dev-cycle` — a cadence composed over delivery, sprints holding stories by link.
4. Cafe operations — inventory → purchasing → accounting: humans, tools, timers, recurrence, signed events.
5. Incident → post-mortem → fix in `dev-cycle`.
6. Patent — research → draft → refuter → filing.
7. Agency — lead → estimate → project (`dev-cycle`) → invoice.

## Dependencies

- Curator: release Skillfile schema 2 from its flag; `dependencies.skills` gains `directory` selection (spec §8.3).
- Agent selection (`skill-project-management/.specs/drafts/agent-selection.md`): the operator file that binds models and effort to needs; §9 of the spec amends its §3 and §8.
- waggle: approver signatures for risky human acts, hand-off offers between orchestrators, scope leases for composed orchestrators.
- Session host: marks hosted sessions so the kernel can tell agent sessions from people; runs `task-board tick` every minute.
- task-board: the seam for PC1; the goal ledger stays there.

## Risks

- Reproducing today's permissive status model exactly: the kernel has no transition table and several gates depend on run context (spawned run, manual move). PC1 may accept documented differences instead of identity; spec §14 lists them.
- Authority is only as strong as the actor signal: until hosted sessions are marked and actions signed, refusals for agent runs are a guardrail, so risky human acts require signatures from the start.
- Two sources of truth for roles during migration (`.roles`, `~/.roles`, installed skills): the PC2 lookup order must be explicit and logged.
- Schema churn before PC1: keep `schema_version` at 1 until the module ships, then version changes explicitly.

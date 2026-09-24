# Track: process configuration

Status: **DRAFT**, 2026-09-24. Specification: [`process-configuration.md`](process-configuration.md). Roadmap: `wiki/roadmap/ecosystem-roadmap.md`, process-configuration line (it replaces the earlier playbooks track PB0–PB4 of `skill-project-management/.specs/drafts/playbooks-track.md`).

## Why

The board's structure — element types, statuses, the producer–reviewer loop, the roles — is Go code inside task-board. The operator and Alexey decided on 2026-09-24 that extracting it into an external, project-level configuration handled by its own module is the first priority: it lets anyone describe their own process, it makes the board usable beyond software, and the next board can reuse the module instead of rewriting it. The current board stays in maintenance; new capabilities arrive as modules connected through thin seams.

## Milestones

| # | Delivers | Done when |
| --- | --- | --- |
| PC0 | draft specification, JSON Schema, examples (this repository); first refuter run on the specification against the wiki's recorded decisions | reviewed and merged as a draft; the refuter's verdict is recorded |
| PC1 | the module: schema, parser, resolver for `extends`, `uses` and slots, `playbook validate` (library and CLI); a thin seam in task-board that loads `default.playbook.json` | task-board behaves byte-identically on the default playbook, or every difference is documented and accepted |
| PC2 | role skills; the `curator-playbook-dev-cycle` package (orchestrator plus role skills, template in the orchestrator's data); task-board resolves roles from installed skills before `.roles` and `~/.roles`; `setup.sh` stops copying `~/.roles` | a clean machine gets every role from one Skillfile selector; this realizes roadmap milestone M1 |
| PC3 | custom entity types and workflows live in the board; delivery-module seams (CR, worktree, integrate, validation as guards and actions); the demo playbooks and the generality vectors as a conformance suite | every demo runs from a poorly specified input to a terminal state under its orchestrator |

## Demo playbooks

Each demo takes a deliberately rough input and runs it through its playbook under one orchestrator. Chains show one playbook's output becoming another's input.

1. `dev-cycle` — today's delivery process, cleaned up.
2. `bug-hunt` → `dev-cycle` — a proven finding becomes a development task.
3. `scrum` over `dev-cycle` — cadence composed over delivery.
4. Cafe operations — inventory → purchasing → accounting: humans, tools, timers, recurrence.
5. Incident → post-mortem → fix in `dev-cycle`.
6. Patent — research → draft → refuter → filing.
7. Agency — lead → estimate → project (`dev-cycle`) → invoice.

## Dependencies

- Curator: release of Skillfile schema 2 sources and selectors; the proposal that `dependencies.skills` gains directory selection (specification §8.3).
- Agent selection (`skill-project-management/.specs/drafts/agent-selection.md`): the operator file that binds concrete models to needs.
- waggle: approver signatures for `on_unmet: ask` and human approvals across machines.
- task-board: the seam for PC1; the goal ledger stays there.

## Risks

- Reproducing today's permissive status model exactly: the kernel has no transition table and several gates depend on run context (spawned producer, manual move). PC1 may accept documented differences instead of byte identity.
- Two sources of truth for roles during migration (`.roles`, `~/.roles`, installed skills): the lookup order in PC2 must be explicit and logged.
- Schema churn before PC1: keep `schema_version` at 1 until the module ships, then version changes explicitly.

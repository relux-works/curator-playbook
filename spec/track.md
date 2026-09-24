# Track: process configuration

Status: **DRAFT v3**, 2026-09-24. Specification: [`process-configuration.md`](process-configuration.md). Roadmap: `wiki/roadmap/ecosystem-roadmap.md`, process-configuration line (it replaces the earlier playbooks track PB0–PB4 of `skill-project-management/.specs/drafts/playbooks-track.md`). Refuter runs: [`../reviews/2026-09-24-refuter-v0.md`](../reviews/2026-09-24-refuter-v0.md) on v1 and [`../reviews/2026-09-24-refuter-v1.md`](../reviews/2026-09-24-refuter-v1.md) on v2. v3 applies the operator's decisions of 2026-09-24 (spec §17.2).

## Why

The board's structure — element types, statuses, the producer–reviewer loop, the roles — is Go code inside task-board. The operators decided on 2026-09-24 that extracting it into an external, project-level configuration handled by its own module is the first priority: it lets anyone describe their own process, it makes the board usable beyond software, and the next board can reuse the module instead of rewriting it. The current board stays in maintenance; new capabilities arrive as modules connected through thin seams.

## Milestones

| # | Delivers | Done when |
| --- | --- | --- |
| PC0 | draft specification, JSON Schema, examples (this repository); the refuter's first run on v1 and second run on v2, answered by v2 and v2.1; the operator's decisions applied in v3 | every finding of both runs is resolved (spec §18, §19), every decision that belongs to a person is taken (spec §17.2), and a refuter pass on the sections v3 changed finds no contradiction; the specification is then frozen |
| PC1 | the module: schema, parser, resolver for `extends`, `uses` and slots, `playbook validate` (library and CLI, portable and machine checks); a thin seam in task-board that loads the project file from the head of the default branch and `default.playbook.json`; new types from `Playbook.json` on the generic engine, the `field` guard included; a structured `assignment.role`; `rejected` in the goal ledger; `policy.wip`; the version-control policy read from `Playbook.json`; local check sets over today's landing-gate suite; the legacy operator layer read from today's config; the constructor-ready schema; the minimal delivery seams over today's commands and the local hand-off on one board, which the demos need (spec §15) | task-board behaves identically on the default playbook, or every difference is documented and accepted; a new type declared in `Playbook.json` runs end to end; the three first demos can start |
| Demos | right after PC1 *(decided 2026-09-24)*: `dev-cycle`, `bug-hunt` → `dev-cycle` and `scrum` over `dev-cycle`, all on one board; the cafe with bookkeeping after PC3 and the waggle amendment | each demo runs from a poorly specified input to a terminal state under its orchestrator |
| PC2 | role skills, installed on demand once Curator ships it (description stopgap until then); the JSON Schema of `data/role.json`; the `curator-playbook-dev-cycle` package (orchestrator plus role skills in `skills/`, template in the orchestrator's data, tool providers outside the collection) on Skillfile schema 2; task-board resolves roles from the project overlay `.roles`, then installed role skills, then `~/.roles` read-only for one release with a warning; `setup.sh` stops copying `~/.roles` | a clean machine gets every role from one Skillfile selector; this realizes roadmap milestone M1 |
| PC3 | the delivery module proper (the PC1 seams become a module that checks its invariants itself); hosted check sets and the local-mirror rule; `task-board tick` with its user-level system timer and the tool executor; signed events and approvals once the waggle amendment lands with CM2; hand-off across boards through waggle | the delivery invariants hold on every delivery workflow; the cafe demo runs its SLA, schedule, tool and payment event |

PC4 of the first plan is dropped *(decided)*: a playbook is a collection of skills. The constructor-ready schema, which the roadmap's PC4 row also carried, moves into PC1; the Curator changes are the Curator goal. The wiki's roadmap rows are updated there.

## Demo playbooks

Each demo takes a deliberately rough input and runs it through its playbook under one orchestrator. Chains show one playbook's output becoming another's input.

1. `dev-cycle` — today's delivery process, cleaned up, with the delivery invariants. After PC1.
2. `bug-hunt` → `dev-cycle` — a proven finding is offered to the fixing process on the same board. After PC1.
3. `scrum` over `dev-cycle` — a cadence composed over delivery, sprints holding stories by link, estimate fields delegated by the slot binding. After PC1.
4. Cafe operations with bookkeeping — inventory → purchasing → accounting: humans, a tool, timers, recurrence, signed events, a hand-off whose result comes back. After PC3 and the waggle amendment.
5. Incident → post-mortem → fix in `dev-cycle`.
6. Patent — research → draft → refuter → filing.
7. Agency — lead → estimate → project (`dev-cycle`) → invoice.

## Dependencies

- **Curator:** release Skillfile schema 2 from its flag; `dependencies.skills` gains `directory` selection, so an orchestrator can depend on a tool provider in its own repository; activation `on-demand` and a skills launch channel, so a role skill reaches only the launches that run its role (spec §8.1, §8.3; curator-spec issue #91, curator issue #91). Runtime-mode dependencies narrowed to commands already exist (curator-spec core §4.4).
- **Agent selection** (`skill-project-management/.specs/drafts/agent-selection.md`): the spec amends its §3 (need vocabulary and spelling), §7 (admission without an operator file) and §8 (temporary relaxation), and LP D9 (launch profiles) for the case without an operator file; `Playbook.json` supersedes its committed project layer for role needs, admission, data policy and waivers.
- **waggle:** approver signatures, hand-off offers and scope leases, which exist; and **an amendment**, decided on 2026-09-24, that is written into the waggle specification now and implemented with milestone CM2 (spec §17.2): a class for board approvals and signed transitions, an event class with expiry and de-duplication, a service principal kind for event bridges, a `coord` type that returns a hand-off's result, and user verification available to board policy. PC3 and the cafe demo need it; the PC1 demos do not.
- **Scheduler:** `task-board tick` is called every minute by a user-level system timer — a launchd agent on macOS, a systemd user timer on Linux — installed by one task-board command; the Keeper takes it over later (spec §6.9). PC3.
- **Lease store:** the board's own lease on one machine; waggle's lease store (NATS JetStream KV, milestone CM2) for a board shared across machines. Until CM2 a shared board runs tick on one machine only.
- **Session host:** marks hosted sessions so the kernel can tell agent sessions from people (spec §4.2).
- **Hosting provider:** hosted check sets read the provider's required checks on the exact head that lands (spec §6.12). PC3.
- **Operator files:** `~/.curator/models.toml`, `runtimes.toml`, `engines.toml` and `network.toml`, together (spec §2.3); commands that edit them are an open question (spec §17.3).
- **task-board:** the seam for PC1, the delivery seams over today's commands, the local hand-off, `rejected` in the goal ledger, `policy.wip`, check sets over today's landing-gate suite, and the legacy reading of today's config keys.

## Risks

- Reproducing today's permissive status model exactly: the kernel has no transition table and several gates depend on run context (spawned run, own element, manual move). PC1 may accept documented differences instead of identity; spec §14 lists them.
- Authority is only as strong as the actor signal and the default branch: until hosted sessions are marked and actions signed, refusals for agent runs are a guardrail; a project without git, or one whose landings are not reviewed, weakens the revision in force.
- Signatures show that a person acted only when they carry user verification; keys that cannot carry it (Secure Enclave keys today) protect the key but not presence. v3 requires user verification by default where a project turns signatures on, so such a project needs FIDO keys or lowers the requirement to `if-supported`.
- Role skills stay visible to every session until Curator ships on-demand activation; the description stopgap depends on the model obeying it.
- Two sources of truth for roles during migration (`.roles`, `~/.roles`, installed skills): the PC2 lookup order must be explicit and logged.
- PC1 grows by the delivery seams and the local hand-off that the early demos need; keep each seam a thin adapter over today's commands.
- Schema churn before PC1: keep `schema_version` at 1 until the module ships, then version changes explicitly.

**Next.** A refuter pass on the sections v3 changed, then the freeze. PC1 starts with the seam that loads the default playbook from the head of the default branch and passes the goldens; the delivery seams and the local hand-off follow, and the three first demos start as soon as those land.

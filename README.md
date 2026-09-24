# curator-playbook

Process configuration for agent systems. A project describes how it works — who does what, which element types exist, how each type moves from state to state and what must hold before it moves — in one file, `Playbook.json`, beside its `Skillfile.json`. This module will own the file's schema, its parser, the resolver for extension and composition, and the static validator `playbook validate`.

## Status

**Draft design, v4.0, the freeze candidate.** Nothing is implemented yet. The specification, the JSON Schema and the examples are drafts (milestone PC0). Six refuter runs challenged the drafts, and each was answered; all six are kept in [`reviews/`](reviews/). v4.0 applies the operator's decisions of 2026-09-25 — the user-facing names, the trust framework and the production principles — and splits the draft: the core ([`spec/process-configuration.md`](spec/process-configuration.md)) keeps the process model and is frozen after the operator's choice on which changes wait for a person (spec §17.3 item 13) and a check of v4.0's changes, while the trust machinery moves to [`spec/trust.md`](spec/trust.md), which continues its own refuter rounds. The Russian translations of the specification and the track still describe v2.1 and follow the freeze; the English files are canonical.

## Layout

| Path | Content |
| --- | --- |
| [`spec/process-configuration.md`](spec/process-configuration.md) ([RU](spec/process-configuration.ru.md)) | the draft specification |
| [`spec/trust.md`](spec/trust.md) | the trust machinery: who counts as a person, confirmations and their records, presence-bound keys, the operator companion app, headless confirmation, the operator's files; stays private until the licensing and counsel review |
| [`spec/track.md`](spec/track.md) ([RU](spec/track.ru.md)) | milestones PC0–PC3, demo playbooks, dependencies |
| [`reviews/`](reviews/) | refuter runs on the specification |
| [`schema/playbook-v1.schema.json`](schema/playbook-v1.schema.json) | draft JSON Schema for `Playbook.json` |
| [`examples/`](examples/) | today's task-board behaviour as a playbook, dev-cycle, bug-hunt, Scrum composed over dev-cycle (with the project's check sets in `scrum/task-board.config.json`), a cafe composed with bookkeeping, and a prove2.me-style formal process |

## How it fits

- **Today:** task-board loads a playbook through a thin seam; the default playbook reproduces its current behaviour. Development machinery (Change Requests, worktrees, integration, validation) is a delivery module that contributes guards, actions and invariants.
- **Playbooks are data.** A playbook never names a program, script or URL. Anything that executes is a skill installed and audited through Curator: a tool is a command of a provider skill that the orchestrator skill depends on in runtime mode. Side effects a package proposes take effect only when the project enables them, the kernel reads the project's file from the head of its default branch, and a change to the resolved process takes effect only once a person confirms it, unless it stays on a small allowlist that cannot move authority (proposal pending). A person may delegate named human acts to their own orchestrator sessions within limits the project sets.
- **Roles are skills.** A published playbook is a repository of skills — the orchestrator that carries the template plus one skill per role — installed with one Skillfile selector (Curator Skillfile schema 2), on demand, so each role skill reaches only the launches that run its role. Packages are named `curator-playbook-<purpose>`.
- **Later:** other hosts (a main loop managing several processes, a distributed runtime) load the same files.

The ecosystem roadmap lives in `wiki/roadmap/ecosystem-roadmap.md`.

## Checking the examples

```bash
check-jsonschema --check-metaschema schema/playbook-v1.schema.json
check-jsonschema --schemafile schema/playbook-v1.schema.json examples/*.json examples/*/*.[Pp]laybook.json
```

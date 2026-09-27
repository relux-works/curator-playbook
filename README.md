# curator-playbook

Process configuration for agent systems. A project describes how it works — who does what, which element types exist, how each type moves from state to state and what must hold before it moves — in one file, `Playbook.json`, beside its `Skillfile.json`. This module will own the file's schema, its parser, the resolver for extension and composition, and the static validator, `curator playbook check`.

## Status

**Draft design, v4.4, the freeze candidate.** Nothing is implemented yet. The specification, the JSON Schema and the examples are drafts (milestone PC0). Seven refuter runs challenged the drafts, and each was answered; all seven are kept in [`reviews/`](reviews/). v4.0 applied the operator's decisions of 2026-09-25 — the user-facing names, the trust framework and the production principles — and split the draft into the core ([`spec/process-configuration.md`](spec/process-configuration.md)), which keeps the process model, and the trust machinery. v4.1 applies the operator's later decisions of that day: capabilities granted once per kind, batched touches and an explicit yolo mode decide which changes wait for a person. It moves the trust machinery to its own private repository, relux-works/curator-trust ([`spec/trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md)), which continues its own refuter rounds, and it answers the seventh run. v4.2 records the operator's later discussion of that day: role skills reach their runs through per-role profiles instead of on-demand activation, and two proposals, dream mode and a decision consistency check after PC1. The core is frozen after a check of the changes since v4.1. The Russian translations of the specification and the track still describe v2.1 and follow the freeze; the English files are canonical. v4.3 turns loop limits into template parameters a project tunes, down to unbounded, and proposes how entering a dream is approved and conditioned (spec §29). v4.4 applies the command-line shape adopted on 2026-09-27: the validator is `curator playbook check|explain|bind`, a person confirms with `task-board trust confirm`, and each operator file has one `curator <noun>` command family.

## Layout

| Path | Content |
| --- | --- |
| [`spec/process-configuration.md`](spec/process-configuration.md) ([RU](spec/process-configuration.ru.md)) | the draft specification |
| relux-works/curator-trust, [`spec/trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) | the trust machinery, in its own private repository: who counts as a person, confirmations and their records, capabilities and yolo mode, presence-bound keys and the web of trust, multisignatures, the operator companion app, headless confirmation, the operator's files, privileged operations |
| [`spec/track.md`](spec/track.md) ([RU](spec/track.ru.md)) | milestones PC0–PC3, demo playbooks, dependencies |
| [`reviews/`](reviews/) | refuter runs on the specification |
| [`schema/playbook-v1.schema.json`](schema/playbook-v1.schema.json) | draft JSON Schema for `Playbook.json` |
| [`examples/`](examples/) | today's task-board behaviour as a playbook, dev-cycle, bug-hunt, Scrum composed over dev-cycle (with the project's check sets in `scrum/task-board.config.json`), a cafe composed with bookkeeping, and a prove2.me-style formal process |

## How it fits

- **Today:** task-board loads a playbook through a thin seam; the default playbook reproduces its current behaviour. Development machinery (Change Requests, worktrees, integration, validation) is a delivery module that contributes guards, actions and invariants.
- **Playbooks are data.** A playbook never names a program, script or URL. Anything that executes is a skill installed and audited through Curator: a tool is a command of a provider skill that the orchestrator skill depends on in runtime mode. Side effects a package proposes take effect only when the project enables them, the kernel reads the project's file from the head of its default branch, and a change to the resolved process takes effect only once a person confirms it, unless a capability the person granted once covers its kind. Capabilities are asked for once per kind, the way a phone app asks for a permission, and never cover workflow structure or authority; everything else waits and is confirmed in batches, and an explicit yolo mode exists for a single operator on one machine. A person may delegate named human acts to their own orchestrator sessions within limits the project sets.
- **Roles are skills.** A published playbook is a repository of skills — the orchestrator that carries the template plus one skill per role — installed with one Skillfile selector (Curator Skillfile schema 2); each role skill reaches only the runs of its role, through that role's Curator profile. Packages are named `curator-playbook-<purpose>`.
- **Later:** other hosts (a main loop managing several processes, a distributed runtime) load the same files.

The ecosystem roadmap lives in `wiki/roadmap/ecosystem-roadmap.md`.

## Checking the examples

```bash
check-jsonschema --check-metaschema schema/playbook-v1.schema.json
check-jsonschema --schemafile schema/playbook-v1.schema.json examples/*.json examples/*/*.[Pp]laybook.json
```

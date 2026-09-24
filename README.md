# curator-playbook

Process configuration for agent systems. A project describes how it works — who does what, which element types exist, how each type moves from state to state and what must hold before it moves — in one file, `Playbook.json`, beside its `Skillfile.json`. This module will own the file's schema, its parser, the resolver for extension and composition, and the static validator `playbook validate`.

## Status

**Draft design, v2.1.** Nothing is implemented yet. The specification, the JSON Schema and the examples are drafts (milestone PC0). v2 answered the first refuter run on v1 and v2.1 the second run on v2; both runs are kept in [`reviews/`](reviews/). The decisions that belong to a person carry proposed defaults and open questions (spec §17).

## Layout

| Path | Content |
| --- | --- |
| [`spec/process-configuration.md`](spec/process-configuration.md) | the draft specification |
| [`spec/track.md`](spec/track.md) | milestones PC0–PC3, demo playbooks, dependencies |
| [`reviews/`](reviews/) | refuter runs on the specification |
| [`schema/playbook-v1.schema.json`](schema/playbook-v1.schema.json) | draft JSON Schema for `Playbook.json` |
| [`examples/`](examples/) | today's task-board behaviour as a playbook, dev-cycle, bug-hunt, Scrum composed over dev-cycle, a cafe composed with bookkeeping, and a prove2.me-style formal process |

## How it fits

- **Today:** task-board loads a playbook through a thin seam; the default playbook reproduces its current behaviour. Development machinery (Change Requests, worktrees, integration, validation) is a delivery module that contributes guards, actions and invariants.
- **Playbooks are data.** A playbook never names a program, script or URL. Anything that executes is a skill installed and audited through Curator: a tool is a command of a provider skill that the orchestrator skill depends on in runtime mode. Side effects a package proposes take effect only when the project enables them, and the kernel reads the project's file at its committed revision.
- **Roles are skills.** A published playbook is a repository of skills — the orchestrator that carries the template plus one skill per role — installed with one Skillfile selector (Curator Skillfile schema 2). Packages are named `curator-playbook-<purpose>`.
- **Later:** other hosts (a main loop managing several processes, a distributed runtime) load the same files.

The ecosystem roadmap lives in `wiki/roadmap/ecosystem-roadmap.md`.

## Checking the examples

```bash
check-jsonschema --check-metaschema schema/playbook-v1.schema.json
check-jsonschema --schemafile schema/playbook-v1.schema.json examples/*.json examples/*/*.json
```

# curator-playbook

Process configuration for agent systems. A project describes how it works — who does what, which element types exist, how each type moves from state to state and what must hold before it moves — in one file, `Playbook.json`, beside its `Skillfile.json`. This module will own the file's schema, its parser, the resolver for extension and composition, and the static validator `playbook validate`.

## Status

**Draft design.** Nothing is implemented yet. The specification, the JSON Schema and the examples are drafts under review (milestone PC0).

## Layout

| Path | Content |
| --- | --- |
| [`spec/process-configuration.md`](spec/process-configuration.md) | the draft specification |
| [`spec/track.md`](spec/track.md) | milestones PC0–PC3, demo playbooks, dependencies |
| [`schema/playbook-v1.schema.json`](schema/playbook-v1.schema.json) | draft JSON Schema for `Playbook.json` |
| [`examples/`](examples/) | today's task-board behaviour as a playbook, dev-cycle, bug-hunt, Scrum composed over dev-cycle, a cafe, and a prove2.me-style formal process |

## How it fits

- **Today:** task-board loads a playbook through a thin seam; the default playbook reproduces its current behaviour. Development machinery (Change Requests, worktrees, integration, validation) is a delivery module that contributes guards and actions.
- **Roles are skills.** A published playbook is a repository of skills — the orchestrator that carries the template plus one skill per role — installed with one Skillfile selector (Curator Skillfile schema 2). Packages are named `curator-playbook-<purpose>`.
- **Later:** other hosts (a main loop managing several processes, a distributed runtime) load the same files.

The ecosystem roadmap lives in `wiki/roadmap/ecosystem-roadmap.md`.

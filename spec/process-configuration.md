# Process configuration: specification

Status: **DRAFT**, 2026-09-24. Not normative. Statements marked *decided (2026-09-24)* were confirmed by the operator in the design discussion of that day; everything else is *proposed*. Roadmap: `wiki/roadmap/ecosystem-roadmap.md` (process-configuration line, milestones PC0–PC3). Track: [`track.md`](track.md). Schema: [`../schema/playbook-v1.schema.json`](../schema/playbook-v1.schema.json). Examples: [`../examples/`](../examples/).

Supersedes the roles-and-routes model of `skill-project-management/.specs/drafts/playbooks.md`: the core object is no longer a list of roles with routes between them but entity types with workflows whose states are owned by roles. Related: `skill-project-management/.specs/drafts/agent-selection.md` (selection layers), curator-spec `protocol/skillfile-sources.md` (Skillfile schema 2 sources and selectors) and `protocol/core.md` §4.4 (skill dependencies), `waggle/spec/waggle.md` (approver signatures).

---

## 0. The design on one screen

- A project describes how it works in one file, `Playbook.json`, in its root beside `Skillfile.json`. *decided*
- The file has four parts: **roles** (aliases bound to role skills, humans, tools or other processes), **entities** (element types, their nesting and their fields), **workflows** (states, each with exactly one owner, and transitions with guards and actions) and **policy**. *decided*
- A project can **extend** one published playbook and write only its differences, and can **compose** several playbooks, binding their abstract types (slots) to concrete ones. *decided*
- **Roles are skills.** A published playbook is a repository that is a collection of skills: the root orchestrator, whose data carries the process template, and one skill per role. One Skillfile source plus one collection selector installs the whole process. *decided*
- Model wishes are **needs**, never model ids; the operator's private file binds models; an unmet need blocks, falls back, asks for approval or degrades with extra review. *decided*
- A static **validator** (`playbook validate`) checks a configuration before anything runs. *decided*
- Development machinery (Change Requests, worktrees, integration, validation suites) is a **delivery module** that contributes guards and actions; the kernel knows nothing about git. *decided*
- The current task-board behaviour is itself a playbook ([`default.playbook.json`](../examples/default.playbook.json)); PC1 makes the kernel read it with byte-identical behaviour or documented differences. *decided*

## 1. Goals and non-goals

Goals:

1. Any process — software delivery, a restaurant's purchasing, formalizing a proof, a family calendar — is configuration, not kernel code.
2. The same file works for every teammate, whatever harnesses and models each one has.
3. A process can be read, validated and eventually drawn and edited by a visual constructor without losing anything.
4. Processes compose: the output of one can be the input of another, and a cadence (Scrum) can sit over a delivery process.

Non-goals of this revision: the orchestrator's agent loop; scheduling across machines; the goal system itself (`board-goal-v2` stays in task-board; the configuration only names which element is an orchestrator's goal, §6.7); storage format of board elements.

## 2. Files and responsibilities

### 2.1 `Playbook.json` and `Skillfile.json`: two files, two jobs *(decided)*

| File | Answers | Read by | Carries |
| --- | --- | --- | --- |
| `Skillfile.json` | what is installed | Curator | skill names, sources, versions (tags, revisions) |
| `Playbook.json` | how the project works | the board (task-board today) and the validator | role aliases bound to skill names, entities, workflows, policy |

`Playbook.json` never carries a version, a URL or a source; `Skillfile.json` never carries a role, a state or a transition. They meet only through skill names. When a role names a skill that is not installed, the validator reports it with the fix: the `curator add` invocation for a named skill, or the Skillfile selector to add (§8). Installing (Skillfile) and using (Playbook) are two edits with two meanings, like a dependency in `go.mod` and its `import`; a helper such as `playbook bind <alias> <skill>` can make both.

Without Curator the same files work: skills are copied into the harness skill directories by hand and `Playbook.json` does not change.

### 2.2 Format *(decided)*

- JSON with a published JSON Schema: every file starts with `"$schema"` and `"schema_version": 1`. Editors get validation and completion; a constructor reads and writes the file without loss.
- Canonical JSON (the same canonical form Curator uses for digests) defines the semantic digest of a resolved playbook. The `ui` blocks and `description` fields are excluded from the digest.
- Comments are `description` fields on any element; a constructor shows them.
- Every element may carry a `ui` block (positions, colours, collapsed groups) for a visual constructor. The kernel ignores it.
- Keys starting with `x-` are extensions; the kernel ignores them.
- Names of roles, entity types, workflows and states are kebab-case identifiers; field names are snake_case (`task_class`), matching today's element fields.

### 2.3 The operator's private file

Concrete models, harness bindings and personal approvals live in the operator's private file (`$XDG_CONFIG_HOME/task-board/operator.json` as proposed in the agent-selection draft). It is never committed. §9 describes how needs meet it.

## 3. Document structure

```json
{
  "$schema": "…/playbook-v1.schema.json",
  "schema_version": 1,
  "name": "dev-cycle",
  "description": "…",
  "extends": "…",
  "uses": ["…"],
  "slots": { },
  "bind": { },
  "root": "orchestrator",
  "roles": { },
  "entities": { },
  "workflows": { },
  "schedules": [ ],
  "policy": { },
  "ui": { }
}
```

`name` is required for a published template and optional in a project file. `extends` and `uses` are covered in §7, `slots` and `bind` in §7.2, `schedules` in §6.6.

## 4. Roles

### 4.1 Role aliases *(decided)*

`roles` maps a project-level alias to what performs the work. Workflow states name aliases, never skills, so replacing who does a job is one line.

```json
"roles": {
  "developer": { "skill": "dev-developer", "capability": "producer",
                 "needs": { "prefer": { "publisher": "openai", "effort": "max" } } },
  "reviewer":  { "skill": "dev-reviewer", "capability": "reviewer",
                 "needs": { "require": { "not_same_publisher_as": "developer" }, "on_unmet": "ask" } },
  "manager":   { "kind": "human", "who": "cafe-manager", "capability": "approver" },
  "lean":      { "kind": "tool", "tool": "lean-kernel-check", "capability": "checker" },
  "fixers":    { "kind": "comb", "process": "dev" }
}
```

### 4.2 Owner kinds *(decided)*

| `kind` | Performs the work | Bound by | Examples |
| --- | --- | --- | --- |
| `agent` (default) | an agent running the role skill | `skill` | developer, reviewer, refuter |
| `human` | a person or a group | `who`, resolved by the operator or project directory | approving a purchase, confirming a delivery, a human-only decision |
| `tool` | a program, module hook or webhook | `tool` | CI, the Lean kernel, a payment gateway |
| `comb` | another process | `process` | hand-off of a proven finding to the process that fixes it |

`needs` apply to `agent` roles only.

### 4.3 Capabilities

`capability` is the archetype guards reason about: `orchestrator`, `producer`, `reviewer`, `analyst`, `tester`, `approver` (humans), `checker` (tools); custom capabilities use an `x-` prefix. Today's archetypes map directly: implementer → producer, analyst → analyst, reviewer → reviewer, tester → tester (task-board `tools/board-cli/internal/spawn/role.go:60-90`). A guard that requires a capability refuses an owner without it; the validator reports the mismatch before runtime (§10).

### 4.4 The root role *(decided)*

`root` names the orchestrator of the process. Every process has one; in a published playbook it is the skill that carries the template (§8). In a composition the project names the root (§7.2); the other playbooks' orchestrators keep owning their own states.

## 5. Entities

### 5.1 Types and nesting *(decided)*

```json
"entities": {
  "epic":  { "root": true, "contains": ["story", "research-task", "decision"], "workflow": "epic" },
  "story": { "parents": ["epic"], "contains": ["dev-task", "research-task", "design-task"], "workflow": "story" }
}
```

- `root: true` lets a type stand at the top; any other type needs `parents`.
- `contains` and `parents` must agree: a type lists a parent only if that parent contains it.
- `workflow` names the workflow the type runs. A type with `derived_status: true` aggregates its status from its children except where a declared transition sets it (today's epics and stories, §14).

### 5.2 Fields *(decided)*

Non-development processes need data: a purchase order has a supplier, lines and an amount. Types declare typed fields:

```json
"fields": {
  "supplier": { "type": "string", "required": true },
  "amount":   { "type": "money", "currency": "EUR", "required": true },
  "due":      { "type": "date" },
  "severity": { "type": "enum", "values": ["low", "medium", "high", "critical"] },
  "invoice":  { "type": "ref", "ref": "accounting.invoice" },
  "low_items":{ "type": "list", "of": "string" }
}
```

Types: `string`, `text`, `integer`, `number`, `boolean`, `date`, `datetime`, `duration`, `money`, `enum`, `ref`, `list`.

### 5.3 Estimates *(decided)*

Estimates have named dimensions, so complexity and duration are separate:

```json
"estimates": [ { "name": "complexity", "scale": "fibonacci" }, { "name": "duration", "scale": "hours" } ]
```

Scales: `fibonacci`, `points`, `tshirt`, `hours`, `days`. Capacity rules (§6.8) sum one dimension.

### 5.4 Privacy *(proposed)*

`privacy` on a type (`inherit`, `private`, `team`, `public`) and `policy.privacy` (`visibility`, `redact_fields`) keep a family process private and redact amounts from team views.

## 6. Workflows

### 6.1 States *(decided)*

```json
"states": {
  "open":        { "owner": "orchestrator", "requires": ["description"] },
  "development": { "owner": "developer" },
  "review":      { "owner": "reviewer" },
  "estimating":  { "owner": "facilitator", "helpers": [ { "role": "estimator", "min": 3, "blind": true } ] },
  "integrating": { "owner": "ci", "locked": true },
  "done":        { "terminal": true },
  "closed":      { "terminal": true, "resolution": ["wont-do", "duplicate", "obsolete"] }
}
```

- **Exactly one owner** per non-terminal state. The owner is accountable and is what guards check. *decided*
- **Helpers** are roles the owner may engage; they never own the state. Planning poker: owner `facilitator`, helpers `estimator`, answered blind and revealed by the owner. *decided*
- When the owner of a state must be picked per element from a fixed set (today's `development` is worked by a developer, a tester or a doc-writer depending on the assignment), the owner is `{ "one_of": [...], "by": "assignment.role" }`: still exactly one owner per element. *proposed*
- `requires` lists entry conditions: guards applied to every transition into the state.
- `terminal` states have no owner. Leaving a terminal state needs a transition marked `reopen: true`; without one, terminal is final (today's rule, §14).
- `resolution: true` requires a resolution on entry; a list restricts the values.
- `locked: true` means only transitions that name the state in `from` leave it; `from: "*"` never matches it.

Two rules keep ownership clear *(decided)*:

1. Two different results are two states: "built it" and "checked it" are `development` and `review`, never one state with two owners.
2. Parallel work is child elements plus a join guard (`children.terminal`) on the parent's transition, not parallel states.

### 6.2 Transitions *(decided)*

```json
"transitions": [
  { "from": "open", "to": "development", "guards": ["estimate", "assignee", "blockers.clear"], "actions": ["spawn.owner"] },
  { "from": "review", "to": "development", "guards": [ { "guard": "verdict", "is": "changes-requested" } ],
    "limit": { "max": 3, "then": "escalated" } },
  { "from": "review", "to": "validation", "via": "accept_cr", "guards": ["review.accepted"] },
  { "from": "sent", "to": "escalated", "on": { "after": "PT15M" } },
  { "from": "invoiced", "to": "paid", "on": { "event": "payment.received" } },
  { "from": "*", "to": "closed" }
]
```

- `from` is a state, a list of states, or `*` (every non-terminal, non-locked state).
- `via` names the one command that performs the transition (`accept_cr`, `integrate`); any other route to that state is refused.
- `guards` must all pass; `actions` run after the move.
- `on` makes the transition automatic: on an `event` (a tool reports, a webhook fires, all blockers finish) or `after` a duration in the source state (timers, SLA).
- `limit` bounds a loop: after `max` firings for one element the transition leads to `then` instead (rework three times, then escalate).

### 6.3 Guards and actions come from modules *(decided)*

A guard or action is a name, or an object whose `guard`/`action` key names it and whose other keys are its arguments. Names are namespaced by the module that provides them.

| Provider | Guards | Actions |
| --- | --- | --- |
| kernel | `description`, `estimate`, `estimates.present`, `assignee`, `blockers.clear`, `children.terminal`, `children.terminal-or-carried`, `checklist.complete`, `outcome.present`, `fields.present`, `approvals`, `verdict`, `capacity` | `spawn.owner`, `assign`, `create`, `notify`, `handoff`, `escalate`, `record.verdict`, `compute.velocity` |
| delivery module (development only) | `delivery.cr-published`, `review.accepted`, `change_request.absent`, `commit.acknowledged`, `integration.evidence`, `handoff.evidence` | `delivery.integrate`, `close.checkpointed-siblings` |
| tool owners | `tool.passed`, `tool.failed` (as events) | tool invocation on entry |
| timers and events | — | automatic transitions via `on` |

Change Requests, worktrees, integration and validation suites are development-specific. They live in a delivery module that contributes the guards and actions above; a restaurant's playbook never loads it. The kernel keeps only what every process needs. The validator refuses a guard or action no loaded module provides (§10).

### 6.4 Approvals and quorum

`{ "guard": "approvals", "role": "manager", "min": 1 }` needs approvals from that role; `min` above one is quorum voting. For human approvers the approval is a board action or a waggle approver signature (`waggle/spec/waggle.md`), recorded with the element.

### 6.5 Hand-off between processes *(decided)*

A state owned by a `comb` role is worked in another process. The `handoff` action creates the element there and links it back; an event brings the result home:

```json
{ "from": "verifying", "to": "fixing", "guards": ["review.accepted"],
  "actions": [ { "action": "handoff", "to": "fixers", "create": "fix-target", "link_field": "fix" } ] },
{ "from": "fixing", "to": "fixed", "on": { "event": "handoff.terminal" } }
```

`fix-target` here is a slot (§7.2): bug-hunt does not know which type the fixing process uses; the project binds it.

### 6.6 Schedules *(decided)*

Recurring work is created by schedules:

```json
"schedules": [ { "every": "P1W", "create": { "type": "inventory-check", "title": "Weekly inventory" } } ]
```

`every` is an ISO 8601 duration; `cron` is the alternative.

### 6.7 Short goals *(decided)*

Orchestrators lose the thread on large goals; the weaker the model, the shorter the step must be. `policy.goals.orchestrator_goal` names the type whose current instance is the orchestrator's goal — a story, a milestone or a sprint — so the long-term goal lives in the hierarchy and the orchestrator works one bounded step at a time. The goal ledger itself stays in task-board.

### 6.8 Policy

| Key | Meaning |
| --- | --- |
| `review.default` | `required`, `light` or `none` for new elements (today's review policy) |
| `capacity` | per container type, the maximum sum of one estimate dimension (a sprint of 100 complexity points) |
| `privacy` | visibility and redacted fields |
| `version_control` | whether moving work to done needs a commit acknowledgement and for which types (today's policy) |
| `loop_detection` | rework threshold (today's detector) |
| `goals` | §6.7 |

## 7. Extension and composition

### 7.1 `extends`: only the differences *(decided)*

A project starts from one published playbook and writes only what differs. When the playbook publishes a new version, the project's differences stay and everything else updates.

```json
{
  "extends": "dev-cycle",
  "roles": {
    "developer": { "skill": "role-ios-developer" },
    "tester":    { "skill": "role-ios-tester", "capability": "tester" }
  },
  "workflows": { "dev": {
    "states": { "qa": { "owner": "tester" } },
    "transitions": [
      { "from": "review", "to": "done", "remove": true },
      { "from": "review", "to": "qa", "guards": ["review.accepted"] },
      { "from": "qa", "to": "done", "guards": ["outcome.present"] }
    ] } }
}
```

Merge rules *(proposed)*:

1. One parent. Chains are allowed (a project extends `dev-cycle-ios`, which extends `dev-cycle`); cycles are refused.
2. Maps merge by key, recursively: a role, type, workflow, state or field in the child replaces or extends the parent's entry of the same name.
3. Transitions are keyed by `from → to` (and `via` when present): a child transition with the same key replaces the parent's; `remove: true` deletes it. A state with `remove: true` deletes the state and every transition touching it.
4. Lists other than transitions (`contains`, `parents`, `helpers`, guards, actions) are replaced as a whole.
5. The validator runs on the resolved result (§10) and reports which file introduced each error.

### 7.2 `uses` and slots: composition *(decided)*

A project can run several playbooks as one process. A playbook that refers to a type it does not define declares a **slot**; the project binds it:

```json
{
  "uses": ["dev-cycle", "scrum"],
  "bind": { "scrum.work-item": "story" },
  "root": "scrum-orchestrator",
  "entities": { "story": { "parents": ["epic", "sprint"],
    "estimates": [ { "name": "complexity", "scale": "fibonacci" }, { "name": "duration", "scale": "days" } ] } }
}
```

- A slot may state a contract (`requires`: estimate dimensions, fields, terminal states); the validator checks the bound type satisfies it. Scrum requires its work item to carry `complexity` and `duration` estimates.
- Names must be unique across the composition; a clash is resolved by qualifying with the playbook name (`dev-cycle.reviewer`).
- Scrum is not a base to extend: it is a **cadence layer** that decides which work is selected and when, not how an item is built. Composed over dev-cycle, its sprint contains dev-cycle's stories; dev-cycle still owns how each story is delivered. The same mechanism puts bug-hunt in front of dev-cycle.

## 8. Distribution: roles are skills *(decided)*

### 8.1 A published playbook is a collection of skills

```
curator-playbook-dev-cycle/
  README.md
  skills/
    orchestrator/        SKILL.md (name: dev-orchestrator) — the root role
      data/playbook.json — the process template (name: dev-cycle)
    developer/           SKILL.md (name: dev-developer)
      data/role.json     — capability, default needs, definition-of-done items
    reviewer/            SKILL.md (name: dev-reviewer)
    refuter/             SKILL.md (name: dev-refuter)
    researcher/ designer/
```

Every role directory is a real skill: instructions in `SKILL.md`, optional `references/`, `assets/`, `data/` — the roots Curator installs. Role metadata sits in `data/role.json` so it survives any frontmatter rules *(proposed)*. The template lives in the orchestrator skill because the root role owns the process. Packages are named `curator-playbook-<purpose>` *(decided)*.

### 8.2 One Skillfile entry installs the process

Curator's Skillfile schema 2 (curator-spec `protocol/skillfile-sources.md` §1, unreleased extension `skillfile-sources-v1`) declares repository sources and selects skills from them: an individual `{name, from, directory}` selector or a collection `{from, directory, include, exclude?}` selector over immediate child directories. A collection expands every selected directory into an ordinary installed skill.

```json
{
  "schema_version": 2,
  "agents": ["claude_code", "codex_cli"],
  "sources": {
    "dev-cycle": { "git": "https://github.com/relux-works/curator-playbook-dev-cycle", "tag": "v1.0.0" },
    "roles":     { "git": "https://github.com/relux-works/curator-playbook-roles", "tag": "v0.3.0" }
  },
  "skills": [
    { "from": "dev-cycle", "directory": "skills", "include": ["*"] },
    { "name": "security-reviewer", "from": "roles", "directory": "skills/security-reviewer" }
  ]
}
```

The first selector installs the orchestrator and every role of dev-cycle; the second adds one reusable role from another repository, which `Playbook.json` then binds to an alias. `Playbook.json` says only `"extends": "dev-cycle"` plus differences. The board resolves `dev-cycle` by finding the installed skill whose `data/playbook.json` has that name *(proposed)*.

### 8.3 Proposed Curator change

Skill manifests declare their own dependencies through `dependencies.skills` (curator-spec `protocol/core.md` §4.4: `git`, exact `ref`, activation `mode`, optional command narrowing). That entry has no directory selection, so a playbook skill cannot depend on one role inside a multi-skill repository. Proposal: `dependencies.skills` entries gain the same `directory` selection as Skillfile schema 2 individual selectors, with the same containment rules. Until then a playbook depends on its own roles through the collection selector in the project's Skillfile, which is the common case. Schema 2 must also be released; it is unreleased today.

### 8.4 Without Curator

The same directories are copied into the harness skill directories (`.claude/skills`, `.codex/skills`, `agents/skills`). The resolver finds role skills and templates there by name; nothing in `Playbook.json` changes.

## 9. Model needs and unmet needs *(decided)*

A committed file must not break for a teammate with other harnesses, so roles state needs, never model ids:

- `require` — hard: the role cannot run without it.
- `prefer` — soft: ranking only.
- Terms: `publisher`, `family`, `capability`, `effort`, `local`, `not_same_publisher_as` (independent review).

The operator's private file binds concrete models (agent-selection draft, operator layer). When the operator cannot meet a `require`, `on_unmet` decides:

| `on_unmet` | Behaviour |
| --- | --- |
| `block` (default for `require`) | refuse, naming the layer that emptied the candidate set |
| `fallback` | try the playbook's ordered `fallback` needs |
| `ask` | request the operator's approval — a board card or a waggle approver signature — recorded with `approval.scope` (element, story, epic, project) and `approval.expires` |
| `degrade` | run on the best available, mark the result degraded and insert an extra review state before the next terminal state |

Example: the reviewer requires a publisher different from the developer's; a teammate has only one publisher; `ask` fires; the teammate approves "same-publisher review for this epic, 30 days" once. Every selection records which layer supplied what and whether a fallback, approval or degradation applied.

## 10. Static validation *(decided)*

`playbook validate` is a library and a CLI in this module, independent of task-board. task-board runs it when it loads a playbook; a visual constructor runs it on every edit. It resolves `extends` and `uses` first, then checks:

| Check | Example error |
| --- | --- |
| terminal reachability | `workflow dev: no path from review to a terminal state (review → development → review)` |
| undefined states | `workflow dev: transition review → qa: state qa is not defined` |
| unbound aliases | `workflow dev, state review: owner tester is not a role` |
| missing skills | `role developer: skill role-ios-developer is not installed — run curator add role-ios-developer, or add its Skillfile selector` |
| capability mismatch | `workflow dev, transition review → done: guard review.accepted needs a reviewer, owner developer is a producer` |
| estimate dimensions | `transition open → development: guard estimate, but dev-task declares no estimates` |
| slot bindings | `bind scrum.work-item → story: story lacks estimate duration required by the slot` |
| nesting consistency | `entity story: parent sprint does not contain story` |
| cycles without exit | `workflow dev: review and development form a cycle with no limit and no exit` |
| loop limits | `transition review → development: limit.then escalated is not a state` |
| providers | `guard delivery.cr-published: no loaded module provides it` |
| terminal exits | `transition done → open leaves a terminal state without reopen: true` |

A throwaway prototype of the structural checks (terminal reachability, undefined states, unbound aliases, nesting consistency, loop-limit targets, terminal exits, with `uses` and `bind` resolved) was run over [`../examples/`](../examples/); every example passes it and the schema. Skill presence, capability, estimate, slot-contract and provider checks need the module and are not prototyped.

## 11. Generality vectors *(decided)*

The module ships a conformance suite. Each vector is a case the model must express without kernel changes; each demo playbook covers several.

| Vector | Covered by |
| --- | --- |
| custom typed fields | cafe, bug-hunt, formal missions |
| human owners | cafe (approvals, receiving), formal missions (captains), dev-cycle (escalation) |
| tool owners | cafe (supplier portal, payments), formal missions (Lean kernel), dev-cycle (CI) |
| timers and SLA | cafe (supplier confirmation in 15 minutes), formal missions (stalled after 14 days) |
| recurrence | cafe (weekly inventory) |
| external events | cafe (`supplier.confirmed`, `payment.received`) |
| hand-off between processes | bug-hunt → dev-cycle, cafe → accounting |
| parallel branches and join | cafe (delivery lines), scrum (sprint children) |
| quorum voting | scrum (three blind estimators), `approvals` with `min` |
| loop limits with escalation | dev-cycle (rework), cafe (approval), formal missions (five failed checks) |
| resolutions and reopen | dev-cycle (`closed` resolutions, decision superseded) |
| process privacy | cafe (team visibility, redacted amount) |
| multi-dimensional estimates and capacity | scrum |

## 12. The refuter role and decision memory

A draft specification is challenged before it is approved *(decided)*:

- **Where.** A `challenge` state after `draft` (dev-cycle's `spec` workflow), owned by a `refuter` role skill.
- **Input.** The draft plus board memory: `decision` elements (statement, scope, status, links), the specifications in force and their invariants.
- **Method: proof by contradiction.** Assume the draft is implemented as written and derive consequences; look for conflicts with recorded decisions, internal contradictions (two acceptance criteria that cannot hold together), unstated assumptions and guards no role can satisfy.
- **Output: a typed verdict.** `no-contradiction` with the list of decisions checked, so the pass is auditable; `contradiction` with pairs of claim and decision; `human-decision` when only a person can choose.
- **Transitions.** Contradiction → back to `draft`; human decision → `decision-needed` owned by the operator, which records a new `decision`; no contradiction → `approved`.

Agents are good at finding contradictions; what they lack is memory of what was decided. The board supplies it.

## 13. Examples

| File | Shows |
| --- | --- |
| [`default.playbook.json`](../examples/default.playbook.json) | today's task-board behaviour (§14) |
| [`dev-cycle.playbook.json`](../examples/dev-cycle.playbook.json) | delivery with rework limits, refuter, decisions, needs and approvals |
| [`bug-hunt.playbook.json`](../examples/bug-hunt.playbook.json) | proving findings and handing them to the fixing process through a slot |
| [`scrum/scrum.playbook.json`](../examples/scrum/scrum.playbook.json), [`scrum/project.Playbook.json`](../examples/scrum/project.Playbook.json) | a cadence layer composed over dev-cycle; planning poker; capacity |
| [`cafe.playbook.json`](../examples/cafe.playbook.json) | fields, humans, tools, timers, recurrence, events, join, hand-off, privacy |
| [`prove2me-style.playbook.json`](../examples/prove2me-style.playbook.json) | missions split into lemmas, each attempt accepted only by a tool (the Lean kernel), as on prove2.me |

The last example is the expressibility check for machine-verified acceptance: prove2.me splits results into missions and milestone lemmas and accepts a contribution only when the Lean kernel checks it. Here that is a `checking` state owned by a `tool` role. The principle carries over: every milestone should have the strongest machine-checkable acceptance available — tests and validation for code, the static validator for configurations, the refuter for specifications, a model checker for critical protocols.

## 14. Today's task-board as a playbook

[`default.playbook.json`](../examples/default.playbook.json) maps the current kernel (skill-project-management origin/main `4b0a75d6`):

| Today | Code | In the default playbook |
| --- | --- | --- |
| types epic, story, task, bug | `pkg/board/types.go:13-16` | `entities` |
| epic has no parent; story under epic; task and bug under story | `types.go:184-218` | `root`, `parents`, `contains` |
| ten statuses | `types.go:23-32` | states of `leaf` and `container` |
| done and closed are terminal and never exited | `pkg/board/escalation.go:105-107`, `pkg/board/terminal_status.go:16-21` | `terminal`, no `reopen` |
| container status aggregates children | `escalation.go:73`, `:140` | `derived_status` |
| a container closes only when every descendant is terminal | `pkg/board/validation.go:34-43` | guard `children.terminal` |
| forward statuses refused while blockers are active | `validation.go:46-58` | guard `blockers.clear` |
| task and bug must be estimated before development | `validation.go:60-68` | guard `estimate` |
| manual moves to development or to-review need an assignee; to-dev deliberately does not | `validation.go:73-90` | guard `assignee` with `manual_only` |
| spawned producers reach to-review only through handoff (checklist complete, `<ID>_results.md` stored) | `tools/board-cli/cmd/producer_handoff_guard.go:24`, `pkg/board/handoff.go:122`, `pkg/board/errors.go:26-44` | guard `handoff.evidence` |
| integrating is entered only through `accept_cr` | `pkg/board/consistency.go:166-167` | `via: accept_cr` |
| integrating is left only through the integration path | `consistency.go:169-172` | `locked`, `via: integrate` |
| done is refused while a Change Request exists | `consistency.go:128-141`, `:158` | guard `change_request.absent` |
| commit acknowledgement for story and bug when policy confirms | `pkg/board/version_control.go:89-123`, `:277-279` | guard `commit.acknowledged`, `policy.version_control` |
| rejection routes to to-dev | `tools/board-cli/cmd/change_request.go:753`, `:825`, `:856` | `via: reject_cr` |
| linked blocked items release to backlog when blockers finish | `pkg/board/blocked_release.go:8-24`, `:87` | `on: { event: blockers.terminal }` |
| roles, archetypes, start and end statuses | `internal/spawn/role.go:71-90`, `:102-103`; `.roles/*/role.md` | roles and state owners |
| orchestrator (and coordinator) has no lifecycle statuses | `pkg/board/goal_role.go:10-11`, `:25-33` | `root` |
| reviewers start in reviewing | `goal_role.go:36-45` | owner of `reviewing` |
| review policy defaults to required | `types.go:106-133` | `policy.review` |
| task class | `types.go:136-141` | field `task_class` |
| rework loop detection floor 3 | `internal/changerequest/loopdetect/loopdetect.go:129` | `policy.loop_detection` |

Open points of the mapping:

1. The kernel has no transition table: any non-terminal status may be set when its gates pass. The default reproduces that with `from: "*"`; stricter workflows apply only to new types.
2. The aggregation order of container statuses (`escalation.go:11-100`) stays kernel code; the configuration only declares `derived_status`.
3. `one_of … by assignment.role` is new: today the role is chosen at spawn time and only recorded in the assignee string.
4. `closed` carries no resolution today.
5. Dependency links (`blocked_by`) are kernel data; the configuration refers to them only through `blockers.clear` and the `blockers.terminal` event.
6. Loop detection counts rework across Change Request revisions; it is not the same as a transition `limit` and is kept as policy.
7. `defer_cr` and `withdraw_cr` are not mapped yet.
8. The review policies `light` and `none` change reviewer routing, not the workflow; how they appear in a playbook is open.

## 15. Integration plan

| Step | Delivers |
| --- | --- |
| PC0 | this draft, the schema, the examples; the refuter's first run is on this document |
| PC1 | the module: schema, parser, resolver (`extends`, `uses`, slots), validator CLI; a thin seam in task-board that loads `default.playbook.json` with byte-identical behaviour or documented differences |
| PC2 | role skills and the `curator-playbook-dev-cycle` package; task-board resolves roles from installed skills before `.roles` and `~/.roles`; `setup.sh` stops copying `~/.roles` (this is roadmap milestone M1) |
| PC3 | custom types and workflows live; the delivery module seams; the demo playbooks |

Details: [`track.md`](track.md).

## 16. Names

Specifications use technical terms. User-facing material uses the Apiary names *(decided)*: Hive = one node's agent system, Comb = a process, Keeper = the main loop, Bee = the agent loop, Waggle = the messaging protocol; a playbook keeps its name. The ecosystem wiki records the mapping; this is the only place in the specification that mentions it.

## 17. Decisions and open questions

Decided on 2026-09-24: the file and its format; the four sections; one owner per state with helpers; owner kinds; guards and actions from modules with a delivery module for development; `extends` and `uses` with slots; roles as skills and playbooks as skill collections installed by one Skillfile selector; package names `curator-playbook-<purpose>`; needs and `on_unmet`; the static validator; the generality vectors; the refuter; the default playbook as the PC1 target.

Open:

1. `data/role.json` or skill frontmatter for role metadata.
2. The template lookup rule (by name in installed skills).
3. Merge details for `extends` (list replacement, removal of states).
4. The `dependencies.skills` directory proposal to curator-spec and the release of Skillfile schema 2.
5. Operator directory for human `who` names.
6. How review policies `light` and `none` are expressed.
7. The open points of the mapping in §14.

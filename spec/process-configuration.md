# Process configuration: specification

Status: **DRAFT v2**, 2026-09-24. Not normative. v2 answers the first refuter run on v1 ([`../reviews/2026-09-24-refuter-v0.md`](../reviews/2026-09-24-refuter-v0.md), 36 findings) with the operator's decisions of the same day; §18 maps every finding to its resolution or to an open question. Statements marked *decided (2026-09-24)* were confirmed by the operator; everything else is *proposed*. Roadmap: `wiki/roadmap/ecosystem-roadmap.md` (process-configuration line). Track: [`track.md`](track.md). Schema: [`../schema/playbook-v1.schema.json`](../schema/playbook-v1.schema.json). Examples: [`../examples/`](../examples/).

Supersedes the roles-and-routes model of `skill-project-management/.specs/drafts/playbooks.md`: the core object is no longer a list of roles with routes between them but entity types with workflows whose states are owned by roles. Related: `skill-project-management/.specs/drafts/agent-selection.md` (selection layers; §9 amends its §3 and §8), curator-spec `protocol/skillfile-sources.md` (Skillfile schema 2), `protocol/core.md` §3 and §4.4 (execution boundary, skill dependencies), `protocol/environments.md` (package-influence boundary), `waggle/spec/waggle.md` (approver signatures, hand-off offers, scope leases).

---

## 0. The design on one screen

- A project describes how it works in one file, `Playbook.json`, in its root beside `Skillfile.json`. *decided*
- The file has four core parts: **roles** (aliases bound to role skills, people, tools or other processes), **entities** (element types, nesting, fields), **workflows** (states and transitions) and **policy**. *decided*
- **Each element has exactly one owner in each state**; a state may admit several roles and assignment picks one; helpers work for the owner. *decided*
- **Playbooks are data only.** They never name a program, script, webhook or URL. Everything that executes is a skill, installed and audited through Curator; a tool state runs a command exported by an installed skill. Schedules and automatic spawns a package proposes take effect only when the project enables them. *decided*
- **Authority stays outside the model.** An agent run never moves work out of a human-owned state, never records a human approval and never posts a tool event. Risky human acts need a waggle approver signature. *decided*
- A project **extends** one published playbook and writes only its differences, and **composes** several, binding their slots; its `extends`/`uses` is its list of admitted playbooks. *decided*
- **Roles are skills.** A published playbook is a repository that is a collection of skills: the root orchestrator, whose data carries the template, and one skill per role. One Skillfile schema 2 selector installs the whole process. *decided*
- Roles state **needs**, never models or efforts; the operator's private file binds models and effort; only the project relaxes a hard need permanently. *decided*
- A static **validator** (`playbook validate`) checks a configuration before anything runs, split into portable checks and machine checks. *decided*
- Development machinery is a **delivery module** whose invariants hold whatever a playbook says. *decided*
- Until the main loop exists, `task-board tick` fires timers, schedules and expiries. *decided*
- Today's task-board is itself a playbook ([`default.playbook.json`](../examples/default.playbook.json)); PC1 makes the kernel read it with identical behaviour or documented differences. *decided*

## 1. Goals and non-goals

Goals:

1. Any process — software delivery, a restaurant's purchasing, formalizing a proof, a family calendar — is configuration, not kernel code.
2. The same file works for every teammate, whatever harnesses and models each one has.
3. A process can be read, validated and eventually drawn and edited by a visual constructor without losing anything.
4. Processes compose: the output of one can be the input of another, and a cadence (Scrum) can sit over a delivery process.
5. A configuration can never make an installed package execute something the project did not admit.

Non-goals of this revision: the orchestrator's agent loop; an always-on scheduler (the main loop, later; until then §6.9); the goal system itself (`board-goal-v2` stays in task-board; the configuration only names each orchestrator's goal type, §6.7); the storage format of board elements.

## 2. Files and responsibilities

### 2.1 `Playbook.json` and `Skillfile.json`: two files, two jobs *(decided)*

| File | Answers | Read by | Carries |
| --- | --- | --- | --- |
| `Skillfile.json` | what is installed | Curator | skill names, sources, versions (tags, revisions) |
| `Playbook.json` | how the project works | the board (task-board today) and the validator | admitted playbooks, role aliases bound to skill names, tool bindings, people, entities, workflows, policy |

`Playbook.json` never carries a version, a URL or a source; `Skillfile.json` never carries a role, a state or a transition. They meet only through skill names. When a role or tool names a skill that is not installed, the validator reports it with the fix: the `curator add` invocation for a named skill, or the Skillfile selector to add (§8). Installing (Skillfile) and using (Playbook) are two edits with two meanings, like a dependency in `go.mod` and its `import`; a helper such as `playbook bind <alias> <skill>` can make both.

Without Curator the same files work: skills are copied into the harness skill directories by hand and `Playbook.json` does not change (§8.4).

### 2.2 Format *(decided)*

- JSON with a published JSON Schema: every file starts with `"$schema"` and `"schema_version": 1`. Editors get validation and completion; a constructor reads and writes the file without loss.
- Canonical JSON (the canonical form Curator uses for digests) defines the semantic digest of a resolved playbook. The `ui` blocks, `description` fields and `x-` keys are excluded from the digest.
- Comments are `description` fields on any element; a constructor shows them.
- Every element may carry a `ui` block (positions, colours, collapsed groups) for a visual constructor. The kernel ignores it.
- Keys starting with `x-` are extensions; the kernel ignores them.
- Names of roles, entity types, workflows and states are kebab-case identifiers; field names are snake_case (`task_class`), matching today's element fields.

### 2.3 The operator's private file *(proposed; name pending the operator's confirmation)*

Concrete models, harness bindings and effort live in the operator's private file, `~/.curator/models.toml`. It is TOML, like `runtimes.toml` and `engines.toml`, because people edit it by hand; it is never committed.

```toml
[bindings]
"dev-cycle/developer" = { run = "codex/gpt-6-luna", effort = "max" }
"*/refuter"           = { run = "claude/claude-opus-5-5", effort = "high" }
"@reviewer"           = { run = "claude/claude-opus-5-5", effort = "medium..high" }
"@producer"           = { run = "codex/gpt-6-luna", effort = "high..max" }

[allow]
models = ["codex/*", "claude/claude-opus-5-5", "local/qwen3.6-27b"]

[windows]
"local/qwen3.6-27b" = "22:00-07:00"
```

- One `[bindings]` table; keys go from specific to general: `<playbook>/<role>`, then `*/<role>`, then `@<capability>`. The most specific matching key wins.
- `run` is `<runtime>/<model>`; `effort` is one value or a range (`"medium..high"`). A project floor (`policy.effort_floor`, §6.8) narrows the range from below; the selection records the effort it used.
- `[allow]` narrows which models may run at all; `[windows]` limits when.
- **No file is not an error.** The resolver then picks the best admitted model that satisfies the role's needs among harnesses that are installed and logged in (the registry lineup plus availability), prints the choice and why, and suggests pinning it. Every choice, from the file or not, must still satisfy the role's needs (§9), else `on_unmet` applies.
- Tool bindings and people are not in this file: they are project facts and live in `Playbook.json` (§4.2, §8.5).

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
  "tools": { },
  "people": { },
  "events": { },
  "entities": { },
  "workflows": { },
  "schedules": [ ],
  "policy": { },
  "ui": { }
}
```

`name` is required for a published template and optional in a project file. `extends` and `uses` are covered in §7, `slots` and `bind` in §7.2, `tools` in §8.5, `people` in §4.2, `events` and `schedules` in §6.9. Keys that only a project may set (`people`, `policy.enable`, `policy.effort_floor`, `policy.signatures`, tool bindings to other packages' skills) are ignored with a warning when they appear in a template.

## 4. Roles

### 4.1 Role aliases *(decided)*

`roles` maps a project-level alias to what performs the work. Workflow states name aliases, never skills, so replacing who does a job is one line.

```json
"roles": {
  "developer": { "skill": "dev-developer", "capability": "producer", "needs": { "prefer": { "publisher": "openai" } } },
  "reviewer":  { "skill": "dev-reviewer", "capability": "reviewer",
                 "needs": { "require": { "not_same_publisher_as": "developer" }, "on_unmet": "ask" } },
  "manager":   { "kind": "human", "who": "cafe-managers", "capability": "approver" },
  "lean":      { "kind": "tool", "tool": "lean.check", "capability": "checker" },
  "fixers":    { "kind": "process", "process": "fix-process" }
}
```

### 4.2 Owner kinds and who may move work *(decided)*

| `kind` | Performs the work | Bound by | May move work out of a state it owns |
| --- | --- | --- | --- |
| `agent` (default) | an agent running the role skill | `skill` | the spawned run of that role, and the root orchestrator for routing; guards still apply |
| `human` | a person or a group | `who`: a group of the project's `people` map, whose members are waggle roster principals | only a human actor: the operator's own CLI action outside any run, or a waggle approver signature from a member principal; acts that policy marks risky require the signature |
| `tool` | a command exported by an installed skill (§8.5) | `tool`: a name the `tools` map binds | only the kernel, from its own invocation of the bound command: `tool.passed` / `tool.failed` |
| `process` | another process | `process`: a process slot the project binds (§6.5) | only the receiving process, whose result returns as `process.returned` |

**Actor classes.** Every transition records its actor: `run` (a spawned run with its role), `session` (a hosted orchestrator session), `human` (a CLI action outside any run and session, or a waggle signature), `kernel` (automatic transitions, tool invocation, timers) or `event` (a signed external event, §6.9). An agent run or session never fires a transition out of a human-owned state, never records a human approval and never posts a tool or external event. `people` maps groups to principals:

```json
"people": { "cafe-managers": { "principals": ["op:alice@cafe", "op:bob@cafe"] } }
```

`policy.signatures` (§6.8) names which human acts need a waggle approver signature and at what key assurance (`software`, `hardware-bound`, `attested`, waggle §4.5): approvals, spending, halts, relaxing a requirement, or every human-owned transition.

Enforcement grows with the stack. Today the only actor signal the kernel has is the `TASK_BOARD_RUN_ID` convention, which the producer guard itself calls "not an operator authorization boundary" (task-board `tools/board-cli/cmd/producer_handoff_guard.go:12-15`). Until the session host marks hosted sessions and waggle signs actions, refusals for `run` actors are a workflow guardrail and a human act is trusted only when it carries a signature. Transitions that policy marks risky therefore require the signature from the start. *decided*

### 4.3 Capabilities

`capability` is the archetype guards reason about. Agents: `orchestrator`, `producer`, `reviewer`, `analyst`, `tester`; humans: `approver`; tools: `checker`. Custom capabilities use an `x-` prefix. The schema refuses a capability on the wrong kind, so a tool cannot pose as a reviewer. Today's archetypes map directly: implementer → producer, analyst → analyst, reviewer → reviewer, tester → tester (task-board `tools/board-cli/internal/spawn/role.go:60-90`).

A guard that needs a capability checks the actor that produced its evidence, not the owner of the source state: `delivery.cr-accepted` and `verdict` need a verdict recorded by a reviewer or an approver. The validator requires that some state of the workflow is owned by a role with that capability, so every verdict has a producer (§10).

### 4.4 The root role *(decided)*

`root` names the orchestrator of the process. Every resolved process has one; in a published playbook it is the skill that carries the template (§8). In a composition the project names the root (§7.2); the other playbooks' orchestrators keep owning their own states and goals (§6.7). Needs on the root apply only to spawned sub-orchestrators: a primary orchestrator session starts through `curator run`, whose model and effort precedence is closed (curator-spec Decision 0021 §1, §4; Decision 0013 §6.2).

## 5. Entities

### 5.1 Types, nesting and creation *(decided)*

```json
"entities": {
  "epic":  { "root": true, "contains": ["story", "research-task", "decision"], "workflow": "epic" },
  "story": { "parents": ["epic"], "contains": ["dev-task", "research-task", "design-task"], "workflow": "story" }
}
```

- `root: true` lets a type stand at the top; any other type needs `parents`. `contains` and `parents` must agree. Containment is a tree: an element has one parent.
- `workflow` names the workflow the type runs; the workflow's `initial` state is where every element of the type starts.
- **Creation is entry into `initial`** and is subject to that state's `requires`. The `create` action and schedules carry field values; a created element is placed under the nearest ancestor, starting with the current element, whose type contains the created type, or under an explicit `parent`. The validator refuses a `create` no ancestor can hold (§10).
- **`derived_status: true`** means the kernel aggregates the type's status from its children in a fixed order (today `pkg/board/escalation.go:11-100`). Aggregated moves are not transitions: they ignore `terminal`, `locked` and guards, and they may reopen a container from `done` or `closed` when a child becomes active again (`escalation.go:151-162, 198-200`) and move it into and out of `integrating` (`escalation.go:65-68`). A blocked container is left alone. Declared transitions on a derived type are the direct moves people and orchestrators make. Reachability checks do not apply to derived types.

### 5.2 Fields *(decided)*

Non-development processes need data: a purchase order has a supplier, lines and an amount. Types declare typed fields:

```json
"fields": {
  "supplier": { "type": "string", "required": true },
  "amount":   { "type": "money", "currency": "EUR", "required": true },
  "due":      { "type": "date" },
  "severity": { "type": "enum", "values": ["low", "medium", "high", "critical"] },
  "invoice":  { "type": "ref", "ref": "invoice" },
  "low_items":{ "type": "list", "of": "string" }
}
```

Types: `string`, `text`, `integer`, `number`, `boolean`, `date`, `datetime`, `duration`, `money`, `enum`, `ref`, `list`. **`required: true` means the field must be set before the element leaves its initial state**; creation may leave it empty, so a draft can be filled in its first state. A `ref` names an entity type or a slot.

### 5.3 Estimates *(decided)*

Estimates have named dimensions, so complexity and duration are separate:

```json
"estimates": [ { "name": "complexity", "scale": "fibonacci" }, { "name": "duration", "scale": "hours" } ]
```

Scales: `fibonacci`, `points`, `tshirt`, `hours`, `days`. Capacity rules (§6.8) sum one dimension. Estimate dimensions are unrelated to reasoning effort, which never appears in a playbook (§9).

### 5.4 Privacy *(proposed)*

`privacy` on a type (`inherit`, `private`, `team`, `public`) and `policy.privacy` (`visibility`, `redact_fields`) describe views only. Board elements are committed files, so redaction in views does not remove data from history; real privacy needs a separate store or encryption. Until then privacy is not a conformance vector (§11).

## 6. Workflows

### 6.1 States *(decided)*

```json
"states": {
  "open":        { "owner": "orchestrator", "requires": ["description"] },
  "development": { "owner": "developer" },
  "review":      { "owner": "reviewer" },
  "estimating":  { "owner": "facilitator", "helpers": [ { "role": "estimator", "min": 3, "blind": true } ] },
  "accepted":    { "owner": "orchestrator", "locked": true },
  "done":        { "terminal": true },
  "closed":      { "terminal": true, "resolution": ["wont-do", "duplicate", "obsolete"] }
}
```

- **Each element has exactly one owner in each state; a state may admit several roles and assignment picks one.** The owner is accountable and is what authority checks use. *decided*
- A state that admits several roles uses `{ "one_of": [...], "by": "assignment.role" }`, or `{ "one_of": { "capability": [...] }, "by": "assignment.role" }` for every installed role with those capabilities (today's open-ended roles). The element records the structured role alias on assignment or spawn; until it has one, the root orchestrator owns it. *decided*
- **Helpers** are roles the owner may engage; they never own the state. Planning poker: owner `facilitator`, helpers `estimator`, answered blind and revealed by the owner. *decided*
- `requires` lists entry conditions: guards applied to every transition into the state and to creation when the state is `initial`.
- `terminal` states have no owner. Leaving one needs a transition marked `reopen: true` (aggregation, §5.1, is not a transition).
- `resolution: true` requires a resolution on entry; a list restricts the values. Automatic entries carry theirs: `resolution` on the transition, or on a `limit` redirect.
- `locked: true` means only transitions that name the state in `from` leave it; `from: "*"` never matches it.

Two rules keep ownership clear *(decided)*:

1. Two different results are two states: "built it" and "checked it" are `development` and `review`, never one state with two owners.
2. Parallel work is child elements plus a join guard (`children.terminal`, `children.in`) on the parent's transition, not parallel states.

### 6.2 Transitions *(decided)*

```json
"transitions": [
  { "from": "open", "to": "development", "guards": ["estimate", "assignee", "blockers.clear"], "actions": ["spawn.owner"] },
  { "from": "review", "to": "development", "guards": [ { "guard": "verdict", "is": "changes-requested" } ],
    "limit": { "max": 3, "then": "escalated" } },
  { "from": "review", "to": "accepted", "via": "accept_cr", "guards": ["delivery.cr-accepted"] },
  { "from": "sent", "to": "escalated", "on": { "after": "PT15M" } },
  { "from": "invoiced", "to": "paid", "on": { "event": "payment.received" } },
  { "from": "*", "to": "closed" }
]
```

- `from` is a state, a list of states, or `*` (every non-terminal, non-locked state).
- A transition is identified by `(from, to, via)`. `via` names the one command that performs that triple (`accept_cr`, `integrate`); it does not restrict other triples into the same state, so a manual route and a command route may coexist. An explicit transition shadows the `*` expansion for the same triple.
- `guards` must all pass; `actions` run after the move. Guard objects may carry `applies_to` (`actor`: `run`, `human`, `any`; `except_capability`) to scope a guard to one actor class, as today's handoff evidence gate is.
- `on` makes the transition automatic: on an `event` or `after` a duration in the source state. The kernel fires it; §6.9 says when.
- `limit` bounds a loop: after `max` firings for one element the transition leads to `then` instead, recording `resolution` when `then` requires one.
- Determinism: the validator refuses two automatic transitions from the same state on the same trigger with the same guards, and two transitions with the same triple.

### 6.3 Guards, actions and events come from modules *(decided)*

A guard or action is a name, or an object whose `guard`/`action` key names it and whose other keys are its arguments; the schema keeps the two shapes apart. Names of module hooks carry the module prefix. The validator refuses any guard, action, event or `via` command that no loaded module provides and no `events` declaration names.

| Provider | Guards | Actions | Events | `via` commands |
| --- | --- | --- | --- | --- |
| kernel | `description`, `estimate` (`grandfather_legacy`), `estimates.present` (`scope`, `dimensions`), `assignee` (`manual_only`), `blockers.clear`, `children.terminal` (`types`), `children.in` (`states`), `children.any-in` (`states`), `members.terminal-or-carried`, `checklist.complete`, `outcome.present`, `fields.present` (`fields`), `approvals` (`role`, `min`), `verdict` (`is`), `capacity` (`dimension`, `scope`), `process.result` (`in`) | `spawn.owner`, `assign`, `create` (`type`, `per`, `fields`, `parent`), `handoff` (`to`, `create`, `link_field`, `from_outcome`), `escalate`, `notify` (to an owner or a group), `record.verdict`, `compute.velocity` (`dimension`) | `blockers.terminal`, `children.changed`, `process.returned`, `tool.passed`, `tool.failed` | — |
| delivery module (development only, §6.10) | `delivery.cr-published`, `delivery.cr-accepted`, `delivery.cr-absent`, `delivery.commit-acknowledged`, `delivery.integration-evidence`, `delivery.external-integration-evidence`, `delivery.restore-authorized`, `delivery.rollback-authorized` | `delivery.integrate`, `delivery.close-checkpointed-siblings` | `delivery.integrated`, `delivery.integration-failed`, `delivery.rework-started` | `accept_cr`, `reject_cr`, `integrate`, `integrate_external`, `restore_done`, `landed_rework`, `invalidate`, `converge` |
| timers | — | — | `after` durations | — |
| a playbook's `events` | — | — | declared external events (§6.9) | — |

Verdicts are recorded by reviewers and approvers: `accepted`, `changes-requested`, `rejected`, and the refuter's `contradiction`, `no-contradiction`, `human-decision-needed` (§12). `tool.passed` and `tool.failed` arise only from a state owned by a tool role (§8.5).

### 6.4 Approvals and quorum *(decided)*

`{ "guard": "approvals", "role": "manager", "min": 1 }` needs approvals from that role; `min` above one is quorum voting and needs distinct principals. An approval is a board record bound to the approving principal. It is a human act (§4.2): the operator's own CLI action counts only where policy does not require a signature; otherwise it is a waggle approver signature at the configured assurance. The validator flags `min` greater than the number of principals in the role's group.

### 6.5 Hand-off between processes *(decided)*

A state owned by a `process` role is worked in another process. A process is a playbook instance on a board: a member of the project's composition, or a process on another board reached through waggle. The playbook names a process **slot**; the project binds it (§7.2). The `handoff` action offers the work:

```json
{ "from": "verifying", "to": "fixing", "guards": [ { "guard": "verdict", "is": "accepted" } ],
  "actions": [ { "action": "handoff", "to": "fixers", "create": "fix-target", "link_field": "fix" } ] },
{ "from": "fixing", "to": "fixed", "on": { "event": "process.returned" }, "guards": [ { "guard": "process.result", "in": ["done"] } ] }
```

- Between orchestrators the hand-off is a waggle offer the receiver accepts (`coord.handoff-offer`, `coord.handoff-accept`); the receiving orchestrator creates the element, places it in its own hierarchy and sets the link. Writes into another orchestrator's leased scope go through that orchestrator.
- Within one composition under one orchestrator the kernel creates the element directly, placed by the rule of §5.1.
- `process.returned` carries the final state of the received element; `process.result` guards on it.

### 6.6 Schedules *(decided)*

Recurring work is created by named schedules:

```json
"schedules": [ { "name": "weekly-inventory", "every": "P1W", "create": { "type": "inventory-check", "title": "Weekly inventory" } } ]
```

`every` is an ISO 8601 duration; `cron` is the alternative. A schedule a package proposes takes effect only when the project enables it (`policy.enable.schedules`, §8.5). `task-board tick` fires schedules (§6.9).

### 6.7 Short goals *(decided)*

Orchestrators lose the thread on large goals; the weaker the model, the shorter the step must be. `policy.goals.orchestrator_goal` names the type whose current instance is an orchestrator's goal — a story, a milestone or a sprint. In a composition it is a map from orchestrator role to type, so each orchestrator has its own. The long-term goal lives in the hierarchy and each orchestrator works one bounded step at a time. The goal ledger itself stays in task-board.

### 6.8 Policy

| Key | Meaning | Set by |
| --- | --- | --- |
| `review.default` | `required`, `light` or `none` for new elements (today's review policy) | template or project |
| `capacity` | per container type, the maximum sum of one estimate dimension (a sprint of 100 complexity points) | template or project |
| `privacy` | view visibility and redacted fields (§5.4) | template or project |
| `goals` | §6.7 | template or project; the project wins in a composition |
| `effort_floor` | per role, the lowest effort a run may use (§9) | project only |
| `enable` | `schedules` (all or named), `auto_spawn`, `relaxations` (§8.5, §9) | project only |
| `signatures` | which human acts need a waggle approver signature, and the minimum key assurance (§4.2) | project only |
| `delivery.version_control` | `confirm`, `desired_commit_time` (required when `confirm`), `commit_time_policy` | template or project |
| `delivery.loop_detection` | rework threshold `max(floor, factor × median)` | template or project |

### 6.9 Timers, schedules and external events before the main loop *(decided)*

Time-based transitions (SLAs, `after`), schedules and approval expiries need something that watches the clock. The main loop will; until it exists, an idempotent `task-board tick` does:

- it is called every minute by the session host or by launchd;
- it fires due `after` transitions, due schedules and expiries of `ask` approvals, recording actor `kernel`;
- it takes a board lease first, so on a shared board exactly one teammate's tick fires each item;
- it is safe to run twice: every firing is keyed by element, transition and due time.

External events enter through `task-board event post <event> --element <id>` with a signed payload. A playbook declares the events it reacts to and who may sign them:

```json
"events": { "payment.received": { "signers": ["tool:payment-gateway@cafe"] } }
```

An unsigned or wrongly signed event is refused and logged. Webhooks need a service the ecosystem does not run yet; a bridge that receives one posts the signed event.

### 6.10 Delivery invariants *(decided)*

For every type whose workflow uses the delivery module, these hold whatever the playbook declares; a playbook may add stricter checks, never weaker ones:

1. No success terminal before integration: an element under an active Change Request reaches `done` only through integration (`integrate`, `integrate_external`, `restore_done`, or `delivery.integrated`); inline work without a Change Request reaches it only with `delivery.cr-absent`.
2. No self-review: the run that produced a revision cannot record its acceptance.
3. Review only after a published Change Request revision; integration only after an accepted immutable revision.
4. Leaving integration otherwise than to done needs a rollback authorization.

The validator refuses a delivery workflow with a declared route into `done` that bypasses rule 1. `dev-cycle` keeps accepted leaves in a non-terminal `accepted` state until their story integrates, as `integrating` does today, so it neither deadlocks nor lets a task finish before integration.

## 7. Extension and composition

### 7.1 `extends`: only the differences *(decided)*

A project starts from one published playbook and writes only what differs. When the playbook publishes a new version, the project's differences stay and everything else updates.

```json
{
  "extends": "dev-cycle",
  "roles": {
    "developer": { "skill": "ios-developer" },
    "tester":    { "skill": "ios-tester", "capability": "tester" }
  },
  "workflows": { "dev": {
    "states": { "qa": { "owner": "tester" } },
    "transitions": [
      { "from": "development", "to": "review", "remove": true },
      { "from": "development", "to": "qa", "guards": ["outcome.present"], "actions": ["spawn.owner"] },
      { "from": "qa", "to": "review", "guards": ["outcome.present", "delivery.cr-published"], "actions": ["spawn.owner"] }
    ] } }
}
```

Here an iOS project swaps the developer skill and puts a tester's QA pass before review.

Merge rules *(proposed)*:

1. One parent. Chains are allowed; cycles are refused. The object form `{ "name": …, "package": "relux-works/curator-playbook-dev-cycle@v1.0.0" }` pins the parent's package identity, which the resolver checks against the installed lock; an extending template must use it.
2. Maps merge by key, recursively.
3. Transitions are keyed by `(from, to, via)`: a child transition with the same key replaces the parent's; `remove: true` deletes it. A state with `remove: true` deletes the state and every transition touching it. Removing a key the parent does not have is an error, so drift after a parent update is loud.
4. Lists other than transitions (`contains`, `parents`, `helpers`, guards, actions) are replaced as a whole.
5. The validator runs on the resolved result (§10) and reports which file introduced each error.
6. The board records the resolved template's package identity and semantic digest; teammates who resolve a different digest get a refusal naming both. A template update that removes or renames states declares `migrations` (old state → new state) or the validator lists the stranded elements and refuses.

### 7.2 `uses` and slots: composition *(decided)*

A project can run several playbooks as one process. A playbook that refers to a type or a process it does not define declares a **slot**; the project binds it:

```json
{
  "uses": ["dev-cycle", "scrum"],
  "bind": { "scrum.work-item": "story", "scrum.delivery": "dev-cycle" },
  "root": "scrum-orchestrator",
  "entities": { "story": { "estimates": [ { "name": "complexity", "scale": "fibonacci" }, { "name": "duration", "scale": "days" } ] } }
}
```

- A type slot may state a contract (`requires`: estimate dimensions, fields, terminal states); the validator checks the bound type satisfies it. A process slot (`kind: "process"`) is bound to a playbook of the composition or to another board.
- Names must be unique across the composition; a clash is resolved by qualifying with the playbook name (`dev-cycle.reviewer`). Installed skill names cannot be qualified (Skillfile schema 2 refuses repeated names), so published packages prefix their skills (`dev-`, `hunt-`, `scrum-`).
- The project overrides members of used playbooks by name (qualified when ambiguous). Policy keys that two used playbooks both set must be set by the project; the validator refuses the clash otherwise.
- **Scrum is a cadence layer, not a base to extend.** It decides which work is selected and when; dev-cycle decides how an item is built. Containment stays a tree: a story stays under its epic, and a sprint holds its stories by **membership link** (`members`), at most one open sprint per story; carrying a story over relinks it.
- Orchestrators map onto waggle scope leases: each orchestrator of a composition leases the subtrees it owns; a transition into a state owned by another orchestrator's role is an offer, not a write (§6.5).

## 8. Distribution: roles are skills *(decided)*

### 8.1 A published playbook is a collection of skills

```
curator-playbook-dev-cycle/
  README.md
  skills/
    orchestrator/        SKILL.md (name: dev-orchestrator) — the root role
      data/playbook.json — the process template (name: dev-cycle)
    developer/           SKILL.md (name: dev-developer)
      data/role.json     — capability, default needs, definition-of-done items, skills the role needs
    reviewer/            SKILL.md (name: dev-reviewer)
    refuter/             SKILL.md (name: dev-refuter)
    researcher/ designer/
```

Every role directory is a real skill: instructions in `SKILL.md`, optional `references/`, `assets/`, `data/` — the roots Curator installs. The template lives in the orchestrator skill because the root role owns the process. Packages are named `curator-playbook-<purpose>`. Role metadata sits in `data/role.json`; when the project's `Playbook.json` role entry disagrees with it, the project entry wins. `data/role.json` lists the other skills the role needs, as today's `.roles/<role>/skills.md` does.

### 8.2 One Skillfile entry installs the process

Curator's Skillfile schema 2 (curator-spec `protocol/skillfile-sources.md`) declares repository sources and selects skills from them: an individual `{name, from, directory}` selector or a collection `{from, directory, include, exclude?}` selector over immediate child directories. It exists in Curator behind a flag; PC2 uses it (§15).

```json
{
  "schema_version": 2,
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

The first selector installs the orchestrator and every role of dev-cycle; the second adds a reusable role from another repository, which `Playbook.json` then binds to an alias. The board resolves `"extends": "dev-cycle"` to the installed skill whose `data/playbook.json` has that name. Template names must be unique among installed skills in the effective scope (project before global); two candidates are an error, and only playbooks the project admits (§8.5) are candidates.

### 8.3 The Curator work *(decided)*

Two changes belong to Curator, not to this module:

1. Release Skillfile schema 2 from its flag.
2. `dependencies.skills` entries in a skill manifest (curator-spec `protocol/core.md` §4.4: `git`, exact `ref`, activation `mode`, command narrowing) gain the same `directory` selection as schema 2 individual selectors, with the same containment rules, so a playbook can depend on a role that lives in a subfolder of another repository.

### 8.4 Without Curator

The same directories are copied into the harness skill directories (`.claude/skills`, `.codex/skills`, `agents/skills`). The resolver finds role skills and templates there by name; nothing in `Playbook.json` changes. The machine checks (§10) warn that the skills are unaudited, and tool commands of unaudited skills run only when the operator confirms them.

### 8.5 Playbooks are data *(decided)*

A playbook never names a program, script, webhook or URL. This keeps Curator's boundary intact one layer up: "Profiles are data end to end. No file in a package snapshot is executed, sourced, or interpreted as configuration for the manager itself" (curator-spec `protocol/environments.md:94-98`); "A profile chooses what the context says, never how a process is launched. This is the package-influence boundary of the core execution policies" (`environments.md:3144-3146`); a manager "MUST NOT transfer execution control to a package- or repository-selected program, argument vector, environment, hook" (`protocol/core.md:85-91`).

- **Tools are skills.** A tool role names a tool; `tools` binds the name to a command exported by an installed skill, which Curator audits, signs, locks and asks approval for on change like any skill. The kernel runs only such exported commands, never a string from a playbook.

  ```json
  "tools": { "lean.check": { "skill": "proof-lean-kernel", "command": "check" } }
  ```

  A published template may bind a tool only to a skill of its own package; every other binding comes from the project file.
- **Side effects are proposals.** Schedules (§6.6), `spawn.owner` actions, and `fallback`/`degrade` relaxations (§9) that a package declares take effect only when the project enables them (`policy.enable`).
- **Admission.** The project's `extends` and `uses` are its list of admitted playbooks. A template outside that list is never resolved, whatever is installed.

## 9. Model needs and unmet needs *(decided)*

A committed file must not break for a teammate with other harnesses, so roles state needs, never model ids and never efforts:

- `require` — hard: the role cannot run without it.
- `prefer` — soft: ranking only.
- Terms: `publisher`, `family`, `capability`, `local`, `not_same_publisher_as` (independent review).

Effort is not a need. It lives in the operator file (§2.3); a project may set a floor per role (`policy.effort_floor`), and the operator chooses within it. This keeps agent-selection §3 ("neither names a model id, runtime id, effort, …").

When the operator's bindings cannot meet a `require`, `on_unmet` decides:

| `on_unmet` | Behaviour |
| --- | --- |
| `block` (default for `require`) | refuse, naming the layer that emptied the candidate set |
| `fallback` | try the ordered `fallback` needs; active only when the project enables relaxations |
| `ask` | ask a person for a temporary exception (below) |
| `degrade` | run on the best available through a statically declared `degraded_state` (a review state with its own owner); active only when the project enables relaxations; refused for independence requirements, because more reviews from the same publisher cannot restore independence |

**Who relaxes a hard requirement** *(decided in principle; details open)*:

- Permanently: only a reviewed change to the project's `Playbook.json` (it can drop the requirement or change `on_unmet`). This amends agent-selection §8 ("only the project can waive") by adding the temporary path below.
- Temporarily: through `ask`, an operator approves an exception for **their own runs only**, scoped to an element subtree (`element`, the nearest ancestor of a type, or `project`) and time-limited. The exception is recorded on the board as a `decision` bound to the approving principal and never reaches a teammate's runs. `policy.signatures` may require a waggle signature for it.

Example: the reviewer requires a publisher different from the developer's; a teammate has only one publisher; `ask` fires; the teammate approves "same-publisher review under this epic, 30 days" for their own runs. Every selection records which layer supplied what and whether a fallback, an approval or a degradation applied.

Open: where exactly exception records live (a `decision` element or a dedicated record), how expiry interacts with a run already in flight, and how the operator file's `[allow]` interacts with an exception.

## 10. Static validation *(decided)*

`playbook validate` is a library and a CLI in this module, independent of task-board. task-board runs it when it loads a playbook; a visual constructor runs it on every edit. It resolves `extends` and `uses` first. Checks are split so that CI and every teammate get the same structural verdict:

**Portable checks** (depend only on the files):

| Check | Example error |
| --- | --- |
| undefined states | `workflow dev: transition review → qa: state qa is not defined` |
| initial and creation | `workflow dev: initial open is not a state` |
| reachability | `workflow dev: state escalated unreachable from open` (derived types exempt) |
| terminal reachability | `workflow dev: no path from review to a terminal state` |
| owners | `workflow dev, state review: owner tester is not a role` |
| capability producers | `workflow cafe.purchase, transition approval → draft: guard verdict has no reviewer or approver producing it` |
| capability per kind | enforced by the schema: a tool cannot be a reviewer |
| providers | `guard delivery.cr-publish: no loaded module provides it`; `event supplier.confirmd: not provided or declared` |
| tool events | `workflow attempt, state proving: tool.passed from a state no tool owns` |
| delivery invariants | `workflow dev: validation → done bypasses integration` |
| determinism | `workflow dev: two automatic transitions on delivery.integrated from accepted with the same guards` |
| resolutions | `workflow finding: limit redirect to rejected carries no resolution` |
| placement | `workflow retro: create work-item: no ancestor of retro contains it` |
| hand-off | `handoff to accounting: not a process role` |
| slot contracts | `bind scrum.work-item → story: story lacks estimate duration` |
| nesting | `entity story: parent sprint does not contain story` |
| loop limits | `transition review → development: limit.then escalated is not a state` |
| quorum | `approvals role manager min 3: group cafe-managers has 2 principals` |
| guard bypass | `workflow purchase: escalated → sent skips the approvals guard of approval → sent` |
| project-only keys | `template dev-cycle sets policy.enable: ignored` (warning) |

**Machine checks** (depend on what is installed and on the operator): skills installed and audited, tool commands exported by those skills, `people` principals present in the waggle roster, operator bindings able to satisfy each role's needs. They report typed "unbound on this machine" refusals and never change the portable verdict.

A throwaway prototype of most portable checks (undefined states, initial, reachability, terminal reachability, owners, capability producers, providers, tool events, delivery invariants, determinism, resolutions, placement, hand-off, slot contracts, loop limits) was run over [`../examples/`](../examples/): every v2 example passes, with warnings only for human groups a template leaves to the project; the same prototype flags the v1 examples (unprovided guards and events, the integration bypass, redirects without resolutions). All examples validate against the schema, and the schema refuses program strings, URLs, effort needs, tools posing as reviewers and action objects inside guards.

## 11. Generality vectors *(decided)*

The module ships a conformance suite. Each vector is a case the model must express without kernel changes; each demo playbook covers several.

| Vector | Covered by |
| --- | --- |
| custom typed fields | cafe, bug-hunt, formal missions |
| human owners | cafe (approvals, receiving), formal missions (captains), dev-cycle (escalation, decisions) |
| tool owners | cafe (supplier portal), formal missions (Lean kernel) |
| timers and SLA | cafe (supplier confirmation in 15 minutes), formal missions (stalled after 14 days) — fired by `task-board tick` |
| recurrence | cafe (weekly inventory, enabled by the project) |
| external events | cafe (`supplier.confirmed`, `payment.received`, signed) |
| hand-off between processes | bug-hunt → dev-cycle, cafe → accounting, scrum retro → delivery |
| parallel branches and join | cafe (delivery lines), dev-cycle (story waits for accepted leaves) |
| quorum voting | scrum (three blind estimators), `approvals` with `min` |
| loop limits with escalation | dev-cycle (rework), cafe (approval), formal missions (five failed checks) |
| resolutions and reopen | dev-cycle (`closed` resolutions, decision superseded), automatic resolutions on limit redirects |
| multi-dimensional estimates and capacity | scrum |

Process privacy returns to the list when a separate store exists (§5.4).

## 12. The refuter role and decision memory

A draft specification is challenged before it is approved *(decided)*:

- **Where.** A `challenge` state after a draft (dev-cycle's `spec` workflow and the epic's `hld`), owned by a `refuter` role skill.
- **Input.** The draft plus board memory: `decision` elements (statement, scope, status, links), the specifications in force and their invariants.
- **Method: proof by contradiction.** Assume the draft is implemented as written and derive consequences; look for conflicts with recorded decisions, internal contradictions (two acceptance criteria that cannot hold together), unstated assumptions and guards no role can satisfy.
- **Output: a typed verdict.** `no-contradiction` with the list of decisions checked, so the pass is auditable; `contradiction` with pairs of claim and decision; `human-decision-needed` when only a person can choose. The same spelling is used in the spec, the examples and the role.
- **Transitions.** `contradiction` → back to the draft; `human-decision-needed` → `decision-needed`, owned by a person, who records a new `decision`; `no-contradiction` → approved.

Agents are good at finding contradictions; what they lack is memory of what was decided. The board supplies it. The first refuter run, on v1 of this document, is [`../reviews/2026-09-24-refuter-v0.md`](../reviews/2026-09-24-refuter-v0.md).

## 13. Examples

| File | Shows |
| --- | --- |
| [`default.playbook.json`](../examples/default.playbook.json) | today's task-board behaviour (§14) |
| [`dev-cycle.playbook.json`](../examples/dev-cycle.playbook.json) | delivery with the delivery invariants, rework limits, refuter, decisions, needs and `ask` |
| [`bug-hunt.playbook.json`](../examples/bug-hunt.playbook.json) | proving findings and offering them to the fixing process through slots |
| [`scrum/scrum.playbook.json`](../examples/scrum/scrum.playbook.json), [`scrum/project.Playbook.json`](../examples/scrum/project.Playbook.json) | a cadence layer composed over dev-cycle; membership links; planning poker; capacity; project-only policy |
| [`cafe.playbook.json`](../examples/cafe.playbook.json) | fields, humans, a tool that is a skill command, timers, a named schedule, signed events, join, hand-off |
| [`prove2me-style.playbook.json`](../examples/prove2me-style.playbook.json) | missions split into lemmas, each attempt accepted only by a tool (the Lean kernel), as on prove2.me |

The last example is the expressibility check for machine-verified acceptance: prove2.me splits results into missions and milestone lemmas and accepts a contribution only when the Lean kernel checks it. Here that is a `checking` state owned by a `tool` role whose command a skill of the package exports. The principle carries over: every milestone should have the strongest machine-checkable acceptance available — tests and validation for code, the static validator for configurations, the refuter for specifications, a model checker for critical protocols.

## 14. Today's task-board as a playbook

[`default.playbook.json`](../examples/default.playbook.json) maps the current kernel (skill-project-management origin/main `4b0a75d6`). Every row is reproduced unless it says otherwise.

| Today | Code | In the default playbook |
| --- | --- | --- |
| types epic, story, task, bug | `pkg/board/types.go:13-16` | `entities` |
| epic has no parent; story under epic; task and bug under story | `types.go:184-218` | `root`, `parents`, `contains` |
| ten statuses | `types.go:23-32` | states of `leaf` and `container` |
| no transition table: any status may be set when its gates pass | the gates below | transitions `from: "*"` |
| leaves never leave done and closed | `pkg/board/escalation.go:105-107`, `pkg/board/terminal_status.go:16-21` | `terminal`, no `reopen` |
| containers aggregate from children, and aggregation reopens done and closed containers and moves them into and out of integrating; blocked containers are left alone | `escalation.go:65-68, 73, 140, 151-162, 198-200` | `derived_status` (§5.1) |
| container statuses may also be set directly; only the close is gated | `pkg/board/validation.go:34-43` | declared `*` transitions on `container` |
| forward statuses refused while blockers are active | `validation.go:46-58` | guard `blockers.clear` |
| task and bug must be estimated before development; legacy records without an estimate and idempotent writes pass | `validation.go:60-68` | guard `estimate` with `grandfather_legacy` |
| manual moves to development or to-review need an assignee; to-dev deliberately does not | `validation.go:73-90` | guard `assignee` with `manual_only` |
| every spawned run whose role ends in to-review, except reviewers, hands off with its checklist complete and `<ID>_results.md` stored — testers and analyst-archetype roles included | `tools/board-cli/cmd/producer_handoff_guard.go:71`, `pkg/board/handoff.go:122`, `pkg/board/errors.go:26-44` | guards `checklist.complete` and `outcome.present` with `applies_to: { actor: run, except_capability: [reviewer] }` |
| roles are open-ended: any installed role can be spawned | `tools/board-cli/internal/spawn/role.go:83-93, 303-335` | owners `{ one_of: { capability: [...] } }` |
| integrating is entered only through `accept_cr` | `pkg/board/consistency.go:166-167` | `via: accept_cr`, `locked` |
| integrating is left to done through integration, external integration or a restore, and to to-dev through a rollback (landed rework, invalidation, converge) | `consistency.go:169-172`; `tools/board-cli/internal/integration/start_landed_rework.go:67`, `invalidate.go:117-118`, `converge.go:426-427`, `restore_done.go:85` | `via` triples with `delivery.*-authorized` guards |
| `worktree integrate` accepts only the story-final leaf as the story's integration unit; the orchestrator integrates the story | `pkg/board/version_control.go:151-156` | `integrating` owned by the orchestrator, no tool invocation on entry |
| done is refused while a Change Request exists; inline work passes | `consistency.go:128-141, 158` | guard `delivery.cr-absent` |
| commit acknowledgement for the enforced types when policy confirms | `version_control.go:89-123, 277-279` | guard `delivery.commit-acknowledged`, `policy.delivery.version_control` |
| rejection routes to to-dev | `tools/board-cli/cmd/change_request.go:753, 825, 856` | `reviewing → to-dev via reject_cr` beside the manual route |
| linked blocked items, leaves and containers, release to backlog when blockers finish | `pkg/board/blocked_release.go:8-24, 76-87` | `on: { event: blockers.terminal }` on both workflows |
| orchestrator (and coordinator) has no lifecycle statuses | `pkg/board/goal_role.go:10-11, 25-33` | `root` |
| review policy defaults to required | `types.go:106-133` | `policy.review` |
| task class | `types.go:136-141` | field `task_class` |
| rework loop detection: threshold `max(3, 2 × median)` | `internal/changerequest/loopdetect/loopdetect.go:114, 126-129` | `policy.delivery.loop_detection` (`floor` 3, `factor` 2) |

Accepted differences and open points of the mapping:

1. The aggregation order of container statuses stays kernel code; the configuration only declares `derived_status`.
2. `one_of … by assignment.role` needs a structured role field; today the role is recorded only inside the assignee display string (`tools/board-cli/cmd/spawn.go:3856`). PC1 adds the field; until then the root orchestrator owns unassigned elements.
3. `closed` carries no resolution today.
4. Dependency links (`blocked_by`) are kernel data; the configuration refers to them only through `blockers.clear` and `blockers.terminal`.
5. Loop detection counts rework across Change Request revisions; it stays delivery policy, distinct from a transition `limit`.
6. `defer_cr` and `withdraw_cr` are not mapped yet.
7. The review policies `light` and `none` change reviewer routing, not the workflow; how they appear in a playbook is open.
8. The version-control policy lives in `task-board.config.json` today with authority class `project_policy` (`pkg/remoteconfig/authority_schema.go:117`); PC1 decides whether `Playbook.json` or the config owns it, and the other defers. The enforced types are fixed by the delivery module (`version_control.go:18-19`).
9. The linkless `blocked` state is owned by the orchestrator in the default, because today's kernel does not restrict who unblocks; stricter playbooks give it a human owner.

## 15. Integration plan

| Step | Delivers |
| --- | --- |
| PC0 | this draft, the schema, the examples; the refuter's first run (v1) and this revision (v2) |
| PC1 | the module: schema, parser, resolver (`extends`, `uses`, slots), validator CLI with portable and machine checks; a thin seam in task-board that loads `default.playbook.json` with identical behaviour or documented differences; new types from `Playbook.json` run on the generic engine; the structured `assignment.role` |
| Demos (right after PC1) | `dev-cycle`, `bug-hunt` → `dev-cycle`, `scrum` over `dev-cycle` first; the cafe once `task-board tick` and signed events exist |
| PC2 | role skills and the `curator-playbook-dev-cycle` package on Skillfile schema 2; task-board resolves roles from the project overlay `.roles` first, then installed role skills, then `~/.roles` read-only for one release with a warning; `setup.sh` stops copying `~/.roles` (this is roadmap milestone M1) |
| PC3 | the delivery module seams (Change Requests, worktrees, integration, validation as guards, actions and invariants); `task-board tick`; signed events and approvals; hand-off through waggle offers |

PC4 of the earlier plan, a Curator package kind for playbooks, is dropped: a playbook is a collection of skills. Details: [`track.md`](track.md).

## 16. Names

Specifications use technical terms. User-facing material uses the Apiary names *(decided)*: Hive = one node's agent system, Comb = a process, Keeper = the main loop, Bee = the agent loop, Waggle = the messaging protocol; a playbook keeps its name. The ecosystem wiki records the mapping; this is the only place in the specification that mentions it, and no identifier uses the user-facing names (the v1 owner kind `comb` is now `process`).

## 17. Decisions and open questions

Decided on 2026-09-24: the file and its format; the four core sections; one owner per element in each state with helpers, and `one_of` for states that admit several roles; owner kinds and who may move work out of each; authority outside the model with signatures for risky human acts; playbooks as data only, tools as skill commands, side effects enabled by the project, admission by `extends`/`uses`; needs without effort, effort in the operator file with a project floor; relaxing a requirement permanently only in the project file and temporarily through `ask` for the approver's own runs; guards, actions and events from modules; the delivery module and its invariants; `task-board tick` until the main loop; `extends` and `uses` with slots and membership links; roles as skills and playbooks as skill collections installed by one Skillfile schema 2 selector, with Curator releasing schema 2 and adding `directory` to `dependencies.skills`; package names `curator-playbook-<purpose>`; the static validator with portable and machine checks; the generality vectors; the refuter; demos right after PC1; PC4 dropped; the default playbook as the PC1 target.

Open:

1. The operator file's name (`~/.curator/models.toml` proposed) and its exact TOML shape.
2. Exception records for `ask` (§9): storage, expiry during a running run, interaction with `[allow]`.
3. How role skills are kept from triggering in sessions that do not run that role, once they are installed into the project.
4. The verdict vocabulary each role may record, and how it extends task-board's goal ledger (today reviewers record `accepted`, `changes_requested`, `stop_the_line`; `pkg/board/goal.go:40-49`); the goal kind an `orchestrator_goal` maps to.
5. `data/role.json` or skill frontmatter for role metadata.
6. Whether `Playbook.json` or `task-board.config.json` owns the version-control policy (§14 point 8).
7. How review policies `light` and `none` are expressed.
8. The open points of the mapping in §14.

## 18. Change log: v1 → v2

Each finding of the refuter's first run ([`../reviews/2026-09-24-refuter-v0.md`](../reviews/2026-09-24-refuter-v0.md)) and where v2 answers it.

| Finding | Resolution in v2 |
| --- | --- |
| F1 authority | actor classes; agent runs never act for humans, approve or post tool events; `people` maps groups to principals; `policy.signatures` for risky acts; honest note on enforcement strength (§4.2, §6.4) |
| F2 package bytes | playbooks carry no executables; tools are commands exported by installed skills; schedules, spawns and relaxations need `policy.enable`; admission by `extends`/`uses`; Curator's boundary cited (§8.5) |
| F3 default fails its own validator | capability checks apply to the producer of the evidence; complete provider table with events and `via` commands; declared external events; the prototype passes the default (§4.3, §6.3, §10) |
| F4 containers reopen | `derived_status` defined; aggregated moves are not transitions; §14 corrected (§5.1, §14) |
| F5 further code differences | every row added to §14 as reproduced or an accepted difference; handoff gate scoped by `applies_to`; rollback and restore exits; integration unit; open-ended roles; version-control keys; loop-detection formula; estimate grandfathering; container release |
| F6 `via` and `*` precedence | transitions identified by `(from, to, via)`; explicit shadows `*` for the same triple (§6.2) |
| F7 `one_of` | decision reworded to one owner per element in each state; structured `assignment.role`; root owns unassigned elements; capability selectors (§6.1) |
| F8 role lookup order | project overlay first, then installed skills, then `~/.roles` (§15) |
| F9 effort, `ultra` | effort removed from needs and from the schema's needs; operator file and project floor (§2.3, §9); `ultra` gone |
| F10 who relaxes | permanent only in the project file; temporary `ask` for own runs; amends agent-selection §8; open details (§9, §17.2) |
| F11 approval storage and reach | approvals are board records bound to a principal; scope as element, nearest ancestor of a type or project; own runs only (§6.4, §9) |
| F12 `degrade` | statically declared `degraded_state`; refused for independence; needs project enablement (§9) |
| F13 initial states | creation is entry into `initial`, subject to `requires` (§5.1) |
| F14 hand-off | process identity; process slots; waggle offer and accept; placement by the receiver (§6.5) |
| F15 timers before the main loop | `task-board tick` with a board lease; signed `task-board event post` (§6.9) |
| F16 sequencing | Skillfile schema 2 exists behind a flag; demos right after PC1; PC4 dropped (§15) |
| F17 composition | project sets clashing policy keys; per-orchestrator goals; orchestrators map onto scope leases (§6.7, §7.2) |
| F18 scrum multi-parent | membership links; containment stays a tree (§7.2, scrum example) |
| F19 delivery invariant | unconditional invariants; dev-cycle keeps accepted leaves in `accepted` until the story integrates (§6.10) |
| F20 template resolution | admitted templates only; unique names with project-before-global precedence; recorded package identity and digest; `migrations` (§7.1, §8.2) |
| F21 portability | portable versus machine checks; tool and people bindings in `Playbook.json` (§10) |
| F22 `create` and `required` | `required` means before leaving the initial state; `create` carries fields and a placement rule (§5.1, §5.2) |
| F23 extends across packages | `extends` object pins the parent package identity; Curator's `directory` for dependencies (§7.1, §8.3) |
| F24 needs on the root | apply only to spawned sub-orchestrators (§4.4) |
| F25 `comb`, verdict spelling | kind `process`; one verdict spelling `human-decision-needed` (§4.2, §12) |
| F26 schema and text | `initial` defined; guard and action shapes separate; `$schema` required; argument lists per hook in §6.3; `x-` keys out of the digest. Still open: per-hook argument schemas in each provider module |
| F27 extends example | example rewritten against the current dev-cycle; removing a missing key is an error; determinism rule (§6.2, §7.1) |
| F28 automatic resolutions | `resolution` on transitions and `limit` redirects; validator check (§6.1) |
| F29 epic `human-decision-needed` | epic gains `decision-needed` (dev-cycle example) |
| F30 guard bypass, capability freedom | cafe `escalated → sent` guarded; capabilities restricted per kind by the schema; quorum and bypass checks (§4.3, §10) |
| F31 roles as skills | `data/role.json` lists needed skills; the project entry wins; ambient triggering is open (§8.1, §17.3) |
| F32 privacy | stated as views only; out of the conformance vectors until a separate store (§5.4, §11) |
| F33 module namespacing | every delivery hook carries the `delivery.` prefix; generic verdict and evidence guards stay in the kernel; git policies under `policy.delivery` (§6.3, §6.8) |
| F34 where the spec lives | this repository is the home (wiki decision); the roadmap rows and a supersession note in `playbooks.md` are updated outside this repository |
| F35 goal ledger | open question §17.4 |
| F36 skill-name clashes | published packages prefix their skill names; the default keeps unprefixed built-in names (§7.2) |

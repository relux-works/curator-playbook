# Process configuration: specification

Status: **DRAFT v3**, 2026-09-24. Not normative until frozen after the refuter pass on the sections v3 changed. Roadmap: `wiki/roadmap/ecosystem-roadmap.md` (process-configuration line). Track: [`track.md`](track.md). Schema: [`../schema/playbook-v1.schema.json`](../schema/playbook-v1.schema.json). Examples: [`../examples/`](../examples/).

A team's process is code today. task-board's element types, statuses, gates and roles are compiled into its kernel, so a team that works differently — or a cafe ordering supplies, or a group formalizing a proof — cannot describe its process without changing the board. This specification answers one question: how does a project state who does what, which kinds of work exist and how each moves, in one file that every teammate can share and a validator can check before anything runs? It covers that file, its schema, extension and composition, distribution through Curator, model needs, and validation. The agent loop, an always-on scheduler and the goal ledger stay outside it (§1).

v3 applies the operator's decisions of 2026-09-24 on every point v2.1 left to a person. The operator confirmed the author's resolutions of both refuter runs, decided the proposed defaults H1, H2, H3, H5 and H7 (H2 more strictly than proposed), and added standing answers, speed needs, field conditions, work-in-progress limits and check sets. §17 lists the decisions and what stays open, and §20 maps each decision to the sections it changed. §19 and §18 keep the mapping of the second refuter run ([`../reviews/2026-09-24-refuter-v1.md`](../reviews/2026-09-24-refuter-v1.md)) and the first ([`../reviews/2026-09-24-refuter-v0.md`](../reviews/2026-09-24-refuter-v0.md)).

Every statement in this revision is *decided (2026-09-24)* unless §17.3 lists it as open or it is marked *future*.

Supersedes the roles-and-routes model of `skill-project-management/.specs/drafts/playbooks.md`: the core object is no longer a list of roles with routes between them but entity types with workflows whose states are owned by roles. Related: `skill-project-management/.specs/drafts/agent-selection.md` (selection layers; §9 amends its §3, §7 and §8), `skill-project-management/.specs/drafts/launch-profiles.md` (LP; §9 amends D9 for the case without an operator file, §2.3 moves D2's runtime catalog), curator-spec `protocol/skillfile-sources.md` (Skillfile schema 2, package identities), curator-spec issue #91 (on-demand skills and the skills launch channel, §8.1), `protocol/core.md` §3, §4.1 and §4.4 (execution boundary, command shims, skill dependencies), `protocol/environments.md` (package-influence boundary), `profiles/manager.md` (scope precedence), `waggle/spec/waggle.md` (approver signatures, hand-off offers, scope leases, the roster at a pinned revision).

---

## 0. The design on one screen

- A project describes how it works in one file, `Playbook.json`, in its root beside `Skillfile.json`. *decided*
- The file has four core parts: **roles** (aliases bound to role skills, people, tools or other processes), **entities** (element types, nesting, fields), **workflows** (states and transitions) and **policy**. *decided*
- **Each element has exactly one owner in each state; a state may admit several roles and assignment picks one**; helpers work for the owner. *decided (H8)*
- **Playbooks are data only.** They never name a program, script, webhook or URL. Everything that executes is a skill installed through Curator: a tool is a command that a provider skill exports, and the orchestrator skill reaches it through a runtime dependency narrowed to that command. Schedules, automatic spawns and tool invocations a package proposes take effect only when the project enables them; a tool runs only when the project lists it. *decided*
- **Authority stays outside the model.** An agent run never moves work out of a human-owned state, never records a human approval and never posts a tool or external event. Risky human acts need a waggle approver signature that shows a person acted. *decided*
- **The process in force is the committed one.** The kernel reads the project's `Playbook.json` from the head of the default branch of the repository that owns the board, never from the working tree, so no session can edit its own authority. *decided*
- A project **extends** one published playbook and writes only its differences, and **composes** several, binding their slots; its `extends`/`uses` is its list of admitted playbooks. *decided*
- **Roles are skills.** A published playbook is a repository whose `skills/` collection holds the root orchestrator, whose data carries the template, and one skill per role. One Skillfile schema 2 selector installs the whole process; tool providers sit outside the collection. Role skills are installed on demand and reach only the launches that run their role. *decided*
- Roles state **needs** — requirements and preferences in one vocabulary, speed included — never models or efforts; the operator's private files bind models and effort; only the project relaxes a hard need permanently, and an operator may answer a temporary relaxation in advance. *decided*
- **Guards read field values**, so review levels and thresholds are routes; **policy bounds work in progress**; and **check sets** — local, on the project's CI, or both — supply delivery evidence. *decided*
- A static **validator** (`playbook validate`) checks a configuration before anything runs, split into portable checks and machine checks. *decided*
- Development machinery is a **delivery module** whose invariants bind to element state and hold whatever a playbook says. *decided*
- Until the main loop exists, `task-board tick`, called every minute by a user-level system timer, fires timers, schedules, expiries and tool invocations. *decided*
- Today's task-board is itself a playbook ([`default.playbook.json`](../examples/default.playbook.json)); PC1 makes the kernel read it with identical behaviour or documented differences, and the first demos run right after PC1. *decided (H6)*

## 1. Goals and non-goals

Goals:

1. Any process — software delivery, a restaurant's purchasing, formalizing a proof, a family calendar — is configuration, not kernel code.
2. The same file works for every teammate, whatever harnesses and models each one has.
3. A process can be read, validated and eventually drawn and edited by a visual constructor without losing anything.
4. Processes compose: the output of one can be the input of another, and a cadence (Scrum) can sit over a delivery process.
5. A configuration can never make an installed package execute something the project did not admit.

Non-goals of this revision: the orchestrator's agent loop; an always-on scheduler (the main loop, later; until then §6.9); the goal system itself (`board-goal-v2` stays in task-board; the configuration only names each orchestrator's goal type, §6.7); the storage format of board elements; the waggle amendment this design depends on (§17), which is written in the waggle specification.

## 2. Files and responsibilities

### 2.1 The project's files *(decided)*

| File | Answers | Read by | Carries |
| --- | --- | --- | --- |
| `Skillfile.json` | what is installed | Curator | skill names, sources, versions (tags, revisions), activation |
| `Playbook.json` | how the project works | the board (task-board today) and the validator | admitted playbooks, role aliases bound to skill names, tool bindings, people, event signer groups, entities, workflows, policy: review, work in progress, version control, signatures, data |
| `task-board.config.json` | where and with what the board runs | the board | board location, integration base branch, board repository, check sets (§6.12), team runtime names |

All three are committed. Process rules that sit in `task-board.config.json` today move to `Playbook.json`: the version-control policy (commit confirmation and time) and the per-board run limit `spawn.max_parallel`, which becomes `policy.wip.board` (§6.8); the old keys are read for one release with a warning *(decided 2026-09-24)*. Model choices leave both committed files for the operator's private files, and so do machine limits (§2.3).

`Playbook.json` never carries a version, a URL or a source; `Skillfile.json` never carries a role, a state or a transition. They meet only through skill names. When a role or tool names a skill that is not installed, the validator reports it with the fix: the `curator add` invocation for a named skill, or the Skillfile selector to add (§8). Installing (Skillfile) and using (Playbook) are two edits with two meanings, like a dependency in `go.mod` and its `import`; a helper such as `playbook bind <alias> <skill>` can make both.

Without Curator, role skills can be copied into the harness skill directories by hand and `Playbook.json` does not change; tools do not run (§8.4).

### 2.2 Format *(decided)*

- JSON with a published JSON Schema: every file starts with `"$schema"` and `"schema_version": 1`. Editors get validation and completion; a constructor reads and writes the file without loss.
- Canonical JSON (the canonical form Curator uses for digests) defines the semantic digest of a resolved playbook. The `ui` blocks, `description` fields and `x-` keys are excluded from the digest.
- Comments are `description` fields on any element; a constructor shows them.
- Every element may carry a `ui` block (positions, colours, collapsed groups) for a visual constructor. The kernel ignores it.
- Keys starting with `x-` are extensions; the kernel ignores them.
- Names of roles, entity types, workflows and states are kebab-case identifiers; field names are snake_case (`task_class`), matching today's element fields.

### 2.3 The operator's private files *(decided 2026-09-24)*

What belongs to one operator and one machine lives in the operator's private files under `~/.curator/`, never in a repository:

| File | Holds | Read by |
| --- | --- | --- |
| `models.toml` | role bindings to models and efforts, admitted models and billing classes, time windows, standing answers to `ask` | the selection resolver (§9) |
| `runtimes.toml` | this machine's runtime bindings (harness binaries, accounts, homes) and its capacity: parallel runs, concurrent check sets, rate-limit ladders | the launch plane; the kernel, for machine capacity |
| `engines.toml` | local inference engines and their models | curator-inference-manager |
| `network.toml` | network profiles | curator-network-profiles |

They are TOML because people edit them by hand, and they are never committed. Keeping them together amends LP D2, which placed runtime bindings in `$XDG_CONFIG_HOME/agents-management/runtimes.toml`, and the network-profiles specification, which placed its profiles in `network.json` under `$XDG_CONFIG_HOME`. Which commands add and remove entries is open (§17.3).

```toml
[bindings]
"dev-cycle/developer" = { run = "codex/gpt-6-luna", effort = "max" }
"*/refuter"           = { run = "claude/claude-opus-5-5", effort = "high" }
"@reviewer"           = { run = ["claude/claude-opus-5-5", "codex/*"], effort = "medium..high" }
"@producer"           = { run = "codex/gpt-6-luna", effort = "high..max" }

[allow]
models  = ["codex/*", "claude/claude-opus-5-5", "local/qwen3.6-27b"]
billing = ["subscription", "local"]

[windows]
"local/qwen3.6-27b" = "22:00-07:00"

[answers]
"dev-cycle/reviewer" = { relax = ["not_same_publisher_as"] }
"*/refuter"          = { relax = ["context_window"], projects = ["github.com/relux-works/skill-project-management"], until = "2026-12-31" }
```

- One `[bindings]` table; keys go from specific to general: `<playbook>/<role>`, then `*/<role>`, then `@<capability>`. The most specific matching key wins.
- `run` is one `<runtime>/<model>` or an ordered array of candidates; a pattern such as `codex/*` matches every registry row of that runtime. The resolver takes the first candidate that is admitted, available and satisfies the role's needs; a candidate that is unavailable moves the choice to the next one. The selection records which candidate won and why each earlier one did not.
- `effort` is one value or a range (`"medium..high"`). A project floor (`policy.effort_floor`, §6.8) narrows it from below per model row (§9); the selection records the effort it used.
- `[allow]` narrows which models may run at all, and `billing` lists the billing classes this operator pays for. Only this file admits `metered` billing: a project may narrow billing classes, never widen them (§9). `[windows]` limits when a model may run.
- `[answers]` holds standing answers to `ask` (§9). Keys follow `[bindings]`; `relax` names the requirement terms the answer relaxes; `projects` and `until` bound it. A standing answer applies to this operator's own runs only and never widens `[allow]`.
- A file that exists but cannot be read or parsed refuses every selection: a read failure is never headroom (LP D9).
- **Without `models.toml`** the resolver chooses automatically, never chooses a metered runtime, and explains its choice (§9).
- The model keys of today's `task-board.config.json` — `spawn.ceilings`, per-role defaults, `workload_classes` and `preferred_agentic_system` — move to `models.toml`. Until the file exists they are read as a legacy operator layer with a warning, and one command writes them into the file.
- Machine limits never sit in a committed file, because teammates run different machines: parallel runs, concurrent check sets and rate-limit ladders live in `runtimes.toml`. The process's own limits on work in progress are `policy.wip` (§6.8).
- Every choice, from the file or not, must satisfy the role's needs (§9), else `on_unmet` applies.
- Tool bindings, people and event signers are not in these files: they are project facts and live in `Playbook.json` (§4.2, §8.5).

### 2.4 Which revision is in force *(decided 2026-09-24, H1)*

The orchestrator's session works in the project checkout with ordinary file tools. If the kernel read `Playbook.json` from the working tree, one unreviewed edit could drop a signature requirement, a hard need or a human owner (refuter v1, N1). waggle meets the same problem for its roster by reading it "at a pinned revision" (waggle §3). So:

- The kernel reads the project's `Playbook.json` from the committed file at the head of the default branch of the repository that owns the board, whatever Story is in scope. It never reads the working tree. One board therefore has one process in force at a time, and no branch or fork runs rules of its own.
- It records that revision, the **pinned revision** the rest of this specification refers to, and the file's semantic digest on the board at every load, beside the resolved templates' identities and digests (§7.1).
- The authority-bearing sections are `people`, `services`, `policy.signatures`, `policy.enable`, `policy.data`, role `needs` and `on_unmet`, `tools`, and `extends`/`uses`. State owners and guards also decide who may move work, so the whole file follows the same rule rather than a list of keys.
- An edit takes effect when it lands on the default branch, which makes a change to the process a change reviewed like code. The price is that trying a process change needs a landing. `playbook validate`, previews and a visual constructor read the working tree for that; the kernel never does.
- When the project has a waggle roster, the kernel also verifies the signature of the commit it read against the roster at that revision.
- A project that is not a git repository has no default branch: the kernel reads the file as it is and warns that its authority policy is a guardrail only.

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
  "services": { },
  "events": { },
  "entities": { },
  "workflows": { },
  "schedules": [ ],
  "policy": { },
  "ui": { }
}
```

`name` is required for a published template and optional in a project file. `extends` and `uses` are covered in §7, `slots` and `bind` in §7.2, `tools` in §8.5, `people` and `services` in §4.2, `events` and `schedules` in §6.9. Keys that only a project may set — `people`, `services`, `policy.enable`, `policy.effort_floor`, `policy.signatures` and `policy.data` — are ignored with a warning when they appear in a template. A template may bind a tool only under the rule of §8.5.

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

| `kind` | Performs the work | Bound by | Fires the declared exits of a state it owns |
| --- | --- | --- | --- |
| `agent` (default) | an agent running the role skill | `skill` | the spawned run of that role, and the root orchestrator for routing; guards still apply |
| `human` | a person or a group | `who`: a group of the project's `people` map | a human actor whose principal belongs to the group; acts that policy marks need a signature |
| `tool` | a command exported by a provider skill installed through Curator (§8.5) | `tool`: a name the `tools` map binds | the kernel's tool executor, from its own invocation: `tool.passed`, `tool.failed` (§6.11) |
| `process` | another process | `process`: a process slot the project binds (§6.5) | the kernel, when the received element returns: `process.returned` |

Four rules complete the table:

1. **Automatic exits leave any state.** A transition with `on` — a timer, a kernel or module event, a declared external event — may leave a state of any owner kind. The kernel fires it (actor `kernel` or `event`); no agent does.
2. **People may override tools and processes.** A human actor (a principal of the project's `people`) may fire the declared manual exits of a tool- or process-owned state, typically a cancel. The act is recorded as an override, a running tool invocation is cancelled, and policy may require a signature for it (`overrides`).
3. **Reopening needs a person when the type starts with one.** A transition out of a terminal state (`reopen: true`) has no source owner. A human actor may fire it; the root orchestrator may fire it only when the type's `initial` state is not human-owned. So only a person supersedes an accepted `decision`, whose initial state `proposed` a person owns.
4. **A stale assignment falls back to the root.** When an element enters a state whose owner does not admit its recorded `assignment.role` (§6.1), the record is cleared and the root orchestrator owns the element until the next assignment.

**Actor classes.** Every transition records its actor: `run` (a spawned run with its role), `session` (a hosted orchestrator session), `human` (a CLI action outside any run and session, or a waggle signature), `kernel` (automatic transitions, tool invocation, timers) or `event` (a signed external event, §6.9). An agent run or session never fires a transition out of a human-owned state, never records a human approval and never posts a tool or external event.

`human` is defined by absence today, so an untracked `curator run` session, a bare harness binary and a run that clears `TASK_BOARD_RUN_ID` all count as human. *Decided (2026-09-24, H1):* without a signature policy, an operator's CLI act outside any run counts as human, and this specification calls that what it is: a guardrail against well-behaved agents, not a boundary against a hostile one. The session host narrows the gap by marking the sessions it hosts, which then act as `session`. The boundary is a signature that shows a person acted (**Signatures**, below), checked against the committed file (§2.4). The validator warns when a workflow has human-owned states and the project sets no `policy.signatures` (§10). Today the only actor signal the kernel has is the `TASK_BOARD_RUN_ID` convention, which the producer guard itself calls "not an operator authorization boundary" (task-board `tools/board-cli/cmd/producer_handoff_guard.go:12-15`).

**People.** `people` maps groups to waggle operator principals, and only to them:

```json
"people": { "cafe-managers": { "principals": ["op:alice@cafe", "op:bob@cafe"] } }
```

- Orchestrator and worker principals (`orch:`, `worker:`) are agents and cannot be people; the schema refuses them.
- A pattern (`op:*@acme`) is resolved against the waggle roster at the pinned revision. A quorum (§6.4) counts distinct principals present in that roster. The roster is a committed file, so presence in it is a portable check (§10).

**Services.** `services` maps the signer groups that event declarations name to the principals of the bridges that may sign those events (§6.9). The service principal kind (`svc:` in the examples) comes with the waggle amendment this design depends on (§17).

**Signatures.** `policy.signatures` names which human acts need a waggle approver signature, at what key assurance, and whether the signature must show user verification. Its categories are defined by construct:

| Category | Covers |
| --- | --- |
| `approvals` | recording an approval that an `approvals` guard counts (§6.4) |
| `relaxations` | approving an `ask` exception (§9) |
| `spending` | firing a transition marked `"risk": "spending"` |
| `halts` | a waggle `cmd.halt` or `cmd.cancel` to a run of this process, and firing a transition marked `"risk": "halt"` |
| `overrides` | a human leaving a tool- or process-owned state (rule 2) |
| `human-transitions` | every transition a human actor fires out of a human-owned state |

- `min_assurance` is `software`, `hardware-bound` or `attested` (waggle §4.5).
- `user_verified` is `required` or `if-supported`. Only a FIDO2 signature whose user-verified flag is set shows that a person was there; an agent running as the operator's OS user can obtain `software` signatures from `ssh-agent` freely, and `hardware-bound` ones whenever the key does not demand presence (waggle §4.4). A key in a Secure Enclave cannot be copied, but its signature does not show whether a person confirmed it. *Decided (2026-09-24, H7):* whenever a project sets a signature policy, `user_verified` defaults to `required` for approvals, relaxations and halts; a project whose operators hold keys that cannot show user verification lowers it to `if-supported`.
- Approvals are signed only through the operator-facing command that renders the payload before signing ("what you see is what you sign", waggle §4.2); an agent never holds a key.
- waggle has no class, namespace or principal kind for board approvals, signed transitions or signed events yet. The waggle amendment adds them (§17); it is written into the waggle specification now and implemented with milestone CM2. Until then signed human acts work only in the forms waggle already defines, and the cafe demo waits (§15).

### 4.3 Capabilities

`capability` is the archetype guards reason about. Agents: `orchestrator`, `producer`, `reviewer`, `analyst`, `tester`; humans: `approver`; tools: `checker`. Custom capabilities use an `x-` prefix. The schema refuses a capability on the wrong kind, so a tool cannot pose as a reviewer. Today's archetypes map directly: implementer → producer, analyst → analyst, reviewer → reviewer, tester → tester (task-board `tools/board-cli/internal/spawn/role.go:60-90`).

A guard that needs a capability checks the actor that produced its evidence, not the owner of the source state: `delivery.cr-accepted` and `verdict` need a verdict recorded by a reviewer or an approver. The validator requires that some state of the workflow is owned by a role with that capability, so every verdict has a producer (§10). Evidence must also be fresh (§6.3).

### 4.4 The root role *(decided)*

`root` names the orchestrator of the process, and the validator requires that role to have capability `orchestrator`. Every resolved process has one; in a published playbook it is the skill that carries the template (§8). In a composition the project names the root (§7.2); the other playbooks' orchestrators keep owning their own states and goals (§6.7). Needs on the root apply only to spawned sub-orchestrators: a primary orchestrator session starts through `curator run`, whose model and effort precedence is closed (curator-spec Decision 0021 §1, §4; Decision 0013 §6.2).

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
- **`derived_status: true`** means the kernel aggregates the type's status from its children in a fixed order (today `pkg/board/escalation.go:11-100`). Aggregated moves are not transitions: they ignore `terminal`, `locked` and declared guards, and they may reopen a container from `done` or `closed` when a child becomes active again (`escalation.go:151-162, 198-200`) and move it into and out of `integrating` (`escalation.go:65-68`). They do obey the kernel's blockers check: a container with active blockers is not aggregated forward (`escalation.go:163-172`; `validation.go:46-58`), and a blocked container is left alone. Declared transitions on a derived type are the direct moves people and orchestrators make. Reachability and exit checks do not apply to derived types.

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

Types: `string`, `text`, `integer`, `number`, `boolean`, `date`, `datetime`, `duration`, `money`, `enum`, `ref`, `list`. **`required: true` means the field must be set before the element leaves its initial state**; creation may leave it empty, so a draft can be filled in its first state. A `ref` names an entity type or a slot. Transitions route on field values through the `field` guard (§6.3).

### 5.3 Estimates *(decided)*

Estimates have named dimensions, so complexity and duration are separate:

```json
"estimates": [ { "name": "complexity", "scale": "fibonacci" }, { "name": "duration", "scale": "hours" } ]
```

Scales: `fibonacci`, `points`, `tshirt`, `hours`, `days`. Capacity rules (§6.8) sum one dimension. Estimate dimensions are unrelated to reasoning effort, which never appears in a playbook's needs (§9).

### 5.4 Privacy *(decided 2026-09-24, H8)*

`privacy` on a type (`inherit`, `private`, `team`, `public`) and `policy.privacy` (`visibility`, `redact_fields`) express per-process privacy, one of the generality vectors (§11): a conforming kernel shows each view only what its visibility allows and redacts the listed fields. Board elements are committed files, so redaction in views does not remove data from history. Until a separate store or encryption exists, the vector tests views only, and the specification says so wherever privacy appears.

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

- **Each element has exactly one owner in each state; a state may admit several roles and assignment picks one.** The owner is accountable and is what authority checks use. *decided (H8)*
- Every non-terminal state of a resolved workflow has an owner; the validator checks the resolved result, so a file that extends a parent may override a state partially.
- A state that admits several roles uses `{ "one_of": [...], "by": "assignment.role" }`, or `{ "one_of": { "capability": [...] }, "by": "assignment.role" }` for every installed role with those capabilities (today's open-ended roles). The element records the structured role alias on assignment or spawn; until it has one, and after a move into a state that does not admit it (§4.2 rule 4), the root orchestrator owns it. *decided*
- **Helpers** are roles the owner may engage; they never own the state. Planning poker: owner `facilitator`, helpers `estimator`, answered blind and revealed by the owner. *decided*
- `requires` lists entry conditions: guards applied to every transition into the state and to creation when the state is `initial`.
- `terminal` states have no owner. Leaving one needs a transition marked `reopen: true`, whose actor rule is §4.2 rule 3 (aggregation, §5.1, is not a transition).
- `resolution: true` requires a resolution on entry; a list restricts the values. Automatic entries carry theirs: `resolution` on the transition, or on a `limit` redirect.
- `locked: true` means only transitions that name the state in `from` leave it; `from: "*"` never matches it.

Two rules keep ownership clear *(decided)*:

1. Two different results are two states: "built it" and "checked it" are `development` and `review`, never one state with two owners.
2. Parallel work is child elements plus a join guard (`children.terminal`, `children.in`) on the parent's transition, not parallel states. A `children.in` join must list every terminal state of every child type it covers, or it can never pass once a child ends in a state it omits (§10).

### 6.2 Transitions *(decided)*

```json
"transitions": [
  { "from": "open", "to": "development", "guards": ["estimate", "assignee", "blockers.clear"], "actions": ["spawn.owner"] },
  { "from": "review", "to": "development", "guards": [ { "guard": "verdict", "is": "changes_requested" } ],
    "limit": { "max": 3, "then": "escalated" } },
  { "from": "review", "to": "accepted", "via": "accept_cr", "guards": ["delivery.cr-accepted"] },
  { "from": "approval", "to": "sending", "risk": "spending", "guards": [ { "guard": "approvals", "role": "manager", "min": 1 } ] },
  { "from": "awaiting-confirmation", "to": "escalated", "on": { "after": "PT15M" } },
  { "from": "invoiced", "to": "paid", "on": { "event": "process.returned" }, "guards": [ { "guard": "process.result", "in": ["paid"] } ] },
  { "from": "*", "to": "cancelled" }
]
```

- `from` is a state, a list of states, or `*` (every non-terminal, non-locked state).
- A transition is identified by `(from, to, via)`. `via` names the one command that performs that triple (`accept_cr`, `integrate`); it does not restrict other triples into the same state, so a manual route and a command route may coexist. An explicit transition shadows the `*` expansion for the same triple.
- `guards` must all pass; `actions` run after the move. A guard object may carry `applies_to` to scope it: `actor` (`run`, `human`, `any`), `except_capability`, `own_element` (only when a run acts on the element it was spawned for) and `role_ends_in` (only for runs whose role's end state is the named state). Today's hand-off evidence gate needs the last two (§14).
- `on` makes the transition automatic: on an `event`, or `after` a duration in the source state. The kernel fires it; §6.9 says when and how events are kept.
- `limit` bounds a loop: after `max` firings for one element the transition leads to `then` instead, recording `resolution` when `then` requires one.
- `risk` marks a transition that a signature policy may cover: `spending` (it commits money) or `halt` (it stops work under way); custom markers use `x-` (§4.2).
- Determinism: the validator refuses two automatic transitions from the same state on the same trigger with the same guards, and two transitions with the same triple.

### 6.3 Guards, actions and events come from modules *(decided)*

A guard or action is a name, or an object whose `guard`/`action` key names it and whose other keys are its arguments; the schema keeps the two shapes apart and refuses arguments that carry a URL, shell syntax or a program-selecting name (§10). Names of module hooks carry the module prefix. The validator refuses any guard, action, event or `via` command that no loaded module provides and no `events` declaration names.

| Provider | Guards | Actions | Events | `via` commands |
| --- | --- | --- | --- | --- |
| kernel | `description`, `estimate` (`grandfather_legacy`), `estimates.present` (`scope`, `dimensions`), `assignee` (`manual_only`), `blockers.clear`, `children.terminal` (`types`), `children.in` (`states`, `types`), `children.any-in` (`states`), `members.terminal-or-carried`, `checklist.complete`, `outcome.present`, `fields.present` (`fields`), `field` (`name`; one of `is`, `in`, `lt`, `lte`, `gt`, `gte`), `approvals` (`role`, `min`), `verdict` (`is`, `detail`), `capacity` (`dimension`, `scope`), `process.result` (`in`) | `spawn.owner`, `assign`, `create` (`type`, `per`, `fields`, `parent`), `handoff` (`to`, `create`, `link_field`, `from_outcome`), `escalate`, `notify` (to an owner or a group), `record.verdict` (`is`, `detail`), `compute.velocity` (`dimension`) | `blockers.terminal`, `children.changed`, `process.returned`, `tool.passed`, `tool.failed` | — |
| delivery module (development only, §6.10) | `delivery.cr-published`, `delivery.review-ready`, `delivery.cr-accepted`, `delivery.cr-absent`, `delivery.commit-acknowledged`, `delivery.checks-passed` (`set`), `delivery.integration-evidence`, `delivery.external-integration-evidence`, `delivery.restore-authorized`, `delivery.rollback-authorized` | `delivery.integrate` (`checks`), `delivery.close-checkpointed-siblings` | `delivery.integrated`, `delivery.integration-failed`, `delivery.rework-started` | `accept_cr`, `reject_cr`, `integrate`, `integrate_external`, `restore_done`, `landed_rework`, `invalidate`, `converge` |
| timers | — | — | `after` durations | — |
| a playbook's `events` | — | — | declared external events (§6.9) | — |

`delivery.review-ready` passes for an element without a Change Request, or one whose latest revision is published. `delivery.checks-passed` passes when the named check set has evidence for the revision the transition moves (§6.12). `delivery.integration-failed` is emitted only together with a rollback authorization (§6.10). `delivery.integrate` integrates the accepted revisions of the unit, gated by the check set its `checks` argument names; a unit with none integrates at once.

**Field conditions** *(decided 2026-09-24)*. The `field` guard compares one field of the element with a value: `is` (equal), `in` (one of a list), `lt`, `lte`, `gt`, `gte` (numbers, money in the field's currency, dates and date-times). It turns declared data into routes. A review level is a field whose value selects the route: in dev-cycle `{ "guard": "field", "name": "review", "is": "none" }` lets inline work skip review. A threshold selects whether a purchase needs an approval: `{ "guard": "field", "name": "amount", "gt": 500 }` beside the `approvals` guard. The guard-bypass check reads field conditions on one field (§10). The guard names a field the element's type declares; an unset field fails every comparison.

**Verdicts** *(closes the first run's F35)*. Verdicts use the goal ledger's vocabulary and spelling (task-board `pkg/board/goal.go:40-49`): `accepted`, `changes_requested`, `stop_the_line`, plus `rejected` — the work is refused and will not be reworked. The ledger gains `rejected` in PC1, a task-board change, so any type may route on it *(decided 2026-09-24)*. A verdict may carry a typed `detail`, and a `verdict` guard may match it: the refuter records `accepted` with `no-contradiction`, `changes_requested` with `contradiction`, and `stop_the_line` with `human-decision-needed` (§12). Reviewers and approvers record verdicts; `record.verdict` records one for the acting reviewer or approver run and nothing for any other actor. `tool.passed` and `tool.failed` arise only from a state owned by a tool role (§6.11).

**Evidence must be fresh.** A verdict, an approval or an outcome satisfies a guard only when it was recorded after the element last entered the transition's source state; for work under a Change Request it must also name the revision the guard evaluates. This extends today's rule for reviewer verdicts, which count only when recorded after the latest producer outcome (`tools/board-cli/internal/spawnruntime/goal_acceptance.go:172-173`). So a refuter's old verdict cannot move a redrafted epic, a QA state waits for the tester's own outcome, and sending a purchase order again after an escalation needs a new approval.

### 6.4 Approvals and quorum *(decided)*

`{ "guard": "approvals", "role": "manager", "min": 1 }` needs approvals from that role; `min` above one is quorum voting and needs distinct principals present in the roster at the pinned revision. An approval is a board record bound to the approving principal, and it counts only while fresh (§6.3). It is a human act (§4.2): the operator's own CLI action counts only where policy does not require a signature; otherwise it is a waggle approver signature at the configured assurance and user verification, made through the command that renders the payload. The validator flags `min` greater than the number of principals in the role's group.

### 6.5 Hand-off between processes *(decided)*

A state owned by a `process` role is worked in another process: a playbook instance on the same board (a member of the project's composition) or on another board. The playbook names a process **slot**; the project binds it (§7.2). The `handoff` action offers the work; the receiving orchestrator accepts the offer, creates the element in its own hierarchy (placed by the rule of §5.1) and sets the link:

```json
{ "from": "verifying", "to": "fixing", "guards": [ { "guard": "verdict", "is": "accepted" } ],
  "actions": [ { "action": "handoff", "to": "fixers", "create": "fix-target", "link_field": "fix" } ] },
{ "from": "fixing", "to": "fixed", "on": { "event": "process.returned" }, "guards": [ { "guard": "process.result", "in": ["done"] } ] }
```

- **On one board** — every process of the project's composition — the offer and the acceptance are board records, and the kernel emits `process.returned` to the offering element when the received element reaches a terminal state. This path needs no waggle and ships in PC1, so the first demos can run (§15).
- **Across boards** the offer and the acceptance are waggle messages (`coord.handoff-offer`, `coord.handoff-accept`), and the result returns through the `coord` type the waggle amendment adds (§17). Until the amendment is implemented, with CM2, cross-board hand-off is not available.
- `process.returned` carries the final state of the received element; `process.result` guards on it. The exits of a process-owned state must cover every terminal state of the type the hand-off creates, so no result is left without a route (§10).
- Writes into another orchestrator's leased scope go through that orchestrator (§7.2).

### 6.6 Schedules *(decided)*

Recurring work is created by named schedules:

```json
"schedules": [ { "name": "weekly-inventory", "every": "P1W", "create": { "type": "inventory-check", "title": "Weekly inventory" } } ]
```

`every` is an ISO 8601 duration; `cron` is the alternative. A schedule a package proposes takes effect only when the project enables it (`policy.enable.schedules`, §8.5). `task-board tick` fires schedules, and each firing is keyed by (schedule, due time) in the lease store, so two ticks never create two elements for one firing (§6.9).

### 6.7 Short goals *(decided)*

Orchestrators lose the thread on large goals; the weaker the model, the shorter the step must be. `policy.goals.orchestrator_goal` names the type whose current instance is an orchestrator's goal — a story, a milestone or a sprint. In a composition it is a map from orchestrator role to type, so each orchestrator has its own. The long-term goal lives in the hierarchy and each orchestrator works one bounded step at a time. The goal ledger itself stays in task-board, and verdicts use its vocabulary (§6.3).

### 6.8 Policy

| Key | Meaning | Set by |
| --- | --- | --- |
| `review.default` | `required`, `light` or `none`: the default of the element field `review` for new elements (today's review policy); routes select on the field with `field` (§6.3) | template or project |
| `wip` | limits on work in progress: `board`, `per_role`, `per_state` (below) | template or project; the project wins |
| `capacity` | per container type, the maximum sum of one estimate dimension (a sprint of 100 complexity points) | template or project |
| `privacy` | view visibility and redacted fields (§5.4) | template or project |
| `goals` | §6.7 | template or project; the project wins in a composition |
| `effort_floor` | per role, the lowest effort a run may use, applied per model row (§9) | project only |
| `enable` | `schedules` (`all` or named), `auto_spawn`, `tools` (`all` or named), `relaxations` (§8.5, §9) | project only |
| `signatures` | which human acts need a waggle approver signature, the minimum key assurance and `user_verified` (§4.2) | project only |
| `data` | publishers denied, billing classes admitted, local only (§9) | project only |
| `delivery.version_control` | `confirm`, `desired_commit_time` (required when `confirm`), `commit_time_policy`; owned here since v3, and `task-board.config.json`'s `version_control` keys are read for one release with a warning (§2.1) | template or project |
| `delivery.loop_detection` | rework threshold `max(floor, factor × median)` | template or project |

**Work in progress** *(decided 2026-09-24)*. `policy.wip` bounds active runs, the runs working the board's elements:

```json
"wip": { "board": 6, "per_role": { "developer": 3, "reviewer": 2 }, "per_state": { "dev.review": 2 } }
```

- `board` bounds the active runs of the whole board; `per_role` those of one role; `per_state` those working elements in one state, named `<workflow>.<state>`, or a bare state name for that state in every workflow that has one.
- When starting a run would exceed a limit, the kernel does not start it: the element keeps its state, the start waits, and the kernel reports `wip_limit_reached` with the limit that held and the element's place in line. The start proceeds when a run ends.
- The limits describe the process. What one computer can carry — parallel runs across every board, concurrent local check sets, rate-limit ladders — is the operator's machine capacity (`runtimes.toml`, §2.3), and the stricter of the two holds.
- Today's per-board limit `spawn.max_parallel` becomes `policy.wip.board` (§14).

### 6.9 Timers, schedules, tools and external events before the main loop *(decided)*

Time-based transitions (`after`), schedules, approval expiries and tool invocations need a process that runs without an agent. The main loop will be that process; until it exists, `task-board tick` does the job:

- **Scheduler** *(decided 2026-09-24)*. A user-level system timer calls tick every minute: a launchd agent on macOS or a systemd user timer on Linux, installed by one task-board command. It runs whether or not any session is open. The Keeper takes the job over when it exists.
- **Scope.** Tick fires due `after` transitions, due schedules, expiries of `ask` approvals and pending tool invocations (§6.11), recording actor `kernel`.
- **Lease.** Tick takes a lease first. On one machine that is the board's own lease; a board shared across machines needs waggle's lease store (NATS JetStream KV, milestone CM2). Until CM2, a shared board runs tick on one machine only, which the project documents.
- **Idempotence.** Every firing has a key: (element, transition, due time) for timers and expiries, (schedule, due time) for schedules, (element, entry) for tool invocations. Keys live in the lease store, so running tick twice, or on two machines under one lease store, fires each item once.

External events enter through `task-board event post <event> --element <id>` with a signed payload. A template declares the events it reacts to and names a signer group; only the project binds the group to principals (`services`, §4.2):

```json
"events": { "supplier.confirmed": { "signed_by": "supplier-bridge" } }
```

- An unsigned event, a wrongly signed one and one whose signer group the project has not bound are refused and logged.
- A bridge that turns a provider's webhook into a signed event first verifies the provider's own webhook authentication (its signature scheme or mutual TLS), and signs only events of the groups it is bound to. Receiving webhooks needs a service the ecosystem does not run yet.
- **Events are kept.** An accepted event stays on its element until it fires a transition or the element reaches a terminal state; each kept event fires at most one transition, and one that matches no transition in the current state is logged and kept. Kernel events that describe current state (`children.changed`, `blockers.terminal`) are not kept: on every entry into a state the kernel evaluates that state's automatic transitions on those events as if they had just fired, and its other automatic transitions against the kept events. So a supplier confirmation that arrives after the SLA fired is not lost (cafe), and an attempt accepted while its milestone was stalled still proves the milestone when it reopens (formal missions).

### 6.10 Delivery invariants *(decided)*

The delivery module's invariants hold for every element on a board where the module is loaded, whatever its workflow declares. They bind to element state — an element under a Change Request, an element in integration — not to the hooks a workflow names, so a project that removes the `delivery.*` hooks from a workflow does not escape them while the Change Request commands still work on its elements. A playbook may add stricter checks, never weaker ones.

1. **Done only through integration, or inline.** The state the integration path enters (the target of `integrate`, `integrate_external`, `restore_done` or `delivery.integrated`) is reached by an element under a Change Request only through that path; every other route into it carries `delivery.cr-absent`, and today also the commit acknowledgement. An unreadable Change Request record refuses both (`pkg/board/consistency.go:105-141, 158`).
2. **No self-review.** The run that produced a revision cannot record its acceptance.
3. **Acceptance only of a published revision, and integration only through acceptance.** `accept_cr` accepts a published, immutable revision, and only it enters the state it targets (`consistency.go:166-167`). Reviewing work without a Change Request stays allowed, as today (`consistency.go:115-122`); a playbook that wants review only after publication adds `delivery.review-ready`, as dev-cycle does.
4. **Leaving integration for anything but done needs a rollback authorization:** landed rework, invalidation, convergence, or a failed integration, whose `delivery.integration-failed` event carries one (`consistency.go:169-172`).

The validator checks rules 1, 3 and 4 on every workflow that uses the delivery hooks; rule 2 depends on run provenance and is enforced at runtime only. `dev-cycle` keeps accepted Change Request leaves in a non-terminal `accepted` state until their story integrates, as `integrating` does today, and lets inline work finish on acceptance with `delivery.cr-absent`. Its story joins on every terminal state of its child types, so a story with design or inline work integrates too.

### 6.11 Tool invocation *(decided)*

A tool's verdict must not come from the actor that triggered it. In formal missions the prover's run moves an attempt into `checking`; if the check ran in that run's process, a `lean` earlier on its `PATH` or a replaced shim in the checkout would record `tool.passed` for an unchecked proof as actor `kernel` (refuter v1, N7). So:

- Entering a tool-owned state enqueues one invocation keyed by (element, entry). The actor of the entering transition never runs it.
- A kernel-owned executor runs it: `task-board tick` under its system timer until the Keeper exists (§6.9).
- It runs only the command the binding names, through the shim Curator installed for the provider after checking the shim against the Curator lock (its owner must be the bound skill), or under the enforced `script-worker-v1` policy when the provider declares it (curator-spec `protocol/core.md` §4.1.1).
- The environment is sanitized: a fixed `PATH` of the managed command directory and system directories, and no variables inherited from agent runs. The working directory is the project root.
- The contract: the command receives `--element <id>` and, on standard input, a JSON object with the element's id, type and declared fields, the playbook, the tool name and the entry key. Exit status 0 fires `tool.passed` and 1 fires `tool.failed`; any other status, a crash or the binding's `timeout` (ten minutes by default) fires `tool.failed` with the reason `error` or `timeout`. Bounded standard output is stored as the tool's outcome. The executor never retries; a `tool.failed` exit with a `limit` models retries.
- A person may leave a tool-owned state through its declared manual exits (§4.2 rule 2); a running invocation is then cancelled.
- A tool runs only when the project lists it in `policy.enable.tools` and only from a skill installed through Curator (§8.4, §8.5). Entering a state whose tool the project does not list is refused.

### 6.12 Check sets *(decided 2026-09-24)*

Evidence that a machine produces for delivery — tests, linters, builds — comes from named **check sets**. The playbook says which set must pass where; the project says how and where each set runs.

- A playbook names a set in the guard `{ "guard": "delivery.checks-passed", "set": "handoff" }`, or as the gate of an integration, `{ "action": "delivery.integrate", "checks": "landing" }`. dev-cycle uses `handoff` before review and `landing` for its story's integration.
- `task-board.config.json` defines each set under `checks`:

  ```json
  "checks": {
    "handoff": { "where": "local", "commands": ["make test-fast"],
                 "by_task_class": { "docs": { "where": "local", "commands": [] } } },
    "landing": { "where": ["local", "hosted"], "commands": ["make test-smoke"], "hosted": { "provider": "github" } }
  }
  ```

  - `where` is `local`, `hosted`, or both as a list; every place listed must pass.
  - `commands` are run in order by the kernel on this machine, under its machine capacity (§2.3); they are required when `where` includes `local`. An explicitly empty list is a recorded decision to run nothing, as today.
  - `hosted` names the hosting provider whose required checks must pass on the exact head that lands; it is required when `where` includes `hosted`.
  - `by_task_class` gives a set a variant per task class.
- Evidence binds to what the guarded transition moves: a local set to the tree it ran on, a hosted set to the head whose required checks the provider reported. The delivery module produces it, as today's suite does: it runs a local set when a producer hands off and again at integration when the bound evidence is missing or stale, and it collects a hosted set's result from the provider. The guard only checks that the evidence exists.
- When hosted checks cannot run for a verified external cause, the local-mirror rule applies: a clean checkout of the exact head runs the matching commands and records the evidence. The mirror never produces, edits or stands in for the provider's own status, so branch protection still waits for the provider.
- A process without the delivery module, such as the cafe, has no check sets and no dependency on a git host: its evidence comes from people (approvals), tools (§6.11) and signed events (§6.9).
- A set a playbook names must be defined in `task-board.config.json`; both files are committed, so the validator checks it portably (§10).
- Today's landing-gate suite, `spawn.worktree_isolation.validation.commands`, is a local `landing` set (§14).

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
      { "from": "qa", "to": "review", "guards": ["outcome.present", "delivery.review-ready"], "actions": ["spawn.owner"] }
    ] } }
}
```

Here an iOS project swaps the developer skill and puts a tester's QA pass before review. Because evidence must be fresh (§6.3), `qa → review` waits for the tester's own outcome.

Merge rules:

1. One parent. Chains are allowed; cycles are refused. The object form pins the parent by its Curator package identity: `{ "name": "dev-cycle", "package": { "kind": "network-git", "repository": "github.com/relux-works/curator-playbook-dev-cycle", "commit": { "object_format": "sha1", "hex": "…" }, "directory": "skills/orchestrator" } }`, or a `local-snapshot` digest, or a `configured-git` source (curator-spec `protocol/skillfile-sources.md`). The resolver compares it with the lock entry of the skill that carries the template; an extending template must use it.
2. Maps merge by key, recursively.
3. Transitions are keyed by `(from, to, via)`: a child transition with the same key replaces the parent's; `remove: true` deletes it. A state with `remove: true` deletes the state and every transition touching it. Removing a key the parent does not have is an error, so drift after a parent update is loud.
4. Lists other than transitions (`contains`, `parents`, `helpers`, guards, actions) are replaced as a whole.
5. The validator runs on the resolved result (§10) and reports which file introduced each error.
6. The board records the resolved template's package identity and semantic digest, and the project file's revision and digest (§2.4); teammates who resolve a different digest get a refusal naming both. A template update that removes or renames states declares `migrations` (old state → new state) or the validator lists the stranded elements and refuses.

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
- Names must be unique across the composition; a clash is resolved by qualifying with the playbook name (`dev-cycle.reviewer`). Installed skill names cannot be qualified (Skillfile schema 2 refuses repeated names), so published packages prefix their skills (`dev-`, `hunt-`, `scrum-`), and their commands too (§8.5).
- The project overrides members of used playbooks by name (qualified when ambiguous). Policy keys that two used playbooks both set must be set by the project; the validator refuses the clash otherwise.
- **Scrum is a cadence layer, not a base to extend.** It decides which work is selected and when; dev-cycle decides how an item is built. Containment stays a tree: a story stays under its epic, and a sprint holds its stories by **membership link** (`members`), at most one open sprint per story; carrying a story over relinks it.
- Orchestrators map onto waggle scope leases: each orchestrator of a composition leases the subtrees it owns; a transition into a state owned by another orchestrator's role is an offer, not a write (§6.5).
- **Which writes cross a lease.** Every board write — status, fields, estimates, links — to an element inside another orchestrator's leased subtree. It goes through that orchestrator, with two exceptions. A slot may declare `writes` (estimate dimensions, fields), and binding it grants the composing playbook's orchestrator a field-scoped lease on exactly those fields of the bound type: scrum's `work-item` slot delegates `complexity` and `duration`, which grooming writes on stories. Membership links are stored on the container that holds them (the sprint), inside the cadence orchestrator's own scope.

## 8. Distribution: roles are skills *(decided)*

### 8.1 A published playbook is a collection of skills

```
curator-playbook-cafe/
  README.md
  skills/                    the collection: the orchestrator and the roles
    orchestrator/            SKILL.md (name: cafe-orchestrator), agent-skill.json
      data/playbook.json     the process template (name: cafe-ops)
    stock-keeper/            SKILL.md (name: cafe-stock-keeper)
      data/role.json         capability, default needs, definition-of-done items, skills the role needs
    buyer/                   SKILL.md (name: cafe-buyer)
  tools/                     tool providers, never selected by the collection
    supplier-portal/         SKILL.md (name: cafe-supplier-portal), agent-skill.json exporting cafe-send-order
```

Every role directory is a real skill: instructions in `SKILL.md`, optional `references/`, `assets/`, `data/` — the roots Curator installs. The template lives in the orchestrator skill because the root role owns the process. Packages are named `curator-playbook-<purpose>`. Role metadata sits in `data/role.json`, not in the skill's frontmatter. The file also lists the other skills the role needs, as today's `.roles/<role>/skills.md` does; it has its own JSON Schema, which arrives with PC2, and the validator checks it *(decided 2026-09-24)*. When sources disagree, the project's `Playbook.json` role entry wins over the template's role entry, which wins over `data/role.json`. Tool providers live in `tools/`, outside the collection, for the reason in §8.5.

**Role skills are installed on demand** *(decided 2026-09-24)*. Harnesses pick skills by their description, so a role skill in a discovery directory can fire in any session of the project, and an orchestrator could start acting as a reviewer. Role skills, the root orchestrator's included, are therefore installed with Curator activation `on-demand`: resolved, audited and locked like any skill, and never materialized into the project's or the machine's discovery directories. A launch receives exactly the skill of the role it runs through Curator's skills launch channel, in identity form: the skill body goes through the harness's system-prompt channel together with the base directory of its locked snapshot. A role run gets its role's skill; the project's primary session gets the root orchestrator's skill; a plain harness session in the checkout sees none of them. This is a Curator change (curator-spec issue #91, curator issue #91; §8.3). Until it ships, role skills install as ordinary skills and their descriptions say "use only when spawned as role X", a stopgap that depends on the model obeying it.

### 8.2 One Skillfile entry installs the process

Curator's Skillfile schema 2 (curator-spec `protocol/skillfile-sources.md`) declares repository sources and selects skills from them: an individual `{name, from, directory}` selector or a collection `{from, directory, include, exclude?}` selector over immediate child directories. It exists in Curator behind a flag; PC2 uses it (§15).

```json
{
  "schema_version": 2,
  "sources": {
    "cafe":  { "git": "https://github.com/relux-works/curator-playbook-cafe", "tag": "v1.0.0" },
    "roles": { "git": "https://github.com/relux-works/curator-playbook-roles", "tag": "v0.3.0" }
  },
  "skills": [
    { "from": "cafe", "directory": "skills", "include": ["*"], "activation": "on-demand" },
    { "name": "security-reviewer", "from": "roles", "directory": "skills/security-reviewer", "activation": "on-demand" }
  ]
}
```

The first selector installs the orchestrator and every role of the cafe; the provider in `tools/` arrives as the orchestrator's runtime dependency (§8.5). The second selector adds a reusable role from another repository, which `Playbook.json` then binds to an alias. `activation` is the Curator change of §8.3 item 3; until it ships, the selectors omit it.

**Resolving a template**. The board resolves `"extends": "cafe-ops"` through Curator's locks, in Curator's precedence order: project, hybrid, global (curator-spec `profiles/manager.md`). Among the admitted playbooks (§8.5), the first scope that holds a skill whose `data/playbook.json` has that name wins; two such skills in one scope are an error. The kernel resolves from the scopes of the project checkout, so a run in its own Curator profile (agent-selection §6) does not change the template in force.

### 8.3 The Curator work *(decided)*

Three changes belong to Curator, not to this module:

1. Release Skillfile schema 2 from its flag.
2. `dependencies.skills` entries in a skill manifest (curator-spec `protocol/core.md` §4.4: `git`, exact `ref`, activation `mode`, command narrowing) gain the same `directory` selection as schema 2 individual selectors, with the same containment rules, so an orchestrator can depend on a tool provider in its own repository and a playbook on a role in a subfolder of another repository.
3. Skillfile schema 2 selectors gain activation `on-demand`, and `env resolve` and `curator run` gain a skills launch channel that delivers named on-demand skills to one launch and never into a shared profile home; `curator status --json` lists on-demand skills (curator-spec issue #91, curator issue #91).

### 8.4 Without Curator

The role directories can be copied into the harness skill directories (`.claude/skills`, `.codex/skills`, `agents/skills`). The resolver finds role skills and templates there by name; nothing in `Playbook.json` changes, and the machine checks (§10) warn that the skills are unaudited and that copied role skills are visible to every session (§8.1). **Tools do not run without Curator** *(decided, wiki decisions.md:84)*: a tool binding to a skill Curator did not install is refused with the `curator add` fix, and the states that tool owns cannot be entered.

### 8.5 Playbooks are data *(decided)*

A playbook never names a program, script, webhook or URL. This keeps Curator's boundary intact one layer up: "Profiles are data end to end. No file in a package snapshot is executed, sourced, or interpreted as configuration for the manager itself" (curator-spec `protocol/environments.md:94-98`); "A profile chooses what the context says, never how a process is launched. This is the package-influence boundary of the core execution policies" (`environments.md:3144-3146`); a manager "MUST NOT transfer execution control to a package- or repository-selected program, argument vector, environment, hook" (`protocol/core.md:85-91`).

- **Tools are skills, reached through the orchestrator's manifest** *(decided, wiki decisions.md:84)*. A playbook names a tool; `tools` binds the name to a command a provider skill exports. The orchestrator skill that carries the template declares each provider in its Curator manifest as a skill dependency in `runtime` mode, narrowed to the exported commands (curator-spec `protocol/core.md` §4.4). An excerpt of the cafe orchestrator's `agent-skill.json`:

  ```json
  "dependencies": {
    "skills": {
      "cafe-supplier-portal": {
        "git": "https://github.com/relux-works/curator-playbook-cafe",
        "ref": { "kind": "tag", "value": "v1.0.0" },
        "directory": "tools/supplier-portal",
        "mode": "runtime",
        "commands": ["cafe-send-order"]
      }
    }
  }
  ```

  and the binding in its `data/playbook.json`:

  ```json
  "tools": { "supplier.send-order": { "skill": "cafe-supplier-portal", "command": "cafe-send-order" } }
  ```

  - A runtime dependency activates only the named commands and puts none of the provider's context into sessions. A skill the collection selects directly is a `full` requirement: its context reaches every session and all its commands are active (`protocol/core.md:1418, 1426-1428`). That is why providers live outside `skills/`, or are excluded from the selector.
  - `directory` is the Curator change of §8.3 item 2, needed when the provider lives in the playbook's own repository; a provider in its own repository needs none.
  - A template's binding is admitted only when that manifest declares the provider and the command (a machine check, §10). v2's rule "a template may bind a tool only to a skill of its own package" is gone: in Curator every skill is its own package and installation unit (`protocol/core.md:80`; `skillfile-sources.md:69-71`).
  - Command names are global shims with exactly one owner each (`protocol/core.md:212-213`), so published packages prefix them: `cafe-send-order`, `proof-lean-check`. The machine check verifies that the shim's owner is the bound skill.
  - A project may rebind a tool to another installed provider.
  - A bound tool runs only when the project lists it in `policy.enable.tools`, by name or with `all`; `all` also admits tools that later versions of the admitted templates add. A tool the project does not list is disabled, and the states it owns cannot be entered. There is no separate form for disabling a tool *(decided 2026-09-24, H3)*.
  - The kernel runs only commands exported by skills installed through Curator, never a string from a playbook, and always through the executor of §6.11.
- **Side effects are proposals.** Schedules (§6.6), `spawn.owner` actions, tool invocations and `fallback`/`degrade` relaxations (§9) that a package declares take effect only when the project enables them (`policy.enable`).
- **Admission.** The project's `extends` and `uses`, read at the pinned revision (§2.4), are its list of admitted playbooks. A template outside that list is never resolved, whatever is installed.

## 9. Model needs and unmet needs *(decided)*

A committed file must not break for a teammate with other harnesses, so roles state needs, never model ids and never efforts. `require` is hard: the role cannot run without it. `prefer` is soft: ranking only.

**Vocabulary** *(decided 2026-09-24, H5)*. Agent-selection S1's terms (agent-selection §3, §12) plus speed, as `Playbook.json` spells them. Every term may appear in `require` or in `prefer`; numeric terms are ranges `{ "min": …, "max": … }` with at least one bound.

| Term | Meaning |
| --- | --- |
| `publisher`, `family` | the row's recorded provenance matches one of the values; an empty publisher never matches |
| `capabilities` | the row has every listed model capability (the module registry's vocabulary, for example `tool-use`; agent-selection S2 fills it) |
| `context_window` | the row's context window in tokens, a range |
| `reasoning` | `true`: the row accepts a reasoning effort |
| `local` | the row runs on a local engine |
| `not_same_publisher_as` | independence, below; as a preference it ranks rows of another publisher first |
| `pp` | prompt processing speed in tokens per second, a range |
| `tg` | token generation speed in tokens per second, a range |
| `ttft` | time to first token in seconds, a range |

In `require` a term must hold for the row to be a candidate; in `prefer` it only ranks candidates. `class` sits beside `require` and `prefer`: the requirement class, which defaults to the role's workload class. Agent-selection spells the two blocks `requires` and `prefers`, the hard reasoning term `"reasoning": "required"` and the context term `context_window_min`; this specification amends agent-selection §3 to the spelling above. The old need term `capability` becomes `capabilities`, so it no longer collides with a role's archetype `capability` (§4.3). Roadmap milestone M4 then reads its role needs from `Playbook.json` as written.

```json
"reviewer":      { "needs": { "require": { "not_same_publisher_as": "developer", "reasoning": true },
                              "prefer":  { "publisher": ["anthropic"], "tg": { "min": 50 } } } },
"night-refuter": { "needs": { "require": { "local": true, "tg": { "min": 10 } },
                              "prefer":  { "context_window": { "min": 200000 } } } }
```

**Speed facts** *(decided 2026-09-24)*. A speed term is evaluated against the best fact available for the row on this machine, and the selection records which source it used:

1. **Measured.** For a local model, curator-inference-manager's benchmark of that engine and model on this machine, prompt processing and generation, repeated when either changes. For a cloud model, the median of recent observed runs on this machine and account, which task-board records for every run.
2. **Declared.** Typical ranges shipped with the model registry. A requirement is checked against the conservative end of the range: the lower end of `pp` and `tg`, the upper end of `ttft`.
3. **Unknown.** A requirement is unmet and `on_unmet` applies, with a refusal that names what to measure; a preference scores nothing.

`pp` and `ttft` depend on the length of the prompt, so both are measured at one reference prompt size, which PC1 fixes (§17.3).

**Independence**. `not_same_publisher_as: <role>` is evaluated against the provenance of the run that produced the evidence under review — the run of that role that recorded the outcome or the Change Request revision the reviewer reads — never against a binding. On a shared board that run may come from another teammate's machine, bindings may be candidate lists, and without an operator file there is no binding at all. Run manifests record the runtime and the model (LP D11), and the registry maps the model to its publisher. Work produced by a person satisfies the requirement; work whose producing run has no recorded provenance does not, and `on_unmet` applies. In a composition the value may be a qualified role name.

**Effort** *(decided: H4 keeps the floor)*. Effort is not a need: it lives in the operator file (§2.3). A project may set a floor per role (`policy.effort_floor`), and the operator chooses within it. Model rows own separate effort vocabularies that overlap only by coincidence (task-board `pkg/remoteconfig/reasoningeffort/reasoningeffort.go:98-100`; a local Qwen row may admit only `low`, `medium`, `high`, LP D2), so the floor is applied per row on the common order `minimal < low < medium < high < xhigh < max`:

- the effective floor is the lowest value the row admits at or above the floor;
- a row that accepts no reasoning effort at all (Claude Haiku) satisfies the floor vacuously; a project that wants to exclude such rows requires `reasoning: true`;
- a row with no value at or above the floor is unmet, and the role's `on_unmet` applies with the refusal `selection_effort_floor_unsatisfiable`, which names the row's vocabulary.

**Without the operator file** *(decided 2026-09-24, H2)*. The resolver chooses automatically, never wider than:

1. the rows the project admits (`policy.data`, below), and never a metered runtime: without `models.toml` only `subscription` and `local` rows are candidates, whatever `policy.data.billing` lists, because only the operator's own file may spend the operator's money (§2.3). A runtime whose billing class cannot be determined — one harness may run on a subscription or on an API key, depending on how it is logged in — counts as metered;
2. the rows installed and logged in on this machine: the availability facts of roadmap M5b where they exist; before M5b, installed and bound runtimes, where a launch refused for authentication moves the choice to the next candidate;
3. the rows that satisfy the role's needs and the effort floor.

It prints the choice and the reason and never asks mid-run, because an orchestrator works without a person present. A command writes the choice into `models.toml` as `[bindings]` lines. How the first start guides an operator who has no file is open (§17.3). This amends agent-selection §7 — without an operator file, operator admission is replaced by project admission ∩ machine facts ∩ non-metered billing, never by the whole registry — and LP D9: with no operator config, the project's data policy narrows admission, and a file that exists but cannot be read still refuses, since a read failure is never headroom.

**Project data policy** *(answers the first run's F10)*. `policy.data` carries agent-selection's committed data policy (agent-selection §4): `publishers.deny`, `billing` (`subscription`, `local`, `metered`) and `local_only`. It only narrows: listing `metered` admits metered rows for operators whose own file admits them, and never admits metered billing by itself. `Playbook.json` supersedes agent-selection's committed project layer for role needs, admission (`extends`/`uses` replace `playbooks.allowed`), data policy and waivers; a waiver is a reviewed change to the project's role entry, with the reason in its `description`. Team runtime names and check sets live in `task-board.config.json` (§2.1, §6.12); machine limits live in the operator's `runtimes.toml` (§2.3).

**When needs cannot be met**, `on_unmet` decides:

| `on_unmet` | Behaviour |
| --- | --- |
| `block` (default for `require`) | refuse, naming the layer that emptied the candidate set |
| `fallback` | try the ordered `fallback` needs; active only when the project enables relaxations |
| `ask` | ask a person for a temporary exception (below) |
| `degrade` | run on the best available through a statically declared `degraded_state` (a review state with its own owner); active only when the project enables relaxations; refused for independence requirements, because more reviews from the same publisher cannot restore independence |

**The degraded path**. A run selected under `degrade` marks its element degraded. The next exit from the state that run worked in is redirected into `degraded_state`, recording the original target; `degraded_state` has an owner other than the degraded role and declared exits back into the workflow, and leaving it clears the mark. The validator checks that `degraded_state` exists in each workflow where the role owns a state and that it reaches the original targets.

**Who relaxes a hard requirement** *(decided)*:

- Permanently: only a reviewed change to the project's `Playbook.json`, which takes effect when it lands (§2.4). This amends agent-selection §8 ("only the project can waive") by adding the temporary paths below.
- Temporarily: through `ask`, an operator approves an exception for **their own runs only**, scoped to an element subtree (`element`, the nearest ancestor of a type, or `project`) and time-limited. The exception is a kernel record, `exception`, available in every playbook rather than a type a playbook must declare, and it is bound to the approving principal; it never reaches a teammate's runs. Until runs are bound to principals (waggle worker certificates), "own runs only" is a guardrail. `policy.signatures` may require a signature for it (`relaxations`).
- An exception that expires while a run is working lets that run finish; new runs refuse. An exception relaxes only project requirements, never the operator's own `[allow]` *(decided 2026-09-24)*.
- **Standing answers** *(decided 2026-09-24)*. An operator may answer an `ask` in advance in `models.toml` (`[answers]`, §2.3), so autonomous work does not stop for a question the operator has already answered. When `ask` fires and a standing answer matches the role, the requirement term and the project, the kernel records an `exception` marked as a standing answer from the operator file, bound to that operator, and the run proceeds; the team sees the exception on the board. A standing answer never applies when `policy.signatures` covers `relaxations`, because a file cannot show that a person acted at that moment. It applies to the operator's own runs only and never widens `[allow]`.

Example: the reviewer requires a publisher different from the developer's; a teammate has only one publisher; `ask` fires; the teammate approves "same-publisher review under this epic, 30 days" for their own runs, or has answered it in advance in `[answers]`. Every selection records which layer supplied what and whether a fallback, an approval, a standing answer or a degradation applied.

**Per-element needs** *(future, after PC1)*. An element may add preferences and extra skills for its own runs and narrow its role's requirements, never relax them, within the operator's files.

## 10. Static validation *(decided)*

`playbook validate` is a library and a CLI in this module, independent of task-board. task-board runs it when it loads a playbook; a visual constructor runs it on every edit. It resolves `extends` and `uses` first. Checks are split so that CI and every teammate get the same structural verdict:

**Portable checks** (depend only on committed files):

| Check | Example error |
| --- | --- |
| undefined states | `workflow dev: transition review → qa: state qa is not defined` |
| initial and creation | `workflow dev: initial open is not a state` |
| reachability | `workflow dev: state escalated unreachable from open` (derived types exempt) |
| terminal reachability | `workflow dev: no path from review to a terminal state` |
| owners | `workflow dev, state review: owner tester is not a role`; `workflow dev: non-terminal state review has no owner` (on the resolved workflow) |
| root | `root developer does not have capability orchestrator` |
| owner-kind exits | `workflow purchase: tool-owned state sent lacks a tool.passed or tool.failed exit`; `process-owned state invoiced has no process.returned exit`; every other non-terminal state needs at least one exit (derived types exempt) |
| capability producers | `workflow cafe.purchase, transition approval → draft: guard verdict has no reviewer or approver producing it` |
| capability per kind | enforced by the schema: a tool cannot be a reviewer |
| providers | `guard delivery.cr-publish: no loaded module provides it`; `event supplier.confirmd: not provided or declared` |
| tool events | `workflow attempt, state proving: tool.passed from a state no tool owns` |
| join coverage | `workflow story: join children.in on in-progress → integrating misses terminal states [approved] of design-task` |
| result coverage | `workflow purchase, state invoiced: process.result exits do not cover [rejected]` |
| delivery invariants | rule 1 `workflow dev: review → done bypasses integration`; rule 3 `… enters the acceptance state outside accept_cr`; rule 4 `workflow dev: accepted → development leaves integration without a rollback authorization`; rule 2 is runtime-only |
| guard bypass | `workflow purchase: escalated → sending skips the approvals guard or risk marker of another entry into sending` |
| determinism | `workflow dev: two automatic transitions on delivery.integrated from accepted with the same guards` |
| resolutions | `workflow finding: limit redirect to rejected carries no resolution` |
| placement | `workflow retro: create work-item: no ancestor of retro contains it` |
| hand-off | `handoff to accounting: not a process role` |
| slot contracts | `bind scrum.work-item → story: story lacks estimate duration`; a delegated `writes` field the bound type lacks |
| nesting | `entity story: parent sprint does not contain story` |
| loop limits | `transition review → development: limit.then escalated is not a state` |
| roster and quorum | `people cafe-managers: op:dave@cafe is not in the roster at <revision>`; `approvals role manager min 3: group cafe-managers has 2 principals` (patterns such as `op:*@acme` are counted against the roster at the pinned revision) |
| signer groups | `event supplier.confirmed: signer group supplier-bridge is not bound in services` |
| degraded path | `role reviewer: on_unmet degrade names degraded_state second-review, which workflow dev lacks` |
| project-only keys | `template dev-cycle sets policy.enable: ignored` (warning) |
| tools enabled | `tool supplier.send-order is bound but not enabled` (warning) |
| field conditions | `workflow dev: guard field names review, which dev-task does not declare`; `guard field on amount: gt needs a number, money, date or date-time field` |
| check sets | `workflow dev: delivery.checks-passed names set handoff, which task-board.config.json does not define` |
| work in progress | `policy.wip.per_role: reviewer2 is not a role`; `policy.wip.per_state: dev.qa is not a state` |
| unsigned human acts | `workflow purchase has human-owned states and the project sets no policy.signatures: human acts are a guardrail only` (warning) |

The guard-bypass rule: an `approvals` guard or a `risk` marker on one non-automatic entry into a state must appear on every non-automatic entry into it, or the guard must sit in the state's `requires`. One exception makes thresholds expressible: an entry without the guard is admitted when its `field` condition admits no value that the `field` condition of any guarded entry admits, on the same field, so `amount lte 500` may skip the approval that `amount gt 500` requires; a guarded entry without a `field` condition covers every value. Verdict and `process.result` guards select an exit of their source state and are outside the rule; `blockers.clear`, `estimate` and `assignee` are hygiene guards that rework routes may omit on purpose; the delivery routes are covered by the invariant checks.

**Machine checks** (depend on what is installed and on the operator): skills installed through Curator and audited; each template tool binding declared by the orchestrator's manifest in runtime mode narrowed to the bound command; each command shim owned by the bound skill; operator bindings able to satisfy each role's needs and effort floor. They report typed "unbound on this machine" refusals and never change the portable verdict.

A throwaway prototype of a subset of the portable checks (not committed) was run over [`../examples/`](../examples/) at this revision. It covers states and initial states, owners, reachability and terminal reachability, exits, the root's capability, nesting, joins, owner-kind exits, determinism, loop-limit targets and resolutions, the guard-bypass rule with field conditions, `field` guards, check sets, work-in-progress keys, tool enablement and the signature warning; it checks templates one by one and projects for their own keys, and leaves the delivery invariants, signer groups, the roster, the degraded path, placement and slot contracts to the v2.1 prototype, whose results stand. Every template and both composed projects pass, the scrum project's `task-board.config.json` defining the check sets dev-cycle names. On seeded defects it reports each check v3 adds: a `field` guard on an undeclared field, an ordering comparison on an enum field, a work-in-progress key that names no role or state, a check set the configuration does not define, a bound tool the project does not list, and the warning for human-owned states without a signature policy. An unguarded entry into `sending` with `amount lte 500` beside approvals guarded by `amount gt 500` is admitted; with `amount lte 600` it is refused.

All examples validate against the schema. The schema refuses URLs and shell syntax in hook arguments, schedule fields and event signers; argument names that would select a program, an environment or an endpoint; package pins that are not Curator identities; program strings as tool commands; `effort` and the old `capability` need; non-operator principals in `people`; tools posing as reviewers; and action objects inside guards. Since v3 it also refuses the `null` tool form, `context_window_min`, a boolean `policy.enable.tools`, empty ranges, and `field` or `delivery.checks-passed` guards without their arguments. It cannot make the other hooks' arguments exact: their schemas come with the provider modules in PC1 (§17.3).

## 11. Generality vectors *(decided)*

The module ships a conformance suite. Each vector is a case the model must express without kernel changes; each demo playbook covers several.

| Vector | Covered by |
| --- | --- |
| custom typed fields | cafe, bookkeeping, bug-hunt, formal missions |
| human owners | cafe (approvals, receiving), bookkeeping (payment release), formal missions (captains), dev-cycle (escalation, decisions) |
| tool owners | cafe (supplier portal), formal missions (Lean kernel) |
| timers and SLA | cafe (confirmation within 15 minutes), bookkeeping (payment within three days), formal missions (stalled after 14 days) — fired by `task-board tick` |
| recurrence | cafe (weekly inventory, enabled by the project) |
| external events | cafe (`supplier.confirmed`, `supplier.shipped`), bookkeeping (`payment.received`), signed by bridges the project binds |
| hand-off between processes | bug-hunt → dev-cycle, cafe → bookkeeping (returns `paid` or `rejected`), scrum retro → delivery |
| parallel branches and join | cafe (delivery lines), dev-cycle (a story waits for its leaves) |
| quorum voting | scrum (three blind estimators), `approvals` with `min` |
| loop limits with escalation | dev-cycle (rework), cafe (approval), formal missions (five failed checks) |
| resolutions and reopen | dev-cycle (`closed` resolutions, a decision superseded by a person), automatic resolutions on limit redirects |
| per-process privacy (views) | cafe (`amount` redacted in team views); views only until a separate store exists (§5.4) |
| multi-dimensional estimates and capacity | scrum |
| routes on field values | dev-cycle (review levels, block kinds) |
| limits on work in progress | the scrum project (`policy.wip`) |

## 12. The refuter role and decision memory

A draft specification is challenged before it is approved *(decided)*:

- **Where.** A `challenge` state after a draft (dev-cycle's `spec` workflow and the epic's `hld`), owned by a `refuter` role skill.
- **Input.** The draft plus board memory: `decision` elements (statement, scope, status, links), the specifications in force and their invariants.
- **Method: proof by contradiction.** Assume the draft is implemented as written and derive consequences; look for conflicts with recorded decisions, internal contradictions (two acceptance criteria that cannot hold together), unstated assumptions and guards no role can satisfy.
- **Output: a ledger verdict with a refutation detail** *(the spelling follows §6.3)*. `accepted` with `no-contradiction` and the list of decisions checked, so the pass is auditable; `changes_requested` with `contradiction` and pairs of claim and decision; `stop_the_line` with `human-decision-needed` when only a person can choose. The same spelling is used in the spec, the examples and the role.
- **Transitions.** `contradiction` → back to the draft; `human-decision-needed` → `decision-needed`, owned by a person, who records a new `decision`; `no-contradiction` → approved.

Agents are good at finding contradictions; what they lack is memory of what was decided. The board supplies it. The refuter's runs on this document are in [`../reviews/`](../reviews/): [`2026-09-24-refuter-v0.md`](../reviews/2026-09-24-refuter-v0.md) on v1 and [`2026-09-24-refuter-v1.md`](../reviews/2026-09-24-refuter-v1.md) on v2.

## 13. Examples

| File | Shows |
| --- | --- |
| [`default.playbook.json`](../examples/default.playbook.json) | today's task-board behaviour (§14) |
| [`dev-cycle.playbook.json`](../examples/dev-cycle.playbook.json) | delivery with the delivery invariants, inline and Change Request routes, check sets `handoff` and `landing`, review levels routed by `field`, `blocked` with block kinds and a repeat limit, rework limits, `stop_the_line`, the refuter's verdicts, decisions, needs and `ask` |
| [`bug-hunt.playbook.json`](../examples/bug-hunt.playbook.json) | proving findings and offering them to the fixing process through slots |
| [`scrum/scrum.playbook.json`](../examples/scrum/scrum.playbook.json), [`scrum/project.Playbook.json`](../examples/scrum/project.Playbook.json), [`scrum/task-board.config.json`](../examples/scrum/task-board.config.json) | a cadence layer composed over dev-cycle; membership links on the sprint; delegated estimate fields; planning poker; capacity; work-in-progress limits; project-only policy; the check sets `handoff` and `landing` in the project's `task-board.config.json` |
| [`cafe/cafe.playbook.json`](../examples/cafe/cafe.playbook.json), [`cafe/bookkeeping.playbook.json`](../examples/cafe/bookkeeping.playbook.json), [`cafe/project.Playbook.json`](../examples/cafe/project.Playbook.json) | fields, humans, a tool reached through the orchestrator's runtime dependency, timers, a named schedule, signed events and their signer groups, a join, a hand-off with a returned result, privacy, and the project file that enables the tool and the schedule |
| [`prove2me-style.playbook.json`](../examples/prove2me-style.playbook.json) | missions split into lemmas, each attempt accepted only by a tool (the Lean kernel), as on prove2.me |

The last example is the expressibility check for machine-verified acceptance: prove2.me splits results into missions and milestone lemmas and accepts a contribution only when the Lean kernel checks it. Here that is a `checking` state owned by a `tool` role, whose command the kernel's executor runs outside the prover's run (§6.11). The principle carries over: every milestone should have the strongest machine-checkable acceptance available — tests and validation for code, the static validator for configurations, the refuter for specifications, a model checker for critical protocols.

## 14. Today's task-board as a playbook

[`default.playbook.json`](../examples/default.playbook.json) maps the current kernel (skill-project-management origin/main `4b0a75d6`). Every row is reproduced unless it says otherwise.

| Today | Code | In the default playbook |
| --- | --- | --- |
| types epic, story, task, bug | `pkg/board/types.go:13-16` | `entities` |
| epic has no parent; story under epic; task and bug under story | `types.go:184-218` | `root`, `parents`, `contains` |
| ten statuses | `types.go:23-32` | states of `leaf` and `container` |
| no transition table: any status may be set when its gates pass | the gates below | transitions `from: "*"` |
| leaves never leave done and closed | `pkg/board/escalation.go:105-107`, `pkg/board/terminal_status.go:16-21` | `terminal`, no `reopen` |
| containers aggregate from children; aggregation reopens done and closed containers and moves them into and out of integrating; it obeys the blockers check, and blocked containers are left alone | `escalation.go:65-68, 73, 140, 151-172, 198-200`; `validation.go:46-58` | `derived_status` (§5.1) |
| container statuses may also be set directly; only the close is gated | `pkg/board/validation.go:34-43` | declared `*` transitions on `container` |
| forward statuses refused while blockers are active, on every status change | `validation.go:46-58`; `consistency.go:176` | guard `blockers.clear`, also on the integration exits |
| task and bug must be estimated before development; legacy records without an estimate and idempotent writes pass | `validation.go:60-68` | guard `estimate` with `grandfather_legacy` |
| manual moves to development or to-review need an assignee; to-dev deliberately does not | `validation.go:73-90` | guard `assignee` with `manual_only` |
| a spawned run whose role ends in to-review hands off its own element with its checklist complete and `<ID>_results.md` stored; reviewers (empty end status) and orchestrator-archetype runs (no lifecycle statuses) are not gated | `tools/board-cli/cmd/producer_handoff_guard.go:71-78`, `pkg/board/handoff.go:122`, `pkg/board/errors.go:26-44`, `goal_role.go:25-33` | guards `checklist.complete` and `outcome.present` with `applies_to: { actor: run, own_element: true, role_ends_in: to-review }` |
| roles are open-ended: any installed role can be spawned | `tools/board-cli/internal/spawn/role.go:83-93, 303-335` | owners `{ one_of: { capability: [...] } }` |
| inline work without a Change Request is reviewed and finishes through the acknowledged done route | `consistency.go:105-141` | `* → reviewing` unguarded by delivery; inline `* → done` |
| integrating is entered only through `accept_cr` | `pkg/board/consistency.go:166-167` | `via: accept_cr`, `locked` |
| integrating is left to done through integration, external integration or a restore, and to to-dev through a rollback (landed rework, invalidation, converge) | `consistency.go:169-172`; `tools/board-cli/internal/integration/start_landed_rework.go:67`, `invalidate.go:117-118`, `converge.go:426-427`, `restore_done.go:85` | `via` triples with `delivery.*-authorized` guards |
| `worktree integrate` accepts only the story-final leaf as the story's integration unit; the orchestrator integrates the story | `pkg/board/version_control.go:151-156` | `integrating` owned by the orchestrator, no tool invocation on entry |
| done is refused while a Change Request exists; inline work passes | `consistency.go:128-141, 158` | guard `delivery.cr-absent` |
| commit acknowledgement for the enforced types when policy confirms | `version_control.go:89-123, 277-279` | guard `delivery.commit-acknowledged`, `policy.delivery.version_control` |
| rejection routes to to-dev and records changes requested | `tools/board-cli/cmd/change_request.go:753, 825, 856` | `reviewing → to-dev via reject_cr` with `record.verdict` `changes_requested`, beside the manual route |
| reviewer verdicts are `accepted`, `changes_requested`, `stop_the_line`; a reviewer's Stop-The-Line parks its element at blocked with a new evidence packet | `pkg/board/goal.go:40-49`; `tools/board-cli/internal/spawnruntime/goal_acceptance.go:51-96, 255-256`; `spawnruntime/changerequest.go:403-405` | `reviewing → blocked` with `outcome.present` for the run's own element and `record.verdict` `stop_the_line`; the ledger gains `rejected` in PC1 |
| linked blocked items, leaves and containers, release to backlog when blockers finish | `pkg/board/blocked_release.go:8-24, 76-87` | `on: { event: blockers.terminal }` on both workflows |
| orchestrator (and coordinator) has no lifecycle statuses | `pkg/board/goal_role.go:10-11, 25-33` | `root` |
| review policy defaults to required | `types.go:106-133` | `policy.review` and the field `review` |
| task class | `types.go:136-141` | field `task_class` |
| rework loop detection: threshold `max(3, 2 × median)` | `internal/changerequest/loopdetect/loopdetect.go:114, 126-129` | `policy.delivery.loop_detection` (`floor` 3, `factor` 2) |
| the landing-gate suite `spawn.worktree_isolation.validation.commands`: run by the producer at Change Request build with evidence bound to the candidate tree, re-run by integration when that binding is missing or broken; an absent block refuses integration, an empty list runs nothing | task-board `references/task-board-config.md`, "The landing-gate suite" | a local `landing` check set (§6.12) |
| child runners per control root, `spawn.max_parallel` | `references/task-board-config.md`, "Spawn policy" | `policy.wip.board` (§6.8) |
| concurrent heavyweight suites per OS account, `spawn.validation.max_concurrent_suites` (default 2) | same | machine capacity in `~/.curator/runtimes.toml` (§2.3) |

Accepted differences and open points of the mapping:

1. The aggregation order of container statuses stays kernel code; the configuration only declares `derived_status`.
2. `one_of … by assignment.role` needs a structured role field; today the role is recorded only inside the assignee display string (`tools/board-cli/cmd/spawn.go:3856`). PC1 adds the field; until then, and after a move into a state that does not admit the recorded role, the root orchestrator owns the element.
3. `closed` carries no resolution today.
4. Dependency links (`blocked_by`) are kernel data; the configuration refers to them only through `blockers.clear` and `blockers.terminal`.
5. Loop detection counts rework across Change Request revisions; it stays delivery policy, distinct from a transition `limit`.
6. `defer_cr` and `withdraw_cr` are not mapped yet.
7. Review levels *(decided 2026-09-24)*: the field `review` and the `field` guard express them as routes, as dev-cycle shows. In the default, whose transitions are permissive (`*`), the field changes no route. Two effects stay delivery and ledger behaviour keyed on the same field: the goal ledger accepts a `none` item on the producer's outcome alone (`tools/board-cli/internal/spawnruntime/goal_acceptance.go:167-170`), and the close-landed method `research_outcome_only` admits a research element without a Change Request only with `review=none` and `task_class=research` (`tools/board-cli/internal/integration/landed_methods.go:553-565`). PC1 maps that method onto a `via` command guarded by `field`. `light` differs from `required` today only in the ledger's order of review strength (`pkg/board/goal_mutation.go:268-279`); both route to review.
8. The version-control policy lives in `task-board.config.json` today with authority class `project_policy` (`pkg/remoteconfig/authority_schema.go:117`). *Decided (2026-09-24):* `Playbook.json` owns it (`policy.delivery.version_control`), and the config's `version_control` keys are read for one release with a warning. The enforced types stay fixed by the delivery module (`version_control.go:18-19`).
9. The linkless `blocked` state is owned by the orchestrator in the default, because today's kernel does not restrict who unblocks; stricter playbooks give it a human owner.
10. Evidence freshness (§6.3) is today's rule for reviewer verdicts only; PC1 extends it to approvals and outcomes, which changes nothing the default declares.
11. The default keeps today's guard `delivery.integration-evidence` on its `integrate` route; that evidence is the result of the `landing` check set (§6.12). Today's key `spawn.worktree_isolation.validation.commands` maps onto `checks.landing` with `where: local`, without a change in behaviour.

## 15. Integration plan

| Step | Delivers |
| --- | --- |
| PC0 | this draft, the schema, the examples; the refuter's first run (v1 → v2) and second run (v2 → v2.1); the operator's decisions (v3) and a refuter pass on the sections v3 changed, after which the specification is frozen |
| PC1 | the module: schema, parser, resolver (`extends`, `uses`, slots), validator CLI with portable and machine checks; a thin seam in task-board that loads the project file from the head of the default branch (§2.4) and `default.playbook.json` with identical behaviour or documented differences; new types from `Playbook.json` on the generic engine with the kernel hooks of §6.3, the `field` guard included; the structured `assignment.role`; `rejected` in the goal ledger; `policy.wip` (§6.8); the version-control policy read from `Playbook.json`, with the config's keys read for one release with a warning; local check sets over today's landing-gate suite (§6.12); the legacy operator layer read from today's config until `models.toml` exists (§2.3); the reference prompt size for speed facts (§9); the constructor-ready schema (`ui` blocks, lossless round trip). Because the demos follow PC1 directly *(decided)*, PC1 also carries **the minimal delivery seams over today's commands**: `delivery.cr-published`, `delivery.review-ready`, `delivery.cr-accepted`, `delivery.cr-absent`, `delivery.commit-acknowledged` and the evidence and authorization guards mapped onto today's authorization options; `delivery.integrate` mapped onto the orchestrator's `worktree integrate`; and `delivery.integrated`, `delivery.integration-failed` and `delivery.rework-started` emitted when today's integration transaction, its failure, and landed rework, invalidation or convergence complete. It also carries **the local hand-off on one board** (§6.5). |
| Demos (right after PC1) | `dev-cycle`; `bug-hunt` → `dev-cycle`; `scrum` over `dev-cycle` — all on one board *(decided 2026-09-24)*. The cafe with bookkeeping follows PC3 and the waggle amendment |
| PC2 | role skills and the `curator-playbook-dev-cycle` package on Skillfile schema 2, installed on demand once Curator ships activation `on-demand` and the skills launch channel (§8.1, §8.3), with the description stopgap until then; the JSON Schema of `data/role.json`; task-board resolves roles from the project overlay `.roles` first, then installed role skills, then `~/.roles` read-only for one release with a warning; `setup.sh` stops copying `~/.roles` (this is roadmap milestone M1) |
| PC3 | the delivery module proper: the PC1 seams become a module whose invariants it checks itself; hosted check sets and the local-mirror rule (§6.12); `task-board tick` with the tool executor (§6.9, §6.11) and its user-level system timer; signed events and approvals once the waggle amendment lands with CM2; hand-off across boards through waggle (CM1 and the amendment) |

PC4 of the earlier plan is dropped *(decided)*: a playbook is a collection of skills. The constructor-ready schema, which the roadmap's PC4 row also carried, is part of PC1, and the Curator changes are the Curator goal (§8.3). The roadmap rows live in the wiki and are updated there. Details: [`track.md`](track.md).

## 16. Names

Specifications use technical terms. User-facing material uses the Apiary names *(decided)*: Hive = one node's agent system, Comb = a process, Keeper = the main loop, Bee = the agent loop, Waggle = the messaging protocol; a playbook keeps its name. The ecosystem wiki records the mapping; this is the only place in the specification that mentions it, and no identifier uses the user-facing names (the v1 owner kind `comb` is now `process`).

## 17. Decisions and open questions

### 17.1 Decided in the design discussion of 2026-09-24

- the file and its format, and its four core sections;
- one owner per element in each state, where a state may admit several roles and assignment picks one, with helpers and `one_of`;
- owner kinds and who may move work out of each; authority outside the model, with signatures for risky human acts;
- playbooks as data only; side effects enabled by the project; admission by `extends`/`uses`;
- tools only through Curator skills: the orchestrator's manifest depends on each provider in `runtime` mode narrowed to exported commands, the kernel runs only commands of skills installed through Curator, and a project may rebind a tool (wiki decisions.md:84);
- needs without effort; effort in the operator file `~/.curator/models.toml`, with `run` candidate lists and effort ranges (decisions.md:83), and a committed project floor;
- relaxing a requirement permanently only in the project file, and temporarily through `ask` for the approver's own runs;
- guards, actions and events from modules; the delivery module and its invariants; `task-board tick` until the main loop;
- `extends` and `uses` with slots and membership links;
- roles as skills, and playbooks as skill collections installed by one Skillfile schema 2 selector, with Curator releasing schema 2 and adding `directory` to `dependencies.skills`; package names `curator-playbook-<purpose>`;
- the static validator with portable and machine checks; the generality vectors, including per-process privacy as a views vector; the refuter;
- the demos dev-cycle, bug-hunt and Scrum right after PC1; PC4 dropped; the default playbook as the PC1 target.

### 17.2 The operator's decisions on v2.1

| # | Decision | Sections |
| --- | --- | --- |
| 1 | every resolution the author proposed in answer to both refuter runs is decided | throughout |
| 2 (H1) | the kernel reads `Playbook.json` from the head of the default branch of the repository that owns the board, never the Story's integration base and never the working tree; validator, previews and a constructor read the working tree; a project without git reads the file as it is and warns | §2.4 |
| 3 (H1) | without a signature policy an operator's CLI act outside any run counts as human, stated as a guardrail; the session host marks hosted sessions as `session`; the validator warns about human-owned states without `policy.signatures` | §4.2, §10 |
| 4 (H7) | `user_verified` defaults to `required` for approvals, relaxations and halts whenever a project sets a signature policy; the project may lower it to `if-supported` | §4.2 |
| 5 (H7) | the waggle amendment — a board class for approvals and signed transitions, an event class with expiry and de-duplication, a service principal kind admitted only for that class, a `coord` type that returns a hand-off's result, user verification available to board policy — is written into the waggle specification now and implemented with CM2 | §4.2, §6.5, §6.9, track |
| 6 (H2) | without `models.toml` automatic selection never chooses a metered runtime; only the operator's own file admits metered billing; the project only narrows; an undeterminable billing class counts as metered; no questions mid-run, a command writes the choice | §2.3, §9 |
| 7 (H5) | one need vocabulary: every term in `require` or `prefer`; ranges for `context_window`, `pp`, `tg`, `ttft`; speed facts measured, else declared by the registry, else unknown | §9, schema |
| 8 | an exception that expires mid-run lets the run finish; exceptions never widen `[allow]`; standing answers in `models.toml` `[answers]`, recorded on the board, never applied when relaxations need signatures | §2.3, §9 |
| 9 (H3) | a tool runs only when listed in `policy.enable.tools` (names or `all`); not listed means disabled; no `null` form | §8.5, §6.11, schema |
| 10 | a user-level system timer (launchd, systemd user timer) calls `task-board tick`; the Keeper takes it over later | §6.9, §6.11 |
| 11 | role skills and the root orchestrator's skill install with Curator activation `on-demand` and reach a launch through the skills launch channel; description stopgap until Curator ships it | §8.1, §8.2, §8.3, §15 |
| 12 | role metadata in `data/role.json` with its own JSON Schema (PC2) | §8.1, §15 |
| 13 | the `field` guard; review levels as a field with routes | §6.3, §10, §14 |
| 14 | the goal ledger gains `rejected` | §6.3, §10, §14, §15 |
| 15 | process rules in `Playbook.json`, runtime and board settings in `task-board.config.json`, model choices and machine limits with the operator; legacy keys read for one release | §2.1, §2.3, §6.8, §14 |
| 16 | the operator's files live together under `~/.curator/`: `models.toml`, `runtimes.toml`, `engines.toml`, `network.toml` | §2.3 |
| 17 | `policy.wip` bounds active runs per board, role and state | §6.8, §14 |
| 18 | dev-cycle gains `blocked` with a required `block_kind` and a repeat limit escalating to the operator; the default is unchanged | dev-cycle example |
| 19 | per-element skills and preferences are future work after PC1 | §9 |
| 20 | check sets: playbooks name them, `task-board.config.json` runs them locally, on the hosting provider, or both | §6.3, §6.12, §14, §15 |

H4 (keep the effort floor), H6 (the demos and the PC rows) and H8 (one owner per element in each state; privacy as a views vector; the refuter's verdicts) were decided earlier and stand.

### 17.3 Open questions

1. **First start without `models.toml`.** What the first launch shows an operator who has no file: whether a guided command proposes bindings for the project's roles from the runtimes installed on the machine, and how a refusal explains the missing admission (§2.3, §9).
2. **Commands for the operator's files.** Proposal, pending the operator: one command family through the `curator` umbrella — `curator models`, `curator runtimes`, `curator inference engines`, `curator network` — that lists, shows, adds, sets and removes entries, keeps comments and order, validates before writing, writes atomically with a backup and mode 0600, never stores a secret value, offers `--json`, and from inside an agent session refuses any change that widens admission (`[allow]`, `[answers]`).
3. **The reference prompt size** for `pp` and `ttft` measurements (PC1, §9).
4. **Per-hook argument schemas** for every provider module (PC1). The schema covers the arguments of `field` and `delivery.checks-passed` already.
5. **The remaining mapping points of §14**: `defer_cr` and `withdraw_cr`, and the close-landed methods as `via` commands (PC1).
6. **How a process is created and registered in a hive**: the Keeper's process registry, outside this specification (wiki open questions).
7. **Outside this repository**: this text amends agent-selection §3, §7 and §8 and LP D2 and D9; the drafts in skill-project-management take the amendments when those tracks resume. The waggle amendment lands in the waggle repository; on-demand skills land in Curator (curator-spec issue #91, curator issue #91).

**What this means for PC1.** PC1 can start now: everything it implements is decided. The waggle amendment gates PC3's signed acts and the cafe demo only; Curator's on-demand skills gate PC2's clean install, with the description stopgap until then. The three first demos need none of the open answers.

## 18. Change log: v1 → v2

Each finding of the refuter's first run ([`../reviews/2026-09-24-refuter-v0.md`](../reviews/2026-09-24-refuter-v0.md)) and where v2 answers it. Section numbers in this table refer to v2; v2.1 keeps them, except that the open questions of §17 were renumbered (§17.3). v3 keeps them too, adds §6.12 and §20, and replaces §17.2 and §17.3, so references to open questions in §18 and §19 name the v2 and v2.1 lists.

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

## 19. Change log: v2 → v2.1

Each finding of the refuter's second run ([`../reviews/2026-09-24-refuter-v1.md`](../reviews/2026-09-24-refuter-v1.md)) and where v2.1 answers it. The statuses of H1–H8 in this section are those of v2.1; §17.2 and §20 record the operator's decisions.

### New findings

| Finding | Resolution in v2.1 |
| --- | --- |
| N1 authority policy in the working tree (blocking) | **PROPOSED default (H1):** the kernel reads the project file at a pinned revision and records its revision and digest; an edit takes effect when it lands; the roster verifies the commit when it exists (§2.4; open question 1) |
| N2 no waggle form for board approvals, events, service principals, hand-off results | described as a dependency on a waggle amendment (H7); PC1's local hand-off needs none of it; signed approvals, signed events and cross-board hand-off wait for it (§4.2, §6.5, §6.9, §15; track; open question 5) |
| N3 signatures cannot show a person acted | `policy.signatures.user_verified` (`required`, `if-supported`); **PROPOSED default (H7):** `required` for approvals, relaxations and halts; approvals signed only through the command that renders the payload (§4.2; schema) |
| N4 templates choose event signers | events name a signer group (`signed_by`); only the project binds it (`services`), in the committed file; unbound groups refuse the event; bridges verify the provider's webhook authentication first (§4.2, §6.9; cafe, bookkeeping) |
| N5 tool invocation not gated, not disableable | tool invocations join the project-enabled side effects (`policy.enable.tools`, **PROPOSED default (H3)**); a project disables a tool with `null` (§8.5; schema) |
| N6 decided tool mechanism replaced | decisions.md:84 restored: the orchestrator's manifest depends on each provider in `runtime` mode narrowed to exported commands; providers live outside the `skills/` collection; "own package" dropped; admission requires the manifest declaration (§8.1, §8.5, §10) |
| N7 tool verdict within reach of the triggering actor | a kernel-owned executor with a verified shim or `script-worker-v1`, a sanitized environment, an invocation contract, a timeout into `tool.failed` and idempotence per entry; people may leave tool states (§6.11, §4.2) |
| N8 owner-kind table forbids the cafe's exits | automatic exits leave any state; people may override tool- and process-owned states; the cafe restructured (`sending` left on `tool.*`, `awaiting-confirmation` owned by the orchestrator, `invoiced` left on `process.returned`); owner-kind exit and result coverage checks (§4.2, §10; cafe) |
| N9 story with a design task never integrates | the join lists `approved`; join coverage check (§6.1, §10; dev-cycle) |
| N10 invariant 3 against the default | rule 3 scoped to elements under a Change Request (acceptance of a published revision, integration only through acceptance); inline review stays expressible; dev-cycle adds `delivery.review-ready`; checks for rules 1, 3 and 4, rule 2 runtime-only (§6.10, §10) |
| N11 demos after PC1 need PC3 and waggle | H6 decided the demos stay right after PC1, so PC1 carries the minimal delivery seams over today's commands and the local hand-off on one board; the cafe waits for PC3 (§15; track) |
| N12 scrum writes into dev-cycle's leased scope | lease-crossing writes defined; a slot's `writes` delegates named fields on binding; membership stored on the sprint (§7.2; scrum) |
| N13 committed effort floor not portable | H4 decided to keep the floor; per-row semantics; a row without effort satisfies it vacuously; an unsatisfiable floor goes through `on_unmet` with a typed refusal (§9; schema) |
| N14 automatic selection bypasses admission | **PROPOSED default (H2):** project admission ∩ machine facts ∩ needs; no metered runtime unless admitted; agent-selection §7 and LP D9 amended for the case without a file (§2.3, §9; open question 2) |
| N15 need vocabulary drops agent-selection S1 | **PROPOSED default (H5):** `class`, `capabilities`, `context_window_min`, `reasoning` added; `capability` renamed; agent-selection §3 spelling amended (§9; schema; open question 4) |
| N16 independence checked against a binding | evaluated against the provenance of the run that produced the evidence under review; qualified role names accepted (§9; schema) |
| N17 tick has no scheduler and no cross-clone lease | scheduler and lease store named and listed as dependencies; firing keys, including (schedule, due time), kept in the lease store; one tick host per shared board before CM2 (§6.6, §6.9; track) |
| N18 operator file still marked proposed | §2.3 decided; `run` candidate lists and patterns; the next candidate on unavailability; a read failure refuses (§2.3) |
| N19 invariant 4 strands the failed story; invariants escapable | invariants bound to element state; `delivery.integration-failed` carries a rollback authorization (§6.10; dev-cycle) |
| N20 aggregation obeys blockers | stated; §14 row and the default's description corrected (§5.1, §14) |
| N21 three more differences from the code | `applies_to.own_element` and `role_ends_in`; `blockers.clear` on the three integration exits; the review policy `none` listed as a difference (§6.2, §14; default) |
| N22 verdict spelling and routing | the ledger's spelling; the refuter's verdicts as ledger verdicts with a detail; `rejected` as the one extension; `stop_the_line` routed in the default and dev-cycle (§6.3, §12, §14; all examples) |
| N23 signature categories undefined | categories defined by construct; `risk` marker on transitions; new category `overrides` (§4.2, §6.2; schema; cafe, bookkeeping) |
| N24 `people` admits agents and uncountable patterns | only `op:` principals; patterns counted against the roster at the pinned revision; the roster check is portable (§4.2, §10; schema) |
| N25 command names are global shims | packages prefix their command names; a machine check verifies the shim's owner (§8.5, §10; examples) |
| N26 "installed" and "package identity" not Curator's terms | templates resolve through Curator's locks in the order project, hybrid, global, with ambiguity inside one scope an error; `extends.package` is a Curator package identity (§7.1, §8.2; schema) |
| N27 evidence freshness | verdicts, approvals and outcomes count only when recorded after the last entry into the source state, and for Change Request work against the evaluated revision (§6.3, §6.4) |
| N28 dev-task classes that cannot reach done | inline route guarded by `delivery.cr-absent` and `delivery.review-ready` (dev-cycle) |
| N29 the schema accepts what §10 says it refuses | the schema refuses URLs, shell syntax and program-selecting argument names in hook arguments, schedule fields and signers, and non-identity package pins; the claim narrowed; owners checked on resolved non-terminal states; the `via` and `remove` descriptions fixed; per-hook argument schemas stay open (§10; schema; open question 11) |
| N30 tools run without Curator | refused without Curator; the operator-confirm path removed (§8.4) |
| N31 reopen transitions without an actor rule | a person may reopen; the root orchestrator only when the type's initial state is not human-owned (§4.2; dev-cycle) |
| N32 guard-bypass check undefined | defined for `approvals` guards and `risk` markers on non-automatic entries, and run (§10) |
| N33 events lost in another state | events kept per element until they fire or the element ends; state-describing kernel events re-evaluated on entry; unmatched events logged and kept (§6.9; cafe) |

### First-run findings left open by v2

| Finding | Resolution in v2.1 |
| --- | --- |
| F1 authority | the pinned revision (H1); the guardrail stated as such, with the actors that count as human today; signatures that show user verification (§2.4, §4.2) |
| F2 package bytes and side effects | through N4, N5, N6 and N29 |
| F7 `one_of` | H8's wording recorded as decided; a stale recorded role falls back to the root (§4.2, §6.1) |
| F9 effort | through N13 |
| F10 who relaxes; agent-selection's project layer | `policy.data` carries the data policy; `Playbook.json` supersedes agent-selection's committed project layer for needs, admission, data policy and waivers; the remaining settings are open question 9 (§9) |
| F11 approval storage and reach | exceptions are a kernel `exception` record in every playbook; "own runs only" is a guardrail until runs are bound to principals (§9) |
| F12 `degrade` (minor remainder) | the degraded path's entry and exit, and a validator check (§9, §10) |
| F15 timers before the main loop | through N17 |
| F16 sequencing | H6 decided; PC4 dropped and the constructor-ready schema folded into PC1; the roadmap rows are updated in the wiki (§15) |
| F25 `comb`, verdict spelling | `process` kind kept; the verdict spelling follows the ledger (N22); the glossary line is outside this repository (open question 14) |
| F26 schema and text | the root must be an orchestrator; `not_same_publisher_as` takes qualified names; owners on resolved non-terminal states; per-hook argument schemas open (§4.4, §9, §10) |
| F31 roles as skills | precedence: project entry, then template entry, then `data/role.json`; ambient triggering stays open (§8.1; open question 7) |
| F32 privacy | H8 decided: per-process privacy is a views vector again (§5.4, §11) |
| F34 where the specification lives | outside this repository: the wiki roadmap rows, and the supersession note in skill-project-management PR #389 (open question 14) |
| F35 goal ledger | closed through N22: verdicts use the ledger's vocabulary (§6.3) |

## 20. Change log: v2.1 → v3

The operator's decisions of 2026-09-24 (§17.2) and where v3 applies them.

| Decision | Change in v3 |
| --- | --- |
| 1 author's resolutions | every *proposed* marker removed; the statements are decided (header, §4.2, §6.1–§6.11, §7, §8.2, §9, §12) |
| 2 revision in force (H1) | the head of the default branch of the repository that owns the board, whatever Story is in scope; no branch or fork runs its own rules; no-git projects warn (§0, §2.4) |
| 3 unsigned human acts (H1) | the guardrail stated as decided; the session host marks hosted sessions; validator warning (§4.2, §10) |
| 4 user verification (H7) | `required` by default for approvals, relaxations and halts when a project sets a signature policy; `if-supported` as the project's opt-down; Secure Enclave keys named (§4.2; schema) |
| 5 waggle amendment (H7) | written into the waggle specification now, implemented with CM2 (§1, §4.2, §17.2; track) |
| 6 selection without the operator file (H2) | never metered; only the operator's file admits metered billing; undeterminable billing counts as metered; a command writes the choice; first start open (§2.3, §9, §17.3) |
| 7 need vocabulary (H5) | every term in `require` or `prefer`; ranges; `context_window` replaces `context_window_min`; speed terms `pp`, `tg`, `ttft`; speed facts measured, declared or unknown (§9; schema; dev-cycle) |
| 8 exceptions and standing answers | expiry lets running runs finish; exceptions never widen `[allow]`; `[answers]` in `models.toml` (§2.3, §9) |
| 9 tool enable gate (H3) | `policy.enable.tools` lists tools or says `all`; not listed means disabled; the `null` form removed (§3, §8.5, §6.11; schema) |
| 10 tick scheduler | a user-level system timer; the Keeper later (§0, §6.9, §6.11; track) |
| 11 on-demand role skills | Curator activation `on-demand` and the skills launch channel, identity delivery, description stopgap; Curator's third change (§0, §8.1, §8.2, §8.3, §8.4, §15; track) |
| 12 role metadata | `data/role.json`, not frontmatter, with its own JSON Schema in PC2 (§8.1, §15) |
| 13 field conditions | the kernel guard `field`; review levels as routes; the guard-bypass exception for disjoint conditions (§5.2, §6.3, §6.8, §10, §11, §14; schema; dev-cycle) |
| 14 `rejected` | the goal ledger gains it in PC1; the validator's refusal removed (§6.3, §10, §14, §15) |
| 15 where settings live | the project's three files; process rules in `Playbook.json`; legacy keys read for one release; model keys and machine limits with the operator (§2.1, §2.3, §6.8, §9, §14) |
| 16 operator files together | `~/.curator/models.toml`, `runtimes.toml`, `engines.toml`, `network.toml`; LP D2 and the network file location amended (§2.3) |
| 17 work in progress | `policy.wip` with `board`, `per_role`, `per_state` on active runs; `spawn.max_parallel` maps onto it (§6.8, §14; schema; scrum project) |
| 18 block kinds | dev-cycle's `blocked` state, `block_kind` field and repeat limit (dev-cycle example) |
| 19 per-element needs | marked *future* (§9) |
| 20 check sets | `delivery.checks-passed` and `delivery.integrate`'s `checks`; `checks` in `task-board.config.json`; local, hosted or both; per task class; the local-mirror rule (§6.3, §6.12, §14, §15; schema; dev-cycle) |

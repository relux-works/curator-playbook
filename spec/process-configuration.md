# Process configuration: specification

Status: **DRAFT v4.7**, 2026-09-29, the freeze candidate. Not normative until frozen. Roadmap: `wiki/roadmap/ecosystem-roadmap.md` (process-configuration line). Track: [`track.md`](track.md). Schema: [`../schema/playbook-v1.schema.json`](../schema/playbook-v1.schema.json). Examples: [`../examples/`](../examples/). Trust: [`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) in relux-works/curator-trust.

A team's process is code today. task-board's element types, statuses, gates and roles are compiled into its kernel, so a team that works differently — or a cafe ordering supplies, or a group formalizing a proof — cannot describe its process without changing the board. This specification answers one question: how does a project state who does what, which kinds of work exist and how each moves, in one file that every teammate can share and a validator can check before anything runs? It covers that file, its schema, extension and composition, distribution through Curator, model needs, and validation. The agent loop, an always-on scheduler and the goal ledger stay outside it (§1).

v4.0 split the draft in two. This core keeps the process model, composition, distribution, needs, the validator, delivery, tools as skills and the semantics of delegation, and it is the candidate for the freeze. The trust machinery went to [`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md). It covers who counts as a person, confirmations and their records, capability grants, signatures and key assurance, key enrollment and the web of trust, the operator companion app, confirmation of a headless node, roster and revocation checks, and verification of grants. v4.1 moves that document into its own private repository, relux-works/curator-trust, where it continues its own refuter rounds under principles the operator accepted on 2026-09-25. The core reaches it through one interface: a version takes effect when the trust component reports it confirmed, or when capabilities a person granted cover every one of its differences (§2.4).

v4.1 applies the operator's decision on which changes wait for a person (§17.3 item 13): **capabilities**, granted once the way a phone app asks for a permission, with **batched touches** for everything else and an explicit **yolo mode**. These replace v4.0's allowlist and its path check (§2.4). v4.1 also answers the seventh refuter run ([`../reviews/2026-09-24-refuter-v6.md`](../reviews/2026-09-24-refuter-v6.md)); §27 maps each finding. The history of drafts v1–v4.0 is in the change logs §18–§26, and the seven refuter runs are in [`../reviews/`](../reviews/).

v4.2 records the operator's discussion of 2026-09-25 after v4.1. Two decisions: role skills reach their runs through per-role Curator profiles instead of on-demand activation (§8.1), so curator-spec issue #91 and curator issue #91 are closed; and the `properties` block and a **decision consistency check** come after PC1, outside the freeze (§17.3 items 17 and 18). One proposal: **dream mode** (§6.13), under which a process is rehearsed with every tool in a no-side-effect mode while the agent knows it is dreaming, and which is also the node's nightly offline cycle with a quarantined dream memory. And an explicit bound on dev-cycle's rollback loop (§13). §28 maps each change.

v4.3 records the operator's answers to v4.2. Loop limits become **template parameters** a project tunes without restating transitions, down to unbounded (§7.3): the operator has real cases with thirty legitimate rounds of rework, so dev-cycle's rework and rollback loops are unbounded by default, as today, and the validator warns about unbounded cycles instead of refusing them. **Entering a dream** is approved once per process and conditioned by a schedule and by capacity — subscription quota, quota that would expire unused, a ready local model — and a dreaming orchestrator runs in a session of its own with a profile made for dreaming (§6.13). §29 maps each change.

v4.4 applies the command-line shape the operator adopted on 2026-09-27 (wiki `research/cli-shaping-review.md`). It renames commands and changes no rule. The validator is the Curator provider `curator playbook` (`check`, `explain`, `bind`). A person's confirmation of a process version is `task-board trust confirm`, and a version is proposed without git through `task-board config propose`. The workstation path takes `--node`. Each operator file has one command family named after its noun, and Curator's own settings, the skill sources included, stay in Curator's existing configuration instead of a new `curator.toml`. Beekeeping words appear only in `keeper`'s nouns and the apiary's directories (§16). §30 maps each change.

Every statement in this revision is *decided* unless §17.3 lists it as open or pending, or it is marked *future*. The answers to the refuter runs are the author's and align the text with a recorded decision, the code or this specification's own rules; the operator confirms them with the freeze. Where the author filled a gap in an operator's decision, §17.2 says so.

Supersedes the roles-and-routes model of `skill-project-management/.specs/drafts/playbooks.md`: the core object is no longer a list of roles with routes between them but entity types with workflows whose states are owned by roles. Related: [`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) (who counts as a person and how a person's acts are shown), `skill-project-management/.specs/drafts/agent-selection.md` (selection layers; §9 amends its §3, §4, §7 and §8), `skill-project-management/.specs/drafts/launch-profiles.md` (LP; §9 amends D1's model admission and D9 for the case without an operator file, §2.3 moves D2's runtime catalog), curator-spec `protocol/skillfile-sources.md` (Skillfile schema 2, package identities), curator-spec issue #91 (on-demand skills, closed on 2026-09-25 in favour of per-role profiles, §8.1), curator-spec issue #95 (command-only installs, §8.3), `protocol/core.md` §3, §4.1 and §4.4 (execution boundary, command shims, skill dependencies), `protocol/environments.md` (package-influence boundary), `profiles/manager.md` (scope precedence), `waggle/spec/waggle.md` (approver signatures, hand-off offers, scope leases, the roster at a pinned revision).

## Terms

The specification uses technical terms. The operators' user-facing names stand beside them here, where the terms are first used; the text uses them nowhere else, and §16 records the naming decision *(decided 2026-09-25)*:

| Technical term | User-facing name | Meaning |
| --- | --- | --- |
| the whole system | apiary | every node and process of one owner; one node is a small apiary |
| node | yard | one machine |
| main loop | keeper | the always-on loop of a node, which manages its processes |
| process | hive | one domain — a family calendar, a cafe, the development of a project — with its project directory, board, playbook and root orchestrator |
| root orchestrator | queen | the single orchestrator of a process that is started from outside it; it starts its own sub-orchestrators and runs |
| board | comb | the process's board; its elements are the cells |
| run | bee | one agent run |
| external participants | wild bees | agents of other owners working on part of a process under leases and signatures |
| process package | nuc | a published playbook (`curator-playbook-<purpose>`) from which a new process starts |

A playbook keeps its name. No identifier uses the user-facing names.

---

## 0. The design on one screen

- A project describes how it works in one file, `Playbook.json`, in its root beside `Skillfile.json`. *decided*
- The file has four core parts: **roles** (aliases bound to role skills, people, tools or other processes), **entities** (element types, nesting, fields), **workflows** (states and transitions) and **policy**. *decided*
- **Each element has exactly one owner in each state; a state may admit several roles and assignment picks one**; helpers work for the owner. *decided (H8)*
- **Playbooks are data only.** They never name a program, script, webhook or URL. Everything that executes is a skill installed through Curator: a tool is a command that a provider skill exports, and the orchestrator skill reaches it through a runtime dependency narrowed to that command. Schedules, automatic spawns and tool invocations a package proposes take effect only when the project enables them; a tool runs only when the project lists it. *decided*
- **Authority stays outside the model.** Agents move the steps a playbook gives them; which steps belong to people is the playbook's choice, and signatures apply only to the acts a project marks. An agent run never moves work out of a human-owned state, never records a human approval and never posts a tool or external event, except as a person's **delegate**: a person may delegate named acts to their own orchestrator sessions within limits the project sets, and validates only what the delegation leaves out. Risky human acts need a signature that shows a person acted; what counts as a person and as such a signature is [`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md). *decided*
- **The process in force is the committed one that has taken effect.** The kernel reads the project's `Playbook.json` from the head of the default branch of the repository that owns the board, never from the working tree. A commit is enough; git hosting and pull requests are optional.
  - **What takes effect on commit.** A proposed version takes effect on commit only when **capabilities** that a person granted once cover every one of its differences. A process asks for a capability the way a phone app asks for a permission, and the capabilities cover narrow kinds of change: preferences, estimates, helpers, limits on agents' work, fields no guard reads, schedules within limits, a named tool, audited updates of role instructions.
  - **What waits.** Every other change takes effect only once the trust component reports a person's confirmation: workflow structure, anything that carries authority, and the templates, tools and roster the process resolves. One confirmation covers every pending change (§2.4, [`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §16).
  - **yolo mode.** An explicit, loud yolo mode trusts every change where no one else shares the board and no delegation covers money.
  - Until a change takes effect, the kernel runs the version in force in full.

  *decided 2026-09-25 (§17.2 row 35)*
- A project **extends** one published playbook and writes only its differences, and **composes** several, binding their slots; its `extends`/`uses` is its list of admitted playbooks. *decided*
- **Roles are skills.** A published playbook is a repository whose `skills/` collection holds the root orchestrator, whose data carries the template, and one skill per role. One Skillfile schema 2 selector installs the whole process; tool providers sit outside the collection. Role skills reach only the runs of their role: each run launches with its role's Curator profile, and role skills are never materialized into the project's discovery directories (*decided 2026-09-25*, replacing on-demand activation). Agents may find and install audited skills from the Curator registry unless the project restricts it. What an installed skill changes in the process waits for a person, unless a capability covers it: new instructions for a role, a new version of a tool the person allowed. Skills from unknown sources are refused by default. *decided*
- Roles state **needs** — requirements and preferences in one vocabulary, speed included — never models or efforts; a project may hard-restrict which models run on it; the operator's private files bind models and effort, and what widens admission there — metered billing, standing answers and skill sources outside the Curator registry by default, any other kind the operator chooses — takes effect only after the operator confirms it ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §10); for the whole team only the project relaxes a hard need permanently, and an operator may answer an `ask` in advance, open-endedly for their own runs, under a periodic review that the operator must answer for the answer to stay in force. *decided*
- **Guards read field values**, so review levels and thresholds are routes, and the fields they read are guarded like authority; **policy bounds the elements in progress**; and **check sets** — local, on the project's CI, or both — supply the evidence for Change Request work. *decided*
- A static **validator** (`curator playbook check`) checks a configuration before anything runs, split into portable checks and machine checks. *decided*
- Development machinery is a **delivery module** whose invariants bind to element state and hold whatever a playbook says. *decided*
- Until the main loop exists, `task-board tick`, called every minute by a user-level system timer, fires timers, schedules, expiries and tool invocations. *decided*
- **Dream mode.** A process can be rehearsed in a dream: every tool runs with `--dream` against a copy of the board and changes nothing in the real world, the agent knows it is dreaming, and nothing recorded in a dream counts for a real guard, approval or confirmation. A dream is also the node's nightly offline cycle: replay of distilled session logs, rehearsal, consolidation of memory, and insights into a quarantined dream memory that the system roles turn into proposals (§6.13). *proposed*
- Today's task-board is itself a playbook ([`default.playbook.json`](../examples/default.playbook.json)); PC1 makes the kernel read it with identical behaviour or documented differences, and the first demos run right after PC1. *decided (H6)*
- **PC1 runs an interim trust mode:** one operator, one machine. The kernel reads the committed default-branch version. The trust component is a guardrail stub, in which a confirmation or a capability grant is the operator's command outside any run, unsigned, and every such act carries the principal of the board's anchor operator. PC1 takes versions only on a single-host local board; every other board stays in legacy mode. The stub stops well-behaved agents and is documented as no boundary; real trust arrives with PC3 and CM2 ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §13). *the author's, with the split into trust.md (§17.2 row 31)*

## 1. Goals and non-goals

Goals:

1. Any process — software delivery, a restaurant's purchasing, formalizing a proof, a family calendar — is configuration, not kernel code.
2. The same file works for every teammate, whatever harnesses and models each one has.
3. A process can be read, validated and eventually drawn and edited by a visual constructor without losing anything.
4. Processes compose: the output of one can be the input of another, and a cadence (Scrum) can sit over a delivery process.
5. A configuration can never make an installed package execute something the project did not admit.

Non-goals of this revision: the orchestrator's agent loop; an always-on scheduler (the main loop, later; until then §6.9); the goal system itself (`board-goal-v2` stays in task-board; the configuration only names each orchestrator's goal type, §6.7); the storage format of board elements; the waggle amendment this design depends on (§17), which is written in the waggle specification; who counts as a person and how a person's acts are shown, which is [`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md).

Principles *(decided 2026-09-25)*:

1. **Agents may propose and speculate; only deterministic rules move authoritative state.** A model's judgement shapes what agents propose; guards, the kernel's comparisons and the trust component decide what changes the board. No verdict of a model is a permission (§6.3, §10).
2. **The board's journal is the state, and a model session is a cache.** A session that loses its context recovers from the board goal, the park note and the board; nothing that decides authority lives only in a session.
3. **Delegated and nested authority only narrows.** A delegation stays within the project's ceilings and the grantor's own rights (§4.2); a sub-orchestrator works inside its parent's pool, as today's nested spawns do (`goal_scope_mismatch`); a composed playbook's orchestrator writes only its leased subtree (§7.2).
4. **Write scopes are a security and a concurrency primitive.** A run writes only inside its scope, and two runs whose scopes overlap do not write at once; today's `run_write_boundary` (task-board `spawn.worktree_isolation.run_write_boundary`) and Story worktrees are its first form, and leases carry it across orchestrators (§7.2).
5. **Integration binds the exact tree that lands**, and landing is a compare-and-set on the default branch: evidence binds to the tree it checked (§6.12), and a landing fails closed when the branch moved (§6.10).
6. **Transport is not authority.** Reaching a node, or showing a network identity, grants nothing ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §1).
7. **Privileged operations happen outside agent shells.** Tools run in the kernel's executor, never in the triggering actor's process (§6.11); secrets never enter agent environments. An agent requests a privileged operation — a payment, a publication, a deploy, the use of a secret, a message to someone outside, a physical device, a deletion — and a trusted executor performs it after authorization ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §18).

## 2. Files and responsibilities

### 2.1 The project's files *(decided)*

| File | Answers | Read by | Carries |
| --- | --- | --- | --- |
| `Skillfile.json` | what is installed | Curator | skill names, sources, versions (tags, revisions), activation |
| `Playbook.json` | how the project works | the board (task-board today) and the validator | admitted playbooks, role aliases bound to skill names, tool bindings, people, event signer groups, entities, workflows, policy: review, work in progress, version control, signatures, delegation, data (model admission included) |
| `task-board.config.json` | where and with what the board runs | the board | board location, integration base branch, board repository, check sets (§6.12), team runtime names, run supervision. Once a version is in force, every key the kernel reads from this file is part of the resolved configuration: it is read at the proposed head and runs from the version in force, never from the working tree (§2.4) |

All three are committed. Process rules that sit in `task-board.config.json` today move to `Playbook.json`: the version-control policy (commit confirmation and time) *(decided 2026-09-24)*, and the run write boundary, `spawn.worktree_isolation.run_write_boundary` (`warn` or `refuse`), which becomes `policy.delivery.run_write_boundary` because it gates review and delivery (the author's resolution, §27, refuter v6 V5). The old keys are read for one release with a warning, as part of the resolved configuration (§2.4). The integration base branch and the board repository stay in `task-board.config.json`, are read from the version in force, and must name the board's recorded default branch (§2.4). Model admission moves to `Playbook.json` as `policy.data.models`, where a project may hard-restrict the models used on it (§9) *(decided 2026-09-24, P2)*: today's `spawn.ceilings` with their per-role replacements, the runtime set of `preferred_agentic_system` in either form, and the schedule windows that change that set by time of day. Model choices — bindings, workload classes, the candidate order and preferences — leave both committed files for the operator's `models.toml`. The mapping of each key is in §2.3; that the runtime set, the role ceilings and the windows stay project admission is the author's resolution (§23, S8). Run supervision (today `spawn.ceilings.<runtime>.supervision`) stays in `task-board.config.json` as `spawn.supervision`. Machine limits leave the committed files: today's runner limit `spawn.max_parallel` and suite cap `spawn.validation.max_concurrent_suites` become machine capacity in `runtimes.toml` (§2.3). The process's own limit on work in progress is `policy.wip`, which counts elements (§6.8).

**When the board and the code live in different repositories** — a shared product board over several code repositories, or a fork whose upstream owns the board — `Playbook.json` lives in the board repository, and its default branch is the one §2.4 reads. Each Story's check sets come from its code owner's `task-board.config.json` (§6.12), and templates resolve from the project scope and committed lock of the board checkout (§8.2).

`Playbook.json` never carries a version, a URL or a source; `Skillfile.json` never carries a role, a state or a transition. They meet only through skill names. When a role or tool names a skill that is not installed, the validator reports it with the fix: the `curator add` invocation for a named skill, or the Skillfile selector to add (§8). Installing (Skillfile) and using (Playbook) are two edits with two meanings, like a dependency in `go.mod` and its `import`; a helper, `curator playbook bind <alias> <skill>`, can make both.

Without Curator, role skills and templates copied into the harness skill directories by hand count as skills from unknown sources: they are refused unless the operator's confirmed setting admits them, and tools never run (§8.4).

### 2.2 Format *(decided)*

- JSON with a published JSON Schema: every file starts with `"$schema"` and `"schema_version": 1`. Editors get validation and completion; a constructor reads and writes the file without loss.
- Canonical JSON (the canonical form Curator uses for digests) defines the semantic digest of a resolved playbook. The `ui` blocks, `description` fields and the `x-` keys the kernel ignores are excluded from the digest.
- Comments are `description` fields on any element; a constructor shows them.
- Every element may carry a `ui` block (positions, colours, collapsed groups) for a visual constructor. The kernel ignores it.
- Keys starting with `x-` are extensions; the kernel ignores them. Where the schema gives an `x-` name a meaning — a custom capability, a custom `risk` marker, a custom signature category in `policy.signatures` (its `require_for` values and its `user_verified` keys) — the name is data: the kernel reads it, and it stays in every digest.
- Names of roles, entity types, workflows and states are kebab-case identifiers; field names are snake_case (`task_class`), matching today's element fields.

### 2.3 The operator's private files *(decided 2026-09-24)*

What belongs to one operator and one machine lives in the operator's private files under `~/.curator/`, never in a repository:

| File | Holds | Read by |
| --- | --- | --- |
| `models.toml` | role bindings to models and efforts, admitted models and billing classes, time windows, standing answers to `ask` | the selection resolver (§9) |
| `runtimes.toml` | this machine's runtime bindings (harness binaries, accounts, homes) and its capacity: parallel runs (today's `spawn.max_parallel`, default 20), concurrent local check sets (today's `spawn.validation.max_concurrent_suites`, default 2), rate-limit ladders | the launch plane; the kernel, for machine capacity |
| `engines.toml` | local inference engines and their models | curator-inference-manager |
| `network.toml` | network profiles | curator-network-profiles |
| Curator's own configuration (`~/.curator/config.json`) | Curator's settings, among them the skill sources outside the Curator registry that the operator admits (§8.6); no second `curator.toml` beside it *(decided 2026-09-27)* | Curator |

The four operator files are TOML because people edit them by hand, and none of these files is ever committed. Keeping them together amends LP D2, which placed runtime bindings in `$XDG_CONFIG_HOME/agents-management/runtimes.toml`, and the network-profiles specification, which placed its profiles in `network.json` under `$XDG_CONFIG_HOME`. The operator-file commands (below) edit them safely, and they are the only way an agent changes them.

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
"dev-cycle/reviewer" = { relax = ["not_same_publisher_as"], until = "open" }
"*/reviewer"         = { relax = ["reasoning"], projects = ["github.com/relux-works/skill-project-management"], until = "2026-12-31" }

[review]
answers = "P1M"

[confirmation]
billing = "user"
answers = "user"
skills  = "user"
```

- One `[bindings]` table; keys go from specific to general: `<playbook>/<role>`, then `*/<role>`, then `@<capability>`. The most specific matching key wins.
- `run` is one `<runtime>/<model>` or an ordered array of candidates; a pattern such as `codex/*` matches every registry row of that runtime. The resolver takes the first candidate that is admitted, available and satisfies the role's needs; a candidate that is unavailable moves the choice to the next one. The selection records which candidate won and why each earlier one did not.
- `effort` is one value or a range (`"medium..high"`). A project floor (`policy.effort_floor`, §6.8) narrows it from below per model row (§9); the selection records the effort it used.
- `[allow]` narrows which models may run at all, and `billing` lists the billing classes this operator pays for. Only this file admits `metered` billing, and only once the operator has confirmed it ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §10): a project may narrow billing classes, never widen them (§9). `[windows]` limits when a model may run.
- `[answers]` holds standing answers to `ask` (§9). Keys follow `[bindings]`; `relax` names the requirement terms the answer relaxes; `projects` limits it to the named projects. `until` bounds how long the answer itself applies: an end date, or `open` written explicitly. Each application creates an exception that always takes the role's approval scope and expiry (`needs.approval`, or for a role without it the exception defaults of §9), whatever `until` says. An open-ended answer is allowed *(decided 2026-09-24)*: it relaxes the requirement at every future `ask` of its role for this operator's runs, each application appears on the board as an exception, and a project can forbid standing answers (`policy.enable.standing_answers`, §9). A periodic review asks the operator to re-confirm every answer whose end lies beyond the next review, and an answer not re-confirmed within one period after its review stops applying until it is re-confirmed (§9). A standing answer applies to this operator's own runs only, never widens `[allow]`, and takes effect only once confirmed ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §10).
- `[review]` sets the period of that review: `answers`, an ISO 8601 duration, one month by default (§9); the operator decided "once a month", and the configurable period is the author's. Lengthening it is a widening entry of kind `answers`.
- `[confirmation]` says what each kind of widening needs before it takes effect ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §10).
- A file that exists but cannot be read or parsed refuses every selection: a read failure is never headroom (LP D9).
- **Without `models.toml`** the resolver chooses automatically, never chooses a metered runtime, and explains its choice (§9).
- Today's `task-board.config.json` model keys move out of it without loss *(P2 decided 2026-09-24; amends LP D1; the mapping of each key is the author's, §23, S8)*:
  - `spawn.ceilings` become the project's `policy.data.models` in `Playbook.json`: each runtime's admitted models and effort bounds turn into allow patterns with effort ranges, and each role's replacement admission (`roles.<role>`) into `policy.data.models.roles.<role>` (§9);
  - the runtime set of `preferred_agentic_system`, `exclusive` or `mixed`, becomes allow patterns `<runtime>/*` there; the `mixed` order becomes the operator's candidate order in `[bindings]`;
  - `spawn.schedule.windows`, which replace the runtime set for a time of day, become allow entries with `windows` (§9), still project admission; the operator's `[windows]` narrow them further;
  - per-role defaults and `workload_classes` become the operator's `[bindings]`;
  - `resolution_model` and `adjustment_confirmation` become the operator's candidate order: with `none`, a request outside admission resolves to the first admitted candidate of the operator's `run` list, and the selection records the adjustment; with `required`, it is refused, and the refusal names the admitted candidates;
  - Codex `fast_mode`, which spends the operator's quota on the priority tier, becomes an option of the operator's `[bindings]` entry (`fast_mode = true`), because only the operator spends the operator's quota (§9);
  - `supervision` stays in `task-board.config.json` as `spawn.supervision.<runtime>`, with its role overrides: it bounds how long the board supervises a run, not which model runs.

  Until migrated, the old keys are read as before with a warning — the ceilings, the runtime set and the windows as project admission, the rest as a legacy operator layer — and `task-board config migrate` previews every destination and writes them; it also absorbs today's `task-board migrate-project-config`. In a project with a version in force the old keys are part of the resolved configuration, read at the proposed head (§2.4), and stay in force until a confirmed version replaces them with `policy.data.models`, so the migration never opens a window without admission; on a board in legacy mode they are read as today (§14 point 13).
- Machine limits never sit in a committed file, because teammates run different machines: parallel runs, concurrent check sets and rate-limit ladders live in `runtimes.toml`, with today's defaults of 20 parallel runs and 2 concurrent check sets. For one release, a `spawn.max_parallel` or `spawn.validation.max_concurrent_suites` committed in today's `task-board.config.json` is read as the machine's capacity where `runtimes.toml` does not set it, with a warning. The process's own limits on work in progress are `policy.wip` (§6.8).
- Every choice, from the file or not, must satisfy the role's needs (§9), else `on_unmet` applies.
- Tool bindings, people and event signers are not in these files: they are project facts and live in `Playbook.json` (§4.2, §8.5).

**Widening entries take effect only after the operator confirms them** *(decided 2026-09-24, P3)*. These files sit in the operator's home, where any process running as the operator's OS user can edit them: an agent stuck on an `ask` could answer it for itself, or admit metered billing, and reading the committed process from the default branch (§2.4) does not reach them. So an entry that widens admission — a model or billing class added to `[allow]`, a standing answer added or extended, a window widened, a runtime binding added, an engine or a network profile added, a skill source outside the Curator registry admitted, a confirmation level lowered — takes effect only once the operator confirms it; a narrowing edit applies at once and becomes the new baseline. The effective version, the kinds of widening, the `[confirmation]` levels and their defaults, the confirmation record and its verification are specified in [`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §10. Until the operator-file commands ship, the resolver honours no widening entry whose level is not `none`, and reports each one it ignores.

**How a person confirms.** Every confirmation this specification asks of a person — a change to the process (§2.4), a delegation grant (§4.2), a widening entry of the operator's files (above), a skill from an unknown source (§8.6) — is specified in [`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md): who counts as a person (§3), the confirmation and its record (§4), the operator companion app (§8) and, until it exists, confirmation of a headless node from the operator's workstation (§9). The floor never depends on this file or the project: only a signature by a key that requires a person's presence confirms. In PC1 the trust component is a guardrail stub (§0, [`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §13).

**Commands** *(decided 2026-09-24; the names follow the command-line shape adopted on 2026-09-27)*. One command family through the `curator` umbrella edits these files. Each family is a Curator provider named after its noun, because `curator <noun>` dispatches to the executable `curator-<noun>`; `sources` is a Curator builtin:

| Command | Provider | File | Verbs |
| --- | --- | --- | --- |
| `curator models` | `curator-models`, the selection resolver | `models.toml` | `list`, `show`, `set`, `unset`, `allow`, `answer`, `review`, `explain`, `check`, `confirm`, `init`, `edit` |
| `curator runtimes` | `curator-runtimes`, the launch plane | `runtimes.toml` | `list`, `show`, `add`, `remove`, `check [--probe]`, `confirm`, `limits`, `edit` |
| `curator engines` | `curator-engines`, the binary of curator-inference-manager | `engines.toml` | `list`, `show`, `add`, `remove`, `check`, `bench`, `confirm`; and `status`, `ensure`, `release`, `stop` for the engines themselves |
| `curator network` | `curator-network`, from curator-network-profiles | `network.toml` | `list`, `show`, `add`, `remove`, `check [--probe]`, `confirm`, `exec <name> -- <command>` |
| `curator sources` | Curator itself | Curator's configuration | `list`, `show`, `add`, `remove`, `confirm`, `submit` (a skill for audit, §8.6); it replaces `curator audit --allow` |

- Every command keeps comments and order, validates before writing, writes atomically with a backup and mode 0600, never stores a secret value (only an environment-variable name or a command that yields the secret), and offers `--json`. From inside an agent session it refuses any change that widens admission.
- `curator models explain` prints, for the current project, each role's model and effort and the layer that decided each; it is the same resolver as task-board's spawn preflight. `curator engines bench` records the local speed facts of §9. `curator models init` is the first start (§17.3). `answer` records a standing answer and `review` starts their periodic review (§9). `curator runtimes limits` shows and clears the machine's provider-limit state, which leaves task-board as `task-board limits`.
- `edit` opens a file in the person's editor, then validates the result and detects widenings before it replaces the file, so a hand edit gets the same checks as a command. Adding an engine or a network profile is a widening, because an engine entry names an executable and its arguments and a network profile reroutes traffic; so `engines` and `network` take `confirm` like the other files.
- Each file keeps one owner: `models.toml` the selection resolver, `runtimes.toml` the launch plane, `engines.toml` curator-inference-manager, `network.toml` curator-network-profiles, and Curator's own configuration Curator. Curator provides the umbrella and the directory.
- A change to the process (§2.4) is confirmed with `task-board trust confirm`, which shows the authority report (§10) before it asks for the confirmation; `--node <node>` runs the workstation path of [`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §9. Every mutating `task-board trust` verb refuses to run in an agent session. `task-board config status` shows, before anyone confirms, the version in force, the proposed version, their differences and the grants that cover them. `task-board playbook confirm` stays an alias of `task-board trust confirm` for one release. In an apiary the person types `keeper confirm` (curator-keeper §7.2).

### 2.4 Which version is in force *(decided 2026-09-24, H1; revised in v4.0 and v4.1)*

The orchestrator's session works in the project checkout with ordinary file tools. If the kernel read `Playbook.json` from the working tree, one unreviewed edit could drop a signature requirement, a hard need or a human owner (refuter v1, N1). waggle meets the same problem for its roster by reading it at a pinned revision (waggle §3). So:

- **The proposed version** is the resolved configuration (below) at the head of the default branch of the repository that owns the board, whatever Story is in scope. The kernel never reads the working tree, and no branch or fork runs rules of its own. The board records its default branch when it is created; changing that record is a change to the process.
- **The version in force** is the last proposed version that took effect, either because capabilities in force covered every difference or on a person's confirmation (below). The kernel stores on the board the resolved configuration of every version that takes effect, and every machine derives the version in force from history for itself (below), so every machine runs the same version ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §4, §12).
  - **What runs.** The kernel runs `narrowed(version in force)`: the version in force with every person's narrowing act applied on top of it ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §5), so a tool that a person has disabled does not run while the matching commit waits.
  - **The pinned revision** that the rest of this specification refers to is the revision of the version in force, never the proposed head. At run time a roster pattern, a quorum, a role alias, a ceiling or an admitted playbook is always read from the version in force. The validator, which reads committed files alone, checks a configuration against its own roster (§10).
- **A commit is enough** *(decided 2026-09-24)*. A change is proposed by committing it to the default branch. Git hosting is optional and a pull request is never required: a project that reviews changes through pull requests lands them as commits like any other, and a project with no hosting commits locally. `curator playbook check`, previews and a visual constructor read the working tree; the kernel never does.
- **The resolved configuration** is everything that decides what runs and who may act:
  - the project's `Playbook.json`;
  - the content of every template it resolves through `extends` and `uses`, with each template's package identity, resolved from the project scope and its committed lock, or from the kernel's built-in templates (§8.2);
  - the `data/role.json` contribution — capability, default needs, definition-of-done items, needed skills — of every role skill the resolved roles bind, at the identity the project's lock records (§8.1);
  - the package identity of every skill whose commands the resolved `tools` bind, as the project's lock records it (§6.11, §8.5);
  - every key of `task-board.config.json` that the kernel reads:
    - the check sets the resolved workflows name;
    - the integration base branch and the board repository, which must name the board's recorded default branch;
    - run supervision and team runtime names.

    They are read at the proposed head, or, for check sets whose code lives in another repository, from that repository's default branch (§6.12). For their one release, the legacy `version_control`, `run_write_boundary` and model keys are read from the same file (§2.1, §2.3, §14);
  - the waggle roster, its revocation list and its enrollment records at the proposed head (waggle §3; [`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §3);
  - the board's recorded default branch and the project's confirmers.

  Its **configuration digest** is its semantic digest, leaving out `description` fields, `ui` blocks and the `x-` keys the kernel ignores (§2.2). A difference in those is no difference at all.
- **Which changes wait for a person: capabilities** *(decided 2026-09-25, §17.2 row 35; the kinds below are the author's rendering of the operator's table)*. A process asks a person once for a **capability**, one kind of change, the way a phone app asks for a permission. The person answers *always*, *once* (this pending change only) or *no*, and the answer is a grant recorded on the board ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §16). The kernel compares the proposed version with the version in force section by section. A difference takes effect on commit only when a capability grant in force covers it, and each kind covers exactly the differences listed here:

  | Kind | Covers exactly | Default |
  | --- | --- | --- |
  | `preferences` | a role's `prefer` (§9) | granted |
  | `estimates` | a type's estimate dimensions and scales (§5.3), where no slot contract requires the dimension changed | granted |
  | `helpers` | adding or removing a state's helpers whose roles are agent roles, and raising a helper's `min` (§6.1). A helper whose role is a human role, and a lowered `min`, are not covered | granted |
  | `wip` | raising any limit of `policy.wip`, and adding or lowering a limit that counts only agent-owned states (`per_state`), agent roles (`per_role`) or the board (§6.8). A limit that counts a human-, tool- or process-owned or verification state, or a human role, is not covered | granted |
  | `fields` | adding a declared field, with its description and default, that is not `required` and that no `field` or `fields.present` guard, delegation ceiling, condition or budget, `record_source`, `clear_on_exit`, `privacy` rule or slot contract reads (§5.2) | granted |
  | `schedules` | adding, changing or removing `schedules` entries and naming them in `policy.enable.schedules` (§6.6), within the grant's limits: a minimum interval between firings and a maximum number of elements per firing. A schedule creates only types the version in force declares, and sets only fields that no guard, ceiling, condition or budget reads | asked, per process |
  | `tools:<name>` | listing the tool `<name>` in `policy.enable.tools` and binding it in `tools` to a provider skill at an audited identity (§8.5), and later audited identities of that provider (§6.11). The states the tool owns are workflow structure and are not covered | asked, per tool |
  | `skills` | a new audited identity of a bound role skill whose `data/role.json` contribution is unchanged, or of a template-carrying skill whose template content is unchanged, so that only instructions change (§8.1, §8.2) | granted unless `policy.enable.skills` is `none` |

  - **Never covered.** Every other difference waits for a person's confirmation *(decided 2026-09-25)*:
    - **workflow structure**: entities and their workflows, states, owners and transitions, with their guards, actions, `requires`, `limit`, `locked`, `record_source`, `clear_on_exit`, `resolution` and `on`, and `policy.goals`, `policy.capacity`, `policy.review` and `policy.privacy`;
    - **everything that carries authority**: people, services, confirmers, the roster and enrollments; thresholds and money (`field` conditions, `approvals` guards, `risk` markers); human-, tool-, process-owned and verification states; delegations; `policy.data` and model admission; `policy.signatures`; needs other than `prefer`, `on_unmet` and the effort floor; check sets and delivery policy; `extends`, `uses`, slots and `bind`; `events` and their signers; every other key of `policy.enable`; the board's recorded default branch;
    - a difference the comparison cannot classify.
  - **The comparison** is deterministic kernel code over sections, so it never reasons about paths through a workflow. A change-analyst role may explain a difference to the person, but never decides it *(decided 2026-09-25)*.
  - **Parameters.** A change of a parameter's value (§7.3) is a change to every place that references it. A work-in-progress parameter is covered by `wip` exactly as the limit itself would be; a loop limit or a timer is workflow structure, so a new value waits for a touch, batched like any other change.
  - **Grants.** A grant is a person's act; no delegation covers it. A person may revoke a grant at any time, which takes effect at once. Grants that are not sealed come up at the monthly review, and one re-confirmed at two consecutive reviews is sealed ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §16).
- **The version in force comes from history** (the author's resolution, §27, refuter v6 V3).
  - **The walk.** From the last confirmation record, the kernel walks the first-parent commits of the default branch, or the apply numbers of a project without git. Each commit whose differences from the running version are all covered by the grants in force at that commit takes effect. The first commit that is not covered is pending, and so is every later commit, covered or not: the whole version waits (below).
  - **Nothing is taken on trust.** Every machine re-derives each such link from the stored configurations and the recorded grants, and never accepts a stored claim that a version took effect.
  - **Confirmations.** A confirmation's replaced digest is the version in force derived this way ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §4).
- **Only a valid version takes effect.** A proposed version that fails a portable check (§10) never takes effect, whether capabilities cover it or a person confirms it. The kernel keeps the version in force and reports, and `task-board trust confirm` refuses an invalid head.
- **The whole version waits.** While any difference waits, the kernel runs the version in force in full — every section — and every later commit waits with the head. It reports the pending change on the board and notifies the confirmers. Approvals and grants bind the parts of the version in force they depend on, never the proposed head (§4.2, §6.4).
- **Batched confirmation.** One confirmation covers every change committed since the version in force: the person confirms the head, and the authority report lists everything in between. A process designer or librarian that proposes several changes therefore asks once.
- **The trust interface.** [`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §2–§5 and §16 give the kernel four answers:
  - `confirmed(version)`: whether the trust component reports a version confirmed;
  - `granted(process, kind[, name])`: whether a capability grant is in force;
  - `narrowed(version in force)`: what runs;
  - `verify(act)`: whether a signed act holds.

  They also say who may confirm: the `people` group that `policy.confirmers` names in the version in force, or the board's anchor operator. And they fix, with which keys, what a record binds and how every machine verifies it. A person's narrowing acts take effect at once, and there are exactly three: revoking a key, revoking a grant (a delegation or a capability grant) and disabling a tool *(decided 2026-09-25)*. Every machine runs the same version in force, because every machine verifies the same records on the board and derives the same capability links ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §4, principle e).
- **yolo mode** *(decided 2026-09-25)*. A person may switch on, with one touch and optionally for a duration such as `24h`, a mode in which every change takes effect on commit, for one process or for every process of a node.
  - **Where it is refused.** On a shared board — one with more than one confirmer, one used from more than one machine, or any board from L1 — and on a process where a delegation covers money.
  - **It is loud.** A banner shows it in every status view and in every authority report.
  - **It stays auditable.** Each version is still recorded as a link in the chain, and the emergency stop and the narrowing acts always work.
  - **Switching it off** is revoking a grant ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §17).
- **The first version and legacy mode.** A board without a version in force — every board of today, and a project whose first `Playbook.json` is not yet confirmed — runs the built-in default playbook in legacy mode.
  - **The built-in default** grants nothing beyond today's board and needs no confirmation: it reads `task-board.config.json`, the committed `.roles` overlay and the legacy keys as today (§14).
  - **Nothing of the project's own takes effect** until the board's anchor operator ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §3) confirms the first version: none of its types, tools, schedules, automatic spawns, delegations, relaxations, standing answers or signed events. A project that an agent creates, the process designer included, stays unconfirmed until then. Installing audited skills stays available (§8.6).
  - **The default capabilities** — `preferences`, `estimates`, `helpers`, `wip`, `fields`, and `skills` where `policy.enable.skills` allows it — are granted with the first confirmed version; the confirmer may decline each.
  - **In PC1** a version takes effect only on a single-host local board, and every unsigned act outside a run carries the anchor operator's principal ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §13). Every other board stays in legacy mode.
- **Without git.** A project that is not a git repository has no default branch. `task-board config propose` proposes a version by recording the file and its resolved configuration on the board under an apply sequence number, and the same rules decide when it takes effect. This replaces decision 2a's "reads the file as it is and warns", which predated the confirmation rule; the replacement is the author's fill (§17.2 row 28).

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
  "params": { },
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

`name` is required for a published template and optional in a project file. `extends` and `uses` are covered in §7, `slots` and `bind` in §7.2, `params` in §7.3, `tools` in §8.5, `people` and `services` in §4.2, `events` and `schedules` in §6.9. Keys that only a project may set — `people`, `services`, `policy.enable`, `policy.effort_floor`, `policy.signatures`, `policy.delegation`, `policy.data` and `policy.confirmers` — are ignored with a warning when they appear in a template. A template may bind a tool only under the rule of §8.5. Capability grants and yolo mode are records a person makes on the board, never keys of `Playbook.json` (§2.4).

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

**Actor classes.** Every transition records its actor: `run` (a spawned run with its role), `session` (a hosted orchestrator session), `human` (a CLI action outside any run and session, or a waggle signature), `kernel` (automatic transitions, tool invocation, timers) or `event` (a signed external event, §6.9). An agent run or session never fires a transition out of a human-owned state, never records a human approval and never posts a tool or external event. The one exception is delegation (below): an orchestrator session of the person who granted a delegation, or a session the grant names, may act for that person within it, and the act is recorded as the person's delegated act.

`human` is defined by absence today, so an untracked `curator run` session, a bare harness binary and a run that clears `TASK_BOARD_RUN_ID` all count as human. *Decided (2026-09-24, H1):* without a signature policy, an operator's CLI act outside any run counts as human, and this specification calls that what it is: a guardrail against well-behaved agents, not a boundary against a hostile one. The session host narrows the gap by marking the sessions it hosts, which then act as `session`. The boundary is a signature that shows a person acted (**Signatures**, below; [`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §6), checked against the version in force (§2.4). The validator warns when a workflow has human-owned states and the project sets no `policy.signatures` (§10). Today the only actor signal the kernel has is the `TASK_BOARD_RUN_ID` convention, which the producer guard itself calls "not an operator authorization boundary" (task-board `tools/board-cli/cmd/producer_handoff_guard.go:12-15`).

**People.** `people` maps groups to waggle operator principals, and only to them:

```json
"people": { "cafe-managers": { "principals": ["op:alice@cafe", "op:bob@cafe"] } }
```

- Orchestrator and worker principals (`orch:`, `worker:`) are agents and cannot be people; the schema refuses them.
- At run time a pattern (`op:*@acme`) is resolved against the waggle roster of the version in force, and a quorum (§6.4) counts distinct principals present in that roster. The validator checks a configuration's `people` against the roster of the same configuration, which is a committed file, so presence in it is a portable check (§10).

**Services.** `services` maps the signer groups that event declarations name to the principals of the bridges that may sign those events (§6.9). The service principal kind (`svc:` in the examples) comes with the waggle amendment this design depends on (§17).

**Signatures.** `policy.signatures` names which human acts need a waggle approver signature, at what key assurance, and whether the signature must show user verification. Its categories are defined by construct:

| Category | Covers |
| --- | --- |
| `approvals` | recording an approval that an `approvals` guard counts (§6.4) |
| `relaxations` | approving an `ask` exception (§9) |
| `spending` | firing a transition marked `"risk": "spending"` |
| `halts` | a waggle `cmd.halt` or `cmd.cancel` to a run of this process, and firing a transition marked `"risk": "halt"` |
| `overrides` | a human leaving a tool- or process-owned state (rule 2), or authorizing a landing whose hosted checks could not run (§6.12) |
| `human-transitions` | every transition a human actor fires out of a human-owned state |
| `delegations` | granting or revoking a delegation (below); covered whenever the project sets a signature policy, listed or not |
| `installs` | an installation of a skill into the project by a person or a delegate (§8.6) |

- `min_assurance` (`software`, `hardware-bound`, `attested`) and `user_verified` (`required`, `if-supported`, for every category at once or per category) set how strong the required signatures are. *Decided (2026-09-24, H7):* whenever a project sets a signature policy, the minimum assurance defaults to `hardware-bound`, and `user_verified` defaults to `required` for approvals, relaxations, halts and delegations and to `if-supported` for spending, overrides, human transitions and installs; a project may lower `required` to `if-supported`. What satisfies each value — a FIDO2 signature with the user-verified flag, an attested companion key, a key whose presence was proven at enrollment — is [`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §6; an attested companion key meets `required` for every board act *(decided 2026-09-25)*. `policy.signatures` never governs a confirmation of a change to the process, whose floor no project lowers ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §4).
- Approvals are signed only through the operator-facing command that renders the payload before signing ("what you see is what you sign", waggle §4.2); an agent never holds a key.
- waggle's amendment A1 gives board approvals, signed transitions, grants, delegated acts and signed events their forms (§17); it is written into the waggle specification now and implemented with milestone CM2. Until then signed human acts other than confirmations, capability grants and narrowing acts ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §4, §5, §16) have no form, and the cafe demo waits (§15). A category in `require_for` that has no signable form yet makes the states it covers unleavable, so the validator warns about it and names the milestone that brings the form (§10); the warning also covers `delegations`, which a signature policy includes whether it is listed or not, because before CM2 it makes every grant impossible. The PC1 demo projects set no signature policy.

**Delegation** *(decided 2026-09-24)*. Autonomy comes from the playbook's design, and the authority rules do not take it away. Agents move the steps the playbook gives them, a person owns only the steps the playbook gives a person, and a signature is needed only for the acts the project marks. Where a playbook keeps an act with a person, that person may delegate it to their own orchestrator sessions and keep only what the delegation leaves out, such as spending above a limit or a decision that is critical to the work.

- **What may be delegated** is project policy: `policy.delegation`, project-only; no capability covers it, so a change to it takes effect once a person confirms it (§2.4). Without it nothing may be delegated. No delegation ever covers a confirmation: a change to the process (§2.4), a capability grant (§2.4), a widening entry of the operator's files (§2.3), a skill from an unknown source (§8.6) and a grant itself are always confirmed by a person, so no delegate can widen the authority it acts under — from PC3; in PC1, whose trust component is a guardrail stub, this is a guardrail (§0).

  ```json
  "delegation": {
    "allow": [
      { "acts": ["approvals"], "role": "manager", "states": ["purchase.approval"], "to": ["cafe-orchestrator"],
        "ceiling": [ { "guard": "field", "name": "amount", "lte": 1000 } ], "self_approval": true },
      { "acts": ["transitions"], "states": ["purchase.approval"], "to": ["cafe-orchestrator"],
        "ceiling": [ { "guard": "field", "name": "amount", "lte": 1000 } ] }
    ],
    "never": [ { "acts": ["transitions"], "states": ["purchase.escalated"] } ]
  }
  ```

  An `allow` rule names the acts, the agent roles they may go to (`to`: roles with capability `orchestrator`) and hard ceilings as `field` conditions that every delegated act must meet. The acts are `approvals` of a named human role, optionally only in the listed states, `transitions` out of named human-owned states, `relaxations`, meaning the approval of `ask` exceptions, and `installs`, meaning the installation of audited skills from the Curator registry into the project (§8.6). States are named `<workflow>.<state>`, so a rule never reaches a same-named state of another workflow, and every type a rule reaches must declare the fields its ceilings read (§10). A `never` rule names acts that no delegation may cover, for the roles it lists or for any role, and `never` wins over `allow`. `self_approval: true` on an `allow` rule that covers approvals lets a delegate approve an element that it or its runs filled in (below); without it, it may not. A rule that covers `relaxations` may limit them to the asking roles and the requirement terms it names, `relax: { "roles": [...], "terms": [...] }`; without it, it covers every role and term. `installs` matters where the project restricts `policy.enable.skills` (§8.6). A rule that covers `installs` covers nothing else, so it carries neither ceilings nor `self_approval`: an installation has no element whose fields a ceiling could read, and the schema refuses a rule that mixes installs with other acts.
- **A person grants a delegation.** The kernel record `delegation`, available in every playbook like `exception` (§9), is created through the operator-facing command with a person's confirmation ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §7), and signed whenever the project sets a signature policy (category `delegations`). A person who asks an orchestrator in plain words to "let the cafe approve orders up to 500 EUR" gets the prepared grant to confirm; the orchestrator cannot record it without that confirmation (from PC3; in PC1 a guardrail). It records:
  - the grantor principal and the delegate role;
  - the sessions that may act under it: by default only the grantor's own orchestrator sessions of the delegate role, and `sessions` may name other session principals explicitly, never a pattern;
  - the acts; for relaxations, optionally the asking roles and the requirement terms it may relax (`relax`), within the rule's own `relax`; for installs, optionally the skills it covers;
  - `field` conditions, the policy's ceilings or stricter ones, on the fields those ceilings read and no others, so that a grant never makes another field authority-bearing (§5.2);
  - a subtree scope: an element the grant names, or the project;
  - an expiry;
  - optional budgets per period: a sum of a field that the rule's ceilings read, or a count of acts, over a duration, so a budget never makes another field authority-bearing (§5.2).

  ```json
  { "grantor": "op:alice@cafe", "to": "cafe-orchestrator", "acts": ["approvals"], "role": "manager",
    "where": [ { "guard": "field", "name": "amount", "lte": 500 } ],
    "budget": [ { "sum": "amount", "max": 2000, "per": "P1W" } ],
    "scope": "project", "expires": "P90D" }
  ```

  Here a cafe manager lets their own orchestrator sessions approve orders of up to 500 EUR each and 2,000 EUR a week, within the project's ceiling of 1,000 EUR.
- **Who may grant and who may revoke.** A person may grant only acts their own principal could perform: approvals of a role whose group holds that principal, transitions out of states whose owning group holds it, relaxations for the operator's own runs, and installs when the person is a principal of the project's `people`. The grantor, or any principal of the project's `people`, may revoke, because revoking only narrows authority. The rules hold for the unsigned grants of PC1 as for signed ones.
- **Whose sessions act.** By default a grant counts acts only from the grantor's own orchestrator sessions of the delegate role; a grant may name other session principals explicitly, never a pattern, and a session of another operator never acts under the grant unless the grant names it. How the kernel knows who started a session and which role it runs — the session host's marks in PC1, certificates from CM2 — is [`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §7. The session host extracted as it is today sets delivery markers only; until it marks who started a session and its role alias, the kernel records grants but counts no delegated act (§15).
- **What a grant binds.** A grant binds the parts of the version in force it depends on: the resolved `policy.delegation`, and the roles, states and fields the grant names. A version that takes effect and changes any of them ends the grant; one that touches none of them keeps it, and a pending change ends nothing until it takes effect. A signed grant is verified once, when the board records it ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §7); the grant's own `expires` bounds its use.
- **A delegated act** is recorded as the delegate's act on behalf of its grantor under that delegation. A guard counts it as the grantor's act only inside the delegation's acts, scope, conditions, budgets and expiry and inside the policy's ceilings at the pinned revision, checked when the act is recorded and again when a guard counts it. Freshness (§6.3) and the binding of the element's declared fields (§6.4) apply to it as to the grantor's own act.
- **Quorum and self-approval.** Each delegated act counts as at most one principal toward a quorum, its grantor, and one delegate session counts once whatever grants it acts under: a quorum above one needs distinct principals, acting directly or through distinct delegate sessions of distinct grantors. A delegated approval does not count when the delegate session, or a run it spawned, wrote a declared field of the element since the element was created, unless the `allow` rule that admits it sets `self_approval` — no self-approval, as there is no self-review (§6.10). A project that lets its orchestrator both fill in and approve small orders says so in its committed policy. Delivery invariant 2 and independence (§9) read the delegate's provenance, never the grantor's.
- **Budgets.** The board's kernel alone owns and counts a delegation's budgets. A budget sums a field its rule's ceilings read, over the acts recorded under that grant within the period, a rolling window that ends at the act, using the value each act bound. A grant covering approvals counts its approvals, one covering only transitions counts its transitions, and a relaxation counts as one act. An approval voided before any transition used it gives its amount back. Checking the budget and recording the act are one compare-and-set in the lease store (§6.9). A board shared across machines needs waggle's lease store for that (CM2): before CM2, a grant with a budget is refused on a board that is not single-host, which the project documents.
- **Everything else waits for the person.** An act outside every delegation is not taken: over a ceiling or a budget, under `never`, after expiry or revocation, or from a session the grant does not cover. The kernel notifies the grantor — through a board notice to the grantor's hosted session or the board's inbox until the main loop exists, through the main loop afterwards — and the board shows the act as pending.
- **Revocation and expiry** end a delegation at once and in the same way. A person's revocation is a narrowing act that takes effect without the confirmers ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §5). Delegated approvals under it that no transition has used yet stop counting; exceptions that a relaxation under it created end with it, and a run already working under one finishes; acts already consumed stay valid; nothing new is taken under it. Across machines a revocation takes effect where the lease store is: from CM2 through waggle's lease store, and before CM2 when the board syncs, which the project documents.
- **Signed delegated acts.** Where the project requires signatures for an act's category, the delegate signs with its orchestrator session key, and the board verifies the chain from the grantor's key to the act ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §7; waggle A1 §19.8). `user_verified` applies to the grant, which is the person's act; the delegated acts carry the delegate's signature within it.
- **Without a signature policy** a delegation is an unsigned record, but not an unconfirmed one: the grant needs a person's confirmation ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §7), which, from PC3, an agent cannot give; in PC1 this is a guardrail (§0). What stays a guardrail before signatures is the attribution of acts to the grantor's sessions, which rests on the session host's marks.

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

- **Defaults are written at creation.** A field's `default` is written when the element is created, by every creation path: `create`, schedules, hand-off and migration. For `review`, `policy.review.default` wins over the field's own default. Records created before a field existed are read through the kernel's normalization: an empty `review` reads as `required`, as today (`pkg/board/types.go:127-131`).
- **Fields a guard reads are authority-bearing.** A field that a `field` guard of the element's workflow reads decides routes, and a field that the ceilings, conditions or budgets of `policy.delegation` or of a granted delegation read decides what a delegate may do. No run writes a field that a `field` guard reads, on any element, its own or another; the orchestrator, a person and the kernel may. `review` in particular is set by the orchestrator or a person, never by a producer, whose work it could exempt. One exception keeps drafting possible: the run that works an element's initial state may write, while the element is in that state, a field that only delegation ceilings, conditions or budgets read and no `field` guard does; once the element leaves its initial state, no run writes it. The value it wrote then binds approvals like any other (§6.4), and a delegate approves what its own runs drafted only under `self_approval` (§4.2). A field the workflow only checks for presence (`fields.present`) stays writable by runs. When a version that takes effect makes a field authority-bearing, the values runs wrote earlier stay, and the authority report names the field (§10).
- **Approvals bind every declared field.** An approval, direct or delegated, binds the digest of the fields the element's type declared when it was given, and any change to one of them voids the element's unused approvals (§6.4). A field declared later is not in that digest, so adding one voids nothing. While a delegated approval is unused, the fields its delegation's ceilings, conditions and budgets read are frozen: a write to them is refused until a transition uses the approval or it is voided. A tool acts on the values recorded when its state was entered (§6.11).

### 5.3 Estimates *(decided)*

Estimates have named dimensions, so complexity and duration are separate:

```json
"estimates": [ { "name": "complexity", "scale": "fibonacci" }, { "name": "duration", "scale": "hours" } ]
```

Scales: `fibonacci`, `points`, `tshirt`, `hours`, `days`. Capacity rules (§6.8) sum one dimension. Estimate dimensions are unrelated to reasoning effort, which never appears in a playbook's needs (§9).

### 5.4 Privacy *(decided 2026-09-24, H8)*

`privacy` on a type (`inherit`, `private`, `team`, `public`) and `policy.privacy` (`visibility`, `redact_fields`) express per-process privacy, one of the generality vectors (§11): a conforming kernel shows each view only what its visibility allows and redacts the listed fields. Board elements are committed files, so redaction in views does not remove data from history. Until a separate store or encryption exists, the vector tests views only, and the specification says so wherever privacy appears.

### 5.5 Files, inputs and staleness *(proposed, 2026-09-29, PC3)*

Found by testing the model against published AI video pipelines (wiki research on the Opus 5.5 video wave; `examples/music-video/`). Their outputs are groups of files, their approvals are about exact content, and their work goes stale when something upstream changes: a new version of a character invalidates every shot drawn with the old one. Three of four studies found this independently.

- **File fields.** A field of type `file` holds a reference to content in the element's artifact store:
  - its path, the digest of its content, its size and its media type;
  - a `list` of `file` holds several.

  A file may be a **part set**: indexed parts (frames, chunks) with one digest per part and a declared count. A set is complete only when every part is present. A tool that reads an incomplete set fails with `incomplete` rather than using what exists, so an encode never stops silently at the first missing frame.
- **Approvals bind content.** The digest an approval binds (§5.2, §6.4) includes the content digest of every file field. Approving a storyboard approves exactly that shots file, and a changed file voids the approval like any other field.
- **Provenance and licence.** A file field records who or what wrote it (a tool invocation, a run, a person, an external event), and may carry `licence` and `attribution`. The guard `licence.cleared` passes when every file the transition names has a licence the project admits; a `publish` transition requires it (§6.15).
- **Inputs.** A type may declare `inputs`: fields of other elements it is made from, named through a `ref` field or the parent. Examples: `plate.pick` for a clip, `lead.canon` for a plate. When the element's work is accepted, the kernel pins the digest of every input.
  - **`inputs.fresh`** is a guard that passes while every pinned input still has the pinned digest.
  - **`input.superseded`** is an event the kernel emits on every element whose pinned input changed. A workflow routes it like any event, for example from `picking` back to `prompting`.
  - **Reopening.** A type may declare `reopen_on_superseded: <state>`, and the kernel then reopens a terminal element into that state. Reopenings count against a `limit` like any loop, and an approval the element held is voided.
  - **Impact before a swap.** Before a person approves a change to an element that others pin, the kernel reports the elements that would go stale and the sum of their declared cost fields and estimates. Replacing a character then shows what redoing its shots will cost.
- **Superseding** is a kernel link, not a text field. A new element that `supersedes` an accepted one inherits its place as an input. The dependents of the old one receive `input.superseded` naming the new one, and the old one becomes read-only.
- **Reuse by content.** A tool binding may declare `reuse: "inputs"`. An invocation whose tool, recorded fields and pinned inputs have the same digests as an earlier successful one then reuses its outputs and receipt instead of running again. Re-entering a paid state with unchanged inputs is never charged twice.

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
- A state that admits several roles uses `{ "one_of": [...], "by": "assignment.role" }`, or `{ "one_of": { "capability": [...] }, "by": "assignment.role" }` for every role alias of the resolved `roles` map, in the version in force, whose capability is one of those (today's open-ended roles). A selector never ranges over an installed skill that no alias binds: an installed role skill becomes a role only through an alias, and adding an alias is a change to the process (§2.4). In legacy mode the committed `.roles` overlay supplies the aliases (§14). The element records the structured role alias on assignment or spawn; until it has one, and after a move into a state that does not admit it (§4.2 rule 4), the root orchestrator owns it. *decided*
- **Helpers** are roles the owner may engage; they never own the state. Planning poker: owner `facilitator`, helpers `estimator`, answered blind and revealed by the owner. *decided* A helper's `min` is the number of that helper role's answers the owner's exits wait for, so lowering it, or removing a helper whose role is a human role, is never covered by a capability (§2.4).
- `requires` lists entry conditions: guards applied to every transition into the state and to creation when the state is `initial`.
- `terminal` states have no owner. Leaving one needs a transition marked `reopen: true`, whose actor rule is §4.2 rule 3 (aggregation, §5.1, is not a transition).
- `resolution: true` requires a resolution on entry; a list restricts the values. Automatic entries carry theirs: `resolution` on the transition, or on a `limit` redirect.
- `locked: true` means only transitions that name the state in `from` leave it; `from: "*"` never matches it.
- `record_source` names a field into which the kernel writes the source state on every entry, so a hold state can route back to where the element came from. `clear_on_exit` lists fields the kernel clears on every exit, so a reason recorded for one stay is never read for the next. Only the kernel writes a field that `record_source` names: no run, session or person does, so no actor can steer where the element resumes. The validator checks that the field's type admits every source state of the entries into the state (§10). dev-cycle's `blocked` uses both.

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
- A transition is identified by `(from, to, via, on)`; a manual transition has no `on`. `via` names the one command that performs that transition (`accept_cr`, `integrate`); it does not restrict other transitions into the same state, so a manual route and a command route may coexist, and so may a manual move and an automatic one between the same two states. An explicit transition shadows the `*` expansion for the same key.
- `guards` must all pass; `actions` run after the move. A guard object may carry `applies_to` to scope it: `actor` (`run`, `human`, `any`), `except_capability`, `own_element` (only when a run acts on the element it was spawned for) and `role_ends_in` (only for runs whose role's end state is the named state). Today's hand-off evidence gate needs the last two (§14).
- `on` makes the transition automatic: on an `event`, or `after` a duration in the source state, given as a literal or as a duration parameter (§7.3). The kernel fires it; §6.9 says when and how events are kept.
- `limit` bounds a loop: after `max` firings for one element the transition leads to `then` instead, recording `resolution` when `then` requires one. The kernel keeps one counter per declared transition and element, so a transition whose `from` lists several states shares one counter across them. `max` is a count, `none` or a parameter reference such as `$rework_limit` (§7.3). `none` leaves the loop unbounded: a deliberate choice recorded in the version, with `then` still declared so that a project can set a bound later.
- `risk` marks a transition that a signature policy may cover: `spending` (it commits money) or `halt` (it stops work under way); custom markers use `x-` (§4.2).
- Determinism: the validator refuses two automatic transitions from the same state on the same trigger with the same guards, and two transitions with the same key.

### 6.3 Guards, actions and events come from modules *(decided)*

A guard or action is a name, or an object whose `guard`/`action` key names it and whose other keys are its arguments; the schema keeps the two shapes apart and refuses arguments that carry a URL, shell syntax or a program-selecting name (§10). Names of module hooks carry the module prefix. The validator refuses any guard, action, event or `via` command that no loaded module provides and no `events` declaration names.

| Provider | Guards | Actions | Events | `via` commands |
| --- | --- | --- | --- | --- |
| kernel | `description`, `estimate` (`grandfather_legacy`), `estimates.present` (`scope`, `dimensions`), `assignee` (`manual_only`), `blockers.clear`, `children.terminal` (`types`), `children.in` (`states`, `types`), `children.any-in` (`states`), `members.terminal-or-carried`, `checklist.complete`, `outcome.present`, `fields.present` (`fields`), `field` (`name`; one of `is`, `in`, `lt`, `lte`, `gt`, `gte`), `approvals` (`role`, `min`), `verdict` (`is`, `detail`), `capacity` (`dimension`, `scope`), `process.result` (`in`) | `spawn.owner`, `assign`, `create` (`type`, `per`, `fields`, `parent`), `handoff` (`to`, `create`, `link_field`, `from_outcome`), `escalate`, `notify` (to an owner or a group), `record.verdict` (`is`, `detail`), `compute.velocity` (`dimension`) | `blockers.terminal`, `children.changed`, `process.returned`, `tool.passed`, `tool.failed` | — |
| delivery module (development only, §6.10) | `delivery.cr-published`, `delivery.review-ready`, `delivery.cr-accepted`, `delivery.cr-absent`, `delivery.commit-acknowledged`, `delivery.checks-passed` (`set`), `delivery.integration-evidence` (`set`), `delivery.external-integration-evidence`, `delivery.restore-authorized`, `delivery.rollback-authorized` | `delivery.integrate` (`checks`, required), `delivery.close-checkpointed-siblings` | `delivery.integrated`, `delivery.integration-failed`, `delivery.rework-started` | `accept_cr`, `reject_cr`, `integrate`, `integrate_external`, `restore_done`, `landed_rework`, `invalidate`, `converge` |
| timers | — | — | `after` durations | — |
| a playbook's `events` | — | — | declared external events (§6.9) | — |

`delivery.review-ready` passes for an element without a Change Request, or one whose latest revision is published. `delivery.checks-passed` passes when the named check set has evidence for the revision of the element's Change Request (§6.12); like `delivery.review-ready`, it passes for an element without a Change Request, because inline work runs no suite, as today. `delivery.integration-failed` is emitted only together with a rollback authorization (§6.10). `delivery.integrate` integrates the accepted revisions of the unit, gated by the check set its required `checks` argument names; a unit with no accepted revision integrates at once. A set the project does not define refuses the integration with `checks_not_configured` before any ref moves, as today's `validation_not_configured` does. A set that fails, or a local set that cannot run, emits `delivery.integration-failed` with its rollback authorization; a hosted half that the provider cannot run makes the board wait instead (§6.12). `delivery.integration-evidence` names the same set through its `set` argument where a command route integrates, as the default's `integrate` route does.

**Field conditions** *(decided 2026-09-24)*. The `field` guard compares one field of the element with a value: `is` (equal), `in` (one of a list), `lt`, `lte`, `gt`, `gte` (numbers, money in the field's currency, dates and date-times). It turns declared data into routes. A review level is a field whose value selects the route: in dev-cycle `{ "guard": "field", "name": "review", "is": "none" }` lets inline work skip review. A threshold selects whether a purchase needs an approval: `{ "guard": "field", "name": "amount", "gt": 500 }` beside the `approvals` guard. The guard-bypass check reads field conditions on one field (§10). The guard names a field the element's type declares; an unset field fails every comparison, and defaults are written at creation (§5.2). Fields a guard reads are authority-bearing: no run writes them on any element, approvals bind the digest of every declared field, and a tool receives the values recorded when its state was entered (§5.2, §6.4, §6.11).

**Review levels and the goal ledger.** Today the ledger accepts an item whose review level is `none` on the producer's outcome alone (`tools/board-cli/internal/spawnruntime/goal_acceptance.go:167-170`). That shortcut applies only where the element's workflow declares no `field` route on `review`, as in the default. Where a playbook routes on `review`, the ledger follows the workflow: a `none` item is accepted when its inline route finishes, and Change Request work, whatever its level, is accepted only through review and `accept_cr` (§6.10).

**Verdicts** *(closes the first run's F35)*. Verdicts use the goal ledger's vocabulary and spelling (task-board `pkg/board/goal.go:40-49`): `accepted`, `changes_requested`, `stop_the_line`, plus `rejected` — the work is refused and will not be reworked. The ledger gains `rejected` in PC1, a task-board change, so any type may route on it *(decided 2026-09-24)*. A verdict may carry a typed `detail`, and a `verdict` guard may match it: the refuter records `accepted` with `no-contradiction`, `changes_requested` with `contradiction`, and `stop_the_line` with `human-decision-needed` (§12). Reviewers and approvers record verdicts; `record.verdict` records one for the acting reviewer or approver run and nothing for any other actor. `tool.passed` and `tool.failed` arise only from a state owned by a tool role (§6.11).

**Evidence must be fresh.** A verdict, an approval or an outcome satisfies a guard only when it was recorded after the element last entered the transition's source state; for work under a Change Request it must also name the revision the guard evaluates. This extends today's rule for reviewer verdicts, which count only when recorded after the latest producer outcome (`tools/board-cli/internal/spawnruntime/goal_acceptance.go:172-173`). So a refuter's old verdict cannot move a redrafted epic, a QA state waits for the tester's own outcome, and sending a purchase order again after an escalation needs a new approval.

### 6.4 Approvals and quorum *(decided)*

`{ "guard": "approvals", "role": "manager", "min": 1 }` needs approvals from that role; `min` above one is quorum voting and needs distinct principals present in the roster at the pinned revision. An approval is a board record bound to the approving principal, and it counts only while fresh (§6.3). It is a human act (§4.2): the operator's own CLI action counts only where policy does not require a signature; otherwise it is a waggle approver signature at the configured assurance and user verification, made through the command that renders the payload. The validator flags `min` greater than the number of principals in the role's group.

- **An approval binds the element's declared fields.** Every approval, direct or delegated, signed or not, records the values of all the fields the element's type declares at the time of approval and their digest, and counts only while those fields still hash to that digest. A field declared later is not in the digest. Any change to a declared field voids the element's unused approvals, and the transition waits for a new one: an order approved at 620 is not sent at 6,200, and one approved for one supplier is not sent to another. A signed approval carries the values and the digest in its payload (waggle A1 §19.1), and the board compares the digest (§19.7). While a delegated approval is unused, the fields its delegation's ceilings, conditions and budgets read are frozen (§5.2).
- **An approval binds the parts of the process it depends on.** It records the configuration digest of the version in force (§2.4), never a commit, and the parts that decide it: the transitions that consume it — the guarded exits of the approved state — with their target states and guards, the approving role and its group, the roster lines of that group's principals, and the `policy.signatures` and `policy.delegation` entries that govern it. It never binds the rest of the workflow (the author's resolution, §27, refuter v6 V9). A commit that has not taken effect changes nothing. A version that takes effect and changes any of those parts voids the element's unused approvals; one that changes none of them keeps them valid, so adding a teammate or updating an unrelated type does not cost a manager five new approvals. The authority report lists the unused approvals a pending change would void (§10).
- **A delegated approval** counts as its grantor's approval only within the delegation (§4.2). Each delegated act counts as at most one principal toward a quorum, its grantor, and one delegate session counts once; a delegate does not approve what it or its runs filled in unless the project allows it (§4.2).

### 6.5 Hand-off between processes *(decided)*

A state owned by a `process` role is worked in another process: a playbook instance on the same board (a member of the project's composition) or on another board. The playbook names a process **slot**; the project binds it (§7.2). The `handoff` action offers the work; the receiving orchestrator accepts the offer, creates the element in its own hierarchy (placed by the rule of §5.1) and sets the link:

```json
{ "from": "verifying", "to": "fixing", "guards": [ { "guard": "verdict", "is": "accepted" } ],
  "actions": [ { "action": "handoff", "to": "fixers", "create": "fix-target", "link_field": "fix" } ] },
{ "from": "fixing", "to": "fixed", "on": { "event": "process.returned" }, "guards": [ { "guard": "process.result", "in": ["done"] } ] }
```

- **On one board** — every process of the project's composition — the offer and the acceptance are board records, and the kernel emits `process.returned` to the offering element when the received element reaches a terminal state. This path needs no waggle and ships in PC1, so the first demos can run (§15).
- **Across boards** the offer and the acceptance are waggle messages (`coord.handoff-offer`, `coord.handoff-accept`), and the result returns through the `coord` type the waggle amendment adds (§17). The result is not a model's claim: a host key of the receiving board signs it in the signer role `result` that waggle A1 adds to its verification steps (namespace `result.coord.v1@waggle`), when the receiving kernel records the terminal transition, and it carries the receiving board's revision at that transition. No orchestrator session holds that key, and no process running as the operator can use it ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §11). The offering board verifies the signature against the host principals its committed project file binds to the other board before it emits `process.returned`; the binding form comes with cross-board hand-off in PC3 (§17.3). Until the amendment is implemented, with CM2, cross-board hand-off is not available.
- `process.returned` carries the final state of the received element; `process.result` guards on it. The exits of a process-owned state must cover every terminal state of the type the hand-off creates, so no result is left without a route (§10).
- Writes into another orchestrator's leased scope go through that orchestrator (§7.2).

### 6.6 Schedules *(decided)*

Recurring work is created by named schedules:

```json
"schedules": [ { "name": "weekly-inventory", "every": "P1W", "create": { "type": "inventory-check", "title": "Weekly inventory" } } ]
```

`every` is an ISO 8601 duration; `cron` is the alternative. A schedule a package proposes takes effect only when the project enables it (`policy.enable.schedules`, §8.5): by name, or with `all`, which admits every schedule the admitted templates declare, including ones later versions add. `task-board tick` fires schedules, and each firing is keyed by (schedule, due time) in the lease store, so two ticks never create two elements for one firing (§6.9). Under a `schedules` capability, which a person grants once per process, a process adds, changes and enables schedules within the grant's limits without a touch; any other change to schedules waits for a person (§2.4) *(decided 2026-09-25)*.

### 6.7 Short goals *(decided)*

Orchestrators lose the thread on large goals; the weaker the model, the shorter the step must be. `policy.goals.orchestrator_goal` names the type whose current instance is an orchestrator's goal — a story, a milestone or a sprint. In a composition it is a map from orchestrator role to type, so each orchestrator has its own. The long-term goal lives in the hierarchy and each orchestrator works one bounded step at a time. The goal ledger itself stays in task-board, and verdicts use its vocabulary (§6.3).

### 6.8 Policy

| Key | Meaning | Set by |
| --- | --- | --- |
| `review.default` | `required`, `light` or `none`: the default of the element field `review` for new elements (today's review policy); routes select on the field with `field` (§6.3) | template or project |
| `wip` | limits on the elements in progress: `board`, `per_role`, `per_state` (below) | template or project; the project wins |
| `capacity` | per container type, the maximum sum of one estimate dimension (a sprint of 100 complexity points) | template or project |
| `privacy` | view visibility and redacted fields (§5.4) | template or project |
| `goals` | §6.7 | template or project; the project wins in a composition |
| `dreams` | when this process's orchestrators may dream: `windows`, a per-night `budget` of tokens and minutes, `conditions` that must all hold (idle time; subscription quota above a share, quota that would expire unused, or a ready local engine; AC power), and what the host does `after` a dream, resume or stay parked (§6.13; proposed) | template or project |
| `effort_floor` | per role, the lowest effort a run may use, applied per model row (§9) | project only |
| `enable` | `schedules` (`all` or named), `auto_spawn`, `tools` (`all` or named), `relaxations`, `standing_answers`, `skills` (`audited` by default, named, or `none`) (§8.5, §8.6, §9) | project only |
| `confirmers` | the `people` group whose principals confirm changes to the process; without it, the board's anchor operator (§2.4; [`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §3) | project only |
| `signatures` | which human acts need a waggle approver signature, the minimum key assurance and `user_verified` (§4.2) | project only |
| `delegation` | which human acts a person may delegate, to which orchestrator roles, under which ceilings; `never` wins (§4.2) | project only |
| `data` | publishers denied, models allowed and denied (`models`), billing classes admitted, local only (§9) | project only |
| `delivery.version_control` | `confirm`, `desired_commit_time` (required when `confirm`), `commit_time_policy`; owned here since v3, and `task-board.config.json`'s `version_control` keys are read for one release with a warning (§2.1) | template or project |
| `delivery.loop_detection` | rework threshold `max(floor, factor × median)` | template or project |
| `delivery.run_write_boundary` | `warn` (default) or `refuse`: what the delivery module does when a run writes outside its scope; under `refuse`, reviewer launch, verdict acceptance and delivery refuse until an operator clears the run's marker (today `spawn.worktree_isolation.run_write_boundary`, read for one release, §2.1, §14) | template or project |

**Work in progress** *(decided 2026-09-24)*. `policy.wip` bounds the elements in progress:

```json
"wip": { "board": 12, "per_role": { "developer": 3, "reviewer": 2 }, "per_state": { "dev.review": 2, "purchase.approval": 5 } }
```

- Each count may be a literal or a `count` parameter (§7.3). `per_state` counts the elements in one state, named `<workflow>.<state>`, or a bare state name for that state in every workflow that has one. `per_role` counts the elements whose current state the role owns, or whose `assignment.role` is the role. `board` counts the elements in non-initial, non-terminal states.
- Elements are counted, not runs, so a limit also holds where no run works: "no more than five orders awaiting approval" bounds a human-owned state. Board elements are committed and published, so every teammate counts the same elements.
- A manual or agent transition that would take an element over a limit is refused with `wip_limit_reached`, naming the limit that held, so no run starts for it. An automatic transition into a full scope is held, like a kept event (§6.9), and fires when capacity frees.
- **Escalations to people are never throttled.** A limit never refuses an entry into a human-owned state that a verdict, a `limit` redirect or an automatic escalation routes to, so a limit cannot keep a decision from the person the playbook sends it to (the author's resolution, §27, refuter v6 V2).
- The limits describe the process. What one computer can carry — parallel runs across every board, concurrent local check sets, rate-limit ladders — is the operator's machine capacity (`runtimes.toml`, §2.3); a run whose element fits `policy.wip` still waits for machine capacity. Today's `spawn.max_parallel` and `spawn.validation.max_concurrent_suites` become machine capacity, and a playbook without `policy.wip`, the default included, has no process limit (§14).

### 6.9 Timers, schedules, tools and external events before the main loop *(decided)*

Time-based transitions (`after`), schedules, approval expiries and tool invocations need a process that runs without an agent. The main loop will be that process; until it exists, `task-board tick` does the job:

- **Scheduler** *(decided 2026-09-24)*. A user-level system timer calls tick every minute: a launchd agent on macOS or a systemd user timer on Linux, installed by `task-board tick --install-timer`. It runs whether or not any session is open. The main loop takes the job over when it exists.
- **Scope.** Tick fires due `after` transitions, due schedules, expiries of `ask` approvals and pending tool invocations (§6.11), recording actor `kernel`.
- **Lease.** Tick takes a lease first. On one machine that is the board's own lease; a board shared across machines needs waggle's lease store (NATS JetStream KV, milestone CM2). Until CM2, a shared board runs tick on one machine only, which the project documents. Delegation budgets and revocations use the same store (§4.2).
- **Idempotence.** Every firing has a key: (element, transition, due time) for timers and expiries, (schedule, due time) for schedules, (element, entry) for tool invocations. Keys live in the lease store, so running tick twice, or on two machines under one lease store, fires each item once.

External events enter through `task-board event post <event> --element <id>` with a signed payload. A template declares the events it reacts to and names a signer group; only the project binds the group to principals (`services`, §4.2):

```json
"events": { "supplier.confirmed": { "signed_by": "supplier-bridge" } }
```

- An unsigned event, a wrongly signed one and one whose signer group the project has not bound are refused and logged.
- A bridge that turns a provider's webhook into a signed event first verifies the provider's own webhook authentication (its signature scheme or mutual TLS), and signs only events of the groups it is bound to. Receiving webhooks needs a service the ecosystem does not run yet.
- **Events are kept.** An accepted event stays on its element until it fires a transition or the element reaches a terminal state; each kept event fires at most one transition, and one that matches no transition in the current state is logged and kept. `children.changed` describes current state and is not kept: on every entry into a state the kernel evaluates that state's automatic transitions on it as if it had just fired, and its other automatic transitions against the kept events. `blockers.terminal` is an edge, as today: it fires for an element with at least one `blocked_by` link when its last linked blocker reaches a terminal state, and never for an element without links, whose resume is manual. An element whose blockers were all terminal before it was blocked is reported as stale, not released (`pkg/board/blocked_release.go:13-23`). So a supplier confirmation that arrives after the SLA fired is not lost (cafe), and an attempt accepted while its milestone was stalled still proves the milestone when it reopens (formal missions).

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
- The kernel's executor runs it. The executor is one kernel component: PC1 ships it for local check sets (§6.12), and from PC3 `task-board tick`, under its system timer until the main loop exists (§6.9), calls the same executor for tool invocations, timers and schedules.
- It runs only the command the binding names, from the provider identity in force (below), checked against the Curator lock (its owner must be the bound skill), or under the enforced `script-worker-v1` policy when the provider declares it (curator-spec `protocol/core.md` §4.1.1).
- **It runs the provider identity in force.** The executor runs the provider's package identity recorded in the version in force (§2.4), taken from Curator's content-addressed store, never whatever the shim on `PATH` points to now. Installing a new provider version changes nothing that runs until a version that names it takes effect: under the `tools:<name>` capability for an audited version, or on a person's confirmation (§2.4). No code a person has not let in runs under a confirmed binding, and no invocation fails because an update is pending.
  - **When the identity in force has left the store**, the invocation is held under its (element, entry) key and reported; it runs once the identity is back or a version that names an available identity takes effect.
  - **`tool.failed`** is reserved for executions that actually ran (the author's resolution, §27, refuter v6 V6).
- The environment is sanitized: a fixed `PATH` of the managed command directory and system directories, and no variables inherited from agent runs. The working directory is the project root.
- The contract: the command receives `--element <id>` and, on standard input, a JSON object with the element's id, type and declared fields as the kernel recorded them when the element entered the state, keyed by (element, entry), the playbook, the tool name and the entry key. A write to the element between the entry and the invocation never changes what the tool acts on, and when the state was entered through an approved transition these are the values the approval bound (§6.4). Exit status 0 fires `tool.passed` and 1 fires `tool.failed`; any other status, a crash or the binding's `timeout` (ten minutes by default) fires `tool.failed` with the reason `error` or `timeout`. Bounded standard output is stored as the tool's outcome. The executor never retries; a `tool.failed` exit with a `limit` models retries.
- A person may leave a tool-owned state through its declared manual exits (§4.2 rule 2); a running invocation is then cancelled.
- A tool runs only when the project lists it in `policy.enable.tools` of `narrowed(version in force)` (§2.4), so a tool a person has disabled by a narrowing act does not run, and only from a skill installed through Curator (§8.4, §8.5). Entering a state whose tool the project does not list is refused.
- **In a dream** (§6.13) the executor adds `--dream` to every invocation and refuses a tool whose provider does not declare dream support. *proposed*

### 6.11.1 Tool outputs, receipts and assisted tools *(proposed, 2026-09-28, PC3)*

Found by expressing a real music-video production as a playbook (`examples/music-video/`). A tool often knows more than pass or fail, some tools spend money whose final amount only the provider knows, and some need a person in the middle of a run.

- **Outputs.** A tool binding may declare `outputs`: fields of the element's type that the tool reports in a typed block of its standard output. The kernel alone writes them, only on the invocation's own `tool.passed` or `tool.failed`, and records them with the outcome, so a guard, a delegation ceiling or a later brief can read them. Examples: where a generated clip's soundtrack left the reference vocal, the image the tool picked, a measured score. A field a `field` guard reads may be an output; no run or person writes it while the element is in the tool's state. An output the block omits or mistypes fails the invocation with the reason `bad-output`.
- **Receipts.** A tool that spends declares `spends` with the money field its approvals and ceilings read. It reports a receipt: the provider's reference, the amount actually charged and the currency. The kernel reconciles every budget of the delegation that approved the spending with the receipt instead of the estimate. A receipt above the approved amount by more than the grant's tolerance opens a hold for the approver. An invocation with no receipt keeps the estimate counted and is reported as an unknown outcome, never as free. The entry key on standard input is the idempotency key the provider call carries, so a retried invocation is not charged twice where the provider honours it.
- **Assisted tools.** A tool may declare `assist`: in the middle of a run it can ask a person to act (solve a captcha, sign in again, confirm on a device) through a typed request. The element stays in the tool's state, the request reaches the owner of the named human role through their inbox, and the invocation waits up to the binding's timeout instead of failing. A tool that needs a person's own signed-in session runs under that person's operating-system account, never under the agents' account (curator-trust `spec/trust.md` §12), and the project names whose session it uses.

The schema follows when these are adopted.

### 6.11.2 Long-running tools, machine resources and sandboxes *(proposed, 2026-09-29, PC3)*

- **A pinned tree.** A tool runs in a clean checkout of the version in force at the element's recorded revision, with its pinned inputs materialized, as a local check set does (§6.12), never in the live working tree. Its evidence names the tree, the provider identity and the toolchain versions the provider reports.
- **Progress and resume.**
  - A binding may declare `resumable: true`. The provider keeps a checkpoint under the invocation's key, and the executor invokes it again with the same key after a crash, a restart or a lost lease; the provider continues from the checkpoint.
  - Typed progress lines (a fraction and a phase) are recorded and shown.
  - With `progress_timeout`, the deadline is the time without progress rather than the total run time, so a 23-minute render is not killed at ten.
- **Late results.** A result that arrives for the same key after `tool.failed` with the reason `timeout` is recorded, and the kernel emits `tool.late-result`. A workflow may route it like the late supplier confirmation of the cafe example.
- **Machine resources.** A binding declares the resources it needs by name, for example `gpu`, `render` or `browser:director`. The operator's `runtimes.toml` declares each resource's capacity on this machine (§2.3). The executor leases the resources before running and queues otherwise, and it releases them on exit, crash and cancel. This replaces `policy.wip.per_state` as a stand-in for machine limits: one GPU shared by renders, analysis and lip-sync is a lease, not a process rule.
- **Quality tiers.** A binding may declare `tiers`, for example `preview` and `final`. The tier is part of the invocation key and of its evidence. A guard or a check set may require a tier, and preview evidence never satisfies a requirement for `final`, as dream evidence never counts (§6.13).
- **Tools that run element content.** A binding that executes code an agent wrote, such as a renderer or a scene engine, declares `executes: "element-content"`.
  - The executor runs it in a sandbox profile: no network unless the binding declares it, writes only to the invocation's artifact directory, no credentials, and a CPU and memory cap.
  - The checks such a tool applies come from the binding and the version in force, never from files the element produced, so a scene cannot switch off its own checks.
- **Readiness.** A provider may declare a `probe` command. The kernel runs it when a version that binds the tool takes effect, and before the first entry into a state the tool owns on a machine. A failing probe refuses the entry with `tool_not_ready` and the probe's reason, such as a missing browser or a GPU backend that falls back to software, instead of failing in the middle of a run.

### 6.12 Check sets *(decided 2026-09-24)*

Evidence that a machine produces for delivery — tests, linters, builds — comes from named **check sets**. The playbook says which set must pass where; the project says how and where each set runs.

- A playbook names a set in the guard `{ "guard": "delivery.checks-passed", "set": "handoff" }`, as the required gate of an integration, `{ "action": "delivery.integrate", "checks": "landing" }`, or on a command route through `{ "guard": "delivery.integration-evidence", "set": "landing" }`. dev-cycle uses `handoff` before review and `landing` for its story's integration; the default names `landing` on its `integrate` route.
- `task-board.config.json` defines each set under `checks`:

  ```json
  "checks": {
    "handoff": { "where": "local", "commands": ["make test-fast"],
                 "by_task_class": { "docs": { "where": "local", "commands": [] } } },
    "landing": { "where": ["local", "hosted"], "commands": ["make test-smoke"], "hosted": { "provider": "github" } }
  }
  ```

  - `where` is `local`, `hosted`, or both as a list; every place listed must pass.
  - `commands` are required when `where` includes `local`. An explicitly empty list is a recorded decision to run nothing, as today.
  - `hosted` names the hosting provider whose required checks must pass on the exact head that lands; it is required when `where` includes `hosted`.
  - `by_task_class` gives a set a variant per task class.
- **Local sets run outside the gated actor.** A check set's evidence gates the actor's own transition, so, like a tool (§6.11), it never runs in that actor's process or environment. The kernel's executor, which PC1 ships (§6.11), runs a local set in a clean checkout of the exact tree under evaluation, with a sanitized environment (a fixed `PATH` of the managed command directory and system directories, no variables inherited from agent runs), under the machine's capacity for concurrent check sets (§2.3), and records the executor, the tree and each command's exit status with the evidence.
- **Evidence binds to the revision.** A local set binds to the tree it ran on, a hosted set to the head whose required checks the provider reported. The delivery module produces the evidence: it runs `handoff` when a producer publishes a Change Request revision, runs `landing` again at integration when the bound evidence is missing or stale, and collects a hosted set's result from the provider. The guard only checks that the evidence exists.
- **Inline work runs no suite.** An element without a Change Request has no revision for a set to check, so `delivery.checks-passed` passes for it and dev-cycle's inline-only routes name no set, as today's landing-gate suite binds only to Change Request builds and integration.
- **When hosted checks cannot run.** When the provider cannot run the hosted half of a set for a verified external cause, the kernel's executor may run the matching commands in a clean checkout of the exact head and record the result as `mirror` evidence. Mirror evidence never counts as the hosted half and never produces, edits or stands in for the provider's own status. While the hosted half is missing the board waits. A person may authorize the landing with an override, which `policy.signatures` may require to be signed (`overrides`, §4.2); the board records the override together with the mirror evidence, and the provider's branch protection still decides whether the hosted repository accepts the head.
- A process without the delivery module, such as the cafe, has no check sets and no dependency on a git host: its evidence comes from people (approvals), tools (§6.11) and signed events (§6.9).
- A set a playbook names must be defined in the code owner's `task-board.config.json`. In one repository both files are committed, so the validator checks it portably (§10); when the board and the code live in different repositories (§2.1), each Story's sets come from its code owner's file, and the check runs when that Story's code repository is known. An undefined set refuses integration (§6.3).
- **The definition in force.** The check sets a playbook names are part of the resolved configuration (§2.4): the kernel reads their `where`, `commands`, `hosted` and `by_task_class` from `task-board.config.json` at the proposed head, or from the code owner's default branch, and runs the definition of the version in force. It never reads a set from the working tree or from the tree under evaluation, so a producer cannot empty the set that gates its own Change Request, and emptying a set is a change that waits for a person.
- Today's landing-gate suite, `spawn.worktree_isolation.validation.commands`, is a local `landing` set (§14).
- **Trunk verification after landing** *(proposed, 2026-09-28)*. A landing is not the end of evidence: the hosting provider may run a different pipeline on the trunk push than on the reviewed head, and a trunk can go red after a green review. A set may name `"after_landing": true` together with `hosted`. The delivery module then waits for the provider's trunk pipeline on the landed head and records the result bound to that head. The element stays in its landing state until the result is green; `unknown` is never green. A red result opens a Bug linked to the landed element through `handoff`, and does not reopen the reviewed work. dev-cycle names such a set for its integration.
- **Merge policy for append-only files** *(proposed, 2026-09-28)*. Files that only grow, such as a changelog or a logbook, conflict on almost every landing. The project declares them in its `.gitattributes` with git's built-in `merge=union` driver; the delivery module honours it when it lays an accepted candidate over a moved trunk and lists every file merged that way in the landing evidence, so the reviewer of the landing checks them. The kernel adds no merge code of its own.
- **Hosting providers** *(proposed, 2026-09-28)*. `hosted.provider` takes `github` or `gitlab`. The provider supplies: the required checks for an exact head, the head of a pull or merge request, a review verdict (a comment review where the host refuses self-approval, as GitHub does; the approval API where a pipeline requires an approval, as some GitLab pipelines do through a project token), and the trunk pipeline for a landed head. Credentials stay in the provider CLI's own store; the evidence names a missing variable, never its value. The first implementation is the tool `delivery` (relux-works/skill-delivery), which the landing state's tool owner calls (§6.11); until PC3 the current board calls the same tool and records the landing with `task-board worktree close-landed`.

### 6.13 Dream mode *(proposed, 2026-09-25)*

A process can be rehearsed before it is trusted with the real world: a new playbook from the process designer, a changed workflow, a demo, an evaluation of agents. The operator's image is a dream: everything looks real to the agent, the agent knows it is dreaming, and nothing it does reaches reality.

- **The tool contract.** A tool — a command that a provider skill exports (§8.5) — declares in its Curator manifest that it supports dreaming. Called with `--dream`, it has no real-world side effect: it sends no order, charges nothing and writes to no external system. It returns output as realistic as it can, marked as dream output. A provider that declares support is responsible for keeping that promise, and the skill's audit covers the declaration.
- **The executor.** In a dream the kernel's executor (§6.11) adds `--dream` to every invocation and refuses a tool whose provider does not declare dream support, so the state that tool owns cannot be entered in the dream. Local check sets (§6.12) run as they do outside a dream, in a clean checkout of the dream's tree; a hosted set never runs in a dream, because it would publish to the hosting provider.
- **The sandbox.** A dream runs against a copy of the board: a scratch branch or worktree with its own board state, never the board whose version is in force. Integration in a dream lands on the sandbox's scratch trunk; landing on the default branch and every command that reaches a remote are refused. Starting a dream records which version of the process it rehearses.
- **The agent knows.** A dreaming session is told so by its profile and by every doorbell the session host rings, and every output the executor returns is marked as dream output. Nothing in a dream pretends to be real.
- **Dream evidence never counts.** An approval, verdict, tool result, event or confirmation recorded in a dream never satisfies a guard, an approval, a capability or a confirmation outside it. A dream ends with a report on what it rehearsed — for example "rehearsed in a dream: the order reached `paid` through these steps" — which a person may read before confirming a version; the report is evidence of a rehearsal, never of real work.
- **No real secrets and no real privileged operations.** A dream sandbox holds no real secrets, and privileged operations inside it are only simulated ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md)).
- **Past sessions, distilled** *(proposed)*. In a dream the orchestrator has read-only access to the full harness logs of its own previous sessions. A separate deterministic tool, the **log distiller**, prepares them: it removes noise — code blocks, diffs, long tool outputs, repeated content — without summarizing anything, and keeps the identifiers that link each passage back to level 0 of the memory ladder, the journal (curator-keeper `spec/keeper.md` §13.4). The distiller is ordinary code, not a model, so what the dream reads is the sessions' own words with the noise taken out.
- **Dream memory** *(proposed)*. Insights a dream produces go to a separate store, the dream memory, quarantined until a person reviews them. The system roles — the process designer, the librarian and the sysadmin (curator-keeper §9, §12) — read it and turn insights into proposals: improvements to a playbook, which wait for a batched touch like any other change (§2.4), and improvements to the apiary itself, which are change sets to the apiary's own repository, applied only with the person's consent. Nothing in dream memory changes a process or the apiary by itself, and nothing in it is an instruction.
- **The nightly cycle** *(proposed)*. A dream is therefore also the node's offline cycle, run at night or whenever the node is idle: replay of the distilled logs, rehearsal of processes with `--dream` tools, consolidation into the memory ladder (session and day summaries), and new insights into dream memory. It runs under the main loop's own process, on its schedule (curator-keeper §13.6).
- **Entering a dream** *(the operator's direction, 2026-09-25; the form is proposed)*. A dream is never entered by accident, and it never spends what the person did not allow.
  - **Permission.** A process's orchestrators dream only under a capability grant of the kind `dreams` ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §16), asked once per process. A dream cycle then needs no touch: its outputs carry no authority, and every proposal it produces takes the ordinary batched confirmation (§2.4).
  - **Policy.** `policy.dreams` (§6.8) says when. It gives time windows, a per-night budget of tokens and minutes, and conditions that must all hold:
    - the node is idle: no person attached, no urgent queue;
    - capacity is there: subscription quota above a share, or quota that would expire unused before its reset, or a local engine that is ready or ensurable;
    - the node is on AC power.

    The main loop has a dream policy of its own, and it schedules the dreams of every process within the node's capacity (curator-keeper `spec/keeper.md` §13.6).
  - **Entry.** The session host asks the orchestrator to park, which writes its park note, and stops the session. It then starts a dream session with the role's dream profile: a Curator profile variant of the role's profile, with its own system-prompt modules ("you are dreaming…"), dream-mode tools, the log distiller and the consolidation skills, running on the dream sandbox.
  - **Output.** The dream writes three things:
    - consolidation into the memory ladder, derived from real logs and within the memory budget;
    - quarantined insights into dream memory;
    - rehearsal reports.
  - **After.** As `policy.dreams.after` says, the host either resumes the normal session from its goal, its park note and a pointer to the dream's report, or leaves it parked.
  - **Waking.** A real high-priority doorbell, such as a message from a person, wakes the dreamer. The host parks the dream in its own state and resumes the normal session.
- **Uses:** rehearsing a new or changed process before its version takes effect; the nightly offline cycle; training and evaluating agents and roles; speculative planning; safe demos; red-team exercises against a process.
- **Precedent.** task-board already has a development-only fault injector for provider limits, `internal/limitsimulate`, which arms a token and captures what would have been sent instead of sending it. Dream mode generalizes the idea to every tool a process uses.
- **Open** (§17.3 item 19): how a dream is scoped (an element, a workflow, a whole process), how long its sandbox lives, whether a project may require a dream rehearsal before a structure change is offered for confirmation, and how dream profiles are packaged (item 16).

### 6.14 Budgets, parameters and waves *(proposed, 2026-09-29)*

- **Units other than money.** A budget, in a grant (§4.2) or in `policy.budgets`, sums one of:
  - a declared numeric field, such as a `money` field in its currency;
  - a measured quantity: `tokens` and `minutes`, which the session host and the executor measure per run and invocation;
  - provider `credits`, reported by a tool's receipt (§6.11.1).
- **`policy.budgets`** sets limits per container type, over the container's lifetime or a period. Examples: `{ "type": "video", "sum": "cost", "max": 150 }`, `{ "type": "chapter", "measure": "tokens", "max": 2000000 }`.
  - At 90 % the owner is notified.
  - At 100 % new entries into agent- and tool-owned states under that container are held.
  - An escalation to a person is never held, as for `wip` (§6.8).
- **State budgets.** An agent-owned state may declare `budget: { "tokens": …, "minutes": … }`. The session host enforces it as it enforces harness budgets: a wrap-up notice at 90 %, then a pause and a message to the orchestrator at 100 %.
- **Measured estimates.** An estimate dimension may declare `scale: "measure"` with a `unit`, for example seconds of generated video, so capacity rules sum real units instead of points.
- **Parameters in two more places** (extends §7.3):
  - the comparison value of a `field` guard, such as a platform's maximum duration or a lip-sync threshold;
  - the `settings` a tool binding declares, which the executor passes to the tool on standard input.

  The new parameter types are `number` and `string`. A parameter still never changes who owns a state, which states exist or where a transition leads.
- **Grants tied to a plan.** A spending grant may name an element and bind its approval digest: a plan, such as a storyboard with its shots file. The grant then covers spending only on that element's descendants while the digest holds. A changed plan suspends the grant until the person confirms it again.
- **Waves.** A `create` action with `per` may declare `waves: { "sizes": [2, 4, 8, 10], "barrier": "<state>" }`.
  - The kernel creates the first batch.
  - When every element of a batch is terminal, the parent enters the barrier state, usually owned by a reviewer or a person.
  - The next batch is created on the barrier's exit that names `continue_waves`.
  - A batch with a failed element does not grow the next one.

### 6.15 Publishing and the outside world *(proposed, 2026-09-29)*

The kernel gets hooks only. Platform work (uploads, per-platform formats, release scheduling, rights) belongs to a publishing module, as development machinery belongs to the delivery module (§6.10).

- **`publish` risk.** A transition that releases something outside the process, such as a post, an upload or a message to customers, is marked `risk: "publish"`. `policy.signatures` may require a signed human act for it, and `licence.cleared` (§5.5) must pass for every file it releases.
- **Absolute time.** A trigger may be `on: { "at": "<field>" }`: the transition fires when the time in the named `datetime` field arrives, in the time zone the field records. This is how a release is scheduled.
- **External data.** A field may declare its `source` (a declared external event or a tool) and a `max_age`. The guard `data.fresh` passes when every such field on the element was recorded within its `max_age`, so an explainer is refused while its numbers are stale.
- **Metrics after publishing** arrive as declared, signed external events (§6.9) that write declared fields. Choosing between variants is then an ordinary `field` guard on those fields, after a timer.
- **Intake.** A declared external event may create an element of a named type, with its payload mapped to declared fields (`events.<name>.creates`), subject to `policy.wip` and the process's privacy. A brief sent from a form or a channel then starts the process without a person typing it into the board.
- **Per-platform versions** are child elements created per a list of platforms. Each carries its limits, such as length, aspect ratio, loudness and file size, as parameters.

### 6.16 Review quality *(proposed, 2026-09-29)*

- **Typed findings.** A reviewer's verdict may carry findings. Each has:
  - `where`: a time code, a frame range, a file and line, or a field;
  - `problem` and `fix`;
  - a `severity`.

  The run of the fixing state receives them as structured input, and a `verdict` guard may match the highest severity. Blind helpers answer in the same shape, plus a `score` for each sibling they judge (§4.2).
- **What could not be verified.** An outcome may carry `unverified`, a typed list of claims the run could not check. The guard `unverified.empty` passes only when the list is empty; a workflow routes a non-empty list to a person or a verifier.
- **Seams between siblings.** Parallel siblings, such as chapters, meet at seams. A parent checks them in a `seams` state after `children.terminal`, owned by a reviewer or a tool. This needs no new mechanism, and the generality vectors test it as a pattern (§11).
- **Write scopes.** An agent-owned state may declare `writes`: path patterns, relative to the execution root, that its runs may change, for example one chapter file per animator. The delivery module's run write boundary enforces them as it enforces the Story worktree (§6.8).
- **Model needs.** The `capabilities` of the needs vocabulary gain `vision`, `video-input` and `audio-input`, which the model registry must carry (skill-agents-management). A reviewer of rendered frames `require`s `vision` (§9). One study's machine checks all passed on a render whose character faced away from the camera; only a review of the frames caught it.

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
3. Transitions are keyed by `(from, to, via, on)` (§6.2): a child transition with the same key replaces the parent's; `remove: true` deletes it. A state with `remove: true` deletes the state and every transition touching it. Removing a key the parent does not have is an error, so drift after a parent update is loud.
4. Lists other than transitions (`contains`, `parents`, `helpers`, guards, actions) are replaced as a whole.
5. The validator runs on the resolved result (§10) and reports which file introduced each error.
6. The board records, with the version in force, the resolved template's package identity and content and the project file's revision and digest (§2.4). Every machine runs that stored version; a machine whose own resolution of the proposed head gives a different configuration digest reports both and runs nothing new until the difference is resolved, because the kernel resolves templates only from the project scope and its committed lock (§8.2). A template update that removes or renames states declares `migrations` (old state → new state) or the validator lists the stranded elements and refuses.

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

### 7.3 Parameters *(the operator's direction, 2026-09-25; the form is proposed)*

A template fixes the shape of a process, but some numbers in it are a team's to choose. How many rounds of rework come before a person steps in differs between teams: the operator has real cases where thirty rounds were legitimate. So a template names its tunables once, and a project sets them without restating the transitions that use them.

```json
"params": {
  "rework_limit":   { "type": "limit",    "default": "none", "description": "…" },
  "block_limit":    { "type": "limit",    "default": 3 },
  "confirm_within": { "type": "duration", "default": "PT15M" }
}
```

- **Declaring.** `params` maps a snake_case name to a declaration: a `type` and a `default`. The types are:
  - `limit`: a loop limit's `max`, a count or `none`;
  - `count`: a count, for example a work-in-progress limit;
  - `duration`: an ISO 8601 duration, for example a timer.
- **Using.** Three places may hold a reference `$name` instead of a literal: a transition's `limit.max`, an `on.after` duration, and the counts of `policy.wip`. Nothing else takes a reference. A parameter therefore never changes who owns a state, which states exist, or where a transition leads.
- **Setting.** A project sets a value as a plain value: `"params": { "rework_limit": 30 }`. In a composition a project may qualify the name with the playbook's name, as in `"dev-cycle.rework_limit": 30`, and must do so when two used playbooks declare the same name.
  - A project value wins over the template default.
  - A value for a name that no resolved playbook declares is an error (§10), and so is a value of the wrong type.
- **Unbounded is a choice.** A limit may be `none`: the loop is unbounded, and the version records that like any other value. The transition's `then` stays declared, so a project can set a bound later without restating the transition.
  - The validator warns about every cycle that passes neither a bounded `limit` nor a human-owned state (§10).
  - When a project's `properties` require bounds (§17.3 item 17, after PC1), the warning becomes an error.
- **Changing a value** is a change to every place that references it (§2.4):
  - a work-in-progress parameter is covered by the `wip` capability, as the limit itself would be;
  - a loop limit or a timer is workflow structure, and a new value waits for a person's touch, batched like any other change.
- **dev-cycle** declares `rework_limit` and `rollback_limit` with the default `none`. Today's board has no hard bound on either loop, and delivery loop detection (§6.8) stays the soft signal. `block_limit` defaults to 3 (§13).

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

Every role directory is a real skill: instructions in `SKILL.md`, optional `references/`, `assets/`, `data/` — the roots Curator installs. The template lives in the orchestrator skill because the root role owns the process. Packages are named `curator-playbook-<purpose>`. Role metadata sits in `data/role.json`, not in the skill's frontmatter. The file also lists the other skills the role needs, as today's `.roles/<role>/skills.md` does; it has its own JSON Schema, which arrives with PC2, and the validator checks it *(decided 2026-09-24)*. When sources disagree, the project's `Playbook.json` role entry wins over the template's role entry, which wins over `data/role.json`. What `data/role.json` contributes — capability, default needs, definition-of-done items, needed skills — is part of the resolved configuration at the identity the project's lock records (§2.4), so a newer role skill whose contribution differs waits for a person; an audited one whose contribution is unchanged only gives the role new instructions and takes effect under the `skills` capability, which follows `policy.enable.skills` (§2.4). Tool providers live in `tools/`, outside the collection, for the reason in §8.5.

**Role skills reach only the runs of their role** *(decided 2026-09-24; the mechanism revised on 2026-09-25)*. Harnesses pick skills by their description, so a role skill in a discovery directory can fire in any session of the project, and an orchestrator could start acting as a reviewer. Role skills, the root orchestrator's included, are therefore never materialized into the project's or the machine's discovery directories. Each run launches with the Curator environment profile of its role, whose profile-scoped skill set carries that role's skill, and the project's primary session launches with the root orchestrator's profile; a plain harness session in the checkout sees none of them. The launch-profiles work provides this: a per-role Curator profile at spawn (`spawn --profile`, and `spawn.profile` in the project's configuration, formerly `spawn.context_profile`; launch profiles) and task-board as a consumer of Curator's launch fragment for its tracked children (curator-spec Decisions 0010 and 0019). It replaced on-demand activation and a skills launch channel on 2026-09-25, and curator-spec issue #91 and curator issue #91 are closed. Whether a playbook package ships those profiles itself is §17.3 item 16. Until per-role profiles ship, role skills install as ordinary skills and their descriptions say "use only when spawned as role X", a stopgap that depends on the model obeying it.

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
    { "from": "cafe", "directory": "skills", "include": ["*"] },
    { "name": "security-reviewer", "from": "roles", "directory": "skills/security-reviewer" }
  ]
}
```

The first selector installs the orchestrator and every role of the cafe; the provider in `tools/` arrives as the orchestrator's runtime dependency (§8.5). The second selector adds a reusable role from another repository, which `Playbook.json` then binds to an alias. Until per-role profiles ship (§8.1), these selectors install the role skills as ordinary project skills under the description stopgap. Afterwards the role skills move into their roles' profiles, and the project's Skillfile keeps what every session of the project needs; the concrete form is §17.3 item 16.

**Resolving a template**. The kernel resolves `"extends": "cafe-ops"` only from the project scope and its committed lock, or from its own built-in templates:

- **Installed templates.** Among the admitted playbooks (§8.5), the kernel resolves the skill whose `data/playbook.json` has that name; two such skills in the scope are an error.
- **Built-in templates.** The kernel ships the default playbook and, from PC1, the three demo templates — `dev-cycle`, `bug-hunt` and `scrum` — with their role skills, from this repository. They are part of task-board's release, admitted like the default without Curator, and a project names them in `extends` and `uses` like any template. PC2 publishes them as the packages `curator-playbook-dev-cycle`, `curator-playbook-bug-hunt` and `curator-playbook-scrum` in the Curator registry, which then replace the built-ins (§15; the author's resolution, §27, refuter v6 V7).
- **What waits.** The identity of a template-carrying skill is recorded in the resolved configuration, but a change of identity alone is not a waiting difference. An audited new version whose template content is unchanged takes effect under the `skills` capability; a change of template content is classified section by section like any other (§2.4). A template installed only in a machine's hybrid or global scope is never in force, so every machine resolves the same template for the same head, and a run in its own Curator profile (agent-selection §6) does not change the template in force. The validator and a constructor may report templates found in the other scopes, in Curator's precedence order project, hybrid, global (curator-spec `profiles/manager.md`), as candidates to add to the project.

### 8.3 The Curator work *(decided)*

Five changes belong to Curator, not to this module:

1. Release Skillfile schema 2 from its flag.
2. `dependencies.skills` entries in a skill manifest (curator-spec `protocol/core.md` §4.4: `git`, exact `ref`, activation `mode`, command narrowing) gain the same `directory` selection as schema 2 individual selectors, with the same containment rules, so an orchestrator can depend on a tool provider in its own repository and a playbook on a role in a subfolder of another repository.
3. **Command-only installs** (curator-spec issue #95, curator issue #95): a direct declaration with activation `runtime` publishes a skill's commands without its context, so a package's command line is on the person's `PATH` while its role instructions stay in its profile. The person spells it `curator global add <skill> --commands-only`; the lock may still record the activation as `runtime`. This replaced the item's earlier content, activation `on-demand` and a skills launch channel (curator-spec issue #91, curator issue #91), withdrawn on 2026-09-25 in favour of per-role profiles (§8.1).
4. Registry search for agents: a command that searches the Curator registry and reports, for each skill, what it does, what it exports and which exact package identities its audit covers, so an install can check that the identity it locks is the audited one (§8.6).
5. Skill sources outside the registry: Curator's own configuration holds the sources the operator admits, as a widening of kind `skills` that `curator sources add` records and `curator sources confirm` confirms at a level never below `user`; `curator sources` replaces `curator audit --allow`. `curator sources submit` sends a skill from an unknown source to the registry's audit (§2.3, §8.6).

Three further Curator proposals filed on 2026-09-25 concern installation rather than this module: source shorthands and default package directories (curator-spec issue #92), export and import of profile and overlay declarations for backup and restore (#93), and an attestation-based fast install path with a signed bootstrap (#94).

### 8.4 Without Curator

Role directories copied by hand into the harness skill directories (`.claude/skills`, `.codex/skills`, `agents/skills`) are skills from unknown sources (§8.6) *(the author's resolution, §25)*. The resolver refuses a role skill or a template it finds there unless the operator's confirmed `skills` setting in Curator's configuration (`curator sources`) admits that exact content — a package identity or a content digest, never a location, so a file an agent later writes into an admitted directory is not admitted with it — and the machine checks (§10) report each one it refuses or admits, and that admitted copies are visible to every session (§8.1). **Tools do not run without Curator** *(decided, wiki decisions.md:84)*: a tool binding to a skill Curator did not install is refused with the `curator add` fix, and the states that tool owns cannot be entered.

Today's `.roles` overlays are read on a documented legacy path. The project's `.roles` overlay is committed with the project: on a board in legacy mode it is read as today, and once the project has a version in force the role metadata it gives the default playbook's roles is part of the resolved configuration, so a change to it waits for a person (§2.4). The per-machine `~/.roles`, which lies outside any repository, is read only for today's built-in roles and only by the default playbook, with a warning, until PC2 retires it (§14 points 14 and 15, §15); any other hand-copied role skill or template is an unknown source.

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
  - A bound tool runs only when the project lists it in `policy.enable.tools`, by name or with `all`; `all` also admits tools that later versions of the admitted templates add. Under a `tools:<name>` capability, which a person grants once per tool, a process lists that tool and binds it to an audited provider without a touch (§2.4). A tool the project does not list is disabled, and the states it owns cannot be entered. There is no separate form for disabling a tool *(decided 2026-09-24, H3)*.
  - The kernel runs only commands exported by skills installed through Curator, never a string from a playbook, and always through the executor of §6.11.
- **Side effects are proposals.** Schedules (§6.6), `spawn.owner` actions, tool invocations and `fallback`/`degrade` relaxations (§9) that a package declares take effect only when the project enables them (`policy.enable`).
- **Admission.** The project's `extends` and `uses`, read at the pinned revision (§2.4), are its list of admitted playbooks. A template outside that list is never resolved, whatever is installed.

### 8.6 Finding and installing skills *(decided 2026-09-24)*

A process often needs a capability someone has already packaged: an integration with a supplier, a restaurant's CRM, a calendar. Agents should find and reuse such a skill rather than write their own.

- **Search.** An orchestrator or the process designer may search the Curator registry, whose skills pass Curator's security audit, and may read what each skill does and exports.
- **Install** *(the operator's words: an orchestrator finds and installs audited skills itself)*. By default an agent may install an audited skill from the Curator registry into the project: `policy.enable.skills` defaults to `audited`. A project may restrict it to a list of skill names or to `none`; a delegation of `installs` then lets named orchestrators install where the project's own setting does not (§4.2). An audit covers exact package identities: a skill is audited only at the identity the registry lists as audited, so a newer, unaudited release of an audited skill is an unknown source until it is audited. Installing changes `Skillfile.json` and its lock, committed like any other change, and every installation by an agent is recorded on the board with the skill's lock identity.
- **What waits for a person.** Capabilities decide (§2.4):
  - an audited role skill whose `data/role.json` contribution is unchanged, and an audited template-carrying skill whose template content is unchanged, only change instructions and take effect under the `skills` capability;
  - an audited new version of a bound tool provider takes effect under that tool's `tools:<name>` capability;
  - everything else waits for a person: a role skill whose contribution differs, a change of template content outside every capability, a provider without that capability.

  Meanwhile the kernel keeps running the version in force, and the executor keeps running the provider identity in force (§6.11). A skill's tools run only once `tools` binds them and `policy.enable.tools` lists them: a commit to `Playbook.json` that a person confirms, or that the tool's capability covers.
- **Unknown sources are refused by default**, the way a desktop operating system refuses unsigned software. A skill that is not in the Curator registry, not audited there at the identity being installed, or copied by hand (§8.4), installs only after the operator admits it with `curator sources add` by content — a package identity or a digest, never a location — a deliberate setting that shows a warning and is a widening entry of kind `skills`: confirmed by a person, at a level that cannot be lowered below `user` ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §10). No delegation covers it. The recommended path is to submit the skill for audit with `curator sources submit`, so that it enters the Curator registry and every user can install it safely.

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

1. **Measured.** For a local model, curator-inference-manager's benchmark of that engine and model on this machine, prompt processing and generation, repeated when either changes. For a cloud model, the median of recent observed runs on this machine and account: from PC1 task-board records `ttft` and `tg` for every run whose harness exposes them. A cloud API exposes no prompt-processing speed apart from the time to first token, so a cloud model's `pp` is never measured.
2. **Declared.** Typical ranges shipped with the model registry. A requirement is checked against the conservative end of the range: the lower end of `pp` and `tg`, the upper end of `ttft`. A cloud model's `pp` is declared or unknown.
3. **Unknown.** A requirement is unmet and `on_unmet` applies, with a refusal that names what to measure; a preference scores nothing.

`pp` and `ttft` depend on the length of the prompt, so both are measured at one reference prompt size, which PC1 fixes (§17.3).

**Independence**. `not_same_publisher_as: <role>` is evaluated against the provenance of the run that produced the evidence under review — the run of that role that recorded the outcome or the Change Request revision the reviewer reads — never against a binding. On a shared board that run may come from another teammate's machine, bindings may be candidate lists, and without an operator file there is no binding at all. Run manifests record the runtime and the model (LP D11), and the registry maps the model to its publisher. Work produced by a person satisfies the requirement; work whose producing run has no recorded provenance does not, and `on_unmet` applies. In a composition the value may be a qualified role name.

**Effort** *(decided: H4 keeps the floor)*. Effort is not a need: it lives in the operator file (§2.3). A project may set a floor per role (`policy.effort_floor`), and the operator chooses within it. Model rows own separate effort vocabularies that overlap only by coincidence (task-board `pkg/remoteconfig/reasoningeffort/reasoningeffort.go:98-100`; a local Qwen row may admit only `low`, `medium`, `high`, LP D2), so the floor is applied per row on the common order `minimal < low < medium < high < xhigh < max`:

- the effective floor is the lowest value the row admits at or above the floor;
- a row that accepts no reasoning effort at all (Claude Haiku) satisfies the floor vacuously; a project that wants to exclude such rows requires `reasoning: true`;
- a row with no value at or above the floor is unmet, and the role's `on_unmet` applies with the refusal `selection_effort_floor_unsatisfiable`, which names the row's vocabulary.

**Without the operator file** *(decided 2026-09-24, H2)*. The resolver chooses automatically, never wider than:

1. the rows the project admits (`policy.data`, below, model admission included), and never a metered runtime: without `models.toml` only `subscription` and `local` rows are candidates, whatever `policy.data.billing` lists, because only the operator's own file may spend the operator's money (§2.3). A runtime whose billing class cannot be determined — one harness may run on a subscription or on an API key, depending on how it is logged in — counts as metered;
2. the rows installed and logged in on this machine: the availability facts of roadmap M5b where they exist; before M5b, installed and bound runtimes, where a launch refused for authentication moves the choice to the next candidate;
3. the rows that satisfy the role's needs and the effort floor.

It prints the choice and the reason and never asks mid-run, because an orchestrator works without a person present. A command writes the choice into `models.toml` as `[bindings]` lines. How the first start guides an operator who has no file is open (§17.3). This amends agent-selection §7 — without an operator file, operator admission is replaced by project admission ∩ machine facts ∩ non-metered billing, never by the whole registry — and LP D9: with no operator config, the project's data policy narrows admission, and a file that exists but cannot be read still refuses, since a read failure is never headroom.

**Project data policy** *(answers the first run's F10)*. `policy.data` carries agent-selection's committed data policy (agent-selection §4): `publishers.deny`, `billing` (`subscription`, `local`, `metered`), `local_only` and `models`. It only narrows: listing `metered` admits metered rows for operators whose own file admits them, and never admits metered billing by itself.

**Model admission** *(decided 2026-09-24, P2)*. `policy.data.models` lets a project hard-restrict the models used on it, for example under a vendor's or a company's security policy:

```json
"data": { "models": { "allow": [ "claude/claude-opus-5-5", { "model": "codex/*", "effort": "medium..max" },
                                 { "model": "local/*", "windows": ["22:00-07:00"] } ],
                      "deny":  [ "codex/gpt-6-astra" ],
                      "roles": { "reviewer": { "allow": [ "claude/claude-opus-5-5" ] } } } }
```

- Patterns are `<runtime>/<model>`, `<runtime>/*` or `*/<model>`. An `allow` entry may be an object that also carries an effort range, the committed bound today's `spawn.ceilings` express per runtime, and time windows, so the ceilings and today's `spawn.schedule.windows` migrate without loss (§2.3, §14).
- A row is admitted when an `allow` pattern matches it, or no `allow` list is set, and no `deny` pattern matches it. When several `allow` entries match a row, the most specific decides: `<runtime>/<model>` before `<runtime>/*` before `*/<model>`; the validator refuses two entries of the same specificity that match one row (§10).
- `windows` restricts an entry to times of day, in UTC (`"HH:MM-HH:MM"`, the end excluded, crossing midnight allowed), as today's schedule windows are; outside its windows the entry admits nothing. The operator's `[windows]` narrow further, never wider.
- `roles.<role>` restricts one role further: its `allow` and `deny` apply on top of the project's lists, so a role is admitted only what both admit. Today's per-role ceilings migrate into it.
- A row's admitted efforts are the intersection of the deciding entry's range, the operator's `effort` and the project's floor; an empty intersection is unmet, and the role's `on_unmet` applies. A row that accepts no reasoning effort at all meets an entry's effort range vacuously, as it meets the floor (above).
- Selection admits project admission (`publishers`, `billing`, `local_only`, `models`) ∩ the operator's `[allow]` ∩ machine facts ∩ needs. Without an operator file, the automatic choice stays inside the project's list.
- A list of concrete models is portable only among teammates who have those models: the validator reports, as information, the roles that only listed models can serve ("teammates without codex or claude cannot run role developer", §10). This amends LP D1, which kept the ceilings in `task-board.config.json`, and agent-selection §4 and §7. `Playbook.json` supersedes agent-selection's committed project layer for role needs, admission (`extends`/`uses` replace `playbooks.allowed`), data policy and waivers; a waiver is a reviewed change to the project's role entry, with the reason in its `description`. Team runtime names and check sets live in `task-board.config.json` (§2.1, §6.12); machine limits live in the operator's `runtimes.toml` (§2.3).

**When needs cannot be met**, `on_unmet` decides:

| `on_unmet` | Behaviour |
| --- | --- |
| `block` (default for `require`) | refuse, naming the layer that emptied the candidate set |
| `fallback` | try the ordered `fallback` needs; active only when the project enables relaxations |
| `ask` | ask a person for a temporary exception (below) |
| `degrade` | run on the best available through a statically declared `degraded_state` (a review state with its own owner); active only when the project enables relaxations; refused for independence requirements, because more reviews from the same publisher cannot restore independence |

**The degraded path**. A run selected under `degrade` marks its element degraded. The next exit from the state that run worked in is redirected into `degraded_state`, recording the original target; `degraded_state` has an owner other than the degraded role and declared exits back into the workflow, and leaving it clears the mark. The validator checks that `degraded_state` exists in each workflow where the role owns a state and that it reaches the original targets.

**Who relaxes a hard requirement** *(decided)*:

- Permanently, for the whole team: only a change to the project's `Playbook.json`, which no capability covers, so it takes effect once it is committed and a person has confirmed it (§2.4).
- Permanently, for one operator's own runs: an open-ended standing answer (below) *(decided 2026-09-24)*. This amends agent-selection §8 ("only the project can waive") by adding it and the temporary paths below.
- Temporarily: through `ask`, an operator approves an exception for **their own runs only**, scoped to an element subtree (`element`, the nearest ancestor of a type, or `project`) and time-limited. The role's `needs.approval` sets the scope and the expiry; for a role without it, the exception covers the element that asked and expires after 30 days. The exception is a kernel record, `exception`, available in every playbook rather than a type a playbook must declare, and it is bound to the approving principal; it never reaches a teammate's runs. Until runs are bound to principals (waggle worker certificates), "own runs only" is a guardrail. `policy.signatures` may require a signature for it (`relaxations`).
- An exception that expires while a run is working lets that run finish; new runs refuse. An exception relaxes only project requirements, never the operator's own `[allow]` *(decided 2026-09-24)*.
- **Standing answers** *(decided 2026-09-24)*. An operator may answer an `ask` in advance in `models.toml` (`[answers]`, §2.3), so autonomous work does not stop for a question the operator has already answered. A standing answer is a widening entry: it takes effect only once the operator has confirmed it ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §10, P3). When `ask` fires and a confirmed standing answer matches the role, the requirement term and the project, the kernel records an `exception` marked as a standing answer from the operator file, bound to that operator, and the run proceeds; the team sees the exception on the board. The exception always takes the role's approval scope and expiry; the answer's `until` — an end date, or `open` written explicitly — bounds only when the answer itself applies.
- **Open-ended standing answers** *(decided 2026-09-24)*. An answer with `until = "open"` relaxes the requirement at every future `ask` of its role, for this operator's own runs, and each application appears on the board as an exception. A project forbids standing answers, open-ended ones included, with `policy.enable.standing_answers: false` (the key is the author's, §17.2 row 25). A standing answer applies to the operator's own runs only and never widens `[allow]`. It never applies where `policy.signatures` covers `relaxations`: there every relaxation must be a signed act the board verifies, and a standing answer is an entry in a file on one machine, whose confirmation only that machine's resolver can check. A delegated relaxation differs in exactly that: its grant is a board record created with a person's confirmation and, under a signature policy, signed by the grantor, and the delegate signs each act within it, so the board verifies the chain (§4.2).
- **Periodic review** *(decided 2026-09-24, "с подтверждением": the review binds)*. Once a month, as the operator decided — that `[review] answers` in `models.toml` may set another period is the author's (§2.3) — the main loop, and until it exists a board notice that points to `curator models review` (in an apiary, `keeper review`), lists every standing answer whose end lies beyond the next review, the open-ended ones included, and how often each was applied since the last review. The operator re-confirms each one, which is the same confirmation as a widening entry (§2.3), or removes it; the answer and the date are recorded. An answer not re-confirmed within one period after its review stops applying until the operator re-confirms it; the board shows it as lapsed.
- **Removing or lapsing an answer** ends the exceptions it created; a run already working under one finishes.
- **Through a delegation** *(decided 2026-09-24)*. Where `policy.delegation` allows `relaxations`, a session the grant covers may approve an `ask` exception on its grantor's behalf, for the grantor's own runs and within the delegation (§4.2). Such an approval answers one `ask`. The exception it creates takes the intersection of the grant's scope and the role's approval scope, with the role's expiry, and the rule's and the grant's ceilings are checked each time the exception applies, so it never reaches an element outside the grant or over a ceiling. The rule or the grant may limit it to the asking roles and the requirement terms it names (`relax`, §4.2); it never becomes a standing answer, which only the operator writes in their own file (the author's resolution, §24), and it ends when the grant ends (§4.2).

Example: the reviewer requires a publisher different from the developer's; a teammate has only one publisher; `ask` fires; the teammate approves "same-publisher review under this epic, 30 days" for their own runs, or has answered it in advance in `[answers]`. Every selection records which layer supplied what and whether a fallback, an approval, a standing answer or a degradation applied.

**Per-element needs** *(future, after PC1)*. An element may add preferences and extra skills for its own runs and narrow its role's requirements, never relax them, within the operator's files.

## 10. Static validation *(decided)*

The validator is a library and a CLI in this module, independent of task-board. Its command is `curator playbook check`, through the Curator provider `curator-playbook` that this repository ships; `curator playbook explain` prints the authority report below for a working tree, and `curator playbook bind` makes the two edits of §2.1. There is no bare `playbook` command, because command names are global shims with one owner each (§8.5). task-board runs it when it loads a playbook; a visual constructor runs it on every edit. It resolves `extends` and `uses` first. Checks are split so that CI and every teammate get the same structural verdict:

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
| parameters | `transition review → development: limit.max $rework_limt names no declared parameter`; `params block_limit: PT15M is not a limit`; `project sets params.review_limit, which no resolved playbook declares` |
| unbounded cycles | `workflow dev: cycle development → review → accepted → development passes no bounded limit and no human-owned state` (warning; an error when the project's properties require bounds, §7.3) |
| roster and quorum | `people cafe-managers: op:dave@cafe is not in the roster of this configuration`; `approvals role manager min 3: group cafe-managers has 2 principals` (patterns such as `op:*@acme` are counted against the roster of the same configuration; at run time the kernel reads the version in force, §2.4) |
| signer groups | `event supplier.confirmed: signer group supplier-bridge is not bound in services` |
| degraded path | `role reviewer: on_unmet degrade names degraded_state second-review, which workflow dev lacks` |
| project-only keys | `template dev-cycle sets policy.enable: ignored` (warning) |
| tools enabled | `tool supplier.send-order is bound but not enabled` (warning) |
| field conditions | `workflow dev: guard field names review, which dev-task does not declare`; `guard field on amount: gt needs a number, money, date or date-time field` |
| check sets | `workflow dev: delivery.checks-passed names set handoff, which task-board.config.json does not define`; the same for `delivery.integrate`'s `checks` and `delivery.integration-evidence`'s `set` |
| ranges | `role reviewer: need tg has min 50 above max 10` |
| kernel-written fields | `workflow dev, state blocked: record_source names blocked_from, which dev-task does not declare`; `workflow dev, state blocked: record_source field blocked_from does not admit source state qa of an entry` |
| work in progress | `policy.wip.per_role: reviewer2 is not a role`; `policy.wip.per_state: dev.qa is not a state` |
| unsigned human acts | `workflow purchase has human-owned states and the project sets no policy.signatures: human acts are a guardrail only until signed forms arrive with the waggle amendment (CM2)` (warning) |
| signable forms | `policy.signatures.require_for approvals: board approvals have no signable form before the waggle amendment (CM2); the states that need them cannot be left until then`; `policy.signatures covers delegations whether listed or not: no grant can be signed before the waggle amendment (CM2)` (warnings) |
| model admission | `policy.data.models.allow: codex/gpt-6-luna effort high..medium is an empty range`; `policy.data.models: allow codex/gpt-6-astra is also denied`; `policy.data.models.allow: codex/* and codex/* both match codex/gpt-6-luna at the same specificity`; `policy.data.models.roles: reviewer2 is not a role`; `roles developer, reviewer can run only on models policy.data.models admits: teammates without them cannot run these roles` (information) |
| delegation | `policy.delegation: role manager in approvals is not a human role`; `delegate cafe-orchestrator does not have capability orchestrator`; `state purchase.draft is not human-owned`; `state approval must be named <workflow>.<state>`; `ceiling field amount is not declared by inventory-check, whose approvals by manager the rule also covers`; `relax.roles names reviewer2, which is not a role`; `allow rule for purchase.escalated is shadowed by never` (information) |
| confirmers | `policy.confirmers names owners, which people does not define` |

The guard-bypass rule: an `approvals` guard or a `risk` marker on one entry into a state must appear on every other entry into it that a person or an agent fires, and on every automatic entry that a timer (`after`) or a kernel state event (`children.changed`, `blockers.terminal`) triggers, or the guard must sit in the state's `requires`. The automatic entries joined the rule in v3.5 (§25): such an entry has no actor whose authority could stand in for the guard. Automatic entries triggered by `tool.passed`, `tool.failed`, `process.returned`, a delivery event or a declared signed external event carry their own authority — a tool the project enabled, a result signed by the receiving board, the delivery invariants, a signer group the project bound — and stay outside the rule. One exception makes thresholds expressible: an entry without the guard or the marker is admitted when its `field` condition admits no value that the `field` condition of any guarded or marked entry admits, on the same field, so `amount lte 500` may skip the approval and the `spending` marker that `amount gt 500` carries; a guarded or marked entry without a `field` condition covers every value. The exception is sound at run time only because fields a guard reads are authority-bearing (§5.2), approvals bind the digest of every declared field (§6.4) and a tool acts on the values recorded at entry (§6.11). It trusts the value the element carries: the orchestrator that writes the amount decides the route, as the project's threshold allows. A project that must stop an agent from splitting spending under a threshold keeps the approval on every entry, or delegates it with a budget (§4.2). Verdict and `process.result` guards select an exit of their source state and are outside the rule; `blockers.clear`, `estimate` and `assignee` are hygiene guards that rework routes may omit on purpose; the delivery routes are covered by the invariant checks.

**Machine checks** (depend on what is installed and on the operator): skills installed through Curator and audited at the exact identity the lock records; each template tool binding declared by the orchestrator's manifest in runtime mode narrowed to the bound command; each command shim owned by the bound skill; operator bindings able to satisfy each role's needs and effort floor; each skill `policy.enable.skills` names listed as audited by the Curator registry; no installed skill from an unknown source, hand-copied role skills and templates included, without its confirmed setting (§8.4, §8.6). They report typed "unbound on this machine" refusals and never change the portable verdict.

**Authority report** (at every load, from the board, in `task-board config status` and in `task-board trust confirm`). The kernel compares the proposed version with the version in force (§2.4) and lists every difference, section by section: templates, role contributions and tool providers whose identity or content changed, check sets, roster lines and enrollments, roles, entities, workflows, policy. It classifies each difference by the capability kind that covers it, if any, and names the grant in force (§2.4). A difference that no grant covers waits for a person. It then gives the verdict for the whole version: it takes effect on commit, or it waits. A version that fails a portable check never takes effect, and the report says so. Among the differences that wait it marks first those that move work past a person, a tool or a guard:
- an automatic transition added or changed, above all one that leaves a human- or tool-owned state, or enters a state that the other entries reach only through a person, a tool, an `approvals` guard or a `risk` marker;
- a transition or state added, removed or changed on a route to or from a human-, tool- or process-owned state or a verification state;
- a human-owned or verification state removed or re-owned;
- an `approvals` guard, a `risk` marker, a check set or a delivery guard removed or emptied;
- a `field` condition on a route to a person changed, or a field that became authority-bearing (§5.2);
- a tool binding, a tool provider's identity or `policy.enable` changed;
- a role's `who`, `kind`, `skill`, contribution, `needs` other than `prefer`, or `on_unmet` changed;
- a roster line, a revocation or an enrollment changed.

It also lists the unused approvals and the grants the proposed version would void (§4.2, §6.4), and every version that took effect under a capability or in yolo mode since the last confirmation. The marks inform the confirmer, and a change-analyst role may explain them. They decide nothing, because capability coverage alone decides what waits. `curator playbook explain` prints the same report for a working tree against the version in force, so a change can be reviewed before it is committed.

A throwaway prototype of a subset of the portable checks (not committed) was run over [`../examples/`](../examples/) at each revision since v2.1; §19–§25 record what each revision added. At v4.0 it covers:
- states, initial states, owners, reachability, terminal reachability and exits;
- the root's capability, nesting and joins;
- owner-kind exits, and determinism with the four-part transition key;
- loop-limit targets and resolutions;
- the guard-bypass rule with field conditions and automatic entries;
- `field` guards and kernel-written fields;
- check sets, ranges and work-in-progress keys;
- tool enablement and both signature warnings;
- model admission, delegation and confirmers.

The delivery invariants, signer groups, the roster, the degraded path, placement and slot contracts rest on the v2.1 prototype, whose results stand.

Every template and both composed projects pass with no error and seven intended notices:
- the cafe project's four `require_for` categories have no signable form before CM2;
- the scrum project, a PC1 demo, has human-owned states without a signature policy in both of its playbooks;
- as information, the scrum project's model admission limits every role to the listed models.

Seeded defects are each reported: an automatic `draft → sending` in the cafe, on a timer or on `children.changed` (guard bypass); an unknown `policy.confirmers` group; and a `relax.roles` that names no role.

The second throwaway prototype of v4.0, which classified changes by an allowlist with a path check, is withdrawn together with the allowlist (§27). Capability coverage compares sections, so it needs no path reasoning. The seventh refuter run's probes change workflow structure (a manual twin of a signed event, a `limit` redirect, a new state with `record_source`, a re-owned state) or add a limit on a human-owned state, so under capabilities they all wait for a person.

All examples validate against the schema. The schema refuses URLs and shell syntax in hook arguments, schedule fields and event signers; argument names that would select a program, an environment or an endpoint; package pins that are not Curator identities; program strings as tool commands; `effort` and the old `capability` need; non-operator principals in `people`; tools posing as reviewers; and action objects inside guards. Since v3 it also refuses the `null` tool form, `context_window_min`, a boolean `policy.enable.tools` or `policy.enable.schedules`, empty ranges, and `field` or `delivery.checks-passed` guards without their arguments. Since v3.1 it refuses `delivery.integrate` without `checks`, `delivery.integration-evidence` without `set`, the bare names of all three, and an unknown category in a per-category `user_verified`. Since v3.2 it refuses a model pattern that is not `<runtime>/<model>`, `<runtime>/*` or `*/<model>`, an effort range outside the common order, a delegation rule without acts, approvals without a `role`, transitions without `states`, a `role` or `states` without the act that uses it, an `allow` rule without `to`, and a `never` rule with ceilings. Since v3.3 it refuses a bare state name in a delegation rule, `states` on a rule that covers neither approvals nor transitions, `self_approval` on a `never` rule, a window that is not `HH:MM-HH:MM`, and an empty or unknown per-role model restriction. Since v3.4 it admits `installs` among delegation acts and the `policy.enable` keys `standing_answers` and `skills`, and refuses any other value for them. Since v3.5 it admits `none` for `policy.enable.skills`, whose default is `audited`, the key `policy.confirmers`, and `relax` with roles and need terms on a rule that covers relaxations; it refuses `relax` on any other rule, `self_approval` on a rule without approvals, and ceilings on a rule that covers installs only. Since v4.0 it refuses a delegation rule that mixes `installs` with other acts, and admits the signature category `installs`. Since v4.1 it admits `policy.delivery.run_write_boundary` (`warn` or `refuse`). It cannot compare a range's bounds, which the validator does, and it cannot make the other hooks' arguments exact: their schemas come with the provider modules in PC1 (§17.3).

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
| routes on field values | dev-cycle (review levels; resuming a blocked element in the state it was blocked from, recorded by the kernel) |
| limits on work in progress | the scrum project (`policy.wip` over elements); the cafe project (a limit on a human-owned state) |

## 12. The refuter role and decision memory

A draft specification is challenged before it is approved *(decided)*:

- **Where.** A `challenge` state after a draft (dev-cycle's `spec` workflow and the epic's `hld`), owned by a `refuter` role skill.
- **Input.** The draft plus board memory: `decision` elements (statement, scope, status, links), the specifications in force and their invariants.
- **Method: proof by contradiction.** Assume the draft is implemented as written and derive consequences; look for conflicts with recorded decisions, internal contradictions (two acceptance criteria that cannot hold together), unstated assumptions and guards no role can satisfy.
- **Output: a ledger verdict with a refutation detail** *(the spelling follows §6.3)*. `accepted` with `no-contradiction` and the list of decisions checked, so the pass is auditable; `changes_requested` with `contradiction` and pairs of claim and decision; `stop_the_line` with `human-decision-needed` when only a person can choose. The same spelling is used in the spec, the examples and the role.
- **Transitions.** `contradiction` → back to the draft; `human-decision-needed` → `decision-needed`, owned by a person, who records a new `decision`; `no-contradiction` → approved.

Agents are good at finding contradictions; what they lack is memory of what was decided. The board supplies it. The refuter's runs on this document are in [`../reviews/`](../reviews/): [`2026-09-24-refuter-v0.md`](../reviews/2026-09-24-refuter-v0.md) on v1, [`2026-09-24-refuter-v1.md`](../reviews/2026-09-24-refuter-v1.md) on v2, [`2026-09-24-refuter-v2.md`](../reviews/2026-09-24-refuter-v2.md) on v3, [`2026-09-24-refuter-v3.md`](../reviews/2026-09-24-refuter-v3.md) on v3.1 and v3.2, [`2026-09-24-refuter-v4.md`](../reviews/2026-09-24-refuter-v4.md) on v3.3 and v3.4, [`2026-09-24-refuter-v5.md`](../reviews/2026-09-24-refuter-v5.md) on v3.5, answered by v4.0 and [`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md), and [`2026-09-24-refuter-v6.md`](../reviews/2026-09-24-refuter-v6.md) on the core of v4.0, answered by v4.1.

## 13. Examples

| File | Shows |
| --- | --- |
| [`default.playbook.json`](../examples/default.playbook.json) | today's task-board behaviour (§14) |
| [`dev-cycle.playbook.json`](../examples/dev-cycle.playbook.json) | delivery with the delivery invariants, inline and Change Request routes, check sets `handoff` and `landing` for Change Request work, review levels routed by `field`, `blocked` with a block kind, a kernel-recorded source state and a repeat limit, template parameters for its loop limits (`rework_limit` and `rollback_limit` unbounded by default, `block_limit` 3, §7.3), `stop_the_line`, the refuter's verdicts, decisions, needs and `ask` |
| [`bug-hunt.playbook.json`](../examples/bug-hunt.playbook.json) | proving findings and offering them to the fixing process through slots |
| [`scrum/scrum.playbook.json`](../examples/scrum/scrum.playbook.json), [`scrum/project.Playbook.json`](../examples/scrum/project.Playbook.json), [`scrum/task-board.config.json`](../examples/scrum/task-board.config.json) | a cadence layer composed over dev-cycle; membership links on the sprint; delegated estimate fields; planning poker; capacity; work-in-progress limits over elements; a project value for dev-cycle's rework limit (`params`, §7.3); project-only policy, including model admission that restricts the project to named models and its reviewer to one (§9); the local check sets `handoff` and `landing` in the project's `task-board.config.json`. As a PC1 demo it sets no signature policy and runs no hosted set (§4.2, §15) |
| [`cafe/cafe.playbook.json`](../examples/cafe/cafe.playbook.json), [`cafe/bookkeeping.playbook.json`](../examples/cafe/bookkeeping.playbook.json), [`cafe/project.Playbook.json`](../examples/cafe/project.Playbook.json) | fields, humans, a tool reached through the orchestrator's runtime dependency, timers, a named schedule, signed events and their signer groups, a join, a hand-off with a returned result, privacy, a work-in-progress limit on a human-owned state, and the project file that enables the tool and the schedule, lets the cafe managers delegate approvals of purchase orders up to 1,000 EUR to their own orchestrator sessions, including orders those sessions' buyer runs drafted in `draft`, where they may write the `amount` the ceiling reads (`self_approval`, §4.2, §5.2), lets agents install audited skills and forbids standing answers (§8.6, §9), names the cafe managers as the confirmers of process changes (§2.4), and sets the signature policy of the post-CM2 demo |
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
| roles are open-ended: any installed role can be spawned | `tools/board-cli/internal/spawn/role.go:83-93, 303-335` | owners `{ one_of: { capability: [...] } }` over the role aliases the version in force binds; in legacy mode the committed `.roles` overlay supplies them (§6.1) |
| inline work without a Change Request is reviewed and finishes through the acknowledged done route | `consistency.go:105-141` | `* → reviewing` unguarded by delivery; inline `* → done` |
| integrating is entered only through `accept_cr` | `pkg/board/consistency.go:166-167` | `via: accept_cr`, `locked` |
| integrating is left to done through integration, external integration or a restore, and to to-dev through a rollback (landed rework, invalidation, converge) | `consistency.go:169-172`; `tools/board-cli/internal/integration/start_landed_rework.go:67`, `invalidate.go:117-118`, `converge.go:426-427`, `restore_done.go:85` | `via` transitions with `delivery.*-authorized` guards |
| `worktree integrate` accepts only the story-final leaf as the story's integration unit; the orchestrator integrates the story | `pkg/board/version_control.go:151-156` | `integrating` owned by the orchestrator, no tool invocation on entry |
| done is refused while a Change Request exists; inline work passes | `consistency.go:128-141, 158` | guard `delivery.cr-absent` |
| commit acknowledgement for the enforced types when policy confirms | `version_control.go:89-123, 277-279` | guard `delivery.commit-acknowledged`, `policy.delivery.version_control` |
| rejection routes to to-dev and records changes requested | `tools/board-cli/cmd/change_request.go:753, 825, 856` | `reviewing → to-dev via reject_cr` with `record.verdict` `changes_requested`, beside the manual route |
| reviewer verdicts are `accepted`, `changes_requested`, `stop_the_line`; a reviewer's Stop-The-Line parks its element at blocked with a new evidence packet | `pkg/board/goal.go:40-49`; `tools/board-cli/internal/spawnruntime/goal_acceptance.go:51-96, 255-256`; `spawnruntime/changerequest.go:403-405` | `reviewing → blocked` with `outcome.present` for the run's own element and `record.verdict` `stop_the_line`; the ledger gains `rejected` in PC1 |
| linked blocked items, leaves and containers, release to backlog when their last blocker finishes in the current mutation; an element without links is never released, and one whose blockers were done before is reported stale | `pkg/board/blocked_release.go:8-24, 76-87` | `on: { event: blockers.terminal }` on both workflows, an edge event with today's semantics (§6.9) |
| orchestrator (and coordinator) has no lifecycle statuses | `pkg/board/goal_role.go:10-11, 25-33` | `root` |
| review policy defaults to required | `types.go:106-133` | `policy.review` and the field `review` |
| task class | `types.go:136-141` | field `task_class` |
| rework loop detection: threshold `max(3, 2 × median)` | `internal/changerequest/loopdetect/loopdetect.go:114, 126-129` | `policy.delivery.loop_detection` (`floor` 3, `factor` 2) |
| the landing-gate suite `spawn.worktree_isolation.validation.commands`: run by the producer at Change Request build with evidence bound to the candidate tree, re-run by integration when that binding is missing or broken; an absent block refuses integration, an empty list runs nothing | task-board `references/task-board-config.md`, "The landing-gate suite"; `tools/board-cli/internal/integration/validation.go:422-433` | a local `landing` check set named by the `integrate` route's `delivery.integration-evidence` (§6.12); an undefined set refuses integration with `checks_not_configured` |
| child runners per control root, `spawn.max_parallel` (default 20) | `references/task-board-config.md`, "Spawn policy"; `pkg/remoteconfig/spawn.go:13-15` | machine capacity in `~/.curator/runtimes.toml` (§2.3); the default sets no `policy.wip` |
| concurrent heavyweight suites per OS account, `spawn.validation.max_concurrent_suites` (default 2) | same | machine capacity in `~/.curator/runtimes.toml` (§2.3) |
| per-runtime model admission and effort bounds, `spawn.ceilings`, with per-role replacements (`roles.<role>`) | `references/task-board-config.md`, "`spawn-policy-v3` / `spawn-policy-v4`" | `policy.data.models` allow patterns with effort ranges, and `policy.data.models.roles.<role>` (§9); the default sets none |
| the allowed or preferred runtime set, `spawn.preferred_agentic_system`, `exclusive` or `mixed` | `references/task-board-config.md`, "Spawn policy" | allow patterns `<runtime>/*` in `policy.data.models`; the `mixed` order becomes the operator's candidate order in `[bindings]` (§2.3) |
| `spawn.schedule.windows`, which replace the runtime set by time of day | same | allow entries with `windows` in `policy.data.models` (§9); the operator's `[windows]` narrow them |
| per-role defaults, `workload_classes` | `references/task-board-config.md`, "Per-role model preferences", "Workload-class recommendations" | the operator's `[bindings]` in `models.toml` (§2.3) |
| `resolution_model`, `adjustment_confirmation` | `references/task-board-config.md`, "`spawn-policy-v3` / `spawn-policy-v4`" | the operator's candidate order: `none` resolves to the first admitted candidate and records it, `required` refuses naming the candidates (§2.3) |
| Codex `fast_mode` (the priority tier) | same | an option of the operator's `[bindings]` entry (§2.3) |
| `spawn.ceilings.<runtime>.supervision`, with role overrides | `references/task-board-config.md`, "Supervision (minutes)" | `spawn.supervision.<runtime>` in `task-board.config.json`, unchanged in meaning (§2.1) |
| the run write boundary, `spawn.worktree_isolation.run_write_boundary` (`warn` or `refuse`) | `references/task-board-config.md`, "Story worktree isolation" | `policy.delivery.run_write_boundary` in `Playbook.json`; the config's key is read for one release as part of the resolved configuration (§2.1, §6.8) |
| `integration_base_branch`, `board_repository` | same | stay in `task-board.config.json`, read from the version in force, and must name the board's recorded default branch (§2.1, §2.4) |

Accepted differences and open points of the mapping:

1. The aggregation order of container statuses stays kernel code; the configuration only declares `derived_status`.
2. `one_of … by assignment.role` needs a structured role field; today the role is recorded only inside the assignee display string (`tools/board-cli/cmd/spawn.go:3856`). PC1 adds the field; until then, and after a move into a state that does not admit the recorded role, the root orchestrator owns the element.
3. `closed` carries no resolution today.
4. Dependency links (`blocked_by`) are kernel data; the configuration refers to them only through `blockers.clear` and `blockers.terminal`.
5. Loop detection counts rework across Change Request revisions; it stays delivery policy, distinct from a transition `limit`.
6. `defer_cr` and `withdraw_cr` are not mapped yet.
7. Review levels *(decided 2026-09-24)*: the field `review` and the `field` guard express them as routes, as dev-cycle shows. In the default, whose transitions are permissive (`*`), the field changes no route. Two effects stay delivery and ledger behaviour keyed on the same field: the goal ledger accepts a `none` item on the producer's outcome alone (`tools/board-cli/internal/spawnruntime/goal_acceptance.go:167-170`), a shortcut that applies only where the workflow declares no `field` route on `review` (§6.3), and the close-landed method `research_outcome_only` admits a research element without a Change Request only with `review=none` and `task_class=research` (`tools/board-cli/internal/integration/landed_methods.go:553-565`). PC1 maps that method onto a `via` command guarded by `field`. `light` differs from `required` today only in the ledger's order of review strength (`pkg/board/goal_mutation.go:268-279`); both route to review.
8. The version-control policy lives in `task-board.config.json` today with authority class `project_policy` (`pkg/remoteconfig/authority_schema.go:117`). *Decided (2026-09-24):* `Playbook.json` owns it (`policy.delivery.version_control`), and the config's `version_control` keys are read for one release with a warning. The enforced types stay fixed by the delivery module (`version_control.go:18-19`).
9. The linkless `blocked` state is owned by the orchestrator in the default, because today's kernel does not restrict who unblocks; stricter playbooks give it a human owner.
10. Evidence freshness (§6.3) is today's rule for reviewer verdicts only; PC1 extends it to approvals and outcomes, which changes nothing the default declares.
11. The default keeps today's guard `delivery.integration-evidence` on its `integrate` route and names the set explicitly (`"set": "landing"`); that evidence is the result of the `landing` check set (§6.12). Today's key `spawn.worktree_isolation.validation.commands` maps onto `checks.landing` with `where: local` until the config is migrated, and a project with neither refuses integration, as today. One difference is accepted: today the producer's own `task-board` process runs the suite in the environment it inherits (`validation.go:422-433`); from PC1 the kernel's executor runs it in a clean checkout of the exact tree with a sanitized environment (§6.12).
12. Today's runner limit `spawn.max_parallel` (default 20 per control root) becomes machine capacity in `runtimes.toml`, counted per machine rather than per checkout. For one release a value committed in today's config is read as the machine's capacity where `runtimes.toml` does not set it, with a warning, and the same holds for `spawn.validation.max_concurrent_suites` (default 2). The default sets no `policy.wip`, so today's boards keep no process limit beyond machine capacity.
13. Today's `spawn.ceilings`, runtime set and schedule windows sit in `task-board.config.json`, which the kernel reads from the working tree today. In a project with a version in force they are part of the resolved configuration, read at the proposed head (§2.4), and stay in force until a confirmed version replaces them with `policy.data.models`; the default sets no model admission, so a project that pinned models, runtimes or windows carries them over with the migration command (§2.3), and nothing it admits today is lost. The same holds for the legacy `version_control` keys.
14. Role definitions: the project's committed `.roles` overlay is part of the resolved configuration once the project has a version in force, so a change to it then waits for a person; the per-machine `~/.roles` is read only for today's built-in roles and only by the default playbook, with a warning, until PC2 retires it (§8.4). Every other hand-copied role skill or template is an unknown source.
15. **Legacy mode.** A board without a version in force, today's boards included, runs the built-in default, which grants nothing beyond today's board and needs no confirmation (§2.4). In legacy mode the kernel reads, as today and from the working tree, `task-board.config.json` with its landing-gate suite, its legacy model and `version_control` keys and its machine limits, and the committed `.roles` overlay. Legacy mode is the one place where anything is read from the working tree; it ends when the board's anchor operator confirms the project's first version, and the board then behaves identically on the default playbook, or every difference is documented here.

## 15. Integration plan

| Step | Delivers |
| --- | --- |
| PC0 | this draft, the schema, the examples; seven refuter runs, each answered (v2, v2.1, v3.1, v3.3, v3.5, v4.0, v4.1); the operator's decisions (v3, v3.2, v3.4, and those of 2026-09-25 in v4.0 and v4.1), item 13 included (capabilities); the split into this core and [`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) (v4.0), which v4.1 moves to relux-works/curator-trust; a check of the changes since v4.1, after which the core is frozen. trust.md continues its own rounds |
| PC1 | the module: schema, parser, resolver (`extends`, `uses`, slots, parameters §7.3), the validator `curator playbook check` with portable and machine checks, `explain` and `bind`; a thin seam in task-board that loads the proposed version from the head of the default branch and runs the version in force (§2.4), and `default.playbook.json` with identical behaviour or documented differences, with legacy mode for boards without a version in force (§14 point 15); new types from `Playbook.json` on the generic engine with the kernel hooks of §6.3, the `field` guard included, with guard-read fields protected from the runs they gate and field defaults written at creation (§5.2); `record_source` and `clear_on_exit` (§6.1); approvals bound to the digest of the element's declared fields and to the parts of the process they depend on (§6.4), with the fields a pending delegated approval depends on frozen; the structured `assignment.role`; `rejected` in the goal ledger; `policy.wip` over elements (§6.8); the version-control policy read from `Playbook.json`, with the config's keys read for one release; the kernel's executor, one component that runs local check sets from the definition in force over today's landing-gate suite in a clean checkout with a sanitized environment (§6.11, §6.12), runs the provider identity in force from Curator's store and holds an invocation whose identity left the store, and that tick reuses in PC3; the legacy operator layer read from today's config until `models.toml` exists, and model admission from `policy.data.models` with today's ceilings, runtime set and windows read as project admission until migrated (§2.3, §9); `spawn.supervision` in `task-board.config.json`; the resolved configuration of §2.4 — templates from the project scope and committed lock, role contributions, tool providers, check sets, roster — its configuration digest, capability coverage (§2.4) with capability grants, their monthly review and sealing, and yolo mode as board records ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §16, §17), the version in force derived from history and stored, batched confirmation through `task-board trust confirm` (with `task-board playbook confirm` as its alias for one release), `task-board config status`, `task-board config propose` for projects without git, and the authority report at every load (§2.4, §10); **the trust component as a guardrail stub**: `task-board trust init` records the anchor operator's principal without keys, and a confirmation, a capability grant, a delegation grant or a narrowing act is the operator's command outside any run, unsigned, carrying that principal and recorded with the fields its signed form will carry; versions take effect only on a single-host local board ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §13); `policy.delivery.run_write_boundary`, with the config's key read for one release; the built-in demo templates `dev-cycle`, `bug-hunt` and `scrum` with their role skills, from this repository (§8.2); delegation records and delegated acts without signatures, counted only from the grantor's own sessions as the session host marks them, with budgets on single-host boards and the grantor notified by a board notice (§4.2) — the marks naming who started a session and its role alias are a follow-up to the session host's as-is extraction, and until they land the kernel records grants and counts no delegated act; per-run recording of `ttft` and `tg` where the harness exposes them, and the reference prompt size for speed facts (§9); `policy.enable.standing_answers` and the board notice that starts the monthly review of standing answers, which lapses an answer not re-confirmed within a period (§9); `policy.enable.skills`, `audited` by default, and installation of audited skills by agents, recorded on the board, with an installation that changes the resolved configuration waiting for a person (§8.6); the legacy reads of today's `.roles` overlays (§8.4, §14); the constructor-ready schema (`ui` blocks, lossless round trip). Because the demos follow PC1 directly *(decided)*, PC1 also carries **the minimal delivery seams over today's commands**: `delivery.cr-published`, `delivery.review-ready`, `delivery.cr-accepted`, `delivery.cr-absent`, `delivery.commit-acknowledged` and the evidence and authorization guards mapped onto today's authorization options; `delivery.integrate` mapped onto the orchestrator's `worktree integrate`; and `delivery.integrated`, `delivery.integration-failed` and `delivery.rework-started` emitted when today's integration transaction, its failure, and landed rework, invalidation or convergence complete. It also carries **the local hand-off on one board** (§6.5). |
| Demos (right after PC1) | `dev-cycle`; `bug-hunt` → `dev-cycle`; `scrum` over `dev-cycle` — all on one board *(decided 2026-09-24)*. The cafe with bookkeeping follows PC3 and the waggle amendment |
| PC2 | role skills and the packages `curator-playbook-dev-cycle`, `curator-playbook-bug-hunt` and `curator-playbook-scrum` on Skillfile schema 2, published and audited in the Curator registry so that they replace PC1's built-ins (§8.2), delivered to their runs through per-role profiles once the launch-profiles work ships them (§8.1), with the description stopgap until then; the JSON Schema of `data/role.json`; role aliases bind installed role skills, and an installed role skill becomes a role only through an alias (§6.1); in legacy mode the committed `.roles` overlay supplies the aliases, and `~/.roles` is read for one release with a warning; `setup.sh` stops copying `~/.roles` (this is roadmap milestone M1) |
| PC3 | the delivery module proper: the PC1 seams become a module whose invariants it checks itself; hosted check sets, `mirror` evidence and the landing override (§6.12); `task-board tick` and its user-level system timer (`tick --install-timer`), calling PC1's executor for tools, timers and schedules (§6.9, §6.11); **real trust** ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §13): presence-bound keys and their enrollment, the web of trust, the trust anchor with keys, confirmations, capability grants and a person's narrowing acts as signed board records that every machine verifies against the roster of the version in force — no CM2 needed — the workstation client for headless nodes, and the confirmation of the operator's files; once the waggle amendment lands with CM2, signed events, approvals, grants and delegated acts, delegation budgets and revocations across machines through waggle's lease store, and `second-person` confirmation of the operator's files; hand-off across boards through waggle (CM1 and the amendment), with results signed by the receiving board's host keys and the binding of a process slot to another board's hosts (§6.5) |

The operator companion app is a track of its own ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §8). Until it ships, a person confirms on the machine in a local session, and a headless node is confirmed from the operator's workstation ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §9).

PC4 of the earlier plan is dropped *(decided)*: a playbook is a collection of skills. The constructor-ready schema, which the roadmap's PC4 row also carried, is part of PC1, and the Curator changes are the Curator goal (§8.3). The roadmap rows live in the wiki and are updated there. Details: [`track.md`](track.md).

## 16. Names

Specifications use technical terms, and each specification opens with a term map that gives the user-facing name beside each technical term where it is first used *(decided 2026-09-25)*. The operators approved the names of the term map at the top of this document on 2026-09-25, their English forms included: the whole system is the apiary, a node a yard, the main loop the keeper, a process a hive, its root orchestrator the queen, its board the comb, a run a bee, external participants wild bees, and a process package a nuc; a playbook keeps its name. This supersedes the mapping of 2026-09-24, in which a hive was a node's system and a comb a process. The wiki's naming page keeps the mapping; no identifier uses the names (the v1 owner kind `comb` is now `process`).

*Decided 2026-09-27:* beekeeping words appear only where people type them in an apiary: the `keeper` command and its nouns (`keeper hive …`), the apiary's directories (`~/apiary`, `hives/`) and the profile name `apiary`. Every other command, flag, configuration key, provider and identifier uses the technical terms, including task-board's, the session host's and Curator's command lines. The same rule stands in the main-loop specification (Names) and the session-host specification (Names).

## 17. Decisions and open questions

### 17.1 Decided in the design discussion of 2026-09-24

- the file and its format, and its four core sections;
- one owner per element in each state, where a state may admit several roles and assignment picks one, with helpers and `one_of`;
- owner kinds and who may move work out of each; authority outside the model, with signatures for risky human acts;
- playbooks as data only; side effects enabled by the project; admission by `extends`/`uses`;
- tools only through Curator skills: the orchestrator's manifest depends on each provider in `runtime` mode narrowed to exported commands, the kernel runs only commands of skills installed through Curator, and a project may rebind a tool (wiki decisions.md:84);
- needs without effort; effort in the operator file `~/.curator/models.toml`, with `run` candidate lists and effort ranges (decisions.md:83), and a committed project floor;
- relaxing a requirement permanently only in the project file, and temporarily through `ask` for the approver's own runs (amended in v3.4: an operator may also relax permanently for their own runs through an open-ended standing answer, §17.2 row 25);
- guards, actions and events from modules; the delivery module and its invariants; `task-board tick` until the main loop;
- `extends` and `uses` with slots and membership links;
- roles as skills, and playbooks as skill collections installed by one Skillfile schema 2 selector, with Curator releasing schema 2 and adding `directory` to `dependencies.skills`; package names `curator-playbook-<purpose>`;
- the static validator with portable and machine checks; the generality vectors, including per-process privacy as a views vector; the refuter;
- the demos dev-cycle, bug-hunt and Scrum right after PC1; PC4 dropped; the default playbook as the PC1 target;
- on 2026-09-25: the user-facing names of the term map; the trust framework, whose mechanism lives in [`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md), since v4.1 in relux-works/curator-trust; the principles of §1; capabilities, batched touches and yolo mode as the answer to which changes wait for a person (§2.4).

### 17.2 The operator's decisions

Rows 1–20 were taken on v2.1, rows 21–24 on v3.1, rows 25–29 on v3.3, rows 30–34 on v3.5, rows 35–39 on v4.0, rows 40–41 on v4.1, rows 42–43 on v4.2.

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
| 10 | a user-level system timer (launchd, systemd user timer) calls `task-board tick`; the main loop takes it over later | §6.9, §6.11 |
| 11 | role skills and the root orchestrator's skill install with Curator activation `on-demand` and reach a launch through the skills launch channel; description stopgap until Curator ships it. *Revised on 2026-09-25 (row 40):* they reach their runs through per-role profiles instead | §8.1, §8.2, §8.3, §15 |
| 12 | role metadata in `data/role.json` with its own JSON Schema (PC2) | §8.1, §15 |
| 13 | the `field` guard; review levels as a field with routes | §6.3, §10, §14 |
| 14 | the goal ledger gains `rejected` | §6.3, §10, §14, §15 |
| 15 | process rules in `Playbook.json`, runtime and board settings in `task-board.config.json`, model choices and machine limits with the operator, model admission with the project (row 21); legacy keys read for one release | §2.1, §2.3, §6.8, §14 |
| 16 | the operator's files live together under `~/.curator/`: `models.toml`, `runtimes.toml`, `engines.toml`, `network.toml` | §2.3 |
| 17 | `policy.wip` bounds the elements in progress per state, per role and per board; over a limit no run starts; parallel runs are machine capacity in `runtimes.toml` | §6.8, §14 |
| 18 | dev-cycle gains `blocked` with a required `block_kind` and a repeat limit escalating to the operator; the default is unchanged | dev-cycle example |
| 19 | per-element skills and preferences are future work after PC1 | §9 |
| 20 | check sets: playbooks name them, `task-board.config.json` runs them locally, on the hosting provider, or both | §6.3, §6.12, §14, §15 |
| 21 (P2) | a project may hard-restrict the models used on it: `policy.data.models` with allow and deny patterns and effort ranges, intersected with the operator's `[allow]`; today's `spawn.ceilings` migrate into it; bindings, workload classes and preferences go to `models.toml`; amends LP D1. How the runtime set, role ceilings, windows and the other ceiling keys map is the author's resolution (§23, S8) | §2.1, §2.3, §9, §10, §14; schema; scrum project |
| 22 (P3) | entries of the operator's files that widen admission take effect only after the operator confirms them through the operating system's user authentication (macOS LocalAuthentication, Windows Hello, polkit or PAM, or a FIDO2 key with user verification); narrowing edits apply at once; `[confirmation]` sets what each kind of widening needs; the operator-file commands through the `curator` umbrella are the mechanism. The effective baseline and the scope of the defaults are the author's resolution (§23, S6); open-ended standing answers are decided in row 25, and confirmation through the companion app in row 27 | §2.3, §9 |
| 23 (P4) | the formal-missions example in the style of prove2.me stays | §13 |
| 24 | delegation: a person may delegate approvals, transitions out of human-owned states and relaxations to an orchestrator role within `policy.delegation` and a granted `delegation` record with conditions, scope, expiry and budgets; everything else waits for the person. Whose sessions act, how approvals bind, how budgets are counted, how revocation and expiry end a grant, quorum and self-approval are the author's resolutions (§23, S1–S5, S14) | §0, §4.2, §6.4, §6.8, §9, §10, §15; schema; cafe project; waggle A1 §19.8 |
| 25 | open-ended standing answers are allowed (`until = "open"`), for the operator's own runs, each application visible on the board, under a periodic review, monthly by default, "с подтверждением". This amends the rule that only the project file relaxes a requirement permanently: for the whole team it does, for one operator's own runs an open-ended answer may. *The author's:* the key `policy.enable.standing_answers` and how far a delegated relaxation reaches (§24); that the review binds — an answer not re-confirmed within a period of its review stops applying — is v3.5's alignment with the operator's wording, replacing v3.4's "stays in force" (§25) | §2.3, §9, §17.1; schema |
| 26 | what grants authority in `Playbook.json` takes effect only once a person confirms it. *The author's:* v3.4 wrote this as a list of sections, with the rest of the file taking effect on commit and the first version confirmed at creation (§24); v3.5 replaces the list, which one committed transition could pass (refuter v4, T1), with the default that every change to the resolved configuration — templates, tool providers and the roster included — waits for a person, as whole versions, batched, confirmed by the project's confirmers, a person confirming the first version (§25). The choice between that default and a classified variant was decided on 2026-09-25 as capabilities (row 35), and the rule before CM2 as one confirmation for every machine (row 32) | §0, §2.4, §4.2, §10, §15 |
| 27 | a person confirms on the machine (the operating system's user authentication) or through the operator companion app on a phone: a key in Secure Enclave or Android Keystore behind biometrics, paired with the node at initialization through an out-of-band handshake that pins keys, requests over a channel that resists a man in the middle (a tailnet, or a relay encrypted end to end to the pinned keys), the approved content rendered before signing; headless nodes rely on the app, the common case; the app's design, transport and SSH administration are open. That an attested companion signature meets `user_verified: required` for board acts follows the operator's direction that people confirm through the phone. *The author's:* the pairing invariants and, until the app exists, confirmation of a headless node from the operator's workstation, which replaces v3.4's terminal-session path (§25) | §2.3, §4.2, §15, §17.3; waggle A1 §19.5 |
| 28 | rule changes are committed to the default branch; git hosting is optional and pull requests are never required. *The author's:* `task-board playbook apply` for a project without git, with the same confirmation, which replaces decision 2a's read-as-is (§24, §25); named `task-board config propose` since v4.4 (§30) | §0, §2.4 |
| 29 | agents search the Curator registry and install audited skills themselves; skills from unknown sources are refused by default and enabled only through a deliberate, confirmed operator setting with a warning, with submission for audit into the Curator registry as the recommended path. v3.5 makes installing the default (`policy.enable.skills: audited`), as the operator put it, where v3.4's default off was the author's. *The author's:* that a project may restrict installs to named skills or `none`, the `installs` delegation act, that an audit covers exact package identities, and that an installation which changes the resolved configuration waits for a person (§24, §25) | §0, §2.3, §4.2, §8.4, §8.6, §10; schema |
| 30 | the user-facing names: apiary = the whole system, one node or many; yard = node; keeper = main loop; hive = process; queen = root orchestrator; comb = board; bee = run; wild bees = external participants; nuc = process package. Each specification opens with a term map. The English forms were confirmed on 2026-09-25, in the answers to v4.0 | Terms, §16 |
| 31 | the trust framework: only a signature by a key that requires a person's presence confirms, and no project lowers this; the signing device computes and renders what it signs; confirmations form a chain and never revive after a revert; confirmations are signed records on the board that every machine verifies against the roster of the version they replace, without CM2; the first enrollment is protected; staged trust from one operator (L0) to several (L1) and public participants (L2), with the same invariants; transport is not authority; secrets never enter agent environments. The first enrollment (f), staged trust (g) and secrets kept out of agents (i) were confirmed on 2026-09-25, in the answers to v4.0, with the web of trust and multisignatures (row 37). *The author's:* the split of the mechanism into [`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) behind one interface, and PC1's guardrail stub | §0, §1, §2.3, §2.4, §4.2, §15; trust.md |
| 32 | a board shared across machines: each machine runs what its operator confirmed, and, because each machine verifies every confirmation itself (row 31), one confirmation serves every machine and every machine runs the same version in force; v3.5's per-machine versions and their divergence report are withdrawn (§17.3 item 14) | §2.4, §7.1; trust.md §4 |
| 33 | a person's signed narrowing acts take effect at once, without the confirmers, and there are exactly three: revoking a key, revoking a grant, and disabling a tool (refuter v5, D4; confirmed on 2026-09-25). A grant is a delegation grant or, since v4.1, a capability grant | §2.4, §4.2; trust.md §5 |
| 34 | principles for production, taken from the operator's architecture note "very ascetically": agents speculate and deterministic rules move authoritative state; the journal is the state and a session its cache; delegated and nested authority only narrows; write scopes as a security and concurrency primitive; integration of the exact tree with a compare-and-set landing; transport is not authority; privileged operations outside agent shells. *The author's:* which principles were selected | §1 |
| 35 | which changes wait for a person (§17.3 item 13): **capabilities**, granted once per kind the way a phone app asks for a permission, with the answers always, once or no. Revocation takes effect at once. There is a monthly review, and a grant is sealed after a couple of re-confirmations ("seal the decision after a couple of reminders", in the operator's words). **Batched touches** cover everything else, and an explicit **yolo mode** is available. v4.0's allowlist and path check are withdrawn. *The author's:* the exact kinds and what each covers (§2.4), following the operator's table; two consecutive re-confirmations to seal; an unanswered review keeping a grant in force; yolo mode's refusals; the version in force derived from history (§27) | §0, §2.4, §6.6, §8.5, §8.6, §10, §15; trust.md §16, §17 |
| 36 | an attested companion signature meets `user_verified: required` for board approvals and every other board act (refuter v5, D3) | §4.2; trust.md §6 |
| 37 | the web of trust: the first key is trusted at initialization with the operator present, and keys signed by an already trusted key are trusted, so trust moves to a new phone; multisignatures (M of N) are designed now and switched on from L1 | trust.md §1, §3, §19 |
| 38 | the trust specification moves to its own private repository, relux-works/curator-trust; the change-analyst role only explains and never decides | Status, §2.4; trust.md §4 |
| 39 | privileged operations stay outside agents: the agent requests, and a trusted executor acts after authorization. A separate OS account for agents, with a broker for the edits they may make, is the lasting boundary | §1; trust.md §12, §18 |
| 40 | role skills reach the runs of their role through per-role Curator profiles, and never through the project's discovery directories; on-demand activation and the skills launch channel are withdrawn, and curator-spec issue #91 and curator issue #91 are closed (2026-09-25) | §0, §8.1, §8.2, §8.3, §15 |
| 41 | the `properties` block and the decision consistency check (`decision-check`) come after PC1 and are not part of the freeze (2026-09-25) | §17.3 items 17 and 18 |
| 42 | loop limits are tunable per project, down to unbounded; dev-cycle's rework and rollback loops are unbounded by default, as today (2026-09-25). The `params` form — declarations with defaults in a template, plain values in a project, references in `limit.max`, `on.after` and `policy.wip` — and the unbounded-cycle warning are the author's rendering | §6.2, §7.3, §10, §13 |
| 43 | entering a dream is approved, scheduled and conditioned — for example on remaining subscription quota or a ready local model — and a dreaming orchestrator runs in a session of its own with a profile made for dreaming (the operator's direction, 2026-09-25). The capability kind `dreams`, `policy.dreams` and the entry and wake sequence are the author's proposal | §6.8, §6.13; trust.md §16 |

H4 (keep the effort floor), H6 (the demos and the PC rows) and H8 (one owner per element in each state; privacy as a views vector; the refuter's verdicts) were decided earlier and stand.

### 17.3 Open questions

1. *Decided in v3.4:* open-ended standing answers are allowed, with a periodic review, and a delegated relaxation answers one `ask` at a time (§9, §17.2 row 25). The item keeps its number so that earlier change logs still point to it.
2. **First start without `models.toml`.** What the first launch shows an operator who has no file: whether `curator models init` proposes bindings for the project's roles from the runtimes installed on the machine, and how a refusal explains the missing admission (§2.3, §9).
3. **OS-level protection of configuration files, keys and executables** ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §12, §14 item 1). How the operator's files, their confirmed versions and the confirmation key are protected from replacement by a process running as the operator, and how a project's root configuration is: file permissions, edits only through a privileged helper or `sudo`, a separate owning account, immutability flags (§2.3). This covers the operator's files, the project's root configuration and the host keys that sign hand-off results (§6.5) alike. It also covers the version in force the kernel stores (§2.4), the keys paired with the companion app or registered for the workstation path, and the `task-board` and `curator` executables, which any process of the operator's account can replace today. It applies to every root configuration, not only `models.toml`. *The direction is decided on 2026-09-25 (§17.2 row 39):* the real boundary is a separate OS account for agents, whose edits to the operator's files go through a broker ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §12, milestone TR6); until then every protection named here is a guardrail. What stays open is the broker's interface and the hardening that comes before it.
4. **The reference prompt size** for `pp` and `ttft` measurements (PC1, §9).
5. **Per-hook argument schemas** for every provider module (PC1). The schema covers the arguments of `field` and `delivery.checks-passed` already.
6. **The remaining mapping points of §14**: `defer_cr` and `withdraw_cr`, and the close-landed methods as `via` commands (PC1).
7. **How a process is created and registered on a node**, how the main loop routes a request among processes and resolves a request that fits several, and how playbooks are generated and generalized: the main loop's process registry, outside this specification: its draft specification is relux-works/curator-keeper `spec/keeper.md`, and the wiki keeps the research note and open questions.
8. **Outside this repository**: this text amends agent-selection §3, §4, §7 and §8 and LP D1, D2 and D9; the drafts in skill-project-management take the amendments when those tracks resume. The waggle amendment, delegation included, lands in the waggle repository; command-only installs land in Curator (curator-spec issue #95, curator issue #95), and per-role profiles come with the launch-profiles work (on-demand skills, curator-spec issue #91, were withdrawn on 2026-09-25); the operator-file commands land in the components that own each file.
9. **Cross-board result signers** (PC3): the form in which a project binds a process slot on another board to that board's host principals (§6.5).
10. **A project-wide delegation budget.** Budgets belong to grants, so two managers' grants of 2,000 EUR a week each let the orchestrator commit 4,000 (§4.2). Whether `policy.delegation` rules may carry a cap across every grant under them is open.
11. **The operator companion app** ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §8, §14 item 5): its design and platforms, iOS first; a desktop variant that manages dedicated nodes; administration of the node over SSH; the pairing code for nodes without a screen. Until the app exists, confirmations are made on the machine or, for a headless node, from the operator's workstation ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §9).
12. **Submitting a skill for audit** (§8.6): the channel through which `curator sources submit` sends a skill from an unknown source to the Curator registry's audit, and how the result returns to the operator.
13. *Decided on 2026-09-25 (§17.2 row 35):* which changes wait for a person is answered by capabilities, granted once per kind, with batched touches for everything else and an explicit yolo mode (§2.4; [`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §16, §17). v4.0's allowlist and path check are withdrawn. The item keeps its number so that earlier change logs still point to it.
14. *Resolved on 2026-09-25 (§17.2 row 32):* confirmations are signed records every machine verifies, so one confirmation serves every machine. The item keeps its number so that earlier change logs still point to it.
15. *Decided on 2026-09-25 (§17.2 row 36):* an attested companion signature meets `user_verified: required` for board approvals and every other board act ([`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) §6). The item keeps its number.
16. **Profiles as package content.** Whether a playbook package ships Curator environment profiles, whether a role names the profile its sessions run with (`roles.<alias>.profile`), and how an operator's persona overlay composes over them without weakening a role's rules (wiki open questions).
17. **After PC1: verifiable process properties** *(decided 2026-09-25 to come after PC1, not part of the freeze; §17.2 row 41)*. The operators asked whether task flows can be formally linked and verified. A playbook's workflows are finite graphs, so a small declarative `properties` block could state a few property kinds, and the validator could check them. The kinds would be path safety ("done only after review"), never-without-guard, reachability (every state can reach a terminal state, as the validator's terminal-reachability check does today) and liveness under fairness. An optional export to TLA+ or Quint would serve deeper checks. Task-level verification is already expressible through tool-owned checking states and joins. Properties of a model's judgement, hyperproperties and statistical properties stay out of reach. This would be a small addition after PC1, not part of the freeze (wiki `research/verifiable-process-properties.md`). The property that found dev-cycle's unbounded rollback loop (§28) — every cycle passes a bounded `limit` or a human-owned state — is of this kind. Since v4.3 the bound is the project's choice (§7.3), and until this block exists the validator warns about unbounded cycles.
18. **After PC1: a decision consistency check** *(decided 2026-09-25 to come after PC1, not part of the freeze; §17.2 row 41)*. When a new element arrives — an epic, a story, a decision — the question is whether it contradicts the chain of decisions already accepted, and if it does, which decisions it affects and how the contradiction can be resolved.
    - An optional `formal` field on `decision` elements holds a statement over the project's controlled vocabulary, for example `auth.method in {sso}`, `storage.encrypted = true` or `latency_ms <= 200`. A new element may carry one too.
    - An optional tool skill, `decision-check`, runs an SMT solver (for example Z3) over the accepted decisions' statements together with the new one. Satisfiable means no contradiction. Unsatisfiable yields a minimal unsatisfiable core — exactly the accepted decisions the new element contradicts, which is the scope of affected decisions — and the minimal correction sets, which are the resolution options, each with its scope: "change decision D12", or "change the new element".
    - In a playbook it is a tool-owned `consistency-check` state before an epic or a decision is accepted. A conflict routes to `decision-needed` with the core attached, where a person chooses a resolution.
    - It is the formal twin of the refuter (§12): the refuter finds contradictions by reasoning over everything, and the check proves them over what has been formalized.
    - Limits: only formalized statements are checked, and a wrong formalization is the main risk, so a person or the refuter reviews every formal statement, and a project starts with a small vocabulary (wiki `research/verifiable-process-properties.md`).
19. **Dream mode details** (§6.13). Entry is proposed in v4.3: a `dreams` capability, `policy.dreams`, park, dream profile, resume. Open are how a dream is scoped (an element, a workflow, a whole process), how long its sandbox lives, whether a project may require a dream rehearsal before a structure change is offered for confirmation, and how a role's dream profile is packaged with its role profile (item 16).
20. **Tool outputs, receipts and assisted tools** (§6.11.1). Proposed on 2026-09-28 for PC3. Open: the output block's format on standard output, a grant's tolerance for receipts above the approved amount, and how an assisted request is shown on the phone and at the terminal.
21. **Files and staleness** (§5.5). Open:
    - where artifacts live (a content-addressed store, Git LFS, or both) and how long they are kept;
    - which transitions pin inputs, and how deep a reopening may cascade before it asks a person;
    - how the impact report reads cost when elements declare no cost field.
22. **Tools over time and resources** (§6.11.2). Open:
    - the checkpoint contract a resumable provider implements;
    - resource leases across machines (with waggle's leases, CM2);
    - the sandbox profile on each operating system (a container, a separate account, or both; curator-trust §12).
23. **Budgets and waves** (§6.14). Open:
    - whether token and minute budgets per container need the Keeper, or the session host is enough;
    - how a wave's failure policy is declared beyond "do not grow".
24. **The publishing module** (§6.15). Its scope, its first platforms, and who owns rights and licence checks.
25. **Typed findings and helper answers** (§6.16): their schema, delivered with the provider modules in PC1.

**What this means for PC1.** PC1 can start: everything it implements is decided or has a stated default. Item 13 is decided, so the classification of §2.4 is capability coverage. PC1 ships the trust component as a guardrail stub; the mechanisms of [`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md) arrive in PC3. The gates on later work are these:

- the waggle amendment gates PC3's signed board acts and the cafe demo;
- per-role profiles, from the launch-profiles work, gate PC2's clean install;
- the operator-file commands gate widening entries of the operator's files;
- the session host's marks gate the counting of delegated acts.

The three first demos, built in to PC1 (§8.2), need none of the open answers.

## 18. Change log: v1 → v2

Each finding of the refuter's first run ([`../reviews/2026-09-24-refuter-v0.md`](../reviews/2026-09-24-refuter-v0.md)) and where v2 answers it. Section numbers in this table refer to v2; v2.1 keeps them, except that the open questions of §17 were renumbered (§17.3). v3 keeps them too, adds §6.12 and §20, and replaces §17.2 and §17.3, so references to open questions in §18 and §19 name the v2 and v2.1 lists. v3.1 adds §21, v3.2 adds §22, v3.3 adds §23 and renumbers §17.3, v3.4 adds §24, v3.5 adds §25 and items 13 and 14 of §17.3, v4.0 adds the term map, the principles of §1, items 15 and 16 of §17.3, §26 and [`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md), and v4.1 adds §27 and item 17 of §17.3, closes items 13 and 15, and moves trust.md to relux-works/curator-trust.

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
| 9 tool enable gate (H3) | `policy.enable.tools` lists tools or says `all`; not listed means disabled; the `null` form removed. `policy.enable.schedules` takes the same form, `all` or a list, where v2.1 took a boolean; `all` admits every schedule the admitted templates declare, including ones later versions add (§3, §6.6, §8.5, §6.11, §10; schema) |
| 10 tick scheduler | a user-level system timer; the main loop later (§0, §6.9, §6.11; track) |
| 11 on-demand role skills | Curator activation `on-demand` and the skills launch channel, identity delivery, description stopgap; Curator's third change (§0, §8.1, §8.2, §8.3, §8.4, §15; track) |
| 12 role metadata | `data/role.json`, not frontmatter, with its own JSON Schema in PC2 (§8.1, §15) |
| 13 field conditions | the kernel guard `field`; review levels as routes; the guard-bypass exception for disjoint conditions (§5.2, §6.3, §6.8, §10, §11, §14; schema; dev-cycle) |
| 14 `rejected` | the goal ledger gains it in PC1; the validator's refusal removed (§6.3, §10, §14, §15) |
| 15 where settings live | the project's three files; process rules in `Playbook.json`; legacy keys read for one release; model keys and machine limits with the operator (§2.1, §2.3, §6.8, §9, §14) |
| 16 operator files together | `~/.curator/models.toml`, `runtimes.toml`, `engines.toml`, `network.toml`; LP D2 and the network file location amended (§2.3) |
| 17 work in progress | `policy.wip` with `board`, `per_role`, `per_state`; v3 counted active runs and mapped `spawn.max_parallel` onto it, which v3.1 corrects to elements, with `spawn.max_parallel` as machine capacity (§21, R1) (§6.8, §14; schema; scrum project) |
| 18 block kinds | dev-cycle's `blocked` state, `block_kind` field and repeat limit (dev-cycle example) |
| 19 per-element needs | marked *future* (§9) |
| 20 check sets | `delivery.checks-passed` and `delivery.integrate`'s `checks`; `checks` in `task-board.config.json`; local, hosted or both; per task class; the local-mirror rule (§6.3, §6.12, §14, §15; schema; dev-cycle) |

## 21. Change log: v3 → v3.1

Each finding of the refuter's third run ([`../reviews/2026-09-24-refuter-v2.md`](../reviews/2026-09-24-refuter-v2.md)) and where v3.1 answers it. The two findings v3.1 left to the operator (P2, P3) were decided after it and are applied in v3.2 (§22).

| Finding | Resolution in v3.1 |
| --- | --- |
| R1 `policy.wip` counted runs (blocking) | `policy.wip` counts elements: `per_state` in the state, `per_role` in states the role owns or assigned to it, `board` in non-initial non-terminal states; a manual or agent transition over a limit is refused with `wip_limit_reached`, an automatic one is held until capacity frees; `spawn.max_parallel` and the suite cap become machine capacity in `runtimes.toml`; the default sets no `policy.wip` (§2.1, §2.3, §6.8, §14 row and point 12, §17.2 row 17, §20 row 17; schema; scrum project; wiki) |
| R2 check sets ran in the gated actor's environment | local sets run through the kernel's executor in a clean checkout of the exact tree with a sanitized environment, the executor recorded with the evidence; today's in-process run is an accepted difference (§6.11, §6.12, §14 point 11, §15) |
| R3 mirror evidence both satisfied and did not satisfy a hosted set | mirror evidence is recorded as `mirror`, never as the hosted half; the board waits while the hosted half is missing; a person may authorize the landing with an override, recorded with the mirror evidence; the rule is defined here (§6.12, §15) |
| R4 the landing gate could be left out | `delivery.integrate` requires `checks` (schema); an undefined set refuses integration with `checks_not_configured`; a failed or unavailable set emits `delivery.integration-failed` with its rollback authorization; the default names `landing` through `delivery.integration-evidence` (§6.3, §6.12, §14; schema; default) |
| R5 check-set evidence undefined for inline work | `delivery.checks-passed` passes for an element without a Change Request; inline routes name no set and inline classes run no suite, as today (§6.3, §6.12; dev-cycle) |
| R6 Change Request work with review `none` had no route | dev-cycle's `development → review` no longer routes on `review`, so any level may be reviewed and Change Request work always reaches `accept_cr`; the ledger's `none` shortcut applies only where a workflow declares no `field` route on `review` (§6.3, §14 point 7; dev-cycle) |
| R7 field routes and approvals did not bind their values | fields a `field` guard reads are authority-bearing: runs do not write them on their own element, and `review` is set by the orchestrator or a person; every approval records the values of the fields its transition reads and counts only while they are unchanged; a tool receives the field values recorded at entry; the disjoint-field exception covers approvals guards and `risk` markers; waggle A1 §19.1 and §19.7 changed with it (§5.2, §6.3, §6.4, §6.11, §10). The fourth run found the binding too narrow: v3.3 binds the digest of every declared field, makes the fields that delegation ceilings, conditions and budgets read authority-bearing, and freezes them while a delegated approval is unused (§23, S2) |
| R8 the operator's files as an authority source | decided by the operator after v3.1 (P3): widening entries take effect only after confirmation through the operating system's user authentication (§2.3, §9; §22). v3.3 measures widening against the effective version, so re-widening after a narrowing needs confirmation again, and leaves open-ended standing answers to the operator (§23, S6, S7) |
| R9 the scrum example could not run as a PC1 demo | the scrum project sets no signature policy and runs its sets locally; the validator warns, naming the milestone, when `require_for` names a category that has no signable form yet (§4.2, §10, §13; scrum project and configuration) |
| R10 the cross-board result was a model's claim | a host key of the receiving board signs `coord.handoff-result` when its kernel records the terminal transition, with the receiving board's revision; the offering board verifies it against the hosts its project file binds; waggle A1 §19.4 changed with it (§6.5, §15, §17.3) |
| R11 model admission moved to the operator's files | decided by the operator after v3.1 (P2): a project may hard-restrict the models used on it through `policy.data.models`, and the ceilings migrate there (§2.3, §9; §22). v3.3 makes the migration lossless: the runtime set, role ceilings and windows stay project admission, and every other ceiling key has a target (§23, S8) |
| R12 dev-cycle's `blocked` | two kernel-written state keys, `record_source` and `clear_on_exit`: `blocked` records the source state in `blocked_from` and clears `block_kind` and `blocked_from` on exit; the element resumes in the state it was blocked from, so a block from `open` returns to `open` and meets `open → development`'s guards; one limit counter per declared transition and element; a transition's key gains `on`, so a manual resume and the automatic dependency release between the same states coexist; §11 no longer claims routes on block kinds (§6.1, §6.2, §7.1, §11; schema; dev-cycle) |
| R13 field defaults had no semantics | a field default is written at creation by every creation path; `policy.review.default` wins over the field's own default for `review`; legacy records read an empty `review` as `required` (§5.2) |
| R14 signature defaults | whenever a signature policy is set, `min_assurance` defaults to `hardware-bound`; `user_verified` defaults to `required` for approvals, relaxations and halts and to `if-supported` for spending, overrides and human transitions, settable for all categories or per category (§4.2; schema; waggle A1 §19.5; wiki) |
| R15 who signs relaxations and overrides; who governs halts | waggle A1 §19.1 names the approver set per board type, and §19.5 lets the project's process configuration govern `cmd.halt` and `cmd.cancel` to runs of the process |
| R16 signed acts bound a moving commit | acts bind and the board compares the semantic digest of the resolved configuration, not the commit (§6.4; waggle A1 §19.1, §19.7) |
| R17 boards owned by another repository | `Playbook.json` lives in the board repository; each Story's check sets come from its code owner's `task-board.config.json`; templates resolve in the board checkout's scopes (§2.1, §6.12) |
| R18 cloud speed was never measured | PC1 records `ttft` and `tg` per run where the harness exposes them; a cloud model's `pp` is declared or unknown (§9, §15) |
| R19 user-facing names in specification text | "the main loop", "the main loop's process registry" and "node" in the text and the track; §16 alone keeps the mapping (§6.9, §6.11, §17.2, §17.3, §20; track) |
| R20 schema and text drift | the `services` and `service-group` descriptions name the amendment as decided; the `enable.schedules` change is recorded and `all` defined for schedules; the validator checks `min ≤ max` in ranges (§6.6, §10, §20; schema) |

## 22. Change log: v3.1 → v3.2

The operator's decisions on v3.1 (§17.2, rows 21–24) and where v3.2 applies them.

| Decision | Change in v3.2 |
| --- | --- |
| P2 (refuter v2, R11): a project may hard-restrict the models used on it | `policy.data.models` with `allow` and `deny` patterns (`<runtime>/<model>`, `<runtime>/*`, `*/<model>`) and effort ranges on allow entries; project admission ∩ operator `[allow]` ∩ machine facts ∩ needs; without an operator file the automatic choice stays inside the list; an information check names roles only listed models can serve; `spawn.ceilings` and the `exclusive` runtime set migrate into it, while bindings, workload classes, the `mixed` order and schedule windows go to `models.toml`; LP D1 and agent-selection §4 and §7 amended (§2.1, §2.3, §9, §10, §14 rows and point 13; schema; scrum project) |
| P3 (refuter v2, R8): widening entries of the operator's files | a widening entry takes effect only after the operator confirms it through the operating system's user authentication, recorded as a signature by a key that needs that authentication; the resolver applies the more restrictive of the current file and the last confirmed version; narrowing edits apply at once; `[confirmation]` levels `none`, `user`, `second-person`, defaults `user` for metered billing and standing answers; a standing answer takes the role's approval scope and expiry unless `until` sets a date or says `open`; the operator-file commands decided, with `confirm`; OS-level protection of configuration files opened as a question (§2.3, §9, §17.3) |
| P4: the formal-missions example | kept as it is (§13) |
| Delegation | `policy.delegation` (project-only, authority-bearing) with `allow` and `never` rules over approvals, transitions out of human-owned states and relaxations, delegate roles with capability `orchestrator`, and `field` ceilings; the kernel record `delegation` with grantor, delegate, acts, conditions, scope, expiry and period budgets, granted with user authentication and signed when a signature policy exists (new signature category `delegations`, `user_verified` required by default); delegated acts recorded on behalf of their grantor and counted only within the delegation; everything else waits for the person, notified through the main loop; revocation immediate and never retroactive; signed delegated acts verified through the chain of waggle A1 §19.8 (§0, §4.2, §6.4, §6.8, §9, §10, §13, §15; schema; cafe project) |
| Open questions | the operator-file commands, P2 and P3 leave §17.3; OS-level protection of configuration files, and routing and playbook generation on the main loop's side, join it (§17.3) |

## 23. Change log: v3.2 → v3.3

Each finding of the refuter's fourth run ([`../reviews/2026-09-24-refuter-v3.md`](../reviews/2026-09-24-refuter-v3.md)), on v3.1 and v3.2, and where v3.3 answers it. The resolutions are the author's and align the text with the operator's record; one point, whether a standing answer may be open-ended, waits for the operator (§17.3, item 1).

| Finding | Resolution in v3.3 |
| --- | --- |
| S1 a delegation could be exercised by any operator's session of the delegate role | by default a grant counts acts only from the grantor's own orchestrator sessions — the ones the session host marks as the grantor's in PC1, a guardrail, and the ones whose certificates the grantor's key issued from CM2 — and a grant may name other session principals explicitly, never a pattern; a session belongs to a role through the alias its launch records, and from CM2 through the certificate extension `waggle-role@relux.works`, checked at the pinned revision; waggle A1 §19.8's chain checks the issuing key (§0, §4.2, §9; waggle A1 §19.8) |
| S2 approvals and delegation ceilings did not bind the values they depend on | every approval, direct or delegated, binds the digest of all the element's declared fields, and any change to a declared field voids unused approvals; the board compares the digest; fields read by delegation ceilings, conditions and budgets are authority-bearing and frozen while a delegated approval is unused; conditions and ceilings are checked when the act is recorded and again when a guard counts it; §10's threshold argument restated; waggle A1 §19.1 and §19.7 aligned (§5.2, §6.3, §6.4, §6.11, §10; cafe) |
| S3 delegation budgets had no single counter | the board's kernel alone owns and counts budgets: a sum of a named field over the acts recorded under the grant within a rolling period, using the value each act bound, approvals counted for grants that cover them, voided approvals refunded; check and record are one compare-and-set in the lease store; before CM2 a grant with a budget is refused on a board that is not single-host; a project-wide cap is open (§4.2, §6.9, §17.3 item 10; waggle A1 §19.8) |
| S4 revocation and expiry treated in-flight acts differently | revocation and expiry end a delegation the same way: unused delegated approvals stop counting, consumed acts stay, nothing new is taken; across machines a revocation takes effect through the lease store from CM2 and when the board syncs before it (§4.2; waggle A1 §19.8) |
| S5 one delegate could meet a quorum and approve what it wrote | acts under delegation never count as more than one principal toward a quorum, which needs distinct human principals acting directly or distinct delegate sessions of distinct grantors; a delegated approval does not count when the delegate session or a run it spawned wrote a declared field of the element, unless the admitting rule sets `self_approval`; invariant 2 and independence read the delegate's provenance (§4.2, §6.4; schema; cafe project) |
| S6 an agent could undo a narrowing without authentication | the resolver keeps the effective version — the last confirmed one narrowed by every later narrowing — and a narrowing becomes the baseline at once, so re-widening needs confirmation again; the headline and the defaults say which kinds need confirmation; before the commands ship, only widening entries whose level is not `none` are ignored; a runtime binding's billing class comes from facts, never from the binding (§0, §2.3, §17.2 row 22) |
| S7 standing answers could be permanent waivers | not decided: whether a standing answer may be open-ended, and how far a delegated relaxation reaches, is pending the operator (§17.3 item 1); the text stands meanwhile and is flagged where it appears (§2.3, §9, §17.2 row 22) |
| S8 the P2 migration lost admission | lossless: the runtime set of `preferred_agentic_system` in either form becomes allow patterns, its `mixed` order the operator's candidate order; per-role ceilings become `policy.data.models.roles`; schedule windows become allow entries with `windows`, which the operator's `[windows]` narrow; `resolution_model` and `adjustment_confirmation` become the operator's candidate order; Codex `fast_mode` an option of the operator's binding; `supervision` stays in `task-board.config.json` as `spawn.supervision`; the most specific allow entry decides; a row without reasoning effort meets an entry's range vacuously; row 21 no longer records the windows as the operator's decision (§2.1, §2.3, §9, §10, §14 rows and point 13, §17.2 row 21; schema; scrum project) |
| S9 PC1 needed an executor scheduled for PC3 | the kernel's executor is one component that PC1 ships for local check sets and that tick calls from PC3 for tools, timers and schedules (§6.11, §6.12, §15) |
| S10 the orchestrator could steer where a blocked element resumes | only the kernel writes a field that `record_source` names; the validator checks that its type admits every source state of the entries (§6.1, §10; dev-cycle) |
| S11 `blockers.terminal` was undefined for elements without links | an edge event, as today: it fires when the last linked blocker ends for an element with at least one link, never for an element without links, whose resume is manual; an element whose blockers were already done is reported stale (§6.9, §14) |
| S12 `second-person` and roster-verified confirmations had no form | `second-person` is refused, naming CM2, until a second operator's waggle-signed confirmation can be produced and verified; `[confirmation]` in `models.toml` governs every file of the directory; the roster sentence is dropped (§2.3, §15) |
| S13 the hand-off result's namespace and host key | waggle A1 amends its §4.3 step 2 with the signer role `result`, whose namespace is `result.coord.v1@waggle`; the receiving board's host key is at least `hardware-bound` or held by a separate OS account; host keys join the open question on OS-level protection (§6.5, §17.3 item 3; waggle A1 intro, §19.4, §19.6) |
| S14 delegation rules left to waggle; PC1's gaps | who may grant (a person whose principal could perform every act the grant covers) and who may revoke (the grantor or any `people` principal) are stated for signed and unsigned grants; delegation rules name states as `<workflow>.<state>`, and approvals rules may list the states they cover; a grant binds the configuration digest, ends when the resolved delegation policy or its roles and states change, and is verified once when recorded; its scope is an element it names or the project; PC1 notifies the grantor through a board notice and carries the operating system's user authentication in task-board; the signable-forms warning covers the implicit `delegations` category (§4.2, §10, §15; schema; waggle A1 §19.8; track) |
| S15 an unavailable hosted half both failed and waited | a set that fails, or a local set that cannot run, emits `delivery.integration-failed`; an unavailable hosted half makes the board wait (§6.3, §6.12) |
| S16 committed machine limits lost their legacy read | for one release a committed `spawn.max_parallel` or `spawn.validation.max_concurrent_suites` is read as the machine's capacity where `runtimes.toml` does not set it, with a warning; the defaults are 20 and 2 (§2.1, §2.3, §14 point 12) |
| S17 examples, validator and text drift | the scrum project's `per_state` limit replaced by one that can bind; the cafe's approvals rule limited to `purchase.approval`, with a check that every type a rule reaches declares its ceiling fields; the `[answers]` example uses a role that asks; an exception without `needs.approval` covers the element for 30 days; overlapping allow entries resolved by specificity and a row without effort defined; schema descriptions of `via` and `remove` use the four-part key; version labels, the related-specifications line, the track and §16 corrected; author resolutions marked as such in the wiki (§2.3, §9, §10, §16; schema; examples; track; wiki) |

## 24. Change log: v3.3 → v3.4

The operator's answers that followed v3.3 (§17.2, rows 25–29) and where v3.4 applies them. Three gaps in them are filled by the author and marked as such. The fifth refuter run found further fills recorded as the operator's (refuter v4, T14): the list of authority-bearing sections and the rest of the file taking effect on commit, an unreviewed answer staying in force, the key `policy.enable.standing_answers`, `task-board playbook apply`, agent installs off by default with the `installs` act, and the terminal-session path for headless nodes. They are the author's; §17.2 now marks them, and each row below says where v3.5 supersedes it (§25).

| Decision | Change in v3.4 |
| --- | --- |
| Open-ended standing answers (row 25; refuter v3, S7) | allowed with `until = "open"`, for the operator's own runs, each application an exception on the board; `policy.enable.standing_answers: false` lets a project forbid standing answers; a monthly review by default (`[review] answers`), started by the main loop or, until it exists, a board notice pointing to `curator models answers review`, lists open-ended answers for re-confirmation, and an unanswered one stays in force, shown as unreviewed (superseded in v3.5: an answer not re-confirmed within a period stops applying, and every answer whose end lies beyond the next review is reviewed, §25, T7); the rule on permanent relaxation now distinguishes the whole team (only the project file) from one operator's own runs; §17.3 item 1 is closed. *Author's resolution:* a delegated relaxation answers one `ask` at a time with the role's approval scope and expiry, may be limited to named roles and terms (`relax`), and never becomes a standing answer (§2.3, §9, §17.1; schema) |
| Authority-bearing sections need a person (row 26) | the authority digest of the committed version takes effect only once a person confirms it; until then the kernel applies the last confirmed authority sections, reports the pending change and notifies the operators; the other sections take effect on commit and may change within the project's delivery and its delegations; the authority report at every load and in `playbook validate`. *Author's resolutions:* workflow changes that take authority from people — a human-owned state removed or re-owned, an `approvals` guard or `risk` marker removed, a `field` condition that routes to a person loosened, a delivery guard removed — count as authority-bearing, because otherwise one commit could bypass the rule; a change that depends on a pending authority change waits with it; a project's first version is confirmed at creation, and until then no tool runs and no delegation or relaxation is honoured (§0, §2.4, §10, §15). *Superseded in v3.5:* the list, the rest of the file taking effect on commit and the enumerated workflow changes give way to the default that every change to the resolved configuration waits for a person, as whole versions; a person confirms the first version (§25, T1, T4, T8) |
| How a person confirms (row 27) | one definition for every confirmation: the machine's user authentication, or the operator companion app — a key in the phone's secure hardware behind biometrics, paired at initialization through a QR code printed in the operator's terminal that pins keys on both sides, requests over a tailnet or an end-to-end-encrypted relay, the approved content rendered before signing; a companion signature meets `user_verified: required` through the app's attested enforcement (waggle A1 §19.5); delegation grants, operator-file widenings and unknown skill sources use it; no delegation covers a confirmation; until the app ships, confirmations are made on the machine and a headless node over the operator's terminal session (*superseded in v3.5* by the workstation path, §25, T6); the app is a track of its own, and its design, transport and SSH administration are §17.3 item 11 (§2.3, §4.2, §15, §17.3) |
| Commits, not pull requests (row 28) | a change takes effect when committed to the default branch; git hosting is optional and pull requests are never required; a project without git applies versions with `task-board playbook apply` and the same confirmation (§0, §2.4, §15) |
| Skills (row 29) | new §8.6: agents search the Curator registry and install audited skills where `policy.enable.skills` (`audited` or named skills) or a delegation (`installs`) allows it, each installation recorded on the board (*superseded in v3.5:* installing is the default, `audited`, as the operator put it, §25, Q2); installing gives instructions and runs nothing, and a skill's tools run only after a confirmed commit to `tools` and `policy.enable.tools`; skills from unknown sources are refused by default, allowed only through a deliberate Curator setting that warns and is a widening of kind `skills`, never below `user` and never delegable; submitting a skill for audit is the recommended path, its channel §17.3 item 12; machine checks for both (§0, §2.3, §4.2, §8.6, §10; schema) |

## 25. Change log: v3.4 → v3.5

Each finding of the refuter's fifth run ([`../reviews/2026-09-24-refuter-v4.md`](../reviews/2026-09-24-refuter-v4.md)), on v3.3 and v3.4, and where v3.5 answers it. The resolutions are the author's. Two align the text with the operator's own words: Q2, agents install audited skills by default, and T7, the review "с подтверждением" binds. One follows the operator's direction, Q3: people confirm through the phone. Two choices stay with the operator (§17.3 items 13 and 14).

| Finding | Resolution in v3.5 |
| --- | --- |
| T1 the authority boundary was a list, and one committed transition passed it (blocking) | every change to the resolved configuration waits for a person: the authority digest covers the project file, the resolved templates with their package identities, the identities of the bound tool providers, the waggle roster, its revocation list and enrollment records, the recorded default branch and the confirmers, leaving out only descriptions, `ui` blocks, `x-` keys and roles' `prefer`; no list of harmless changes is attempted; confirmations are batched, one covering every pending change; the guard-bypass rule covers automatic entries triggered by a timer, `children.changed` or `blockers.terminal`; the authority report lists every difference and marks those that move work past a person, a tool or a guard. The choice between this default and a classified variant is pending (§17.3 item 13) (§0, §2.4, §10, §17.2 row 26) |
| T2 templates and hand-copied skills changed the process outside the digest | the digest covers resolved templates and their identities, and the identities of bound tool providers; installing a template-carrying skill, a new version of one or a new version of a bound provider waits for a person; an audit covers exact package identities; hand-copied role skills and templates are unknown sources, refused unless the operator's confirmed setting in `curator.toml` admits them; today's `.roles` overlays are a documented legacy path: the committed overlay is part of the resolved configuration, and `~/.roles` is read only for built-in roles by the default until PC2 (§2.1, §8.4, §8.6, §10, §14 point 14) |
| T3 the roster and enrollments decided who counts as a person, unconfirmed | the roster, its revocation list and the enrollment records enter the digest; a commit is verified against the roster of its parent revision, so no commit admits its own signer; verifiers re-check enrollment evidence against the platform vendors' roots; in waggle §3 a teammate is added by a confirmed commit instead of a reviewed pull request (§2.3, §2.4; waggle §3, §4.5, A1 §19.5) |
| T4 what the kernel ran while a change was pending was undefined | the whole version is the unit of waiting: the kernel runs the last confirmed version in full, from the resolved configuration it stored at confirmation, and every later commit waits with the head; approvals, grants and delegated acts bind the authority digest of the version in force (§2.4, §4.2, §6.4) |
| T5 who confirmed was unstated, and one board could run two processes | a person is a principal of the `people` group that `policy.confirmers` names, by default the operator who created the project; before CM2 each machine runs what its own confirmer confirmed, the board records every machine's confirmed digest and the kernel reports divergence, pending the operator's choice against single-host only (§17.3 item 14); from CM2 a confirmation is a signed board act, `board.confirm`, verified against the roster (§2.4, §6.8, §15; schema; cafe project; waggle A1 §19.1) |
| T6 a headless node confirmed through a key that shows nothing | before the app exists, a headless node is confirmed from the operator's workstation: the pending change is fetched over SSH, rendered and signed there with a key that needs user verification, and only the signature returns; the confirmation key is never used through a forwarded agent; on-machine confirmation means a local session; v3.4's terminal-session path is withdrawn (§2.3, §15; waggle A1 §19.5) |
| T7 the review of open-ended answers did not bind | aligned with "с подтверждением": an answer not re-confirmed within one period after its review stops applying until re-confirmed; the review covers every answer whose end lies beyond the next review; an exception from a standing answer always takes the role's scope and expiry, `until` bounding only when the answer applies; removing or lapsing an answer ends its exceptions, and a run already working finishes (§2.3, §9, §17.2 row 25) |
| T8 the first confirmation and the unconfirmed state were partly defined | a person confirms the first version, and a project an agent creates stays unconfirmed; until then it runs the built-in default, which grants nothing and needs no confirmation, and none of its own types, tools, schedules, automatic spawns, delegations, relaxations, standing answers or signed events take effect, while audited installs stay available; creating a project without git performs its first apply; the legacy read of today's model keys is the one exception to the confirmation rule for its release; the board records its default branch at creation, and changing it is a change to the process (§2.3, §2.4, §14 points 13 and 15) |
| T9 pairing lacked invariants | the node verifies the app key's attestation at every pairing and refuses a key without it; the QR secret is single-use, short-lived and never logged or put in a transcript; pairing refuses an agent session; initializing or re-initializing a node that has a registered key, and every later pairing, needs a confirmation; the key's own access control requires biometrics (§2.3) |
| T10 waggle A1 understated what it changes and lacked installs | A1 lists §3, §4.3, §4.4 and §4.5 among the places it changes and adds the companion provider and verifier to §4.5; `board.install` and `installs` join §19.1 and §19.8, with any principal of `people` as the approver set; `board.confirm` joins §19.1 (waggle A1) |
| T11 residuals of S2, S12 and S13 | no run writes an authority-bearing field of any element, its own or another; a level no form can satisfy yet is read as `user`, and `user` may lower it; a host key that signs hand-off results is held by an OS account other than the operator's or a privileged service, whether or not it is hardware-bound (§2.3, §5.2, §6.3, §6.5; waggle A1 §19.4) |
| T12 delegated relaxations and installs rules | exceptions created under a grant end with it, and a run already working finishes; relaxations rules may carry `relax` with roles and terms; the difference between a delegated relaxation and a standing answer under a signature policy is stated — a grant is a board record the board verifies, a standing answer a file entry only one machine can check; the schema refuses ceilings and `self_approval` on a rule that covers installs only (§4.2, §9; schema) |
| T13 wording | quorum: each delegated act counts as at most one principal, its grantor, and one delegate session counts once; grants and approvals react to confirmed changes, never to a pending commit; an unsigned grant still needs a person's confirmation, and only the attribution of acts before signatures is a guardrail; the authority report compares against the confirmed configuration, not a digest (§4.2, §6.4, §10; waggle A1 §19.1, §19.7) |
| T14 author resolutions recorded as the operator's | §17.2 rows 25–29 and the wiki mark each fill as the author's: the list of sections and the rest on commit (superseded, T1), an unreviewed answer staying in force (superseded, T7), `policy.enable.standing_answers`, `task-board playbook apply` (which replaces decision 2a's read-as-is), installs off by default (superseded, Q2) and the `installs` act, the terminal-session path (superseded, T6); §24's intro and rows point to v3.5 (§17.2, §24) |
| T15 drift | the track's status, PC0, PC1 and PC3 rows, dependencies and risks; the schema's version labels; §14 point 13 ("once committed and confirmed"); §8.3 lists five Curator changes; §2.3's file and command tables name `curator.toml` and `curator sources`; §0 names skill sources among the default confirmations; PC1 names the session host's marks as a follow-up to its as-is extraction, without which no delegated act counts; the wiki's records and open questions (§0, §2.3, §4.2, §8.3, §14, §15; track; schema; wiki) |
| Q2 the default for agent installs | aligned with the operator's words: `policy.enable.skills` defaults to `audited`; a project may restrict it to named skills or `none`; an installation that changes the resolved configuration still waits for a person (§0, §8.6, §6.8; schema) |
| Q3 companion signatures and `user_verified: required` | follows the operator's direction that people confirm through the phone: an attested companion key whose own access control requires biometrics meets `required` for board acts, and verifiers re-check its enrollment evidence (§4.2; waggle A1 §19.5) |

## 26. Change log: v3.5 → v4.0

v4.0 applies the operator's decisions of 2026-09-25 (§17.2, rows 30–34) and answers the refuter's sixth run ([`../reviews/2026-09-24-refuter-v5.md`](../reviews/2026-09-24-refuter-v5.md)), on v3.5. The trust machinery moves to [`trust.md`](https://github.com/relux-works/curator-trust/blob/main/spec/trust.md), which answers the findings that concern it (its §15); this core answers the rest. The resolutions are the author's except where a row names a decision.

| Finding | Where | Resolution |
| --- | --- | --- |
| U1 an agent could sign `board.confirm` from CM2 (blocking) | trust | only a key that requires a person's presence confirms; `policy.signatures` never governs confirmations and no project lowers the floor (principle a; trust.md §4, §6; §4.2) |
| U2 a batch could admit its own confirmer | trust | a confirmation is verified against the roster, revocations, enrollments and confirmers of the version it replaces, never the parent of the head (trust.md §4) |
| U3 "the pinned revision" was the unconfirmed head | core | the pinned revision is the revision of the version in force; the head is the proposed version, used only for the classification, the report and the confirmation (§2.4) |
| U4 `task-board.config.json` decided what ran, from the working tree | core | check sets named by the playbook, and the legacy `version_control` and model keys, are part of the resolved configuration, read at the proposed head and run from the version in force, never from the working tree or the tree under evaluation; the legacy model keys stay in force until a confirmed version replaces them, so migration opens no window (§2.3, §2.4, §6.12, §14 point 13) |
| U5 role defaults and admitted locations escaped the digest | core | each bound role skill's `data/role.json` contribution is part of the resolved configuration at its locked identity; an unknown source is admitted by content, never by location (§2.4, §8.1, §8.4, §8.6) |
| U6 the executor ran new provider code under the confirmed binding | core | provider identities come from the project's lock; the executor runs a tool only when the installed identity equals the one in force, and otherwise fails with `error` and reports the pending update (§2.4, §6.11, §8.6) |
| U7 an old record re-confirmed a reverted version | trust | records bind the board, the head, both configuration digests and a chain sequence; a record is accepted only when its replaced digest is the version in force; no revival (principle c; trust.md §4) |
| U8 a revocation waited for a confirmation | trust | a person's signed narrowing acts — revoking a key, a grant or a tool — take effect at once (principle d, row 33; trust.md §5; §4.2) |
| U9 per-machine versions contradicted §7.1 rule 6 | trust, core | one confirmation for every machine, which each machine verifies itself (principle e, row 32); every machine runs the stored version in force; the kernel resolves templates only from the project scope and its committed lock (§7.1 rule 6, §8.2; trust.md §4) |
| U10 the signer signed a digest the node computed | trust | the signing device receives the full configurations, computes both digests and the report, and signs only what it computed (principle b; trust.md §4, §8, §9) |
| U11 the first workstation key was unprotected | trust | the workstation path accepts only FIDO2 keys with required user verification and attestation; the first registration is local or attested; later ones are confirmed (principle f; trust.md §3, §9) |
| U12 confirmers were unenforceable and undefined | trust | confirmers are read from the version in force; a version never names its own confirmers; the board's anchor operator confirms the first version; recovery through a local re-anchor (trust.md §3, §4; §2.4) |
| U13 `x-` keys left out of the digest carried meaning | core | only the `x-` keys the kernel ignores leave the digest; an `x-` name the schema gives meaning — a custom capability, `risk` marker or signature category — is data (§2.2) |
| U14 an agent session could raise the confirmation prompt | trust | the confirm and grant commands refuse agent sessions; the prompt carries the short digest and the count of marked changes (trust.md §4; §2.3) |
| U15 the cafe's buyer could not write the required `amount` | core | the run working an element's initial state may write, while it is there, fields only delegation ceilings, conditions or budgets read; a grant's conditions read only fields its rule's ceilings read; values written before a field became authority-bearing stay and are reported (§4.2, §5.2, §13; cafe project) |
| U16 relaxation ceilings, mixed install rules, `board.install` | core | a rule that covers installs covers nothing else (schema); an exception created by a delegated relaxation takes the intersection of the grant's scope and the role's approval scope, and ceilings are checked each time it applies; a grant's `relax` lies within its rule's; a new signature category `installs` (§4.2, §9; schema) |
| U17 `.roles` overlays on boards running the built-in default | core | legacy mode: a board without a version in force reads its committed overlay and its legacy keys as today; the overlay joins the resolved configuration with the first confirmed version (§2.4, §14 points 14 and 15) |
| U18 every confirmation voided every unused approval | core | an approval binds the parts of the process it depends on; a version that changes none of them keeps it; the report lists the approvals and grants a pending change would void (§6.4, §10) |
| U19 companion signatures and iOS | trust | the attestation text aligned with waggle A1 §19.5; companion signatures meet the confirmation floor; for other board acts they are pending the operator (§17.3 item 15; trust.md §6, §8); the configurable review period is marked as the author's (§2.3, §9) |
| U20 where the confirmed configuration lives | trust | the version in force is stored on the board with its digest; every machine recomputes and verifies it at every load and fails closed; the executables join the open question on OS-level protection (trust.md §12; §17.3 item 3) |
| U21 drift | core, wiki | "What this means for PC1" widened (§17.3); the wiki's records annotated. curator-keeper's references to v3.5 are outside this change |
| D1 which changes wait (item 13) | core | the author's proposal: a default-deny allowlist with the path check, pending the operator (§2.4, §17.3 item 13) |
| D2 confirmations before CM2 (item 14) | trust | resolved by principle (e), row 32 (§17.3 item 14) |
| D3 companion signatures for board acts | trust | pending the operator (§17.3 item 15) |
| D4 a person's narrowing acts | trust | decided: they take effect at once (row 33) |
| Terms (row 30) | core | the term map opens the specification; §16 records the decision |
| Principles (row 34) | core | §1 |

## 27. Change log: v4.0 → v4.1

v4.1 applies the operator's decisions taken on the answers to v4.0 (§17.2, rows 35–39):

- capabilities, batched touches and yolo mode for item 13;
- companion signatures for board acts;
- the web of trust and multisignatures;
- the move of the trust specification;
- the analyst role that only explains;
- the privileged operations and the separate OS account;
- the confirmation of rows 30, 31 and 33.

It also answers the refuter's seventh run ([`../reviews/2026-09-24-refuter-v6.md`](../reviews/2026-09-24-refuter-v6.md)), on the core of v4.0. The run found that the allowlist was not bounded as claimed; the operator's decision on item 13 replaces it. The resolutions are the author's except where a row names a decision.

| Finding | Resolution in v4.1 |
| --- | --- |
| V1 the path check missed triggers, `limit.then`, state keys and owners (blocking for the allowlist) | withdrawn together with the allowlist (row 35). Capabilities are section-scoped, and workflow structure — states, owners, transitions with their triggers, guards, limits and state keys — is never covered, so each of the four probes waits for a person (§2.4, §10) |
| V2 work-in-progress limits on a person's states suppressed escalations | the `wip` capability covers raising limits and limits over agent-owned states, agent roles or the board only. A limit never refuses an entry into a human-owned state that a verdict, a `limit` redirect or an automatic escalation routes to. A helper's `min` gates the owner's exits, and lowering it or removing a human helper is never covered. `policy.goals` and `policy.capacity` are workflow structure (§2.4, §6.1, §6.8) |
| V3 allowlisted links carried no record | the version in force comes from history: from the last confirmation, each first-parent commit whose differences the grants in force cover takes effect, the first uncovered one is pending, and every machine re-derives each link from the stored configurations and the recorded grants, never trusting a stored claim; a confirmation replaces the derived version in force (§2.4; trust.md §4) |
| V4 capability selectors admitted installed role skills | a selector ranges over the role aliases of the resolved `roles` map in the version in force, never over installed skills no alias binds; adding an alias waits; in legacy mode the committed `.roles` overlay supplies the aliases (§6.1, §14, §15 PC2; schema) |
| V5 `task-board.config.json` set a delivery gate and the landing branch outside the resolved configuration | `run_write_boundary` moves to `policy.delivery.run_write_boundary`, with a one-release legacy read inside the resolved configuration; the integration base branch and the board repository are read from the version in force and must name the recorded default branch; every key of the file the kernel reads is part of the resolved configuration (§2.1, §2.4, §6.8, §14; schema) |
| V6 a pending provider update fired `tool.failed` | the executor runs the provider identity in force from Curator's store, so an install changes nothing that runs until a version that names it takes effect; an identity that left the store holds the invocation; `tool.failed` only for executions that ran (§6.11, §8.6) |
| V7 the PC1 demos had no source for their templates and role skills | the kernel ships the three demo templates and their role skills as built-ins from this repository, admitted like the default; PC2 publishes them as `curator-playbook-dev-cycle`, `-bug-hunt` and `-scrum` in the Curator registry (§8.2, §15; track) |
| V8 a version failing the portable checks could take effect | a proposed version that fails a portable check never takes effect, by capability or by confirmation; `task-board playbook confirm` refuses it (§2.4, §10) |
| V9 an allowlisted edit voided unused approvals silently | an approval binds the transitions that consume it with their targets and guards, not the whole workflow, and the fields declared when it was given; workflow changes wait and are reported (§5.2, §6.4) |
| V10 the validator's roster check read "the pinned revision" | the validator checks `people` against the roster of the same configuration; runtime acts read the version in force (§2.4, §4.2, §10) |
| V11 the PC1 stub: three boundary claims, the anchor, the principal of a CLI act | the three sentences are qualified ("from PC3; in PC1 a guardrail"); `task-board trust init` without keys joins PC1; every unsigned act outside a run carries the anchor operator's principal; a demo commits a roster or names its principals; PC1 takes versions only on a single-host local board (§0, §2.4, §4.2, §15; trust.md §13) |
| V12 a template identity waited while a role skill's instructions did not | a template-carrying skill is judged by its resolved content: its identity is recorded, not a waiting difference; audited instruction-only updates take effect under the `skills` capability, and tool providers under `tools:<name>` (§2.4, §8.1, §8.2, §8.6) |
| V13 a grant's budget could make another field authority-bearing | a budget sums only a field its rule's ceilings read (§4.2) |
| V14 the kernel ran the version in force without narrowing acts | the kernel runs `narrowed(version in force)`, and the executor and `policy.enable.tools` read it (§2.4, §6.11) |
| V15 author fills recorded as the operator's | the operator confirmed the English names, principles (f), (g) and (i), and the three narrowing acts on 2026-09-25 (rows 30, 31, 33) |
| V16 drift | the `confirmers` default is the board's anchor operator; the schema's `policy.enable` and `policy.delegation` descriptions; row 26; legacy mode is "the one place where anything is read from the working tree"; "external participants" without its user-facing name in trust.md (§6.8, §14, §17.2; schema; trust.md §13) |
| D1 item 13 | decided: capabilities, batched touches and yolo mode (row 35) |
| D2 the change analyst | decided: it explains, never decides (row 38) |
| D3 root skill updates | the author's resolution, recorded for the operator: an audited update whose template content or role contribution is unchanged takes effect under the `skills` capability, which follows `policy.enable.skills`; a change of content waits (§2.4, §8.2) |
| D4 rows 30, 31 and 33 | confirmed (V15) |

## 28. Change log: v4.1 → v4.2

v4.2 records the operator's discussion of 2026-09-25 after v4.1. The resolutions are the author's except where a row names a decision.

| Change | Where |
| --- | --- |
| **Decided:** role skills reach their runs through per-role Curator profiles; on-demand activation and the skills launch channel are withdrawn, and curator-spec issue #91 and curator issue #91 are closed (§17.2 row 40) | §0, §8.1, §8.2, §8.3, §15, §17.3 items 8 and PC1's gates; track; README |
| Curator's third change is now command-only installs (curator-spec issue #95, curator issue #95); the installation proposals #92, #93 and #94 are noted | §8.3 |
| **Proposed:** dream mode — a tool contract with `--dream`, the executor that adds it and refuses tools without support, a sandbox on a copy of the board, the agent told it is dreaming, dream evidence that never counts, no real secrets or privileged operations | §0, §6.11, §6.13; trust.md |
| **Decided to come after PC1 (row 41):** a decision consistency check — an optional `formal` field on decisions, a `decision-check` tool skill with an SMT solver, the unsatisfiable core as the scope of affected decisions and the minimal correction sets as resolution options, a tool-owned `consistency-check` state; and the `properties` block of item 17 | §17.3 items 17 and 18 |
| **Proposed:** in a dream, read-only access to the orchestrator's previous sessions through a deterministic log distiller that removes noise without summarizing and keeps level-0 identifiers; a quarantined dream memory that the system roles turn into proposals — playbook improvements under a batched touch, apiary improvements as change sets applied with the person's consent; the dream as the node's nightly offline cycle | §0, §6.13; trust.md §20; curator-keeper §13.6 |
| dev-cycle's rollback loop — development → review → accepted → development through `delivery.rework-started` — had no bound; the rollback transition now carries `limit: { "max": 2, "then": "escalated" }`, so the third rollback of one element's accepted work goes to the operator (found by the wiki research note on verifiable process properties) | `examples/dev-cycle.playbook.json`; §17.3 item 17 |
| Open: dream mode details | §17.3 item 19 |

## 29. Change log: v4.2 → v4.3

v4.3 records the operator's answers to v4.2 of 2026-09-25. The resolutions are the author's except where a row names a decision.

| Change | Where |
| --- | --- |
| **Decided (row 42):** loop limits are tunable per project, down to unbounded. **Proposed form:** a template declares `params` with a type (`limit`, `count`, `duration`) and a default; `limit.max`, `on.after` and the counts of `policy.wip` may reference them as `$name`; a project sets plain values, qualified in a composition. A parameter's value is part of every place that references it for §2.4 | §2.4, §3, §6.2, §6.8, §7.3; schema |
| `limit.max` accepts `none`, an unbounded loop recorded as a choice, with `then` still declared | §6.2, §7.3; schema |
| The validator checks parameters and warns about unbounded cycles, an error only when a project's `properties` require bounds | §10, §17.3 item 17 |
| dev-cycle declares `rework_limit` and `rollback_limit`, unbounded by default as today (replacing v4.2's fixed bound of two rollbacks), and `block_limit` 3; the scrum project sets `dev-cycle.rework_limit` to 30 | §13; `examples/dev-cycle.playbook.json`, `examples/scrum/project.Playbook.json` |
| **The operator's direction (row 43), proposed form:** entering a dream is approved once per process through the capability kind `dreams`, scheduled and conditioned by `policy.dreams` (windows, a per-night budget, idleness, quota or a ready local engine, AC power); the host parks the session, starts a dream session with the role's dream profile, and resumes or leaves it parked afterwards; a real high-priority doorbell wakes the dreamer; the main loop schedules dreams within the node's capacity | §6.8, §6.13; schema; trust.md §16; curator-keeper §13.6 |
| Open: dream scope, sandbox lifetime, rehearsal before structure changes, packaging of dream profiles | §17.3 item 19 |

## 30. Change log: v4.3 → v4.4

v4.4 applies the command-line shape the operator adopted on 2026-09-27 (wiki `research/cli-shaping-review.md`). It renames commands and moves no rule.

| Change | Where |
| --- | --- |
| The validator is the Curator provider `curator playbook`: `check` (was `playbook validate`), `explain` for the authority report of a working tree, `bind <alias> <skill>`; no bare `playbook` command | §0, §2.1, §2.4, §10, §15 |
| A person confirms a version with `task-board trust confirm` (was `task-board playbook confirm`, kept as an alias for one release); `--node <node>` (was `--remote <node>`) runs the workstation path; every mutating `task-board trust` verb refuses agent sessions; `keeper confirm` fronts it in an apiary | §2.3, §2.4, §10, §15 |
| `task-board config propose` (was `task-board playbook apply`) proposes a version without git; `task-board config status` shows the version in force, the proposed one, their differences and the grants that cover them; `task-board config migrate` moves the old model keys and absorbs `migrate-project-config` | §2.3, §2.4, §10, §15 |
| One command family per operator file, each a Curator provider named after its noun: `curator models` (`answer` and `review` replace `answers` and `answers review`; `edit` added), `curator runtimes` (`check --probe`, `limits`, `edit`), `curator engines` (was `curator inference engines`; `check`, `confirm` and the engines' lifecycle), `curator network` (`check --probe` replaces `test`; `confirm`; `exec`) | §2.3, §9 |
| Adding an engine or a network profile is a widening entry, confirmed like the others | §2.3 |
| Curator's settings, the skill sources included, stay in Curator's existing configuration; `curator.toml` is dropped; `curator sources add` and `confirm` replace `curator audit --allow` | §2.3, §8.3, §8.4, §8.6 |
| The tick timer's installer is `task-board tick --install-timer` | §6.9, §15 |
| The per-role Curator profile at spawn is `spawn --profile`, with `spawn.profile` in the configuration (was `spawn.context_profile`); the person's spelling of a command-only install is `curator global add <skill> --commands-only` | §8.1, §8.3 |
| Beekeeping words only in `keeper`'s nouns and the apiary's directories | §16 |

## 31. Change log: v4.4 → v4.5

Proposed on 2026-09-28 from the landing work of the cocoaskills orchestrator, where two releases shipped with a red trunk and changelog conflicts were resolved by hand:

| Change | Where |
| --- | --- |
| Trunk verification after landing: `after_landing` hosted sets, the element waits for a green trunk pipeline on the landed head, red opens a linked Bug | §6.12 |
| Append-only files merge with git's `merge=union` from the project's `.gitattributes`; the landing evidence lists them | §6.12 |
| Hosting providers `github` and `gitlab` with one contract; the first implementation is the `delivery` tool in relux-works/skill-delivery | §6.12, §6.11 |

## 32. Change log: v4.5 → v4.6

Proposed on 2026-09-28 from the music-video example:

| Change | Where |
| --- | --- |
| Tool outputs: typed fields a tool reports and only the kernel writes | §6.11.1 |
| Receipts: the amount actually charged reconciles the delegation budgets; a missing receipt is an unknown outcome, never free | §6.11.1 |
| Assisted tools: a tool may ask a person mid-run and wait; a tool that uses a person's own session runs under that person's account | §6.11.1 |
| Open item 20 | §17.3 |

## 33. Change log: v4.6 → v4.7

Proposed on 2026-09-29 after testing the model against published AI video pipelines: the music-video example, a one-prompt chapter-animated video, a skill that produces motion-graphics videos in waves, an agent 3D video toolkit, and three catalogs of cases (wiki research). The model held: every pipeline's stages, roles, judges, tools, spending and loops were expressible. The gaps were about artifacts, long-running tools, units, the outside world and review.

| Change | Where |
| --- | --- |
| File fields with content digests and part sets; approvals bind content; provenance and licence; pinned inputs, `inputs.fresh`, `input.superseded`, reopening, impact before a swap, superseding as a link, reuse by content | §5.5 |
| Tools run on a pinned tree; progress, resume and late results; machine resources as leases; quality tiers; sandboxes for tools that run element content; readiness probes | §6.11.2 |
| Budgets in money, tokens, minutes and credits, per container and per state; measured estimates; parameters as guard thresholds and tool settings; grants tied to a plan; waves | §6.14 |
| `publish` risk, absolute-time triggers, external data with freshness, metrics as signed events, intake from events, per-platform versions; platform work in a publishing module | §6.15 |
| Typed findings, the unverified list, seams, write scopes, `vision`, `video-input` and `audio-input` needs | §6.16 |
| Open items 21–25 | §17.3 |

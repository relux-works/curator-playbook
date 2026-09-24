# Process configuration: specification

Status: **DRAFT v3.4**, 2026-09-24. Not normative until frozen: the freeze follows a check of the changes of v3.3 and v3.4 (§23, §24). Roadmap: `wiki/roadmap/ecosystem-roadmap.md` (process-configuration line). Track: [`track.md`](track.md). Schema: [`../schema/playbook-v1.schema.json`](../schema/playbook-v1.schema.json). Examples: [`../examples/`](../examples/).

A team's process is code today. task-board's element types, statuses, gates and roles are compiled into its kernel, so a team that works differently — or a cafe ordering supplies, or a group formalizing a proof — cannot describe its process without changing the board. This specification answers one question: how does a project state who does what, which kinds of work exist and how each moves, in one file that every teammate can share and a validator can check before anything runs? It covers that file, its schema, extension and composition, distribution through Curator, model needs, and validation. The agent loop, an always-on scheduler and the goal ledger stay outside it (§1).

v3 applies the operator's decisions of 2026-09-24 on every point v2.1 left to a person. The operator confirmed the author's resolutions of both refuter runs, decided the proposed defaults H1, H2, H3, H5 and H7 (H2 more strictly than proposed), and added standing answers, speed needs, field conditions, work-in-progress limits and check sets. §17 lists the decisions and what stays open, and §20 maps each decision to the sections it changed. §19 and §18 keep the mapping of the second refuter run ([`../reviews/2026-09-24-refuter-v1.md`](../reviews/2026-09-24-refuter-v1.md)) and the first ([`../reviews/2026-09-24-refuter-v0.md`](../reviews/2026-09-24-refuter-v0.md)). v3.1 answers the third refuter run, on v3 ([`../reviews/2026-09-24-refuter-v2.md`](../reviews/2026-09-24-refuter-v2.md)), and §21 maps each of its findings to a resolution. v3.2 records the operator's answers to the two points that run left open: a project may hard-restrict the models used on it (P2), and entries of the operator's files that widen admission take effect only after the operator confirms them through the operating system's user authentication, with the operator-file commands decided (P3). It keeps the formal-missions example (P4) and adds delegation: a person may delegate named human acts to an orchestrator within limits the project sets, and validates only what the delegation leaves out. §22 maps each to the sections it changed. v3.3 answers the fourth refuter run, on v3.1 and v3.2 ([`../reviews/2026-09-24-refuter-v3.md`](../reviews/2026-09-24-refuter-v3.md)), and §23 maps each of its findings to a resolution. v3.4 records the operator's answers that followed: open-ended standing answers are allowed, with a periodic review; the authority-bearing sections of `Playbook.json` take effect only once a person confirms them; a person confirms through the machine's user authentication or a companion app paired with the node, the common case on a headless node; rule changes are commits, and pull requests are never required; agents may install audited skills, while skills from unknown sources are refused by default. §24 maps each to the sections it changed.

Every statement in this revision is *decided (2026-09-24)* unless §17.3 lists it as open or it is marked *future*. The answers to the third and fourth refuter runs (§21, §23) are the author's: each aligns the text with a recorded decision, the code or this specification's own rules, and the operator confirms them with the freeze. The changes of v3.2 and v3.4 (§22, §24) apply the operator's decisions; where v3.4 had to fill a gap in them, §24 says so.

Supersedes the roles-and-routes model of `skill-project-management/.specs/drafts/playbooks.md`: the core object is no longer a list of roles with routes between them but entity types with workflows whose states are owned by roles. Related: `skill-project-management/.specs/drafts/agent-selection.md` (selection layers; §9 amends its §3, §4, §7 and §8), `skill-project-management/.specs/drafts/launch-profiles.md` (LP; §9 amends D1's model admission and D9 for the case without an operator file, §2.3 moves D2's runtime catalog), curator-spec `protocol/skillfile-sources.md` (Skillfile schema 2, package identities), curator-spec issue #91 (on-demand skills and the skills launch channel, §8.1), `protocol/core.md` §3, §4.1 and §4.4 (execution boundary, command shims, skill dependencies), `protocol/environments.md` (package-influence boundary), `profiles/manager.md` (scope precedence), `waggle/spec/waggle.md` (approver signatures, hand-off offers, scope leases, the roster at a pinned revision).

---

## 0. The design on one screen

- A project describes how it works in one file, `Playbook.json`, in its root beside `Skillfile.json`. *decided*
- The file has four core parts: **roles** (aliases bound to role skills, people, tools or other processes), **entities** (element types, nesting, fields), **workflows** (states and transitions) and **policy**. *decided*
- **Each element has exactly one owner in each state; a state may admit several roles and assignment picks one**; helpers work for the owner. *decided (H8)*
- **Playbooks are data only.** They never name a program, script, webhook or URL. Everything that executes is a skill installed through Curator: a tool is a command that a provider skill exports, and the orchestrator skill reaches it through a runtime dependency narrowed to that command. Schedules, automatic spawns and tool invocations a package proposes take effect only when the project enables them; a tool runs only when the project lists it. *decided*
- **Authority stays outside the model.** Agents move the steps a playbook gives them; which steps belong to people is the playbook's choice, and signatures apply only to the acts a project marks. An agent run never moves work out of a human-owned state, never records a human approval and never posts a tool or external event, except as a person's **delegate**: a person may delegate named acts to their own orchestrator sessions within limits the project sets, and validates only what the delegation leaves out. Risky human acts need a waggle approver signature that shows a person acted. *decided*
- **The process in force is the committed one.** The kernel reads the project's `Playbook.json` from the head of the default branch of the repository that owns the board, never from the working tree, so no session can edit its own authority. A commit is enough; git hosting and pull requests are optional. The sections that grant authority take effect only once a person confirms them, on the machine or through a companion app paired with it. *decided*
- A project **extends** one published playbook and writes only its differences, and **composes** several, binding their slots; its `extends`/`uses` is its list of admitted playbooks. *decided*
- **Roles are skills.** A published playbook is a repository whose `skills/` collection holds the root orchestrator, whose data carries the template, and one skill per role. One Skillfile schema 2 selector installs the whole process; tool providers sit outside the collection. Role skills are installed on demand and reach only the launches that run their role. Agents may find and install audited skills from the Curator registry where the project allows it; skills from unknown sources are refused by default. *decided*
- Roles state **needs** — requirements and preferences in one vocabulary, speed included — never models or efforts; a project may hard-restrict which models run on it; the operator's private files bind models and effort, and what widens admission there — metered billing and standing answers by default, any other kind the operator chooses — takes effect only after the operator confirms it; for the whole team only the project relaxes a hard need permanently, and an operator may answer an `ask` in advance, open-endedly for their own runs, under a periodic review. *decided*
- **Guards read field values**, so review levels and thresholds are routes, and the fields they read are guarded like authority; **policy bounds the elements in progress**; and **check sets** — local, on the project's CI, or both — supply the evidence for Change Request work. *decided*
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
| `Playbook.json` | how the project works | the board (task-board today) and the validator | admitted playbooks, role aliases bound to skill names, tool bindings, people, event signer groups, entities, workflows, policy: review, work in progress, version control, signatures, delegation, data (model admission included) |
| `task-board.config.json` | where and with what the board runs | the board | board location, integration base branch, board repository, check sets (§6.12), team runtime names, run supervision |

All three are committed. Process rules that sit in `task-board.config.json` today move to `Playbook.json`: the version-control policy (commit confirmation and time); the old keys are read for one release with a warning *(decided 2026-09-24)*. Model admission moves to `Playbook.json` as `policy.data.models`, where a project may hard-restrict the models used on it (§9) *(decided 2026-09-24, P2)*: today's `spawn.ceilings` with their per-role replacements, the runtime set of `preferred_agentic_system` in either form, and the schedule windows that change that set by time of day. Model choices — bindings, workload classes, the candidate order and preferences — leave both committed files for the operator's `models.toml`. The mapping of each key is in §2.3; that the runtime set, the role ceilings and the windows stay project admission is the author's resolution (§23, S8). Run supervision (today `spawn.ceilings.<runtime>.supervision`) stays in `task-board.config.json` as `spawn.supervision`. Machine limits leave the committed files: today's runner limit `spawn.max_parallel` and suite cap `spawn.validation.max_concurrent_suites` become machine capacity in `runtimes.toml` (§2.3). The process's own limit on work in progress is `policy.wip`, which counts elements (§6.8).

**When the board and the code live in different repositories** — a shared product board over several code repositories, or a fork whose upstream owns the board — `Playbook.json` lives in the board repository, and its default branch is the one §2.4 reads. Each Story's check sets come from its code owner's `task-board.config.json` (§6.12), and templates resolve in the Curator scopes of the board checkout (§8.2).

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
| `runtimes.toml` | this machine's runtime bindings (harness binaries, accounts, homes) and its capacity: parallel runs (today's `spawn.max_parallel`, default 20), concurrent local check sets (today's `spawn.validation.max_concurrent_suites`, default 2), rate-limit ladders | the launch plane; the kernel, for machine capacity |
| `engines.toml` | local inference engines and their models | curator-inference-manager |
| `network.toml` | network profiles | curator-network-profiles |

They are TOML because people edit them by hand, and they are never committed. Keeping them together amends LP D2, which placed runtime bindings in `$XDG_CONFIG_HOME/agents-management/runtimes.toml`, and the network-profiles specification, which placed its profiles in `network.json` under `$XDG_CONFIG_HOME`. The operator-file commands (below) edit them safely, and they are the only way an agent changes them.

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
- `[allow]` narrows which models may run at all, and `billing` lists the billing classes this operator pays for. Only this file admits `metered` billing, and only once the operator has confirmed it (below): a project may narrow billing classes, never widen them (§9). `[windows]` limits when a model may run.
- `[answers]` holds standing answers to `ask` (§9). Keys follow `[bindings]`; `relax` names the requirement terms the answer relaxes; `projects` limits it to the named projects. An applied answer takes the role's approval scope and expiry (`needs.approval`, or for a role without it the exception defaults of §9) unless `until` sets an end date or says `open`. An open-ended answer is written explicitly, `until = "open"`, and is allowed *(decided 2026-09-24)*: it relaxes the requirement at every future `ask` of its role for this operator's runs, each application appears on the board as an exception, a project can forbid standing answers (`policy.enable.standing_answers`, §9), and a periodic review asks the operator to re-confirm it (§9). A standing answer applies to this operator's own runs only, never widens `[allow]`, and takes effect only once confirmed (below).
- `[review]` sets how often open-ended standing answers are listed for re-confirmation: `answers`, an ISO 8601 duration, one month by default (§9). Lengthening it is a widening entry of kind `answers`.
- `[confirmation]` says what each kind of widening needs before it takes effect (below).
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

  Until migrated, the old keys are read as before with a warning — the ceilings, the runtime set and the windows as project admission, the rest as a legacy operator layer — and a migration command previews every destination and writes them.
- Machine limits never sit in a committed file, because teammates run different machines: parallel runs, concurrent check sets and rate-limit ladders live in `runtimes.toml`, with today's defaults of 20 parallel runs and 2 concurrent check sets. For one release, a `spawn.max_parallel` or `spawn.validation.max_concurrent_suites` committed in today's `task-board.config.json` is read as the machine's capacity where `runtimes.toml` does not set it, with a warning. The process's own limits on work in progress are `policy.wip` (§6.8).
- Every choice, from the file or not, must satisfy the role's needs (§9), else `on_unmet` applies.
- Tool bindings, people and event signers are not in these files: they are project facts and live in `Playbook.json` (§4.2, §8.5).

**Widening entries take effect only after the confirmation their kind needs** *(decided 2026-09-24, P3)*. These files sit in the operator's home, and any process running as the operator's OS user can edit them: an agent stuck on an `ask` could answer it for itself, or admit metered billing. Reading the committed process from the default branch (§2.4) does not reach them. So:

- The resolver keeps the **effective version** of each file: the last confirmed version, narrowed by every narrowing edit made since. A narrowing edit applies at once and becomes the new baseline without authentication, so an entry the operator removed stays removed until the operator confirms it again. A change that admits more than the effective version is a **widening entry**: a model or a billing class added to `[allow]` (metered billing included), a `[bindings]` candidate outside the effective admission, a standing answer added or extended, a window widened, a runtime binding added to `runtimes.toml`, a skill source outside the Curator registry allowed in Curator's settings (§8.6), a confirmation level lowered. For each kind of widening whose level (below) is not `none`, the resolver keeps applying the effective version until the operator confirms, so restoring an entry after a narrowing needs the authentication again. Before the first confirmation the baseline is the file-less one of §9: no metered billing and no standing answers.
- The operator confirms with `curator models confirm` (and the same verb for the other files), which shows the widening entries and asks for a person's confirmation (**How a person confirms**, below). The command refuses inside an agent session.
- The confirmation record is a signature over the digest of the confirmed entries, made by a key whose every use needs that confirmation, registered once through the same confirmation. The resolver verifies the record before it honours a widening entry; a record that does not verify leaves the baseline in force for the widening kinds.
- `[confirmation]` sets, per kind of widening (`billing`, `models`, `answers`, `windows`, `runtimes`, `skills`), what it needs: `none`, `user` (a person's confirmation, below) or `second-person` (the confirmation of another operator, for a machine several people administer). The defaults are `user` for metered billing, standing answers and skills from unknown sources (§8.6) and `none` for the others; a kind whose level is `none` takes effect at once. The `skills` kind cannot be lowered below `user`. The table sits in `models.toml` and governs every file of the directory; the launch plane reads it for `runtimes`. It is itself a widening entry when it lowers a level, and lowering a level needs the level it lowers.
- `second-person` becomes available only when a second operator's confirmation can be produced and verified: a waggle-signed act, which arrives with the waggle amendment (CM2). Until then the operator-file commands refuse the level and name the milestone, so no process can set a level that nobody can satisfy.
- A runtime binding's billing class comes from facts — how the harness is logged in (§9) — never from what the binding says, so a `runtimes` level of `none` never admits metered billing.
- This stops an agent that cannot answer the authentication prompt. How the registered key and the confirmed versions are protected from replacement by a process running as the operator belongs to the open question on OS-level protection of configuration files (§17.3).
- Until the commands ship, the resolver honours no widening entry whose level is not `none`, and reports each one it ignores.

**How a person confirms** *(decided 2026-09-24)*. Every confirmation this specification asks of a person — a widening entry of the operator's files (above), the authority-bearing sections of `Playbook.json` (§2.4), a delegation grant (§4.2), a skill from an unknown source (§8.6) — is made in one of two ways:

1. **On the machine**, through the operating system's user authentication: LocalAuthentication on macOS (Touch ID or the account password), Windows Hello on Windows, polkit or PAM on Linux, or, on any system, a FIDO2 key with user verification. The key that signs the confirmation record needs that authentication for every use: a Secure Enclave key with user presence, a Windows Hello key, a key released through polkit or PAM, or the FIDO2 key.
2. **Through the operator companion app** on a phone, the way a headless node is confirmed, and the common case:
   - The app holds a key in the phone's secure hardware (Secure Enclave on iOS, Android Keystore) that it uses only after the person authenticates on the phone with biometrics.
   - The app is paired with the node when the node is initialized, through an out-of-band handshake. The initialization prints a QR code in the terminal where the operator runs it, locally or over SSH; the app scans it, and each side pins the other's key. Only someone who sees that terminal can pair, and pairing another phone later needs a confirmation from an already paired one or on the machine.
   - Requests travel over an authenticated channel that resists a man in the middle: the operator's tailnet, or a relay that carries only messages encrypted end to end to the pinned keys, so the relay can read and forge nothing.
   - The app renders what is being approved — the entries, the sections and their digest, the grant and its limits — before it signs, and the signature covers exactly what it rendered.
   - A confirmation from the app is recorded like one made on the machine: a signature over the confirmed digest by a key registered at pairing.

The app's design, its transport (a tailnet, a relay, push to wake the app), its platforms and whether it also carries administration of the node over SSH are open (§17.3). Until the app exists, confirmations are made on the machine.

**Commands** *(decided 2026-09-24)*. One command family through the `curator` umbrella edits these files:

| Command | File | Verbs |
| --- | --- | --- |
| `curator models` | `models.toml` | `list`, `show`, `set`, `unset`, `allow`, `answers`, `explain`, `init`, `confirm`, `check` |
| `curator runtimes` | `runtimes.toml` | `list`, `show`, `add`, `remove`, `check`, `confirm` |
| `curator inference engines` | `engines.toml` | `list`, `show`, `add`, `remove`, `bench` |
| `curator network` | `network.toml` | `list`, `show`, `add`, `remove`, `test` |

- Every command keeps comments and order, validates before writing, writes atomically with a backup and mode 0600, never stores a secret value (only an environment-variable name or a command that yields the secret), and offers `--json`. From inside an agent session it refuses any change that widens admission.
- `curator models explain` prints, for the current project, each role's model and effort and the layer that decided each. `bench` records the local speed facts of §9. `init` is the first start (§17.3).
- Each file keeps one owner: `models.toml` the selection resolver, `runtimes.toml` the launch plane, `engines.toml` curator-inference-manager, `network.toml` curator-network-profiles. Curator provides the umbrella and the directory.

### 2.4 Which revision is in force *(decided 2026-09-24, H1)*

The orchestrator's session works in the project checkout with ordinary file tools. If the kernel read `Playbook.json` from the working tree, one unreviewed edit could drop a signature requirement, a hard need or a human owner (refuter v1, N1). waggle meets the same problem for its roster by reading it "at a pinned revision" (waggle §3). So:

- The kernel reads the project's `Playbook.json` from the committed file at the head of the default branch of the repository that owns the board, whatever Story is in scope. It never reads the working tree. One board therefore has one process in force at a time, and no branch or fork runs rules of its own.
- It records that revision, the **pinned revision** the rest of this specification refers to, and the file's semantic digest on the board at every load, beside the resolved templates' identities and digests (§7.1).
- **A commit is enough** *(decided 2026-09-24)*. A change takes effect when it is committed to the default branch. Git hosting is optional and a pull request is never required: a project that reviews changes through pull requests lands them as commits like any other, and a project with no hosting commits locally. The price is that trying a process change needs a commit. `playbook validate`, previews and a visual constructor read the working tree for that; the kernel never does.
- **The authority-bearing sections need a person** *(decided 2026-09-24)*. The sections that grant authority are `people`, `services`, `policy.signatures`, `policy.enable`, `policy.data` (model admission included), `policy.delegation`, each role's `needs.require` and `on_unmet`, `tools`, and `extends`/`uses`. They take effect only once a person has confirmed the digest of those sections in the committed version, the **authority digest** (§2.3, **How a person confirms**). Until then the kernel keeps applying the last confirmed authority sections, reports the pending change on the board and notifies the operators, so an orchestrator that commits and lands its own changes cannot raise its own ceilings. A project's first version is confirmed when the project is created; until a first confirmation exists, the kernel runs no tool, honours no delegation and no relaxation, and reports the project as unconfirmed. A change to the other sections — entities, workflows, states and transitions, a role's skill and `prefer` — takes effect on commit, and an orchestrator may make it within the project's delivery and its delegations.
- **Workflow changes that take authority from people count as authority-bearing too** (the author's resolution, §24): a change that removes a human-owned state or gives it another owner kind, removes an `approvals` guard or a `risk` marker from an entry, loosens a `field` condition that routes to a person, or removes a delivery guard. The kernel detects them by comparing the resolved workflows of the committed and the confirmed versions, and they wait for the confirmation with the rest. A change that depends on a pending authority change — a state owned by a role whose requirements are not yet confirmed, a transition that names a tool not yet confirmed — waits with it.
- When the project has a waggle roster, the kernel also verifies the signature of the commit it read against the roster at that revision.
- A project that is not a git repository has no default branch. It applies a new version with `task-board playbook apply`, which records the file and its digests on the board and asks the same confirmation for the authority-bearing sections; the kernel reads the version last applied. Before the first apply it reads the file as it is and warns that its authority policy is a guardrail only.

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

`name` is required for a published template and optional in a project file. `extends` and `uses` are covered in §7, `slots` and `bind` in §7.2, `tools` in §8.5, `people` and `services` in §4.2, `events` and `schedules` in §6.9. Keys that only a project may set — `people`, `services`, `policy.enable`, `policy.effort_floor`, `policy.signatures`, `policy.delegation` and `policy.data` — are ignored with a warning when they appear in a template. A template may bind a tool only under the rule of §8.5.

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
| `overrides` | a human leaving a tool- or process-owned state (rule 2), or authorizing a landing whose hosted checks could not run (§6.12) |
| `human-transitions` | every transition a human actor fires out of a human-owned state |
| `delegations` | granting or revoking a delegation (below); covered whenever the project sets a signature policy, listed or not |

- `min_assurance` is `software`, `hardware-bound` or `attested` (waggle §4.5). Whenever a project sets a signature policy it defaults to `hardware-bound`, because an agent running as the operator's OS user can obtain `software` signatures from `ssh-agent` freely.
- `user_verified` is `required` or `if-supported`. Only a FIDO2 signature whose user-verified flag is set, or a signature from the operator companion app (below), shows that a person was there; `hardware-bound` signatures are available to an agent whenever the key does not demand presence (waggle §4.4). A key in a Secure Enclave cannot be copied, but its signature does not show whether a person confirmed it. *Decided (2026-09-24, H7):* whenever a project sets a signature policy, `user_verified` defaults to `required` for approvals, relaxations, halts and delegations, and to `if-supported` for spending, overrides and human transitions; a project sets it for every category at once or per category, and one whose operators hold keys that cannot show user verification lowers it to `if-supported`. With the default `min_assurance`, `if-supported` admits Secure Enclave and other hardware-bound keys, never software keys. A signature from the operator companion app (§2.3) also meets `required`: its key is enrolled with attestation that a genuine companion app holds it in the phone's secure hardware, and the app uses it only after the person authenticates on the phone and sees the rendered payload. Here the user verification is enforced by the attested app rather than written into the signature, and the key's enrollment record says so (waggle A1 §19.5).
- Approvals are signed only through the operator-facing command that renders the payload before signing ("what you see is what you sign", waggle §4.2); an agent never holds a key.
- waggle has no class, namespace or principal kind for board approvals, signed transitions or signed events yet. The waggle amendment adds them (§17); it is written into the waggle specification now and implemented with milestone CM2. Until then signed human acts work only in the forms waggle already defines, and the cafe demo waits (§15). A category in `require_for` that has no signable form yet makes the states it covers unleavable, so the validator warns about it and names the milestone that brings the form (§10); the warning also covers `delegations`, which a signature policy includes whether it is listed or not, because before CM2 it makes every grant impossible. The PC1 demo projects set no signature policy.

**Delegation** *(decided 2026-09-24)*. Autonomy comes from the playbook's design, and the authority rules do not take it away. Agents move the steps the playbook gives them, a person owns only the steps the playbook gives a person, and a signature is needed only for the acts the project marks. Where a playbook keeps an act with a person, that person may delegate it to their own orchestrator sessions and keep only what the delegation leaves out, such as spending above a limit or a decision that is critical to the work.

- **What may be delegated** is project policy: `policy.delegation`, project-only and authority-bearing (§2.4). Without it nothing may be delegated. No delegation ever covers a confirmation: the authority-bearing sections of `Playbook.json` (§2.4), a widening entry of the operator's files (§2.3), a skill from an unknown source (§8.6) and a grant itself are always confirmed by a person, so no delegate can widen the authority it acts under.

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

  An `allow` rule names the acts, the agent roles they may go to (`to`: roles with capability `orchestrator`) and hard ceilings as `field` conditions that every delegated act must meet. The acts are `approvals` of a named human role, optionally only in the listed states, `transitions` out of named human-owned states, `relaxations`, meaning the approval of `ask` exceptions, and `installs`, meaning the installation of audited skills from the Curator registry into the project (§8.6). States are named `<workflow>.<state>`, so a rule never reaches a same-named state of another workflow, and every type a rule reaches must declare the fields its ceilings read (§10). A `never` rule names acts that no delegation may cover, for the roles it lists or for any role, and `never` wins over `allow`. `self_approval: true` on an `allow` rule lets a delegate approve an element that it or its runs filled in (below); without it, it may not.
- **A person grants a delegation.** The kernel record `delegation`, available in every playbook like `exception` (§9), is created through the operator-facing command with a person's confirmation, on the machine or through the companion app (§2.3), and signed whenever the project sets a signature policy (category `delegations`). A person who asks an orchestrator in plain words to "let the cafe approve orders up to 500 EUR" gets the prepared grant to confirm; the orchestrator cannot record it without that confirmation. It records:
  - the grantor principal and the delegate role;
  - the sessions that may act under it: by default only the grantor's own orchestrator sessions of the delegate role, and `sessions` may name other session principals explicitly, never a pattern;
  - the acts; for relaxations, optionally the asking roles and the requirement terms it may relax (`relax`); for installs, optionally the skills it covers;
  - `field` conditions, the policy's ceilings or stricter ones;
  - a subtree scope: an element the grant names, or the project;
  - an expiry;
  - optional budgets per period: a sum of a declared field, or a count of acts, over a duration.

  ```json
  { "grantor": "op:alice@cafe", "to": "cafe-orchestrator", "acts": ["approvals"], "role": "manager",
    "where": [ { "guard": "field", "name": "amount", "lte": 500 } ],
    "budget": [ { "sum": "amount", "max": 2000, "per": "P1W" } ],
    "scope": "project", "expires": "P90D" }
  ```

  Here a cafe manager lets their own orchestrator sessions approve orders of up to 500 EUR each and 2,000 EUR a week, within the project's ceiling of 1,000 EUR.
- **Who may grant and who may revoke.** A person may grant only acts their own principal could perform: approvals of a role whose group holds that principal, transitions out of states whose owning group holds it, relaxations for the operator's own runs, and installs when the person is a principal of the project's `people`. The grantor, or any principal of the project's `people`, may revoke, because revoking only narrows authority. The rules hold for the unsigned grants of PC1 as for signed ones.
- **Whose sessions act.** A session belongs to a delegate role through the role alias its launch records: the session host's mark in PC1, and from CM2 the certificate extension `waggle-role@relux.works`, checked against the alias at the pinned revision. By default a grant counts acts only from sessions started by the grantor: in PC1 the sessions the host marks as the grantor's, a guardrail like every unsigned act; from CM2 sessions whose certificates the grantor's key issued (waggle §3). A session of another operator never acts under the grant unless the grant names it.
- **What a grant binds.** A grant records the semantic digest of the resolved configuration in force when it is granted (§2.4). A landing that changes the resolved `policy.delegation`, or the roles and states the grant names, ends the grant; a landing that changes neither keeps it. A signed grant is verified once, when the board records it; later acts reference the recorded grant, so the envelope's expiry bounds only its delivery, and the grant's own `expires` bounds its use (waggle A1 §19.8).
- **A delegated act** is recorded as the delegate's act on behalf of its grantor under that delegation. A guard counts it as the grantor's act only inside the delegation's acts, scope, conditions, budgets and expiry and inside the policy's ceilings at the pinned revision, checked when the act is recorded and again when a guard counts it. Freshness (§6.3) and the binding of the element's declared fields (§6.4) apply to it as to the grantor's own act.
- **Quorum and self-approval.** Acts under delegation never count as more than one principal toward a quorum: a quorum above one needs distinct human principals acting directly, or distinct delegate sessions of distinct grantors. A delegated approval does not count when the delegate session, or a run it spawned, wrote a declared field of the element since the element was created, unless the `allow` rule that admits it sets `self_approval` — no self-approval, as there is no self-review (§6.10). A project that lets its orchestrator both fill in and approve small orders says so in its committed policy. Delivery invariant 2 and independence (§9) read the delegate's provenance, never the grantor's.
- **Budgets.** The board's kernel alone owns and counts a delegation's budgets. A budget sums the named field over the acts recorded under that grant within the period, a rolling window that ends at the act, using the value each act bound. A grant covering approvals counts its approvals, one covering only transitions counts its transitions, and a relaxation counts as one act. An approval voided before any transition used it gives its amount back. Checking the budget and recording the act are one compare-and-set in the lease store (§6.9). A board shared across machines needs waggle's lease store for that (CM2): before CM2, a grant with a budget is refused on a board that is not single-host, which the project documents.
- **Everything else waits for the person.** An act outside every delegation is not taken: over a ceiling or a budget, under `never`, after expiry or revocation, or from a session the grant does not cover. The kernel notifies the grantor — through a board notice to the grantor's hosted session or the board's inbox until the main loop exists, through the main loop afterwards — and the board shows the act as pending.
- **Revocation and expiry** end a delegation at once and in the same way. Delegated approvals under it that no transition has used yet stop counting; acts already consumed stay valid; nothing new is taken under it. Across machines a revocation takes effect where the lease store is: from CM2 through waggle's lease store, and before CM2 when the board syncs, which the project documents.
- **Signed delegated acts.** Where the project requires signatures for an act's category, the delegate signs with its orchestrator session key, and the board verifies the chain from the grantor's key to the act (waggle A1 §19.8). Either the session key's certificate is certified by the grantor's key with the delegation's constraints, or the act references the delegation record the grantor signed and comes from a session the grant covers. `user_verified` applies to the grant, which is the person's act; the delegated acts carry the delegate's signature within it.
- **Without a signature policy** a delegation is an unsigned record created with a person's confirmation. Like any unsigned human act, it is a guardrail (above).

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
- **Fields a guard reads are authority-bearing.** A field that a `field` guard of the element's workflow reads decides routes, and a field that the ceilings, conditions or budgets of `policy.delegation` or of a granted delegation read decides what a delegate may do. A run may not write such a field on the element it was spawned for; the orchestrator, a person and the kernel may. `review` in particular is set by the orchestrator or a person, never by the producer whose work it would exempt. A field the workflow only checks for presence (`fields.present`) stays writable by runs.
- **Approvals bind every declared field.** An approval, direct or delegated, binds the digest of all the element's declared fields, and any change to a declared field voids the element's unused approvals (§6.4). While a delegated approval is unused, the fields its delegation's ceilings, conditions and budgets read are frozen: a write to them is refused until a transition uses the approval or it is voided. A tool acts on the values recorded when its state was entered (§6.11).

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
- `on` makes the transition automatic: on an `event`, or `after` a duration in the source state. The kernel fires it; §6.9 says when and how events are kept.
- `limit` bounds a loop: after `max` firings for one element the transition leads to `then` instead, recording `resolution` when `then` requires one. The kernel keeps one counter per declared transition and element, so a transition whose `from` lists several states shares one counter across them.
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

**Field conditions** *(decided 2026-09-24)*. The `field` guard compares one field of the element with a value: `is` (equal), `in` (one of a list), `lt`, `lte`, `gt`, `gte` (numbers, money in the field's currency, dates and date-times). It turns declared data into routes. A review level is a field whose value selects the route: in dev-cycle `{ "guard": "field", "name": "review", "is": "none" }` lets inline work skip review. A threshold selects whether a purchase needs an approval: `{ "guard": "field", "name": "amount", "gt": 500 }` beside the `approvals` guard. The guard-bypass check reads field conditions on one field (§10). The guard names a field the element's type declares; an unset field fails every comparison, and defaults are written at creation (§5.2). Fields a guard reads are authority-bearing: runs do not write them on their own element, approvals bind the digest of every declared field, and a tool receives the values recorded when its state was entered (§5.2, §6.4, §6.11).

**Review levels and the goal ledger.** Today the ledger accepts an item whose review level is `none` on the producer's outcome alone (`tools/board-cli/internal/spawnruntime/goal_acceptance.go:167-170`). That shortcut applies only where the element's workflow declares no `field` route on `review`, as in the default. Where a playbook routes on `review`, the ledger follows the workflow: a `none` item is accepted when its inline route finishes, and Change Request work, whatever its level, is accepted only through review and `accept_cr` (§6.10).

**Verdicts** *(closes the first run's F35)*. Verdicts use the goal ledger's vocabulary and spelling (task-board `pkg/board/goal.go:40-49`): `accepted`, `changes_requested`, `stop_the_line`, plus `rejected` — the work is refused and will not be reworked. The ledger gains `rejected` in PC1, a task-board change, so any type may route on it *(decided 2026-09-24)*. A verdict may carry a typed `detail`, and a `verdict` guard may match it: the refuter records `accepted` with `no-contradiction`, `changes_requested` with `contradiction`, and `stop_the_line` with `human-decision-needed` (§12). Reviewers and approvers record verdicts; `record.verdict` records one for the acting reviewer or approver run and nothing for any other actor. `tool.passed` and `tool.failed` arise only from a state owned by a tool role (§6.11).

**Evidence must be fresh.** A verdict, an approval or an outcome satisfies a guard only when it was recorded after the element last entered the transition's source state; for work under a Change Request it must also name the revision the guard evaluates. This extends today's rule for reviewer verdicts, which count only when recorded after the latest producer outcome (`tools/board-cli/internal/spawnruntime/goal_acceptance.go:172-173`). So a refuter's old verdict cannot move a redrafted epic, a QA state waits for the tester's own outcome, and sending a purchase order again after an escalation needs a new approval.

### 6.4 Approvals and quorum *(decided)*

`{ "guard": "approvals", "role": "manager", "min": 1 }` needs approvals from that role; `min` above one is quorum voting and needs distinct principals present in the roster at the pinned revision. An approval is a board record bound to the approving principal, and it counts only while fresh (§6.3). It is a human act (§4.2): the operator's own CLI action counts only where policy does not require a signature; otherwise it is a waggle approver signature at the configured assurance and user verification, made through the command that renders the payload. The validator flags `min` greater than the number of principals in the role's group.

- **An approval binds the element's declared fields.** Every approval, direct or delegated, signed or not, records the values of all the fields the element's type declares and their digest at the time of approval, and counts only while the element's declared fields still hash to that digest. Any change to a declared field voids the element's unused approvals, and the transition waits for a new one: an order approved at 620 is not sent at 6,200, and one approved for one supplier is not sent to another. A signed approval carries the values and the digest in its payload (waggle A1 §19.1), and the board compares the digest (§19.7). While a delegated approval is unused, the fields its delegation's ceilings, conditions and budgets read are frozen (§5.2).
- **An approval binds the process by digest.** It records the semantic digest of the resolved configuration in force (§2.4), not the commit. A landing that leaves the resolved configuration unchanged keeps pending approvals valid; one that changes it voids them.
- **A delegated approval** counts as its grantor's approval only within the delegation (§4.2). Acts under delegation never count as more than one principal toward a quorum, and a delegate does not approve what it or its runs filled in unless the project allows it (§4.2).

### 6.5 Hand-off between processes *(decided)*

A state owned by a `process` role is worked in another process: a playbook instance on the same board (a member of the project's composition) or on another board. The playbook names a process **slot**; the project binds it (§7.2). The `handoff` action offers the work; the receiving orchestrator accepts the offer, creates the element in its own hierarchy (placed by the rule of §5.1) and sets the link:

```json
{ "from": "verifying", "to": "fixing", "guards": [ { "guard": "verdict", "is": "accepted" } ],
  "actions": [ { "action": "handoff", "to": "fixers", "create": "fix-target", "link_field": "fix" } ] },
{ "from": "fixing", "to": "fixed", "on": { "event": "process.returned" }, "guards": [ { "guard": "process.result", "in": ["done"] } ] }
```

- **On one board** — every process of the project's composition — the offer and the acceptance are board records, and the kernel emits `process.returned` to the offering element when the received element reaches a terminal state. This path needs no waggle and ships in PC1, so the first demos can run (§15).
- **Across boards** the offer and the acceptance are waggle messages (`coord.handoff-offer`, `coord.handoff-accept`), and the result returns through the `coord` type the waggle amendment adds (§17). The result is not a model's claim: a host key of the receiving board signs it in the signer role `result` that waggle A1 adds to its verification steps (namespace `result.coord.v1@waggle`), when the receiving kernel records the terminal transition, and it carries the receiving board's revision at that transition. No orchestrator session holds that key, and it is at least `hardware-bound` or held by a separate OS account, so that no process running as the operator can sign a result. The offering board verifies the signature against the host principals its committed project file binds to the other board before it emits `process.returned`; the binding form comes with cross-board hand-off in PC3 (§17.3). Until the amendment is implemented, with CM2, cross-board hand-off is not available.
- `process.returned` carries the final state of the received element; `process.result` guards on it. The exits of a process-owned state must cover every terminal state of the type the hand-off creates, so no result is left without a route (§10).
- Writes into another orchestrator's leased scope go through that orchestrator (§7.2).

### 6.6 Schedules *(decided)*

Recurring work is created by named schedules:

```json
"schedules": [ { "name": "weekly-inventory", "every": "P1W", "create": { "type": "inventory-check", "title": "Weekly inventory" } } ]
```

`every` is an ISO 8601 duration; `cron` is the alternative. A schedule a package proposes takes effect only when the project enables it (`policy.enable.schedules`, §8.5): by name, or with `all`, which admits every schedule the admitted templates declare, including ones later versions add. `task-board tick` fires schedules, and each firing is keyed by (schedule, due time) in the lease store, so two ticks never create two elements for one firing (§6.9).

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
| `effort_floor` | per role, the lowest effort a run may use, applied per model row (§9) | project only |
| `enable` | `schedules` (`all` or named), `auto_spawn`, `tools` (`all` or named), `relaxations` (§8.5, §9) | project only |
| `signatures` | which human acts need a waggle approver signature, the minimum key assurance and `user_verified` (§4.2) | project only |
| `delegation` | which human acts a person may delegate, to which orchestrator roles, under which ceilings; `never` wins (§4.2) | project only |
| `data` | publishers denied, models allowed and denied (`models`), billing classes admitted, local only (§9) | project only |
| `delivery.version_control` | `confirm`, `desired_commit_time` (required when `confirm`), `commit_time_policy`; owned here since v3, and `task-board.config.json`'s `version_control` keys are read for one release with a warning (§2.1) | template or project |
| `delivery.loop_detection` | rework threshold `max(floor, factor × median)` | template or project |

**Work in progress** *(decided 2026-09-24)*. `policy.wip` bounds the elements in progress:

```json
"wip": { "board": 12, "per_role": { "developer": 3, "reviewer": 2 }, "per_state": { "dev.review": 2, "purchase.approval": 5 } }
```

- `per_state` counts the elements in one state, named `<workflow>.<state>`, or a bare state name for that state in every workflow that has one. `per_role` counts the elements whose current state the role owns, or whose `assignment.role` is the role. `board` counts the elements in non-initial, non-terminal states.
- Elements are counted, not runs, so a limit also holds where no run works: "no more than five orders awaiting approval" bounds a human-owned state. Board elements are committed and published, so every teammate counts the same elements.
- A manual or agent transition that would take an element over a limit is refused with `wip_limit_reached`, naming the limit that held, so no run starts for it. An automatic transition into a full scope is held, like a kept event (§6.9), and fires when capacity frees.
- The limits describe the process. What one computer can carry — parallel runs across every board, concurrent local check sets, rate-limit ladders — is the operator's machine capacity (`runtimes.toml`, §2.3); a run whose element fits `policy.wip` still waits for machine capacity. Today's `spawn.max_parallel` and `spawn.validation.max_concurrent_suites` become machine capacity, and a playbook without `policy.wip`, the default included, has no process limit (§14).

### 6.9 Timers, schedules, tools and external events before the main loop *(decided)*

Time-based transitions (`after`), schedules, approval expiries and tool invocations need a process that runs without an agent. The main loop will be that process; until it exists, `task-board tick` does the job:

- **Scheduler** *(decided 2026-09-24)*. A user-level system timer calls tick every minute: a launchd agent on macOS or a systemd user timer on Linux, installed by one task-board command. It runs whether or not any session is open. The main loop takes the job over when it exists.
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
- It runs only the command the binding names, through the shim Curator installed for the provider after checking the shim against the Curator lock (its owner must be the bound skill), or under the enforced `script-worker-v1` policy when the provider declares it (curator-spec `protocol/core.md` §4.1.1).
- The environment is sanitized: a fixed `PATH` of the managed command directory and system directories, and no variables inherited from agent runs. The working directory is the project root.
- The contract: the command receives `--element <id>` and, on standard input, a JSON object with the element's id, type and declared fields as the kernel recorded them when the element entered the state, keyed by (element, entry), the playbook, the tool name and the entry key. A write to the element between the entry and the invocation never changes what the tool acts on, and when the state was entered through an approved transition these are the values the approval bound (§6.4). Exit status 0 fires `tool.passed` and 1 fires `tool.failed`; any other status, a crash or the binding's `timeout` (ten minutes by default) fires `tool.failed` with the reason `error` or `timeout`. Bounded standard output is stored as the tool's outcome. The executor never retries; a `tool.failed` exit with a `limit` models retries.
- A person may leave a tool-owned state through its declared manual exits (§4.2 rule 2); a running invocation is then cancelled.
- A tool runs only when the project lists it in `policy.enable.tools` and only from a skill installed through Curator (§8.4, §8.5). Entering a state whose tool the project does not list is refused.

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
3. Transitions are keyed by `(from, to, via, on)` (§6.2): a child transition with the same key replaces the parent's; `remove: true` deletes it. A state with `remove: true` deletes the state and every transition touching it. Removing a key the parent does not have is an error, so drift after a parent update is loud.
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

### 8.6 Finding and installing skills *(decided 2026-09-24)*

A process often needs a capability someone has already packaged: an integration with a supplier, a restaurant's CRM, a calendar. Agents should find and reuse such a skill rather than write their own.

- **Search.** An orchestrator or the process designer may search the Curator registry, whose skills pass Curator's security audit, and may read what each skill does and exports.
- **Install.** It may install an audited skill from the Curator registry into the project when the project allows it in `policy.enable.skills` — `audited` for any audited skill of the registry, or a list of skill names — or when a delegation covers it (§4.2). Installing widens what can execute, so without that permission a person installs, or confirms the agent's proposal (§2.3). Installing changes `Skillfile.json` and its lock, committed like any other change, and every installation by an agent is recorded on the board with the skill's lock identity.
- **Use.** Installing a skill gives a role its instructions; it runs nothing. A skill's tools run only once `tools` binds them and `policy.enable.tools` lists them, which is a commit to `Playbook.json` in authority-bearing sections: a person confirms it (§2.4).
- **Unknown sources are refused by default**, the way a desktop operating system refuses unsigned software. A skill that is not in the Curator registry, or not audited there, installs only after the operator names its source in a deliberate Curator setting, which shows a warning and is a widening entry of kind `skills`: confirmed by a person, at a level that cannot be lowered below `user` (§2.3). No delegation covers it. The recommended path is to submit the skill for audit, through the operator-facing command, so that it enters the Curator registry and every user can install it safely.

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

- Permanently, for the whole team: only a change to the project's `Playbook.json`. A role's `require` and `on_unmet` are authority-bearing, so the change takes effect once it is committed and a person has confirmed it (§2.4).
- Permanently, for one operator's own runs: an open-ended standing answer (below) *(decided 2026-09-24)*. This amends agent-selection §8 ("only the project can waive") by adding it and the temporary paths below.
- Temporarily: through `ask`, an operator approves an exception for **their own runs only**, scoped to an element subtree (`element`, the nearest ancestor of a type, or `project`) and time-limited. The role's `needs.approval` sets the scope and the expiry; for a role without it, the exception covers the element that asked and expires after 30 days. The exception is a kernel record, `exception`, available in every playbook rather than a type a playbook must declare, and it is bound to the approving principal; it never reaches a teammate's runs. Until runs are bound to principals (waggle worker certificates), "own runs only" is a guardrail. `policy.signatures` may require a signature for it (`relaxations`).
- An exception that expires while a run is working lets that run finish; new runs refuse. An exception relaxes only project requirements, never the operator's own `[allow]` *(decided 2026-09-24)*.
- **Standing answers** *(decided 2026-09-24)*. An operator may answer an `ask` in advance in `models.toml` (`[answers]`, §2.3), so autonomous work does not stop for a question the operator has already answered. A standing answer is a widening entry: it takes effect only once the operator has confirmed it through the operating system's user authentication (§2.3, P3). When `ask` fires and a confirmed standing answer matches the role, the requirement term and the project, the kernel records an `exception` marked as a standing answer from the operator file, bound to that operator, and the run proceeds; the team sees the exception on the board. The exception takes the role's approval scope and expiry unless the answer's `until` sets an end date or, written explicitly, `open`.
- **Open-ended standing answers** *(decided 2026-09-24)*. An answer with `until = "open"` relaxes the requirement at every future `ask` of its role, for this operator's own runs, and each application appears on the board as an exception. A project forbids standing answers, open-ended ones included, with `policy.enable.standing_answers: false`, an authority-bearing key (§2.4); they never apply when `policy.signatures` covers `relaxations`, because a file cannot show that a person acted at that moment. A standing answer applies to the operator's own runs only and never widens `[allow]`.
- **Periodic review** *(decided 2026-09-24)*. Once a month by default (`[review] answers` in `models.toml` sets the period, §2.3), the main loop — and until it exists, a board notice that points to `curator models answers review` — lists every open-ended standing answer and how often each was applied since the last review. The operator re-confirms each one, which is the same confirmation as a widening entry (§2.3), or removes it; the answer and the date are recorded, and a re-confirmed answer is listed again at the next review. An answer the operator leaves unanswered stays in force, is shown on the board as unreviewed, and is listed again.
- **Through a delegation** *(decided 2026-09-24)*. Where `policy.delegation` allows `relaxations`, a session the grant covers may approve an `ask` exception on its grantor's behalf, for the grantor's own runs and within the delegation (§4.2). Such an approval answers one `ask` with the role's approval scope and expiry, and the grant may limit it to the asking roles and the requirement terms it names (`relax`, §4.2); it never becomes a standing answer, which only the operator writes in their own file (the author's resolution, §24).

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
| check sets | `workflow dev: delivery.checks-passed names set handoff, which task-board.config.json does not define`; the same for `delivery.integrate`'s `checks` and `delivery.integration-evidence`'s `set` |
| ranges | `role reviewer: need tg has min 50 above max 10` |
| kernel-written fields | `workflow dev, state blocked: record_source names blocked_from, which dev-task does not declare`; `workflow dev, state blocked: record_source field blocked_from does not admit source state qa of an entry` |
| work in progress | `policy.wip.per_role: reviewer2 is not a role`; `policy.wip.per_state: dev.qa is not a state` |
| unsigned human acts | `workflow purchase has human-owned states and the project sets no policy.signatures: human acts are a guardrail only until signed forms arrive with the waggle amendment (CM2)` (warning) |
| signable forms | `policy.signatures.require_for approvals: board approvals have no signable form before the waggle amendment (CM2); the states that need them cannot be left until then`; `policy.signatures covers delegations whether listed or not: no grant can be signed before the waggle amendment (CM2)` (warnings) |
| model admission | `policy.data.models.allow: codex/gpt-6-luna effort high..medium is an empty range`; `policy.data.models: allow codex/gpt-6-astra is also denied`; `policy.data.models.allow: codex/* and codex/* both match codex/gpt-6-luna at the same specificity`; `policy.data.models.roles: reviewer2 is not a role`; `roles developer, reviewer can run only on models policy.data.models admits: teammates without them cannot run these roles` (information) |
| delegation | `policy.delegation: role manager in approvals is not a human role`; `delegate cafe-orchestrator does not have capability orchestrator`; `state purchase.draft is not human-owned`; `state approval must be named <workflow>.<state>`; `ceiling field amount is not declared by inventory-check, whose approvals by manager the rule also covers`; `allow rule for purchase.escalated is shadowed by never` (information) |

The guard-bypass rule: an `approvals` guard or a `risk` marker on one non-automatic entry into a state must appear on every non-automatic entry into it, or the guard must sit in the state's `requires`. One exception makes thresholds expressible: an entry without the guard or the marker is admitted when its `field` condition admits no value that the `field` condition of any guarded or marked entry admits, on the same field, so `amount lte 500` may skip the approval and the `spending` marker that `amount gt 500` carries; a guarded or marked entry without a `field` condition covers every value. The exception is sound at run time only because fields a guard reads are authority-bearing (§5.2), approvals bind the digest of every declared field (§6.4) and a tool acts on the values recorded at entry (§6.11). It trusts the value the element carries: the orchestrator that writes the amount decides the route, as the project's threshold allows. A project that must stop an agent from splitting spending under a threshold keeps the approval on every entry, or delegates it with a budget (§4.2). Verdict and `process.result` guards select an exit of their source state and are outside the rule; `blockers.clear`, `estimate` and `assignee` are hygiene guards that rework routes may omit on purpose; the delivery routes are covered by the invariant checks.

**Machine checks** (depend on what is installed and on the operator): skills installed through Curator and audited; each template tool binding declared by the orchestrator's manifest in runtime mode narrowed to the bound command; each command shim owned by the bound skill; operator bindings able to satisfy each role's needs and effort floor; each skill `policy.enable.skills` names listed as audited by the Curator registry, and no installed skill from an unknown source without its confirmed setting (§8.6). They report typed "unbound on this machine" refusals and never change the portable verdict.

**Authority report** (at every load, from the board). The kernel compares the authority digest of the committed version with the last confirmed one (§2.4) and reports each pending change: the authority-bearing sections that differ, and the workflow changes that take authority from people — a human-owned state removed or re-owned, an `approvals` guard or `risk` marker removed from an entry, a `field` condition that routes to a person loosened, a delivery guard removed. `playbook validate` prints the same report for a working tree against a given confirmed digest, so a change can be reviewed before it is committed.

A throwaway prototype of a subset of the portable checks (not committed) was run over [`../examples/`](../examples/) at this revision. It covers states and initial states, owners, reachability and terminal reachability, exits, the root's capability, nesting, joins, owner-kind exits, determinism with the four-part transition key, loop-limit targets and resolutions, the guard-bypass rule with field conditions, `field` guards, kernel-written fields, check sets (from `delivery.checks-passed`, `delivery.integrate` and `delivery.integration-evidence`), ranges, work-in-progress keys, tool enablement, both signature warnings, model admission and delegation; it checks templates one by one and projects for their own keys, and leaves the delivery invariants, signer groups, the roster, the degraded path, placement and slot contracts to the v2.1 prototype, whose results stand. Every template and both composed projects pass with no error. It reports seven notices, all intended: the cafe project's four `require_for` categories have no signable form before CM2; the scrum project, a PC1 demo, has human-owned states without a signature policy in both of its playbooks; and, as information, the scrum project's model admission limits every role to the listed models. On seeded defects it reports each check v3.1 adds: an inverted range, a `record_source` naming an undeclared field, a check set named on an integration route that the configuration does not define, and two manual transitions with one key, while a manual and an automatic transition between the same states pass. An unguarded entry into `sending` with `amount lte 500` beside entries that carry the approvals guard and the `spending` marker under `amount gt 500` is admitted; with `amount lte 600` it is refused. On seeded defects of v3.2 it reports each new check: a delegate without capability `orchestrator`, a role in `approvals` that no person holds, a delegated state that no person owns, a ceiling on an undeclared field, an allowed model that is also denied and an inverted effort range; an `allow` rule that a `never` rule shadows is reported as information. At v3.3 the prototype also checks that a `record_source` field admits every source state of the entries, that every type a delegation rule reaches declares the rule's ceiling fields, that delegation rules name states as `<workflow>.<state>`, that no two allow entries match the same rows at the same specificity, that per-role restrictions name roles and allow nothing the project denies, and that a signature policy which leaves `delegations` implicit is warned about. On the examples it reports no error and the same seven notices; on seeded defects it reports each of these checks. At v3.4 the examples give the same result, and a second throwaway prototype computes the authority report: on the cafe template, removing the `approvals` guards, giving `purchase.approval` to the orchestrator and removing the `spending` markers each leave the authority digest unchanged and are each reported as a workflow change that takes authority from people; emptying `tools` changes the digest; a reworded description reports nothing.

All examples validate against the schema. The schema refuses URLs and shell syntax in hook arguments, schedule fields and event signers; argument names that would select a program, an environment or an endpoint; package pins that are not Curator identities; program strings as tool commands; `effort` and the old `capability` need; non-operator principals in `people`; tools posing as reviewers; and action objects inside guards. Since v3 it also refuses the `null` tool form, `context_window_min`, a boolean `policy.enable.tools` or `policy.enable.schedules`, empty ranges, and `field` or `delivery.checks-passed` guards without their arguments. Since v3.1 it refuses `delivery.integrate` without `checks`, `delivery.integration-evidence` without `set`, the bare names of all three, and an unknown category in a per-category `user_verified`. Since v3.2 it refuses a model pattern that is not `<runtime>/<model>`, `<runtime>/*` or `*/<model>`, an effort range outside the common order, a delegation rule without acts, approvals without a `role`, transitions without `states`, a `role` or `states` without the act that uses it, an `allow` rule without `to`, and a `never` rule with ceilings. Since v3.3 it refuses a bare state name in a delegation rule, `states` on a rule that covers neither approvals nor transitions, `self_approval` on a `never` rule, a window that is not `HH:MM-HH:MM`, and an empty or unknown per-role model restriction. Since v3.4 it admits `installs` among delegation acts and the `policy.enable` keys `standing_answers` and `skills`, and refuses any other value for them. It cannot compare a range's bounds, which the validator does, and it cannot make the other hooks' arguments exact: their schemas come with the provider modules in PC1 (§17.3).

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

Agents are good at finding contradictions; what they lack is memory of what was decided. The board supplies it. The refuter's runs on this document are in [`../reviews/`](../reviews/): [`2026-09-24-refuter-v0.md`](../reviews/2026-09-24-refuter-v0.md) on v1, [`2026-09-24-refuter-v1.md`](../reviews/2026-09-24-refuter-v1.md) on v2, [`2026-09-24-refuter-v2.md`](../reviews/2026-09-24-refuter-v2.md) on v3 and [`2026-09-24-refuter-v3.md`](../reviews/2026-09-24-refuter-v3.md) on v3.1 and v3.2.

## 13. Examples

| File | Shows |
| --- | --- |
| [`default.playbook.json`](../examples/default.playbook.json) | today's task-board behaviour (§14) |
| [`dev-cycle.playbook.json`](../examples/dev-cycle.playbook.json) | delivery with the delivery invariants, inline and Change Request routes, check sets `handoff` and `landing` for Change Request work, review levels routed by `field`, `blocked` with a block kind, a kernel-recorded source state and a repeat limit, rework limits, `stop_the_line`, the refuter's verdicts, decisions, needs and `ask` |
| [`bug-hunt.playbook.json`](../examples/bug-hunt.playbook.json) | proving findings and offering them to the fixing process through slots |
| [`scrum/scrum.playbook.json`](../examples/scrum/scrum.playbook.json), [`scrum/project.Playbook.json`](../examples/scrum/project.Playbook.json), [`scrum/task-board.config.json`](../examples/scrum/task-board.config.json) | a cadence layer composed over dev-cycle; membership links on the sprint; delegated estimate fields; planning poker; capacity; work-in-progress limits over elements; project-only policy, including model admission that restricts the project to named models and its reviewer to one (§9); the local check sets `handoff` and `landing` in the project's `task-board.config.json`. As a PC1 demo it sets no signature policy and runs no hosted set (§4.2, §15) |
| [`cafe/cafe.playbook.json`](../examples/cafe/cafe.playbook.json), [`cafe/bookkeeping.playbook.json`](../examples/cafe/bookkeeping.playbook.json), [`cafe/project.Playbook.json`](../examples/cafe/project.Playbook.json) | fields, humans, a tool reached through the orchestrator's runtime dependency, timers, a named schedule, signed events and their signer groups, a join, a hand-off with a returned result, privacy, a work-in-progress limit on a human-owned state, and the project file that enables the tool and the schedule, lets the cafe managers delegate approvals of purchase orders up to 1,000 EUR to their own orchestrator sessions, including orders those sessions' runs drafted (`self_approval`, §4.2), lets agents install audited skills and forbids standing answers (§8.6, §9), and sets the signature policy of the post-CM2 demo |
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
13. Today's `spawn.ceilings`, runtime set and schedule windows sit in `task-board.config.json`, which the kernel reads from the working tree; `policy.data.models` sits in `Playbook.json`, read from the head of the default branch (§2.4), so a change to model admission takes effect when it lands. The default sets no model admission, so a project that pinned models, runtimes or windows carries them over with the migration command (§2.3), and nothing it admits today is lost.

## 15. Integration plan

| Step | Delivers |
| --- | --- |
| PC0 | this draft, the schema, the examples; the refuter's first run (v1 → v2) and second run (v2 → v2.1); the operator's decisions (v3); the third run, on v3, answered by v3.1; the operator's decisions on the two points it left open, and delegation (v3.2); the fourth run, on v3.1 and v3.2, answered by v3.3; the operator's answers of v3.4 (open-ended standing answers, authority confirmation, the companion app, commits, skills); a check of the changes of v3.3 and v3.4, after which the specification is frozen |
| PC1 | the module: schema, parser, resolver (`extends`, `uses`, slots), validator CLI with portable and machine checks; a thin seam in task-board that loads the project file from the head of the default branch (§2.4) and `default.playbook.json` with identical behaviour or documented differences; new types from `Playbook.json` on the generic engine with the kernel hooks of §6.3, the `field` guard included, with guard-read fields protected from the runs they gate and field defaults written at creation (§5.2); `record_source` and `clear_on_exit` (§6.1); approvals bound to the digest of the element's declared fields and to the configuration digest (§6.4), with the fields a pending delegated approval depends on frozen; the structured `assignment.role`; `rejected` in the goal ledger; `policy.wip` over elements (§6.8); the version-control policy read from `Playbook.json`, with the config's keys read for one release with a warning; the kernel's executor, one component that runs local check sets over today's landing-gate suite in a clean checkout with a sanitized environment (§6.11, §6.12) and that tick reuses in PC3; the legacy operator layer read from today's config until `models.toml` exists, and model admission from `policy.data.models` with today's ceilings, runtime set and windows read as project admission until migrated (§2.3, §9); `spawn.supervision` in `task-board.config.json`; delegation records and delegated acts without signatures, granted through the operator-facing command with the operating system's user authentication — a task-board component PC1 carries, shared with the operator-file commands — counted only from the grantor's own sessions as the session host marks them, with budgets on single-host boards and the grantor notified by a board notice (§4.2); per-run recording of `ttft` and `tg` where the harness exposes them, and the reference prompt size for speed facts (§9); the authority digest, its confirmation on the machine and the authority report at every load (§2.4, §10), and `task-board playbook apply` for projects without git; `policy.enable.standing_answers` and the board notice that starts the monthly review of open-ended standing answers (§9); `policy.enable.skills` and installation of audited skills by agents, recorded on the board (§8.6); the constructor-ready schema (`ui` blocks, lossless round trip). Because the demos follow PC1 directly *(decided)*, PC1 also carries **the minimal delivery seams over today's commands**: `delivery.cr-published`, `delivery.review-ready`, `delivery.cr-accepted`, `delivery.cr-absent`, `delivery.commit-acknowledged` and the evidence and authorization guards mapped onto today's authorization options; `delivery.integrate` mapped onto the orchestrator's `worktree integrate`; and `delivery.integrated`, `delivery.integration-failed` and `delivery.rework-started` emitted when today's integration transaction, its failure, and landed rework, invalidation or convergence complete. It also carries **the local hand-off on one board** (§6.5). |
| Demos (right after PC1) | `dev-cycle`; `bug-hunt` → `dev-cycle`; `scrum` over `dev-cycle` — all on one board *(decided 2026-09-24)*. The cafe with bookkeeping follows PC3 and the waggle amendment |
| PC2 | role skills and the `curator-playbook-dev-cycle` package on Skillfile schema 2, installed on demand once Curator ships activation `on-demand` and the skills launch channel (§8.1, §8.3), with the description stopgap until then; the JSON Schema of `data/role.json`; task-board resolves roles from the project overlay `.roles` first, then installed role skills, then `~/.roles` read-only for one release with a warning; `setup.sh` stops copying `~/.roles` (this is roadmap milestone M1) |
| PC3 | the delivery module proper: the PC1 seams become a module whose invariants it checks itself; hosted check sets, `mirror` evidence and the landing override (§6.12); `task-board tick` and its user-level system timer, calling PC1's executor for tools, timers and schedules (§6.9, §6.11); signed events, approvals and delegations, and delegation budgets and revocations across machines through waggle's lease store, once the waggle amendment lands with CM2; `second-person` confirmation of the operator's files; hand-off across boards through waggle (CM1 and the amendment), with results signed by the receiving board's host keys and the binding of a process slot to another board's hosts (§6.5) |

The operator companion app (§2.3) is a track of its own, outside these steps; until it ships, every confirmation is made on the machine, and a headless node is confirmed over the operator's own terminal session to it, with the account password or a FIDO2 key.

PC4 of the earlier plan is dropped *(decided)*: a playbook is a collection of skills. The constructor-ready schema, which the roadmap's PC4 row also carried, is part of PC1, and the Curator changes are the Curator goal (§8.3). The roadmap rows live in the wiki and are updated there. Details: [`track.md`](track.md).

## 16. Names

Specifications use technical terms. User-facing material uses the Apiary names *(decided)*: Hive = one node's agent system, Comb = a process, Keeper = the main loop, Bee = the agent loop, Waggle = the messaging protocol; a playbook keeps its name. The ecosystem wiki records the mapping; this is the only place in the specification that uses those names, apart from the file name of one wiki note that §17.3 cites, and no identifier uses them (the v1 owner kind `comb` is now `process`).

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
- the demos dev-cycle, bug-hunt and Scrum right after PC1; PC4 dropped; the default playbook as the PC1 target.

### 17.2 The operator's decisions

Rows 1–20 were taken on v2.1, rows 21–24 on v3.1, rows 25–29 on v3.3.

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
| 11 | role skills and the root orchestrator's skill install with Curator activation `on-demand` and reach a launch through the skills launch channel; description stopgap until Curator ships it | §8.1, §8.2, §8.3, §15 |
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
| 25 | open-ended standing answers are allowed (`until = "open"`): for the operator's own runs, each application visible on the board, forbidden by a project with `policy.enable.standing_answers: false`; a periodic review, monthly by default, lists them for re-confirmation. This amends the rule that only the project file relaxes a requirement permanently: for the whole team it does, for one operator's own runs an open-ended answer may. How far a delegated relaxation reaches is the author's resolution (§24) | §2.3, §9, §17.1; schema |
| 26 | the authority-bearing sections of `Playbook.json` — people, services, `policy.signatures`, `policy.enable`, `policy.data` (models included), `policy.delegation`, role `require` and `on_unmet`, tools, `extends`/`uses` — take effect only once a person confirms the authority digest of the committed version; until then the kernel applies the last confirmed ones; the other sections take effect on commit and may change under delegation. That workflow changes which take authority from people count too, and the first confirmation at project creation, are the author's resolutions (§24) | §0, §2.4, §4.2, §10, §15 |
| 27 | a person confirms on the machine (the operating system's user authentication) or through the operator companion app on a phone: a key in Secure Enclave or Android Keystore behind biometrics, paired with the node at initialization through an out-of-band handshake that pins keys, requests over a channel that resists a man in the middle (a tailnet, or a relay encrypted end to end to the pinned keys), the approved content rendered before signing; headless nodes rely on the app, the common case; the app's design, transport and SSH administration are open | §2.3, §4.2, §15, §17.3; waggle A1 §19.5 |
| 28 | rule changes take effect when committed to the default branch; git hosting is optional and pull requests are never required; a project without git applies a new version with an explicit command and the same confirmation | §0, §2.4 |
| 29 | agents may search the Curator registry and install audited skills where `policy.enable.skills` or a delegation allows it, and installing widens what can execute; skills from unknown sources are refused by default and enabled only through a deliberate, confirmed operator setting with a warning, with submission for audit into the Curator registry as the recommended path; enabling a skill's tools is a commit to authority-bearing sections | §0, §2.3, §4.2, §8.6, §10; schema |

H4 (keep the effort floor), H6 (the demos and the PC rows) and H8 (one owner per element in each state; privacy as a views vector; the refuter's verdicts) were decided earlier and stand.

### 17.3 Open questions

1. *Decided in v3.4:* open-ended standing answers are allowed, with a periodic review, and a delegated relaxation answers one `ask` at a time (§9, §17.2 row 25). The item keeps its number so that earlier change logs still point to it.
2. **First start without `models.toml`.** What the first launch shows an operator who has no file: whether `curator models init` proposes bindings for the project's roles from the runtimes installed on the machine, and how a refusal explains the missing admission (§2.3, §9).
3. **OS-level protection of configuration files and keys.** How the operator's files, their confirmed versions and the confirmation key are protected from replacement by a process running as the operator, and how a project's root configuration is: file permissions, edits only through a privileged helper or `sudo`, a separate owning account, immutability flags (§2.3). This covers the operator's files, the project's root configuration and the host keys that sign hand-off results (§6.5) alike. It also covers the record of the last confirmed authority digest (§2.4) and the keys paired with the companion app (§2.3). It applies to every root configuration, not only `models.toml`.
4. **The reference prompt size** for `pp` and `ttft` measurements (PC1, §9).
5. **Per-hook argument schemas** for every provider module (PC1). The schema covers the arguments of `field` and `delivery.checks-passed` already.
6. **The remaining mapping points of §14**: `defer_cr` and `withdraw_cr`, and the close-landed methods as `via` commands (PC1).
7. **How a process is created and registered on a node**, how the main loop routes a request among processes and resolves a request that fits several, and how playbooks are generated and generalized: the main loop's process registry, outside this specification (wiki `research/keeper-and-process-lifecycle.md` and open questions).
8. **Outside this repository**: this text amends agent-selection §3, §4, §7 and §8 and LP D1, D2 and D9; the drafts in skill-project-management take the amendments when those tracks resume. The waggle amendment, delegation included, lands in the waggle repository; on-demand skills land in Curator (curator-spec issue #91, curator issue #91); the operator-file commands land in the components that own each file.
9. **Cross-board result signers** (PC3): the form in which a project binds a process slot on another board to that board's host principals (§6.5).
10. **A project-wide delegation budget.** Budgets belong to grants, so two managers' grants of 2,000 EUR a week each let the orchestrator commit 4,000 (§4.2). Whether `policy.delegation` rules may carry a cap across every grant under them is open.
11. **The operator companion app** (§2.3). Its design and platforms (iOS first); the pairing handshake at initialization and how a second phone is added; the transport, whether the operator's tailnet (a dependency on it) or a relay of our own with end-to-end encryption to the pinned keys, and push to wake the app; how it renders each kind of request; whether it also carries administration of the node over SSH; how a lost phone is revoked. Until the app exists, confirmations are made on the machine.
12. **Submitting a skill for audit** (§8.6): the command and channel that send a skill from an unknown source to the Curator registry's audit, and how the result returns to the operator.

**What this means for PC1.** PC1 can start: everything it implements is decided. It confirms on the machine; the companion app (item 11) comes as its own track. The waggle amendment gates PC3's signed acts, signed delegations and the cafe demo only; Curator's on-demand skills gate PC2's clean install, with the description stopgap until then; the operator-file commands gate widening entries of the operator's files, which the resolver ignores until then. The three first demos need none of the open answers.

## 18. Change log: v1 → v2

Each finding of the refuter's first run ([`../reviews/2026-09-24-refuter-v0.md`](../reviews/2026-09-24-refuter-v0.md)) and where v2 answers it. Section numbers in this table refer to v2; v2.1 keeps them, except that the open questions of §17 were renumbered (§17.3). v3 keeps them too, adds §6.12 and §20, and replaces §17.2 and §17.3, so references to open questions in §18 and §19 name the v2 and v2.1 lists. v3.1 adds §21, v3.2 adds §22, and v3.3 adds §23 and renumbers §17.3.

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

The operator's answers that followed v3.3 (§17.2, rows 25–29) and where v3.4 applies them. Three gaps in them are filled by the author and marked as such.

| Decision | Change in v3.4 |
| --- | --- |
| Open-ended standing answers (row 25; refuter v3, S7) | allowed with `until = "open"`, for the operator's own runs, each application an exception on the board; `policy.enable.standing_answers: false` lets a project forbid standing answers; a monthly review by default (`[review] answers`), started by the main loop or, until it exists, a board notice pointing to `curator models answers review`, lists open-ended answers for re-confirmation, and an unanswered one stays in force, shown as unreviewed; the rule on permanent relaxation now distinguishes the whole team (only the project file) from one operator's own runs; §17.3 item 1 is closed. *Author's resolution:* a delegated relaxation answers one `ask` at a time with the role's approval scope and expiry, may be limited to named roles and terms (`relax`), and never becomes a standing answer (§2.3, §9, §17.1; schema) |
| Authority-bearing sections need a person (row 26) | the authority digest of the committed version takes effect only once a person confirms it; until then the kernel applies the last confirmed authority sections, reports the pending change and notifies the operators; the other sections take effect on commit and may change within the project's delivery and its delegations; the authority report at every load and in `playbook validate`. *Author's resolutions:* workflow changes that take authority from people — a human-owned state removed or re-owned, an `approvals` guard or `risk` marker removed, a `field` condition that routes to a person loosened, a delivery guard removed — count as authority-bearing, because otherwise one commit could bypass the rule; a change that depends on a pending authority change waits with it; a project's first version is confirmed at creation, and until then no tool runs and no delegation or relaxation is honoured (§0, §2.4, §10, §15) |
| How a person confirms (row 27) | one definition for every confirmation: the machine's user authentication, or the operator companion app — a key in the phone's secure hardware behind biometrics, paired at initialization through a QR code printed in the operator's terminal that pins keys on both sides, requests over a tailnet or an end-to-end-encrypted relay, the approved content rendered before signing; a companion signature meets `user_verified: required` through the app's attested enforcement (waggle A1 §19.5); delegation grants, operator-file widenings and unknown skill sources use it; no delegation covers a confirmation; until the app ships, confirmations are made on the machine and a headless node over the operator's terminal session; the app is a track of its own, and its design, transport and SSH administration are §17.3 item 11 (§2.3, §4.2, §15, §17.3) |
| Commits, not pull requests (row 28) | a change takes effect when committed to the default branch; git hosting is optional and pull requests are never required; a project without git applies versions with `task-board playbook apply` and the same confirmation (§0, §2.4, §15) |
| Skills (row 29) | new §8.6: agents search the Curator registry and install audited skills where `policy.enable.skills` (`audited` or named skills) or a delegation (`installs`) allows it, each installation recorded on the board; installing gives instructions and runs nothing, and a skill's tools run only after a confirmed commit to `tools` and `policy.enable.tools`; skills from unknown sources are refused by default, allowed only through a deliberate Curator setting that warns and is a widening of kind `skills`, never below `user` and never delegable; submitting a skill for audit is the recommended path, its channel §17.3 item 12; machine checks for both (§0, §2.3, §4.2, §8.6, §10; schema) |

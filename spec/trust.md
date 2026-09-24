# Trust: persons, keys, confirmations and signatures

Status: **DRAFT v0**, 2026-09-25. Split out of the process-configuration specification ([`process-configuration.md`](process-configuration.md)) at its draft v4.0, so that the core can be frozen while this document goes through its own refuter rounds. It carries the parts of draft v3.5 that decide who counts as a person and how a person's acts are shown:

- confirmers and confirmations, with their records;
- signatures and key assurance, and key enrollment;
- the operator companion app and its pairing, and confirmation of a headless node;
- roster and revocation checks;
- the verification of delegation grants and delegated acts;
- the confirmation of the operator's own files.

The operator accepted its principles on 2026-09-25 (§1). Everything else here is the author's and waits for its own review. The sixth refuter run ([`../reviews/2026-09-24-refuter-v5.md`](../reviews/2026-09-24-refuter-v5.md)) found the gaps §15 maps.

This document stays in this private repository for now. waggle is a public protocol under Apache-2.0, and the trust design waits for the licensing work and counsel review (wiki `open-questions.md`, Licensing) before any of it is published. waggle's Amendment A1 keeps the wire forms of board acts and points here for what they mean.

## Terms

| Technical term | User-facing name | Meaning |
| --- | --- | --- |
| the whole system | apiary | every node and process of one owner; one node is a small apiary |
| node | yard | one machine |
| main loop | keeper | the always-on loop of a node, which manages its processes |
| process | hive | one domain: its project, board, playbook and root orchestrator |
| root orchestrator | queen | the single orchestrator of a process that is started from outside it |
| board | comb | the process's board; its elements are the cells |
| run | bee | one agent run |
| external participants | wild bees | agents of other owners working on part of a process under leases and signatures |
| process package | nuc | a published playbook from which a new process starts |

Specifications use the technical terms. The wiki's naming page keeps the mapping.

## 0. The design on one screen

- **The root cause.** Agents run as the operator's OS user. Whatever that account can reach, an agent can reach too: the project's files, `~/.curator/`, `ssh-agent`, any key that does not demand presence, and the `task-board` binary itself. A check that runs inside that account stops a well-behaved agent and nothing else. It is a guardrail, not a boundary.
- **The boundary is a person's presence.** Only a signature made with a key that needs a person's presence for every use shows that a person acted. Three kinds of key qualify:
  - a FIDO2 key with user verification;
  - a key in a Secure Enclave, Windows Hello or a TPM whose access control requires the operating system's user authentication;
  - the attested operator companion app.

  Later, agents run under an OS account of their own, and the stores and binaries they must not change are protected at the OS level (§12, §14).
- **Confirmations are signed records on the board.** Every machine verifies them against the roster of the version in force, without CM2. So one confirmation serves every machine, and each machine checks it for itself.
- **Stages.** L0 now: one operator, one authoritative board. L1: several operators with majority confirmations. L2: public participants. The invariants are the same at every stage (§13).
- **Delivery.**
  - PC1 ships a guardrail stub: an unsigned operator act outside any run.
  - PC3 brings presence-bound keys and signed confirmations.
  - CM2 brings waggle's forms for the other board acts.
  - The companion app is a track of its own.

## 1. Principles *(accepted by the operator, 2026-09-25)*

- **(a) Only a person confirms.** A confirmation is valid only when it is signed with a key that requires a person's presence: a Secure Enclave key behind biometrics, a FIDO2 key with user verification, or the attested companion app. No project setting lowers this.
- **(b) The signer computes what it signs.** The signing device renders the change itself and signs what it rendered, never a digest computed elsewhere.
- **(c) A chain, not a set.** Confirmations form a chain. An old confirmation never revives after a revert.
- **(d) A person's narrowing takes effect at once.** A person's signed narrowing acts take effect immediately: revoking a key, revoking a delegation grant, or disabling a tool.
- **(e) One confirmation for every machine.** Confirmations are signed records on the board. Every machine verifies them against the committed roster of the version the confirmation replaces, without CM2. One confirmation serves all machines. The operator's direction that each machine runs what its operator confirmed still holds, because each machine verifies every confirmation itself.
- **(f) Protected first enrollment.** The first key is enrolled on first use at initialization, with the operator present. Every later enrollment is confirmed by an already enrolled key.
- **(g) Staged trust.** L0 is one operator and one authoritative board, which is now. L1 is several operators with majority confirmations. L2 admits public participants. The invariants stay the same across stages.
- **(h) Transport is not authority.** The channel sits behind an abstraction: a tailnet, a relay or something else. Being able to reach a node, or showing a network identity, grants nothing.
- **(i) Secrets stay out of agents.** Secrets never enter agent environments. Privileged operations run through the kernel's executor after authorization.

## 2. The interface with the core

The core decides which changes to the process wait for a person (process-configuration §2.4), which acts need a signature (§4.2), what may be delegated (§4.2) and what an approval binds (§6.4). This document gives the kernel three answers:

- **`confirmed(version)`**: whether a proposed version of the resolved configuration carries a valid confirmation (§4). The kernel runs a proposed version only when this answer is yes, or when the core's allowlist lets the version take effect on commit.
- **`narrowed(version in force)`**: the version in force with every person's narrowing act applied on top of it (§5).
- **`verify(act)`**: whether a signed act was made by a principal allowed to make it, with a key strong enough. The act may be a confirmation, an approval, a grant or a delegated act (§3, §6, §7).

In PC1 the trust component is a guardrail stub (§13). The core's rules are the same in every stage; only the strength of these three answers changes.

## 3. Persons, keys and enrollment

- **A person** is an operator principal (`op:…`, waggle §3) that is present in the roster of the version in force and has at least one enrolled key of a class below. Orchestrator, worker and host principals are never persons.
- **Presence classes.** Every enrolled key's record states its class:

  | Class | Key | What a verifier checks |
  | --- | --- | --- |
  | `fido-uv` | a FIDO2 key created with required user verification (`sk-ssh-ed25519`, waggle §4.4) | the FIDO attestation at enrollment; the user-verified flag in every signature |
  | `attested-app` | a key of the operator companion app in the phone's secure hardware (§8) | the platform's attestation chain at enrollment, naming the app and, where the platform attests it, the key's access control; presence in each use is enforced by the app and the phone's operating system, not written into the signature |
  | `local-os` | a Secure Enclave, Windows Hello or TPM key whose access control requires the operating system's user authentication | nothing in the signature: the evidence is the enrollment record, made in a local session under that authentication and confirmed like every roster change |

- **Where enrollments live.** The records sit beside the roster (waggle §3): `.waggle/allowed_signers`, the revocation list `.waggle/revoked` and the enrollment records. They are part of the resolved configuration (process-configuration §2.4). Adding a key or a principal is a change to the process and waits for a confirmation by an already enrolled key.
- **The trust anchor** (principle f). A board has no key until its operator enrolls one. `task-board trust init` records the anchor:
  - it runs in a local session and prompts for the operating system's user authentication;
  - it enrolls the operator's first presence key or keys;
  - it records the operator's principal as the board's first confirmer.

  It refuses to run in an agent session, which is a guardrail. It cannot complete without the person's authentication, which is the boundary. A board created before PC1 gets its anchor the first time its operator runs the command. Until then the board runs in legacy mode (process-configuration §14).
- **Later enrollments.** A second workstation key or a second phone is a change to the roster and waits for a confirmation by an enrolled key.
- **Recovery.** When every confirmer's keys are lost or revoked, nobody can confirm. The operator of the board's host re-anchors the board through the same local path, with the operating system's user authentication. The re-anchor is recorded on the board and reported at every load and to every former confirmer. It never runs in an agent session.

## 4. Confirmations

- **What is confirmed.** A proposed version of the resolved configuration that the core's classification sends to a person (process-configuration §2.4). One confirmation covers every change since the version in force.
- **The record** (principle c). The signed bytes bind:
  - the board's identity;
  - the proposed head: a commit, or an apply sequence number for a project without git;
  - the configuration digest of the proposed version;
  - the configuration digest of the version it replaces, which must be the version in force;
  - the confirmation's sequence number in the board's chain;
  - the confirmer principal;
  - the time.
- **The floor** (principle a). A confirmation is valid only with a signature by an enrolled key of a class in §3. `policy.signatures` never governs confirmations: it names which other acts need signatures, and may only add requirements to them. Nothing about a confirmation is read from the version it confirms.
- **Who may confirm.** A principal of the `people` group that `policy.confirmers` names in the version in force; without that key, the board's anchor operator. A proposed version's own `policy.confirmers` never decides who confirms it. The anchor operator confirms a board's first version.
- **Verification** (principle e). A machine verifies a confirmation against the version it replaces — the version in force — never against the proposed version or the parent of its head. It checks the roster, the revocation list, the enrollment records and the confirmers of that version. So a batch cannot admit its own confirmer. A record is accepted only when its replaced digest equals the digest of the version in force; a record that names any other replaced version is refused and reported.
- **No revival** (principle c). A proposed version whose configuration equals an older confirmed version is still a new change. It waits for a new record unless the core's allowlist lets it through. An old record never confirms a version again.
- **One confirmation for every machine** (principle e).
  - The record is committed to the board under `.task-board/.trust/`.
  - Every machine that loads the board verifies the chain of records from the anchor to the version in force, and runs that version.
  - A machine that cannot verify a record keeps running the last version it verified and reports why.
  - No machine runs a version that another machine merely claims.
- **The signer computes what it signs** (principle b). The signing device can be the machine's own command, the workstation client (§9) or the companion app (§8).
  - It receives the complete canonical resolved configuration of the proposed version and of the version in force.
  - It computes both digests and the authority report itself, and renders the report.
  - It signs only digests it computed.

  The kernel runs only a configuration that hashes to the signed digest.
- **Commands and prompts.** `task-board playbook confirm`, and the commands that grant or revoke a delegation, refuse to run in an agent session (a guardrail). The report prints in the terminal of the session that asks. The operating system's prompt carries the short form of the proposed digest and the number of marked changes, so that even a quick touch confirms something the person can check.

## 5. A person's narrowing acts *(principle d)*

- **The fast path.** A person can revoke a key, revoke a delegation grant or disable a tool, and the act takes effect at once. "A person" here means a principal of `people` in the version in force, signing with an enrolled presence key (from PC3; in PC1 the act is unsigned, §13). The act is recorded on the board and applied on top of the version in force, without the confirmers.
- **The matching commit follows as an ordinary change**: a line in `.waggle/revoked`, or a tool removed from `policy.enable.tools`. The narrowing record keeps applying until a confirmed version supersedes it. A version that would restore what the act removed waits for a confirmation, and that confirmation names the narrowing record it lifts.
- **Commits get no fast path.** A narrowing that someone commits without a person's signed act waits like any other change: the fast path belongs to people, not to commits. Wider narrowing acts, such as pausing a schedule, are open (§14).

## 6. Signatures and key assurance for board acts

`policy.signatures.require_for` (process-configuration §4.2) names which acts need a signature. This section decides what satisfies it.

- **`min_assurance`**: `software` < `hardware-bound` < `attested` (waggle §4.5). Whenever a project sets a signature policy, the default is `hardware-bound`, because any process running as the operator's OS user can obtain `software` signatures from `ssh-agent`.
- **`user_verified`**: `required` or `if-supported`. Whenever a project sets a signature policy, the default is:
  - `required` for approvals, relaxations, halts and delegations;
  - `if-supported` for spending, overrides, human transitions and installs.

  A project may set one value for every category, or a value per category, and may lower `required` to `if-supported`.
- **What meets `required`.**
  - A FIDO2 signature with the user-verified flag always does, because anyone can check the flag.
  - An attested companion key meets it for confirmations (§4). For other board acts it is **pending the operator** (§14). Decision 4a asked for a mark in the signature that anyone can check. On iOS the platform may attest the app without attesting the key's biometric access control, and this needs checking against the platform's documentation.
  - A `local-os` key meets the confirmation floor (principle a), because its presence is proven at enrollment. For other board acts it meets only `if-supported`, because its presence cannot be seen in a signature.
- **Rendering.** A board act is signed only through the operator-facing command that renders its payload ("what you see is what you sign", waggle §4.2). An agent never holds an operator key.
- **Before CM2.** Signed board acts other than confirmations and narrowing acts need the forms of waggle A1, which arrive with CM2. The core's validator warns about categories that have no signable form yet (process-configuration §10). Confirmations and narrowing acts are signed board records from PC3 and need no carrier.

## 7. Grants and delegated acts

The core says what may be delegated, to whom and under which ceilings, and how delegated acts count (process-configuration §4.2). This section says how the kernel knows who acted.

- **Whose sessions act.**
  - In PC1, the session host's marks say who started a session and which role alias it runs. The marks are a guardrail, and until the session host sets them the kernel records grants but counts no delegated act (process-configuration §15).
  - From CM2, a session acts under a grant when its certificate was issued by the grantor's key (waggle §3) and its extension `waggle-role@relux.works` names the delegate role, checked against the version in force.
  - A grant may also name session principals explicitly, never by pattern.
- **A grant is a person's act.** It needs the grantor's confirmation at the floor of §4: from PC3 a signature by the grantor's enrolled presence key, in PC1 the unsigned stub. Where the project requires signatures for `delegations`, it is also a signed board act under waggle A1 §19.8. The kernel verifies a grant once, when it records it; later delegated acts refer to the recorded grant.
- **Delegated acts.** A signed delegated act carries the delegate session's signature in the `delegate` role, and the kernel verifies the chain from the grantor's key to the act (waggle A1 §19.8).
- **Revocation.** Revoking a grant is a narrowing act (§5).

## 8. The operator companion app and pairing

The operator companion app is the common way a person confirms and approves, and the way a headless node is served. It is a track of its own.

- **The key.** The app holds a key in the phone's secure hardware (Secure Enclave on iOS, Android Keystore) whose own access control requires the person's biometric authentication. So the phone's operating system, not only the app, refuses a use without the person.
- **Pairing.** The app is paired with the node through an out-of-band handshake when the node is initialized. The node shows a QR code in the terminal where the operator runs the initialization, locally or over SSH. On a node without a screen or camera path, the operator can type a short pairing code instead; both screens then show a short authentication string, which the operator compares. Each side pins the other's key. Pairing keeps four invariants:
  - the node verifies the attestation of the app's key at every pairing, and refuses a key without it. The attestation must show that a genuine companion app generated the key in the phone's secure hardware, naming the app and, where the platform attests it, the access control. Every verifier re-checks that evidence against the platform vendors' roots and never trusts a stored verification result alone (waggle A1 §19.5);
  - the pairing secret is single-use, expires within minutes, and never enters a hosted session's transcript or any log;
  - pairing refuses to run in an agent session;
  - every later pairing, a second phone included, is an enrollment that an enrolled key confirms (§3).
- **Transport** (principle h). Requests travel over an authenticated channel behind an abstraction: the operator's tailnet, or a relay that carries only messages encrypted end to end to the pinned keys, so the relay can read and forge nothing. Push wakes the app.
- **Rendering.** The app renders what is being approved, and signs only what it rendered and computed itself (§4, principle b).
- **A lost phone** is revoked by a narrowing act from another enrolled key (§5).
- **Open questions** (§14): the design and platforms, iOS first; administration of the node over SSH from the app; the app on a desktop, managing dedicated nodes.

## 9. Headless nodes before the app

Until the app exists, a headless node is confirmed from the operator's workstation.

- A client command on the workstation fetches the complete resolved configurations of the proposed version and of the version in force from the node over SSH. It computes the digests and the authority report itself, renders the report, and signs (§4, principle b).
- The signing key must be a FIDO2 key created with required user verification. The node checks its FIDO attestation when the key is registered, and the user-verified flag of every signature.
- The first registration on a node happens in a local session under the operating system's user authentication, or is proven by that attestation (principle f).
- A second workstation key is an enrollment that an enrolled key confirms.
- Registration refuses to run in an agent session.
- The key is never used through a forwarded `ssh-agent`, and the client refuses to sign through one. A forwarded agent lets any process of the operator's account on the node request signatures while the session lasts, and a key that shows nothing approves whichever request reaches it first.

## 10. The operator's files

The operator's private files under `~/.curator/` are listed in process-configuration §2.3. Entries that widen admission take effect only once the operator confirms them *(decided 2026-09-24, P3)*.

- **The effective version.** The resolver keeps the effective version of each file: the last confirmed version, narrowed by every narrowing edit made since. A narrowing edit applies at once and becomes the new baseline.
- **Widening entries.** A change that admits more than the effective version is a widening entry:
  - a model or a billing class added to `[allow]`, metered billing included;
  - a `[bindings]` candidate outside the effective admission;
  - a standing answer added or extended, or a lengthened review period;
  - a window widened;
  - a runtime binding added to `runtimes.toml`;
  - a skill source outside the Curator registry admitted in `curator.toml`;
  - a confirmation level lowered.

  Before the first confirmation the baseline is the file-less one: no metered billing and no standing answers.
- **`[confirmation]`** sets, per kind (`billing`, `models`, `answers`, `windows`, `runtimes`, `skills`), what a widening needs:
  - `none`: it takes effect at once;
  - `user`: a person's confirmation, as in this section;
  - `second-person`: the confirmation of another operator. It becomes available with waggle's signed acts in CM2; until then the commands refuse it, and a hand-edited `second-person` reads as `user`.

  The defaults are `user` for metered billing, standing answers and skills from unknown sources, and `none` for the rest. The `skills` kind can never go below `user`. Lowering a level is itself a widening and needs the level it lowers.
- **The record.** The operator confirms with `curator models confirm` and the same verb for the other files. The command refuses to run in an agent session. It computes the digest of the widening entries itself, shows them, and signs with a presence key registered on this machine (`local-os` or `fido-uv`), per principle b. The resolver verifies the record before it honours a widening entry. These confirmations are machine-local, because the files are.
- **Billing classes come from facts.** A runtime's billing class comes from how the harness is logged in, never from what a binding says, so a `runtimes` level of `none` never admits metered billing.
- **Until the commands ship**, the resolver honours no widening entry whose level is not `none`, and reports each one it ignores.

## 11. Cross-board hand-off results

A hand-off result that returns across boards is signed by a host key of the receiving board, in the signer role `result` (waggle A1 §19.4), when that board's kernel records the terminal transition.

- No orchestrator session holds that key.
- An OS account other than the operator's, or a privileged service, holds it, whether or not it is `hardware-bound`. So no process running as the operator can sign a result.
- The offering board verifies the signature against the host principals that its committed project file binds to the other board (process-configuration §6.5).

## 12. Where things live, and what still depends on the OS

| What | Where | Protected by |
| --- | --- | --- |
| roster, revocation list, enrollment records | committed under `.waggle/` | part of the resolved configuration: changes wait for a person, and a person's revocation acts at once |
| confirmation records and narrowing acts | committed under `.task-board/.trust/`, signed | their signatures, verified against the version in force |
| the resolved configuration of the version in force | committed on the board with its digest | at every load, every machine recomputes the digest and verifies the record chain back to the anchor. It fails closed: it runs the last version it verified, or the built-in default when none verifies, and reports why |
| confirmations of the operator's files | machine-local under `~/.curator/` | signatures by keys registered on that machine; OS-level protection is open |
| keys | secure hardware; public keys in the roster or the machine's registry | presence (§3) |
| hand-off host keys | a separate OS account or a privileged service | the account boundary (§11) |
| the `task-board` and `curator` executables | the operator's account today | open: any process of that account can replace them |

The claim that "no session can edit its own authority" holds only against agents that do not replace the kernel, its stores or the local key registry. Until OS-level protection exists, that part is a guardrail. The path to a real boundary is to run agents under their own OS account, while the operator's account owns the stores, the binaries and the keys (§14).

## 13. Stages and delivery

| Stage | Who | Delivered |
| --- | --- | --- |
| L0 | one operator, one authoritative board | **PC1**: the guardrail stub. A confirmation, a grant or a narrowing act is an operator CLI act outside any run, unsigned, and recorded with the fields of §4; the anchor records the operator's principal without keys. Everything the core specifies runs; nothing is a boundary. **PC3**: presence-bound keys and enrollment, the anchor with keys, signed confirmations and narrowing acts as board records verified against the roster, the workstation client (§9) and the operator-file confirmations (§10). No CM2 is needed. |
| L0, wider | several machines of one operator | **CM2**: waggle A1's forms for approvals, grants, delegated acts, events and hand-off results, and the carrier and lease store that budgets and revocations need across machines; `second-person` for the operator's files |
| L1 | several operators | confirmations by a majority of the confirmers group, a quorum the group states (open, §14); the same records, chain and floor |
| L2 | external participants (wild bees) | admission under leases and signatures; reputation lowers the cost of verification, never grants trust (later) |
| any | the companion app | its own track (§8) |

## 14. Open questions

1. **OS-level protection and a separate account for agents.** This covers the operator's files and their confirmations, the local key registry, the stored configuration's checkout, and the kernel's and Curator's executables. The options are file permissions, edits only through a privileged helper or `sudo`, a separate owning account, and immutability flags.
2. **Pending the operator: companion signatures for board acts** (D3). Whether an attested companion signature meets `user_verified: required` for board approvals and other board acts, beyond confirmations. The platform attestation of the key's access control needs checking on iOS first.
3. **L1 quorum.** How the confirmers group states a majority, and how a quorum of confirmations is recorded.
4. **Wider narrowing acts**, such as pausing a schedule or applying a first version's data-policy narrowing before it is confirmed.
5. **The companion app**: its design and platforms, a desktop variant that manages dedicated nodes, administration over SSH, and the pairing code.
6. **Recovery**: how a re-anchor is announced and how former confirmers can contest it.

## 15. Findings of the sixth refuter run answered here

The run reviewed draft v3.5 ([`../reviews/2026-09-24-refuter-v5.md`](../reviews/2026-09-24-refuter-v5.md)). Process-configuration §26 maps every finding; these are the ones this document answers.

| Finding | Resolution |
| --- | --- |
| U1 an agent could sign `board.confirm` from CM2 | the floor of principle (a): only enrolled presence keys confirm, and `policy.signatures` never governs confirmations (§4, §6) |
| U2 a batch could admit its own confirmer | verification against the version in force, never the parent of the head (§4) |
| U3 the pinned revision was the unconfirmed head | the core redefines it as the revision of the version in force; nothing here reads the proposed version (§4; process-configuration §2.4) |
| U7 an old record re-confirmed a reverted version | records bind the board, the head, both digests and a chain sequence; no revival (§4) |
| U8 a revocation waited for a confirmation | a person's narrowing acts take effect at once (§5) |
| U9 per-machine versions contradicted §7.1 | one confirmation for every machine, verified by each (§4); the operator's per-machine direction holds through each machine's own check |
| U10 the signer signed a digest the node computed | the signer computes the digests and the report itself (§4, §8, §9) |
| U11 the first workstation key was unprotected | FIDO2 with user verification and attestation only; first registration local or attested; later ones confirmed (§3, §9) |
| U12 confirmers were unenforceable and undefined | confirmers read from the version in force; the anchor and first-version rule; recovery (§3, §4) |
| U14 an agent session could raise the prompt | the commands refuse agent sessions; the prompt carries the digest and the count of marked changes (§4) |
| U19 companion signatures and iOS | aligned with waggle A1 §19.5; board acts other than confirmations pending the operator (§6, §8, §14) |
| U20 where the confirmed configuration lives | the table of §12; the kernel recomputes and verifies at every load and fails closed; the executables named |

# Music video: a generality check against a real production

This example expresses a real, publicly documented production as a playbook: a five-minute lyric music video made in about nineteen hours of wall-clock time by one human director and an agent production team (September 2026). The author published the full record: the direction log, the orchestration prompts, every image, song and clip prompt, and the cost. It is the most complex non-software process we have tested the model against, and it went viral, so it doubles as a demonstration.

Files:
- [`music-video.playbook.json`](music-video.playbook.json): the template.
- [`project.Playbook.json`](project.Playbook.json): a project using it, with the director, the tools, spending delegation and machine limits.

Both validate against [`../../schema/playbook-v1.schema.json`](../../schema/playbook-v1.schema.json). None of the original prompts is copied here; the playbook describes the process, not the video.

## The production in numbers

| | |
| --- | --- |
| Song | 5:06, generated; tempo drifting 131.5 → 133.9 BPM |
| Edit | 141 shots in 9 chapters; every word on screen on its time |
| Images | 188 prompts, 636 images, 130 plates used |
| Clips | 47 clips plus 30 continuation takes, 91 submissions, about 93 USD |
| Agents | one production-lead session plus fan-out workflows: 31 agents for research and lyrics, 7 for the storyboard, 40 runs for animation |
| Tokens | 1.4 billion, about 626 USD at list prices; 96 % were cache re-reads of a ~565k-token main session |
| Director | about 50 messages: taste calls, checkpoints, a few decisive notes |

## How the process maps

| In the production | In the playbook | Spec |
| --- | --- | --- |
| The director owns taste and a handful of checkpoints (concept, casting pick, song take by ear, storyboard, the cut) | `director`, a human role; `approvals` guards on `lead.picking`, `song.picking`, `storyboard.approval`, `cut.review` | §4.2, §6.4 |
| The main session plans, writes prompts and briefs, assembles the edit | `orchestrator`, the root role; it owns planning, `plate.prompting`, `clip.queued`, `cut.assembling` | §6.1 |
| Research sweeps in parallel, each fact adversarially verified, then one ranked bank | `research` creates one `sweep` per angle (`create` with `per`); each sweep goes `researching → verifying`; `synthesizing` waits for `children.terminal` | §6.3, §7 |
| Four lyricists from different angles, three judges with different lenses, a head writer merges, a critic checks | one `draft` per angle; `lyrics.judging` is owned by the head writer with three **blind helpers** (`judge`); `critique` loops back under `critic_rounds` | §4.2 (helpers) |
| Three storyboard directors, judges, a head director merges; a constraint script audits the board | the same judged-merge pattern in `storyboard.merging`; `board.check` is a **tool** state that fails the merge back under `board_fix_limit` | §6.11 |
| Image generation in the director's own account, song generation, audio analysis on a GPU box, clip generation, render | tool roles over commands exported by skills installed through Curator: `mj.generate`, `song.generate`, `audio.analyze`, `clip.generate`, `render.master`; the kernel's executor runs them, never an agent | §6.11 |
| Each image grid is picked for identity, framing and room for type; wrong ones re-prompted | `plate.picking` owned by the `art-director` (vision required); `changes_requested` returns to `prompting` under `plate_rework_limit` | §6.2, §7.3 |
| Clips cost money; the director set a target and a cap | `clip.queued → generating` is marked `risk: spending` and needs a spending approval; the project lets the director delegate it to the orchestrator up to 5 USD per clip, and the director's grant carries the budget for the project | §4.2, §6.4 |
| Lip-sync is measured, not eyeballed; where it drifts, cut and continue from that frame; a lip-sync model is the fallback | `sync.check` is a tool state; a failure goes to `diverged`, which records the cut and queues a continuation take (a fresh spending approval); after `cover_limit` takes, `lipsync.repair` | §6.2, §6.11 |
| One animator per chapter, an art director renders and reviews real frames, the animator fixes, a render script checks every word is on screen and the frame time | `chapter`: `animating → review → fixing` under `chapter_rework_limit`, then the `render.check` tool | §6.11, §7.3 |
| Mid-production discoveries change later work (identity in the prompt text; audio-conditioned clips; keep inputs in colour, apply the look in post) | `decision` elements; later briefs read them, and the refuter can check new drafts against them | §12 |
| "If I don't answer in five minutes, go" | not silence: the director grants a delegation for that one storyboard with the wait and an expiry; the project allows it for `storyboard.approval` only | §4.2 |
| One GPU box, one render machine | `policy.wip.per_state` on the states that use them | §6.8 |
| "Log all our learnings so the next video is faster" | the `retrospective` state, owned by a librarian; proposed playbook changes wait for the director's confirmation like any process change | §2.4 |

## What the model makes better

- **Tools run outside the orchestrator's context.** In the original, the main session ran every generator, analysis and render script itself, and its context grew to about 565k tokens; 96 % of 1.4 billion tokens were re-reads. Here those steps are tool states run by the kernel's executor: the orchestrator reads a result, not a transcript.
- **The fan-outs are declared, visible and resumable.** The original ran them as throwaway scripts in one session. Here each draft, sweep, plate, clip and chapter is an element with its own state on the board, so a crash resumes at the element, not at the script.
- **Spending is bounded by the kernel**, not by a sentence in a prompt: the ceiling per clip and the director's budget are checked before any generation starts.
- **Every loop is a parameter** a project tunes, including an unbounded director review of the cut.

## Gaps this example found

Items 1–3 are now proposed for PC3 in spec §6.11.1 (DRAFT v4.6); items 4–6 in §6.14 and §6.16 (DRAFT v4.7), which also cover files, pinned inputs and staleness (§5.5) and long-running tools on machine resources (§6.11.2).

1. **Tool results as fields.** `sync.check` knows where a clip diverged and `clip.generate` knows what it actually cost, but a tool today reports only pass or fail plus evidence. Proposed: a tool may declare output fields that the kernel writes from its result (`diverged_at`, the actual `cost`), so guards, ceilings and later briefs read them.
2. **Actual spending against the budget.** The ceiling reads the estimated cost at approval; the provider's receipt should reconcile the budget. This joins the open item on unknown outcomes and receipts of side-effecting tools.
3. **Tools that need a person mid-run.** A captcha or a lost sign-in in the director's browser fails the invocation here and routes to a human-owned `assist` state. An assisted tool that pauses and rings the person would keep the invocation alive. The same case raises a trust question: a tool that uses the director's signed-in browser must run under the director's account, not the agents'.
4. **Estimate units.** Generated seconds are estimated in `points`; capacity in seconds or money would read better with a declared unit.
5. **Typed helper answers.** Judges return scores per draft; the spec has blind helpers but no typed answer shape, so the merge reads free text. Per-hook schemas arrive with the provider modules in PC1.
6. **The `vision` capability** must exist in the model registry's vocabulary for `needs.require.capabilities`.

Nothing in the kernel had to know what a music video is.

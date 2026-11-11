# Workflow and Ownership

Version: 0.7 | Date: 2026-11-10
This document becomes normative when the producer adopts the v0.7 package.

## 1. Roles and decisions

The producer owns the GDD, scope, game design, art direction, binding targets, policy changes, and milestone acceptance. The lead owns coordination, integration decisions, and every Git commit within those boundaries. Implementation owners choose reversible details within approved contracts. tester owns all retained tests and the gate. Reviewer reports evidence and findings without fixing the reviewed source.

| Role | Runtime | Intended model and effort | Writes |
| --- | --- | --- | --- |
| lead | Claude Code, Agent Teams lead | `claude-fable-5-2`, high | Coordination records and all Git commits |
| gameplay, systems, lookdev | Agent Teams teammates | `claude-opus-5-5`, high | Game source in their own paths |
| tester | Codex, build worktree | `gpt-6-astra`, high | Retained tests, test fixtures and config, gate runner, completion-hook adapter, performance harness |
| art_tools | Codex, build worktree | `gpt-6-astra`, medium | Assets, generators, targets, tools in its own paths |
| reviewer | Codex, review worktree | `gpt-6-astra`, high | Review reports only |
| deps | Claude Code subagent | `claude-haiku-5-5` | Dependency entries in `package.json` and the lockfile only |

The producer decided on 2026-11-10:

- The three Claude teammates write the game source. This applies to this project even where a machine-wide default names a different single writer.
- Testing is independent of implementation. tester (GPT) writes every retained test from the task's acceptance criteria and GDD documents. Implementers never edit tests; tester never edits the source under test.
- The lead makes all Git commits.

Verify each role's actual account, client, model, effort, and tool support before use.

Dependency changes: the lead approves a package and exact version; systems invokes the `deps` subagent (`.claude/agents/deps.md`), which installs it, updates the lockfile, and reports the change. If the runtime does not let a teammate start a subagent, the lead invokes it. The lead commits the result. Browser binaries and other pinned tool installs follow the same path.

For a missing decision, record the question, options, recommendation, affected tasks, decision owner, and the last point at which work can proceed safely. Stop the affected dependency chain, not unrelated tasks. Silence is never approval.

Project records:

- `docs/gdd/` (producer): the GDD: a one-page GDD plus a short document for each system that needs more detail, with permanent document IDs; see [traceability](traceability.md).
- `docs/decisions.md` (producer; lead may draft entries for approval): durable approvals and recorded deviations.
- `tasks/milestones/<milestone>.md` (lead): status, candidate SHAs, and links to the producer's acceptance.
- `handoff/<role>/environment/<run_id>.md` (each role): dated runtime observations with exact versions.

## 2. Ownership map

More specific rows take precedence over broader ones. Unlisted paths need an assignment from the lead before editing.

| Paths or operation | Owner | Notes |
| --- | --- | --- |
| `src/core/` | systems | Stable contracts; changes follow contracts.md |
| `src/systems/`, `src/ui/`, `data/rules.json`, `data/levels/` | systems | Game loop, UI, rules, levels |
| `src/editor/`, `tools/dev/` | systems | Editor framework, dev writeback, commit helper, `ownership.json` |
| `src/main.ts`, root build and lint config, `package.json` scripts | systems | Composition root; other roles request script changes by task |
| Dependency entries in `package.json`, `package-lock.json` | deps | Only after lead approval |
| `.claude/` | systems | Producer-approved settings, hook wiring, and agent definitions only |
| `src/player/`, `src/camera/`, `src/mechanics/`, `data/tuning.json` | gameplay | The player's verbs and the core mechanic (combat, puzzle rules, card effects, and so on, per the GDD); simulation and camera adapters stay separate modules |
| `src/visual/lighting/`, `src/visual/shaders/`, `src/visual/post/`, `src/visual/vfx/`, `data/visual/lighting.json`, `data/visual/vfx/` | lookdev | Lighting, shaders, post-processing, VFX |
| Remaining `src/visual/` and `data/visual/`, `targets/`, `assets/` (including `assets/prompts/`) | art_tools | Generators, palette, registry, asset promotion, prompt archive |
| `tests/` (including `tests/regression/` and fixtures), test-runner and Playwright config | tester | The only committed tests |
| `tools/gates/`, `tools/perf/` | tester | Gate runner, gate worktree, completion-hook adapter, layout check, performance harness |
| Remaining `tools/`, including `tools/validate_assets.ts`, `tools/trace/`, export and target utilities | art_tools | |
| Colocated `*_panel.ts` | Owner of the data it edits | Imported only through development composition |
| `tmp/probes/<role>/` | That role | Git-ignored; emptied before a task completes |
| `AGENTS.md`, `CLAUDE.md`, `docs/` (including `docs/gdd/`), `roles/` | producer | Agents draft changes in a handoff |
| `tasks/codex/` | lead | The assigned Codex role edits only its card's status fields and log |
| `tasks/milestones/` | lead | |
| `tasks/manual_requests/` | Requesting role | One file per request |
| `handoff/<role>/`, `qa/evidence/<task_id>/<role>/` | Named role | Append; never overwrite another author's records |
| Review reports outside the reviewed tree | reviewer | No edits to tracked source, tests, data, or configuration |
| Git commits | lead | Through the commit helper (§4); the producer may commit their own tuning and GDD edits; the producer creates milestone tags |

Producer edits to approved fields in `data/` take precedence over file ownership. Record the approved field paths and source revision in a `chore(tuning)` commit or a decision entry.

The producer drops manual-generation results into `assets/inbox/<request_id>/`. The requesting role may inspect them; art_tools promotes them into managed paths and updates the manifest, credits, and prompt archive.

Ownership changes need the old writer to release or be confirmed stopped, a handoff of its uncommitted files, and a lead assignment within policy. A permanent change to this map needs producer approval.

## 3. Task records

One task ID space (`T-<nn>`) covers Claude and Codex work. The lead allocates IDs. Handoffs, commits, and evidence use the same ID.

**Claude roles: the Agent Teams task list is canonical.** The lead creates each task with the card fields below in its description and uses native dependencies. When a blocked owner or one who needs a decision messages the lead, the lead adds a `blocked:` or `needs_context:` line with the reason. No separate event log is kept.

**Codex roles: one card file per task, status inside the card.** The lead writes `tasks/codex/<task_id>.md`. The assigned Codex role edits only `status`, `status_note`, and `status_at`, and appends dated lines under `## Log`. Any other change is a new card revision by the lead. Status values: `ready`, `in_progress`, `ready_for_commit`, `ready_for_gate`, `done`, `blocked`, `needs_context`, `cancelled`.

Card fields (values are illustrative, not approved design):

```yaml
id: T-07
revision: 1
milestone: M1
owner: gameplay
gdd_refs: [GDD-CORE_MECHANIC@r2]
depends_on: [T-02]
allowed_paths: [src/mechanics/, data/tuning.json]
paired_test_task: T-07t
acceptance:
  - The core verb resolves in the order and under the conditions in GDD-CORE_MECHANIC §2.2.
  - Its timings and parameters come from validated tuning data and appear in the tuning panel.
out_of_scope: [feedback_animation, vfx]
timing_or_performance: true
verification_profile: integration
open_decisions: []
# Codex cards only:
status: ready
status_note: ""
status_at: 2026-11-10T15:00:00-05:00
```

`timing_or_performance: true` turns on the full evidence manifest (validation.md §4).

**Paired test tasks.** When a task's behavior falls in a retained-test category (validation.md §1), the lead creates a paired task for tester at the same time, with the same `gdd_refs` and acceptance criteria. tester drafts the tests from those criteria immediately and finalizes them against the implementer's commit. A new test that fails becomes a failure report with reproduction steps; the lead turns it into a fix task for the implementer. A milestone candidate requires every paired test task to be done.

**Task flow:**

1. The owner finishes the work, deletes probes, and writes the handoff with `changed_paths` and `gdd_refs`.
2. The owner tells the lead the task is ready for commit (Claude: message; Codex: card status `ready_for_commit`).
3. The lead commits it (§4) and reports the SHA (Claude: `commit:` line in the task; Codex: card log).
4. The owner marks the task complete. For Claude tasks the completion hook runs the gate on that SHA; Codex roles run `npm run gate:sha -- <sha>` and set `done` when it passes.

Each owner has one active implementation task by default. Other ready tasks are a backlog, not a concurrency target. Report a dependency failure once with the required action; retry when relevant evidence or state changes.

Handoff fields: `task_id`, `task_revision`, `gdd_refs`, `outcome` (`DONE`, `DONE_WITH_CONCERNS`, `NEEDS_CONTEXT`, `BLOCKED`), `changed_paths`, `commit` (added after the lead commits), `completed`, `not_completed`, `verification` (command and exit result), `evidence`, `risks`, `open_decisions`, `next_owner`. `DONE_WITH_CONCERNS` cannot satisfy a blocking acceptance criterion without a producer decision.

Manual requests (`tasks/manual_requests/<request_id>.md`) record the request ID, requesting role, provider and model, full prompt, parameters, reference inputs, expected output, `gdd_refs`, and the destination `assets/inbox/<request_id>/`. A fulfilled request becomes a prompt record ([traceability](traceability.md) §4). Missing output blocks only work that depends on it; placeholder-based work continues.

## 4. Commits and the gate

All writers share the build worktree. The lead is the only committer, and the gate tests commits in a separate worktree, so writers never need to pause.

- **Commit helper.** systems provides `npm run commit:task -- <task_id>`. It reads the task's handoff, refuses any changed path outside the owner's paths in `tools/dev/ownership.json` (a machine-readable copy of §2) or outside the card's `allowed_paths`, stages only those paths, shows the staged diff, and commits with a Conventional Commit message and the trailer `Task: <task_id>`. The lead inspects the diff before confirming. Never use `git add -A`, `git add .`, or `git commit -a`.
- **Gate worktree.** tester's runner checks out the commit under test, detached, in its own worktree with separate `node_modules`, build output, browser profile, and ports. Runs are queued and execute one at a time. Uncommitted files in the build worktree are never part of a gate run.
- **Dependencies between owners.** A commit that relies on another owner's uncommitted file fails the gate. Report that to the lead as a dependency; do not edit the other owner's file.
- **Data-only commits** (every changed path under `data/`, including producer tuning) need a passing gate and nothing else. They do not go to the reviewer.
- **Behavior-changing commits** (source, shaders, tests, assets, runtime-affecting tools, configuration) go to the reviewer at the next review point (validation.md §5).

Use Conventional Commits with `feat`, `fix`, `refactor`, `perf`, `test`, `docs`, or `chore`; the scope names the area. Producer tuning uses `chore(tuning)`, GDD changes use `docs(gdd)`. Commit on the `jam` branch only after checking that it exists and is intended. Pushing needs explicit authorization. No force pushes, history rewrites, or resets that discard another writer's work. Never commit secrets or `.env` credentials.

## 5. Bootstrap and milestones

Bootstrap has no game-content gate yet. systems sets up the project, commit helper, ownership file, and one editor save slice; tester sets up the gate runner, gate worktree, hook adapter, and `tests/regression/` layout; art_tools supplies the asset validator and trace checks, or reports which is still missing. Record what exists and what does not. Do not require a nonexistent `npm run gate` to create itself, and do not replace missing stages with no-ops that pass.

M0 approves a minimal working slice and stable contract meanings, not every future event or field. Build only the panels and pipelines the current milestone needs. Prove an external generation path before the first task that depends on it.

| Milestone | Required outcome | Producer decision |
| --- | --- | --- |
| M0 | App runs; one tunable field previews, saves, reloads, and detects conflicts; gate, commit helper, and snapshot rehearsal work; initial contracts and environment recorded | Accept foundation and initial contract meanings |
| M1 | The core mechanic playable with placeholder visuals: the player's verbs from the approved one-page GDD and core gameplay system documents, and the minimum opposing or interactive element they act on (for example a dummy enemy, one puzzle, one hand of cards); relevant panels, paired tests, and approved target constraints available | Accept or adjust basic feel |
| M2 | A complete start/play/end loop and one representative level | Continue, revise, or change direction |
| M3 | Everything on the approved content list present: levels, characters and objects, UI, assets, lighting, and VFX, as far as the GDD calls for them | Accept content and level/visual editing |
| M4 | Agreed feel and performance criteria pass; all blocking defects resolved | Explicitly accept any deferred non-blocking issue |
| M5 | Features frozen; final build, recording, report, replacement manifest, and portable exports exist | Accept the final revision and authorize port planning |

Before a milestone's implementation starts, every GDD document with `needed_by` at or before that milestone must have `status: approved` ([traceability](traceability.md) §1). Before M1, the approved one-page GDD and the detail documents marked `needed_by: M1` must together cover the concept, the core loop, the success/failure loop, platform, input, camera or view, time behavior (pause, turns, and any time scaling), the player's verbs and core rules, the feedback those rules depend on, and provisional tolerances. The writing guide (`docs/templates/gdd_guide.md`) shows where each item goes. A rule too vague for tester to test belongs in a detail document, even if the one page mentions it. Genre-specific rules (for example buffer and cancel windows in an action game, or turn order in a card game) go into the relevant gameplay system document. Owners may prototype alternatives while these are open, but may not call an invented default approved.

After M1, development continues piece by piece. Before the first implementation task for a gameplay system, a set of screens, a level, the content plan, or the art and audio set, the producer approves its detail document under `docs/gdd/` (`systems/`, `ui/`, `levels/`, `content/`, or `art/`, from the matching template); tasks then reference that document in `gdd_refs`, not only `GDD-ONE_PAGE`. The one-page GDD stays current as the summary and index; detail goes into the detail documents.

Milestone close:

1. The lead commits all finished work, including paired tests; the gate passes on the candidate commit.
2. The producer tags that commit `snap/mN-k`.
3. Reviewer reviews the candidate; the lead reviews art_tools and tester changes in the same range.
4. The producer playtests and tunes with the editor. Tuning commits are data-only and need the gate only.
5. Repairs that change behavior create a new candidate; reviewer reviews only the delta since the last reviewed candidate.
6. The producer accepts a named SHA, tagged `snap/mN`. Never move an existing tag.

## 6. Documentation boundary

The Chinese explainer for the producer explains why the workflow is shaped this way, when the producer steps in, and how to operate it, and links to these documents. It does not keep a second ownership table, launch configuration, or definition of done. Runtime observations go in dated handoffs with exact versions.

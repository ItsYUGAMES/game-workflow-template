# Workflow and Ownership

Version: 0.8 | Date: 2026-11-13
Proposed policy for adoption. Read the section needed for the current task.

## 1. Roles, activation, and decisions

The producer owns scope, design, art direction, binding targets, policy adoption, and milestone acceptance. The lead coordinates and commits. Owners choose reversible implementation details within approved constraints. Tester independently writes retained tests; reviewer does not edit the reviewed implementation.

The existing requested roster is preserved below. Labels are configuration intentions, not proof of model availability. Verify effective model/client once when starting a role and after an environment change. Do not launch everyone simply because this table lists them.

| Role | Requested runtime/model | Work |
| --- | --- | --- |
| lead | Claude Code, `claude-fable-5-2`, high | Coordination and Git commits |
| gameplay, systems, lookdev | Claude teammates, `claude-opus-5-5`, high | Assigned game-source areas |
| tester | Codex, `gpt-6-astra`, high | Risk tests, integration/candidate verification |
| art_tools | Codex, `gpt-6-astra`, medium | Assets, prompts, targets, tools |
| reviewer | Codex, `gpt-6-astra`, high | Read-only candidate/delta review |
| deps, optional | Claude subagent, `claude-haiku-5-5` | Substantial independent approved installs |

Default activation is lead plus the current feature owner. Add another owner only when its work is independent, paths do not overlap, and its result is useful without repeated coordination. Invoke tester at risk-bearing integration batches and milestone candidates; invoke reviewer for a meaningful candidate/delta. Idle specialists need not stay live.

Lead approves new dependencies and exact versions. systems may install them directly. Delegate to deps only when it is a worthwhile independent job. Existing authorized packages need no repeated approval conversation.

For a genuine missing decision, report the question, recommendation, affected work, and last useful independent step once. Group non-blocking questions for the next review boundary. Silence does not approve a new gameplay rule or binding art direction.

## 2. Ownership and records

Specific paths override broader rows. Transfer ownership after release or confirmed stop; preserve uncommitted work.

| Paths/operation | Owner |
| --- | --- |
| `src/core/`, `src/systems/`, `src/ui/`, `src/editor/`, `src/main.ts` | systems |
| `data/rules.json`, `data/levels/`, `tools/dev/`, root build config, `.claude/` | systems |
| `package.json`, `package-lock.json` | systems; optionally leased to deps for approved installs |
| `src/player/`, `src/camera/`, `src/mechanics/`, `data/tuning.json` | gameplay |
| `src/visual/lighting/`, `src/visual/shaders/`, `src/visual/post/`, `src/visual/vfx/`, `data/visual/lighting.json`, `data/visual/vfx/` | lookdev |
| Remaining `src/visual/`, `data/visual/`, `assets/`, `targets/`, remaining `tools/` | art_tools |
| `tests/`, test/Playwright config, `tools/gates/`, `tools/perf/` | tester |
| Colocated `*_panel.ts` | Data owner; imported through development composition only |
| `AGENTS.md`, `CLAUDE.md`, `docs/`, `roles/` | Producer; agents may edit a scoped draft when requested |
| `tasks/codex/`, `tasks/milestones/`, `tasks/completed_refs.jsonl` | Lead; assigned owner updates its status/handoff |
| `tasks/manual_requests/`, `handoff/<role>/`, `qa/evidence/<task_id>/<role>/`, `tmp/probes/<role>/` | Named/requesting role |
| Review reports | reviewer, outside the reviewed source tree |
| Git commits | lead; producer may commit own tuning/GDD edits |

A generating role may own its explicitly assigned, unique new `assets/prompts/<resource_id>/<generation_id>.md` records. art_tools retains archive/index and manifest ownership; this exception avoids a separate handoff per call.

Producer-approved values override path ownership. Record changed field paths/revisions; preserve newer edits. Lead assigns unlisted paths once.

Records have one home: `docs/gdd/` for design, `docs/decisions.md` for decisions, `tasks/milestones/` for candidates, and `handoff/<role>/environment/` for observed environments. Create records when needed, not an empty documentation tree. Link existing records instead of copying their text into tasks.

## 3. Bounded tasks and context reuse

Use one `T-<nn>` ID space. Claude tasks live in the native task list; Codex uses `tasks/codex/<task_id>.md`. Do not assume task state is shared across clients. Dependency messages contain the result and artifact/revision only.

Use [the task packet](templates/task_packet.md) for a coherent feature or asset batch. Avoid separate tasks for each file, function, screenshot, or check command.

```yaml
id: T-07
revision: 1
milestone: M1
owner: gameplay
gdd_refs: [GDD-CORE_MECHANIC@r2]
depends_on: [T-02]
allowed_paths: [src/mechanics/, data/tuning.json]
acceptance:
  - The approved core verb completes under the specified conditions.
out_of_scope: [new_mechanics]
context_refs:
  - docs/gdd/systems/core_mechanic.md#rules
  - docs/contracts.md#3-time-and-reproducibility
base_commit: <actual_sha>
verification_profile: local
timing_or_performance: false
status: ready
```

Include only this task's constraints, relevant sections, and acceptance. Authoritative references override summaries. Read each section once and retain its revision. Do not open every nested link, reload the full GDD, inspect all archived prompts, or repeat environment discovery.

Refresh after an input revision changes, a finding contradicts cached context, ownership changes, or context is lost. Use the base revision and actual diff to find changes. Check current files before editing shared data.

Use [a checkpoint](templates/checkpoint.md) at handoff, context reset, or meaningful pause only. Keep current decisions, files, evidence links, and next action. Update one active checkpoint; read completed histories only for a specific question.

Delivery states are `ready`, `in_progress`, `implemented`, `integrated`, `verified`, `blocked`, `needs_context`, `cancelled`. Preserve this state in the canonical record even if the client has different native statuses. Native completion may mean `implemented`, never accepted release.

1. Owner implements the whole task and runs selected local checks when ready.
2. Owner records one handoff: changed paths, result, check/result and evidence, unresolved issue/next action. Set `implemented`; notify lead once.
3. Lead inspects the scoped diff and prepares one metadata row for each native-only completed task in `tasks/completed_refs.jsonl`: `task_id`, `gdd_refs`, `changed_paths`, canonical handoff pointer. Commit this compact index with ready work, then record SHA and set `integrated` in the canonical task. The index contains no self-referencing commit SHA and duplicates no full task/handoff. Owner need not return to copy the result.
4. Selected checks are attributed to committed inputs; tester handles integration/milestone work. Lead records `verified` when the profile is satisfied. Only producer accepts a milestone.

One tester task may cover a feature batch and several implementation tasks. Retain tests for risks in validation.md §1, not one paired tester card per implementation card. Send useful failures to the owner with one reproduction; do not repeatedly route an unchanged failure.

Messages are triggered by completion, a blocker, changed shared semantics, or an artifact that unblocks an active dependency. No status pings, acknowledgement chains, speculative advice, or repeated context dumps. Let owners finish coherent steps. Interrupt only for cancellation, immediate destructive/conflicting work, or an invalidated dependency causing substantial waste.

Manual generation requests link a structured prompt, inputs, output destination, and `gdd_refs`. Missing media blocks dependent work only; placeholders can continue. Archive the exact submitted prompt once at generation (traceability.md §4).

## 4. Integration, commits, and verification

Lead remains the only Agent committer. Inspect task-owned diffs and stage explicit paths. Batch compatible finished tasks where useful; include one `Task: <ID>` trailer per task. A future helper may automate this, but its absence does not block this documentation template or authorized manual commits.

Do not commit another active owner's partial work. Integrate a finished dependency first or together with its finished consumer. Do not test a known-incomplete integration just to rediscover the dependency failure.

| Change | Default profile and trigger |
| --- | --- |
| Documentation only | `docs`: scoped consistency/diff check when ready |
| Isolated implementation, art, or tuning | `local`: relevant type/build, schema, or visual check when ready |
| Shared interface, high-risk behavior, combined feature | `integration`: affected tests and one smoke at integration |
| Milestone/release candidate | `milestone`: full gate on exact candidate SHA and complete demo evidence |

M0 has the explicit bootstrap exception in validation.md §3: accept its declared implemented checks, not a full-gate pass. Full candidate gates are required from M1 onward.

Choose by behavior/risk, not path alone: data changing timing or schemas may need integration checks. No full gate per tuning slider, screenshot, or commit. Evidence reuse follows validation.md; a historical candidate pass is never relabeled as a new passing SHA.

Full candidate runs use a separate gate worktree with isolated dependency state, build output, browser profile, and ports. Independent writers can continue. Serialize runs and supersede obsolete queued candidates. Completion hooks inspect records only; they never start the suite.

Use Conventional Commits and verify the intended branch. Pushing needs explicit authorization. No force pushes, destructive resets, secrets, or unrelated changes. Publication and milestone acceptance are separate decisions.

## 5. Bootstrap and milestones

This repository contains specifications/templates, not runtime automation. Command names describe interfaces to implement when needed; absence is not success. Do not build an orchestration platform before the first playable loop.

M0 supplies the app, minimal relevant contracts, run/build commands, one smoke/capture path, and needed tuning/save behavior. Keep commits manual until a helper saves repetition. Add a record-reading hook only once the client and gate exist. Prove a generation path before its output becomes a dependency.

| Milestone | Outcome |
| --- | --- |
| M0 | App/minimal checks run; ownership, environment, relevant contracts, and needed tuning round-trip are known |
| M1 | Core verbs playable with placeholders; relevant risk checks and representative motion/look targets exist |
| M2 | Complete start/play/end slice; storyboard selects important events and supplies their keyframes; complete reference video and runtime walkthrough cover the whole slice |
| M3 | Approved content and representative model/animation quality integrated; new important demo events receive keyframes; routine actions reuse the established shot plan |
| M4 | Agreed feel and measured performance criteria pass; blocking defects resolved; final demo path stable |
| M5 | Final candidate, complete captured gameplay video, reference-video package, report, provenance, portable exports, and remaining concerns delivered |

Before implementation needs a design decision, the relevant GDD states it clearly. Before M1, approved one-page GDD plus only necessary detail documents cover the core loop, success/failure, platform, input, view, time behavior, verbs, feedback, and provisional tolerances. Add detail when existing approved documents cannot describe it clearly; use the five GDD categories. A small change within an approved feature needs neither another design document nor repeated approval of unchanged rules.

Before a milestone, check only its required GDDs. Prototype reversible alternatives while a non-blocking decision is open; label them as proposals. Producer approves new scope, binding direction, and milestones, not every ordinary implementation step.

At milestone close, lead integrates ready work; tester checks the named candidate using M0 bootstrap acceptance or the full gate from M1 onward. Reviewer examines high-risk/behavioral changes; lead reviews tester/art-tool changes where relevant. Producer receives SHA, evidence, the milestone's demo scope, and concise playtest route. Repairs/tuning produce a new candidate: run affected checks while iterating and the required candidate checks once on the final acceptance candidate. Review only the changed delta unless a concrete finding expands scope. Producer accepts a named SHA; never move an existing milestone tag.

## 6. Documentation and cost feedback

Keep canonical ownership, verification, and asset rules in their documents. Task packets link them. Do not duplicate policies across role instructions or README; keep historical narrative outside automatic session entry points.

At each completed feature batch, record available counters once: fresh/cached input tokens separately, output tokens, model, tool calls, inter-agent messages, context reloads, check runs, wall time, and rework. Use `unknown` for unexposed counters. Compare similar tasks including retries. Reduced document size is not a measured token/cost reduction. Remove repeated steps that contribute no decision/evidence while preserving approvals and correctness boundaries.

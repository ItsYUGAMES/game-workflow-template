# Validation, Evidence, and Acceptance

Version: 0.7 | Date: 2026-11-10
The commands below are project interfaces to implement during bootstrap; this document does not claim that they already exist.

## 1. Retained tests and disposable probes

tester (GPT) writes and maintains every committed test, working from each task's acceptance criteria and GDD documents rather than from the implementer's own checks. Implementers never edit tests; tester never edits the source under test.

Keep only tests that protect behavior the producer would notice if it broke. Committed tests live only under `tests/regression/<area>/`:

| Folder | What to retain |
| --- | --- |
| `smoke/` | App boots to controllable gameplay; the current milestone's loop runs end to end |
| `mechanics/` | Rules and timing windows of the core mechanic as the GDD defines them (for example combo or cancel windows, puzzle rule resolution, card effect order); state transitions; contact or trigger timing |
| `editor/` | Preview, save, reload, stale-save conflict, hot reload, protected producer values |
| `data/` | Schema and cross-field validation for every editable data file |
| `replay/` | Deterministic input replay of an approved scenario |
| `lifecycle/` | Shared material and texture lifetime; rebuild without leaks |
| `lookdev/` | Effects trigger from events and never change gameplay state |
| `assets/` | Asset profiles, generator determinism, trace and prompt-record checks on fixtures |

- A retained test outside these categories needs lead approval and a one-line reason in its header.
- Each retained test file starts with a header comment stating its purpose and `gdd_refs`.
- Any role's own checks, one-off scripts, debug scenes, and investigation screenshots are probes. They go under the git-ignored `tmp/probes/<role>/<task_id>/` and are deleted before the task is marked complete.
- The gate's layout check fails when a test or spec file exists outside `tests/regression/`, or when anything under `tmp/` is tracked.
- Reviewer may propose tests in its report; the lead turns accepted proposals into tester tasks.

## 2. The gate and the completion hook

`npm run gate` runs these stages, all required: typecheck, lint, production build, data validation, asset validation, `gdd:check`, test layout check, retained regression tests, and Playwright smoke. A missing or skipped stage is a failure. The gate never calls generation services and never depends on temporary provider download URLs; tests use committed or cached outputs.

tester owns the gate runner and the completion-hook adapter under `tools/gates/`; systems wires the hook in `.claude/settings.json`. The gate runs only in the gate worktree, on a commit SHA (workflow.md §4). Each run writes a record to the git-ignored `.gate/runs/<sha>/<run_id>/`: SHA, command, Node/npm/browser versions, start and end time, result per stage, and log paths. A pass applies only to that SHA. Pin Node, npm, dependencies, browser binaries, fixtures, and seeds.

Claude Code blocks task completion only when the hook exits with code 2. Any other exit code, a crash, or a hook that fails to start is non-blocking, and the task completes anyway. The adapter therefore:

- Reads the task ID from the hook input and finds the lead's commit by its `Task: <task_id>` trailer. No such commit exits 2 with "not committed yet; ask the lead to commit".
- Exits 0 if a passing gate record already exists for that SHA. Otherwise it queues a gate run and waits up to its own limit; a pass exits 0, a failure exits 2 with the failing stage and log path on stderr. A test that exits 1 inside the gate must become exit 2 here.
- Exits 2 with "gate queued for <sha>; mark complete again when it finishes" if the run cannot finish within the limit. Set the hook `timeout` explicitly (command hooks default to 600 s) and keep the adapter's limit below it, so a timeout can never let a task through.
- Reports missing files in its own words. Claude Code treats exit 2 with empty stdout and a missing-file message on stderr (such as "No such file or directory") as non-blocking for TaskCompleted.
- Never marks tasks complete itself and never starts a second gate from inside a gate.

Rehearse four cases before relying on the hook: a passing task, a failing test, an uncommitted task, and the queued/timeout path. Check in the task list that the blocked tasks really stay incomplete; the adapter's exit code alone is not proof.

Codex roles run `npm run gate:sha -- <sha>` (or wait for the queued result) and record the outcome in their card and handoff.

## 3. Definition of done

A task is done when all of these hold:

1. The accepted scope is implemented in the owner's paths.
2. Probes are deleted.
3. New tunable fields have panel metadata and save correctly.
4. Evidence exists as §4 requires.
5. The handoff is complete, including `changed_paths` and `gdd_refs`.
6. The lead has committed the work, and the gate passes for that SHA.

Its paired test task, if any, is done separately by tester. A milestone candidate needs both.

Documentation-only or tooling-only tasks declare a `documentation` or `bootstrap` verification profile on the card before work starts; they never produce an integration pass. A check that could not be performed stays marked unverified.

## 4. Evidence

- Every player-visible change: a screenshot, or a short recording for motion, under `qa/evidence/<task_id>/<role>/<run_id>/`. Never overwrite earlier evidence.
- Only tasks marked `timing_or_performance: true` need the full evidence manifest: commit SHA, data hash, target revision, capture method, browser and version, viewport, device pixel ratio, seed, input sequence, simulation telemetry (events with tick and time), and source-timestamped frames or a contact sheet.
- Finish browser recordings before declaring evidence complete (Playwright saves video when the browser context closes). Keep the recorder's native container; if MP4 is required, transcode and keep the original.
- Exclude secrets and unrelated personal data from logs and captures.

## 5. Review

When reviewer runs:

- On every milestone candidate (`snap/mN-k`).
- On the delta of behavior-changing commits made after the last reviewed candidate, before the producer accepts.
- Not on data-only commits; those need a passing gate only.

Reviewer checks out the tagged SHA in the review worktree and starts from a fresh context with the GDD documents in scope, contracts, the previous reviewed SHA, target approvals, and evidence. Reviewer is read-only for tracked source, data, tests, and configuration; dependency caches and build output go to ignored or external locations, and temporary review probes stay outside the reviewed tree. Reviewer assesses the Claude-authored implementation, overall integration, and whether tester's tests cover the acceptance criteria. The lead reviews art_tools and tester changes in the same range.

Report header: `[SNAP <tag> @ <sha>]: APPROVE | CONCERNS | BLOCKED`.
Report fields: scope and baseline; acceptance results; findings with severity, file or evidence reference, and whether they block an acceptance criterion; tests and exit results; test coverage gaps; target deviations; performance results where in scope; limitations; recommended disposition.

`APPROVE` needs every in-scope required check to pass with no unresolved blocking finding. `CONCERNS` lists actionable defects or justified deviations. `BLOCKED` means evidence or environment was unavailable and must never be turned into approval.

The producer's acceptance names the final SHA, the review reports for it, any data-only tuning commits after review (each with a passing gate), and any deferred non-blocking findings.

## 6. Performance

Intended prototype budgets: at most 100 draw calls and 500,000 rendered triangles per frame, 20 MB of runtime assets, under 3 seconds from navigation to controllable gameplay, and 60 FPS on an agreed integrated-GPU device. tester owns the performance harness under `tools/perf/`.

These become acceptance tests only after the producer approves a short measurement profile: the device, browser and version, viewport and pixel ratio, quality settings with the editor off, the scene and input script, how long to capture, the frame-time statistic used for "60 FPS", what counts toward 20 MB, and cache and network conditions for the load time.

`renderer.info` counts rendering work, not load time, download size, or frame time; count all render passes. Use browser timing and network instrumentation for the rest, and the real device for hardware acceptance. Without an approved profile or the device, report performance as `UNVERIFIED`.

## 7. Startup proof

Before full rollout, take one task from assignment to accepted SHA: instruction loading and role/model checks; the deps subagent installing one pinned package; the commit helper refusing a path outside the owner's area; a small code and data change with live preview and a successful save; a stale-save conflict; a paired test written by tester; a deliberate hook failure that keeps the task incomplete; the lead's commit passing the gate; `gdd:impact` on a sample section; a prompt record that passes validation; a tagged snapshot; review; producer acceptance. This is a rehearsal to perform, not a claim that it has been run.

# Validation, Evidence, and Acceptance

Version: 0.8 | Date: 2026-11-13
This package is a documentation template, with no runtime or `package.json`. Commands below specify interfaces to implement during bootstrap; none is claimed to exist or have passed here.

## 1. Retained tests and disposable probes

tester (GPT) writes and maintains every committed test from acceptance criteria and approved GDD documents. Implementers never edit tests; tester never edits the source under test. Keep tests that protect player-visible behavior or important correctness boundaries, not tests that merely mirror implementation details.

Committed tests live under `tests/regression/<area>/`:

| Folder | What to retain |
| --- | --- |
| `smoke/` | App boots to controllable gameplay; the milestone's loop runs end to end |
| `mechanics/` | Core rules, timing windows, rule order, state transitions, contact or trigger timing; simulation authority independent of presentation |
| `editor/` | Preview, save, reload, stale-save conflict, hot reload, protected producer values |
| `data/` | Schema and cross-field validation for editable data |
| `replay/` | Deterministic input replay of an approved scenario |
| `lifecycle/` | Shared material and texture lifetime; rebuild without leaks |
| `lookdev/` | Effects trigger from events and never change gameplay state |
| `assets/` | Asset profiles, generator determinism, trace and prompt-record checks on fixtures |
| `agent/` | Runtime identity, real command/query/capture paths, test-storage isolation, and production exclusion |

- Each retained test file has a purpose and `gdd_refs` in its header. A test outside these categories needs lead approval and a one-line reason.
- Call tester for important new or changed behavior, a concrete regression risk, an integration batch, or a milestone candidate. Group related risk tests by feature batch; category membership does not create a paired tester card for every implementation task.
- Use existing tests when they cover the criteria. Add or change tests only for a meaningful uncovered risk; never delete useful tests or weaken assertions merely to pass or reduce run cost.
- Checks call the real modules, not a duplicate implementation of their formulas. A new or materially changed critical validator gets a representative invalid/valid fixture check once; do not corrupt project data or repeat this rehearsal per task. Infrastructure tests may use `gdd_refs: []` plus the relevant contract reference rather than inventing a design ID.
- One-off checks, debug scenes, and investigation captures belong in git-ignored `tmp/probes/<role>/<task_id>/`. Remove disposable probes before implementation completion; retain required evidence under §4.
- The gate checks that tests/specs are under `tests/regression/` and no files under `tmp/` are tracked. Reviewer proposes coverage gaps; the lead assigns accepted work to tester.

## 2. Verification profiles, reuse, and the gate

The task card names one profile. Choose it from the changed behavior and the acceptance being requested; a commit or worker completion alone does not trigger more verification.

| Profile | Required verification | When |
| --- | --- | --- |
| `docs` | Relevant document consistency and diff review | Once the edit is ready; no runtime gate |
| `local` | One relevant type/build check or visual/asset check, as the change requires | Once the task is ready |
| `integration` | Affected retained tests and one smoke run for the combined batch | After the batch is committed and ready for integration |
| `milestone` | Full gate, full gameplay demonstration, and risk-based review | On the named candidate SHA before producer acceptance |

M0 uses the explicit bootstrap-acceptance exception in §3. The full candidate gate is mandatory from M1 onward; M0 acceptance must never be labeled a full-gate pass.

Bootstrap uses the applicable profile for what actually exists and lists missing interfaces as `UNVERIFIED`. A missing command is not replaced with a no-op pass. Data-only tuning receives affected schema/cross-field and behavior checks at the appropriate batch; it does not require a full gate for each commit.

**Run only when needed.** Select checks from the changed inputs and concrete risks. Do not run suites after every edit, handoff, message, or commit. Rerun an affected check when its relevant inputs change, a failure needs verification after a fix, or new evidence invalidates the result. Report an unchanged failing command once with its cause and next action; repeating it without changed inputs or environment adds no evidence.

The [game runner and direct interface](agent_interface.md) expose check/simulate/capture capabilities without changing these triggers. Use only the relevant mode. Routine interface calls do not start a gate, require a screenshot, or create a new tester task. A semantic command check does not replace actual input/hit-testing acceptance where that is the behavior under test.

**Reuse scoped evidence honestly.** A prior result may satisfy the same affected check when its scoped source/data/fixtures, configuration, tool versions, and relevant environment are unchanged. Record the original result link, command and scope, originating SHA or input hashes, and why those inputs still match. If relevance or input identity is uncertain, run the affected check. Reused evidence is not a new run and never establishes a full gate pass for a different SHA.

**Full gate.** From M1, the planned `npm run gate` includes typecheck, lint, production build, data validation, asset validation, GDD-reference consistency, test layout, retained regression tests, and Playwright smoke. GDD consistency may use a documented manual review record until `gdd:check` automation is warranted; the gate records that method instead of claiming an unimplemented command passed. Declare applicable checks and any genuinely inapplicable asset/data category before the run. Missing evidence for an applicable stage cannot pass, and lack of a tool is not by itself a reason to call a required behavior inapplicable. The gate never calls generation services; use committed/cached outputs. Pin Node, npm, dependencies, browsers, fixtures, and seeds.

tester owns the gate runner under `tools/gates/`. Full gates run on committed candidate SHAs in the isolated gate worktree (workflow.md §4), one at a time. Each run records SHA, command, Node/npm/browser versions, start/end time, stage results, and logs under git-ignored `.gate/runs/<sha>/<run_id>/`. A full gate pass applies only to that exact SHA. A later commit needs its own full gate before it can be accepted as the final milestone candidate; scoped reuse must not relabel an earlier gate pass.

**Cheap completion hook.** tester owns the local adapter; systems wires it into `.claude/settings.json`. The adapter checks metadata and existing records only; it never starts, queues, waits for, or restarts a suite. Ordinary completion requires a handoff identifying performed checks and pending integration; it does not wait for a commit or batch checks. A candidate task (`milestone` profile) requires matching candidate records: the bootstrap record for M0, full gate from M1. Missing evidence yields one next action without launching work.

Rely on blocking only after the local adapter's exit-code-2 contract is verified in bootstrap. Hook absence, failure, or non-blocking completion never proves verification passed. Codex roles meet the same record requirements without the hook. Neither role runs a full gate merely to close an implementation task.

## 3. Implementation completion and acceptance

An owner may set `implemented` and finish the native task when the accepted scope is implemented in the assigned paths, disposable probes are removed, relevant new tuning has metadata and save support, and one handoff contains `changed_paths`, `gdd_refs`, local checks/evidence, and remaining limitations. A check that could not run stays `UNVERIFIED`; pending batch verification is identified explicitly.

The lead records the commit and sets `integrated`, then sets `verified` only when the selected profile's requirements are met. Implementation completion does not claim integration acceptance, completion of tester's work, or milestone approval. The lead resolves required tester work before accepting the batch or candidate. Blocking failures remain open for the acceptance they affect; the owner need not stay active waiting for another role.

Integration acceptance requires the batch's committed SHA, affected checks, and smoke result. From M1, milestone acceptance requires the final candidate's full gate, demonstration of the current playable scope, required tester work, review disposition, and the producer's named-SHA decision. Complete reference/runtime paths apply from M2 as defined in assets_and_targets.md. Documentation-only results never imply a gameplay pass.

**M0 bootstrap acceptance.** The producer may accept a named foundation SHA using a predeclared checklist of run/build checks, the required direct-Agent access proof in [agent_interface](agent_interface.md) §6, relevant current contract/data checks, and save/conflict checks if writeback is already part of the foundation. The access proof includes a real retrieved capture, isolated defaults, production exclusion, and reconnect identity; a port response alone is insufficient. Record results, reviewer disposition, and deferred stages as `UNVERIFIED`, with when they become required. Future asset pipelines, model simulation before a core game exists, trace automation, commit helpers, and hooks need not be complete merely to accept M0. The minimal direct interface is required; do not call this a full-gate pass. The same exact-SHA attribution and honest evidence rules apply.

## 4. Evidence

- Capture player-visible changes when the task or batch is ready: a screenshot for static changes, a short recording for motion. One capture can cover related tasks if it identifies their scope. Store evidence under `qa/evidence/<task_id>/<role>/<run_id>/`; never overwrite earlier evidence.
- A milestone demonstration shows the full current playable loop, including applicable success, failure, and restart paths. Attribute it to the exact candidate SHA and relevant scenario/data. Do not substitute a short isolated effect clip for the gameplay demonstration.
- Ordinary evidence needs its task/batch, actual source revision or input identity, check/capture scope, result, and link. Identify uncommitted captures as such; do not attribute them to a later commit without establishing matching inputs.
- Interface evidence also identifies the actual game instance, requested/executed action route, and relevant state/tick/frame. Preserve full raw runner output as linked files; concise handoffs do not replace the original record. The Agent must be able to retrieve required image evidence, not merely receive a path it cannot access.
- Only `timing_or_performance: true` tasks need the full manifest: commit SHA, data hash, target revision, capture method, browser/version, viewport, device pixel ratio, seed, input sequence, simulation telemetry with tick/time, and source-timestamped frames or a contact sheet.
- Finish recordings before reporting evidence complete. Playwright video is saved when the browser context closes. Keep the native container; if MP4 is required, retain the original alongside the transcode. Exclude secrets and unrelated personal data.

## 5. Review

Reviewer runs on every milestone candidate (`snap/mN-k`) and earlier only for a concrete risk or decision that needs independent review. Review the delta from the last reviewed SHA and the affected integration paths, expanding scope only when a finding warrants it. There is no separate reviewer pass for every task, ordinary commit, or data-only tuning edit.

Reviewer checks out the exact candidate in the review worktree and reads the task/candidate scope, relevant GDD and contract sections, previous findings, target approvals, and linked evidence. Reuse unchanged context and existing results; do not repeat the full gate as a review ritual. Reviewer is read-only for tracked source, data, tests, and configuration. Build output and caches go to ignored/external paths; review probes stay outside the reviewed tree. The lead reviews art_tools and tester changes in the same range.

Report header: `[SNAP <tag> @ <sha>]: APPROVE | CONCERNS | BLOCKED`.
Record scope/baseline, acceptance findings, severity and source/evidence references, whether each finding blocks acceptance, coverage gaps, applicable target/performance results, limitations, and disposition. Distinguish checks actually run from linked or reused results.

`APPROVE` requires the in-scope required evidence and no unresolved blocking finding. `CONCERNS` lists actionable defects or justified deviations. `BLOCKED` identifies unavailable evidence or environment and cannot stand in for approval.

Producer acceptance names the final SHA, its required verification (M0 bootstrap record or M1+ full gate), demonstration evidence, review reports, and deferred findings. Batch repairs before a replacement candidate, then run its applicable candidate checks and obtain a scoped review disposition. Earlier review is background, not approval of an unreviewed SHA.

## 6. Performance

Intended prototype budgets: at most 100 draw calls and 500,000 rendered triangles per frame, 20 MB of runtime assets, under 3 seconds from navigation to controllable gameplay, and 60 FPS on an agreed integrated-GPU device. tester owns the harness under `tools/perf/`.

These become acceptance tests only after the producer approves a measurement profile: device, browser/version, viewport/pixel ratio, quality settings with the editor off, scene/input script, capture duration, frame-time statistic, what counts toward asset size, and cache/network conditions for load time.

`renderer.info` counts rendering work, not load time, download size, or frame time; include all render passes. Use browser timing/network instrumentation for the rest and the real device for hardware acceptance. Without the approved profile or device, report performance as `UNVERIFIED`. Run performance checks only for relevant changes or the agreed milestone requirement.

## 7. Startup proof

During project bootstrap, rehearse one small batch through assignment, implementation completion, lead commit, and the checks that exist. Include the required M0 direct-access proof and editor save/conflict behavior before relying on those interfaces. When the commit helper, trace tools, and hook are actually introduced, verify their relevant ownership, prompt/impact, and missing-record cases once. Those optional tools do not block an otherwise authorized manual commit or a scoped document check; a missing direct game interface still blocks M0 acceptance. Rehearse the full candidate gate, complete gameplay demonstration, and review/acceptance path before using that path for milestone acceptance. Confirm actual hook blocking behavior before relying on it.

Perform this once when the relevant tools exist; repeat only the part whose contract changes or fails. This is a planned rehearsal, not evidence that the documentation template has executed any runtime validation.

@AGENTS.md

# Claude Code Adapter

Version: 0.8 | Date: 2026-11-13
Status: Proposed with the shared v0.8 package.

## Session setup

- Use the assigned role. The roster in docs/workflow.md §1 lists available roles, not a requirement to launch them all. Resolve ambiguous ownership with the lead before starting another writer.
- Verify actual client/model/effort/tool availability once at setup and after an environment change. Reuse the dated environment record. Do not repeat account, launcher, permission, or hook rehearsals per task. Never silently substitute a requested model.
- Load AGENTS.md through this import once. Give teammates a bounded task packet with relevant sources, not duplicate policy or the entire conversation.

## Tasks and communication

- Native Agent Teams tasks are canonical for Claude; Codex cards remain under `tasks/codex/`. Do not mirror either into a second full task list.
- Use workflow.md §3 fields/states. Keep one active implementation task per owner; reuse that owner for related follow-up while context remains useful.
- Native completion means implementation and selected local checks are finished. Integration and verification remain explicit in the record; completion is not milestone acceptance.
- Put one handoff in the canonical record, or link one file if needed. Send one `implemented` message. The lead adds SHA and the compact completed-reference index described in workflow.md §3; the worker need not wake to copy a report.
- Non-blocking ideas wait for the next task boundary. No status pings or routine interruptions. Report an actual dependency release once with its artifact/revision.

## Responsibilities

- Lead assigns scope, resolves scheduling, inspects task-owned diffs, and commits. Integrate related finished tasks together where practical. Retained-risk tests can share one tester task per feature batch.
- Implementers preserve ownership and tuning, and run their selected checks. They do not edit retained tests. Report a test/source disagreement with one reproduction.
- systems may install already-approved exact dependencies within assigned paths. Use optional `deps` only for substantial independent installation work; a one-command install does not need another agent.
- Route candidates to tester/reviewer at declared integration/milestone points, not every cosmetic or tuning commit.

## Completion hook

When implemented and verified on the installed client, the hook checks records only. It never starts builds, browsers, test suites, generation calls, or agents.

- Implementation task: require a handoff and selected verification record; integration may be pending.
- Candidate task with `verification_profile: milestone`: require a full-gate pass for that exact SHA from M1 onward. M0 instead requires its explicit bootstrap-acceptance record (validation.md §3), without claiming a full pass.
- Missing evidence produces one actionable result, not a repeated retry loop. Verify actual blocking/timeout behavior during setup; this document does not prove it.

## Recovery

After context loss, read the checkpoint, relevant changes since its base revision, and active ownership. Reload only invalidated sections. Confirm that an old writer stopped before transferring its paths. Do not restart healthy teammates because they appear idle.

@AGENTS.md

# Claude Code Adapter

Version: 0.5 | Date: 2026-11-10
Status: Becomes policy with the shared v0.5 package.

## Select the actual role

- This file applies to every Claude session that loads it. Do not assume that every session is the lead.
- A session explicitly assigned `lead` coordinates the team. A session assigned `gameplay`, `systems`, or `lookdev` follows that role's ownership and task scope. The `deps` subagent follows its own definition only.
- If the role is missing or contradictory, ask for the assignment before editing or spawning teammates.

## Roster and startup verification

| Role | Kind | Intended model | Intended effort |
| --- | --- | --- | --- |
| lead | Agent Teams lead | `claude-fable-5-2` | high |
| gameplay | Agent Teams teammate | `claude-opus-5-5` | high |
| systems | Agent Teams teammate | `claude-opus-5-5` | high |
| lookdev | Agent Teams teammate | `claude-opus-5-5` | high |
| deps | Subagent (`.claude/agents/deps.md`) | `claude-haiku-5-5` | default |

- Preserve these models; do not silently substitute another model, effort, provider, or permission mode.
- Verify the installed Claude Code and launcher versions, loaded instruction files, actual model and effort, display mode, task tools, and hook behavior. Record the result under `handoff/<role>/environment/`.
- Definitions under `.claude/agents/` must exist before their roles start. Settings in a definition can behave differently across versions and teammate modes; check the effective configuration after spawning.
- Name the role and model explicitly when spawning a teammate.
- Prompt instructions are not a filesystem sandbox. Verify available tools and write restrictions separately.
- Treat Orca as an optional launcher. Verify its working directory, launch flags, hibernation setting, and recovery behavior on the installed version.
- If the runtime cannot enforce a required restriction or provide a required model, report the concrete blocker and continue only independent work.

## Lead responsibilities

- The Agent Teams task list is the canonical record for Claude work. Create each task with the card fields from workflow.md §3, including `gdd_refs` and `timing_or_performance`.
- Write Codex work, including tester's work, as card files under `tasks/codex/`; read status from the cards. Do not assume Claude task state is visible to Codex.
- When a task's behavior falls in a retained-test category (validation.md §1), create its paired tester card at the same time, with the same `gdd_refs` and acceptance criteria. Turn tester's failure reports into fix tasks for the implementer.
- Dispatch only dependency-ready tasks of the current milestone. Start with one active implementation task per owner and keep a small ready backlog.
- Make every Git commit. When an owner reports a task ready, run `npm run commit:task -- <task_id>`, inspect the staged diff against the card, confirm, and report the SHA (Claude: `commit:` line in the task; Codex: card log). Commit deps changes the same way, and producer tuning or GDD edits when the producer asks.
- Maintain coordination records and draft reports. Do not edit application code, tests, package files, or producer-owned policy.
- Approve dependency requests by package and exact version, then have systems invoke the `deps` subagent. If teammates cannot start subagents in this runtime, invoke `deps` yourself.
- When the producer changes a GDD section, run `npm run gdd:impact`, report stale records by owner with a proposed action, and create rework tasks only for what the producer chooses.
- Check the handoff and commit of the actual dependency owner before dispatching dependent work.
- Review art_tools and tester changes at each candidate. Route behavior-changing commits to the reviewer; data-only commits need the gate only.
- Convert accepted findings into tasks. Resolve or explicitly defer blocking findings before requesting milestone acceptance.
- Present the candidate SHA, gate result, both review scopes, playtest steps for the producer, and remaining concerns. Wait for explicit acceptance before dispatching the next milestone.

## Teammate responsibilities

- Read the shared documents and your task. Work only in your assigned paths; do not revert another writer's changes.
- Do not write or edit committed tests; tester owns `tests/`. Put your own checks in `tmp/probes/<role>/<task_id>/` and delete them before completing. If tester's test and your implementation disagree, report it to the lead instead of changing either side to match.
- Provide editor metadata for new tunable fields. Register panels through the composition root; gameplay modules never import their colocated panels.
- Preserve producer tuning; ask for a field-level decision before changing a protected value.
- Propose contract changes through systems, using the breaking versus non-breaking rules in contracts.md.
- Write the handoff with `changed_paths` and `gdd_refs`, then message the lead that the task is ready for commit. Do not commit. After the lead reports the SHA, add it to the handoff and mark the task complete; the hook runs the gate on that commit.
- If the gate fails because of another owner's files, report it to the lead; do not edit their files or retry an unchanged failure.
- lookdev may generate textures through MCP or a manual request within its task; report each call so art_tools can archive the prompt and register the asset.

## deps subagent

- Installs only packages and versions the lead approved, with exact versions, and updates `package.json` and `package-lock.json` only.
- May also install pinned tool binaries the gate needs, such as Playwright browsers, when the task says so.
- Reports the installed versions, the changed files, and any audit warnings; the lead commits them. It never commits, and never edits source, configuration, or tests.

## Completion hook contract

- tester implements and owns the hook adapter described in validation.md §2; systems wires it in `.claude/settings.json`.
- The adapter finds the task's commit by the lead's `Task: <task_id>` trailer and exits 0 only when a passing gate record exists for that commit. A missing commit, a failure, or an unfinished run exits 2 with an explanation on stderr.
- Claude Code treats any exit code other than 2, a crash, or a hook that fails to start as non-blocking, so a plain `npm run gate` hook does not block completion.
- Set the hook timeout explicitly and keep the adapter's own limit below it.
- Rehearse a passing task, a failing test, an uncommitted task, and a queued run before relying on the hook, and confirm in the task list that blocked tasks stay incomplete.

## Recovery

- After a restart, check live teammates, the task list, active ownership, the current Git diff, and effective permissions before resuming.
- Reconcile stale claims with the lead; never start a second writer because an old session looks idle.
- Do not claim that a resumed conversation restored teammates, runtime state, or earlier gate results without checking.

# Design Traceability and Prompt Archive

Version: 0.5 | Date: 2026-11-10

This document defines two things. First, how a change to one GDD section finds every task, asset, target, data field, and retained test that depends on it, so only the affected work is redone. Second, how every generated asset keeps the exact prompt, model, and inputs that produced it.

## 1. GDD section IDs

The producer owns the GDD under `docs/gdd/`, split into Markdown files by area (for example `docs/gdd/mechanics.md`). `docs/gdd/index.md` lists the areas and their files. The template ships a genre-neutral skeleton; the producer may add, rename, or drop areas before the first section is referenced. Every section that can drive work starts like this:

```markdown
## GDD-MECH-02 Core rules
revision: 2
status: approved
needed_by: M1
```

- ID format: `GDD-<AREA>-<nn>`. The area is an uppercase word from the index; the number has two digits.
- IDs are permanent. Never renumber or reuse an ID. A removed section keeps its heading, gains `status: retired`, and names its replacement if there is one.
- `status` is `draft`, `approved`, or `retired`. Only the producer sets `approved`. A `draft` section may guide prototypes but is not approved design.
- `needed_by` (optional) names the milestone whose implementation needs the section approved (workflow.md §5).
- Raise `revision` by one whenever the meaning of the section changes. Typo and formatting fixes keep the revision.
- Section text may be Chinese. The ID, `revision`, `status`, and `needed_by` lines stay in English so tools can parse them.
- Agents may draft GDD changes in a handoff. Only the producer edits `docs/gdd/`.

## 2. Where references appear

A reference is `GDD-<AREA>-<nn>@r<revision>`, for example `GDD-MECH-02@r2`. It records the revision the work was based on.

| Record | Field | Written by |
| --- | --- | --- |
| Claude task (Agent Teams task list) | `gdd_refs:` line in the task description | lead |
| Codex task card `tasks/codex/<task_id>.md` | `gdd_refs` field | lead |
| Handoff `handoff/<role>/<task_id>/<run_id>.md` | `gdd_refs`, copied from the task | task owner |
| Asset manifest entry in `assets/manifest.json` | `gdd_refs` array | art_tools |
| Prompt record under `assets/prompts/` | `gdd_refs` | requesting role |
| Target package `targets/<target_id>/timing.json` | `gdd_refs` array | art_tools |
| Editable data field metadata | optional `gdd_refs` | data owner |
| Retained test under `tests/regression/` | `gdd_refs:` in the header comment | tester |

Work not driven by any GDD section, such as tooling or infrastructure, uses `gdd_refs: []` and says why in the task. A gameplay, content, or asset task without a reference is incomplete.

## 3. Impact analysis

art_tools implements two commands under `tools/trace/`:

- `npm run gdd:check` runs in the gate. It fails when a Codex card, handoff, manifest entry, target, or retained test names an unknown GDD ID, or when a gameplay, content, or asset record has no reference. A reference to an older revision or to a `draft` section is reported as a warning, not a failure. With `-- --milestone <Mn>` it also lists every section with `needed_by` at or before that milestone that is not yet `approved`; the lead runs this before dispatching a milestone's first implementation task.
- `npm run gdd:impact -- <GDD-ID>` scans every record type in section 2 and prints three groups: `current` (references the section's current revision), `stale` (references an older revision), and `orphan` (references a retired or unknown ID).

Open Claude tasks live in the Agent Teams task list, which these tools do not read. Finished Claude tasks are covered through their handoffs. While a Claude task is still open, the lead checks the task list for the affected ID by hand.

When the producer changes a GDD section:

1. The producer edits the section, raises its revision, and commits `docs(gdd): <summary>`.
2. The lead runs `npm run gdd:impact -- <GDD-ID>` and reports the stale records grouped by owner. Each gets a proposed action: rework, keep as is, or retire.
3. The producer chooses. The lead creates rework tasks only for the records chosen for rework. Unaffected modules are not regenerated or rewritten.
4. Reworked records carry the new revision when their tasks finish. Records the producer chose to keep are updated to the new revision by their owner, and the choice is recorded in `docs/decisions.md`.

## 4. Prompt archive

Every generation call gets a prompt record, including attempts that are later rejected. This covers reference images, reference videos, textures, 3D models, and any other generated asset.

Store each record at `assets/prompts/<resource_id>/<generation_id>.md`, with YAML front matter followed by the verbatim prompt. `generation_id` is `<yyyymmdd>-<hhmm>-<short_slug>` and is never reused. art_tools owns `assets/prompts/` and keeps it complete; a role that runs a call through MCP reports the details to art_tools in its handoff, or art_tools records them directly when it runs the call.

| Field | Meaning |
| --- | --- |
| `generation_id`, `resource_id` | Record identity and the asset it serves |
| `gdd_refs` | GDD sections this generation implements |
| `requested_by`, `operated_by` | Role that asked; role or `producer` that ran the call |
| `method` | `mcp`, `api`, or `manual` |
| `provider`, `model`, `model_version` | As reported by the tool or provider. Write `unknown` when it is not reported; never guess |
| `parameters` | Every other setting sent: size, aspect ratio, duration, quality, seed, mode, and so on |
| `inputs` | Reference files, each with repository path and SHA-256 |
| `provider_job_id` | Request or job ID, when the provider returns one |
| `generated_at` | ISO 8601 time with offset |
| `outputs` | Output files, each with path and SHA-256 |
| `status` | `pending`, `accepted`, `rejected`, or `superseded` |
| `status_by`, `status_at`, `status_reason` | Who changed the status, when, and why |

Rules:

- A manual request (`tasks/manual_requests/<request_id>.md`) becomes a prompt record when the producer fulfils it. art_tools copies the request into the archive and fills in model version, job ID, time, and output hashes from what the producer supplied.
- The manifest entry of a generated asset points to its accepted record in a `prompt_record` field. A generated asset without an accepted prompt record fails asset validation.
- Rejected outputs may be deleted; their records stay, with the rejection reason, so the next attempt does not repeat a failed prompt.
- When a generation edits or extends an earlier output, list that output among `inputs`. This keeps a chain from the final asset back to the first prompt.
- Never write API keys, tokens, or private account data into a record.

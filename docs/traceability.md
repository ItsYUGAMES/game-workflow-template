# Design Traceability and Prompt Archive

Version: 0.8 | Date: 2026-11-13

Read sections 1–3 only when design references change, and section 4 when generating or registering media. This document links affected work to design revisions and preserves exact generation provenance. It is not a mandatory reread for every role or task.

## 1. GDD documents and IDs

The producer owns the GDD under `docs/gdd/`. It is written for people first, following the writing guide in `docs/templates/gdd_guide.md`:

- `docs/gdd/one_page.md`: the one-page GDD (一页策划). It is the kickoff document: it describes the whole game on one page (at most 700 Chinese characters of body text plus one key image). After M1 it stays current as the summary and index of the detail documents.
- Detail documents (细则文档) use the five existing folders/templates: `systems/`, `ui/`, `levels/`, `content/`, and `art/`. Add one when existing approved documents cannot describe the feature clearly. Several related tasks may share one document; routine changes within approved design need no new document or reapproval of unchanged rules.
- `docs/gdd/images/<name>/`: diagrams, mockups, and reference images used by those documents.

Each document starts with a front-matter block. It is the only machine-readable part; headings and body follow normal Chinese document conventions.

```markdown
---
id: GDD-RELIGHT
revision: 2
status: approved
needed_by: M1
---
```

- `id` is `GDD-` plus the file name in uppercase (`one_page.md` → `GDD-ONE_PAGE`, `systems/relight.md` → `GDD-RELIGHT`). File names must be unique across `docs/gdd/`. IDs are permanent: once a document is referenced, never rename its file or reuse its ID. A removed document keeps its file with `status: retired` and names its replacement, if any.
- `status` is `draft`, `approved`, or `retired`. Only the producer sets `approved`. A `draft` document may guide prototypes but is not approved design.
- `needed_by` (optional) names the milestone whose implementation needs the document approved (workflow.md §5).
- Raise `revision` by one whenever the meaning of the document changes, and add a matching row to the document's change log (修改记录) that names the parts that changed. Typo and formatting fixes keep the revision.
- Chapter numbers (一、, 2.2, list items) may shift as a document is edited, so they are not trace keys. Task acceptance text may cite them for readers, for example `GDD-RELIGHT §2.2(3)`.
- The one-page GDD's detail list (细则与待定) links every detail document and shows its status and `needed_by`; the front matter is authoritative.
- Agents may prepare scoped GDD drafts when requested. Only the producer approves design meaning; a draft edit does not imply approval.
- Only Markdown files under `docs/gdd/` define IDs. `docs/templates/` and `docs/examples/` are ignored by the trace tools; never cite their IDs in `gdd_refs`.

## 2. Where references appear

A reference is `GDD-<NAME>@r<revision>`, for example `GDD-RELIGHT@r2`. It records the revision the work was based on.

| Record | Field | Written by |
| --- | --- | --- |
| Claude task (Agent Teams task list) | `gdd_refs:` line in the task description | lead |
| Codex task card `tasks/codex/<task_id>.md` | `gdd_refs` field | lead |
| Handoff in the canonical task, or one linked file under `handoff/<role>/` | Reuse the task's `gdd_refs`; no duplicate full handoff required | task owner |
| `tasks/completed_refs.jsonl` | Minimal index: `task_id`, `gdd_refs`, `changed_paths`, canonical handoff pointer; SHA remains in the canonical task | lead, at integration |
| Asset manifest entry in `assets/manifest.json` | `gdd_refs` array | art_tools |
| Prompt record under `assets/prompts/` | `gdd_refs` | requesting role |
| Target package timing metadata and `targets/<demo_id>/storyboard.md` | `gdd_refs`; important events map selected keyframes; events/action spans map both video timecodes | art_tools |
| Editable data field metadata | optional `gdd_refs` | data owner |
| Retained test under `tests/regression/` | `gdd_refs:` in the header comment | tester |

Work not driven by any GDD document, such as tooling or infrastructure, uses `gdd_refs: []` and says why in the task. A gameplay, content, or asset task without a reference is incomplete.

## 3. Impact analysis

When scale warrants automation, art_tools implements the following interfaces under `tools/trace/`. Until then, a scoped manual reference check is valid and is recorded honestly; this template does not contain those commands. Run changed-record checks at integration and a complete current-candidate check at milestone acceptance, not on every task/hook. Historical records may retain the revisions actually used.

- `npm run gdd:check` checks the selected scope for unknown/missing GDD references, malformed front matter, duplicate IDs, and disagreement between the one-page index and current detail documents. It warns about the one-page size and historical revisions. With `-- --milestone <Mn>` it identifies required designs that are not approved. The lead checks those before dependent implementation; the milestone candidate must not rely on unapproved design as if it were approved.
- `npm run gdd:impact -- <GDD-ID>` scans every record type in section 2 and prints four groups: `current` (references the document's current revision), `stale` (references an older revision), `kept` (references an older revision that a compatibility decision accepts for the current one), and `orphan` (references a retired or unknown ID).

These tools do not read the native Agent Teams task list. For a changed GDD ID, the lead searches relevant open tasks directly. Finished tasks are covered by their file handoff or the compact `tasks/completed_refs.jsonl` index; follow the canonical handoff pointer only when its details matter. The index is emitted during normal integration, without a second full task list, duplicated handoff, or extra agent call. If an older completed task has neither readable handoff nor index row, inspect the relevant completed native history once and record its reference; do not silently treat it as unaffected.

When the producer changes a GDD document:

1. The producer edits the document, raises its revision, adds a change-log row naming the changed parts, and commits `docs(gdd): <summary>`.
2. The lead checks affected references once, using the tool when available or a targeted search. Report only meaningful effects, grouped by owner. Work using unchanged sections normally remains valid.
3. Within the producer-approved change, the lead schedules only affected rework. Ask the producer only when scope, design, binding targets, or acceptance are still unresolved; do not request another decision for every unaffected record.
4. New/reworked records carry the revision used. Keep historical prompt/evidence records unchanged. For a current asset intentionally kept, record one compatibility decision in `docs/decisions.md` naming the record and both revisions, so later impact checks report it as `kept`; do not rewrite every historical record merely to make revision numbers match.

## 4. Prompt archive

Every generation call gets a prompt record, including attempts that are later rejected. This covers reference images, reference videos, textures, 3D models, and any other generated asset.

Store each record at `assets/prompts/<resource_id>/<generation_id>.md`, with YAML front matter and the exact submitted prompt. `generation_id` is unique and never reused. art_tools maintains the archive/index. A generating role may receive explicit ownership of its new record paths and write them directly; otherwise it includes the call details in its single batch handoff. Do not start another agent or send a message for each generation attempt.

Compose prompts from the approved visual baseline, an asset/action specification, and shot-specific differences using [the art prompt template](templates/art_prompt.md). Task context keeps component references/revisions; the generation record stores the fully resolved prompt actually sent, including negative instructions, actual parameters, and input hashes. A hash/link without the submitted text cannot reproduce the request. Archive once without echoing the full prompt through every agent conversation. Load only the relevant accepted record or failed attempt when revising an asset.

For reference videos, link the storyboard revision and relevant `event_id` values. Retain keyframe mappings for selected important events (`keyframe_required: true`) and the reason they matter. Routine spans may have no images; complete-path coverage is checked separately from important-event image coverage (assets_and_targets.md §6).

| Field | Meaning |
| --- | --- |
| `generation_id`, `resource_id` | Record identity and the asset it serves |
| `gdd_refs` | GDD documents this generation implements |
| `prompt_components`, `storyboard_ref`, `event_ids` | Applicable source component revisions and demo events; omit fields unrelated to the output |
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
- Prompt text, inputs, parameters, and original outputs are immutable per attempt. Status metadata may be updated with who/when/why; a revised prompt is a new attempt. Record unavailable provider metadata as `unknown` instead of asking the producer to reconstruct it repeatedly.
- Never write API keys, tokens, or private account data into a record.

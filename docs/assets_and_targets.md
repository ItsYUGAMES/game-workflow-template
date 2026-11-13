# Assets and Reference Targets

Version: 0.8 | Date: 2026-11-13

## 1. Asset pipeline and provenance

Choose the production route from the required appearance and motion, not a fixed source hierarchy:

| Need | Suitable route and acceptance check |
| --- | --- |
| Blockouts, simple geometry, repeated modular environment pieces | Procedural geometry with explicit shape, dimensions, and palette; instance compatible repeated geometry/materials and measure the result |
| Characters or enemies whose identity, silhouette, and deformation matter | Reusable rigged assets with suitable usage rights, or generated models paired with a verified rigging/animation route; check appearance and deformation in the intended game camera before expanding the set |
| Distinctive props or complex static forms | Generated, reusable, or manually authored assets according to silhouette, material, budget, and editing needs |

A procedural blockout is not an accepted finished character unless it meets the approved art direction. Use Blender for required cleanup, rig/clip correction, or export work; do not add a Blender pass to an already compliant asset. Verify the actual tool's image, mesh, rig, animation, and video inputs/outputs before making a task depend on them. A model name does not establish these capabilities.

External generation uses one of three paths: an MCP tool called directly by the role, a reviewed script under `tools/`, or a manual request that the producer runs. Use the provider and model the task names; do not switch silently. Every call gets a prompt record ([traceability](traceability.md) §4), including rejected attempts. Download completed outputs promptly when the provider reports expiry, keep the request ID, and never expose credentials in logs, records, or game source.

Asset IDs use prefixes `chr`, `enm`, `prp`, `env`, `wpn`, `fx`, `ui`, `sfx`, `mus`, and `tgt`. Keep IDs stable. Register every managed asset in `assets/manifest.json` with at least:

- `id`, `kind`, `source` (`procedural`, `generated`, `pack`, or `manual`), and file or generator path;
- approved asset revision (or a distinct versioned file), generator parameter/seed reference where applicable, and schema version; content digest only for explicit L3 integrity needs;
- measured triangles, texture dimensions/formats, required clips, socket profile, and pivot/orientation profile, where applicable;
- `gdd_refs` for the GDD documents the asset serves;
- for generated assets, `prompt_record` pointing to the accepted record in `assets/prompts/`; for packs and manual work, source URL, acquisition date, and license or usage evidence;
- source-file path, such as the `.blend`;
- replacement priority and reference links once the M5 replacement list is prepared.

Use explicit `not_applicable` for metadata that does not apply, not fabricated values. Record generated and external sources in `assets/CREDITS.md`. Unknown usage rights block distribution of the asset; use an approved placeholder if independent development can continue.

Keep editable art source and a repeatable import/export path under the appropriate `tools/` owner when assets need conversion. Automate repeated manual export steps; do not assume the runtime can read every source format. Packaging and demo trimming operate on output copies, preserving source content.

### Art direction and structured prompts

Keep the shared art bible in the approved GDD art document, using [the art/audio template](templates/gdd_art_audio.md). It defines the style, camera/framing, world scale, palette, lighting, shape language, materials, and persistent character identity. Reference its ID/revision and approved images from tasks instead of copying the whole bible into every task. A changed costume, weapon, proportion, palette, or camera is an explicit revision or task-specific exception, not an accidental generation variation.

Use [the art prompt template](templates/art_prompt.md) for images, models, animation, and video. Each prompt specification supplies only applicable fields:

| Block | Required content |
| --- | --- |
| Purpose and references | Asset/demo/shot IDs, `gdd_refs`, art-bible revision, approved identity/scene references, and the specific change requested |
| Appearance and composition | Camera/view, projection or lens/FOV, framing/aspect, scale, palette, lighting, shape/material treatment, and identity details to preserve |
| Technical delivery | Asset kind, format, dimensions/pivot/axes, budgets, rig/sockets, required clips, transparency/looping, and other relevant profile constraints |
| Motion | Key poses, phases, contacts, direction/speed, loop/transition behavior, and root-motion treatment; distinguish desired visual timing from approved simulation timing |
| Video shot | Duration, start/end state, important `event_id` values, ordinary action spans, visible responses, and selected keyframe references |

Resolve references into the exact instructions and supported input fields sent to the provider; a repository path alone does not give a provider access to an image or bible. Store that exact final prompt, actual parameters, input paths/revisions, and output paths/revisions in the append-only prompt archive ([traceability](traceability.md) §4), including rejected attempts. The prompt specification is reusable planning material; it is not evidence that a generation call occurred.

## 2. Interfaces by asset kind

Do not require every prop, wall, icon, sound, or VFX texture to have a humanoid skeleton and eight animation clips.

| Kind | Required profile |
| --- | --- |
| Humanoid character | Resource-ID PascalCase root; foot-center pivot; +Z model front; scale 1; required animation mapping; approved sockets |
| Static prop/environment | Resource-ID PascalCase root; declared placement pivot and dimensions; scale 1; sockets or clips only when its gameplay contract requires them |
| Weapon/interactive prop | Declared grip/attachment pivot, attachment contract, and only relevant animation/socket requirements |
| VFX/texture/UI/audio | Kind-specific dimensions, format, duration or bounds, and usage metadata; no humanoid checks |

The default humanoid height is 1.8 m; deliberate exceptions need a recorded profile. Socket names are `SocketHandR`, `SocketHandL`, `SocketHead`, and `SocketBack` where required. Attach a skinned character's sockets to the corresponding bones.

Character state names are snake_case and declared in each character's asset profile, following the GDD (for example `idle`, `run`, `attack_1`, `hit`, `death` in an action game). Required states are specified per character; a deliberately static dummy does not silently acquire all states. GLB clips use the required state names or an explicitly approved mapping. `AnimationDriver` unifies procedural and clip playback.

For each moving character/action, specify the silhouette and key poses, phase order, planted feet or hand/object contacts, facing/travel direction, loop boundaries, transitions, and intended root displacement. For example, an attack needs readable preparation, contact, and recovery; locomotion needs plausible foot contact and travel speed. Review clips on the actual model, with its props, from the game camera. Check sliding, penetration, joint deformation, identity drift, loop seams, and transitions; a valid clip name or a budget pass alone is not visual acceptance.

Gameplay timing, such as when an action makes contact or takes effect, remains in `data/tuning.json`. Rendering may retime a clip to the approved action; animation callbacks cannot change gameplay state. Root motion is disabled for gameplay authority unless the producer approves a new movement contract.

Retain the prototype budgets of at most 5,000 triangles per character and 1,000 per prop. Declare environment and texture budgets in the performance profile before accepting relevant assets. A triangle budget alone does not establish rendering cost or download size.

## 3. Procedural assets and resource lifetime

Use `src/visual/generators/<resource_id>.ts`, exporting a named factory such as `createChrHero(params, rng)`. Parameters live in `data/visual/<resource_id>.json`, with editor metadata and a rebuild policy. Register previews in `src/visual/registry.ts` using `/?preview=<resource_id>&seed=<n>`.

The contract is deterministic semantic output for identical input and generator version. Factories must not mutate their input parameters or shared palette resources. Compare normalized geometry buffers, transforms, material IDs, sockets, and animation metadata; exclude generated UUIDs and volatile serialization metadata. Document float tolerances. Do not require incidental object identity or serialized UUIDs to match.

Shared materials come from `src/visual/palette.ts`, with the producer-approved colors. A generator borrows shared palette resources; it does not own their destruction. Dispose owned geometry and private resources when replacing an instance. Release shared materials/textures only through their lifecycle owner after the final user releases them. Coalesce slider-driven rebuilds and cancel obsolete asynchronous results.

## 4. Generated models and Blender

art_tools alone operates Blender MCP when a Blender pass is needed. Identify the actual server/version and permissions; save the `.blend` before running Blender Python. Use project-scoped file access where supported, but do not describe an unverified server option as a sandbox.

Promote approved generated/reusable output or `assets/inbox/<request_id>/` into `assets/models/<resource_id>.glb`, retaining source files and any edited `assets/source/<resource_id>.blend`. Where necessary, rename nodes/sockets, set the intended pivot, apply transforms, meet budgets, align clip names, and match the palette. Prove the required skeleton/retargeting mapping and clips on one representative character before producing a roster. Use a requested rig naming convention only when the tool actually supports it; report missing rigging, skinning, or animation rather than inventing a successful mapping.

For the selected Blender exporter, verify the convention once using an asymmetric front-marker fixture; repeat only if the exporter/configuration changes. The intended mapping is Blender -Y model front to exported +Z model front with +Y up. Inspect each asset's exported orientation, scale, pivot, and sockets as its batch becomes ready. Save representative front, side, and top views as evidence.

Keep canonical GLB interchange assets uncompressed. Web-only compression may produce separate build artifacts after compatibility and quality checks; it must not replace the source GLB. Verify the pinned Godot importer and required glTF extensions during port planning rather than assuming permanent engine support or non-support.

`tools/validate_assets.ts` checks profiles by asset kind, required files, provenance references, budgets, animation mapping, sockets, and generator determinism. An ordinary asset without an optional content digest must not fail validation; require one only for an explicitly declared L3 exact-integrity check. Some properties need explicit metadata or a visual fixture; do not pretend every pivot/orientation semantic can be inferred from a file alone.

## 5. Lighting, shaders, and VFX

lookdev owns lighting, shaders, post-processing, and VFX code/data. Store lighting/post parameters in `data/visual/lighting.json` and effect parameters in `data/visual/vfx/<resource_id>.json`. Register their editor panels.

Effects react to discrete events and read-only state; they do not influence gameplay authority. gameplay owns the intent of gameplay-driven feedback such as hitstop and camera shake. Shader files describe their purpose and parameters in English. Measure before/after performance across all render passes for batches that add material rendering cost or show a regression, and at the agreed performance milestone. Do not repeat the performance suite for every cosmetic parameter adjustment.

lookdev may generate textures within its approved task through MCP or a manual request, and may write explicitly assigned new prompt-record paths. art_tools maintains the archive/index, promotion, manifest, and credits. Register the finished resource batch once before declaring it integrated; do not add a coordination message per generation call.

## 6. Complete demos, important-event keyframes, and target measurement

### Plan the complete path

Use [the demo storyboard template](templates/demo_storyboard.md) for `targets/<demo_id>/storyboard.md`. Derive a start-to-end path from the approved GDD: initial screen/state, setup, core play, required transitions and feedback, and the declared ending. Include any failure, retry, pause, or menu action required by that path. Name branches outside the path explicitly; a montage of attractive moments does not establish complete coverage. The plan identifies camera, initial data/seed, inputs, expected state changes, and the endpoint that makes the demo complete.

Create two separate deliverables covering this same path:

- **Generated reference video:** the intended appearance, action readability, and sequence, guided by the approved important-event images. It may be assembled from shorter generated clips.
- **Captured gameplay demo:** the implemented path played in the actual build, with its commit/data revision and observable input/response evidence. Prefer a continuous run; disclose edits and retain original captures with their event/timecode mapping.

Generated reference video is not proof that the game works, a trustworthy physics trace, a source of simulation truth, or permission to add hidden behavior. Reference completion and runtime completion are reported separately. Do not substitute generated footage for captured gameplay or mark a missing artifact complete.

### Select keyframes at important events

Keyframes are event-driven, not per input, per interaction occurrence, or at a fixed time interval. Select an event when a picture is needed to establish an important visual or gameplay outcome. Typical triggers are the first representative use of a core mechanic; a materially different interaction result; a major phase, form, scene, or camera transition; victory/failure feedback; or a difficult pose/contact/impact that needs a clear visual target. Selection follows the demo's purpose and acceptance criteria, not merely whether some state variable changed.

Routine locomotion, repeated actions with already-established feedback, ordinary menu navigation, and unchanged UI operations normally need no individual keyframe. Group them into an action span or input script. A repeated action gets another keyframe only if its outcome, context, composition, or visual requirement changes enough to need a new target. Existing routine actions remain in the complete video; skipping their images does not remove their gameplay.

`event_id` identifies a semantic event or action span in the demo; `interaction_id` identifies its action type where applicable. Mark selected events with `keyframe_required: true` and a short `importance_reason`. Other rows use `keyframe_required: false` and `keyframes: []`; that is intentional absence, not missing/pending evidence. Do not create a row for every keystroke just to document the absence of an image. Important events are explicit rows; routine actions can be summarized by span.

Before generating a shot containing a selected event, provide at least one actual image for that event. Add preparation/contact/end images only where they are necessary to establish its motion or causality. Keyframe inputs are distinct from frames sampled later from the output video.

| Record | Required mapping |
| --- | --- |
| Event or action span | `event_id`, trigger/input or script, precondition, visible response and resulting state |
| Keyframe selection | `keyframe_required`; `importance_reason` for selected events |
| Selected important event | Image path, approved revision and phase; any explicit reuse of an existing suitable image |
| Reference video | Shot ID and final time range for the event/span; response time for important events |
| Runtime verification | Actual capture/time range, expected/observed result, available event/input log or inspection evidence, and `pass`, `fail`, or `unverified` |

Check two different things: the video covers the declared start-to-end path, and every selected important event has its required image and observable reference/runtime outcome. The image-coverage denominator is the number of selected events, not the number of all inputs. Selection changes update the storyboard with a reason; do not quietly demote a failed important event to manufacture a pass. Suitable images may be reused when identity, camera, pose, and state still match. Inspect generated footage rather than assuming it followed an input image. Runtime and reference timecodes are measured separately; precise timing claims also use simulation telemetry.

### Generate shots within verified tool capabilities

Each shot specifies duration, start state, important event IDs and their selected keyframes, routine action spans, input/response beats, end state, and continuity constraints. Keep character identity, scale, props, camera, palette, lighting, and location consistent unless the storyboard changes them. A tool-required shot-boundary image is a technical continuity input; reuse an existing suitable image/frame where possible and do not turn every routine action into an important event to obtain it.

If the tool accepts only start/end frames, split a shot at selected important poses only when needed to preserve those targets. Reuse boundary images where supported and check continuity at joins. Use a documented supported/manual route or report the dependent shot blocked if required conditioning is unavailable. Do not claim arbitrary multi-keyframe support or drop selected important targets to fit a tool. Keep source clips and edit-to-source timecode mappings.

### Target packages and timing

art_tools prepares `targets/<target_id>/`: storyboard and selected important-event keyframes, source/reference clips, sampled output frames, numbered contact sheets, and `timing.json` when timing is being measured. Runtime captures remain under [validation](validation.md) evidence paths and are linked from the storyboard. The producer approves `binding` and `non_binding`; other roles retain the approved revision.

M1 may use placeholder visuals. Before accepting the M2 representative slice, approve and evaluate its visual baseline: recognizable silhouette/identity, composition/readability, material/light treatment, and action poses/contacts/transitions. Use these concrete acceptance criteria through M3–M5. Pixel-exact resemblance to every generated frame is non-binding unless explicitly selected, but required visual quality is no longer waived for the whole Three.js phase. Coverage of selected important-event keyframes and both complete video deliverables remains mandatory at the stated milestones.

Define these fields before timing-dependent implementation:

| Field | Required meaning |
| --- | --- |
| `schema_version`, `target_id`, `source_ref` | Stable schema, package identity, and source clip path plus revision; optional `source_sha256` only when exact-content verification is needed |
| `gdd_refs`, `prompt_record` | GDD documents the target illustrates; prompt record of the generated clip |
| `source_timebase`, `source_fps`, `is_variable_frame_rate` | Original timing, distinct from sampling rate |
| `sample_fps`, `frame_index_base`, `clip_start_s` | Sampling convention; use zero-based indices |
| `frames` | Mapping from each sampled index/file to source presentation timestamp |
| `markers` | Named start/contact/end markers with measured time, uncertainty, and method |
| `binding` | Metric, expected value/range, units, tolerance, measurement method, and producer approval reference |
| `non_binding` | Explicit exclusions and their reason |
| `approval_revision` | Producer-approved target revision, linked to the source reference and version |

For a constant-rate zero-based sample sequence, `t_s = clip_start_s + frame_index / sample_fps`. For ordinary reference and timing packages, stable file paths, revisions, timestamps, and event mappings are sufficient; do not block on a missing source hash. When byte-exact media identity is required (for example frozen M5 deliverables), calculate its digest automatically at packaging/acceptance and record the exact file it covers. Prefer actual source timestamps for variable-rate input. Never confuse a source frame index with a sampled frame index. Twelve samples per second are useful for contact sheets, but adjacent samples are about 83 ms apart; do not infer millisecond hitstop or contact timing from them. Use source-rate inspection and game telemetry for tighter tolerances, and report uncertainty.

Target timings initialize tuning only before producer approval of those fields. Subsequent playtest decisions take precedence and must record the intentional deviation or update the target revision. Reference comparison cannot silently undo approved tuning.

Review comparisons identify the same action start, camera/view settings, input script, data revision, and timebase. Preserve original recordings and logs; use contact sheets for inspection, not as the sole test of gameplay timing.

## 7. Godot handoff

`npm run export:blockouts` exports procedural reference geometry to `exports/blockouts/<resource_id>.glb` with dimensions, pivots, and sockets. Include canonical GLB and retained source files for generated/reusable assets, including Blender sources when used. Record what procedural animation or shader behavior was not exported.

The M5 package includes final data schemas and values, assets with their prompt records, the GDD revision each part implements, replacement priorities, required clips/sockets, target approvals, the complete reference video and captured gameplay demo with their event/keyframe/timecode verification map, original recordings, and unresolved concerns. The port must explicitly map camera/model axes, FOV and aspect policy, physics, input, simulation timing, animation, events, and shader behavior. Similar coordinate handedness does not eliminate those decisions.

The Godot-phase rules are written separately, but these producer requirements carry into them:

- GDScript identifiers follow the Godot style guide in English. Doc comments (`##`) and inline comments are written in Chinese and explain what each script, signal, exported property, and non-obvious block does and why.
- Every ported system gets an illustrated Chinese explainer at `docs/godot/<system>.md`: the scene tree, signal flow, state machine where one exists, how data flows from the JSON files, and which Three.js module it replaces. Use Mermaid diagrams and Godot editor screenshots.
- A task that changes a system's behavior updates its explainer in the same task. An outdated explainer fails the definition of done in the Godot phase.

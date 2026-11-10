# Assets and Reference Targets

Version: 0.5 | Date: 2026-11-10

## 1. Asset pipeline and provenance

Prefer procedural generation, then generated models processed in Blender, then existing asset packs. Use generated 3D models for approved characters, enemies, and key interactive objects. Use basic geometry or procedural assets for repeated environment pieces; instance compatible repeated geometry/materials and measure the result.

External generation uses one of three paths: an MCP tool called directly by the role, a reviewed script under `tools/`, or a manual request that the producer runs. Use the provider and model the task names; do not switch silently. Every call gets a prompt record ([traceability](traceability.md) §4), including rejected attempts. Download completed outputs promptly when the provider reports expiry, keep the request ID, and never expose credentials in logs, records, or game source.

Asset IDs use prefixes `chr`, `enm`, `prp`, `env`, `wpn`, `fx`, `ui`, `sfx`, `mus`, and `tgt`. Keep IDs stable. Register every managed asset in `assets/manifest.json` with at least:

- `id`, `kind`, `source` (`procedural`, `generated`, `pack`, or `manual`), and file or generator path;
- content revision/hash, generator parameter/seed reference where applicable, and schema version;
- measured triangles, texture dimensions/formats, required clips, socket profile, and pivot/orientation profile, where applicable;
- `gdd_refs` for the GDD sections the asset serves;
- for generated assets, `prompt_record` pointing to the accepted record in `assets/prompts/`; for packs and manual work, source URL, acquisition date, and license or usage evidence;
- source-file path, such as the `.blend`;
- replacement priority and reference links once the M5 replacement list is prepared.

Use explicit `not_applicable` for metadata that does not apply, not fabricated values. Record generated and external sources in `assets/CREDITS.md`. Unknown usage rights block distribution of the asset; use an approved placeholder if independent development can continue.

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

Gameplay timing, such as when an action makes contact or takes effect, remains in `data/tuning.json`. Rendering may retime a clip to the approved action; animation callbacks cannot change gameplay state. Root motion is disabled for gameplay authority unless the producer approves a new movement contract.

Retain the prototype budgets of at most 5,000 triangles per character and 1,000 per prop. Declare environment and texture budgets in the performance profile before accepting relevant assets. A triangle budget alone does not establish rendering cost or download size.

## 3. Procedural assets and resource lifetime

Use `src/visual/generators/<resource_id>.ts`, exporting a named factory such as `createChrHero(params, rng)`. Parameters live in `data/visual/<resource_id>.json`, with editor metadata and a rebuild policy. Register previews in `src/visual/registry.ts` using `/?preview=<resource_id>&seed=<n>`.

The contract is deterministic semantic output for identical input and generator version. Factories must not mutate their input parameters or shared palette resources. Compare normalized geometry buffers, transforms, material IDs, sockets, and animation metadata; exclude generated UUIDs and volatile serialization metadata. Document float tolerances. Do not require incidental object identity or serialized UUIDs to match.

Shared materials come from `src/visual/palette.ts`, with the producer-approved colors. A generator borrows shared palette resources; it does not own their destruction. Dispose owned geometry and private resources when replacing an instance. Release shared materials/textures only through their lifecycle owner after the final user releases them. Coalesce slider-driven rebuilds and cancel obsolete asynchronous results.

## 4. Generated models and Blender

art_tools alone operates Blender MCP. Identify the actual server/version and permissions; save the `.blend` before running Blender Python. Use project-scoped file access where supported, but do not describe an unverified server option as a sandbox.

Process approved generated output or `assets/inbox/<request_id>/` into `assets/source/<resource_id>.blend` and `assets/models/<resource_id>.glb`. Rename nodes/sockets, set the intended pivot, apply transforms, meet budgets, align clip names, and match the palette. Use the requested Mixamo-compatible naming for humanoid rigging when the provider supports it; otherwise report the mismatch before inventing a rig mapping.

For the selected Blender exporter, verify the convention using an asymmetric front-marker fixture. The intended mapping is Blender -Y model front to exported +Z model front with +Y up. Inspect exported orientation, scale, pivot, and sockets instead of relying only on an exporter checkbox. Save front, side, and top views as evidence.

Keep canonical GLB interchange assets uncompressed. Web-only compression may produce separate build artifacts after compatibility and quality checks; it must not replace the source GLB. Verify the pinned Godot importer and required glTF extensions during port planning rather than assuming permanent engine support or non-support.

`tools/validate_assets.ts` checks profiles by asset kind, required files, provenance references, budgets, animation mapping, sockets, and generator determinism. Some properties need explicit metadata or a visual fixture; do not pretend every pivot/orientation semantic can be inferred from a file alone.

## 5. Lighting, shaders, and VFX

lookdev owns lighting, shaders, post-processing, and VFX code/data. Store lighting/post parameters in `data/visual/lighting.json` and effect parameters in `data/visual/vfx/<resource_id>.json`. Register their editor panels.

Effects react to discrete events and read-only state; they do not influence gameplay authority. gameplay owns the intent of gameplay-driven feedback such as hitstop and camera shake. Shader files describe their purpose and parameters in English. Record before/after performance measurements across all render passes when adding effects or post-processing.

lookdev may generate textures within its approved task, through MCP directly or through a manual request. art_tools owns inbox promotion, manifest entries, credits, and the prompt archive, so include that dependency before declaring the effect asset ready.

## 6. Target packages and measurement

art_tools prepares each `targets/<target_id>/` package: source clip, sampled frames, numbered contact sheets, and `timing.json`. The producer approves `binding` and `non_binding`; other roles read the approved version. Visual fidelity remains non-binding during the Three.js phase unless the producer explicitly changes scope.

Generated reference video expresses desired direction. It is not a trustworthy physics trace, proof of a realizable mechanic, or a source of unapproved hidden behavior. Use frames/timestamps as a portable evidence format regardless of whether a particular harness can ingest video.

Define these fields before timing-dependent implementation:

| Field | Required meaning |
| --- | --- |
| `schema_version`, `target_id`, `source_sha256` | Stable schema, package identity, and clip provenance |
| `gdd_refs`, `prompt_record` | GDD sections the target illustrates; prompt record of the generated clip |
| `source_timebase`, `source_fps`, `is_variable_frame_rate` | Original timing, distinct from sampling rate |
| `sample_fps`, `frame_index_base`, `clip_start_s` | Sampling convention; use zero-based indices |
| `frames` | Mapping from each sampled index/file to source presentation timestamp |
| `markers` | Named start/contact/end markers with measured time, uncertainty, and method |
| `binding` | Metric, expected value/range, units, tolerance, measurement method, and producer approval reference |
| `non_binding` | Explicit exclusions and their reason |
| `approval_revision` | Producer-approved target revision, linked to the source hash |

For a constant-rate zero-based sample sequence, `t_s = clip_start_s + frame_index / sample_fps`. Prefer actual source timestamps for variable-rate input. Never confuse a source frame index with a sampled frame index. Twelve samples per second are useful for contact sheets, but adjacent samples are about 83 ms apart; do not infer millisecond hitstop or contact timing from them. Use source-rate inspection and game telemetry for tighter tolerances, and report uncertainty.

Target timings initialize tuning only before producer approval of those fields. Subsequent playtest decisions take precedence and must record the intentional deviation or update the target revision. Reference comparison cannot silently undo approved tuning.

Review comparisons identify the same action start, camera/view settings, input script, data revision, and timebase. Preserve original recordings and logs; use contact sheets for inspection, not as the sole test of gameplay timing.

## 7. Godot handoff

`npm run export:blockouts` exports procedural reference geometry to `exports/blockouts/<resource_id>.glb` with dimensions, pivots, and sockets. Include processed GLB and Blender sources for generated assets. Record what procedural animation or shader behavior was not exported.

The M5 package includes final data schemas and values, assets with their prompt records, the GDD revision each part implements, replacement priorities, required clips/sockets, target approvals, recordings, and unresolved concerns. The port must explicitly map camera/model axes, FOV and aspect policy, physics, input, simulation timing, animation, events, and shader behavior. Similar coordinate handedness does not eliminate those decisions.

The Godot-phase rules are written separately, but these producer requirements carry into them:

- GDScript identifiers follow the Godot style guide in English. Doc comments (`##`) and inline comments are written in Chinese and explain what each script, signal, exported property, and non-obvious block does and why.
- Every ported system gets an illustrated Chinese explainer at `docs/godot/<system>.md`: the scene tree, signal flow, state machine where one exists, how data flows from the JSON files, and which Three.js module it replaces. Use Mermaid diagrams and Godot editor screenshots.
- A task that changes a system's behavior updates its explainer in the same task. An outdated explainer fails the definition of done in the Godot phase.

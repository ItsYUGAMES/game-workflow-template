# Architecture, Data, and Editor Contracts

Version: 0.7 | Date: 2026-11-10
This is a proposed contract design, not evidence that these interfaces already exist.

## 1. Dependencies and runtime flow

`src/core/` contains portable types, schemas, event contracts, entity IDs, simulation clock contracts, and seeded RNG utilities. It does not import gameplay, UI, rendering, or the editor.

- Simulation modules consume validated data and commands, own authoritative state, and publish discrete events. They do not import `src/ui/`, `src/visual/`, `src/editor/`, or rendering objects.
- Presentation consumes events plus a read-only state view through core interfaces. Whatever state presentation needs each frame (for example position, facing, health, current action, or board state) must have a defined query/snapshot path; do not encode every transform update as an allocated event.
- Events describe facts such as `attack_started` or `hit_landed`. User input and editor requests enter through command/data boundaries, not presentation writes to simulation state.
- `src/main.ts` is the composition root. systems wires simulation, presentation adapters, camera, UI, and development-only panels there.
- `src/camera/` may contain a presentation adapter owned by gameplay. Gameplay cannot import that adapter; camera-shake intent must cross the approved contract boundary.
- Colocated `*_panel.ts` files are editor adapters, not gameplay modules. Load them only through development composition so production cannot reach editor code transitively.

Before M1, systems publishes tested minimal contracts for entity lifecycle, read-only state access, commands, events, time, and data changes. Each event specifies payload, units, source entity, simulation tick/time, ordering, lifetime, and subscription cleanup. Subscribers must not mutate payloads or rely on undocumented dispatch order.

`AnimationDriver` is a presentation adapter. It maps simulation action state and time to animation states; it does not decide damage, collision, movement, or completion. Document playback speed, blending, interruption, missing-clip fallback, and teardown. Missing required animation support fails asset validation unless a recorded fallback is accepted.

## 2. Core evolution

M0 freezes the initial meaning and compatibility rules of the contracts. It does not make every source file immutable.

- systems owns all `src/core/` edits and coordinates affected consumers.
- Additive, backward-compatible events, optional fields with safe defaults, and new schemas may be approved by lead when existing semantics and saved data remain valid. Include consumer checks and version metadata.
- Renames, removals, required new fields, changed units, changed event ordering, changed timing semantics, or data migrations require producer approval before implementation.
- Do not evade review with an untyped event bus, duplicated schemas, or casts. An allegedly additive change that breaks a consumer is a breaking change.
- Preserve stable resource IDs, event names, input actions, and approved schema semantics across the future port; an adapter may change the engine implementation.

## 3. Time and reproducibility

Use seconds and a documented simulation clock. Seeded randomness alone does not provide a reproducible gameplay replay.

Before gameplay timing depends on it, approve and record the simulation tick policy, step size, catch-up cap, pause/background-tab behavior, any time scaling the GDD calls for (such as hitstop or slow motion), and input sampling order. Turn-based or event-driven games still record how turns or steps map to simulation time. A fixed-step simulation with separate render interpolation is the recommended design; exact values remain a project decision, not an inferred requirement.

Record replay inputs by simulation tick and include the data/contract revision. Keep independent RNG streams for gameplay, assets, and visual effects so changing a particle count cannot change an enemy spawn. Document stream seeding and reset behavior. App-controlled random decisions use seeded RNG, not `Math.random()`.

Avoid unnecessary allocations in measured hot paths; reuse buffers and vectors. Do not interpret this as a ban on initialization, immutable data revisions, or bounded event records. Optimize from evidence without weakening type or ownership contracts.

## 4. Naming, units, and code conventions

| Subject | Convention |
| --- | --- |
| Source files and directories | snake_case, e.g. `attack_chain.ts` |
| Types, classes, interfaces, enums | UpperCamelCase, no `I` interface prefix |
| TypeScript variables, functions, properties | lowerCamelCase; booleans start with is/has/can/should |
| Module constants and enum members | CONSTANT_CASE |
| Events, input actions, animation states, JSON keys | snake_case; events describe past facts, with `_started` / `_finished` for transitions |
| Scene node names | PascalCase, e.g. `SocketHandR` |
| Resource IDs | lowercase `<type>_<name>[_<variant>]` |

Treat acronyms as words (`loadGltfModel`), except external API names. Use TypeScript `private`, not underscore conventions. Use named exports in application code and `import * as THREE from 'three'` in rendering modules. Tool configuration may use a default export when required by its documented API. JSON boundary types may use snake_case in `src/core/data_schemas.ts`; convert to runtime conventions deliberately.

Use strict TypeScript, strict equality, and explicit exhaustive handling for discriminated unions. Reject `any`, `const enum`, and `var`. Do not add a catch-all `default` that hides an unhandled union variant. Consider splitting functions around 40 lines when responsibilities can be separated; line count is not an automatic failure.

The coordinate system is right-handed, +Y up, one unit per meter. Imported model forward is +Z; name it `MODEL_FORWARD`. Camera/view forward is distinct and must not share an ambiguous `FORWARD` constant. Keep axis conversion in adapters and validate orientation with a known fixture.

Runtime calculations use seconds and radians. Persisted measurements use unit suffixes: `_m`, `_s`, `_deg` or `_rad`, `_mps`, `_deg_per_s` or `_rad_per_s`. Convert at the data boundary. `turn_speed_deg` is not a valid angular-speed field. IDs, booleans, counts (`_count`), ratios (`_ratio`), and colors use their own declared type rather than fake physical units.

Keep gameplay tuning in validated data: for example collision dimensions, hurtboxes, action phases, contact times, timers, scoring, and rule parameters. Named algorithmic constants and mathematical factors may remain in code; not every literal belongs in an editor.

## 5. Data metadata and application timing

Each editable schema declares `schema_version`, type, units, range, step, dependencies, default, persistence path, owner, `apply_policy`, and `gdd_refs` where a GDD document defines the field ([traceability](traceability.md)). Unknown versions or invalid changes produce an actionable error and leave the last valid state intact.

Supported application policies:

| Policy | Meaning | Typical use |
| --- | --- | --- |
| `next_tick` | Apply one validated revision at the next simulation/render boundary | Simple speed or lighting adjustments |
| `next_action` | Existing action retains its captured revision; next action gets new values | Action phases, timing windows, contact times, the rules of an action already in progress |
| `rebuild` | Rebuild once from the latest valid revision, then swap and release old owned resources | Geometry and expensive visual structures |
| `restart` | Show that preview requires restarting the affected level/session | Structural level or incompatible configuration changes |

The panel displays the policy and any pending change. An agent must not promise next-frame application for every field. For one action in progress, every phase, window, and contact time it uses (for an attack: windup, active, recovery, cancellation, and contact) must come from the same data revision. The producer approves any different semantics.

Validation includes cross-field rules, such as non-negative phase lengths, explicit inclusive/exclusive interval boundaries, a contact time within its intended action phase, and timing windows inside the approved action timeline. The brief defines gameplay meaning; a schema must not invent it.

## 6. Editor save and hot reload

systems provides the panel registry, development-only writeback, schema validation, revision tracking, and visible save/conflict/error state. Owners supply field metadata and their panels. Add only the panels required by the active milestone; full drag-and-drop level editing is due when level-authoring tasks require it, not automatically before M0.

Keep persisted base data, unsaved editor changes, and the active runtime revision distinct. In-memory preview does not imply a successful disk save.

1. On load, validate the JSON and retain a content hash/revision as the edit base.
2. Preview a validated edit using its declared application policy. Preserve the unsaved patch separately.
3. Save a patch with the base revision to the specific allowlisted data file. The server validates the path, schema, and current revision before writing.
4. If the file changed, reject the stale write and show the changed fields. Preserve both versions; do not silently choose last-write-wins or automatically overwrite producer values.
5. Save atomically, return the actual new revision, and clear dirty state only after acknowledgement. A failed save keeps the unsaved patch visible.
6. On external file change, update a clean editor. For a dirty editor, report a conflict and require resolution before replacing its working copy. Suppress save/watch echo loops by revision.

Agent edits to JSON must re-read the current revision and change only approved fields. During producer tuning, coordinate writes to the same data file; preserving an old whole-file copy is not acceptable. New schema fields must not reset existing tuning. Removing or migrating a protected field requires producer approval.

Writeback is a local development feature: bind to loopback, validate host/origin, restrict real paths to allowlisted JSON data files, reject traversal and symlink escapes, and validate request sizes and schemas. Do not assume Vite's static-file restrictions secure custom middleware. Remote/mobile writeback needs a separately authorized design.

Production builds contain neither editor panels nor writeback routes. Verify both bundle reachability and server behavior. Hiding a panel with CSS or a runtime boolean does not prove exclusion.

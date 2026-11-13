# Agent Access to the Running Game

Version: 0.8 | Date: 2026-11-13
Status: Implementation contract for projects adopting this template. This repository does not contain a running game, MCP server, or HTTP service.

## 1. Required outcome and ownership

By M0, the actual coding Agent can connect to an isolated running game, read a small state snapshot, perform one observable action, retrieve a real rendered image, and close that instance. An installed MCP plugin, a listening port, or a screenshot path alone does not satisfy this requirement.

systems owns `src/dev/`, `tools/agent/`, launch/discovery, transport adapters, and their build integration. Feature owners expose their existing command/query boundaries; tester owns retained contract and integration checks under `tests/regression/agent/`. Do not duplicate gameplay inside a test server.

The normal automation route is the game interface. Extend a missing required operation at its owner boundary instead of repeatedly driving desktop windows. Targeted input/hit-testing checks may use browser/engine input through this same harness; producer playtesting remains separate. One assigned controller mutates an instance; concurrent task batches use separate instances and test storage.

## 2. Choose one working connection

Implement one route that the installed Agent can actually call:

| Route | Suitable connection | Requirements |
| --- | --- | --- |
| Simple local HTTP | Agent tool/script calls a loopback JSON API | Provide the URL, actual launch command, request/response examples, and artifact retrieval route |
| MCP over stdio | The Agent client starts a configured adapter process | Verify the installed client's launch configuration; protocol output goes to stdout and logs to stderr |
| MCP over Streamable HTTP | The Agent client connects to a configured MCP URL | Use a compatible pinned SDK/protocol; verify the actual client can discover and call the tools |

A plain HTTP JSON API is not automatically an MCP server. If a second transport becomes useful, make it a thin adapter over the same game bridge and handlers. Do not require both MCP and HTTP at bootstrap or implement a custom MCP protocol stack.

For the Three.js phase, a small local controller can own an isolated browser context and connect to a development-only typed bridge in the loaded game. The bridge feeds the real input/command pipeline, reads authoritative state, and captures the rendered app. Use trusted internal browser automation as needed; do not expose arbitrary evaluation or shell execution as Agent tools. A later Godot adapter preserves the logical operations while verifying that engine's actual input, timing, capture, and launch capabilities.

Loopback belongs to the machine running the service. A cloud Agent on another machine cannot reach that service merely by using its own `127.0.0.1`. Run the adapter beside the game or use an explicitly authorized restricted connection; record unavailable access honestly.

## 3. Small operation surface

These are project-defined operation names, not standard MCP methods. The HTTP paths are the recommended mapping when that route is selected; MCP tool names wrap the same handlers.

| MCP tool / logical operation | HTTP mapping | Required behavior |
| --- | --- | --- |
| `game_status` | `GET /v1/status` | Actual game readiness, instance/build/data identity, capability revision, supported operations and limits |
| `game_state` | `GET /v1/state` | Selected fields/scopes from one coherent current snapshot; include instance, state revision, scene and tick |
| `game_act` | `POST /v1/commands` | One action or bounded sequence through the declared input/command route; optionally await an observable condition |
| `game_result` | `GET /v1/commands/{command_id}` | Execution status and observations for an already-submitted command; never execute it again |
| `game_capture` | `POST /v1/captures` | Actual game image with instance, source, scene, state/tick/frame metadata, and a retrievable artifact |
| `game_shutdown` | `POST /v1/shutdown` | Close only the assigned test instance and its owned resources; acknowledge separately from observed termination |

Every operation except discovery/status identifies the expected `instance_id`; reject a stale instance before acting or returning evidence for another one. GET requests carry that field in the query. Operations validate their small documented inputs and return actionable errors such as `not_ready`, `stale_instance`, `unsupported`, `invalid_action`, or `timeout`.

Only expose actions the current game actually supports. Capability records specify action IDs/argument schemas, input routes, available snapshot fields, capture/recording support, output limits, and timeouts. Read them once on connection and after their revision changes. Add reset/scenario loading, recording, or other tools when a real task needs them; do not front-load a large editor API.

### Execution and observation

- Actions enter existing input/command boundaries; no direct writes to health, position, game outcomes, or other authoritative state. Dedicated test-fixture setup, when needed, is separate and is not evidence of gameplay execution.
- Distinguish `accepted` from `completed`. Return `command_id`, execution status, applied tick or UI frame, resulting state revision, and the requested observations. A successful HTTP response or MCP call alone does not prove the game reacted correctly.
- Give side-effecting submissions an instance-scoped `command_id`. Keep a bounded receipt cache: an identical retained request returns its existing result; the same ID with different arguments is rejected. After a timeout, query the receipt/state instead of blindly resubmitting. An expired/unknown receipt is not permission to replay an uncertain action. No persistent distributed job system is required.
- A batch declares finite steps and a timeout, stops on the first failed step, and reports what already executed. It is not an atomic transaction. Await a declared state/event condition in the harness, with timeout details, instead of making the Agent poll every frame.
- State-query data and capture metadata come from the same actual instance. A captured frame identifies the state/tick it rendered; do not silently attach a later query's revision to an earlier image.

A semantic command can establish gameplay behavior without establishing a key binding, focus rule, or pointer hit target. For UI/input acceptance, the harness must inject input through the actual browser/engine path and observe the response. Calling a button handler directly does not prove the button can be clicked. Report the route used, for example `command`, `input_action`, or `ui_pointer`.

### Token and artifact policy

Default results are short structured summaries. Allow field selection and bounded action sequences; return only relevant events/errors. Do not repeatedly return the whole scene tree, capability catalog, trace history, image base64 as text, or unchanged state. Logs and captures use artifact references; when the Agent needs to inspect an image, deliver it through a supported image/resource response or a verified accessible file/URL. Test retrieval from the actual Agent environment once.

A capture operation is an observation capability, not a requirement to capture every action. Demo reference keyframes remain restricted to selected important events under [assets and targets](assets_and_targets.md) §6.

## 4. Instance identity, defaults, and release boundaries

The launcher writes an ignored `.agent/runs/<run_id>/connection.json` after the game is ready. Include:

- protocol/schema version, project ID, actual connection mode and address or client configuration reference;
- `instance_id` generated by each game boot/reload, start time, and diagnostic PID where meaningful;
- actual source identity: commit plus dirty/input identity when applicable; data revision and current scene;
- capability revision, render capability, test-storage location, and an authentication reference when needed.

A reused port, PID, MCP connection, or surviving HTTP controller does not identify a newly loaded game. Reconnect after a restart/reload, verify the new game instance/build/data, and invalidate old state/action receipts. Handoffs link one short [connection record](templates/agent_connection.md); do not repeat discovery before every call.

Automation defaults must be implemented by the launcher: muted audio, no visible/focus-stealing window, isolated test saves and browser storage/profile, bounded lifetime, and deterministic fixture/seed where applicable. Never load, overwrite, or delete the player's real saves. A renderer-free simulation can satisfy logic checks but cannot claim screenshot support. If real rendering is unavailable, return an explicit unsupported result and keep capture acceptance incomplete.

Bind local network services to loopback, validate Host/Origin at that boundary, and use a per-launch credential or equivalent verified protection where applicable. Keep credentials out of tracked manifests and logs. Restrict artifact/writeback paths to their project-owned destinations; reuse [existing writeback validation](contracts.md#6-editor-save-and-hot-reload). These are real development-service boundaries, not a reason to add defensive wrappers to every internal function.

Development composition alone imports the bridge. Production must exclude debug routes, panels, arbitrary control access, and their reachable code. Check the actual production bundle and server behavior when this boundary is introduced/changed and at release acceptance; merely hiding UI or setting a runtime flag is insufficient.

## 5. One runner, selected checks

systems supplies one project runner under `tools/agent/`. The Three.js command spelling is `npm run run-game -- <mode> ...`; the future engine adapter may provide an equivalent `run-game` executable. These command names are planned interfaces, not commands already available in this template.

| Mode | Scope | When required |
| --- | --- | --- |
| `--serve` | Start/attach the chosen MCP or HTTP route to an isolated ready game and write discovery information | M0 |
| `--check` | Current data/schema references, registered resource keys, and required UI locale cells; delegate to canonical validators | M0 for data/content that exists |
| `--shot <scenario_id> <output.png>` | Run the declared fixture/input route and capture the real app; an interactive scenario must verify an actual input response | M0 for one representative scene |
| `--simulate --scenario <id> --seed <n> --steps <n>` | Run real simulation modules without rendering; output a bounded report and exit status | M1 when the core mechanic exists |

The runner sets test isolation by default, returns nonzero for failed required checks, invalid invocations, and unavailable requested capabilities, and keeps full raw output in evidence files. JSON summaries include status, instance/source identity where applicable, command/arguments, measured results, and log/artifact paths. Examples and `UNVERIFIED` placeholders are never successful output.

The runner uses the real implementation and existing validation code; do not duplicate formulas inside a second fake game. tester owns retained assertions/scenarios; systems owns the launcher/bridge implementation. Reuse an existing check for a bug when it covers the risk. For a new or changed critical validator, demonstrate one representative invalid fixture fails and a valid one passes, with isolated fixtures rather than corrupting project data. Do this when the checker changes, not after every ordinary task.

## 6. M0 proof and later adoption

M0 acceptance includes one recorded loop from the configured Agent:

1. Launch an isolated test instance and confirm game readiness, source/data identity, and capabilities.
2. Read selected state, perform one observable approved foundation action, await its actual outcome, and read the relevant result.
3. Retrieve and inspect one real image through the declared artifact route; record whether input semantics or actual UI hit testing was exercised.
4. Shut down that instance and observe its termination through the launcher/controller.
5. Relaunch once, confirm a new `instance_id`, and show that an old-instance request is rejected. Then clean up that test run.

Also establish the launcher defaults and production exclusion for this minimal interface. Keep the full actual outputs as linked evidence and summarize their result in the handoff. Failure to connect from the intended Agent or retrieve the image leaves this M0 requirement incomplete; unrelated documentation and authorized implementation can continue.

Do not postpone this direct-access loop until all content, shared libraries, or orchestration tooling exists. Conversely, do not require simulation of a nonexistent core game, video recording, every future action, or a complete MCP editor at M0.

At M1, core verbs expose their command/query hooks and a real simulation scenario. At M2, the complete runtime demo can be driven/replayed and recorded through the harness or its verified capture adapter, retaining the existing full-path and important-event policies. A future engine port re-proves connection, input, time, capture, and shutdown on that engine.

Use the existing [verification profiles](validation.md#2-verification-profiles-reuse-and-the-gate). Interface construction and verification frequency are separate: commands, observations, ordinary messages, and completion hooks do not themselves trigger a full gate.

## 7. Basis and protocol references

The requirement for direct access, isolated defaults, and one runner comes from the producer's supplied “给AI的搭建说明” and subsequent instruction prioritizing MCP or a simple connection. The operation names, receipt semantics, phases, and ownership here are this project's engineering design; no token or quality improvement has been measured.

MCP adapters follow the actual installed client's supported protocol/SDK. Official references checked on 2026-11-13:

- [stdio transport](https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/docs/specification/2026-07-28/basic/transports/stdio.mdx): process transport and protocol/log channels.
- [Streamable HTTP transport](https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/docs/specification/2026-07-28/basic/transports/streamable-http.mdx): MCP endpoint behavior and local endpoint boundaries.
- [Tools](https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/docs/specification/2026-07-28/server/tools.mdx): declared inputs/results, structured output and image/resource content.

These protocol references do not prescribe this game's six operations, test cadence, or architecture.

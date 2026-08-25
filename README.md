# Character Relay Local

**Local execution runtime for Character Relay, enabling characters to use MCP tools, plugins, local agents, game adapters, and live device streaming.**

> Status: **planning / Phase 0 contract freeze**. This repository currently defines the device-side product boundary and accepted runtime contracts. It does not yet provide an installable Local runtime or working game automation.

Character Relay Local is the optional device-side execution layer of [Character Relay](https://github.com/wong001110/character-relay). It is intended to let an authorized Character observe and act inside real applications and interactive environments while Character Relay Cloud remains the authority for Character identity, Deployment scope, permissions, Presence, autonomy admission, and durable lived evidence.

Gaming is the first embodiment use case, not the final product boundary.

## Target experience

```text
Character Relay Cloud
        |
        | secure outbound control / telemetry
        | HTTPS + WSS
        v
Character Relay Local
        |
        +-- Local GUI
        +-- Device runtime
        +-- Plugin / MCP host
        +-- Capture + audio
        +-- Permissioned input
        +-- Optional local agent
        +-- Adapter Lab
        |
        +--> Game / application adapter --> real environment
        |
        +--> WebRTC live media -----------> Character Relay Portal
```

The Local client initiates outbound connections. The design must not require users to expose a public inbound port or expose a local MCP server directly to Character Relay Cloud.

## Core design rules

### Cloud owns identity and autonomy admission; Local owns embodiment

Character Relay Cloud remains responsible for:

- owner/account authorization;
- Character Card and Deployment identity;
- owner-scoped Device access policy;
- Presence and session authority;
- autonomy mode, Activity Rhythm, intent admission, and Resource Scheduler decisions;
- high-level intent and long-term Character memory;
- Portal session/live-view authorization;
- persisted verified session outcomes.

Character Relay Local is responsible for:

- device identity and local secure storage;
- cloud connection, heartbeat, reconnect, and session resume;
- process/window detection;
- target-window and audio capture;
- local input backends;
- WebRTC publishing;
- plugin/MCP lifecycle;
- plugin permissions and Local safety denial;
- Execution Lease enforcement against the cloud-authorized session;
- Adapter Lab and validation;
- optional local Runtime Agent providers;
- optional Coding Agent providers for adapter development/repair.

### MCP is a tool boundary, not the media transport

Planned transport split:

```text
Cloud <-> Local        HTTPS / WSS
Local <-> Plugin       local MCP, preferably stdio/process-local
Local -> Portal video  WebRTC
```

MCP should expose high-level skills such as `run_skill(...)`, `observe(...)`, or environment-specific actions. Normal cloud reasoning should not operate a game by issuing raw mouse coordinates or per-frame key presses.

### Device Protocol has one source of truth

The accepted v1 direction is one versioned Character Relay Device Protocol authority in this repository:

```text
TypeSpec
  -> generated JSON Schema + conformance fixtures
       -> Local TypeScript consumer
       -> Character Relay Cloud Python/Pydantic consumer
```

The protocol uses standard WSS + versioned JSON. Socket.IO protocol is not a platform dependency. Protocol major version is independent from the Local application version.

### Plugin-first adapters

Generic OS/device capabilities stay in Local Core. Game/application-specific knowledge lives in plugins.

- **Plugin** = installable/distributable package.
- **Adapter** = runtime integration inside a plugin for one environment.

V1 exposes Official and explicitly enabled Developer plugins. Verified/untrusted public plugin distribution is deferred until a real OS/process sandbox exists and is validated.

First-party adapters may initially be developed in this repository, but they should use the same plugin contract intended for later external packages.

### Live means live

The target Portal experience is a low-latency video/audio stream, not periodic screenshots.

The initial implementation target is intentionally modest:

- target-window capture;
- 720p30 baseline;
- H.264 video;
- game/system audio where supported;
- hardware encoding when available;
- on-demand publishing only while an authorized viewer is watching;
- direct WebRTC P2P with STUN and TURN fallback;
- no v1 recording/media persistence;
- agent observation frequency independent from viewer frame rate.

### Permissions before autonomy

Local execution must be capability-, policy-, risk-, session-, and lease-gated. A plugin or agent does not receive unrestricted desktop authority merely because it can be called through MCP.

Commercial-game integration must not make anti-cheat bypass, process-memory injection, packet interception, credential extraction, or client tampering part of the Character Relay platform contract.

## Autonomy and Activity Rhythm

Character autonomy is not a prompt permission. The accepted modes are:

```text
OFF
SHADOW
REVIEW
AUTO
```

Local action origins are distinguished as:

```text
user_request
user_delegated
character_initiated
session_recovery
```

Activity autonomy may have Deployment-scoped time preferences similar in UI to Sleeping Rhythm:

- Sleep is a hard gate.
- `allowed` activity windows are hard execution windows for Character-initiated activity.
- `preferred` activity windows are soft preferences that influence when Runtime creates an autonomy opportunity.

The control loop is:

```text
Runtime opportunity
  -> Character high-level intent
  -> deterministic policy admission
  -> Resource Scheduler
  -> Execution Session + Lease
  -> Local execution
  -> verified outcome
```

Characters do not decide who wins a shared-device conflict. The deterministic Resource Scheduler gives conflicting origin priority to explicit user requests, then delegated work, session recovery, and finally Character-initiated activity. Lower-priority work may be deferred and cooperatively preempted at safe checkpoints rather than silently losing its lease.

See [`docs/contracts/autonomy-policy-v1.md`](docs/contracts/autonomy-policy-v1.md).

## Execution modes

The planned runtime supports three execution policies without changing adapter contracts:

1. **Cloud Agent** — Character Relay Cloud decides high-level execution.
2. **Local Agent** — an optional local provider handles bounded local reasoning/tool selection.
3. **Hybrid** — Cloud owns Character intent and long-term state; Local handles immediate environment reasoning/recovery.

Preferred fallback shape:

```text
deterministic skill
  -> optional local Runtime Agent
  -> cloud agent
  -> human takeover
```

A **Coding Agent** is a separate concept from a Runtime Agent. Codex, Claude Code, or other coding agents may later be integrated with Adapter Lab to inspect failure traces, edit adapter code, reload, and run validation. They are not the default game-playing runtime and must not auto-push unreviewed changes to `main`.

## Adapter Lab

Developer Mode is planned as a first-class part of the desktop app.

The Lab should eventually provide:

- target/process inspector;
- live capture/vision inspector;
- input tester with explicit temporary arming;
- MCP Tool Explorer;
- high-level Skill Runner and state trace;
- sanitized session recorder and fixtures;
- L0–L3 validation gates;
- local WebRTC diagnostics;
- hot reload and regression runs;
- diff/promote workflow;
- optional Coding Agent repair flow.

The goal is to prove that a tool works against the real environment before it is promoted into a trusted adapter workflow.

## First delivery boundary

The first product milestone is **not autonomous gameplay**.

It is:

```text
Device Pairing
  -> Device Online/Offline
  -> real game/application detection
  -> Character gaming/activity Presence
  -> Portal session card
  -> authorized Watch Live
```

Only after Presence + Live Watch are stable should the project add MCP plugins, input control, Adapter Lab automation, game skills, or local-agent delegation.

## Planned repository shape

This is a target layout, not proof that these packages already exist:

```text
character-relay-local/
  apps/
    desktop/
      normal/
      adapter-lab/

  packages/
    runtime/
    protocol/
    mcp-host/
    plugin-host/
    plugin-sdk/
    capture/
    input/
    streaming/
    recorder/
    test-harness/

  plugins/
    test-environment/
    # first-party adapters later

  installer/
```

Do not create empty package structure merely to match this diagram. Add directories when a phase actually owns working code/tests.

## Contracts and roadmap

- [`docs/phase-0-contract-freeze.md`](docs/phase-0-contract-freeze.md) — accepted Phase 0 architecture ledger.
- [`docs/contracts/README.md`](docs/contracts/README.md) — Local contract index.
- [`ROADMAP.md`](ROADMAP.md) — phased delivery plan.

The corresponding Character Relay cloud/product roadmap is maintained in the main repository at [`docs/local-execution-roadmap.md`](https://github.com/wong001110/character-relay/blob/main/docs/local-execution-roadmap.md) once that planning change is merged.

## Development status

Runtime implementation has not started. Phase 0 is currently freezing/synchronizing contracts across the Local and Cloud repositories.

Phase 1 may begin only after the synchronized Phase 0 contracts are accepted. At that point re-read the then-current Character Relay `main` and do not assume planning documents prove unchanged runtime schemas.

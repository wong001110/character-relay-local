# Character Relay Local

**Local execution runtime for Character Relay, enabling characters to use MCP tools, plugins, local agents, game adapters, and live device streaming.**

> Status: **planning / pre-implementation**. This repository currently defines the device-side product boundary and roadmap. It does not yet provide an installable Local runtime or working game automation.

Character Relay Local is the optional device-side execution layer of [Character Relay](https://github.com/wong001110/character-relay). It is intended to let an authorized Character observe and act inside real applications and interactive environments while Character Relay Cloud remains the authority for Character identity, Deployment scope, permissions, Presence, and durable lived evidence.

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

### Cloud owns identity; Local owns embodiment

Character Relay Cloud remains responsible for:

- owner/account authorization;
- Character Card and Deployment identity;
- Presence and session authority;
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
- plugin permissions;
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

### Plugin-first adapters

Generic OS/device capabilities stay in Local Core. Game/application-specific knowledge lives in plugins.

- **Plugin** = installable/distributable package.
- **Adapter** = runtime integration inside a plugin for one environment.

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
- agent observation frequency independent from viewer frame rate.

### Permissions before autonomy

Local execution must be capability- and permission-gated. A plugin or agent does not receive unrestricted desktop authority merely because it can be called through MCP.

Commercial-game integration must not make anti-cheat bypass, process-memory injection, packet interception, credential extraction, or client tampering part of the Character Relay platform contract.

## Execution modes

The planned runtime supports three policies without changing adapter contracts:

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

## Roadmap

See [`ROADMAP.md`](ROADMAP.md).

The corresponding Character Relay cloud/product roadmap is maintained in the main repository at [`docs/local-execution-roadmap.md`](https://github.com/wong001110/character-relay/blob/main/docs/local-execution-roadmap.md) once that planning change is merged.

## Development status

Implementation is intentionally deferred while current Character Relay feature work continues.

When development is explicitly started, begin with roadmap **Phase 0 — Contract freeze**, re-read the then-current Character Relay `main`, and do not assume this planning document still matches runtime schemas unchanged.

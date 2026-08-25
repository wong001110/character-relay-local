# Phase 0 — Contract Freeze

Status: **IN PROGRESS — contract draft for review**

Branch: `docs/phase-0-contract-freeze`

This phase freezes the durable boundaries that implementation must follow. It does not add an installable desktop runtime, MCP host, WebRTC publisher, plugin execution, input control, or game automation.

## Accepted architecture decisions

### Product boundary

- `character-relay` remains authoritative for owner/account authorization, Character Card and Deployment identity, Presence/availability, long-term Character memory/evidence, and Portal authorization.
- `character-relay-local` is an optional device execution node. It owns device identity, local connection/session mechanics, local observation and control, streaming, plugin lifecycle, Adapter Lab, and optional local/coding-agent providers.
- Local is not a second Character Relay server and must not invent independent Character or Deployment identity.

### Initial platform and stack

V1 is **Windows-first**. Cross-platform interfaces may be kept where inexpensive, but macOS/Linux parity is not a Phase 1 requirement.

Planned implementation stack:

- Electron desktop shell;
- React + TypeScript + Vite UI;
- Node.js/TypeScript local orchestration;
- Rust native sidecar using Windows APIs for security/performance-sensitive OS capabilities;
- pnpm workspace monorepo;
- official MCP TypeScript SDK for local plugin tool transport;
- standard HTTPS + WSS for Cloud ↔ Local control/telemetry;
- WebRTC for live video/audio;
- SQLite for non-secret local metadata when persistence is required;
- OS-backed secure credential storage for device credentials.

Electron/TypeScript is the primary application/runtime language. Rust is a native boundary, not the default language for Agent, Plugin, Adapter, or GUI logic.

### Cloud ↔ Local protocol

- Use standard WSS + versioned JSON messages; do not make Socket.IO protocol a platform dependency.
- Borrow useful Socket.IO semantics such as typed events, acknowledgement, reconnect, and session recovery at the Character Relay Device Protocol layer.
- Pair/bootstrap/authentication uses HTTPS; steady-state device control/telemetry uses WSS.
- MCP stays local between the Local host and plugins/adapters. Local MCP servers are never exposed directly to Character Relay Cloud or the public Internet.
- Media does not travel through MCP or the control WebSocket. Live media uses WebRTC.

See [`contracts/device-protocol-v1.md`](contracts/device-protocol-v1.md).

### Presence and Gaming representation

Gaming is **an activity, not a new top-level availability enum**.

The planned cloud integration should preserve the existing availability-style Presence state and represent gameplay as rich activity metadata, conceptually:

```text
Presence.state = busy
Presence.activity.kind = gaming
Presence.activity.target = <game>
Presence.activity.session_id = <verified local session>
```

The current Character Relay source only permits `activity_type` on `browsing`, so this decision requires an explicit future cloud schema/runtime change. Until that change exists, Local must not claim that Gaming Presence is implemented.

This avoids adding a new Presence enum for every future embodiment activity such as gaming, creating media, coding, or controlling a robot.

### Execution ownership

- One device may serve multiple Characters.
- A Character does not own the whole machine simply because it has an active session.
- Side-effecting local resources are controlled through **Execution Leases**.
- Exclusive resources such as desktop keyboard/pointer control may have only one active owner at a time.
- Read-only resources such as capture may support shared-read leases when policy permits.
- Human takeover, local safety policy, and explicit Stop always outrank model/agent intent.

See [`contracts/execution-session-v1.md`](contracts/execution-session-v1.md).

### Plugin and Adapter boundary

- Local Core owns generic device capabilities: process/window discovery, capture, input backends, streaming, secure storage, permissions, plugin lifecycle, and agent-provider hosting.
- A Plugin is an installable/distributable package.
- An Adapter is an environment-specific runtime integration inside a Plugin.
- Plugins call permissioned Host Capabilities; MCP does not itself grant OS authority.
- First-party plugins may be trusted code initially, but the product must not pretend arbitrary third-party Node processes are OS-sandboxed before a real sandbox exists.

See [`contracts/plugin-permission-v1.md`](contracts/plugin-permission-v1.md).

### Agent execution

Runtime and coding agents remain separate:

```text
deterministic skill
  -> optional Local Runtime Agent
  -> optional Cloud Agent
  -> human takeover
```

Codex, Claude Code, and similar coding agents belong to the Developer/Adapter Lab path by default. They may inspect bounded source/traces, patch adapters, reload, run live/regression gates, and produce diffs; they do not become the default gameplay runtime and do not auto-push/merge.

### Live Watch

- Live Watch is ephemeral and on-demand.
- Portal authorization is cloud-owned.
- The viewer receives WebRTC video/audio; the control plane only handles signaling/authorization/state.
- P2P is preferred; TURN is fallback.
- No viewer receives local input/control authority merely by watching.
- Cloud does not persist raw video/audio by default.
- Agent observation cadence is independent from viewer frame rate.

### Privacy boundary

Prefer:

```text
raw local environment
  -> local perception / adapter
  -> structured verified event
  -> cloud
```

rather than continuously uploading desktop frames to Cloud. Raw-frame use by a Cloud VLM must be a distinct capability/policy decision and scoped to the active target/session.

## Phase 0 authority map

| Concern | Authority |
| --- | --- |
| Owner / Character / Deployment semantics | `character-relay` current contracts/source |
| Device transport schema | `character-relay-local` Device Protocol contract and future protocol package |
| Local session / lease semantics | `character-relay-local` Execution Session contract |
| Plugin host permissions | `character-relay-local` Plugin Permission contract |
| Portal authorization / Presence persistence | `character-relay` |
| WebRTC media transport | Local + Portal implementation under cloud authorization |
| MCP tool contract | Local Plugin Host / plugin SDK; never public transport |

The future executable Device Protocol schema should have one source of truth in this repository (planned `packages/protocol` or equivalent). The cloud repository may consume generated/versioned artifacts, but must not independently redefine the wire schema. Cloud-domain identifiers retain their semantics from `character-relay`; the Local protocol only transports them.

## Phase 0 remaining gate

Before Phase 1 implementation starts:

- [x] Windows-first and initial stack direction recorded.
- [x] Standard WSS Device Protocol direction recorded.
- [x] Gaming represented as rich activity under availability Presence rather than a new enum.
- [x] Execution Lease and session ownership model drafted.
- [x] Plugin capability/permission/trust model drafted.
- [x] Runtime Agent vs Coding Agent boundary recorded.
- [x] Live Watch / WebRTC and privacy boundaries recorded.
- [ ] Companion `character-relay` roadmap references the same accepted Presence/activity and protocol decisions.
- [ ] Owner accepts the contract PR.

Phase 1 must not start until the final two items are satisfied.

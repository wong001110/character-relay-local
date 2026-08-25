# Phase 0 — Contract Freeze

Status: **IN PROGRESS — contract draft for review**

Branch: `docs/phase-0-contract-freeze`

This phase freezes the durable boundaries that implementation must follow. It does not add an installable desktop runtime, MCP host, WebRTC publisher, plugin execution, input control, or game automation.

## Accepted architecture decisions

### Product boundary

- `character-relay` remains authoritative for owner/account authorization, Character Card and Deployment identity, Presence/availability, long-term Character memory/evidence, Portal authorization, autonomy admission, and durable session/result projection.
- `character-relay-local` is an optional device execution node. It owns device identity, local connection/session mechanics, local observation and control, streaming, plugin lifecycle, Adapter Lab, Local safety enforcement, and optional local/coding-agent providers.
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

### Protocol schema authority

The executable Character Relay Device Protocol has one source of truth in `character-relay-local`.

Accepted v1 direction:

```text
TypeSpec source
  -> JSON Schema artifacts / conformance fixtures
       -> Local TypeScript runtime validation/types
       -> Character Relay Cloud Python/Pydantic consumer
```

- Do not open a third protocol repository for v1.
- Protocol major version is independent from Local application version.
- JSON wire semantics and shared valid/invalid conformance fixtures must be consumed by both repositories.
- Cloud-domain identifiers retain their meaning from `character-relay`; the Device Protocol transports those IDs but does not redefine them.
- Rust native sidecar does not need to implement the Cloud Device Protocol; it communicates through a separate local native IPC boundary.

Library/code-generation details may evolve provided TypeSpec remains the semantic source and both repos validate the same wire contract.

### Device ownership and Deployment authorization

A Device is **owner/account-scoped**, not permanently owned by a Character.

```text
Owner
  -> Device(s)
  -> Deployment(s)
```

A Local execution session binds an authorized Deployment to an owner Device for a bounded session. Runtime authorization resolves to concrete `deployment_id`; Character Card identity may be used for authoring/UX convenience but does not create Character-global device authority.

One Device may serve multiple Characters/Deployments concurrently when their required Execution Surfaces do not conflict.

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

### Autonomy Policy and Activity Rhythm

Character autonomy is admission-controlled Cloud policy, not prompt authority.

Accepted modes:

```text
OFF
SHADOW
REVIEW
AUTO
```

Accepted intent origins:

```text
user_request
user_delegated
character_initiated
session_recovery
```

Activity autonomy may have Deployment-scoped time policy similar in UX to Sleeping Rhythm:

- Sleep is a **hard gate** and blocks Character-initiated Local activity.
- Activity `allowed` windows are hard execution windows.
- Activity `preferred` windows are soft preferences that influence when Runtime offers an autonomy opportunity.
- Activity schedule is durable policy and is not silently self-edited by a Character model.

Runtime determines **when** an opportunity may be considered; Character cognition may choose **what** high-level activity it wants; deterministic admission policy decides **whether** it may execute.

See [`contracts/autonomy-policy-v1.md`](contracts/autonomy-policy-v1.md).

### Shared-device Resource Scheduler

Characters never use an LLM to decide who wins a device conflict.

For otherwise-valid intents contending for the same exclusive resource, canonical origin priority is:

```text
user_request
  > user_delegated (inside its valid window)
  > session_recovery
  > character_initiated
```

Within the same class, deterministic age/deadline/fairness policy may apply. Character-initiated intents have bounded validity and do not remain forever queued behind an offline/busy device.

Higher-priority work normally uses **cooperative preemption** at adapter-declared safe checkpoints. Immediate hard interruption is reserved for human Stop/Take Over, Local safety/security enforcement, permission revocation, or equivalent emergency conditions.

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
- V1 trust levels exposed to users are **Official** and **Developer**. `Verified` third-party distribution is deferred until a real OS/process sandbox exists and is validated.
- First-party/Developer Node processes are trusted code in v1; Host Capability permissions must not be misrepresented as a complete OS sandbox.

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

### Live Watch v1

Live Watch v1 uses **direct WebRTC P2P with STUN and TURN fallback**, behind a transport abstraction so a later SFU/LiveKit backend does not change the product/session contract.

- Live Watch is ephemeral and on-demand.
- Portal authorization is cloud-owned.
- The viewer receives WebRTC video/audio; the control plane only handles signaling/authorization/state.
- No recording or Cloud media persistence is part of v1.
- No viewer receives local input/control authority merely by watching.
- Agent observation cadence is independent from viewer frame rate.
- Portal Stop/session commands travel through Character Relay Cloud, not directly over the P2P media connection.

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
| Device ownership/access + autonomy admission | `character-relay` cloud product/runtime |
| Device transport schema | `character-relay-local` TypeSpec Device Protocol source + generated artifacts |
| Local session / lease semantics | `character-relay-local` Execution Session contract |
| Autonomy/resource scheduler semantics | Cloud admission follows `character-relay-local` Autonomy Policy contract until mirrored as cloud implementation contracts |
| Plugin host permissions | `character-relay-local` Plugin Permission contract |
| Portal authorization / Presence persistence | `character-relay` |
| WebRTC media transport | Local + Portal implementation under cloud authorization |
| MCP tool contract | Local Plugin Host / plugin SDK; never public transport |

The future executable Device Protocol schema has one source of truth in this repository. The cloud repository consumes generated/versioned artifacts and shared conformance fixtures; it must not independently redefine the wire schema.

## Phase 0 remaining gate

Before Phase 1 implementation starts:

- [x] Windows-first and initial stack direction recorded.
- [x] Standard WSS Device Protocol direction recorded.
- [x] TypeSpec → JSON Schema single protocol authority recorded.
- [x] Owner-scoped Device + Deployment-bound session model recorded.
- [x] Gaming represented as rich activity under availability Presence rather than a new enum.
- [x] Autonomy modes, intent origins, Activity Rhythm, risk/admission policy recorded.
- [x] Deterministic multi-Character Resource Scheduler and cooperative preemption recorded.
- [x] Execution Lease and session ownership model drafted.
- [x] Plugin capability/permission/trust model drafted; Verified third-party deferred until real sandboxing.
- [x] Runtime Agent vs Coding Agent boundary recorded.
- [x] Direct WebRTC P2P + STUN/TURN Live Watch v1 and privacy boundaries recorded.
- [ ] Companion `character-relay` roadmap references the same accepted Presence/activity, protocol, Device ownership, autonomy/scheduler, Live Watch, and plugin-trust decisions.
- [ ] Owner accepts the synchronized Phase 0 contract PR as a whole.

Phase 1 must not start until the final two items are satisfied.

# Character Relay Local Roadmap

Status: **PLANNED / DEFERRED — PRE-IMPLEMENTATION**

This roadmap is the device-side delivery plan for Character Relay Local. It intentionally stops short of implementation while the main Character Relay repository is undergoing other feature work.

The cloud/product-side companion roadmap lives in `wong001110/character-relay` as `docs/local-execution-roadmap.md` once merged.

## Delivery principle

Do not start by building a game bot.

Prove the platform in this order:

```text
Device connectivity
  -> Presence
  -> Live Watch
  -> Plugin boundary
  -> Adapter Lab
  -> deterministic control
  -> Character orchestration
  -> optional local/coding agents
```

This ordering keeps network, media, input, plugin, and agent failures independently diagnosable.

## Milestone A — Local device + live Presence

### Phase 0 — Contract freeze

No runtime code should be treated as stable until this phase is accepted.

Define with the then-current Character Relay `main`:

- device identity and pairing contract;
- owner / Character Card / Deployment / device / session scope;
- Cloud ↔ Local protocol and versioning;
- heartbeat, offline, reconnect, interruption, and resume semantics;
- gaming/activity representation in existing Deployment Presence;
- Local Autonomy Policy, Activity Rhythm, and risk/admission semantics;
- multi-Character Resource Scheduler, Execution Lease, contention, and safe preemption;
- live-view authorization and signaling contract;
- plugin capability/permission/trust model;
- security threat model and explicit non-goals;
- first supported desktop OS and packaging assumptions;
- one source of truth for shared protocol schemas.

Gate:

- the cloud and local repositories reference the same accepted contracts;
- no implementation depends on an invented `gaming` Presence state or unversioned event shape;
- Character autonomy cannot bypass deterministic admission or Local safety;
- multiple Characters cannot silently steal the same exclusive execution resource;
- input, capture, streaming, and plugin permissions have explicit authority owners.

### Phase 1 — Desktop shell + device pairing

Build the smallest useful Local client:

- desktop application shell;
- Normal Mode status GUI;
- local secure credential storage;
- pairing flow initiated by the owner;
- outbound HTTPS/WSS connection;
- heartbeat and online/offline state;
- automatic reconnect with bounded backoff;
- session-resume envelope without game behavior;
- local diagnostic logs with secret redaction.

Gate:

- a fresh install can pair and unpair;
- no inbound public port is required;
- cloud reports device online/offline correctly;
- reconnect does not mint duplicate device identity.

### Phase 2 — Generic target/activity detection

Add generic environment observation without control:

- process/window inventory;
- target registry;
- target start/stop detection;
- generic activity events;
- current target session state;
- disconnect/interruption cleanup;
- development test target independent of commercial games.

Gate:

- starting/stopping a test target generates the expected activity lifecycle;
- no keyboard/mouse permission is required;
- Local does not invent Character/Deployment scope itself; it uses the cloud-authorized session binding.

### Phase 3 — Live Watch

Implement real-time media separately from control telemetry:

- target-window capture;
- game/system audio capture where supported;
- 720p30 baseline;
- H.264 baseline codec;
- hardware encoder discovery/fallback;
- direct WebRTC P2P publisher abstraction;
- STUN with TURN fallback;
- authorization/signaling integration;
- viewer-count-driven start/stop;
- reconnect and stream teardown;
- Local GUI preview and diagnostics.

Keep agent observation independent from stream frame rate. V1 does not record or persist Live Watch media.

Gate:

- authorized Portal viewer receives low-latency video and audio;
- unauthorized viewers cannot subscribe;
- closing the viewer tears down publishing when subscriber count reaches zero;
- WebRTC failure does not break Presence/control telemetry.

Milestone A acceptance:

> A paired device can report a real running game/application as Character activity, and the owner can open a live Portal stream without granting automation permissions.

---

## Milestone B — Plugin platform + Adapter Lab

### Phase 4 — Plugin Host + MCP boundary

Introduce environment-specific capability packages.

Local Core owns generic device primitives. Plugins own environment knowledge.

Implement:

- plugin manifest schema;
- plugin identity/version/compatibility;
- capability discovery;
- explicit permission requests;
- Local Host APIs for approved generic primitives;
- plugin lifecycle/start/stop/reload;
- local MCP process/client-host integration, preferring process-local/stdio initially;
- reference `test-environment` plugin;
- protocol/version mismatch handling.

V1 supports Official and explicitly enabled Developer plugins. Verified/untrusted third-party distribution remains deferred until a real OS/process sandbox is implemented and validated.

Do not expose Local MCP servers directly to the Internet.

Gate:

- Local discovers one reference plugin;
- MCP tools can be listed and called locally;
- plugin calls outside approved capability/permission scope are rejected;
- plugin failure cannot take down the Local cloud connection;
- the GUI does not misrepresent Host API permissions as a complete OS sandbox.

### Phase 5 — Adapter Lab GUI

Developer Mode becomes the primary integration harness.

Build:

- Target Inspector;
- Capture Inspector with live preview;
- detected-region/vision overlays;
- Input Tester with explicit temporary **Arm Input** control;
- target-window restriction and emergency stop;
- MCP Tool Explorer with schema-driven arguments;
- Skill Runner;
- state/step trace viewer;
- session recorder;
- sanitized fixture generation;
- hot reload;
- relevant regression rerun;
- WebRTC diagnostics;
- diff/promote workflow.

Validation levels:

- **L0 Host** — process, capture, input, audio, streaming;
- **L1 Adapter primitive** — observe/detect/interact primitive;
- **L2 Skill** — high-level deterministic skill;
- **L3 Session** — cloud-authorized Character session through verified outcome.

Gate:

- a developer can verify a tool against a real test target without invoking Character Relay's cloud model;
- traces/fixtures contain no credentials or unrelated private desktop data;
- successful manual validation can become a repeatable regression fixture;
- Promote, Commit, and Push remain separate actions.

Milestone B acceptance:

> A reference plugin can be inspected, permissioned, exercised, watched live, recorded, and validated in Adapter Lab before any Character uses it.

---

## Milestone C — Deterministic environment control

### Phase 6 — First real adapter

Start with an owned test environment, API/mod-friendly game, or otherwise low-risk target. Do **not** make a closed commercial game the first proof of the architecture.

Implement:

- target detection;
- structured observation;
- known-screen/state recognition;
- high-level skills;
- deterministic state machine;
- local CV/OCR fast path;
- bounded VLM fallback for unknown states if needed;
- local input controller;
- high-frequency motor loop only where necessary;
- timeout, cancel, failure, safe-checkpoint/preemption, and safe-stop behavior.

Preferred tool granularity:

```text
GOOD
run_skill("collect_reward")
open_inventory()
navigate_to(...)

NOT THE CLOUD DEFAULT
mouse_click(x, y)
press_key("W") every frame
```

Gate:

- the same bounded task succeeds repeatedly;
- expected states and outcomes are verified rather than assumed;
- cancel/emergency stop works during execution;
- adapter exposes safe preemption checkpoints where the task cannot be interrupted arbitrarily;
- adapter remains useful for Presence/Live Watch if control is disabled.

### Phase 7 — Character session execution

Connect the proven adapter to Character Relay orchestration:

- receive authorized high-level task;
- bind task to device + Deployment/session;
- apply Cloud Autonomy Policy/admission for Character-initiated work;
- acquire required Execution Leases through the deterministic Resource Scheduler;
- emit structured progress;
- execute local high-level skill;
- handle defer/cancel/interruption/resume and cooperative preemption;
- emit verified completion/result;
- support owner takeover/stop and autonomy suppression;
- ensure stale device/session state cannot leave Portal permanently active.

Gate:

- L3 validation passes;
- Character-initiated intents outside allowed Activity Rhythm windows cannot start;
- conflicting exclusive-resource intents are deferred/expired/preempted according to policy rather than silently stealing control;
- the Character can later describe only verified events from the session;
- Local cannot widen Deployment/owner scope.

Milestone C acceptance:

> A Character can execute one bounded real environment task through a local adapter while Portal Presence and Live Watch stay synchronized and the final outcome is objectively recorded.

---

## Milestone D — Optional local intelligence

### Phase 8 — Runtime Agent providers

Runtime Agent support is optional. Adapters must work without it where deterministic skills are sufficient.

Provider candidates:

- local OpenAI-compatible endpoint;
- Ollama-class local runtime;
- LM Studio-class runtime;
- custom provider.

Execution policy:

```text
deterministic skill
  -> local Runtime Agent (optional)
  -> cloud Agent (optional)
  -> human takeover
```

Support privacy policy such as:

- local frames stay local;
- cloud receives structured state/progress only;
- raw frame upload requires separate policy/permission.

Gate:

- enabling/disabling Local Agent does not change plugin contracts;
- a task can fall back safely;
- Local Agent cannot bypass plugin, autonomy, risk, lease, or input permissions.

### Phase 9 — Coding Agent providers

Coding Agent is a Developer Mode capability, separate from Runtime Agent.

Candidate integrations:

- Codex;
- Claude Code;
- custom coding-agent CLI/SDK.

Adapter Lab should expose a bounded development tool surface such as:

- inspect adapter/tool status;
- inspect sanitized failure trace;
- capture/observe test target;
- run tool/skill;
- reload adapter;
- run targeted regression;
- run full adapter gate.

Workflow:

```text
failure
  -> coding agent receives bounded repo + Lab context
  -> patch
  -> reload
  -> live/test validation
  -> diff
  -> human accept
  -> optional local commit
  -> separate push action
```

Never display or persist a provider's private chain-of-thought. Retain action summaries, tool calls, test evidence, and diffs instead.

Gate:

- an intentionally broken reference adapter can be repaired through the Lab;
- repair cannot bypass validation/permission/diff review;
- no automatic push/merge to `main`.

---

## Milestone E — Distribution and ecosystem

### Phase 10 — Packaging, updater, and plugin distribution

After the runtime/SDK contracts prove stable:

- production installer;
- code signing strategy;
- core updater and rollback;
- plugin install/update/rollback;
- compatibility/version matrix;
- Official / Developer trust levels for v1;
- real sandbox requirement before introducing Verified/untrusted public plugins;
- plugin SDK docs;
- first-party adapter packaging independent from Local Core release;
- commercial-game support matrix split into Presence / Integration / Visual Control.

Gate:

- broken plugin update can roll back independently of Local Core;
- users can inspect permissions/trust before enabling a plugin;
- Local update does not silently increase plugin authority.

## Commercial game policy

For games such as Genshin Impact or Honkai: Star Rail, support must be capability-specific rather than assumed:

```text
Presence      potentially supported
Live Watch    potentially supported
Control       evaluate per target / rules / compatibility
```

Character Relay Local must not implement anti-cheat bypass, memory injection, packet interception, credential extraction, or game-client tampering as platform features.

A commercial-game adapter may remain Presence/Live-only even when the rest of the platform supports control.

## Deferred beyond the Local MVP

- Cloud GPU Game Runner;
- headless remote runner infrastructure;
- simulation-only Game Runner;
- generic unrestricted desktop agent;
- public plugin marketplace / Verified third-party plugins before real sandboxing;
- automatic coding-agent merge/push;
- direct cloud access to local MCP servers;
- multi-OS parity before the first OS path is proven;
- SFU/LiveKit transport unless multi-viewer/media-server requirements justify it.

## Start condition

Do not treat unchecked roadmap items as active work.

Development begins only after the owner explicitly promotes this repository into an implementation phase. At that time:

1. re-read Character Relay's current source/tests/contracts;
2. reconcile any Presence/Portal/runtime changes;
3. freeze Phase 0 contracts;
4. create the actual project structure only as each phase needs it;
5. use real validation evidence before advancing to the next phase.

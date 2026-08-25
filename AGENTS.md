# Character Relay Local — Agent Instructions

This repository is the device-side companion to `wong001110/character-relay`.

Current repository status: **planning / Phase 0 contract freeze**.

Do not turn roadmap text into claimed implementation. Do not create speculative package scaffolding, empty modules, fake test success, or placeholder runtime behavior unless the owner explicitly starts an implementation phase.

## Before changing anything

Read in this order:

1. this file;
2. `README.md`;
3. `docs/phase-0-contract-freeze.md`;
4. `docs/contracts/README.md` and the task-relevant Phase 0 contracts;
5. `ROADMAP.md`;
6. the current `wong001110/character-relay` repository `AGENTS.md` and task-relevant current contracts/source/tests;
7. the active branch plan when implementation has begun and that plan explicitly names the current branch.

Chat history and roadmap prose are intent/evidence, not proof of current source behavior.

## Cross-repository authority

`character-relay` owns cloud/product authority, including Character Card, Deployment, Presence, durable memory/evidence, owner authorization, Portal behavior, Device access policy, cloud-side autonomy admission, and final durable interpretation of verified Local outcomes.

`character-relay-local` owns device-side execution, including local connection/runtime, capture, input, streaming, Plugin SDK/Host, Adapter Lab, Local safety enforcement, local Runtime Agent providers, Coding Agent integrations, and the versioned Device Protocol source.

Do not duplicate a Cloud ↔ Local protocol contract independently in both repositories. The accepted v1 direction uses one TypeSpec source under `character-relay-local`, generated JSON Schema/conformance artifacts, and consumers in Local TypeScript and Character Relay Cloud Python/Pydantic.

## Critical invariants

- Local device connections are outbound by default. Do not require public inbound ports for normal operation.
- Do not expose a local MCP server directly to the public Internet.
- MCP is for tools/capabilities; WebRTC is for live media; WSS/HTTPS is for cloud control/telemetry.
- Device Protocol is standard versioned JSON over WSS; do not make Socket.IO protocol a platform dependency.
- Character lived state remains scoped according to the current Character Relay Deployment/runtime contracts. Do not create Character-global consciousness through Local session data.
- Devices are owner-scoped; execution authorization resolves to concrete Deployment/session scope.
- Model/agent intent never outranks owner, device, autonomy, plugin, risk, lease, human-presence, or Local safety permissions.
- Gaming is rich activity under availability Presence (`busy` + `activity.kind=gaming` direction), not a new already-implemented Presence enum.
- Persist only verified local outcomes as lived evidence. Do not manufacture game/activity completion from model narration, SHADOW intents, REVIEW proposals, deferred desires, or adapter availability.
- Local verified outcomes do not directly mutate Character personality/preferences. Owner-configured farming priorities are policy, not proof of Character preference.
- Autonomy cognition receives bounded semantic context; do not make raw desktop/media, secrets, unrelated files/windows, low-level plugin internals, cross-Deployment private context, or private model chain-of-thought default autonomy inputs.
- Plugins/adapters must not receive unrestricted desktop authority merely because they can be called by MCP.
- V1 Official and Developer plugins are trusted executable code; do not describe Host Capability permissions as a complete OS sandbox. Verified/untrusted public plugins are deferred until a real sandbox exists.
- Never implement anti-cheat bypass, process-memory injection/read for cheating automation, packet interception/manipulation, credential extraction, or game-client tampering as Character Relay platform behavior.
- Never persist secrets, provider credentials, login state, unrelated private desktop content, or private model chain-of-thought in traces/fixtures.

## Autonomy and shared-device authority

Read `docs/contracts/autonomy-policy-v1.md` and `docs/contracts/autonomy-context-v1.md` before changing autonomous execution.

Canonical Local action origins:

```text
user_request
user_delegated
session_recovery
character_initiated
```

Resource conflict priority:

```text
physical human / local owner
  > Local safety
  > user_request
  > user_delegated
  > session_recovery
  > character_initiated
```

Runtime/Policy rules:

- `OFF`, `SHADOW`, `REVIEW`, and `AUTO` are distinct autonomy modes.
- Sleep is a hard gate for Character-initiated activity.
- Activity `allowed` windows are hard admission windows; `preferred` windows are soft opportunity preferences.
- Device availability modes are `autonomy_allowed`, `explicit_only`, and `do_not_use`; the Local owner can always narrow authority.
- Physical human interaction owns interactive desktop resources and immediately disarms conflicting automation.
- Character-initiated intents expire; do not create permanent offline/busy queues.
- Resource arbitration is deterministic, not LLM-decided.
- A competing valid intent is normally deferred rather than silently stealing an exclusive lease.
- Higher-priority work uses cooperative preemption at adapter-declared safe checkpoints where possible.
- Human Stop/Take Over and Local safety may interrupt immediately and may create autonomy suppression.
- R3 sensitive/irreversible actions are never ordinary AUTO in v1.
- Character cognition returns high-level intent only; it does not allocate devices/leases or plan raw input.

## Routine game automation boundary

Read `docs/contracts/routine-game-automation-v1.md` before implementing game control.

Character Relay Gaming v1 is **routine/maintenance/repeatable farming automation**, not broad autonomous gameplay.

Potentially automatable when owner-approved and reliably classifiable:

- bounded daily/routine maintenance;
- routine reward claiming;
- ordinary stamina/energy spending on configured targets;
- repeatable relic/artifact/material farming;
- replaying known repeatable encounters;
- returning to a known safe state.

Human-only in v1:

- story/campaign/character narrative progression;
- limited/new events and event story/gameplay;
- first-time experiential content;
- exploration, puzzles, treasure/chests, novel secrets;
- meaningful dialogue choices;
- gacha/premium currency, purchases, destructive/account/security actions.

Unknown or novel content fails closed with a human-required outcome.

Do not use closed commercial games as the first proof of the control architecture. Prove capture/input/leases/skills/cancellation in the owned/reference test environment first.

After that proof, the first intended commercial adapters are:

- **Honkai: Star Rail** — menu/state-heavy, turn-based/repeatable routine validation;
- **Genshin Impact** — real-time 3D movement/visual-control validation for already-approved repeatable routines.

Both must use the same Plugin/Host contracts and remain routine/farming-only in v1.

## Core vs Plugin rule

Use this test:

> If the target game/application were replaced with another environment, would this code still make sense?

If yes, it likely belongs in Local Core (capture, input, process detection, streaming, permissions, plugin lifecycle).

If no, it likely belongs in a Plugin/Adapter (screen knowledge, environment rules, high-level skills, environment-specific recovery).

Terminology:

- **Plugin** — installable/distributable package.
- **Adapter** — environment integration inside a plugin.

## Runtime Agent vs Coding Agent

Keep these separate.

**Runtime Agent** executes bounded environment tasks for a Character. It is optional; deterministic skills should not depend on it when unnecessary.

**Coding Agent** edits/repairs adapter code in Developer Mode. Codex, Claude Code, and custom coding agents belong here. They may inspect bounded traces, source, diffs, and Adapter Lab tools, but they must not bypass validation gates or automatically push/merge unreviewed changes.

Do not expose or persist private model chain-of-thought. Record action summaries, tool calls, test evidence, and code diffs.

## Development workflow when implementation is explicitly started

1. Re-read current `character-relay` `main`; do not assume the roadmap still matches current Presence/Portal/runtime types.
2. Start from accepted Phase 0 contracts, not chat memory.
3. Create a feature branch and a branch-local active development plan naming that branch.
4. Implement phase-by-phase, with one coherent ownership boundary per phase.
5. Add tests with each implemented contract.
6. Use Adapter Lab/live evidence for environment integration; source-only reasoning is insufficient proof that a real tool works.
7. Prove the owned/reference control environment before commercial-game control.
8. Advance a phase only after its gate is satisfied.
9. Keep Promote/Accept, Commit, Push, and Merge as separate authority boundaries.

## Validation expectations

When applicable, distinguish:

- L0 Host validation — process/capture/input/audio/streaming;
- L1 Adapter primitive validation;
- L2 Skill validation;
- L3 end-to-end Character session validation.

A coding agent may run and report these gates, but must not mark a gate passed without retained evidence from the relevant test/live run.

For game adapters, L2/L3 evidence must also show that unknown/event/story/first-time content cannot be silently crossed by an approved routine skill.

## Scope discipline

- Prefer the smallest accepted phase over broad refactors.
- Do not build cloud runners, plugin marketplace, multi-OS parity, SFU/LiveKit, generic unrestricted desktop control, or broad autonomous game playthroughs during the Local MVP unless the roadmap is explicitly reprioritized.
- Do not add a closed commercial game as the first architecture proof; use an owned/test/API-friendly environment first.
- Keep first-party adapters plugin-compatible even if they initially live in this monorepo.
- Do not create empty directories only to match the target repository diagram.

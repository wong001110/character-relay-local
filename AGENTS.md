# Character Relay Local — Agent Instructions

This repository is the device-side companion to `wong001110/character-relay`.

Current repository status: **planning / pre-implementation**.

Do not turn roadmap text into claimed implementation. Do not create speculative package scaffolding, empty modules, fake test success, or placeholder runtime behavior unless the owner explicitly starts an implementation phase.

## Before changing anything

Read in this order:

1. this file;
2. `README.md`;
3. `ROADMAP.md`;
4. the current `wong001110/character-relay` repository `AGENTS.md` and task-relevant current contracts/source/tests;
5. the active branch plan when implementation has begun and that plan explicitly names the current branch.

Chat history and roadmap prose are intent/evidence, not proof of current source behavior.

## Cross-repository authority

`character-relay` owns cloud/product authority, including Character Card, Deployment, Presence, durable memory/evidence, owner authorization, and Portal behavior.

`character-relay-local` owns device-side execution, including local connection/runtime, capture, input, streaming, Plugin SDK/Host, Adapter Lab, local Runtime Agent providers, and Coding Agent integrations.

Do not duplicate a Cloud ↔ Local protocol contract independently in both repositories. When implementation begins, establish one versioned schema authority and generate/import consumers from that authority.

## Critical invariants

- Local device connections are outbound by default. Do not require public inbound ports for normal operation.
- Do not expose a local MCP server directly to the public Internet.
- MCP is for tools/capabilities; WebRTC is for live media; WSS/HTTPS is for cloud control/telemetry.
- Character lived state remains scoped according to the current Character Relay Deployment/runtime contracts. Do not create Character-global consciousness through Local session data.
- Model/agent intent never outranks owner, device, plugin, or session permissions.
- Do not treat `gaming` as an already-implemented Presence enum until the main repository contract/source explicitly says so.
- Persist only verified local outcomes as lived evidence. Do not manufacture game/activity completion from model narration.
- Plugins/adapters must not receive unrestricted desktop authority merely because they can be called by MCP.
- Never implement anti-cheat bypass, process-memory injection, packet interception, credential extraction, or game-client tampering as Character Relay platform behavior.
- Never persist secrets, provider credentials, login state, or unrelated private desktop content in traces/fixtures.

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
2. Start with `ROADMAP.md` Phase 0 contract freeze.
3. Create a feature branch and a branch-local active development plan naming that branch.
4. Implement phase-by-phase, with one coherent ownership boundary per phase.
5. Add tests with each implemented contract.
6. Use Adapter Lab/live evidence for environment integration; source-only reasoning is insufficient proof that a real tool works.
7. Advance a phase only after its gate is satisfied.
8. Keep Promote/Accept, Commit, Push, and Merge as separate authority boundaries.

## Validation expectations

When applicable, distinguish:

- L0 Host validation — process/capture/input/audio/streaming;
- L1 Adapter primitive validation;
- L2 Skill validation;
- L3 end-to-end Character session validation.

A coding agent may run and report these gates, but must not mark a gate passed without retained evidence from the relevant test/live run.

## Scope discipline

- Prefer the smallest accepted phase over broad refactors.
- Do not build cloud runners, plugin marketplace, multi-OS parity, or generic unrestricted desktop control during the Local MVP unless the roadmap is explicitly reprioritized.
- Do not add a closed commercial game as the first architecture proof; use an owned/test/API-friendly environment first.
- Keep first-party adapters plugin-compatible even if they initially live in this monorepo.
- Do not create empty directories only to match the target repository diagram.

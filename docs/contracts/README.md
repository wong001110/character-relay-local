# Character Relay Local Contracts

Status: **Phase 0 authority draft**

These documents define intended Local runtime boundaries before implementation. They are stronger than roadmap prose for the concerns they explicitly freeze, but they are not proof that runtime code exists.

Read in this order when implementing Local:

1. [`../phase-0-contract-freeze.md`](../phase-0-contract-freeze.md) — accepted architecture decisions and Phase 0 gate.
2. [`device-protocol-v1.md`](device-protocol-v1.md) — Cloud ↔ Local control/telemetry contract.
3. [`autonomy-policy-v1.md`](autonomy-policy-v1.md) — autonomy modes, Activity Rhythm, deterministic admission, resource scheduling, and contention policy.
4. [`execution-session-v1.md`](execution-session-v1.md) — Character session, resource lease, disconnect, and takeover semantics.
5. [`plugin-permission-v1.md`](plugin-permission-v1.md) — Plugin/Adapter capability, permission, trust, and sandbox boundary.
6. current `character-relay` source/contracts for owner, Deployment, Presence, Portal, and durable Character evidence semantics.

## Authority split

`character-relay` remains authoritative for cloud product/domain meaning. These Local contracts must not redefine owner/Deployment identity or long-term Character scope.

`character-relay-local` is authoritative for the device transport, autonomy/resource scheduling semantics, execution lease/session semantics, and Local Plugin Host permission boundary once the contracts are accepted.

When machine-readable schemas are introduced, they must be generated/maintained from one versioned protocol authority rather than hand-maintained divergent copies across repositories.

## Change rule

A later implementation may refine field names, timing constants, library choices, scoring weights, or internal class layout without reopening Phase 0, provided it preserves these authority/security semantics.

Changes that alter any of the following require an explicit contract revision:

- Cloud vs Local authority;
- protocol compatibility/idempotency semantics;
- autonomy modes, intent-origin priority, or who owns admission/resource arbitration;
- sleep vs Activity Rhythm hard/soft semantics;
- session/lease ownership, cooperative preemption, or human-stop priority;
- whether raw local data is uploaded by default;
- Plugin Host permission or sandbox claims;
- MCP vs WSS vs WebRTC transport responsibilities;
- Gaming/other activity projection into Character Relay Presence.

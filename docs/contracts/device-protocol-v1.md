# Character Relay Device Protocol v1

Status: **Phase 0 contract draft**

This contract defines the Cloud ↔ Local control and telemetry boundary. It does not define MCP, WebRTC media payloads, Character memory semantics, or plugin-internal protocols.

## 1. Transport split

```text
Pairing / bootstrap / token refresh   HTTPS
Steady-state control + telemetry      WSS
Local plugin tools                    MCP over process-local transport
Live media                            WebRTC
```

The Local client initiates outbound connections. Normal operation must not require an inbound public port.

## 2. Versioning

Every wire message carries a protocol identifier:

```json
{
  "protocol": "cr-device/1"
}
```

Breaking wire changes require a new major protocol identifier. Additive optional fields may be introduced within a major version only when older peers can safely ignore them.

Handshake must exchange:

- Local client version;
- supported Device Protocol major versions;
- device capability summary;
- cloud-selected protocol version.

An incompatible peer fails closed with an explicit version-mismatch reason. It must not guess field semantics.

## 3. Envelope

Conceptual envelope:

```json
{
  "protocol": "cr-device/1",
  "message_id": "msg_...",
  "kind": "command",
  "type": "session.start",
  "sent_at": "2026-08-25T15:00:00Z",
  "device_id": "dev_...",
  "deployment_id": "dep_...",
  "session_id": null,
  "operation_id": "op_...",
  "reply_to": null,
  "payload": {}
}
```

Required semantics:

- `message_id` uniquely identifies one wire message.
- `kind` is one of `command`, `event`, `ack`, or `error`.
- `type` is a versioned semantic event/command name.
- `sent_at` is an RFC 3339 UTC instant.
- `device_id` is the cloud-issued device identity after pairing.
- `deployment_id` is included only when the message is bound to an authorized Deployment.
- `session_id` is included only when an execution/activity session exists.
- `operation_id` is required for side-effecting commands and remains stable across retries/reconnect.
- `reply_to` identifies the message being acknowledged or rejected.

Payloads are schema-validated. Unknown required semantics fail closed; unknown optional fields may be ignored within the negotiated major version.

## 4. Acknowledgement vs completion

An `ack` means the receiving peer accepted or rejected the message envelope/command for processing. It is **not** proof that a long-running action completed.

Example acknowledgement:

```json
{
  "kind": "ack",
  "type": "command.ack",
  "reply_to": "msg_123",
  "operation_id": "op_123",
  "payload": {
    "status": "accepted"
  }
}
```

Execution outcome is emitted separately as structured session/operation events, for example:

```text
operation.started
operation.progress
operation.completed
operation.failed
operation.cancelled
```

Only a verified completion event may become durable Character lived evidence.

## 5. Idempotency

Side-effecting commands use `operation_id` as the idempotency key.

Local must retain enough bounded recent operation state to avoid executing the same accepted side effect twice after a retry or reconnect. A repeated `operation_id` returns/re-emits the known result or active state rather than starting a second operation.

The protocol does not promise global exactly-once message delivery. It provides idempotent side-effect semantics at the operation boundary.

## 6. Heartbeat and liveness

Steady-state WSS includes explicit device heartbeat/liveness events. Exact timing values are implementation configuration, not wire authority.

Cloud may mark a device unavailable after missed liveness according to server policy. Local must not infer that a Deployment is deleted or a Character identity changed from a connection loss.

## 7. Reconnect and resume

Reconnect is automatic with bounded backoff.

After authentication, Local sends a resume snapshot containing at minimum:

- device identity;
- active local session IDs;
- current session states;
- held Execution Leases;
- active operation IDs and their last known state;
- Local client/protocol version.

Cloud reconciles this snapshot against authoritative owner/Deployment/session state and returns explicit continuation, cancellation, or stop instructions.

Local must not independently restart a completed/cancelled cloud session merely because it has stale local state.

## 8. Disconnect behavior

When the control connection is lost:

- no new cloud-originated side-effecting action can start;
- Local safety policy remains authoritative;
- an already-running bounded action may finish only its current atomic step and move to a safe checkpoint when the adapter declares that behavior safe;
- new skill/operation transitions require a valid session policy;
- human Stop/Take Over remains available locally;
- session/lease state is retained only for the bounded resume window defined by implementation policy.

If reconciliation does not authorize resume, Local stops the session and releases leases safely.

## 9. Pairing and device credential boundary

Pairing is owner-initiated and uses HTTPS rather than an unauthenticated public Local listener.

Required properties:

- one-time or short-lived pairing authorization;
- cloud-issued stable `device_id`;
- renewable device credential stored in OS-backed secure storage;
- short-lived session/access material for API/WSS use;
- explicit unpair/revoke path;
- revocation prevents future WSS authentication.

Exact cryptographic/key-storage implementation is deferred to Phase 1, provided these properties hold.

## 10. Authorization

A valid device connection does not grant unrestricted Character or Deployment authority.

Every side-effecting session/command is checked against:

```text
owner/account authorization
+ device authorization
+ Deployment binding
+ session policy
+ plugin capability/permission
+ Execution Lease
+ local safety policy
```

A model-generated command cannot widen any of these scopes.

## 11. Initial semantic families

Names are conceptual until executable schemas are created, but v1 must cover these families without mixing media payloads into the channel:

```text
device.hello
device.ready
device.heartbeat
device.resume
device.capabilities

session.start
session.accepted
session.state
session.stop
session.cancel
session.result

operation.start
operation.progress
operation.result
operation.cancel

activity.started
activity.updated
activity.ended

stream.request
stream.signaling
stream.state

command.ack
protocol.error
```

The executable schema phase may refine names, but semantic responsibilities must remain separated.

## 12. WebRTC signaling

WebRTC offer/answer/ICE signaling may be carried as bounded signaling payloads through the Device Protocol. Encoded video/audio frames never are.

The Portal receives stream authorization from Cloud. Live viewing alone never grants input/control permission.

## 13. Schema source of truth

When implementation begins, machine-readable Device Protocol schemas live under one versioned authority in `character-relay-local` (planned protocol package/artifacts). Character Relay Cloud consumes the same versioned schema/artifact and must not maintain a semantically divergent duplicate.

Cloud-domain IDs retain their meaning from the main `character-relay` contracts; this protocol only transports them.

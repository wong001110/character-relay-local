# Execution Session and Lease Contract v1

Status: **Phase 0 contract draft**

This contract defines who may use a local execution surface, how a Character session progresses, and how local safety behaves across contention, cancellation, disconnect, and human takeover.

## 1. Identities

A local execution session is bound to explicit identities:

```text
owner_id
+ deployment_id
+ character_card_id (reference only; Deployment remains the lived runtime scope)
+ device_id
+ session_id
+ adapter/plugin identity
```

Local never invents or widens owner/Deployment identity. A device may serve multiple Deployments/Characters at the same time when their resources do not conflict.

## 2. Execution Surface

An **Execution Surface** is a local resource against which observation or side effects occur.

Examples:

```text
desktop-input
window:<ephemeral-window-id>
process:<ephemeral-process-id>
game-session:<adapter-scoped-id>
headless-environment:<adapter-scoped-id>
```

OS handles/PIDs are local ephemeral identifiers and are never durable Character identity.

## 3. Lease model

Every side-effecting local capability that can conflict uses an **Execution Lease**.

Lease modes:

- `exclusive` — one session at a time;
- `shared-read` — multiple authorized observers may coexist without side effects.

Examples:

```text
desktop keyboard/pointer     exclusive
target window input          exclusive
target window capture        shared-read when policy permits
live Portal viewing          shared-read media subscription only
```

A session cannot call a Host Capability merely because the plugin exposes an MCP tool. The required lease must exist and match the authorized target.

## 4. Lease ownership and authority

Authority order:

1. explicit human Stop / Take Over;
2. Local safety/permission policy;
3. cloud-authorized session/autonomy policy;
4. Runtime Agent / deterministic skill intent;
5. plugin implementation request.

A lower layer cannot override a higher denial.

Leases are never silently stolen by another Character. Resource contention is arbitrated by the deterministic Resource Scheduler defined in [`autonomy-policy-v1.md`](autonomy-policy-v1.md).

A valid competing intent may be:

- admitted immediately when required resources are free;
- deferred while its validity window remains open;
- expired when its validity window closes;
- cooperatively preempted at a safe checkpoint when a higher-priority accepted intent requires the same exclusive resource;
- denied when policy, permission, risk, or safety does not permit execution.

Contention is not itself proof that an intent is forbidden. Likewise, priority does not grant permission that the intent did not already have.

### 4.1 Canonical origin priority for conflicting resources

When two otherwise-valid intents require the same exclusive resource, scheduler priority is:

```text
user_request
  > user_delegated (within its explicit valid window)
  > session_recovery
  > character_initiated
```

Within one priority class, implementation may consider age, explicit deadline, and deterministic fairness/recent-device-usage weighting. LLM output is not the arbitration authority.

### 4.2 Cooperative preemption

Higher priority does not normally mean arbitrary hard interruption.

If the current session cannot safely release an exclusive lease immediately, Runtime requests preemption and waits for the adapter/skill to report the next safe checkpoint. At that checkpoint the current session pauses or stops, releases the resource, and the higher-priority session may acquire it.

Immediate hard interruption is reserved for explicit human Stop/Take Over, Local safety/security enforcement, permission revocation, or equivalent emergency conditions.

Adapters that contain non-trivial atomic operations should expose bounded preemptibility/safe-checkpoint information rather than forcing the scheduler to infer it.

## 5. Session state machine

Canonical conceptual states:

```text
requested
  -> accepted
  -> preparing
  -> running
  <-> paused
  -> stopping
  -> completed | failed | cancelled | interrupted
```

Semantics:

- `requested` — cloud intent exists but Local has not accepted it.
- `accepted` — Local validated device/session/plugin policy and reserved required startup resources.
- `preparing` — target/plugin/runtime is being prepared; no claim of gameplay/task success.
- `running` — the authorized session may execute bounded operations.
- `paused` — leases may be retained according to policy, but no new side-effecting operation starts.
- `stopping` — safe teardown is in progress.
- terminal states release execution leases and stop side effects.

A deferred autonomy/delegated intent is **not yet an Execution Session** merely because it is waiting for a lease. The Cloud scheduler may keep it as a bounded pending intent until admitted or expired.

Terminal states are immutable for that `session_id`.

## 6. Operation boundary

A Session contains one or more bounded Operations/Skills. Each side-effecting operation has a stable `operation_id` and explicit outcome.

```text
session
  -> operation A
  -> operation B
  -> ...
```

The adapter/runtime must distinguish:

- accepted/requested intent;
- action in progress;
- observed/verified completion;
- inferred or model-narrated completion.

Only observed/verified completion can be emitted as a successful operation result.

For preemptible workflows, an operation/skill may additionally report whether it can pause/stop safely now and the next known safe checkpoint. This is execution evidence, not model reasoning.

## 7. Human takeover

The Local GUI must always provide an accessible local emergency path once input/control exists:

```text
Pause
Stop
Take Over
```

Human takeover:

- prevents new autonomous side effects immediately;
- transitions the session to `paused` or `stopping` according to the chosen action;
- disarms autonomous input authority;
- may create a bounded autonomy suppression under the Cloud Autonomy Policy so the same Character does not instantly recreate the stopped activity;
- cannot be reversed remotely without a new local/session authorization boundary.

A Portal live viewer is read-only by default. Remote takeover/control is a separate future permission and is not part of v1.

## 8. Disconnect policy

Loss of Cloud WSS does not make Local free-running.

On disconnect:

- do not start a new session;
- do not start a new cloud-originated operation;
- retain local safety and human controls;
- a running adapter may finish only the current explicitly bounded atomic step when the adapter declares that step safe to finish offline;
- otherwise pause/stop at the nearest safe checkpoint;
- retain session + lease snapshot for bounded reconciliation;
- release leases if the resume window expires or Cloud rejects reconciliation.

There is no default "keep playing indefinitely while offline" behavior.

## 9. Resume policy

Reconnect does not automatically mean resume.

Local reports active sessions, operations, and leases. Cloud reconciles Deployment/session authority. Resume occurs only when:

- owner/device authorization is still valid;
- Deployment binding is still valid;
- original session/recovery authority is still valid;
- plugin/version is still compatible;
- required permissions are still granted;
- required leases are still valid/reacquirable;
- Local safety policy permits continuation.

A `session_recovery` intent competes for resources under the Autonomy Policy scheduler. Reconnection does not silently preempt a higher-priority explicit user request.

Otherwise the session stops safely.

## 10. Target focus and desktop input

Generic desktop input is a high-risk exclusive resource.

For target-bound keyboard/pointer control:

- the session must hold the relevant exclusive input lease;
- the Host must verify the active/foreground target where the OS API permits reliable verification;
- unexpected focus/target loss disarms or pauses input rather than sending events to an unrelated window;
- Adapter Lab manual input requires explicit time-bounded arming;
- Runtime automation additionally requires plugin permission + session policy + lease.

## 11. Presence/activity projection

Local session state does not directly mutate Character Relay durable Presence on its own.

Local emits verified activity/session events; Cloud remains authoritative for the persisted Deployment Presence projection.

Accepted Phase 0 direction:

```text
availability Presence state: busy
activity kind: gaming
activity target: game/application
activity session_id: verified Local session
```

Ending/interruption of the Local session must produce a cleanup event so Portal cannot remain stuck on Gaming.

A pending/deferred Character intent does not make Presence `busy` and must not be presented as actual gameplay.

## 12. Evidence boundary

A session result may include bounded structured evidence such as:

- target/game identity;
- start/end timestamps;
- verified skill names and outcomes;
- progress counters;
- interruption/failure reason codes;
- sanitized adapter trace references.

Do not promote raw desktop frames, credentials, unrelated window content, model reasoning, shadow intents, review proposals, or deferred/expired desires into durable Character evidence by default.

## 13. Concurrency examples

Allowed when resources do not conflict:

```text
Ann  -> Local game window / desktop input lease
Ning -> headless Minecraft adapter without desktop-input lease
Zhi  -> ComfyUI API adapter
```

Contended:

```text
Ann  -> exclusive desktop-input
Ning -> exclusive desktop-input
```

The second otherwise-valid intent is deferred/expired/preempted according to Autonomy Policy and safe-checkpoint rules rather than silently stealing control.

A higher-priority explicit user request may cause cooperative preemption of a lower-priority autonomous session at a safe checkpoint. Human Stop/Take Over may interrupt immediately under local safety semantics.

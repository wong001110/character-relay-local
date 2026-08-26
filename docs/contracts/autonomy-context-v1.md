# Autonomy Context Contract v1

Status: **Phase 0 contract draft**

This contract defines the bounded context a Character may receive when an Autonomy Opportunity exists, the shape of the high-level intent it may return, and how verified Local outcomes may later contribute to Character Relay memory/evidence without turning device telemetry into fabricated personality.

## 1. Core boundary

Autonomy cognition is not a raw desktop agent.

```text
Runtime creates an Autonomy Opportunity
  -> Cloud builds bounded Autonomy Context
  -> Character cognition chooses a high-level intent
  -> deterministic policy/scheduler admits or rejects it
  -> Local executes an authorized session
  -> verified outcome returns to Cloud evidence/memory pipeline
```

Character cognition decides **what it would like to do**. It does not decide device allocation, permission, lease arbitration, plugin selection, low-level tool plans, or input coordinates.

## 2. Scope

Autonomy Context is Deployment-scoped.

It may use current Character Relay memory/evidence only through the then-current cloud retrieval/runtime contracts for the authorized Deployment. It must not create cross-Deployment lived state or import another Character's private memory merely because the same owner controls both.

Local provides semantic device/activity facts; Cloud owns Character/domain context assembly.

## 3. Allowed context families

The bounded pack may include relevant information from these families.

### Current Deployment state

- current time and authoritative Deployment/server timezone;
- awake/sleeping and current availability Presence;
- current rich activity, if any;
- Activity Rhythm allowed/preferred windows;
- autonomy mode, suppression/cooldown state;
- broad session/daily activity budget remaining.

### Relevant motivations and continuity

- relevant goals/interests already represented by current Character Relay contracts;
- unresolved intentions/commitments;
- relevant beliefs/memory/evidence retrieved by existing cloud logic;
- relevant Discovery/curiosity candidates where current Character Relay semantics permit them;
- recent verified activity summaries.

Retrieval must be relevance-bounded rather than dumping the complete memory store into every autonomy decision.

### Recent social/conversation context

- bounded recent conversation summary;
- relevant recent topics;
- explicit user requests, preferences, or commitments relevant to the candidate activity;
- unresolved promises/tasks that the current Deployment is authorized to know.

Unrelated channels/conversations are not included merely to increase context volume.

### Semantic environment availability

Character cognition may receive product-level availability such as:

```text
available activities: gaming
eligible targets: Honkai: Star Rail, Genshin Impact
gaming device: available
broad constraints: <= 45 minutes
```

It should not receive OS/process implementation details merely to choose a preference.

### Recent verified lived activity

Examples:

```text
yesterday: completed one configured game routine
today: no game routine completed yet
recently: farmed configured repeatable resource target
```

These summaries come from verified session outcomes, not model narration.

## 4. Excluded by default

Autonomy Context must not include by default:

- raw desktop/video/audio frames;
- credentials, tokens, login state, or secrets;
- unrelated files or window contents;
- low-level PID/HWND/process internals;
- raw plugin traces or developer fixtures;
- another Deployment's private conversation/memory;
- another Character's private memory;
- private model chain-of-thought;
- unrestricted host filesystem/network state.

A separately authorized perception task may use a scoped frame through its own capability/policy; that does not make raw desktop media a default autonomy input.

## 5. Character intent output boundary

Autonomy cognition returns bounded high-level preference, conceptually:

```json
{
  "activity": "gaming",
  "target_preference": "honkai-star-rail",
  "goal": "complete an allowed routine and spend configured stamina",
  "duration_preference_minutes": 30,
  "alternatives": ["genshin-impact", "remain_idle"],
  "reason_summary": "preferred gaming window and today's routine is unfinished"
}
```

Field names are conceptual until executable cloud schemas exist.

The intent must not contain or control:

- concrete `device_id` allocation unless a user explicitly targeted a device;
- Execution Lease decisions;
- plugin binary/provider selection;
- MCP tool sequence;
- mouse coordinates or per-frame key input;
- permission/risk overrides.

`reason_summary` is a concise user-visible decision summary, not private reasoning.

## 6. Alternatives

A Character may express ordered high-level alternatives.

The scheduler does not silently invent a substitute activity. If a primary candidate is unavailable, Runtime may evaluate an already-expressed alternative or return semantic unavailability for a later cognition step.

Every alternative still passes the same autonomy, risk, device, plugin, and lease admission gates.

## 7. Verified outcome -> durable evidence

Local does not write Character memory directly.

Local emits bounded verified session outcomes. Character Relay Cloud decides whether and how those outcomes enter the existing evidence/memory pipeline.

Good durable summaries are semantic and bounded, for example:

```text
completed configured daily routine
spent configured stamina/resin on approved repeatable farming target
routine was interrupted by human takeover
```

Do not persist a full click/key/vision trace as ordinary lived memory.

Raw traces may exist separately as sanitized developer/diagnostic artifacts under their own retention rules.

## 8. Preference formation is conservative

One successful or repeated farming operation does **not** directly mutate Character personality into a new preference.

In particular:

- owner-configured farming priority is user policy, not proof that the Character likes that target;
- adapter availability is capability, not preference;
- one Character-selected routine is intent evidence, not a durable trait;
- repeated verified Character-initiated choices may become evidence that the existing Character Relay pattern/insight/memory pipeline can consider later;
- any durable preference must be produced through current cloud evidence/consolidation semantics rather than Local setting a `likes_game=true` flag.

This preserves the distinction between **what the owner configured**, **what the Character chose**, and **what the system has enough evidence to treat as a durable pattern**.

## 9. No fabricated experience

These are not proof that the Character performed an activity:

- an Autonomy Opportunity;
- an intent proposal;
- a SHADOW decision;
- a REVIEW proposal;
- a deferred/expired request;
- adapter availability;
- model narration about intended actions.

Only verified Execution Session outcomes may support claims that the Character actually completed a Local routine.

## 10. Relationship to other contracts

```text
Autonomy Policy
  -> determines when context may be built and whether intent may execute

Autonomy Context
  -> bounds what cognition knows and what high-level intent it may return

Execution Session / Lease
  -> owns actual resource/session execution

Routine Game Automation
  -> restricts which game content classes may be automated

Character Relay Cloud memory/evidence contracts
  -> own durable lived evidence and later preference/pattern formation
```

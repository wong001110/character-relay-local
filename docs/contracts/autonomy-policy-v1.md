# Autonomy Policy v1

Status: **Phase 0 contract draft**

This contract defines when a Character may propose use of a Local device, how Runtime admits or rejects that proposal, and how multiple Characters share finite device resources. It does not give a model authority to bypass owner, Deployment, device, plugin, skill, risk, lease, human-presence, or Local safety policy.

## 1. Core authority rule

Autonomy is split into three distinct decisions:

```text
Runtime decides WHEN an autonomous opportunity may exist
Character decides WHAT it wants to do
Policy decides WHETHER that intent may execute
```

The Local runtime executes only a cloud-authorized session after deterministic admission. A Character model never grants itself device authority.

## 2. Intent origins

Every Local action intent has one origin:

- `user_request` — the user explicitly asks for the action now;
- `user_delegated` — the user previously delegated/scheduled an action within an explicit execution window;
- `character_initiated` — the Character independently proposes an activity;
- `session_recovery` — reconciliation/resumption of an already-authorized session after interruption.

Character Autonomy Mode primarily controls `character_initiated` intents. Turning Character autonomy off does not prohibit explicit `user_request` or valid `user_delegated` work.

`session_recovery` is not a fresh autonomous desire. It may only continue authority that already existed and still passes reconciliation.

## 3. Autonomy modes

Deployment-scoped Local autonomy uses four modes:

### `off`

The Character does not independently propose Local-device activity for execution.

### `shadow`

The Character may produce an intent for observability/evaluation, but Runtime never starts a Local session from it. Shadow records must be clearly marked non-executed and must not become lived-experience evidence.

### `review`

A valid Character-initiated intent becomes a bounded approval proposal. The owner may approve once, deny, or change a durable policy separately. Approval proposals expire and cannot remain indefinitely executable.

### `auto`

A valid Character-initiated intent may start without per-session approval only when every deterministic admission gate passes. `auto` is pre-authorized bounded autonomy, not unrestricted desktop authority.

Default product posture for newly enabled Local autonomy is conservative; exact UX defaults are owned by the cloud product implementation, but no migration may silently broaden an existing user's authority.

## 4. Policy layering

Effective permission is the intersection of:

```text
Owner global policy
  -> Device access/availability policy
  -> Deployment autonomy policy
  -> Activity policy
  -> Plugin permission
  -> Skill/risk policy
  -> Session policy
  -> Execution Lease
  -> Human activity gate
  -> Local safety policy
```

A denial at any higher layer cannot be overridden by a Character, Runtime Agent, Cloud Agent, plugin, or MCP tool.

The primary lived-character scope is `deployment_id`. Character Card identity may be used for authoring convenience, but Runtime authorization must resolve to concrete Deployment scope rather than creating Character-global device authority.

## 5. Local Device Availability Modes

Each paired device exposes an owner-controlled Local availability mode:

- `autonomy_allowed` — Character-initiated sessions may be admitted when all other policy gates pass;
- `explicit_only` — new `character_initiated` sessions are blocked, while valid `user_request` / `user_delegated` work may still be considered;
- `do_not_use` — no new Character execution session may start on the device.

The Local GUI is authoritative for this device mode. Cloud, Character models, Runtime Agents, plugins, or remote prompts cannot silently widen it.

Changing from a more restrictive mode to a less restrictive mode requires an owner-visible Local/product action; reconnect or session recovery does not upgrade the mode.

## 6. Human Activity Gate

Interactive desktop resources belong to the physical device owner first.

For `character_initiated` work requiring resources such as `desktop-input`, keyboard, pointer, or foreground-window control, admission requires Local evidence that the interactive device session is currently suitable for autonomous control.

V1 assumptions for interactive Windows control:

- the supported execution target is the current unlocked interactive Windows session;
- Windows lock, sleep, interactive-session switch, target disappearance, or equivalent loss of safe interactive context disarms autonomous input;
- configurable recent physical-human activity may block a new autonomous interactive session;
- exact idle thresholds are product/Local configuration, not hard-coded by this contract.

This human-activity gate applies to resources that can interfere with the owner. It does not automatically block non-conflicting headless/API work. For example, an authorized ComfyUI API job may continue while the owner uses the desktop if it does not require the same exclusive resource and all other policy gates pass.

### Physical human input wins

When Local detects credible physical human keyboard/pointer interaction while an autonomous session holds an interactive input lease:

```text
physical human input
  -> immediately disarm autonomous input
  -> pause/interrupt at Local safety boundary
  -> release or safely unwind interactive lease
  -> report human takeover to Cloud
  -> apply bounded autonomy suppression where policy requires
```

This decision is Local and deterministic. It does not wait for an LLM or remote approval.

A `user_request` has higher priority than another Character intent, but it still does not silently steal an actively human-used desktop. Product UX may offer `allow now`, `start when idle`, or `cancel`; the default contract is that physical human/local-owner control remains superior.

## 7. Activity Rhythm

Activity autonomy may use time preferences similar in concept to Sleeping Rhythm, but sleep and activity have different semantics.

### Sleep

Sleep is a **hard availability gate**. A sleeping Deployment cannot start a `character_initiated` Local session.

### Activity windows

Each activity type may define:

- `allowed` windows — hard windows in which Character-initiated execution may occur;
- `preferred` windows — soft preferences that increase the likelihood of an autonomy opportunity but do not themselves authorize execution.

Conceptual example:

```yaml
activities:
  gaming:
    allowed:
      - "18:00-23:00"
    preferred:
      - "20:00-22:30"
```

Meaning:

- outside `allowed`: Character-initiated gaming admission fails;
- inside `allowed` but outside `preferred`: gaming may still be considered;
- inside `preferred`: the Runtime opportunity score may favor gaming more strongly;
- sleep always overrides both.

Activity windows are Deployment policy, not silently self-editable model state. A Character may later propose a preference change, but v1 does not let a model mutate its durable autonomy schedule without an explicit product policy/owner boundary.

Time interpretation uses the authoritative Deployment/server timezone contract from Character Relay, not Local machine timezone guesses.

## 8. Autonomy Opportunity

The system must not poll a Character model continuously with "do you want to act?".

A deterministic Runtime creates bounded **Autonomy Opportunities** from current evidence such as:

- Deployment is awake/available;
- activity is inside its allowed window;
- proximity to preferred window;
- no conflicting active Deployment session according to policy;
- authorized device availability mode;
- device online state and required capabilities;
- human-activity suitability for requested interactive resources;
- recent user interaction/activity policy;
- configured daily/session budget;
- current resource contention;
- suppression/cooldown state.

Only then may Character cognition select a high-level activity/goal.

Device/resource/human availability should influence opportunity generation so obviously unavailable activities are less likely to create futile intents. It must not fabricate a successful action merely because an opportunity existed.

## 9. Intent validity and expiry

Every autonomous or delegated intent is bounded by a validity window.

A Character-initiated intent must not remain permanently queued while a device is offline, human-active, or busy.

Conceptually:

```text
intent created
  -> admitted now
  -> deferred while still valid
  -> expired when validity window ends
```

When an intent expires, Runtime does not later revive it merely because a device becomes available.

A `user_delegated` task may have its own explicit execution window. Missing that window produces a skipped/expired outcome rather than delayed surprise execution.

Exact TTL durations are configuration/product policy, not fixed by this contract.

## 10. Deterministic admission gate

Before creating an Execution Session from an intent, Cloud policy plus Local admission evaluates deterministic gates including at least:

1. owner/account authorization;
2. Deployment validity and current Presence availability;
3. Sleeping hard gate;
4. intent origin and autonomy mode;
5. intent validity/expiry;
6. activity allowed window;
7. activity/category permission;
8. configured daily/session limits;
9. device authorization, availability mode, online state, and required capabilities;
10. human-activity suitability for requested interactive resources;
11. plugin enabled/compatible state;
12. required Host Capability permissions;
13. requested skill allowlist/policy;
14. action risk ceiling;
15. required Execution Surface/Lease availability;
16. human-stop or Local safety suppression;
17. duplicate intent/session protection.

An LLM judge is not the authority for these gates.

Cloud may authorize a candidate session, but Local remains able to reject/start-defer it when current physical human activity, Local device mode, focus, lock state, or safety evidence has changed since the cloud decision.

## 11. Risk classes

Actions are classified generically so autonomy does not depend only on tool names:

- `R0 observe` — read-only observation, detection, capture under authorized policy;
- `R1 reversible` — bounded side effects that are normally easy to undo or abandon;
- `R2 resource_consuming` — consumes ordinary in-environment resources or changes durable environment state;
- `R3 sensitive_irreversible` — sensitive, high-impact, financial, credential, destructive, public-posting, premium-currency, or equivalently irreversible actions.

Policy may allow `auto` up to a configured ceiling for R0–R2.

R3 is **never ordinary AUTO in v1**. It requires explicit review/confirmation or is denied entirely according to product/skill policy.

Risk classification does not weaken the separate prohibition on anti-cheat bypass, credential extraction, client tampering, or other platform non-goals.

## 12. Resource Scheduler

Characters express intents; they do not arbitrate device contention.

A deterministic **Resource Scheduler** owns allocation of conflicting Execution Surfaces and Leases.

Physical human ownership and Local safety are not Character intent classes; they supersede the scheduler. Among otherwise-admissible Character/user intents, canonical origin priority for a conflicting resource is:

```text
user_request
  > user_delegated (within its valid window)
  > session_recovery
  > character_initiated
```

Within the same priority class, Runtime may prefer the older still-valid intent, subject to fairness policy and explicit deadlines.

Fairness may apply a deterministic penalty/weight based on recent resource usage so one autonomous Character cannot monopolize a shared device indefinitely. Fairness never overrides a higher-priority explicit user request and never overrides physical human/local-owner control.

The exact scoring formula is implementation policy; the ordering and no-LLM-arbitration rule are contractual.

## 13. Device selection

A Character normally expresses the desired high-level activity/target, not a low-level claim on a specific machine.

When more than one authorized device can satisfy an intent, deterministic device selection considers:

```text
owner/device authorization
+ Local availability mode
+ online/unlocked/safe interactive state where required
+ target/plugin availability
+ required capabilities
+ current Execution Surface contention
+ activity-specific preferred-device ranking
```

Activity policy may define preferred devices, for example Gaming PC before Laptop. A busy or human-active preferred device does not force the Character model to arbitrate; the scheduler may select another eligible device or defer/expire the intent according to policy.

## 14. Resource model

A device is not one indivisible lease. It exposes Execution Surfaces/resources, for example:

```text
desktop-input              exclusive
target:<game-window>        exclusive for control
screen-capture              shared-read when authorized
live-view subscription      shared-read
headless API adapter        policy-specific
gpu capacity                capacity/policy-specific future resource
```

Therefore multiple Characters may use one physical device concurrently when their required resources do not conflict.

Example:

```text
Ann  -> desktop-input + game window
Ning -> headless server/API adapter
Zhi  -> ComfyUI API/queue
```

This may be valid if every resource/permission policy passes.

Two sessions requiring the same exclusive `desktop-input` cannot run concurrently.

## 15. Contention: defer, do not silently steal

When a valid intent cannot acquire a required exclusive resource, the normal outcome is **deferred due to resource contention**, not silent lease theft.

A deferred intent remains eligible only until its validity window expires and its policy still passes.

Human activity may similarly cause a valid request to wait for device availability, but it does not become a Character-vs-Character contention contest. Human/local-owner interactive control remains superior.

The scheduler may choose not to keep a literal queue entry when the same semantics can be recreated from a bounded pending-intent record; implementation detail is open.

The Character may receive a semantic unavailable/deferred result and choose another activity in a future cognition step. The scheduler itself does not invent a substitute Character intent.

## 16. Cooperative preemption

A higher-priority Character/user intent does not normally hard-kill an existing lower-priority session at an arbitrary unsafe point.

Instead:

```text
higher-priority request
  -> preemption requested
  -> current session reaches adapter-declared safe checkpoint
  -> current session pauses/stops and releases lease
  -> higher-priority session may acquire lease
```

Adapters/skills should expose whether the current step is safely preemptible and the next safe checkpoint where relevant.

Immediate hard interruption remains reserved for:

- physical human takeover / explicit human Stop or Take Over;
- Local safety enforcement;
- security/permission revocation;
- equivalent emergency conditions.

## 17. Human stop and suppression

A human Stop, Take Over, or detected physical takeover is authoritative and must prevent an autonomous Character from immediately recreating the same activity/session.

Runtime records a bounded **autonomy suppression** scoped to the relevant Deployment/activity/device/session policy. The owner may choose a product action equivalent to:

- stop this session;
- suppress this activity for a period/day;
- switch the device to `explicit_only` or `do_not_use`;
- disable autonomous use for this activity/device.

Exact cooldown durations are configurable product policy.

Remote model output cannot clear a human/local suppression or widen the Local availability mode.

## 18. Retry and recovery

Autonomous execution uses bounded recovery. A failed session must not recursively spawn unlimited replacement sessions.

Preferred conceptual recovery chain remains:

```text
deterministic skill
  -> optional Local Runtime Agent
  -> optional Cloud Agent
  -> fail/stop or human intervention
```

If recovery is exhausted, the session fails and policy may apply cooldown/suppression before a new independent Character-initiated opportunity is considered.

`session_recovery` after connection interruption follows the separate Execution Session reconciliation contract and does not bypass risk, permission, expiry, human-presence, device-mode, or lease checks.

## 19. Portal observability

Autonomous action must be visible as autonomous action.

Portal/session observability should expose semantic evidence such as:

- intent origin;
- autonomy mode;
- activity/target;
- selected device;
- policy/admission outcome;
- deferred/expired/human-active reason;
- execution session state;
- current resource/lease contention where useful;
- owner Stop/Take Over controls.

Show decision summaries/reason codes, not private model chain-of-thought.

A `shadow` intent must never be presented as an activity the Character actually performed.

## 20. Evidence boundary

Only verified Execution Session outcomes may become lived Character evidence.

These are not lived evidence by themselves:

- an autonomy opportunity;
- a Character desire/intent;
- a REVIEW proposal;
- a SHADOW decision;
- a deferred or expired request;
- a human-active/device-unavailable admission result;
- model narration about what it intended to do.

## 21. Cross-contract relationship

This policy sits above the Local Execution Session contract:

```text
Autonomy Opportunity
  -> Character/User Intent
  -> Cloud Admission Policy
  -> Device/Human Availability Gate
  -> Resource Scheduler
  -> Execution Session + Lease
  -> Local execution
  -> Verified Result
```

Device Protocol transports accepted commands/events; it does not become the policy authority. Plugin Permission constrains what the accepted session can actually call. Local safety and physical human ownership remain the final device-side denial/takeover boundaries.

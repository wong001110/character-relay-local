# Routine Game Automation Contract v1

Status: **Phase 0 contract draft**

This contract defines the product boundary for game control in Character Relay Local v1.

The goal is **routine maintenance and repeatable farming automation**, not autonomous playthroughs and not replacement of the player's experience.

## 1. Product boundary

Character Relay Gaming v1 may automate bounded, repeatable, low-novelty maintenance work that the owner has explicitly enabled.

Examples of eligible categories:

- configured daily/routine maintenance;
- claiming routine rewards;
- consuming ordinary regenerating stamina/energy on owner-approved targets;
- repeatable material/relic/artifact farming;
- replaying already-known repeatable encounters where the adapter can verify state and completion;
- returning to a known safe state and stopping cleanly.

The existence of an adapter does not make every part of a game automatable.

## 2. Human-only content

V1 treats experiential, novel, first-time, story, and event content as **human-only**.

Do not autonomously execute:

- main story / campaign progression;
- story quests, character quests, hangouts, or equivalent narrative content;
- limited-time/new event gameplay or event story;
- first-time content/challenges when completion would consume the player's first experience;
- exploration intended to discover new areas;
- puzzles, treasure/chest discovery, or novel secrets;
- meaningful dialogue/branching choices;
- account/social/public-posting actions;
- gacha/wishes/warps;
- premium-currency spending;
- purchases or financial actions;
- destructive inventory/account actions;
- account/security/settings changes.

If a routine unexpectedly transitions into human-only/unknown content, the adapter stops or returns to a safe state rather than improvising through it.

Event detection may be surfaced semantically to the owner/Portal, but detection is not permission to enter or complete the event.

## 3. Routine Catalog

Automation operates from an owner-approved **Routine Catalog / Routine Profile**, not unrestricted game goals.

Conceptually:

```yaml
gaming:
  target: genshin-impact
  allowed_routines:
    - claim_safe_routine_rewards
    - spend_resin_on_configured_artifact_domain
  farming_priority:
    - configured_artifact_domain_a
    - configured_material_target_b
  human_only:
    - story
    - events
    - exploration
    - gacha
```

The exact schema is implementation detail, but the authority boundary is contractual:

- owner policy defines what routine classes/targets are allowed;
- Character cognition may choose among allowed high-level routines when autonomy permits;
- adapter/runtime chooses the deterministic execution plan;
- unknown/novel content fails closed.

Owner-configured farming priority is not evidence that the Character personally prefers that content.

## 4. Risk boundary

Routine farming commonly consumes ordinary in-game resources and is therefore generally at least `R2 resource_consuming` under Autonomy Policy.

AUTO execution requires explicit policy permitting the relevant R2 skill/resource class.

R3 sensitive/irreversible actions remain review-only or denied under Autonomy Policy and are outside normal game AUTO behavior.

Premium currency, purchases, gacha, account changes, and similarly high-impact actions are not part of routine automation v1.

## 5. Commercial-game capability boundary

Commercial games may support different tiers independently:

```text
Presence
Live Watch
Routine Control
```

A game may remain Presence/Live-only if automation is not compatible, permitted, or reliable.

Character Relay must not add anti-cheat bypass, process-memory injection/read for cheating automation, packet interception/manipulation, credential extraction, client tampering, or equivalent circumvention as part of a game adapter.

## 6. Architecture proof vs first game adapters

Closed commercial games are **not** the first proof of the Plugin/Adapter architecture.

Before game-control work, Phase 4/5 use the owned/reference test environment to validate:

- Plugin Host/MCP lifecycle;
- capture/input Host Capabilities;
- permissions and Execution Leases;
- Adapter Lab tooling;
- deterministic skills, cancellation, safe checkpoints, and regression evidence.

After that foundation is proven, the first intended commercial game adapters are:

1. **Honkai: Star Rail**;
2. **Genshin Impact**.

They should share the same Plugin/Host contracts rather than receiving game-specific Core privileges.

## 7. Why both first adapters

The two adapters intentionally test different embodiment shapes while keeping the product scope narrow.

### Honkai: Star Rail

Primary validation emphasis:

- menu/state-heavy navigation;
- deterministic screen transitions;
- turn-based/repeatable combat flow;
- owner-approved stamina consumption and relic/material farming;
- routine reward/maintenance flows where reliably classifiable;
- verified completion and clean stop.

This adapter should favor explicit state machines and known-state recognition rather than unnecessary general-purpose agent control.

### Genshin Impact

Primary validation emphasis:

- real-time 3D environment control;
- target/window focus and movement safety;
- visual navigation to already-approved repeatable routine targets;
- owner-approved Resin consumption and artifact/material farming;
- routine maintenance only when the adapter can classify it as safe/repeatable;
- verified completion, safe checkpoint, and clean stop.

The adapter must not turn its 3D navigation ability into autonomous exploration, event participation, or story progression.

Together they exercise both a more menu/turn-based workflow and a more real-time 3D-control workflow without broadening v1 into a generic game-playing agent.

## 8. Conservative daily-task rule

A feature being called "daily" by a game does not automatically make it safe to automate.

The adapter may automate only daily entries that are classified as routine/repeatable under the owner-approved catalog and that do not cross into novel narrative/event/first-time content.

If the adapter cannot confidently classify a daily task, it must skip/stop and report `human_required` or an equivalent reason rather than guessing.

## 9. Skill granularity

Preferred skills are high-level and bounded, for example:

```text
inspect_routine_status()
claim_allowed_routine_rewards()
spend_stamina_on_configured_target()
run_configured_farming_loop(max_runs=N)
return_to_safe_state()
```

The Cloud Character model should not control per-frame movement or raw mouse/keyboard input.

Where high-frequency movement is needed, it remains inside the Local adapter/controller under an authorized skill/session and lease.

## 10. Safe stop and preemption

Every routine skill must have bounded stop/failure behavior.

Adapters should declare safe checkpoints such as:

- outside an active encounter;
- after result/reward verification;
- at a known menu/safe state;
- before consuming the next unit of stamina/resource.

Human Stop/Take Over and Local safety still outrank routine completion and may interrupt immediately where required.

Higher-priority Character/user work normally waits for or requests cooperative preemption at the next safe checkpoint.

## 11. Verified result boundary

Successful routine outcomes must be objectively verified before being emitted as completed.

Examples of useful semantic results:

```text
routine completed
configured farming target completed N times
ordinary stamina/resource consumed: verified amount/range if reliably observable
reward/result screen observed
stopped because human-only content was encountered
```

Do not claim exact drops, inventory changes, or resource totals unless the adapter can actually observe and validate them reliably.

## 12. Memory boundary

Game routine outcomes feed the Autonomy Context / Character Relay evidence pipeline only as bounded verified summaries.

Completing routine farming does **not** imply the Character experienced story/event content or personally developed a preference for the farm target.

If the owner later talks with the Character about an event/story the owner played, that conversational knowledge is separate from Local execution evidence.

## 13. First adapter acceptance target

For each first commercial adapter, the initial control milestone is deliberately narrow:

```text
detect authorized target
  -> establish known routine state
  -> select one owner-approved routine/farm target
  -> execute bounded repeatable loop
  -> verify progress/result
  -> stop at known safe state
```

Acceptance requires repeated success in Adapter Lab with cancellation, human takeover, focus-loss safety, safe checkpoints, and evidence validation.

Broad "play the game for me" capability is explicitly not a v1 success criterion.

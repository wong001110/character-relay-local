# Plugin Capability and Permission Contract v1

Status: **Phase 0 contract draft**

This contract defines the authority boundary between Local Core and installable Plugins/Adapters.

## 1. Terminology

- **Plugin** — installable/distributable package loaded by Character Relay Local.
- **Adapter** — environment-specific integration inside a Plugin.
- **Host Capability** — generic Local Core ability exposed to an authorized Plugin, such as target capture or input.
- **Permission Grant** — owner-approved authorization for one Plugin identity to use a bounded Host Capability.
- **Trust Level** — distribution/review status; it is not a substitute for runtime permission checks.

## 2. Core vs Plugin ownership

Local Core owns target-independent capabilities:

- process/window discovery;
- capture backends;
- audio capture;
- input backends;
- WebRTC publishing;
- secure credential storage;
- Cloud connection/device identity;
- Execution Lease enforcement;
- Plugin lifecycle and permission enforcement;
- Runtime/Coding Agent provider hosting.

Plugins own target-specific knowledge:

- screen/state recognition;
- game/application rules;
- high-level skills;
- target-specific deterministic state machines;
- target-specific recovery logic;
- optional target-specific CV/OCR/model orchestration;
- safe-checkpoint/preemptibility reporting for long-running skills when needed.

A Plugin does not become the owner of generic OS input/capture just because its adapter needs those capabilities.

## 3. Manifest identity

Every Plugin manifest must declare at minimum:

```text
plugin_id
publisher identity
plugin version
supported Local/plugin API range
adapters/environments
requested Host Capabilities
MCP/tool entry point
```

Plugin identity is stable across versions. Publisher identity changes are treated as a new trust/authorization boundary rather than silently inheriting grants.

## 4. Initial Host Capability families

Names are conceptual until the executable Plugin SDK schema is created.

Read/observe examples:

```text
process.observe
window.enumerate
window.observe
capture.window
capture.audio
```

Side-effect examples:

```text
target.launch
input.keyboard
input.pointer
```

Development capabilities are separately gated:

```text
lab.trace
lab.fixture
lab.reload
lab.test
```

The normal Plugin Host must not expose arbitrary shell, unrestricted filesystem, browser credential access, process-memory read/write, packet interception, or generic network proxy capabilities as implicit conveniences.

## 5. Permission evaluation

A Host Capability call succeeds only when all required boundaries pass:

```text
plugin identity/version compatibility
+ requested manifest capability
+ owner permission grant
+ authorized target/session scope
+ cloud autonomy/risk/session policy when applicable
+ Execution Lease when side-effecting
+ Local safety policy
```

MCP tool availability does not bypass this evaluation. MCP is a discovery/call transport, not an authorization primitive.

## 6. Permission scope

Grants should be as target-specific as the Host can enforce.

Examples:

```text
capture.window -> authorized target window only
input.keyboard -> active target/session only
input.pointer  -> active target/session only
capture.audio  -> approved target/system loopback policy
```

When reliable target restriction is not technically possible, the GUI/manifest must describe the broader real authority instead of presenting a false narrow permission.

## 7. Permission updates

Plugin updates never silently widen authority.

- adding a permission requires new approval;
- widening target/resource scope requires new approval;
- changing publisher identity requires new approval;
- a reduced permission set may continue with the remaining grants;
- Local Core update must not silently promote an existing Plugin grant into a stronger capability.

Permission state is versioned/auditable separately from Plugin code version.

## 8. Input control

Keyboard/pointer input is high-risk.

Runtime input requires:

```text
plugin input permission
+ active authorized Character session
+ exclusive required Execution Lease
+ target/focus safety check
+ Local safety policy
```

Adapter Lab manual input additionally requires an explicit time-bounded **Arm Input** action. Arming expires automatically and is cancelled by Stop/Take Over, target loss, or safety failure.

A Plugin cannot ask the model to override these controls.

A long-running input skill that cannot be interrupted safely at every instant should expose bounded safe-checkpoint/preemptibility state so the Resource Scheduler can cooperatively transfer an exclusive lease when higher-priority accepted work is waiting.

## 9. Trust levels

V1 user-visible trust levels:

### Official

Maintained/released by Character Relay. Official status does not remove permission checks, but code may initially run as trusted first-party process code while the architecture is being proven.

### Developer

Locally loaded development Plugin. The GUI must clearly state that Developer Plugins are user-trusted executable code and may not be OS-sandboxed. They are intended for adapter development/testing, not arbitrary untrusted distribution.

### Verified — deferred

A future third-party distribution level. Do **not** expose Verified as a v1 runtime trust choice and do not advertise arbitrary third-party Plugins as safely permission-sandboxed until a real OS/process sandbox exists and has acceptance tests.

## 10. Honest sandbox boundary

A Node.js subprocess running with the user's normal account can access more OS resources than a Host API permission list unless the process is actually sandboxed.

Therefore v1 makes this distinction explicit:

```text
Host Capability permission enforcement
!=
complete OS sandbox for arbitrary plugin code
```

For the MVP:

- first-party Official Plugins and explicit Developer Plugins are trusted-code execution;
- they should still use Host Capabilities for capture/input/session integration so contracts remain portable;
- untrusted public Plugin distribution is deferred.

Before a public/Verified ecosystem launches, Character Relay Local must add and validate an OS-level sandbox/restricted execution model (for example an appropriate Windows restricted token/AppContainer-style boundary or another proven mechanism). The exact sandbox implementation is deferred; the requirement is not.

## 11. Plugin process isolation

Even trusted Plugins run out-of-process where practical.

Required behavior:

- Local Host starts/stops the Plugin process;
- local MCP transport is process-local/stdio by default;
- Plugin crash does not terminate Cloud WSS/device runtime;
- stdout/stderr/log capture is bounded and secret-redacted;
- Local may kill a hung Plugin;
- Plugin cannot expose its MCP server publicly through Local configuration by default.

## 12. Network and filesystem

The MVP Host Capability API does not provide generic unrestricted network or filesystem access to game/application Plugins.

If a future Adapter legitimately requires such capability, it must be an explicit permission family with a real scope and threat review rather than an informal escape hatch.

Developer/Official trusted process code may technically possess account-level OS access before sandboxing; this is a trust limitation that must be disclosed, not hidden behind the Host API model.

## 13. Commercial game boundary

A Plugin must declare supported capability tiers independently:

```text
Presence
Live Watch
Integration
Visual Control
```

A target may support Presence/Live Watch while control remains disabled or unsupported.

Character Relay platform behavior must not include:

- anti-cheat bypass;
- process-memory injection/read for cheating automation;
- packet interception/manipulation;
- credential extraction;
- game-client tampering.

## 14. Coding Agent boundary

Coding Agents such as Codex or Claude Code do not inherit Plugin runtime authority.

Adapter Lab may grant a Coding Agent a separate bounded developer surface:

```text
read/write assigned adapter source
read sanitized failure traces
invoke selected Lab tools
reload adapter
run targeted/full adapter validation
inspect diff
```

Commit, Push, and Merge remain separate authority boundaries. A Coding Agent cannot use its repo access to bypass Local runtime permission/lease checks during live validation.

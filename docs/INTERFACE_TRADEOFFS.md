# Flight Safety Monitor & Verification Platform

## Phase 2B-0 — Interface Representation Tradeoff Review

**Status:** Reviewed and accepted for Phase 2B-1
**Lifecycle phase:** Phase 2 — Architecture and interface design
**Implementation status:** No C++ implementation authorized by this document

---

## 1. Purpose

This document evaluates the concrete representation choices that were
intentionally deferred by the Phase 0 and Phase 1 baselines.

It exists to prevent implementation convenience from silently deciding:

- modeled-time representation;
- numeric widths;
- sequence-number width and rollover behavior;
- physical numeric representation;
- configuration and evaluation identity representation;
- runtime-event representation;
- optional-state representation;
- evidence types;
- reason-code representation and ordering;
- deterministic-core public API shape;
- serialization boundaries.

This document does not introduce operational aerospace thresholds,
procedures, coordinate systems, or protected operational behavior.

---

## 2. Decision Status Vocabulary

Each item is classified as one of:

### 2.1 RECOMMENDED

The existing requirements and architecture constrain the problem strongly
enough that one design is currently preferred.

A `RECOMMENDED` item still requires engineering review before becoming
baselined interface design.

### 2.2 OPEN

More than one design remains technically credible.

The decision shall not be made implicitly during implementation.

### 2.3 DEFERRED

The existing requirements intentionally do not require the behavior or
representation yet.

No implementation shall invent it.

---

## 3. Evaluation Criteria

Representation choices shall be evaluated primarily against:

1. deterministic replay;
2. explicit semantics;
3. exact required boundary behavior;
4. absence of hidden host-time dependencies;
5. prevention of invalid states where practical;
6. traceability to requirements;
7. simple independent verification;
8. minimal mutable state;
9. minimal accidental coupling;
10. suitability for the host-based synthetic verification scope;
11. avoidance of unnecessary frameworks or abstraction;
12. no introduction of operational aerospace behavior.

Performance optimization is secondary unless a later requirement makes it
relevant.

---

# 4. Modeled Time Representation

## 4.1 Baseline constraints

The current baseline requires:

- explicit externally supplied modeled time;
- one synthetic scenario-time domain;
- source, receive, and evaluation time in that domain;
- no decision dependence on host wall-clock time;
- deterministic ordering;
- exact freshness, timeout, and persistence boundary behavior;
- elapsed modeled-time persistence;
- same-time events to remain valid when caller order is explicit.

## 4.2 Candidate A — floating-point seconds

Example conceptual representation:

```text
double seconds
```

### Advantages

- simple projection arithmetic;
- familiar;
- easy to print and inspect.

### Disadvantages

- equality at modeled-time boundaries becomes dependent on binary
  floating-point representation;
- subtraction may produce representation artifacts;
- invites epsilon policies that the current semantic model does not define;
- source, receive, evaluation time, and duration remain weakly typed.

### Assessment

Not preferred.

The monitor has exact time-boundary semantics, making floating-point modeled
time an unnecessary source of ambiguity.

## 4.3 Candidate B — unsigned integral time

Example:

```text
uint64_t ticks
```

### Advantages

- exact ordering;
- exact equality;
- large positive range.

### Disadvantages

- subtraction requires special handling to avoid underflow;
- the semantic model does not require scenario time to be inherently
  unsigned;
- durations and absolute scenario time can be accidentally conflated.

### Assessment

Usable, but weaker than a signed strong representation.

## 4.4 Candidate C — signed strong scenario-time and duration types

Conceptually:

```text
ScenarioTime
ScenarioDuration
```

Both use a signed fixed-width integral representation internally, but are
distinct domain types.

### Advantages

- exact ordering and equality;
- exact elapsed-duration arithmetic;
- prevents accidental interchange of absolute time and duration;
- supports explicit overflow checking;
- does not assume the scenario zero point must be the minimum representable
  time;
- no wall-clock semantics are implied.

### Disadvantages

- requires small value-type definitions;
- projection must explicitly convert a duration into seconds for physical
  arithmetic.

### Status

**RECOMMENDED**

The strong-type approach best matches the current deterministic semantics.

---

# 5. Modeled-Time Unit

The underlying integral unit remains a design decision.

## 5.1 Candidate A — milliseconds

Advantages:

- highly readable;
- very large representable duration.

Disadvantages:

- unnecessarily coarse if later synthetic tests require finer event
  separation.

## 5.2 Candidate B — microseconds

Advantages:

- ample resolution for synthetic verification;
- large range;
- relatively readable.

Disadvantages:

- still an arbitrary resolution decision.

## 5.3 Candidate C — nanoseconds

Advantages:

- conventional high-resolution integral time unit;
- minimizes risk that a later synthetic verification scenario requires a
  finer unit;
- still provides a range vastly larger than required by this project when
  backed by signed 64-bit storage.

Disadvantages:

- precision greatly exceeds current behavioral need;
- projection conversion to seconds must be explicit.

### Status

**RECOMMENDED: nanoseconds**

Use one nanosecond as the repository-wide modeled-time unit.

This is a representation choice, not a claim that the synthetic system has
nanosecond physical accuracy.

With signed 64-bit storage, the available range remains vastly larger than
needed by the host-based synthetic verification scope.

Using the finest conventional integral unit avoids a later interface revision
solely because a synthetic scenario or host-to-target experiment requires
finer event separation.

---

# 6. Modeled-Time Width

## 6.1 Candidate A — signed 32-bit

Rejected.

Its range is unnecessarily restrictive for an otherwise host-based
verification platform.

## 6.2 Candidate B — signed 64-bit

Advantages:

- large range;
- conventional fixed-width representation;
- straightforward overflow analysis;
- deterministic across supported platforms.

### Status

**RECOMMENDED**

Use a signed 64-bit underlying representation for both `ScenarioTime` and
`ScenarioDuration`.

The domain types remain distinct even if their storage type is identical.

---

# 7. Sequence Number Width

## 7.1 Baseline constraints

The current baseline defines:

- a monotonically increasing unsigned logical sequence number;
- equal means duplicate;
- lower means regression/reordering;
- higher means forward progress;
- gaps remain observable;
- scenarios shall not depend on rollover;
- rollover behavior is intentionally deferred.

## 7.2 Candidate A — unsigned 32-bit

Advantages:

- common protocol-sized field;
- more than sufficient for many synthetic scenarios.

Disadvantages:

- reaches rollover materially earlier;
- creates pressure to define rollover semantics that the project does not
  currently need.

## 7.3 Candidate B — unsigned 64-bit

Advantages:

- practically eliminates rollover from initial verification scenarios;
- preserves simple numeric ordering;
- does not require serial-number arithmetic;
- fixed and deterministic across platforms.

Disadvantages:

- wider than necessary for small tests.

### Status

**RECOMMENDED**

Use an unsigned 64-bit logical sequence value.

---

# 8. Sequence Rollover

Possible policies include:

1. serial-number arithmetic;
2. explicit wrap from maximum to zero;
3. execution reset before exhaustion;
4. rollover unsupported.

The existing baseline explicitly states that scenarios shall not depend on
rollover.

### Status

**RECOMMENDED: unsupported in the initial core**

The maximum value shall not create implicit permission to wrap to zero.

If rollover is later required, its ordering rules shall be introduced through
a reviewed requirements change.

This avoids silently importing protocol serial-number semantics into a
synthetic logical counter.

---

# 9. Physical Position and Velocity Representation

## 9.1 Baseline constraints

The initial state model is:

- synthetic local Cartesian;
- position in meters;
- velocity in meters per second;
- three dimensions;
- finite physical values required;
- constant-velocity projection;
- no general-purpose epsilon;
- represented values used directly for envelope boundary semantics.

## 9.2 Candidate A — fixed-point integers

Advantages:

- exact represented arithmetic for chosen units;
- deterministic boundary representation.

Disadvantages:

- requires choosing physical quantization not required by the baseline;
- projection multiplication/division becomes more complicated;
- risks introducing arbitrary scale policy into the domain;
- less natural for varied synthetic test values.

## 9.3 Candidate B — IEEE-754 double precision

Advantages:

- natural representation for Cartesian physical state;
- supports projection arithmetic directly;
- NaN/infinity can be explicitly rejected as already required;
- widely understood and testable;
- no additional quantization policy required.

Disadvantages:

- not all decimal values are exactly representable;
- arithmetic may round.

The baseline already avoids a generic epsilon and defines boundary semantics
using represented values directly.

### Status

**RECOMMENDED**

Use binary64 / C++ `double` for Cartesian position, velocity, projection, and
envelope bounds.

Tests should use deliberately chosen values appropriate for exact boundary
verification where exact equality is under test.

---

# 10. Physical Vector Representation

## 10.1 Candidate A — six independent scalar fields everywhere

Advantages:

- explicit.

Disadvantages:

- repetitive;
- easy to swap axis values;
- projection and envelope APIs become noisy.

## 10.2 Candidate B — distinct Cartesian value types

Conceptually:

```text
Position3
Velocity3
```

Each contains x, y, z scalar components.

### Advantages

- preserves unit/domain distinction;
- easy to test;
- avoids treating position and velocity as interchangeable arrays;
- no generic vector-math framework required.

### Status

**RECOMMENDED**

Use small distinct Cartesian value types rather than a general linear-algebra
dependency.

---

# 11. Configuration Identity

The requirements require decisions to identify the configuration that
produced them.

They do not currently prescribe identity encoding.

## 11.1 Candidate A — numeric counter

Advantages:

- small;
- simple.

Disadvantages:

- identity meaning depends on external lifecycle state;
- harder to inspect manually;
- restart/replay identity semantics need additional definition.

## 11.2 Candidate B — opaque stable string value

Advantages:

- scenario/test configurations may assign explicit readable identities;
- deterministic input value;
- no hidden allocation counter;
- easy to include in evidence.

Disadvantages:

- variable-size storage.

## 11.3 Candidate C — content hash

Advantages:

- identity can correspond directly to configuration content.

Disadvantages:

- requires canonical encoding before hashing;
- serialization/canonicalization is currently deferred;
- adds unnecessary cryptographic machinery.

### Status

**RECOMMENDED: opaque caller-supplied stable identity value**

The exact C++ storage type remains open until the public interface is drafted.

Do not introduce hashing solely to create configuration identity.

---

# 12. Execution Identity

Several evidence requirements allow execution/configuration identity when
available.

The exact execution-identity representation has not yet been baselined.

Possible choices include:

- caller-supplied opaque identity;
- internally generated counter;
- no separate execution ID beyond configuration and execution boundary.

### Status

**RECOMMENDED: no separate core-generated execution identity**

The initial deterministic core shall not generate an execution identifier.

Initialization and reinitialization establish execution boundaries
structurally, while configuration identity remains explicit in required
decision evidence.

External scenario, verification, or run-management tooling may associate an
execution/run identity with a core instance when needed for artifact
correlation.

Such external identity shall not alter deterministic monitor behavior.

The core shall not create execution identity from randomness, wall-clock
values, process-global counters, or other hidden environmental state.

---

# 13. Evaluation Identity

`DecisionRecord` must provide an evaluation identity directly or by stable
reference.

Possible implementations include:

1. internal monotonically increasing evaluation counter;
2. accepted-runtime-event ordinal;
3. compound identity derived from execution identity plus event ordinal;
4. externally supplied runtime-event reference.

### Tradeoff

An internal counter is simple but creates additional authoritative mutable
state used only for evidence identity.

The architecture intentionally avoided inventing unnecessary persistent
decision-sequence state.

Every accepted runtime event already corresponds to exactly one logical
evaluation and exactly one `DecisionRecord`.

A stable caller-supplied runtime-event reference can therefore identify the
triggering event and, for accepted events, the resulting evaluation without
creating another mutable sequencing mechanism inside the core.

The same reference can also appear in runtime-event rejection evidence when
the event is rejected before normal evaluation.

### Status

**RECOMMENDED: evaluation identity derives from the triggering runtime-event reference**

Each submitted normal runtime event shall carry a stable caller-supplied
`RuntimeEventReference`.

For an accepted runtime event, that reference also identifies the one logical
evaluation produced by the event.

For a rejected runtime event, the same reference identifies the rejected
event in runtime-event rejection evidence.

`RuntimeEventReference` is evidence identity only.

It shall not determine or modify:

- runtime-event ordering;
- event-time acceptance;
- navigation-observation acceptance;
- interface-activity qualification;
- persistence;
- simulated-action behavior.

### Concrete representation

`RuntimeEventReference` shall be a strong domain value backed by an unsigned
64-bit integer.

The caller supplies the value for every submitted normal runtime event.

The reference shall be unique within one initialized execution across both
accepted and rejected runtime events so that normal decisions and rejection
evidence remain unambiguous.

All unsigned 64-bit values are representable; no sentinel value is reserved.

The deterministic core shall preserve the reference verbatim in evidence but
shall not:

- increment it;
- derive it;
- compare it for event ordering;
- use it for event-time acceptance;
- use it for navigation acceptance;
- use it for persistence or simulated-action behavior.

Reference uniqueness is an interface contract on the event source rather than
a new modeled runtime-event rejection rule.

The initial deterministic core therefore does not maintain a historical
seen-reference set solely to validate evidence identifiers.

Scenario, replay, and integration tooling shall ensure reference uniqueness
before or while constructing the ordered event stream.

### Status

**RECOMMENDED: strong caller-supplied unsigned 64-bit runtime-event reference**

The initial core shall not maintain a separate logical decision-sequence
counter solely to manufacture evaluation identity.

---

# 14. Protocol / Version Identity

Navigation observations conceptually carry protocol/version identity.

Phase 2B distinguishes three different version concepts:

1. navigation-observation schema version;
2. deterministic-core software/interface revision;
3. future transport/wire-protocol version.

These identities shall not be conflated.

## 14.1 Navigation-observation schema version

The version carried with the logical navigation observation shall represent
the navigation-observation schema only.

It shall be named conceptually as `NavigationObservationVersion`.

It shall not imply:

- TCP, UDP, serial, or another transport;
- wire framing;
- serialization encoding;
- byte order;
- deterministic-core software revision.

## 14.2 Representation

`NavigationObservationVersion` shall be a strong domain value backed by an
unsigned 16-bit integer.

The field is explicit metadata carried with the observation.

No sentinel value is required.

## 14.3 Initial behavioral authority

The current baselined requirements do not define:

- supported-versus-unsupported schema-version behavior;
- compatibility rules between schema versions;
- version negotiation;
- version-specific navigation semantics.

Therefore Phase 2B shall not invent rejection or compatibility behavior based
on the numeric schema-version value.

The deterministic core initially preserves the field as observation/evidence
metadata without allowing it to alter navigation acceptance or monitor
behavior.

If a later requirement defines supported schema versions or version-specific
semantics, that behavior shall be introduced through reviewed requirements and
tests.

## 14.4 Future wire-protocol version

A later flight/ground or simulator/core transport may define its own protocol
version.

That future wire-protocol version is a separate interface concept and shall
not silently reuse `NavigationObservationVersion`.

### Status

**RECOMMENDED: unsigned 16-bit navigation-observation schema version metadata**

The distinction between schema version, software revision, and future wire
protocol version is part of the interface contract.

# 15. Runtime Event Representation

The normal runtime event categories are:

- navigation arrival;
- evaluation tick.

Initialization and explicit reset are lifecycle/control operations and are not
normal runtime evaluations.

## 15.1 Candidate A — unrelated public methods

Conceptually:

```text
process_navigation(...)
process_tick(...)
```

Advantages:

- very explicit.

Disadvantages:

- weakens representation of the core as consuming one ordered event stream;
- replay code must dispatch externally.

## 15.2 Candidate B — closed tagged runtime-event value

Conceptually:

```text
RuntimeEvent =
    NavigationArrival
    | EvaluationTick
```

Advantages:

- directly models the ordered event stream;
- closed event set;
- easy deterministic replay;
- one event-time gating path;
- naturally testable.

Disadvantages:

- requires a tagged-union representation such as a variant.

### Status

**RECOMMENDED**

Represent normal runtime inputs as one closed runtime-event sum type.

Initialization, reinitialization, and explicit reset remain separate
lifecycle/control operations.

---

# 16. Optional State Representation

The core has naturally absent state:

- no previously accepted runtime-event time;
- no accepted navigation yet;
- no qualifying interface activity yet;
- no active persistence interval;
- possibly no prior configuration/execution identity in rejection evidence.

## 16.1 Candidate A — sentinel values

Rejected.

Examples such as sequence zero, timestamp minus one, or NaN would overload
valid domain values with presence semantics.

## 16.2 Candidate B — explicit optional values

Advantages:

- absence is represented directly;
- no sentinel collision;
- matches existing semantic distinctions such as `NOT_YET_OBSERVED`.

### Status

**RECOMMENDED**

Use explicit optional-value semantics.

---

# 17. Accepted Navigation Snapshot

## 17.1 Candidate A — separate mutable accepted-state structure duplicating the observation

Disadvantages:

- risks duplicate authoritative ownership;
- can diverge from the original accepted observation identity.

## 17.2 Candidate B — immutable accepted-observation value

The Navigation State Manager owns one optional accepted observation.

Event-local pre-update and post-update snapshots are immutable copies/views
for evaluation.

### Status

**RECOMMENDED**

The accepted observation itself should carry the sequence number, source time,
physical state, validity, and health needed to reconstruct accepted state.

Do not create a second authoritative sequence/time store.

---

# 18. Interface Availability Representation

The required semantic states are:

- no qualifying activity observed yet;
- available;
- timed out.

Possible representations:

1. explicit enum;
2. two booleans;
3. optional last-activity time plus derived classification.

### Assessment

The authoritative persistent state need only be the optional last qualifying
receive time.

Availability classification is derived for each evaluation.

### Status

**RECOMMENDED**

Do not persist an independent availability enum if it can be deterministically
derived from the authoritative timestamp and evaluation time.

A decision record may still contain the derived interface-status result.

---

# 19. Freshness Representation

Likewise, freshness is derived from:

- accepted source time;
- current evaluation time;
- configured freshness limit.

### Status

**RECOMMENDED**

Do not persist independent fresh/stale mutable state.

Represent freshness as an immutable evaluation result derived from
authoritative input/state.

---

# 20. Aggregate Monitor Operating-State Enum

The baseline intentionally avoided inventing one.

Freshness, interface availability, source health, envelope state,
action eligibility, persistence, and action latch are orthogonal.

### Status

**RECOMMENDED: do not introduce one**

A later state machine requires a demonstrated behavioral need and reviewed
requirements.

---

# 21. Reason Identifier Representation

Requirements demand that non-nominal normal runtime decision evidence use
stable machine-readable reason identifiers.

The catalog must also support:

- more than one simultaneous reason;
- deterministic external ordering;
- no mandatory authoritative primary reason;
- observation-rejection evidence inside an accepted navigation-arrival event;
- diagnostic conditions that do not independently create a simulated action.

A reason identifier is not a severity ranking and is not a replacement for the
structured status fields carried by `DecisionRecord`.

## 21.1 Catalog scope

The initial normal-runtime `DecisionReason` catalog shall cover semantic
conditions that are already baselined and may need to be exposed as reasons
for an accepted runtime evaluation.

It shall not include:

- runtime-event rejection reasons for events rejected before normal
  evaluation;
- configuration-validation errors;
- external logging/evidence-sink failures;
- transport failures that occur before a recognized runtime event reaches the
  deterministic core.

Those belong to separate domains.

## 21.2 Stable external reason identifiers

The initial proposed stable external identifiers are:

| Stable ID | Meaning |
|---|---|
| `NAV_VALIDITY_INVALID` | The received navigation observation reports invalid navigation validity. |
| `NAV_SOURCE_HEALTH_INVALID` | The received navigation observation reports source health `INVALID` and is not eligible to replace accepted state. |
| `NAV_PHYSICAL_VALUE_NONFINITE` | A required physical value in the received navigation observation is NaN or infinity. |
| `NAV_SOURCE_TIME_FUTURE` | The received observation source time is later than its navigation-arrival receive time. |
| `NAV_SEQUENCE_DUPLICATE` | The received observation sequence number equals the last accepted sequence number. |
| `NAV_SEQUENCE_REGRESSION` | The received observation sequence number is lower than the last accepted sequence number. |
| `NAV_SOURCE_TIME_DUPLICATE` | The received observation source time equals the last accepted source time. |
| `NAV_SOURCE_TIME_REGRESSION` | The received observation source time is lower than the last accepted source time. |
| `NAV_SEQUENCE_GAP` | The received observation indicates forward sequence progress with one or more missing sequence values. |
| `NAV_NO_ACCEPTED_STATE` | No accepted navigation state exists for the current evaluation. |
| `NAV_STALE` | Accepted navigation exists but is stale at the current evaluation time. |
| `PERSISTENCE_INTERRUPTED_STALE_GAP` | An active action-eligible violation interval was interrupted because the previously accepted navigation became stale between processed runtime events. |
| `NAV_SOURCE_HEALTH_DEGRADED` | Accepted current navigation reports source health `DEGRADED`; it may remain current-state classifiable but is not action eligible. |
| `NAV_INTERFACE_TIMEOUT` | Qualifying interface activity has previously occurred and interface silence exceeds the configured timeout limit. |
| `CURRENT_ENVELOPE_OUTSIDE` | The current-state-evaluation-eligible current position is classified `OUTSIDE`. |
| `PROJECTED_ENVELOPE_OUTSIDE` | Eligible constant-velocity projection is classified `OUTSIDE`; this remains advisory. |
| `POST_RESET_CONFIRMATION_REQUIRED` | The simulated-action latch was explicitly reset and new post-reset accepted navigation confirmation is still required before re-latch. |
| `PERSISTENT_CURRENT_VIOLATION_CONFIRMED` | A qualifying newer action-eligible `OUTSIDE` observation confirms the persistent current-state violation condition used for initial latch creation. |

## 21.3 Observation-ordering reasons may coexist

Ordering evidence shall not be collapsed into one generic
`NAV_ORDERING_ERROR`.

For example, if sequence number progresses while source time does not, the
source-time reason remains visible.

Likewise, a forward sequence gap can coexist with an observation-rejection
reason when the sequence indicates missing observations but another acceptance
condition fails.

This preserves the requirement that simultaneous applicable conditions remain
observable.

## 21.4 Status fields are not duplicated mechanically as reasons

`DecisionRecord` may contain structured fields such as:

- interface status;
- freshness status;
- current envelope result;
- projected envelope result;
- simulated-action result.

The reason catalog shall not automatically create a reason identifier for
every possible nominal status value.

For example, `INSIDE`, `HEALTHY`, and interface `AVAILABLE` do not require
nominal reason identifiers.

`NOT_YET_OBSERVED` remains a structured interface-status value rather than an
initial reason identifier; when no accepted navigation exists,
`NAV_NO_ACCEPTED_STATE` already captures the decision-relevant absence of
current navigation evidence.

## 21.5 Action and persistence reasons

`CURRENT_ENVELOPE_OUTSIDE` does not by itself mean that a new simulated action
was created.

`PERSISTENT_CURRENT_VIOLATION_CONFIRMED` identifies the specific confirming
condition that satisfies the sole initial latch trigger.

After the latch is already active, the latch state remains represented by the
structured simulated-action result. The persistent-violation-confirmed reason
does not need to be reasserted merely because the latch remains set.

`PERSISTENCE_INTERRUPTED_STALE_GAP` is required because a navigation-arrival
event may install a fresh accepted state after the previously accepted state
already crossed its freshness boundary. In that case, current freshness alone
would not explain why the prior persistence interval was interrupted.

`POST_RESET_CONFIRMATION_REQUIRED` may coexist with current-state outside
evidence when reset has cleared the latch but no qualifying post-reset
navigation observation has yet supplied the required new confirmation.

## 21.6 Internal representation

The initial C++ design should use a closed reason-code domain such as an
`enum class` or equivalent strongly typed value.

The enum's underlying integer values shall not themselves become the stable
external identifier contract.

A separate explicit mapping shall associate each internal reason value with
its stable external string identifier.

### Status

**RECOMMENDED: closed internal reason domain with explicit stable external IDs**

The stable external IDs listed above form the initial proposed normal-runtime
reason catalog for Phase 2B review.

Any future addition shall be explicit and shall not silently reinterpret an
existing identifier.

---

# 22. Reason Ordering

Requirements require deterministic external ordering whenever more than one
reason applies to one normal runtime evaluation.

Ordering is for reproducible evidence only.

It shall not imply:

- severity;
- causal dominance;
- action authority;
- primary-reason status.

## 22.1 Rejected alternatives

### Enum underlying value

Rejected as the external ordering contract.

It couples externally visible behavior to an implementation detail.

### Declaration order

Rejected as the external ordering contract.

Refactoring declarations could accidentally change evidence order.

### Alphabetical identifier order

Deterministic but semantically arbitrary and vulnerable to identifier naming
changes.

## 22.2 Explicit canonical rank table

Use an explicit rank associated with each normal-runtime reason.

The proposed initial order is:

| Rank | Stable ID |
|---:|---|
| 10 | `NAV_VALIDITY_INVALID` |
| 20 | `NAV_SOURCE_HEALTH_INVALID` |
| 30 | `NAV_PHYSICAL_VALUE_NONFINITE` |
| 40 | `NAV_SOURCE_TIME_FUTURE` |
| 50 | `NAV_SEQUENCE_DUPLICATE` |
| 60 | `NAV_SEQUENCE_REGRESSION` |
| 70 | `NAV_SOURCE_TIME_DUPLICATE` |
| 80 | `NAV_SOURCE_TIME_REGRESSION` |
| 90 | `NAV_SEQUENCE_GAP` |
| 100 | `NAV_NO_ACCEPTED_STATE` |
| 110 | `NAV_STALE` |
| 120 | `PERSISTENCE_INTERRUPTED_STALE_GAP` |
| 130 | `NAV_SOURCE_HEALTH_DEGRADED` |
| 140 | `NAV_INTERFACE_TIMEOUT` |
| 150 | `CURRENT_ENVELOPE_OUTSIDE` |
| 160 | `PROJECTED_ENVELOPE_OUTSIDE` |
| 170 | `POST_RESET_CONFIRMATION_REQUIRED` |
| 180 | `PERSISTENT_CURRENT_VIOLATION_CONFIRMED` |

The rank values are ordering metadata, not reason identities.

Spacing ranks by ten leaves room for later additions without requiring an
immediate renumbering of all subsequent entries.

A later addition shall still receive explicit engineering review.

## 22.3 Ordering rationale

The canonical order follows the logical decision pipeline:

1. observation validity and source acceptability;
2. observation ordering and missing-update evidence;
3. accepted-information availability and freshness;
4. persistence interruption caused by an unobserved stale interval;
5. source-health and interface conditions;
6. current and projected envelope results;
7. reset-confirmation and persistent-action confirmation.

This provides a stable reproducible presentation order without claiming that
an earlier reason is more severe or more important.

## 22.4 Separate runtime-event rejection reason domain

Runtime events rejected before normal evaluation do not use
`DecisionReason`.

The initial baselined runtime-event rejection condition is regressive modeled
event time.

The separate rejection-reason domain therefore initially requires:

```text
RUNTIME_EVENT_TIME_REGRESSION
```

Additional rejection reasons may be added only when the runtime-event gate
gains another baselined rejection condition.

The rejection-evidence record does not participate in the normal
`DecisionReason` canonical-order table because it is not a normal
`DecisionRecord`.

### Status

**RECOMMENDED: explicit canonical rank table with no primary reason**

The catalog and ordering above should be reviewed as one decision because the
external reason IDs and deterministic ordering contract are coupled evidence
interfaces.

# 23. Decision Record Representation

The requirements define a logical record, not a serialization.

## 23.1 Candidate A — logging object

Rejected.

Logging is explicitly non-authoritative and must not affect decision
semantics.

## 23.2 Candidate B — immutable domain value

Contains the reconstructability fields required by the system requirements.

External sinks may later serialize or log it.

### Status

**RECOMMENDED**

`DecisionRecord` should be a deterministic domain value produced after the
monitor decision has already been determined.

It should not own external I/O behavior.

---

# 24. Runtime-Event Rejection Evidence

The requirements explicitly distinguish runtime-event rejection evidence from
normal decision records.

## 24.1 Candidate A — one record with a boolean `rejected`

Rejected.

This blurs events that entered normal evaluation with events rejected before
evaluation.

## 24.2 Candidate B — separate evidence value type

### Status

**RECOMMENDED**

Use a distinct runtime-event rejection evidence type.

Both evidence types may share small common value types, but they remain
semantically distinct outputs.

---

# 25. Core Output Type

A normal submitted runtime event has two top-level outcomes:

1. accepted runtime event -> exactly one `DecisionRecord`;
2. rejected runtime event -> runtime-event rejection evidence.

## 25.1 Candidate A — exceptions for regressive-time rejection

Not preferred.

A regressive external event is an expected modeled input condition requiring
deterministic evidence, not an exceptional host failure.

## 25.2 Candidate B — explicit result sum type

Conceptually:

```text
RuntimeEventResult =
    DecisionRecord
    | RuntimeEventRejectionEvidence
```

### Status

**RECOMMENDED**

This preserves the required semantic distinction explicitly in the type
system.

---

# 26. Public Runtime Processing API

## 26.1 Candidate A — separate processing APIs per runtime event

Possible but weakens the common ordered-runtime-event abstraction.

## 26.2 Candidate B — one normal-event processing entry point

Conceptually:

```text
process(RuntimeEvent) -> RuntimeEventResult
```

Advantages:

- one event-time gate;
- closed event set;
- deterministic ordered replay;
- simple cardinality contract.

### Status

**RECOMMENDED**

Exact C++ names remain deferred until the interface specification increment.

---

# 27. Initialization API

Initialization:

- validates configuration;
- establishes an execution;
- establishes initial modeled time;
- creates fresh execution-specific state.

It is not a normal runtime evaluation.

Potential designs:

1. constructor that can fail;
2. factory returning success/error;
3. default construction followed by initialize.

### Assessment

Default construction followed by mutable initialization permits an
uninitialized monitor object to exist.

A throwing constructor mixes modeled validation failure with host exception
policy.

### Status

**RECOMMENDED: explicit validated factory/result creation**

Exact result/error type remains for the concrete interface specification.

---

# 28. Reinitialization API

Reinitialization establishes a new execution rather than mutating selected
state within the prior execution.

Possible designs:

1. mutate the same monitor object;
2. return/create a fresh monitor execution object;
3. use the same initialization factory externally.

### Status

**RECOMMENDED: create a fresh initialized execution object**

Reinitialization shall use the same validated creation path as initialization
to establish a new monitor execution with fresh execution-specific state.

The initial design shall not require an in-place mutating
`reinitialize(...)` operation on an existing execution object.

This makes the new-execution boundary structural and reduces the risk that
accepted navigation, runtime-event history, interface history, persistence,
reset-confirmation state, or other execution-specific state can accidentally
leak into the new execution.

A caller may replace its old monitor object with the newly created execution,
but that replacement occurs outside the semantics of the old execution.

Explicit reset remains a distinct narrow in-execution mutation.

---

# 29. Explicit Reset API

The reset semantics are already baselined:

- clear simulated-action latch;
- preserve unrelated execution state;
- preserve an otherwise continuous active violation interval;
- establish a new post-reset confirmation boundary;
- ticks alone cannot re-latch.

### Status

**RECOMMENDED**

Expose reset as a dedicated control operation rather than a normal
`RuntimeEvent`.

Its concrete return type remains open.

Reset shall not produce a normal `DecisionRecord` unless a later requirement
changes the current event/evidence contract.

---

# 30. Configuration Validation Result

Invalid configuration blocks monitoring.

Potential designs:

1. boolean;
2. exception;
3. explicit validation result containing stable validation issues.

### Status

**RECOMMENDED: explicit result**

A boolean loses evidence.

Exceptions are unnecessary for expected invalid synthetic configuration.

The exact error catalog remains interface-design work.

---

# 31. Serialization

The requirements intentionally defer serialization.

### Status

**DEFERRED**

The deterministic core shall not depend on:

- JSON;
- YAML;
- protobuf;
- database schemas;
- transport framing.

Adapters may be added later.

---

# 32. Evidence Sink

The requirements state that logging shall not change monitor semantics.

The behavior of a failed external evidence sink remains intentionally open.

### Status

**DEFERRED outside the deterministic core**

Core values shall be complete before any external sink receives them.

A sink failure must not retroactively change the already-produced logical
decision.

No sink implementation is required for the initial core interface design.

---

# 33. Initialization Acquisition Deadline

The current baseline explicitly leaves this unrequired.

### Status

**DEFERRED**

Before first qualifying interface activity, the interface remains
`NOT_YET_OBSERVED`.

Do not invent an acquisition timeout.

---

# 34. Configuration Structural Validity and Numeric Ranges

The baseline defines semantic relationships but intentionally does not define
application-specific numeric ranges.

Phase 2B distinguishes structural validity from policy range selection.

## 34.1 Required configuration fields

A monitor configuration shall explicitly provide, at minimum:

- configuration identity;
- navigation freshness limit;
- interface timeout limit;
- current-envelope persistence duration;
- projection horizon;
- minimum and maximum Cartesian envelope bounds for x, y, and z.

The deterministic core shall not silently substitute default threshold values
for omitted required configuration.

## 34.2 Duration validity

The following configured durations shall be non-negative:

- navigation freshness limit;
- interface timeout limit;
- current-envelope persistence duration;
- projection horizon.

Zero is structurally valid for each of these durations.

Zero-valued configurations preserve deterministic and testable semantics:

- a zero freshness limit permits freshness only at zero navigation age;
- a zero interface-timeout limit permits availability by the silence
  criterion only at zero interface silence;
- a zero persistence duration still requires the newer confirming navigation
  evidence required by the persistent-violation rules;
- a zero projection horizon produces a projection at the current accepted
  position under the constant-velocity equation.

No requirement currently justifies imposing an arbitrary positive minimum.

Because modeled durations use an integral representation, NaN and infinity do
not apply to duration storage.

## 34.3 Envelope validity

Every Cartesian envelope bound shall be finite.

For each axis:

```text
minimum < maximum
```

shall be required.

Equal minimum and maximum values are invalid configuration.

No arbitrary absolute position limit is introduced by the deterministic core.

## 34.4 Cross-field relationships

No ordering relationship is currently required between:

- navigation freshness limit;
- interface timeout limit;
- persistence duration;
- projection horizon.

For example, the initial core shall not invent requirements such as:

```text
interface_timeout > navigation_freshness
persistence_duration < navigation_freshness
projection_horizon < interface_timeout
```

unless later requirements provide a behavioral reason.

These configuration values represent distinct semantic concepts.

## 34.5 Configuration identity

Configuration identity is required for evidence traceability.

Its exact storage representation remains part of the concrete interface-type
design.

Validation shall require a valid identity value once that identity type is
defined.

The core shall not generate configuration identity implicitly.

## 34.6 Upper numeric ranges

No arbitrary maximum duration or envelope magnitude is introduced solely as a
defensive implementation convention.

Representability and arithmetic overflow shall be handled explicitly by the
domain-type and arithmetic design.

If later analysis demonstrates that a bounded configuration range is required
for correct arithmetic or another concrete engineering reason, that bound
shall be documented and verified rather than introduced silently.

## 34.7 Immutability

After successful configuration validation and execution creation, the active
configuration remains immutable for that execution.

Changing configuration requires creation of a new initialized execution.

### Status

**RECOMMENDED: baseline semantic structural constraints only**

The initial configuration validator should enforce the structural rules above.

Application-specific threshold selections and arbitrary policy maxima remain
deferred.

Synthetic verification configurations shall continue to be chosen for test
clarity and boundary coverage rather than copied from operational aerospace
material.

# 35. Projection Duration Conversion

With integral `ScenarioDuration` and double-valued physical state, projection
requires an explicit conversion of duration to seconds.

Conceptually:

```text
dt_seconds =
    exact_integer_duration / fixed_units_per_second
```

followed by:

```text
projected_position =
    current_position + current_velocity * dt_seconds
```

### Status

**RECOMMENDED**

The conversion shall occur in one clearly defined projection path rather than
through scattered unit conversion.

Its numerical behavior shall receive dedicated tests.

---

# 36. Concurrency

The core consumes an explicitly ordered event sequence.

No requirement introduces concurrent core mutation.

### Status

**RECOMMENDED: single ordered semantic interface**

Concurrency, if ever required by an integration layer, belongs outside the
deterministic state transition path.

Do not introduce locks, internal queues, workers, or asynchronous APIs into
the initial core.

---

# 37. Dynamic Allocation Policy

The initial deterministic monitor core is a host-based C++20 component.

No current requirement establishes:

- hard real-time deadlines;
- bounded-latency allocation requirements;
- an embedded memory budget;
- a prohibition on heap allocation;
- a requirement for fixed-capacity containers.

The broader project guidance encourages conservative C++ design and asks that
dynamic allocation in steady-state paths be limited where justified, but it
also explicitly rejects applying such rules dogmatically.

## 37.1 Rejected alternatives

### Blanket prohibition on dynamic allocation

Rejected for the initial host core.

A blanket ban would import an embedded constraint that the current
requirements do not contain and could force unnecessary custom containers or
ownership mechanisms.

### Unconstrained allocation as an architectural assumption

Also rejected.

The absence of a blanket prohibition does not justify allocation-heavy design,
hidden ownership, or unnecessary heap traffic.

## 37.2 Initial policy

Phase 2B-1 may use standard C++ value types and standard-library containers
where they provide clear ownership, bounded conceptual scope, and simple
verification.

Concrete interfaces should prefer:

- value semantics;
- explicit ownership;
- immutable inputs and outputs where practical;
- fixed-size domain values for position, velocity, time, sequence, and status;
- avoiding allocation solely for polymorphism or abstraction machinery.

The deterministic core shall not introduce:

- background allocators;
- custom memory pools;
- allocator frameworks;
- fixed-capacity container libraries;
- arena ownership schemes

without a demonstrated engineering need.

Allocation behavior may be measured during later profiling and host-to-target
work.

If a later embedded target, timing budget, memory budget, or measured
performance result creates a concrete allocation constraint, that constraint
shall be introduced through requirements/design change and verified.

### Status

**RECOMMENDED: no blanket allocation restriction in the initial host core**

Allocation constraints do not block Phase 2B-1.

The initial design shall remain simple and ownership-explicit rather than
pretending that an embedded allocation policy already exists.

# 38. Decision Summary

| Topic | Current disposition |
|---|---|
| Strong scenario-time type | RECOMMENDED |
| Strong duration type | RECOMMENDED |
| Signed 64-bit time storage | RECOMMENDED |
| Exact time unit | RECOMMENDED nanoseconds |
| Unsigned 64-bit sequence | RECOMMENDED |
| Initial sequence rollover | RECOMMENDED unsupported |
| Physical scalar | RECOMMENDED binary64 / `double` |
| Position/velocity value types | RECOMMENDED distinct Cartesian types |
| Configuration identity | RECOMMENDED opaque caller-supplied identity |
| Execution identity | RECOMMENDED no separate core-generated identity |
| Evaluation identity | RECOMMENDED triggering runtime-event reference |
| Runtime-event reference storage | RECOMMENDED strong unsigned 64-bit value |
| Protocol/version representation | RECOMMENDED unsigned 16-bit navigation schema version metadata |
| Runtime events | RECOMMENDED closed sum type |
| Optional state | RECOMMENDED explicit optional semantics |
| Accepted navigation | RECOMMENDED immutable observation snapshot |
| Persisted availability state | RECOMMENDED no; derive it |
| Persisted freshness state | RECOMMENDED no; derive it |
| Aggregate operating-state enum | RECOMMENDED none |
| Reason representation | RECOMMENDED explicit catalog + stable external IDs |
| Reason ordering | RECOMMENDED explicit canonical rank table |
| DecisionRecord | RECOMMENDED immutable domain value |
| Runtime rejection evidence | RECOMMENDED separate domain value |
| Runtime event result | RECOMMENDED explicit sum type |
| Runtime processing API | RECOMMENDED single ordered event entry point |
| Initialization | RECOMMENDED explicit validated creation |
| Reinitialization API | RECOMMENDED fresh initialized execution object |
| Explicit reset | RECOMMENDED separate control operation |
| Config validation | RECOMMENDED explicit result |
| Serialization | DEFERRED |
| Evidence sink policy | DEFERRED outside core |
| Acquisition deadline | DEFERRED |
| Configuration structural validity | RECOMMENDED semantic constraints only |
| Complete config policy ranges | DEFERRED; no arbitrary maxima |
| Concurrency | RECOMMENDED outside core |
| Allocation constraints | RECOMMENDED no blanket host-core restriction |

---

# 39. Final Non-Blocking Deferral Review

Phase 2B-0 engineering review confirms that no unresolved `OPEN` disposition
blocks concrete domain-type or public-interface design.

The remaining `DEFERRED` dispositions are intentional and non-blocking.

## 39.1 Serialization

Serialization remains deferred.

The deterministic core can define typed domain inputs, outputs, decision
evidence, and rejection evidence without selecting JSON, YAML, Protobuf,
binary framing, or another external encoding.

Serialization belongs to a later adapter/interface increment and shall not
change core decision semantics.

## 39.2 Evidence-sink failure policy

Evidence-sink failure behavior remains deferred outside the deterministic core.

The core produces logical evidence values.

An external sink may later define retry, persistence, reporting, or failure
handling, but sink behavior shall not retroactively alter an already
determined monitor result.

## 39.3 Initialization acquisition deadline

An initialization acquisition deadline remains deferred.

The current baseline already defines the pre-activity and no-accepted-state
conditions without requiring a deadline-triggered transition.

No use case currently justifies inventing an acquisition deadline.

## 39.4 Complete configuration policy ranges

Application-specific configuration ranges remain deferred.

The structural validity rules required for deterministic semantics are now
defined, while arbitrary maxima and operationally sourced thresholds remain
outside the initial core design.

Representability and arithmetic correctness shall be handled by the domain-type
and arithmetic design rather than by unexplained policy limits.

## 39.5 Exit-review conclusion

The remaining deferrals do not prevent Phase 2B-1 from defining:

- concrete C++ domain types;
- immutable value types and snapshots;
- initialization and reset interfaces;
- runtime-event input types;
- normal decision and rejection-evidence types;
- reason-code representations and canonical ordering;
- deterministic public monitor signatures.

No implementation is authorized merely by this trade study.

Phase 2B-1 remains an interface/design increment and shall preserve all
baselined system semantics, requirements, hazard controls, and synthetic-only
scope.

# 40. Phase 2B-0 Exit Criteria

This tradeoff review is complete when:

- every intentionally deferred representation decision is either resolved,
  explicitly left open, or explicitly deferred;
- no implementation choice is being smuggled in as an unstated assumption;
- time representation preserves exact modeled-time semantics;
- sequence representation does not invent rollover behavior;
- physical representation preserves finite-value and exact-boundary rules;
- authoritative mutable state remains minimal;
- optional absence is not represented through sentinels;
- normal decision evidence remains distinct from runtime-event rejection
  evidence;
- reset and reinitialization remain distinct;
- serialization remains outside the core unless later required;
- no unnecessary monitor-state machine is introduced;
- no operational aerospace values, thresholds, timing, or procedures are
  introduced;
- no C++ implementation has begun.

Only after this review is accepted should Phase 2B-1 define concrete domain
types and public interface signatures.

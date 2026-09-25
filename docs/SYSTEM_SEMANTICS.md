# Flight Safety Monitor & Verification Platform
## Phase 0 — System Semantic Model

**Status:** Draft for engineering review
**Lifecycle phase:** Phase 0 — Engineering Definition
**Purpose:** Define the information, event, time, evaluation, and decision semantics that later requirements and architecture must preserve.

---

## 1. Purpose

This document defines the logical semantics of the synthetic Flight Safety Monitor before software requirements or implementation architecture are baselined.

It answers five questions:

1. What information exists?
2. What events can change what the monitor knows?
3. What does time mean?
4. When does evaluation occur?
5. What must a decision record mean?

The document intentionally avoids prescribing C++ classes, Python modules, serialization formats, IPC mechanisms, files, threads, or process boundaries.

---

## 2. Governing Principles

### 2.1 Deterministic logical behavior

For a fixed:

- software revision;
- supported execution environment;
- initial monitor state;
- validated configuration;
- ordered external event sequence;
- event payloads;
- modeled-time values;

the monitor shall produce the same externally observable decision sequence.

Wall-clock time, thread scheduling, filesystem timing, network timing, logging latency, and host load shall not influence core decision semantics.

### 2.2 Synthetic-only interpretation

All state, limits, trajectories, thresholds, timing values, health states, and action outputs are synthetic.

Nothing in this document defines operational launch-vehicle or range-safety behavior.

### 2.3 Event-driven evaluation

The monitor is conceptually driven by explicit external events.

The passage of modeled time must be representable even when no navigation observation arrives.

This is necessary so timeout and stale-data behavior can be verified deterministically.

### 2.4 Separation of validity from safety consequence

The following are distinct concepts:

- whether an input observation is structurally valid;
- whether it is temporally acceptable;
- whether it is accepted as the current vehicle state;
- whether monitoring capability is degraded;
- whether a synthetic safety condition exists;
- whether a simulated safety action is requested.

Later requirements may connect these concepts, but this semantic model shall not collapse them into one condition.

---

## 3. Initial Synthetic State Model

The first project version shall use a deliberately simple three-dimensional Cartesian state model.

### 3.1 Position

Synthetic position consists of:

- `position_x_m`
- `position_y_m`
- `position_z_m`

Units are meters.

The coordinate frame is a synthetic local Cartesian frame.

It does not represent a real launch range, geodetic reference system, ECEF frame, orbital frame, or protected operational coordinate system.

### 3.2 Velocity

Synthetic velocity consists of:

- `velocity_x_mps`
- `velocity_y_mps`
- `velocity_z_mps`

Units are meters per second.

### 3.3 Acceleration

Acceleration is not part of the initial monitored state.

The first projection model uses constant velocity.

Acceleration may be introduced only if a later requirement creates a specific need.

### 3.4 Navigation validity

The initial semantic model uses one explicit navigation-validity indication:

- `VALID`
- `INVALID`

Additional quality levels are not introduced unless later requirements need behavior that cannot be represented cleanly with the binary validity state.

### 3.5 Vehicle/system health

The initial semantic model includes one explicit synthetic health indication:

- `HEALTHY`
- `DEGRADED`
- `INVALID`

These values represent synthetic source-reported health information.

They are not the same as the monitor's own internal operating state.

---

## 4. Navigation Observation

A navigation observation conceptually contains:

- protocol/version identity;
- sequence number;
- source timestamp;
- position;
- velocity;
- navigation-validity indication;
- source health indication.

A navigation observation is an input fact presented to the monitor.

It is not automatically an accepted vehicle state.

### 4.1 Observation identity

Every observation shall have enough information to distinguish it from preceding observations.

The first version uses:

- source timestamp;
- sequence number.

### 4.2 Numeric validity

Every physical numeric field presented to the monitor must represent a finite value.

NaN and positive or negative infinity are invalid input values.

The implementation representation is not yet baselined by this document.

### 4.3 Acceptance concept

The monitor may:

- receive an observation;
- validate the observation;
- reject the observation;
- accept the observation as the newest accepted navigation state.

These are distinct semantic stages.

A rejected observation shall not silently replace the last accepted navigation state.

---

## 5. Configuration Model

Configuration is supplied before an execution begins.

For the initial project version, configuration is immutable from successful initialization until explicit execution reset or termination.

Configuration conceptually contains at least:

- configuration identity/version;
- synthetic envelope limits;
- persistence duration;
- projection horizon;
- navigation freshness limit;
- interface timeout limit;
- any state-machine parameters later justified by requirements.

### 5.1 Configuration validation

Configuration must be validated before monitoring begins.

Invalid or internally inconsistent configuration shall prevent successful initialization.

The monitor shall not enter normal monitoring with partially accepted configuration.

### 5.2 Configuration provenance

Every decision record shall identify the configuration under which that decision was produced.

---

## 6. Time Model

Time is represented explicitly and never obtained implicitly by the deterministic core.

The semantic model distinguishes:

- source time;
- receive time;
- evaluation time;
- processing time.

### 6.1 Modeled time domain

Source time, receive time, and evaluation time shall belong to one synthetic monotonic scenario-time domain.

The zero point of this domain is scenario-defined.

The domain does not represent UTC, GPS time, Unix epoch time, or any real mission clock.

### 6.2 Source time

Source time is the modeled time at which the synthetic source claims the observation applies.

It is carried by the navigation observation.

### 6.3 Receive time

Receive time is the modeled time at which the monitor is told that the observation became available to it.

Receive time is supplied externally to the deterministic core.

### 6.4 Evaluation time

Evaluation time is the modeled time at which the monitor evaluates its current information and produces a decision record.

### 6.5 Processing time

Real host processing duration is not part of core safety semantics during the initial project phases.

Performance may be measured later, but host execution time shall not determine freshness, persistence, timeout, or safety decisions.

### 6.6 Time ordering

Within one execution, externally supplied modeled event time shall never move backward.

A runtime event whose modeled time is earlier than the previously accepted runtime-event time is invalid.

Such an event shall not advance monitor state or trigger a normal safety evaluation.

The rejection shall remain observable in verification evidence.

The eventual interface/design shall define whether this rejection is represented by a decision-record form or by a separate interface-level diagnostic record. That representation is not baselined during Phase 0.

### 6.7 Data age

Navigation data age is conceptually:

`evaluation_time - last_accepted_source_time`

when an accepted observation exists.

This measures the age of the state represented by the accepted navigation data.

### 6.8 Interface silence duration

Interface silence is conceptually:

`evaluation_time - last_qualifying_interface_receive_time`

when at least one qualifying interface activity has been received.

What constitutes qualifying interface activity is intentionally not yet baselined.

This is distinct from navigation data age.

An observation may arrive recently while still carrying old source data.

### 6.9 Persistence duration

Boundary-violation persistence shall be based on elapsed modeled time, not merely on a count of consecutive observations.

This avoids changing safety behavior merely because the synthetic source update rate changes.

---

## 7. External Event Model

The deterministic monitor is conceptually driven by an ordered sequence of external events.

The initial event categories are:

1. initialization;
2. navigation observation arrival;
3. evaluation tick;
4. explicit reset.

These are semantic events, not transport messages or C++ types.

---

## 8. Initialization Event

Initialization provides:

- validated configuration;
- initial modeled time.

Initialization establishes a new execution context.

Successful initialization shall establish deterministic initial monitor state.

Initialization shall not depend on:

- wall clock;
- random state;
- filesystem contents;
- previous process executions;
- logging state.

The exact monitor operating state after initialization remains to be derived from requirements.

---

## 9. Navigation Observation Arrival Event

A navigation-arrival event provides:

- the observation;
- receive time.

Processing an event that passes runtime-event validation conceptually performs:

1. event-time validation;
2. structural/numeric validation;
3. temporal/sequence validation;
4. observation acceptance or rejection;
5. update of qualifying interface-receipt information as defined by later requirements;
6. evaluation at the event's receive time;
7. production of a decision record.

If event-time validation fails, the remaining normal evaluation steps do not occur.

Important distinction:

An observation may be rejected as current navigation state while still proving that the interface delivered something at a particular receive time.

Whether malformed input updates interface-availability evidence is not yet baselined and must be resolved before requirements are finalized.

---

## 10. Evaluation Tick Event

An evaluation tick provides:

- evaluation time.

It contains no new navigation observation.

Its purpose is to make the passage of modeled time explicit.

A tick enables deterministic detection of conditions such as:

- stale accepted navigation;
- interface timeout;
- elapsed boundary-persistence duration;
- other time-dependent conditions.

This prevents timeout logic from depending on the arrival of another packet.

### 10.1 Tick frequency

Core semantics do not depend on a fixed wall-clock scheduler frequency.

Scenario and integration layers may choose when to supply ticks.

However, verification must ensure that expected time-dependent transitions are evaluated at relevant boundary times.

---

## 11. Explicit Reset Event

Reset begins a new logical monitoring execution.

Reset semantics shall:

- clear execution-specific monitor state;
- clear any latched simulated action;
- clear accepted-navigation history;
- clear persistence timers;
- require configuration to be established according to the initialization contract.

A simulated action latch, once established, shall not disappear merely because later navigation becomes nominal.

It may clear only through explicit reset/reinitialization.

This defines latch meaning but does not yet define which conditions are permitted to create a latch.

---

## 12. Event Ordering

Events are processed in the exact order presented to the deterministic core.

For two events with distinct modeled times, the earlier modeled time must appear first.

For multiple events at the same modeled time, their order must be explicit in the event sequence.

The monitor shall not invent an ordering based on host scheduling.

Verification scenarios must therefore specify same-time event ordering when it matters.

---

## 13. Sequence Number Semantics

The first interface version shall use a monotonically increasing unsigned logical sequence number.

Within one execution:

- an equal sequence number indicates a duplicate;
- a lower sequence number indicates sequence regression or reordering;
- a higher sequence number indicates forward progress;
- a gap indicates one or more missing observations.

The exact integer width and rollover behavior remain implementation/interface-design decisions.

For the initial host-based verification scope, scenarios shall not depend on sequence-number rollover.

A skipped sequence number is evidence of missing data but does not, by itself, determine whether the newly received observation must be rejected.

That behavior requires a requirement.

---

## 14. Source Timestamp Semantics

For observations eligible to become accepted navigation state, source timestamp shall progress strictly forward relative to the last accepted observation.

Equal source timestamp is not considered newer state.

A lower source timestamp is temporally regressive.

The final acceptance behavior for combinations such as:

- higher sequence number with older source time;
- lower sequence number with newer source time;

must be specified explicitly in software requirements.

The semantic model deliberately exposes this conflict instead of silently choosing one field as authoritative.

---

## 15. Evaluation Model

Every accepted runtime event produces one logical evaluation.

Runtime events are:

- navigation observation arrival;
- evaluation tick.

Runtime-event acceptance is distinct from navigation-observation acceptance.

A navigation-arrival event may be a valid runtime event even when the navigation observation contained by that event is rejected.

Initialization and reset establish execution context but do not need to represent normal monitoring evaluations unless later requirements demand a decision record for them.

### 15.1 Evaluation inputs

A logical evaluation may use:

- current monitor operating state;
- validated immutable configuration;
- current evaluation time;
- last accepted navigation state;
- last qualifying interface-activity evidence;
- accumulated persistence state;
- any previously latched simulated action;
- current event result.

### 15.2 Evaluation ordering

Within one evaluation, conceptual ordering is:

1. validate event time;
2. process event-specific input;
3. determine current information validity/freshness;
4. determine current interface status;
5. determine current synthetic envelope condition;
6. determine projection eligibility and projected condition;
7. update persistence state;
8. evaluate state-machine transition rules;
9. determine simulated action;
10. produce decision record.

This ordering is a semantic dependency ordering.

It does not require one monolithic implementation function.

---

## 16. Initial Synthetic Safety Envelope

The initial project version shall use an axis-aligned three-dimensional rectangular envelope in the synthetic Cartesian frame.

Configuration defines:

- minimum x;
- maximum x;
- minimum y;
- maximum y;
- minimum z;
- maximum z.

For valid configuration:

- minimum must be less than maximum for each axis.

### 16.1 Boundary semantics

A position exactly on a configured boundary is considered inside the envelope.

A current-state envelope violation exists only when at least one coordinate is strictly outside its configured minimum/maximum interval.

This choice provides unambiguous boundary tests without introducing a general floating-point epsilon policy.

### 16.2 Rationale

The axis-aligned box is intentionally simple.

It is sufficient to exercise:

- inside behavior;
- exact-boundary behavior;
- single-axis violation;
- multi-axis violation;
- transient excursion;
- persistent excursion;
- projected violation.

It avoids introducing computational geometry that is unrelated to the project's principal engineering goals.

---

## 17. Persistence Semantics

A current-state envelope violation begins a violation interval.

If later valid accepted state returns inside the envelope before the configured persistence duration is reached, the interval ends without becoming a persistent violation.

If the violation remains continuously present for at least the configured persistence duration, a persistent violation condition exists.

Persistence is based on modeled elapsed time.

### 17.1 Continuity

Only information considered eligible by later requirements may extend or clear a persistence interval.

For example, the project must explicitly decide whether loss of valid navigation:

- freezes the interval;
- clears the interval;
- makes the interval indeterminate;
- causes another monitor condition.

This remains unresolved and must not be guessed in implementation.

---

## 18. Projection Model

The initial synthetic projection uses constant velocity.

For projection horizon `dt`:

`projected_position = current_position + current_velocity * dt`

independently for x, y, and z.

The projection horizon is supplied by validated immutable configuration.

### 18.1 Projection eligibility

Projection requires an accepted navigation state whose:

- numeric values are valid;
- navigation-validity indication permits use;
- age is within the eventual projection-eligibility limit.

Projection shall not silently proceed from data already considered unusable.

Exact eligibility conditions belong in requirements.

### 18.2 Projection envelope evaluation

Projected position is evaluated against the same synthetic envelope semantics as current position unless future requirements establish a distinct projected envelope.

### 18.3 Projection limitation

The projection is a synthetic verification model.

It does not claim realistic aerospace trajectory prediction.

---

## 19. Monitor Operating State

The semantic model requires an explicit monitor operating state, but does not baseline state names yet.

A state shall exist only when it represents behavior that is materially different from other states.

Potential concepts such as:

- initializing;
- monitoring;
- degraded;
- latched action;

remain candidates rather than requirements.

`STANDBY` shall not be introduced unless a distinct required behavior justifies it.

---

## 20. Information Status Versus Monitor State

The following must remain distinguishable:

- navigation valid/invalid;
- navigation fresh/stale;
- interface available/timed out;
- envelope inside/outside;
- projected envelope inside/outside;
- monitor operating state;
- simulated action state.

A monitor state must not become a dumping ground for all information-status combinations.

---

## 21. Decision Model

Every logical runtime evaluation produces one `DecisionRecord` concept.

The name is semantic; implementation naming may differ later.

A decision record shall make it possible to reconstruct why the evaluation produced its result.

Conceptually it contains:

- evaluation identity;
- evaluation time;
- triggering event category;
- configuration identity;
- prior monitor state;
- resulting monitor state;
- observation identity when relevant;
- accepted/rejected status when relevant;
- current information status;
- current envelope status;
- projected envelope status when evaluated;
- simulated action result;
- stable reason codes.

### 21.1 Simulated action domain

The initial semantic action domain is:

- `NO_SIMULATED_SAFETY_ACTION`
- `SIMULATED_SAFETY_ACTION_REQUEST`

No operational hardware action exists.

### 21.2 Action latch

If a simulated action becomes latched by future requirements, every subsequent evaluation shall continue to report the latched simulated action until explicit reset/reinitialization.

The triggering evidence must remain reconstructable.

---

## 22. Reason-Code Model

A decision may have more than one applicable reason code.

Reason codes are stable machine-readable identifiers.

Free-form explanatory strings may accompany them later but shall not be the authoritative decision explanation.

### 22.1 Multiple reasons

Simultaneous conditions shall not be discarded merely because one condition is selected as primary.

The decision record shall support:

- zero or one primary reason;
- zero or more contributing reasons.

Whether a primary reason is required for every non-nominal decision will be determined in requirements.

### 22.2 Stable ordering

When multiple reason codes are emitted, their output order must be deterministic.

The ordering shall be defined by a stable reason-code precedence table rather than incidental evaluation order.

The precedence table itself is not yet defined.

---

## 23. Observation Rejection Semantics

Observation rejection means:

- the observation does not replace the last accepted navigation state;
- the rejection is observable in the decision record;
- an explicit reason identifies why it was rejected.

Observation rejection does not, by itself, define:

- monitor-state transition;
- degraded operation;
- simulated safety action;
- recovery behavior.

Those are later requirements decisions.

---

## 24. Missing, Duplicate, and Reordered Data

### 24.1 Missing observation evidence

A sequence-number gap provides evidence that one or more expected observations were not observed.

It does not automatically imply interface timeout.

### 24.2 Duplicate observation

A duplicate does not represent newer vehicle state.

### 24.3 Reordered observation

An observation that regresses relative to accepted sequencing/time does not represent newer accepted state.

The exact monitor consequence of repeated duplicates, gaps, or reordering remains requirements-driven.

---

## 25. Interface Availability Semantics

Interface availability and navigation freshness are distinct.

### 25.1 Navigation freshness

Freshness concerns how old the represented vehicle state is.

It is based on accepted source time.

### 25.2 Interface availability

Availability concerns how long it has been since the monitor received interface activity qualifying under the final interface requirements.

It is based on receive time.

A recently received stale observation may therefore imply:

- recent interface activity;
- stale navigation state.

The monitor must be capable of representing both facts simultaneously.

---

## 26. Recovery Semantics

Phase 0 does not yet assign recovery behavior to every fault.

However, recovery shall always be explicit.

No condition shall recover merely because a later code path happens to overwrite earlier state.

Requirements must define, for each recoverable condition:

- recovery trigger;
- required number or duration of valid observations, if any;
- whether persistence history is retained;
- whether reason history is retained;
- whether monitor operating state changes;
- whether a latched simulated action prevents recovery.

---

## 27. Numerical Semantics

Physical inputs must be finite.

Configuration bounds must be finite.

No general-purpose comparison epsilon is defined.

Exact-boundary semantics use the represented numeric values directly.

If later numerical analysis demonstrates that tolerances are needed for a specific computation, the tolerance shall:

- have a documented rationale;
- apply only to the relevant computation;
- be configuration-controlled or explicitly constant by requirement;
- have dedicated boundary tests.

---

## 28. Determinism Contract

The deterministic contract for the core monitor is:

> Given the same supported software revision, validated configuration, initial execution state, ordered event sequence, event payloads, and modeled-time values, the monitor shall produce the same sequence of observable decision records.

This contract excludes:

- log formatting timestamps generated outside the core;
- host execution duration;
- filesystem metadata;
- CI job identifiers;
- other non-semantic artifact metadata.

Verification artifacts must distinguish semantic outputs from incidental execution metadata.

---

## 29. Semantic Invariants

The following invariants are proposed for later requirement derivation.

### INV-001

Modeled time processed by the monitor never moves backward within one execution.

### INV-002

Rejected navigation observations never silently replace the last accepted navigation state.

### INV-003

Core decisions never depend on wall-clock time.

### INV-004

Core decisions never depend on logging success or failure.

### INV-005

Configuration does not change during one initialized execution.

### INV-006

Every accepted runtime event produces exactly one logical runtime evaluation, and every such evaluation produces exactly one logical decision record.

### INV-007

Every non-finite physical input value is invalid.

### INV-008

An exact synthetic envelope boundary point is inside the envelope.

### INV-009

Persistent envelope behavior is based on modeled elapsed time rather than packet count alone.

### INV-010

A latched simulated action, once established, persists until explicit reset/reinitialization.

### INV-011

Navigation freshness and interface availability are represented as distinct concepts.

### INV-012

Multiple simultaneous abnormal conditions may coexist and remain observable.

---

## 30. Questions Still Open After This Increment

This semantic model intentionally does not resolve every requirement-level question.

The following remain open:

1. Which exact observations qualify as interface activity for timeout purposes?
2. What conditions cause monitor degradation?
3. Does a `DEGRADED` operating state actually need to exist?
4. What valid information is required before normal monitoring can begin?
5. What exact conditions cause `SIMULATED_SAFETY_ACTION_REQUEST`?
6. Which conditions are allowed to latch that action?
7. What recovery rules apply to invalid navigation?
8. What recovery rules apply to interface timeout?
9. How do sequence-number conflicts and timestamp conflicts interact?
10. What happens to an active persistence interval when usable navigation is lost?
11. Which reason code takes primary precedence when multiple conditions coexist?
12. What monitoring-state transitions are permitted?
13. Are decision records required for initialization/reset, or only runtime evaluations?
14. What numeric representation and integer widths will the interface use?
15. What configuration ranges are valid?
16. What exact threshold values will synthetic verification configurations use?
17. How shall rejected runtime events be represented in external evidence: as a decision-record form or as a separate interface-level diagnostic record?

These are suitable inputs to requirements and hazard-analysis work.

---

## 31. Critical Review

### 31.1 Three-dimensional state is sufficient but intentionally artificial

A 3D Cartesian model introduces one more dimension than strictly necessary for state-machine testing.

However, it provides realistic vector-shaped interfaces without introducing geodesy or aerospace-coordinate complexity.

This is justified as long as the geometry remains synthetic and axis-aligned.

### 31.2 Explicit evaluation ticks add an event concept but remove hidden timing

A simpler system could evaluate only on observation arrival.

That would make deterministic timeout detection impossible when observations stop completely.

The explicit tick is therefore justified despite adding one semantic event type.

### 31.3 Separate source and receive time is necessary

Using one timestamp for both would prevent the project from distinguishing:

- delayed old data;
- fresh data delivered recently;
- interface silence.

Because those are distinct failure modes, two times are justified.

### 31.4 Elapsed-time persistence is preferable to packet-count persistence

Packet-count persistence is simpler but makes behavior depend on source update rate.

Elapsed modeled time better represents the intended semantic concept and remains deterministic.

### 31.5 The latch-reset rule is intentionally strong

Defining latch persistence now removes ambiguity about what "latched" means.

The model still does not define which failures are severe enough to create a latch.

That decision remains requirement/hazard driven.

### 31.6 The state machine is intentionally not defined yet

The semantic model establishes information and evaluation behavior first.

Deriving states afterward reduces the risk of inventing states based on familiar terminology rather than behavioral necessity.

### 31.7 Reason precedence remains unresolved

A stable output ordering is required for determinism, but inventing primary-reason precedence before requirements exist would be premature.

The project should define precedence only when the abnormal-condition taxonomy is baselined.

---

## 32. Phase 0 Increment Exit Criteria

This semantic-model increment is acceptable when engineering review agrees that:

- the information model is sufficient to express initial scenarios;
- source, receive, evaluation, and processing time are not conflated;
- time can advance without navigation arrival;
- navigation freshness and interface availability are distinct;
- the initial envelope has unambiguous boundary semantics;
- persistence is independent of packet rate;
- projection is simple and independently testable;
- rejected observations cannot silently become accepted state;
- decisions can preserve multiple simultaneous reasons;
- no monitor-state names have been adopted without behavioral justification;
- no operational aerospace behavior has entered the model;
- unresolved requirement-level decisions remain explicitly visible.

If these criteria are met, the project may proceed to Phase 1 requirements derivation and preliminary hazard analysis without first writing implementation code.

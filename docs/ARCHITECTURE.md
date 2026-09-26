# Flight Safety Monitor & Verification Platform

## Phase 2A — Logical Architecture and Responsibility Boundaries

**Status:** Draft for engineering review
**Lifecycle phase:** Phase 2 — Architecture
**Scope:** Logical software architecture derived from the approved Phase 0 semantics and synchronized Phase 1 requirements, hazard analysis, and failure policy

---

## 1. Purpose

This document defines the logical architecture of the synthetic Flight Safety Monitor & Verification Platform.

The architecture exists to allocate already-approved system behavior to explicit software responsibilities before implementation begins.

It does not introduce new operational aerospace behavior.

The system remains:

- synthetic;
- deterministic;
- replayable;
- non-operational;
- software-only.

The only simulated action remains:

`SIMULATED_SAFETY_ACTION_REQUEST`

No component defined here commands real hardware or a destructive or hazardous mechanism.

---

## 2. Architectural Principles

### 2.1 Requirements own behavior

Architecture allocates behavior.

It does not silently redefine system requirements, failure policy, timing semantics, recovery rules, or hazard mitigations.

When architecture and an approved requirement disagree, the requirement takes precedence until the discrepancy is resolved through change control.

### 2.2 One authoritative owner per mutable state concept

A mutable state concept shall have one logical owner.

Examples include:

- last accepted runtime-event time;
- last accepted navigation state;
- last qualifying interface receive time;
- active violation interval;
- simulated-action latch.

Derived values may be recomputed elsewhere only when they cannot become an independent conflicting source of truth.

### 2.3 Modeled time is explicit

Host wall-clock time and host execution duration are not part of monitor decision semantics.

All decision-relevant time is supplied through the modeled scenario-time domain defined by the approved system semantics.

### 2.4 Evaluation is deterministic

For identical:

- immutable configuration;
- initial state;
- accepted event sequence;
- modeled event times;
- navigation observations;

the core monitor shall produce identical logical state transitions and evidence.

### 2.5 Diagnostics do not control decisions

Logging, presentation, serialization, and external evidence sinks shall observe monitor results.

They shall not alter:

- event acceptance;
- navigation acceptance;
- freshness;
- interface timeout;
- eligibility;
- envelope classification;
- persistence;
- projection;
- latch behavior.

### 2.6 Rejection is not normal evaluation

A runtime event rejected before normal evaluation does not enter the normal monitoring path and does not create a normal decision record.

Navigation-observation rejection inside an accepted navigation-arrival runtime event is a different condition.

### 2.7 Architecture shall remain minimal

The initial architecture shall not introduce:

- distributed services;
- network protocols;
- databases;
- message brokers;
- dependency-injection frameworks;
- dynamic plugin systems;
- unnecessary inheritance hierarchies;
- concurrency requirements;
- a ceremonial monitor operating-state machine.

Such mechanisms require an explicit requirement or demonstrated engineering need.

---

## 3. System Context

The logical system receives:

1. immutable synthetic monitor configuration;
2. initialization/reset control;
3. navigation-arrival runtime events;
4. evaluation-tick runtime events.

The logical system produces:

1. normal runtime decision evidence for accepted runtime evaluation events;
2. runtime-event rejection evidence for runtime events rejected before normal evaluation;
3. the current simulated-action latch state.

The logical core does not own:

- scenario generation;
- human-readable report formatting;
- persistent external storage;
- host scheduling;
- wall-clock acquisition;
- real vehicle interfaces.

---

## 4. Logical Component Model

The initial logical architecture contains seven primary responsibilities:

1. **Configuration Validator**
2. **Runtime Event Gate**
3. **Navigation State Manager**
4. **Monitoring Evaluator**
5. **Persistence and Action Manager**
6. **Decision Evidence Builder**
7. **Monitor Coordinator**

These are logical responsibilities.

They are not yet mandates for seven C++ classes or seven translation units.

Implementation structure shall be decided only after responsibility boundaries are reviewed.

---

## 5. Configuration Validator

### 5.1 Responsibility

The Configuration Validator determines whether a complete monitor configuration is valid before monitoring begins.

It owns validation rules for configuration relationships that have been baselined.

### 5.2 Inputs

- candidate immutable configuration.

### 5.3 Outputs

Either:

- validated immutable configuration;

or:

- a configuration-validation failure result that prevents monitoring from beginning.

The concrete external representation of configuration-validation failure remains later interface-design work.

### 5.4 Prohibited responsibilities

The Configuration Validator shall not:

- evaluate runtime events;
- own navigation state;
- classify envelope position;
- accumulate persistence;
- mutate configuration after monitoring begins.

---

## 6. Runtime Event Gate

### 6.1 Responsibility

The Runtime Event Gate owns acceptance of runtime-event ordering.

It determines whether a supplied runtime event may enter normal runtime evaluation.

### 6.2 Authoritative state

The Runtime Event Gate owns one conceptual optional value:

- last accepted runtime-event modeled time.

Absence of that value means no runtime event has yet been accepted.

The architecture shall not require a second independently writable boolean representing the same fact.

### 6.3 Acceptance rule

A runtime event whose modeled event time is earlier than the last accepted runtime-event time is rejected before normal evaluation.

Equal modeled event time is not rejected solely for equality.

### 6.4 Accepted-event behavior

For an accepted runtime event, the gate permits exactly one logical runtime evaluation.

### 6.5 Rejected-event behavior

For a rejected runtime event, the gate:

- does not advance last accepted runtime-event time;
- does not permit normal safety evaluation;
- does not permit normal monitor-state mutation;
- does not count the event as qualifying interface activity;
- requests runtime-event rejection evidence.

### 6.6 Prohibited responsibilities

The Runtime Event Gate shall not decide:

- whether a navigation observation replaces accepted navigation state;
- freshness;
- envelope classification;
- persistence;
- projection;
- simulated-action creation.

---

## 7. Navigation State Manager

### 7.1 Responsibility

The Navigation State Manager owns accepted navigation state and navigation-arrival evidence relevant to interface activity.

### 7.2 Authoritative state

It owns two conceptual optional values:

- last accepted navigation observation, including that observation's sequence number and source timestamp;
- last qualifying navigation-interface receive time.

Absence of the first means no navigation observation has yet been accepted.

Absence of the second means no qualifying interface activity has yet occurred.

Sequence number, source timestamp, and the corresponding existence facts shall be read from these authoritative values rather than maintained as independently writable duplicate state.

### 7.3 Qualifying interface activity

An accepted runtime event recognized as a navigation-arrival event counts as qualifying interface activity even when the contained navigation observation is not eligible to replace accepted navigation state.

Unrecognized malformed transport is outside the deterministic navigation-arrival path and does not qualify unless later requirements define otherwise.

### 7.4 Navigation-state replacement

A contained navigation observation may replace accepted state only when all applicable acceptance conditions are satisfied.

These include:

- finite required physical values;
- navigation validity `VALID`;
- source health not `INVALID`;
- source timestamp not later than event receive time;
- strictly increasing source timestamp when prior accepted navigation exists;
- strictly increasing sequence number when prior accepted navigation exists.

A forward sequence gap is observable but does not by itself prohibit acceptance.

### 7.5 Failed observation acceptance

A rejected observation:

- does not replace accepted navigation state;
- does not create a separate recovery latch;
- may coexist with qualifying interface activity when its containing runtime event is accepted.

### 7.6 Navigation transition view

Processing a navigation-arrival event shall preserve enough event-local information to distinguish:

- the accepted navigation state immediately before the event;
- whether the contained observation was accepted for state replacement;
- the accepted navigation state immediately after observation processing.

The prior accepted state is not a second authoritative state owner.

It is an immutable event-local snapshot used so downstream evaluation can determine whether previously active persistence lost action eligibility during modeled time before the newly accepted observation arrived.

This transition view is especially required when the prior accepted state would have become stale between runtime events.

### 7.7 Prohibited responsibilities

The Navigation State Manager shall not:

- determine current envelope classification;
- accumulate persistence;
- create the simulated-action latch;
- define reason ordering.

---

## 8. Monitoring Evaluator

### 8.1 Responsibility

The Monitoring Evaluator derives monitoring facts for one accepted runtime evaluation from:

- immutable configuration;
- modeled evaluation time;
- current accepted navigation state;
- current interface-activity state.

It does not own long-lived persistence or latch state.

### 8.2 Pre-update continuity derivation

For a navigation-arrival event that may replace accepted navigation state, the Monitoring Evaluator shall receive the immutable pre-event accepted-navigation snapshot when one exists.

Using that snapshot and the current modeled evaluation time, it shall derive whether the previously accepted navigation remained action eligible continuously through the elapsed modeled interval before the new observation arrived.

This continuity derivation occurs independently of whether the newly arrived observation is accepted.

If the prior state would have become stale before the current event time, the derived continuity result shall identify that the prior action-eligible interval was interrupted at the freshness boundary.

The pre-event snapshot is event-local evidence only and does not become a second mutable navigation state.

### 8.3 Derived navigation age

When accepted navigation exists:

`navigation_age = evaluation_time - accepted_source_time`

Accepted navigation source time cannot be later than its receive time, preventing accepted future-dated navigation from producing negative age under the modeled-time policy.

### 8.4 Freshness

Navigation is:

- fresh when age is less than or equal to the configured freshness limit;
- stale when age is greater than the configured freshness limit.

No accepted navigation is distinct from stale navigation.

### 8.5 Interface state

After qualifying interface activity exists:

`interface_silence = evaluation_time - last_qualifying_interface_receive_time`

Interface state is:

- available when silence is less than or equal to the configured timeout limit;
- timed out when silence is greater than the configured timeout limit.

Before any qualifying interface activity, interface state is `NOT_YET_OBSERVED`, not timed out.

### 8.6 Current-state evaluation eligibility

Current-state envelope evaluation is eligible only when accepted navigation:

- is fresh;
- reports validity `VALID`;
- reports source health `HEALTHY` or `DEGRADED`;
- contains required finite physical values.

### 8.7 Action eligibility

Action eligibility additionally requires source health `HEALTHY`.

`DEGRADED` state may therefore support current-state classification while remaining ineligible for persistence, projection, or new action authority.

### 8.8 Interface timeout independence

Interface timeout does not by itself invalidate otherwise fresh accepted navigation.

Navigation eligibility and interface availability remain separate derived facts.

### 8.9 Envelope classification

When current-state evaluation is eligible:

- exact boundary is `INSIDE`;
- only strict bound exceedance is `OUTSIDE`.

When current-state evaluation is not eligible:

- result is `NOT_EVALUATED`.

### 8.10 Projection

Projection uses the approved constant-velocity model only when projection eligibility is satisfied.

Projected classification uses the same synthetic envelope boundary semantics as current-state classification.

Projection is advisory.

If otherwise-eligible constant-velocity projection arithmetic produces a
non-finite projected Cartesian position component:

- the projected envelope result is `NOT_EVALUATED`;
- the condition is supplied to evidence construction through a stable reason;
- no projected `INSIDE` or `OUTSIDE` classification is produced from that
  failed calculation.

Projected `OUTSIDE` and a non-finite failed projection do not:

- accumulate current-state violation persistence;
- independently create a simulated-action latch.

### 8.11 Prohibited responsibilities

The Monitoring Evaluator shall not:

- mutate accepted navigation;
- mutate runtime-event ordering state;
- own the action latch;
- serialize external evidence.

---

## 9. Persistence and Action Manager

### 9.1 Responsibility

The Persistence and Action Manager owns state that must persist across accepted runtime evaluations for:

- current-envelope violation intervals;
- the simulated-action latch.

### 9.2 Authoritative state

It owns:

- whether an action-eligible outside interval is active;
- start modeled time of the active interval;
- identity/reference of the accepted observation that started the interval;
- whether `SIMULATED_SAFETY_ACTION_REQUEST` is latched.

### 9.3 Interval start

A new violation interval begins when an action-eligible evaluation first classifies current position as `OUTSIDE`.

### 9.4 Interval continuation

The interval may remain active while the previously accepted navigation state remains action eligible and `OUTSIDE`.

Evaluation ticks may advance elapsed modeled interval age.

Ticks alone cannot confirm persistent violation.

### 9.5 Interruption

Loss of action eligibility ends the active interval.

This includes modeled staleness even when no runtime event occurs at the exact freshness boundary.

At the next processed event, continuity shall be evaluated across the elapsed modeled interval.

If the previous accepted navigation would have become stale before the later event, the previous interval is treated as interrupted at that freshness boundary.

For a navigation-arrival event, this interruption decision shall be applied from the pre-update continuity result before the newly accepted navigation state is allowed to continue or begin a violation interval.

A fresh replacement observation therefore cannot erase or retroactively bridge a stale period that occurred before its arrival.

### 9.6 No interval resurrection

A later action-eligible outside observation after interruption begins a new interval.

Prior elapsed duration does not carry forward.

### 9.7 Persistence confirmation

A violation becomes persistent for simulated-action purposes only when:

- an active action-eligible outside interval exists;
- elapsed modeled duration is greater than or equal to the configured persistence duration;
- a newer accepted navigation observation than the interval-start observation exists;
- that observation is evaluated at or after the persistence boundary;
- that observation remains action eligible;
- that observation classifies current position as `OUTSIDE`.

Exact equality at the persistence boundary qualifies.

### 9.8 Action creation

The only initial condition permitted to create a new `SIMULATED_SAFETY_ACTION_REQUEST` latch is the confirmed persistent current-state violation defined above.

The following do not independently create the latch:

- stale navigation;
- no accepted navigation;
- interface timeout;
- rejected observation;
- invalid observation;
- duplicate observation;
- reordered observation;
- sequence gap;
- source health `DEGRADED`;
- source health `INVALID`;
- projected envelope violation;
- logging failure;
- rejected runtime event.

### 9.9 Latch persistence

Once latched, the simulated action remains latched through later monitoring results.

Only explicit reset/reinitialization clears execution-specific latch state.

### 9.10 Prohibited responsibilities

The Persistence and Action Manager shall not:

- accept runtime events;
- accept navigation observations;
- perform external serialization;
- use host wall-clock duration.

---

## 10. Decision Evidence Builder

### 10.1 Responsibility

The Decision Evidence Builder constructs externally visible logical evidence from already-determined monitor facts and state transitions.

### 10.2 Normal runtime decision evidence

Every accepted runtime evaluation event produces exactly one logical normal decision record.

The record shall contain enough evidence to reconstruct the logical decision according to the approved requirements.

### 10.3 Multiple simultaneous reasons

All applicable reason identifiers remain observable.

The initial architecture does not designate one reason as the authoritative primary reason.

External reason ordering shall be deterministic once the stable reason catalog is defined.

### 10.4 Runtime-event rejection evidence

A runtime event rejected before normal evaluation produces rejection evidence rather than a normal decision record.

Rejection evidence shall identify, directly or by stable reference:

- rejected event category;
- supplied event time;
- last accepted runtime-event time;
- rejection reason;
- execution/configuration identity when available.

### 10.5 Prohibited responsibilities

The evidence layer shall not alter monitor behavior.

Evidence generation failure must not become an alternate path that changes:

- state acceptance;
- persistence;
- eligibility;
- latch state.

The exact response to evidence-output failure remains a later architecture/requirements question.

---

## 11. Monitor Coordinator

### 11.1 Responsibility

The Monitor Coordinator sequences one logical event through the approved responsibilities.

It coordinates behavior but should not become a second owner of component state.

### 11.2 State ownership

The Monitor Coordinator owns no independent copy of domain state already owned by another logical responsibility.

Its responsibility is sequencing and orchestration.

### 11.3 Navigation-arrival processing

For a navigation-arrival event:

1. submit runtime-event time to Runtime Event Gate;
2. if rejected, request runtime-event rejection evidence and stop normal processing;
3. capture an immutable event-local snapshot of the accepted navigation state that existed before processing this arrival, if any;
4. record qualifying interface activity for the accepted recognized navigation-arrival event;
5. evaluate the contained navigation observation for state replacement and retain whether replacement occurred;
6. establish the post-processing accepted-navigation snapshot;
7. derive pre-update continuity from the prior snapshot and current modeled evaluation time;
8. derive current monitoring facts from the post-processing accepted-navigation snapshot;
9. apply any persistence interruption implied by pre-update continuity before allowing the post-processing state to continue or begin a violation interval;
10. update persistence and simulated-action state using the current evaluation facts;
11. build exactly one normal decision record.

The pre-event and post-event snapshots are immutable views for one logical evaluation.

Only the Navigation State Manager owns the authoritative accepted navigation state.

### 11.4 Evaluation-tick processing

For an evaluation tick:

1. submit tick time to Runtime Event Gate;
2. if rejected, request runtime-event rejection evidence and stop normal processing;
3. retain accepted navigation state unchanged;
4. derive monitoring facts at tick evaluation time;
5. update persistence state, including interruption if freshness was lost across the elapsed interval;
6. do not permit the tick alone to confirm persistent violation;
7. build exactly one normal decision record.

### 11.5 Initialization, reinitialization, and explicit reset

A newly initialized execution begins with fresh execution-specific state and an immutable validated configuration.

Reinitialization therefore establishes a new execution boundary rather than mutating selected state inside the prior execution.

Configuration replacement occurs only through such a new initialized execution.

For an explicit reset within an execution, the simulated-action latch is cleared while accepted navigation, runtime-event ordering history, interface-activity history, and an active persistence interval remain unchanged solely because of reset.

Reset does not itself interrupt an otherwise continuous violation interval.

It establishes a confirmation boundary for subsequent latch creation: pre-reset evidence and evaluation ticks cannot by themselves create a new latch.

A later latch requires newly accepted post-reset navigation evidence plus all otherwise applicable persistent-violation and action-eligibility conditions.

If violation continuity is lost after reset, the normal persistence-interruption and interval-restart rules apply.

---

## 12. Logical Data Flow

```text
Immutable Configuration
        |
        v
Configuration Validator
        |
        v
Validated Configuration
        |
        +----------------------------------------------+
                                                       |
Runtime Event                                          |
     |                                                 |
     v                                                 |
Runtime Event Gate                                     |
     |                                                 |
     +---- rejected ------------------------------+    |
     |                                            |    |
     |                                            v    |
     |                              Rejection Evidence  |
     |                                                 |
     v                                                 |
Accepted Runtime Event                                 |
     |                                                 |
     +---- navigation arrival ---> Navigation State Manager
     |                                  |
     |                                  v
     |                           Accepted State Snapshot
     |                                  |
     +----------------------------------+
                                        |
                                        v
                              Monitoring Evaluator
                                        |
                                        v
                              Derived Monitoring Facts
                                        |
                                        v
                           Persistence & Action Manager
                                        |
                                        v
                            Updated Persistent State
                                        |
                                        v
                           Decision Evidence Builder
                                        |
                                        v
                              Normal Decision Record
```

This diagram describes logical dependency and event flow, not thread or process topology.

---

## 13. State Ownership Matrix

| State / fact | Logical owner | Persistent across evaluations? |
|---|---|---:|
| Validated configuration | initialized execution context | yes, immutable |
| Optional last accepted runtime-event time | Runtime Event Gate | yes |
| Optional last accepted navigation observation, including sequence and source time | Navigation State Manager | yes |
| Optional last qualifying interface receive time | Navigation State Manager | yes |
| Navigation age | Monitoring Evaluator | no, derived |
| Navigation freshness | Monitoring Evaluator | no, derived |
| Interface silence | Monitoring Evaluator | no, derived |
| Interface availability | Monitoring Evaluator | no, derived |
| Current envelope result | Monitoring Evaluator | no, derived |
| Projected envelope result | Monitoring Evaluator | no, derived |
| Active violation interval | Persistence and Action Manager | yes |
| Violation interval start time | Persistence and Action Manager | yes |
| Simulated-action latch | Persistence and Action Manager | yes |
| Normal decision record | Decision Evidence Builder | output |
| Runtime-event rejection evidence | Decision Evidence Builder | output |

No implementation component should duplicate authoritative ownership of persistent state listed above.

---

## 14. Dependency Direction

The intended logical dependency direction is:

```text
Configuration
     |
     v
Pure domain calculations
     ^
     |
State-owning monitor responsibilities
     ^
     |
Monitor Coordinator
     |
     v
Evidence / adapters / presentation
```

More concretely:

- core decision logic shall not depend on CLI code;
- core decision logic shall not depend on JSON/YAML serialization libraries;
- core decision logic shall not depend on filesystem logging;
- core decision logic shall not depend on Python simulation code;
- evidence formatting may depend on core domain types;
- simulation/adapters may depend on the public core interface.

Dependency direction shall make the deterministic monitor core independently testable.

---

## 15. Deterministic Replay Boundary

The deterministic replay boundary contains:

- validated immutable configuration;
- runtime-event ordering state;
- accepted navigation state;
- interface-activity state;
- monitoring calculations;
- persistence state;
- simulated-action latch;
- logical evidence construction.

Replay must not require:

- wall-clock timestamps;
- thread scheduling;
- network timing;
- filesystem timing;
- random values not explicitly supplied as scenario input.

A recorded synthetic event sequence plus configuration shall therefore be sufficient to reproduce logical monitor behavior.

---

## 16. Pure Calculations Versus Stateful Responsibilities

The architecture should prefer pure functions for calculations that do not require historical ownership.

Likely pure calculations include:

- finite-value validation;
- pre-update action-eligibility continuity over modeled time;
- navigation age;
- interface silence;
- freshness classification;
- interface-timeout classification;
- current envelope geometry;
- constant-velocity projected position;
- projected envelope geometry;
- eligibility derivation from an immutable snapshot.

Stateful logic should be limited to behavior that genuinely depends on history:

- runtime-event monotonicity;
- accepted navigation replacement;
- last qualifying interface activity;
- persistence interval history;
- simulated-action latch.

This separation reduces hidden state and improves unit-test precision.

---

## 17. Initialization and Reset Boundary

### 17.1 New initialized execution

A new initialized execution begins with fresh execution-specific state across the logical state owners and a validated immutable configuration for that execution.

No state from the prior execution is implicitly carried into the new execution unless a later requirement explicitly defines cross-execution evidence retention outside the deterministic monitor state.

### 17.2 Explicit reset within an execution

Explicit reset clears the simulated-action latch inside the current execution.

It does not by itself clear or replace:

- accepted navigation state;
- runtime-event ordering history;
- interface-activity history;
- an active persistence interval.

An otherwise continuous active violation interval therefore remains active across reset.

Reset establishes a post-reset confirmation boundary for new latch creation.

Pre-reset navigation evidence and evaluation ticks cannot by themselves recreate the latch.

A new latch requires navigation evidence accepted after reset and all otherwise applicable persistent-violation and action-eligibility conditions.

If action eligibility or continuity is later lost, the normal persistence-interruption and interval-restart rules apply.

Reinitialization remains distinct: it establishes a new execution with fresh execution-specific state.

---

## 18. Error and Evidence Boundaries

The architecture distinguishes at least four categories:

### 18.1 Configuration rejection

Occurs before monitoring execution begins.

### 18.2 Runtime-event rejection

Occurs before normal runtime evaluation.

Produces rejection evidence, not a normal decision record.

### 18.3 Navigation-observation rejection

Occurs inside an accepted navigation-arrival runtime event.

The accepted runtime event still receives one normal runtime evaluation and decision record.

### 18.4 Monitoring condition

Examples include:

- stale navigation;
- interface timeout;
- degraded source health;
- current `OUTSIDE`;
- projected `OUTSIDE`.

These are evaluated facts, not parser/runtime-event rejection mechanisms.

Conflating these categories is an architectural defect.

---

## 19. Explicitly Prohibited Couplings

The initial architecture shall avoid the following:

### 19.1 Navigation manager controlling the action latch

Navigation acceptance provides state evidence.

It does not own action policy.

### 19.2 Evidence code mutating monitor state

Serialization or logging must not become a hidden state-transition path.

### 19.3 Projection controlling current-state persistence

Projected position is advisory and cannot feed the current violation interval.

### 19.4 Interface timeout clearing navigation state automatically

Interface availability and navigation freshness are separate dimensions.

### 19.5 Runtime-event rejection entering the normal decision path

Rejected runtime events do not produce normal runtime evaluations.

### 19.6 Tick events becoming synthetic navigation observations

Ticks advance evaluation time.

They do not create newer vehicle-state evidence and cannot confirm persistent violation.

### 19.7 Host duration controlling modeled safety time

No timeout or persistence behavior may depend on how long the program takes to execute.

### 19.8 Navigation replacement erasing prior continuity

A newly accepted navigation observation shall not make the immediately preceding accepted state unavailable to the event-local continuity calculation.

The architecture shall determine whether the prior state became stale before applying the new state to persistence continuation.

---

## 20. Architecture-Driving Hazards

The following hazard themes materially shape the architecture:

- stale navigation treated as current;
- invalid or temporally inconsistent navigation accepted;
- interface timeout not independently observable;
- regressive runtime time mutating state;
- persistence depending on event/packet rate;
- persistence bridging stale evidence gaps;
- ticks falsely confirming persistent violation;
- projection using unusable state;
- non-finite projection arithmetic being treated as valid envelope evidence;
- degraded or advisory information creating action authority;
- latch clearing unintentionally;
- simultaneous reasons losing evidence;
- invalid configuration entering monitoring;
- rejected runtime events appearing as normal decisions;
- diagnostic behavior altering monitoring semantics.

The architecture responds by separating:

- runtime-event acceptance;
- navigation acceptance;
- immutable pre-update/post-update navigation views for one evaluation;
- derived monitoring facts;
- historical persistence/action state;
- evidence construction.

---

## 21. Requirements Allocation and Traceability

Each baselined system requirement has exactly one **primary allocation**.

A secondary participant may consume an output, provide derived information, or participate in orchestration, but it does not become a second authoritative owner of the primary requirement behavior.

Three allocation categories below are not additional stateful components:

- **Initialized execution context** represents immutable execution-wide configuration ownership.
- **Cross-cutting deterministic-core constraint** represents a property that several core responsibilities must jointly preserve.
- **System boundary / external architectural constraint** represents a system-level boundary that must not be incorrectly assigned to an internal state owner.

| Requirement | Requirement title | Primary allocation | Secondary participant(s) |
|---|---|---|---|
| FSS-SYS-001 | Configuration validation before monitoring | Configuration Validator | Monitor Coordinator |
| FSS-SYS-002 | Invalid configuration blocks monitoring | Configuration Validator | Monitor Coordinator |
| FSS-SYS-003 | Configuration immutability during execution | Initialized execution context | Configuration Validator |
| FSS-SYS-004 | Configuration identity in evidence | Decision Evidence Builder | Initialized execution context |
| FSS-SYS-005 | Explicit modeled time | Cross-cutting deterministic-core constraint | Runtime Event Gate; Monitoring Evaluator; Persistence and Action Manager |
| FSS-SYS-006 | Monotonic runtime-event time | Runtime Event Gate | Monitor Coordinator |
| FSS-SYS-007 | Regressive-time event rejection | Runtime Event Gate | Decision Evidence Builder |
| FSS-SYS-008 | Regressive-time rejection preserves state | Runtime Event Gate | Monitor Coordinator |
| FSS-SYS-009 | Time may advance without navigation arrival | Monitor Coordinator | Monitoring Evaluator |
| FSS-SYS-010 | One evaluation per accepted runtime event | Monitor Coordinator | Runtime Event Gate |
| FSS-SYS-011 | One decision record per runtime evaluation | Decision Evidence Builder | Monitor Coordinator |
| FSS-SYS-012 | Runtime-event acceptance distinct from observation acceptance | Monitor Coordinator | Runtime Event Gate; Navigation State Manager |
| FSS-SYS-013 | Non-finite physical values invalid | Navigation State Manager | — |
| FSS-SYS-014 | Rejected observation cannot replace accepted state | Navigation State Manager | — |
| FSS-SYS-015 | Strict source-time progress for newer accepted state | Navigation State Manager | — |
| FSS-SYS-016 | Duplicate sequence number is not newer state | Navigation State Manager | — |
| FSS-SYS-017 | Regressive sequence number is not newer state | Navigation State Manager | — |
| FSS-SYS-018 | Sequence gaps are observable | Navigation State Manager | Decision Evidence Builder |
| FSS-SYS-019 | Navigation age basis | Monitoring Evaluator | Navigation State Manager |
| FSS-SYS-020 | Interface-silence basis | Monitoring Evaluator | Navigation State Manager |
| FSS-SYS-021 | Freshness and availability remain distinct | Monitoring Evaluator | — |
| FSS-SYS-022 | Initial envelope form | Monitoring Evaluator | Initialized execution context |
| FSS-SYS-023 | Envelope configuration ordering | Configuration Validator | — |
| FSS-SYS-024 | Exact boundary is inside | Monitoring Evaluator | — |
| FSS-SYS-025 | Outside-envelope classification | Monitoring Evaluator | — |
| FSS-SYS-026 | Persistence based on modeled elapsed time | Persistence and Action Manager | Monitoring Evaluator |
| FSS-SYS-027 | Transient return inside ends violation interval | Persistence and Action Manager | Monitoring Evaluator |
| FSS-SYS-028 | Constant-velocity projection | Monitoring Evaluator | — |
| FSS-SYS-029 | Projection requires eligible accepted state | Monitoring Evaluator | Navigation State Manager |
| FSS-SYS-030 | Projected position uses envelope semantics | Monitoring Evaluator | — |
| FSS-SYS-031 | Stable machine-readable reason identifiers | Decision Evidence Builder | — |
| FSS-SYS-032 | Multiple simultaneous conditions observable | Decision Evidence Builder | Monitoring Evaluator; Persistence and Action Manager |
| FSS-SYS-033 | Deterministic reason ordering | Decision Evidence Builder | — |
| FSS-SYS-034 | Decision record reconstructability | Decision Evidence Builder | All core state owners |
| FSS-SYS-035 | Simulated action is non-operational | System boundary / external architectural constraint | Persistence and Action Manager |
| FSS-SYS-036 | Latched simulated action persists | Persistence and Action Manager | — |
| FSS-SYS-037 | Reset clears execution-specific latch state | Persistence and Action Manager | Monitor Coordinator |
| FSS-SYS-038 | Deterministic decision sequence | Cross-cutting deterministic-core constraint | Monitor Coordinator; Decision Evidence Builder |
| FSS-SYS-039 | Logging does not affect decision semantics | Cross-cutting deterministic-core constraint | Decision Evidence Builder; external adapters |
| FSS-SYS-040 | Host execution duration does not define modeled safety time | Cross-cutting deterministic-core constraint | Runtime Event Gate; Monitoring Evaluator; Persistence and Action Manager |
| FSS-SYS-041 | Qualifying navigation interface activity | Navigation State Manager | Monitor Coordinator |
| FSS-SYS-042 | Observation rejection does not cancel interface activity | Navigation State Manager | Monitor Coordinator |
| FSS-SYS-043 | Unrecognized malformed transport is not qualifying activity | System boundary / external architectural constraint | Navigation State Manager interface boundary |
| FSS-SYS-044 | Future-dated navigation state rejected | Navigation State Manager | — |
| FSS-SYS-045 | Both ordering fields shall progress | Navigation State Manager | — |
| FSS-SYS-046 | Current-state evaluation eligibility | Monitoring Evaluator | Navigation State Manager |
| FSS-SYS-047 | Action eligibility requires healthy source state | Monitoring Evaluator | Persistence and Action Manager |
| FSS-SYS-048 | Projection eligibility | Monitoring Evaluator | — |
| FSS-SYS-049 | Invalid source health cannot replace accepted state | Navigation State Manager | — |
| FSS-SYS-050 | Navigation freshness boundary | Monitoring Evaluator | — |
| FSS-SYS-051 | Stale navigation restrictions | Monitoring Evaluator | Persistence and Action Manager |
| FSS-SYS-052 | Recovery from stale navigation | Monitoring Evaluator | Navigation State Manager |
| FSS-SYS-053 | Interface timeout boundary | Monitoring Evaluator | Navigation State Manager |
| FSS-SYS-054 | Interface state before first qualifying activity | Monitoring Evaluator | Navigation State Manager |
| FSS-SYS-055 | Interface timeout does not independently invalidate fresh state | Monitoring Evaluator | — |
| FSS-SYS-056 | Interface recovery is independent of navigation recovery | Monitoring Evaluator | Navigation State Manager |
| FSS-SYS-057 | Not-evaluated envelope result | Monitoring Evaluator | — |
| FSS-SYS-058 | Violation interval start | Persistence and Action Manager | Monitoring Evaluator |
| FSS-SYS-059 | Loss of action eligibility interrupts persistence | Persistence and Action Manager | Monitoring Evaluator |
| FSS-SYS-060 | Staleness between runtime events interrupts persistence | Persistence and Action Manager | Monitoring Evaluator |
| FSS-SYS-061 | Interrupted persistence does not resume | Persistence and Action Manager | — |
| FSS-SYS-062 | Confirming navigation required for persistent violation | Persistence and Action Manager | Navigation State Manager; Monitoring Evaluator |
| FSS-SYS-063 | Persistent-violation equality boundary | Persistence and Action Manager | — |
| FSS-SYS-064 | Projection is advisory | Monitoring Evaluator | Persistence and Action Manager |
| FSS-SYS-065 | Sole initial simulated-action trigger | Persistence and Action Manager | Monitoring Evaluator |
| FSS-SYS-066 | Non-triggering abnormal conditions | Persistence and Action Manager | Monitoring Evaluator; Navigation State Manager |
| FSS-SYS-067 | Rejected runtime-event evidence is separate | Decision Evidence Builder | Runtime Event Gate |
| FSS-SYS-068 | Rejected runtime-event evidence content | Decision Evidence Builder | Runtime Event Gate; Initialized execution context |
| FSS-SYS-069 | No required primary reason | Decision Evidence Builder | — |
| FSS-SYS-070 | Recovery after rejected observation | Navigation State Manager | — |
| FSS-SYS-071 | Recovery from degraded action eligibility | Persistence and Action Manager | Navigation State Manager; Monitoring Evaluator |
| FSS-SYS-072 | Explicit reset scope | Persistence and Action Manager | Monitor Coordinator |
| FSS-SYS-073 | Post-reset re-latch requires new confirming navigation | Persistence and Action Manager | Navigation State Manager; Monitoring Evaluator; Monitor Coordinator |
| FSS-SYS-074 | Non-finite projection result is not evaluated | Monitoring Evaluator | Decision Evidence Builder |

### 21.1 Allocation interpretation

Primary allocation means the named responsibility owns the behavior required to satisfy the requirement or owns the authoritative state transition that implements it.

It does not mean that responsibility operates independently.

For example:

- `FSS-SYS-060` is primarily allocated to the Persistence and Action Manager because that responsibility owns the violation interval that must be interrupted. The Monitoring Evaluator supplies the derived stale-gap continuity fact.
- `FSS-SYS-062` is primarily allocated to the Persistence and Action Manager because it determines whether the historical interval has received the required confirming evidence. Navigation and monitoring responsibilities supply accepted-state and eligibility facts.
- `FSS-SYS-035` is intentionally a system-boundary constraint rather than an internal monitor component responsibility.
- `FSS-SYS-038` through `FSS-SYS-040` constrain the deterministic core across component boundaries and therefore are explicitly cross-cutting rather than assigned to an artificial single state owner.

### 21.2 Traceability rule

Every baselined system requirement from `FSS-SYS-001` through `FSS-SYS-074` shall appear exactly once as a primary row in this allocation table.

Later architecture increments may refine secondary participation or implementation structure, but changing a primary allocation that affects state ownership or dependency direction requires architectural review.

---

## 22. Open Architecture Questions

The following remain intentionally open:

1. What exact C++ types represent time, sequence numbers, coordinates, and configuration values?
2. What concrete public API shall the deterministic monitor core expose?
3. Should logical responsibilities map one-to-one to classes, or should some remain pure functions/value types?
4. What exact immutable snapshot type shall cross evaluation boundaries?
5. What concrete `DecisionRecord` and runtime-event rejection-evidence types shall be used?
6. What stable reason-code catalog and deterministic ordering shall be defined?
7. Does any explicit monitor operating-state enum provide value beyond the already-defined eligibility/status facts?
8. Is an initialization acquisition deadline needed?
9. What serialization format, if any, belongs in the initial repository?
10. What diagnostic behavior is required if an external evidence sink fails?
11. What numeric ranges and representations are required for deterministic validation?
12. How shall sequence-number rollover be treated, if supported at all?

These questions shall be resolved through later architecture/interface increments rather than guessed during implementation.

---

## 23. Critical Architecture Review

Before implementation, review shall challenge at least the following:

### 23.1 Is the coordinator becoming a god object?

It may sequence responsibilities but should not become the authoritative owner of all monitor state and policy.

### 23.2 Are state owners duplicated?

There should not be multiple writable copies of:

- accepted navigation;
- persistence interval;
- action latch;
- runtime-event ordering state.

### 23.3 Are pure calculations unnecessarily stateful?

Geometry, freshness classification, timeout classification, and projection should remain deterministic calculations where possible.

### 23.4 Can evidence-generation code alter decisions?

If yes, the dependency boundary is wrong.

### 23.5 Can one outside observation plus ticks create an action?

If yes, the architecture violates the approved failure policy.

### 23.6 Can a stale interval be bridged because no event occurred at the freshness boundary?

If yes, the persistence design is incorrect.

### 23.7 Can future-dated navigation become accepted state?

If yes, temporal validation is allocated incorrectly.

### 23.8 Can interface timeout automatically erase otherwise fresh navigation?

If yes, interface and navigation responsibilities have been improperly coupled.

### 23.9 Does a projected violation enter current-state persistence?

If yes, projection has exceeded its advisory authority.

### 23.10 Does a rejected runtime event produce a normal decision record?

If yes, event gating and evidence responsibilities are improperly coupled.

### 23.11 Can navigation replacement hide a stale gap?

If the persistence path can see only the newly accepted fresh navigation state and cannot determine whether the prior state became stale before the arrival, the event transaction design is incorrect.

### 23.12 Does explicit reset preserve its narrow scope and post-reset evidence boundary?

If explicit reset clears accepted navigation, runtime-event history, interface history, or persistence merely because the latch is reset, the implementation violates the reset-scope requirement.

If pre-reset evidence or an evaluation tick can recreate the latch without newly accepted post-reset navigation evidence, the implementation violates the post-reset confirmation requirement.

---

## 24. Phase 2A Exit Criteria

Phase 2A is acceptable when review agrees that:

- every authoritative mutable state concept has one logical owner;
- runtime-event acceptance is separate from navigation-observation acceptance;
- interface activity is separate from navigation usability;
- modeled-time behavior does not depend on host execution time;
- freshness and timeout are derived rather than independently mutable truths;
- geometry and projection can remain pure deterministic calculations;
- persistence has one historical owner;
- ticks cannot supply confirming navigation evidence;
- persistence cannot bridge a stale interval;
- navigation replacement cannot erase the pre-update continuity evidence required to detect a stale gap;
- projection cannot feed current-state persistence;
- the simulated-action latch has one owner;
- normal decision evidence and runtime-event rejection evidence are distinct;
- evidence/diagnostic code cannot alter core decision behavior;
- new-execution initialization and explicit reset are not conflated;
- explicit reset has no unbaselined side effects;
- explicit reset preserves non-latch execution state, retains an otherwise continuous violation interval, and requires newly accepted post-reset navigation evidence before re-latch;
- deterministic replay remains possible from configuration plus synthetic event sequence;
- no unjustified concurrency, distribution, storage, framework, or operating-state machinery has been introduced;
- no operational aerospace thresholds or procedures have entered the architecture.

Only after this review passes should detailed C++ interface and type design begin.

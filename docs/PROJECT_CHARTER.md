# Flight Safety Monitor & Verification Platform
## Phase 0 — Engineering Definition

**Status:** Draft for engineering review
**System classification:** Synthetic, non-operational software engineering and verification platform
**Primary implementation languages:** C++20 and Python 3.12
**Current lifecycle phase:** Phase 0 — Engineering Definition

## Purpose

The Flight Safety Monitor & Verification Platform is a synthetic software system for studying the engineering of deterministic, safety-oriented monitoring software.

The project will develop:

1. a deterministic C++ monitoring core;
2. a controlled Python simulation and verification environment;
3. requirements-driven interfaces and behavior;
4. reproducible verification evidence;
5. explicit requirements-to-design-to-implementation-to-verification traceability.

The system will process synthetic vehicle navigation and health information, determine the validity and freshness of that information, evaluate synthetic vehicle behavior against explicitly configured monitoring criteria, maintain an explicit monitoring state, and produce deterministic decision records.

Where defined by future requirements, the system may emit a:

`SIMULATED_SAFETY_ACTION_REQUEST`

This output is exclusively a software simulation artifact.

The system shall not command, interface with, or provide operating parameters for destructive, hazardous, or real flight-safety hardware.

## Engineering objective

The primary objective is not to reproduce an operational flight termination system.

The objective is to construct an auditable engineering system in which another competent engineer can:

- understand what the software is supposed to do;
- understand why its architecture exists;
- reproduce its behavior;
- challenge its assumptions;
- trace implementation to requirements;
- reproduce verification results;
- inspect failures;
- determine why any safety decision occurred;
- continue development without relying on undocumented knowledge.

## Success criterion

The project is successful when its behavior is explainable and reproducible from controlled engineering artifacts, rather than merely when the software compiles or passes a nominal demonstration.

## Problem statement

Given a controlled stream of synthetic navigation and vehicle-health observations, an explicit notion of time, and versioned synthetic monitoring configuration, the system shall eventually determine deterministically whether the available information is acceptable for monitoring, determine the current monitoring-system state, evaluate defined synthetic safety conditions, and produce an auditable decision with stable reason codes and sufficient evidence to reconstruct that decision later.

The problem includes both nominal and abnormal information conditions.

Examples include:

- valid navigation observations;
- stale observations;
- duplicate observations;
- reordered observations;
- missing observations;
- invalid numeric values;
- degraded navigation validity;
- interface loss;
- transient synthetic-envelope excursions;
- persistent synthetic-envelope violations;
- projection-related violations.

A decision is not considered adequately engineered merely because the final action is correct.

The system must also make it possible to determine:

- what inputs were considered;
- which inputs were rejected;
- what configuration was active;
- which system state existed before the decision;
- what transition occurred;
- which conditions were evaluated;
- why the final decision occurred.

## System boundary

The project contains two principal software domains.

### Flight Safety Monitor

The C++ monitoring software is responsible for deterministic monitoring behavior.

Its eventual responsibilities may include:

- accepting already-delivered synthetic observations through a defined interface;
- validating observation contents;
- validating temporal and sequential consistency;
- tracking accepted state;
- evaluating navigation/system-health status;
- projecting synthetic vehicle state where required;
- evaluating configured synthetic monitoring boundaries;
- executing the defined monitoring state machine;
- producing deterministic decisions;
- producing stable reason codes;
- emitting structured decision records.

### Simulation and Verification Environment

The Python environment is responsible for controlled stimulus and verification.

Its eventual responsibilities may include:

- synthetic trajectory generation;
- scenario definition;
- deterministic fault injection;
- configuration generation;
- invoking the monitor;
- collecting monitor outputs;
- comparing actual and expected behavior;
- executing higher-level verification scenarios;
- producing verification artifacts;
- analyzing results.

### Boundary interfaces

Conceptually:

```text
Synthetic Scenario / Configuration
              |
              v
      Python Simulation
              |
              v
Synthetic Navigation / Health Observations
              |
              v
       C++ Monitor Core
              |
              v
Decision + Reason Codes + Evidence
              |
              v
Python Verification / Artifact Generation
```

The exact transport mechanism between Python and C++ is intentionally unspecified during Phase 0.

The behavioral contract must be defined before selecting IPC, serialization, subprocess, library-binding, or other integration mechanisms.

## Explicitly in-scope functionality

The intended project scope includes:

### Input handling

- synthetic navigation/state observations;
- synthetic health/status information;
- versioned configuration;
- explicitly represented time;
- deterministic scenario metadata.

### Input validation

Potential validation areas include:

- finite numeric values;
- permitted numeric ranges;
- timestamps;
- observation ordering;
- sequence numbers;
- observation freshness;
- health validity;
- interface availability;
- malformed or incomplete data.

### State management

- explicit monitoring-system states;
- explicit state-transition rules;
- invalid-transition detection;
- deterministic recovery or latching semantics.

The final state set will be derived from requirements.

### Synthetic safety monitoring

- configurable synthetic monitoring envelopes;
- boundary evaluation;
- transient-versus-persistent violation behavior;
- simple documented synthetic state projection;
- deterministic simulated decision generation.

### Fault handling

The verification system should eventually exercise conditions including:

- stale data;
- dropped data;
- duplicated data;
- reordered data;
- malformed data;
- invalid numeric data;
- health degradation;
- communication loss;
- state discontinuities;
- boundary excursions.

### Verification

- unit verification;
- integration verification;
- state-machine verification;
- boundary testing;
- interface testing;
- negative testing;
- fault injection;
- deterministic simulation scenarios;
- regression testing;
- static analysis;
- sanitizers;
- coverage measurement where useful.

### Engineering evidence

- requirements;
- design records;
- assumptions;
- change history;
- traceability;
- verification artifacts;
- reproducibility information;
- configuration identification.

## Explicitly out-of-scope functionality

The following are not project objectives.

### Operational aerospace control

No real vehicle or hazardous mechanism shall be commanded.

### Real-world flight termination criteria

No authentic operational thresholds, range-safety criteria, destruct rules, corridor data, or protected vehicle parameters shall be reproduced.

### High-fidelity vehicle dynamics

The project will not become an orbital-mechanics, guidance-navigation-control, aerodynamics, or six-degree-of-freedom simulation project.

The synthetic motion model exists to exercise monitoring software.

### Navigation algorithm development

The project will not implement a GPS receiver, inertial navigation system, sensor fusion algorithm, Kalman filter, or flight navigation solution unless a future engineering objective independently justifies a simplified synthetic component.

### Formal certification

The project will not claim:

- DO-178C compliance;
- MISRA compliance;
- RCC compliance;
- flight certification;
- production flight suitability.

Concepts from safety-critical engineering may inform the development approach without implying certification.

### Distributed-system complexity for its own sake

Networking, message brokers, databases, container orchestration, cloud infrastructure, and distributed services are out of scope unless a later requirement creates a specific engineering need.

### Hardware deployment during the core project

Embedded deployment remains an optional later phase and shall not influence early architecture unless avoiding host-specific coupling requires it.

## Stakeholders and actors

Human stakeholders and software actors are intentionally distinguished.

### Human stakeholders

#### Developer / maintainer

Builds and modifies the system while preserving requirements, design, traceability, and verification integrity.

#### Verification engineer

Defines verification procedures, expected results, scenario evidence, and requirement coverage.

This may initially be the same person as the developer, but the roles should remain conceptually distinct.

#### Engineering reviewer

Challenges requirements, assumptions, architecture, test sufficiency, failure behavior, and unsupported claims.

#### Future inheriting engineer

Represents an engineer who did not participate in the original design but must nevertheless be able to understand and reproduce the system.

Maintainability and documentation are explicit project objectives.

### Software actors

#### Synthetic scenario generator

Produces controlled scenario definitions and synthetic vehicle behavior.

#### Synthetic data source

Provides navigation and health observations according to the defined interface contract.

#### Flight Safety Monitor

Consumes controlled observations and produces deterministic monitoring decisions.

#### Verification harness

Evaluates observed monitor behavior against expected behavior.

#### Artifact generator

Produces machine-readable and human-readable verification evidence.

#### CI environment

Reproduces defined quality gates independently from the developer's interactive environment.

## High-level operational concept

A verification execution is conceptually performed as follows.

### Step 1 — Identify execution context

The verification environment identifies:

- software revision;
- build configuration;
- compiler/toolchain;
- scenario definition;
- scenario version;
- monitor configuration;
- deterministic random seed where applicable.

### Step 2 — Initialize the monitor

The monitor begins in a defined initialization state.

Its initial internal state shall not depend on uncontrolled external conditions such as the current wall clock.

### Step 3 — Provide synthetic observations

The Python environment generates or replays synthetic observations according to the selected scenario.

Each observation follows the eventual Interface Control Document.

### Step 4 — Validate observations

The monitor determines whether an observation is structurally and semantically acceptable.

Input acceptance and system safety decisions are separate concepts.

For example, rejecting one invalid observation does not automatically define what overall monitoring-system state should result.

That behavior must come from requirements.

### Step 5 — Update accepted system state

When permitted by requirements, validated information updates the monitor's internal representation of vehicle state and health.

### Step 6 — Evaluate monitoring conditions

The monitor evaluates whichever conditions are required for the current state.

These may eventually include:

- information freshness;
- system health;
- synthetic envelope status;
- persistent violation logic;
- projected synthetic state.

### Step 7 — Execute state-machine logic

The monitor performs a defined state transition according to explicit transition rules.

The transition model must prohibit accidental or undocumented transitions.

### Step 8 — Produce decision

The monitor produces a structured result containing enough information to determine:

- monitor state;
- decision;
- reason code or codes;
- relevant observation identity;
- relevant configuration identity;
- relevant timing information.

### Step 9 — Verify

The Python verification environment compares observed behavior with scenario expectations.

### Step 10 — Preserve evidence

Verification results are recorded in machine-readable artifacts from which higher-level reports may later be generated.

## Preliminary failure philosophy

This section is intentionally preliminary.

Detailed responses to individual failures must eventually come from requirements and hazard analysis.

### 1. No silent acceptance

Malformed, invalid, non-finite, temporally invalid, or otherwise unacceptable information must never be silently treated as valid information.

### 2. No silent failure

An important inability to perform the intended monitoring function must produce observable evidence.

### 3. Preserve distinction between data validity and safety decision

An invalid observation is not inherently equivalent to a synthetic envelope violation.

For example:

- bad navigation data;
- unavailable navigation;
- a valid observation indicating an envelope violation

represent different conditions and should remain distinguishable.

### 4. Deterministic failure handling

The same:

- previous state;
- input;
- configuration;
- modeled time

shall produce the same result.

Failure handling must therefore not depend on scheduling accidents, uncontrolled wall-clock behavior, or nondeterministic iteration ordering.

### 5. Explicit degraded operation

If the system supports degraded monitoring, its permitted capabilities and exit conditions must be explicitly defined.

`DEGRADED` must not become a generic destination for every poorly understood condition.

### 6. Explicit latching

Any latched simulated safety decision must:

- have a defined triggering condition;
- have defined persistence semantics;
- have defined reset semantics;
- preserve the triggering reason.

The existence of a `SAFETY_ACTION_LATCHED` concept does not itself determine those semantics.

### 7. Failure should preserve evidence

Failure handling shall not destroy the information necessary to understand the failure.

### 8. Conservative behavior must be requirement-defined

The project should not casually use the phrase "fail safe."

Different synthetic failures may reasonably demand different responses.

For example, loss of navigation could potentially imply:

- degraded monitoring;
- inability to make a decision;
- a simulated safety action;
- retention of the last accepted state for a limited period.

Phase 0 does not choose among those behaviors.

They require hazard reasoning and explicit requirements.

## Major engineering risks

### RISK-01 — Premature implementation

**Risk:** Code is written before behavior is adequately defined.

**Consequence:** Requirements are later reverse-engineered from implementation.

**Control:** No monitoring implementation until requirements governing that behavior are baselined.

### RISK-02 — State-machine ambiguity

**Risk:** States receive intuitive names without precise entry, exit, and transition semantics.

**Consequence:** Different engineers interpret identical conditions differently.

**Control:** Define transitions from behavioral requirements before implementing the state machine.

### RISK-03 — Ambiguous time semantics

**Risk:** Source time, receive time, processing time, and simulation time are conflated.

**Consequence:** Freshness and timeout behavior become inconsistent or nondeterministic.

**Control:** Establish the authoritative time model before related requirements are baselined.

### RISK-04 — Undefined precedence between simultaneous conditions

**Risk:** Multiple faults occur during the same processing step.

Example:

- navigation is stale;
- health is invalid;
- projected state violates an envelope.

**Consequence:** Behavior depends on implementation ordering rather than requirements.

**Control:** Define whether decisions have precedence, aggregation, or another deterministic composition rule.

### RISK-05 — Synthetic model accidentally drives architecture

**Risk:** The simple vehicle model becomes tightly coupled to monitoring logic.

**Consequence:** Safety decisions become difficult to test independently.

**Control:** Treat projection and monitoring as separate responsibilities.

### RISK-06 — Verification oracle derived from implementation

**Risk:** Python verification reproduces the C++ algorithm.

**Consequence:** The same conceptual defect can exist in both implementation and test oracle.

**Control:** Define expected behavior from requirements and scenario specifications, not by translating implementation into Python.

### RISK-07 — Overengineering traceability

**Risk:** Significant tooling is built before enough requirements exist to justify it.

**Consequence:** More effort is spent maintaining process infrastructure than verifying the system.

**Control:** Begin with simple machine-readable conventions and automate only when repetition or human error justifies it.

### RISK-08 — Process imitation mistaken for assurance

**Risk:** Aerospace terminology creates an appearance of rigor unsupported by evidence.

**Consequence:** The project becomes bureaucratic instead of technically rigorous.

**Control:** Every artifact and process must answer a specific engineering question.

### RISK-09 — Configuration ambiguity

**Risk:** Verification behavior changes because configuration provenance is unclear.

**Consequence:** Results cannot be reproduced.

**Control:** Version and identify configuration in verification artifacts.

### RISK-10 — Platform-dependent numerical behavior

**Risk:** Floating-point corner cases produce unexpected differences.

**Consequence:** Decision boundaries may behave inconsistently.

**Control:** Define numeric assumptions, comparison semantics, finite-value handling, tolerances where justified, and supported toolchains.

### RISK-11 — Integration mechanism contaminates core design

**Risk:** IPC, files, networking, or Python bindings enter monitoring logic.

**Consequence:** Core behavior becomes difficult to execute deterministically in isolation.

**Control:** Keep transport outside the deterministic monitoring domain.

### RISK-12 — Hardware scope creep

**Risk:** Embedded deployment begins before the host implementation is mature.

**Consequence:** Target-specific constraints obscure unresolved software-design problems.

**Control:** Treat hardware as a gated later phase.

## Initial assumptions

These are provisional working assumptions, not baselined requirements.

Each must either be confirmed, changed, or explicitly retired before relevant requirements are finalized.

### A-001 — Synthetic-only environment

All vehicle data, configurations, thresholds, and failure conditions are synthetic.

### A-002 — Single monitored synthetic vehicle

The first system version monitors one logical vehicle at a time.

Multi-vehicle monitoring is not initially required.

### A-003 — Single authoritative navigation stream

The first system version receives one logical navigation stream.

Redundant navigation voting and sensor fusion are excluded.

### A-004 — Host-based execution

Initial development and verification occur on Linux host systems.

### A-005 — Controlled execution model

Core monitoring logic can be evaluated synchronously using explicitly supplied inputs and explicitly supplied time.

No concurrency requirement is currently assumed for the core decision logic.

### A-006 — Synthetic kinematic projection

Projection uses an intentionally simple mathematical model sufficient for monitoring-software verification.

It is not intended to provide realistic aerospace trajectory prediction.

### A-007 — Configuration fixed during an execution

A verification scenario initially uses one immutable monitor configuration from initialization through scenario completion.

Dynamic reconfiguration is not assumed.

### A-008 — Observability is external to decisions

Recording and presentation mechanisms must not alter decision semantics.

### A-009 — Bounded verification scenarios

Synthetic executions are finite and have explicit scenario start and completion conditions.

### A-010 — No real-time guarantee yet

Deterministic logical timing is required.

Hard real-time execution deadlines are not currently assumed.

If timing deadlines later become project objectives, they require separate requirements and measurement methodology.

## Proposed project lifecycle

### Phase 0 — Engineering Definition

Define:

- purpose;
- problem;
- system boundary;
- scope;
- actors;
- operational concept;
- preliminary failure philosophy;
- risks;
- assumptions;
- lifecycle;
- requirement-baseline blockers.

**Exit condition:** engineering definition survives review without major unresolved conceptual contradictions.

### Phase 1 — Requirements Baseline

Develop:

- system requirements;
- software requirements;
- interface requirements;
- assumptions and constraints;
- initial hazard-derived requirements;
- requirement quality checks.

No production monitoring implementation should precede the minimum behavioral requirements required by that component.

**Exit condition:** requirements governing the first implementation increment are uniquely identified, testable, internally consistent, and reviewable.

### Phase 2 — Architecture and Interface Baseline

Develop:

- software architecture;
- component responsibility boundaries;
- state-machine design;
- controlled time model;
- interface contract;
- configuration model;
- decision/reason-code model;
- initial traceability structure.

**Exit condition:** each component has a defensible responsibility and every important architectural behavior traces to a requirement or explicit engineering constraint.

### Phase 3 — Deterministic Core Implementation

Implement the smallest requirements-backed vertical slices of the C++ monitor.

Candidate progression might be:

1. domain data types;
2. navigation validation;
3. accepted-state tracking;
4. state-machine behavior;
5. envelope evaluation;
6. state projection;
7. integrated decision logic.

This order remains provisional until requirements and architecture determine dependencies.

**Exit condition:** implemented requirements have direct unit-level verification evidence.

### Phase 4 — Simulation and Integration Verification

Develop the Python scenario system and clean C++ integration boundary.

Introduce controlled scenarios incrementally.

**Exit condition:** end-to-end synthetic scenarios execute reproducibly and verify requirement-defined behavior.

### Phase 5 — Fault Injection and Robustness Verification

Add systematic:

- malformed input;
- dropped updates;
- duplicates;
- reordered data;
- stale data;
- timing violations;
- health degradation;
- boundary faults;
- corrupted input.

**Exit condition:** defined failure behaviors have repeatable verification evidence.

### Phase 6 — Traceability and Automated Quality Gates

Mature:

- requirement metadata;
- design traceability;
- test traceability;
- traceability validator;
- static analysis;
- sanitizers;
- CI quality gates;
- artifact provenance.

Automation is introduced where manual maintenance has become demonstrably error-prone.

**Exit condition:** CI can detect meaningful integrity failures, not merely compilation failures.

### Phase 7 — System Verification and Reproducibility

Perform controlled clean-environment verification.

Generate verification reports from observed project data.

Validate:

- clean checkout;
- documented toolchain;
- complete verification execution;
- artifact generation;
- traceability completeness;
- configuration identification;
- artifact integrity.

**Exit condition:** an independent engineer can reproduce the supported verification workflow from repository documentation.

### Phase 8 — Optional Host-to-Target Extension

Only after host software reaches sufficient maturity:

- evaluate hardware targets;
- define target constraints;
- cross-compile;
- establish communication;
- execute host-to-target integration tests;
- measure target behavior.

This phase is optional and shall not be required for the host-based project to be considered technically complete.

## Questions that must be resolved before requirements baseline

The following are behavioral questions rather than implementation details.

They should be resolved before the affected requirements are baselined.

### 1. Coordinate and state model

What synthetic vehicle state exists?

For example:

- 2D or 3D position?
- Cartesian coordinates?
- velocity components?
- acceleration?
- health fields?

A monitoring envelope cannot be defined rigorously until the monitored quantities are known.

### 2. Units

What canonical units are used for every physical quantity?

Units must be explicit in both interfaces and configuration.

### 3. Time authority

Which clock defines:

- sample age;
- timeout;
- persistence duration;
- scenario progression?

How do source timestamp and receive time interact?

### 4. Timestamp rules

Must timestamps be strictly increasing?

Are equal timestamps allowed?

What is the representation and resolution?

Can timestamps wrap?

### 5. Sequence-number semantics

What constitutes:

- duplicate;
- skipped sample;
- regression;
- rollover?

Does a skipped sequence number invalidate the next observation or merely create an event?

### 6. Sample rejection semantics

When an individual observation is rejected:

- does the last accepted state remain usable?
- for how long?
- does monitor health degrade immediately?
- can subsequent valid observations recover operation?

### 7. Navigation validity model

Is navigation simply valid/invalid, or are multiple quality states required?

Avoid adding multiple quality states unless a real behavioral need exists.

### 8. Interface availability

What precisely constitutes loss of interface?

No observation for some duration?

Explicit transport failure?

Both?

### 9. Monitoring state machine

What behavior requires distinct states?

We should not begin from the proposed names and work backward.

We should derive states from differing required behavior.

### 10. Initialization behavior

What information is required before monitoring can begin?

Examples might include:

- valid configuration;
- first valid navigation observation;
- multiple observations needed to estimate continuity.

### 11. Standby semantics

Does `STANDBY` actually represent required behavior, or is it merely an intuitive aerospace-sounding state?

If no requirement demands behavior distinct from initialization or monitoring, it should not exist.

### 12. Degraded-state semantics

What capabilities remain available during degradation?

What conditions enter degradation?

What conditions allow recovery?

### 13. Latching semantics

What conditions cause simulated action latching?

Can it ever automatically clear?

Is reset possible only through explicit reinitialization?

This is a major safety-behavior decision and cannot remain implicit.

### 14. Envelope model

What geometric or mathematical object defines the initial synthetic monitoring envelope?

It should be sufficiently expressive to exercise boundary behavior without introducing unnecessary geometry complexity.

### 15. Boundary semantics

Is exactly-on-boundary considered:

- acceptable;
- violation;
- configurable?

Floating-point comparison behavior must follow from this definition.

### 16. Persistence semantics

What distinguishes a transient excursion from a persistent violation?

Possible models include:

- consecutive observations;
- elapsed synthetic time;
- accumulated duration.

The project should choose one intentionally.

### 17. Projection model

What initial synthetic projection equation will be used?

What is its prediction horizon?

Is projection continuous or evaluated only when new accepted observations arrive?

### 18. Projection trust

Under what conditions is projection prohibited?

For example:

- stale source state;
- invalid velocity;
- excessive projection horizon.

### 19. Multiple simultaneous conditions

Can multiple reason codes be emitted for one evaluation?

If so, are they:

- ordered;
- unordered;
- prioritized?

If one decision reason must be primary, its precedence must be specified.

### 20. Processing model

What is the logical unit of processing?

For example:

`process_observation(observation, current_time) -> decision`

or does evaluation also occur when time advances without receiving an observation?

This question is important because timeouts can occur in the absence of incoming data.

### 21. Configuration lifecycle

When is configuration validated?

Can configuration change during monitoring?

For the first system version, immutable configuration is likely simpler and easier to verify, but that remains a design proposal rather than a requirement.

### 22. Configuration invalidity

What happens if configuration is internally inconsistent?

The preferred direction is to reject invalid configuration before monitoring begins rather than discover it while evaluating vehicle state.

### 23. Expected decision frequency

Does the monitor produce a decision record:

- for every input observation;
- for every evaluation cycle;
- only when something changes?

This affects observability and verification.

### 24. Recovery philosophy

Which faults are recoverable?

Examples:

- one malformed packet;
- temporary dropout;
- invalid health;
- timestamp regression;
- interface timeout.

Recovery behavior must not emerge accidentally from implementation.

### 25. Supported numerical behavior

Which numeric type is used for synthetic physical state?

What non-finite values are invalid?

Are comparison tolerances ever appropriate?

No general-purpose epsilon should be introduced without a specific numerical rationale.

## Critical review of Phase 0

### 1. The project scope is ambitious

The requested final system includes:

- requirements engineering;
- architecture;
- interface control;
- state-machine design;
- simulation;
- fault injection;
- static analysis;
- sanitizers;
- coverage;
- traceability;
- CI;
- reproducibility;
- generated verification evidence;
- potentially embedded deployment.

That is enough work for a substantial multi-week or multi-month engineering project if performed seriously.

The appropriate response is not to reduce rigor.

The appropriate response is to control sequencing.

Many of these capabilities should appear only after the underlying system is mature enough for them to provide engineering value.

### 2. The documentation set should not become an upfront checklist

Creating all proposed documents during Phase 0 would be counterproductive.

For example, `DETAILED_DESIGN.md` has little useful content before architecture exists.

Likewise, `VERIFICATION_REPORT.md` should not exist before real verification evidence exists.

Documents should be introduced when the lifecycle produces information worth controlling.

### 3. The proposed state names are not yet justified

`INITIALIZING`, `STANDBY`, `MONITORING`, `DEGRADED`, and `SAFETY_ACTION_LATCHED` sound plausible.

That is not enough.

Each state must exist because the system has materially different required behavior in that state.

Particularly questionable at this point are:

- `STANDBY`;
- `DEGRADED`.

They may ultimately be correct, but Phase 0 has not demonstrated that yet.

### 4. We have not yet defined what an evaluation cycle is

This is one of the most important unresolved concepts.

A monitor that only evaluates when navigation arrives cannot independently detect that navigation stopped arriving unless some external event causes evaluation.

Therefore we must eventually distinguish at least conceptually between:

- observation arrival;
- passage of modeled time;
- safety evaluation.

This must be resolved before writing timeout requirements.

### 5. Determinism needs a project-specific definition

The term should eventually mean more than "usually produces the same result."

A likely form is:

> For a specified software revision, supported execution environment, initial state, configuration, ordered input-event sequence, and modeled-time sequence, externally observable monitor decisions shall be reproducible.

That definition still requires refinement around floating-point and platform assumptions.

### 6. Safety envelope is currently underdefined

Before requirements begin referring extensively to an envelope, we need a deliberately simple first model.

A complicated corridor or trajectory geometry would create testing work unrelated to the principal engineering objective.

The initial model should allow us to test:

- inside;
- boundary;
- outside;
- transient outside;
- persistent outside;
- projected outside

without introducing unnecessary computational geometry.

### 7. We must avoid building a pseudo-certification process

The strongest aspect of this project should be technical consistency:

```text
Requirement
    |
    v
Design
    |
    v
Implementation
    |
    v
Verification Evidence
```

The project becomes weaker, not stronger, if it accumulates aerospace-looking documentation that does not influence engineering decisions.

### 8. Traceability automation should arrive incrementally

A sophisticated traceability database or framework would be premature.

Initially, stable IDs and simple machine-readable conventions are sufficient.

Automation becomes justified once enough requirements, tests, and design elements exist that manual consistency checking becomes error-prone.

### 9. The Python/C++ integration mechanism should remain undecided

Choosing pybind11, subprocess JSON, sockets, shared libraries, or another interface now would be premature.

First define the logical contract.

Then select the simplest mechanism that preserves it.

### 10. The preliminary failure philosophy deliberately leaves major decisions unresolved

This is intentional.

It would be inappropriate during Phase 0 to decide, without hazard reasoning, that:

`NAV_STALE -> SIMULATED_SAFETY_ACTION_REQUEST`

or:

`NAV_STALE -> DEGRADED`

Those are requirements decisions.

The project should resist encoding intuitive "safe" behavior before defining what safe behavior means within the synthetic system.

## Phase 0 review status

The project concept is coherent enough to continue within Phase 0, but it is not yet ready for a software requirements baseline.

The strongest unresolved areas are:

1. event and evaluation model;
2. time semantics;
3. minimum vehicle-state definition;
4. initial envelope model;
5. state-machine behavioral distinctions;
6. failure and recovery semantics;
7. latching and reset policy;
8. simultaneous-condition handling.

The next Phase 0 engineering increment should therefore define the system semantic model, particularly:

- information model;
- event model;
- time model;
- evaluation model;
- decision model.

Only after those concepts are stable should behavioral requirements be derived.

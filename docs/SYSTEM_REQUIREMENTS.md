# Flight Safety Monitor & Verification Platform
## Phase 1A — System Requirements Baseline Candidate

**Status:** Draft for engineering review
**Lifecycle phase:** Phase 1 — Requirements Baseline
**Scope:** System-level requirements justified by the approved Phase 0 charter and semantic model
**Implementation status:** No implementation authorized by this document

---

## 1. Purpose

This document defines the first candidate baseline of system-level requirements for the synthetic Flight Safety Monitor & Verification Platform.

The requirements in this increment are intentionally limited to behavior already justified by the Phase 0 engineering definition.

This document does not yet define:

- the final monitor operating-state machine;
- degradation entry or recovery rules;
- exact simulated safety-action trigger conditions;
- detailed interface serialization;
- numeric storage widths;
- concrete synthetic threshold values;
- software component architecture.

Those items remain open until their behavior is derived and reviewed.

---

## 2. Requirement Quality Rules

Every requirement in this document shall:

- have a stable identifier;
- state one primary obligation where practical;
- be testable or reviewable;
- avoid naming implementation classes;
- avoid assuming a transport mechanism;
- avoid real operational aerospace parameters;
- identify its primary verification method;
- include a rationale.

Verification method labels used in this draft are:

- **TEST** — executable verification is expected;
- **ANALYSIS** — deterministic analysis or generated evidence is expected;
- **INSPECTION** — document/interface/configuration review is expected.

---

## 3. Configuration and Initialization Requirements

### FSS-SYS-001 — Configuration validation before monitoring

**Requirement:** The system shall validate the complete monitor configuration before allowing normal monitoring to begin.

**Rationale:** Monitoring behavior must not depend on partially valid or unchecked configuration.

**Primary verification:** TEST

---

### FSS-SYS-002 — Invalid configuration blocks monitoring

**Requirement:** If monitor configuration is invalid or internally inconsistent, the system shall prevent entry into normal monitoring behavior.

**Rationale:** Invalid configuration can invalidate every downstream safety decision.

**Primary verification:** TEST

---

### FSS-SYS-003 — Configuration immutability during execution

**Requirement:** After successful initialization, the active monitor configuration shall remain unchanged until explicit reset or reinitialization.

**Rationale:** Mid-execution configuration changes would make behavior and verification evidence harder to reproduce.

**Primary verification:** TEST

---

### FSS-SYS-004 — Configuration identity in evidence

**Requirement:** Every runtime decision record shall identify the configuration under which the decision was produced.

**Rationale:** Verification evidence must be attributable to the configuration that produced it.

**Primary verification:** TEST

---

## 4. Modeled-Time Requirements

### FSS-SYS-005 — Explicit modeled time

**Requirement:** Core monitoring behavior shall use externally supplied modeled time rather than obtaining wall-clock time implicitly.

**Rationale:** Unit and scenario verification must control time deterministically.

**Primary verification:** INSPECTION + TEST

---

### FSS-SYS-006 — Monotonic runtime-event time

**Requirement:** Within one initialized execution, an accepted runtime event shall not have a modeled time earlier than the modeled time of the previously accepted runtime event.

**Rationale:** Backward time movement would invalidate age, timeout, and persistence calculations.

**Primary verification:** TEST

---

### FSS-SYS-007 — Regressive-time event rejection

**Requirement:** A runtime event whose modeled time is earlier than the previously accepted runtime-event time shall be rejected before normal safety evaluation.

**Rationale:** Invalid event-time ordering must not contaminate monitor state.

**Primary verification:** TEST

---

### FSS-SYS-008 — Regressive-time rejection preserves state

**Requirement:** Rejection of a regressive-time runtime event shall not advance normal monitor state.

**Rationale:** Invalid external event ordering must not change accepted monitoring state.

**Primary verification:** TEST

---

### FSS-SYS-009 — Time may advance without navigation arrival

**Requirement:** The system shall support a runtime event that advances evaluation time without providing a new navigation observation.

**Rationale:** Staleness and timeout behavior must remain detectable after input arrival stops.

**Primary verification:** TEST

---

## 5. Runtime Evaluation Requirements

### FSS-SYS-010 — One evaluation per accepted runtime event

**Requirement:** Each accepted runtime event shall produce exactly one logical runtime evaluation.

**Rationale:** A clear event-to-evaluation relationship improves determinism and traceability.

**Primary verification:** TEST

---

### FSS-SYS-011 — One decision record per runtime evaluation

**Requirement:** Each logical runtime evaluation shall produce exactly one logical decision record.

**Rationale:** Every evaluation must leave reconstructable evidence.

**Primary verification:** TEST

---

### FSS-SYS-012 — Runtime-event acceptance distinct from observation acceptance

**Requirement:** Acceptance of a navigation-arrival runtime event shall not imply acceptance of the navigation observation contained by that event.

**Rationale:** Transport/event validity and navigation-data validity are separate concerns.

**Primary verification:** TEST

---

## 6. Navigation Observation Requirements

### FSS-SYS-013 — Non-finite physical values invalid

**Requirement:** A navigation observation containing NaN or positive or negative infinity in any physical state field shall be considered invalid for navigation-state acceptance.

**Rationale:** Non-finite values can invalidate comparisons, projection, and envelope evaluation.

**Primary verification:** TEST

---

### FSS-SYS-014 — Rejected observation cannot replace accepted state

**Requirement:** A rejected navigation observation shall not replace the last accepted navigation state.

**Rationale:** Invalid input must not silently become authoritative vehicle state.

**Primary verification:** TEST

---

### FSS-SYS-015 — Strict source-time progress for newer accepted state

**Requirement:** A navigation observation shall not be eligible to replace the last accepted navigation state unless its source timestamp is greater than the source timestamp of the last accepted navigation observation.

**Rationale:** Equal or regressive source time does not represent newer state under the Phase 0 time model.

**Primary verification:** TEST

---

### FSS-SYS-016 — Duplicate sequence number is not newer state

**Requirement:** A navigation observation with a sequence number equal to the sequence number of the last accepted navigation observation shall not be treated as newer navigation state.

**Rationale:** Duplicate observations must not masquerade as new vehicle information.

**Primary verification:** TEST

---

### FSS-SYS-017 — Regressive sequence number is not newer state

**Requirement:** A navigation observation with a sequence number lower than the sequence number of the last accepted navigation observation shall not be treated as newer navigation state.

**Rationale:** Reordered or regressive observations must not replace newer accepted state.

**Primary verification:** TEST

---

### FSS-SYS-018 — Sequence gaps are observable

**Requirement:** When a received navigation observation indicates a forward sequence-number gap relative to the last accepted navigation observation, the system shall make the gap observable in runtime evidence.

**Rationale:** Missing-update evidence must not be silently discarded even when the newer observation remains usable.

**Primary verification:** TEST

---

## 7. Freshness and Interface-Availability Requirements

### FSS-SYS-019 — Navigation age basis

**Requirement:** When an accepted navigation observation exists, navigation data age shall be computed from evaluation time and the source time of the last accepted navigation observation.

**Rationale:** Navigation freshness concerns the age of the represented vehicle state.

**Primary verification:** TEST

---

### FSS-SYS-020 — Interface-silence basis

**Requirement:** When qualifying interface activity has occurred, interface silence duration shall be computed from evaluation time and the receive time of the last qualifying interface activity.

**Rationale:** Interface availability concerns observed communication activity, not source-state age.

**Primary verification:** TEST

---

### FSS-SYS-021 — Freshness and availability remain distinct

**Requirement:** The system shall represent navigation freshness and interface availability as distinct conditions.

**Rationale:** Recently received stale data and complete communication silence are different failure conditions.

**Primary verification:** TEST

---

## 8. Synthetic Envelope Requirements

### FSS-SYS-022 — Initial envelope form

**Requirement:** The initial synthetic monitoring envelope shall be representable as independent minimum and maximum Cartesian bounds for x, y, and z position.

**Rationale:** An axis-aligned box is sufficient to exercise monitoring behavior without unnecessary geometric complexity.

**Primary verification:** TEST + INSPECTION

---

### FSS-SYS-023 — Envelope configuration ordering

**Requirement:** For each Cartesian axis, valid envelope configuration shall require the configured minimum bound to be less than the configured maximum bound.

**Rationale:** Equal or inverted bounds do not define the intended interval.

**Primary verification:** TEST

---

### FSS-SYS-024 — Exact boundary is inside

**Requirement:** A valid position exactly equal to a configured synthetic envelope boundary shall be classified as inside the envelope.

**Rationale:** Boundary behavior must be explicit and independently testable.

**Primary verification:** TEST

---

### FSS-SYS-025 — Outside-envelope classification

**Requirement:** A valid current position shall be classified as outside the synthetic envelope when at least one Cartesian coordinate is strictly less than its configured minimum or strictly greater than its configured maximum.

**Rationale:** Envelope violation semantics must be deterministic and unambiguous.

**Primary verification:** TEST

---

## 9. Persistence Requirements

### FSS-SYS-026 — Persistence based on modeled elapsed time

**Requirement:** Persistent current-state envelope violation determination shall use modeled elapsed time rather than navigation-observation count alone.

**Rationale:** Persistence behavior should not change merely because source update rate changes.

**Primary verification:** TEST

---

### FSS-SYS-027 — Transient return inside ends violation interval

**Requirement:** If an eligible accepted navigation state returns inside the synthetic envelope before the configured persistence duration is reached, the active current-state envelope-violation interval shall end without becoming a persistent violation.

**Rationale:** Transient and persistent excursions must remain distinct.

**Primary verification:** TEST

---

## 10. Projection Requirements

### FSS-SYS-028 — Constant-velocity projection

**Requirement:** The initial synthetic projection shall compute projected Cartesian position using current accepted position, current accepted velocity, and a configured projection horizon under a constant-velocity model.

**Rationale:** The initial projection should remain understandable and independently verifiable.

**Primary verification:** TEST

---

### FSS-SYS-029 — Projection requires eligible accepted state

**Requirement:** The system shall not perform synthetic state projection from navigation state that is not eligible for projection under the active requirements.

**Rationale:** Projection from unusable source data can create misleading downstream safety evidence.

**Primary verification:** TEST

---

### FSS-SYS-030 — Projected position uses envelope semantics

**Requirement:** When projected position is evaluated against the synthetic envelope, it shall use the same boundary semantics as current-state envelope evaluation unless a later baselined requirement explicitly defines otherwise.

**Rationale:** Different implicit boundary rules for current and projected state would be difficult to justify and verify.

**Primary verification:** TEST

---

## 11. Decision and Evidence Requirements

### FSS-SYS-031 — Stable machine-readable reason identifiers

**Requirement:** Non-nominal runtime decision evidence shall use stable machine-readable reason identifiers rather than relying only on free-form explanatory text.

**Rationale:** Traceability and automated verification require stable identifiers.

**Primary verification:** INSPECTION + TEST

---

### FSS-SYS-032 — Multiple simultaneous conditions observable

**Requirement:** When multiple abnormal conditions are simultaneously applicable to one runtime evaluation, the decision evidence shall be capable of representing more than one applicable reason.

**Rationale:** One condition must not erase evidence of another merely because a single primary reason is later selected.

**Primary verification:** TEST

---

### FSS-SYS-033 — Deterministic reason ordering

**Requirement:** When more than one reason identifier is emitted for one runtime evaluation, their external ordering shall be deterministic.

**Rationale:** Output ordering must not depend on incidental implementation traversal order.

**Primary verification:** TEST

---

### FSS-SYS-034 — Decision record reconstructability

**Requirement:** A runtime decision record shall contain sufficient information to determine, directly or by stable reference:

- evaluation identity;
- evaluation time;
- triggering runtime-event category;
- active configuration identity;
- prior monitor operating state;
- resulting monitor operating state;
- relevant observation identity when applicable;
- observation acceptance/rejection status when applicable;
- applicable information-status results;
- current envelope status when evaluated;
- projected envelope status when evaluated;
- simulated action result;
- applicable stable reason identifiers.

**Rationale:** An engineer must be able to reconstruct why a decision occurred.

**Primary verification:** INSPECTION + TEST

---

## 12. Simulated Action and Reset Requirements

### FSS-SYS-035 — Simulated action is non-operational

**Requirement:** The system shall expose simulated safety-action results only as software data and shall not provide an interface that commands an actual destructive or hazardous mechanism.

**Rationale:** The project is explicitly non-operational.

**Primary verification:** INSPECTION

---

### FSS-SYS-036 — Latched simulated action persists

**Requirement:** Once a simulated safety action has become latched under a future baselined triggering requirement, subsequent runtime evaluations shall continue to report the latched simulated action until explicit reset or reinitialization.

**Rationale:** This defines the meaning of "latched" without prematurely selecting latch-triggering conditions.

**Primary verification:** TEST

---

### FSS-SYS-037 — Reset clears execution-specific latch state

**Requirement:** Explicit reset or reinitialization shall clear any latched simulated safety-action state from the prior execution.

**Rationale:** A new execution must not inherit the previous execution's latch state.

**Primary verification:** TEST

---

## 13. Determinism and Observability Requirements

### FSS-SYS-038 — Deterministic decision sequence

**Requirement:** For the same supported software revision, validated configuration, initial execution state, ordered accepted runtime-event sequence, event payloads, and modeled-time values, the system shall produce the same sequence of logical runtime decision records.

**Rationale:** Reproducibility is a core project objective.

**Primary verification:** TEST + ANALYSIS

---

### FSS-SYS-039 — Logging does not affect decision semantics

**Requirement:** Success or failure of diagnostic logging shall not alter core monitoring decision semantics.

**Rationale:** Observability must not contaminate the decision path.

**Primary verification:** TEST + INSPECTION

---

### FSS-SYS-040 — Host execution duration does not define modeled safety time

**Requirement:** Host processing duration shall not be used as the modeled time basis for navigation freshness, interface silence, boundary persistence, or synthetic projection.

**Rationale:** Host load must not change logical safety behavior.

**Primary verification:** INSPECTION + TEST

---

## 14. Requirements Intentionally Deferred

The following are not baselined in this increment:

- which received inputs qualify as interface activity;
- exact navigation freshness threshold;
- exact interface-timeout threshold;
- exact persistence threshold;
- exact projection horizon;
- exact synthetic envelope dimensions;
- monitor operating-state names and transitions;
- degradation-entry conditions;
- degradation-recovery conditions;
- invalid-navigation recovery rules;
- interface-timeout recovery rules;
- exact simulated safety-action trigger conditions;
- exact latch-triggering conditions;
- primary reason-code precedence;
- decision-record behavior for rejected runtime events;
- numeric storage widths;
- sequence-number rollover behavior.

No implementation shall invent these behaviors merely because this document defers them.

---

## 15. Preliminary Requirement Review Findings

### 15.1 Requirements that are ready for baseline consideration

The requirements governing:

- controlled modeled time;
- event/evaluation cardinality;
- invalid numeric rejection;
- rejected-state preservation;
- source-time and sequence progression;
- freshness/availability distinction;
- synthetic envelope geometry;
- exact boundary semantics;
- elapsed-time persistence;
- constant-velocity projection;
- decision evidence;
- deterministic output;
- non-operational action boundaries

are sufficiently defined for review because their semantics were established during Phase 0.

### 15.2 Requirements that should remain deferred

The project should not yet baseline:

- state-machine transitions;
- degraded-state behavior;
- safety-action trigger logic;
- recovery rules;
- precedence among abnormal conditions.

Those decisions should be informed by preliminary hazard analysis rather than selected for convenience.

---

## 16. Phase 1A Exit Criteria

This candidate requirements increment is acceptable for baseline when:

- each requirement has one stable unique identifier;
- no requirement depends on undefined operational aerospace data;
- every requirement is testable or inspectable;
- no requirement silently resolves a Phase 0 open question;
- requirements and the preliminary hazard log are mutually consistent;
- no state-machine behavior has been invented merely to satisfy implementation convenience;
- each requirement's verification method is plausible;
- review identifies no internal contradiction with the Phase 0 semantic model.

Only after those conditions are met should the requirements be committed as a baseline candidate.

# Flight Safety Monitor & Verification Platform
## Phase 1 — System Requirements Baseline Candidate

**Status:** Draft for engineering review
**Lifecycle phase:** Phase 1 — Requirements Baseline
**Scope:** System-level requirements justified by the approved Phase 0 charter, semantic model, and reviewed Phase 1B failure policy
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

**Requirement:** Explicit reset within an execution shall clear the current simulated safety-action latch. Reinitialization shall begin a new execution without inheriting latched simulated safety-action state from the prior execution.

**Rationale:** Explicit reset and reinitialization both clear latched action state, but they are distinct lifecycle operations: reset acts within the current execution, while reinitialization establishes a new execution boundary.

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

## 14. Phase 1B Failure-Policy Requirements

The following requirements derive from the reviewed Phase 1B failure and decision policy.

They append to the existing requirement baseline without renumbering or redefining `FSS-SYS-001` through `FSS-SYS-040`.

### FSS-SYS-041 — Qualifying navigation interface activity

**Requirement:** An accepted runtime event recognized as a navigation-arrival event shall count as qualifying navigation interface activity.

**Rationale:** Interface availability concerns recognizable arrival activity, not whether the contained vehicle state is usable.

**Primary verification:** TEST

---

### FSS-SYS-042 — Observation rejection does not cancel interface activity

**Requirement:** Rejection of the navigation observation contained by an otherwise accepted navigation-arrival runtime event shall not, by itself, prevent that event from counting as qualifying interface activity.

**Rationale:** Navigation-state acceptance and interface activity are intentionally distinct.

**Primary verification:** TEST

---

### FSS-SYS-043 — Unrecognized malformed transport is not qualifying activity

**Requirement:** Transport input that cannot be recognized as a navigation-arrival runtime event shall not count as qualifying navigation interface activity unless a later reviewed interface requirement explicitly defines otherwise.

**Rationale:** Unrecognized transport data must not silently refresh interface availability.

**Primary verification:** TEST + INSPECTION

---

### FSS-SYS-044 — Future-dated navigation state rejected

**Requirement:** A navigation observation whose source timestamp is later than the receive time of its navigation-arrival runtime event shall not be eligible to replace accepted navigation state.

**Rationale:** Accepted future-dated state could produce negative navigation age and undermine freshness semantics.

**Primary verification:** TEST

---

### FSS-SYS-045 — Both ordering fields shall progress

**Requirement:** When an accepted navigation state already exists, a later navigation observation shall not be eligible to replace it unless both the observation sequence number and source timestamp strictly increase relative to the last accepted observation.

**Rationale:** The initial policy does not silently choose sequence number or source timestamp as the sole ordering authority.

**Primary verification:** TEST

---

### FSS-SYS-046 — Current-state evaluation eligibility

**Requirement:** Accepted navigation state shall be eligible for current-state envelope evaluation only when it is fresh, reports navigation validity `VALID`, reports source health `HEALTHY` or `DEGRADED`, and contains the required finite physical state values.

**Rationale:** Current-state classification requires usable present-state evidence while still permitting diagnostically useful degraded data.

**Primary verification:** TEST

---

### FSS-SYS-047 — Action eligibility requires healthy source state

**Requirement:** Navigation state shall be action eligible only when it is current-state evaluation eligible and its source health is `HEALTHY`.

**Rationale:** Degraded source information may remain diagnostically useful without receiving simulated-action authority.

**Primary verification:** TEST

---

### FSS-SYS-048 — Projection eligibility

**Requirement:** Navigation state shall be projection eligible only when it is action eligible, all required projection inputs are finite, and the configured projection horizon is valid.

**Rationale:** The projection path shall not use information that is less trustworthy than the current-state action path.

**Primary verification:** TEST

---

### FSS-SYS-049 — Invalid source health cannot replace accepted state

**Requirement:** A navigation observation reporting source health `INVALID` shall not replace the last accepted navigation state.

**Rationale:** Explicitly invalid source state must not become authoritative monitor state.

**Primary verification:** TEST

---

### FSS-SYS-050 — Navigation freshness boundary

**Requirement:** When accepted navigation exists, it shall be classified as fresh when navigation age is less than or equal to the configured navigation freshness limit and stale when navigation age is greater than that limit.

**Rationale:** Equality behavior at the freshness boundary must be deterministic.

**Primary verification:** TEST

---

### FSS-SYS-051 — Stale navigation restrictions

**Requirement:** Stale accepted navigation shall not be eligible for current-state envelope evaluation, persistence accumulation, projection, or creation of a new simulated safety-action latch.

**Rationale:** Historical vehicle state shall not be treated as current action evidence.

**Primary verification:** TEST

---

### FSS-SYS-052 — Recovery from stale navigation

**Requirement:** Stale navigation shall recover when a later navigation-arrival event supplies an observation that is accepted as the newest navigation state and is fresh at that event's evaluation time.

**Rationale:** Recovery must be explicit without introducing an unjustified multi-sample confirmation rule.

**Primary verification:** TEST

---

### FSS-SYS-053 — Interface timeout boundary

**Requirement:** After qualifying navigation interface activity has occurred, the interface shall be classified as available by the silence criterion when interface silence is less than or equal to the configured interface timeout limit and timed out when interface silence is greater than that limit.

**Rationale:** Timeout equality behavior must be explicit.

**Primary verification:** TEST

---

### FSS-SYS-054 — Interface state before first qualifying activity

**Requirement:** Before the first qualifying navigation interface activity in an initialized execution, the interface shall be represented as not yet observed rather than timed out.

**Rationale:** Absence of an initial arrival is distinct from expiration of a timeout measured from a prior arrival.

**Primary verification:** TEST

---

### FSS-SYS-055 — Interface timeout does not independently invalidate fresh state

**Requirement:** Interface timeout shall not, by itself, invalidate navigation state that otherwise remains fresh, accepted, and action eligible.

**Rationale:** Navigation freshness and interface silence are separate information dimensions.

**Primary verification:** TEST

---

### FSS-SYS-056 — Interface recovery is independent of navigation recovery

**Requirement:** Interface timeout shall recover upon the next qualifying interface activity, and that recovery shall not by itself establish navigation usability.

**Rationale:** A newly arrived event may restore communication evidence while containing unusable navigation data.

**Primary verification:** TEST

---

### FSS-SYS-057 — Not-evaluated envelope result

**Requirement:** When navigation is not eligible for current-state envelope evaluation, the current envelope result shall be represented as `NOT_EVALUATED` rather than `INSIDE` or `OUTSIDE`.

**Rationale:** Lack of permitted evidence must not be converted into a geometric conclusion.

**Primary verification:** TEST

---

### FSS-SYS-058 — Violation interval start

**Requirement:** An action-eligible current-envelope violation interval shall begin when an action-eligible runtime evaluation first classifies current position as `OUTSIDE`.

**Rationale:** Persistence timing requires an explicit start condition.

**Primary verification:** TEST

---

### FSS-SYS-059 — Loss of action eligibility interrupts persistence

**Requirement:** If action eligibility is lost while a current-envelope violation interval is active, that interval shall end and cease accumulating persistence duration.

**Rationale:** The monitor shall not claim continuous action-eligible evidence through a period in which such evidence is unavailable.

**Primary verification:** TEST

---

### FSS-SYS-060 — Staleness between runtime events interrupts persistence

**Requirement:** If the previously accepted navigation state becomes stale before the next processed runtime event, any active action-eligible violation interval shall be considered interrupted at the modeled freshness boundary even when no runtime event occurred at that exact boundary time.

**Rationale:** Event sparsity must not retroactively bridge an interval in which the underlying state was no longer action eligible.

**Primary verification:** TEST + ANALYSIS

---

### FSS-SYS-061 — Interrupted persistence does not resume

**Requirement:** After an action-eligible violation interval is interrupted, a later action-eligible `OUTSIDE` evaluation shall begin a new violation interval without carrying elapsed duration from the interrupted interval.

**Rationale:** Separate evidence intervals must not be silently combined.

**Primary verification:** TEST

---

### FSS-SYS-062 — Confirming navigation required for persistent violation

**Requirement:** A current-state violation shall not become persistent for simulated-action purposes until a navigation observation newer than the observation that started the active interval is accepted, is action eligible, is evaluated at or after the configured persistence-duration boundary, and classifies current position as `OUTSIDE`.

**Rationale:** Evaluation ticks may measure elapsed time but shall not turn one isolated outside observation into action-authoritative evidence.

**Primary verification:** TEST

---

### FSS-SYS-063 — Persistent-violation equality boundary

**Requirement:** A qualifying newer action-eligible `OUTSIDE` observation evaluated at exact equality with the configured persistence-duration boundary shall be permitted to confirm a persistent violation.

**Rationale:** Equality behavior at the persistence threshold must be deterministic.

**Primary verification:** TEST

---

### FSS-SYS-064 — Projection is advisory

**Requirement:** A projected `OUTSIDE` envelope result shall be observable in decision evidence but shall not create action-eligible persistence or independently create a new simulated safety-action latch.

**Rationale:** The initial constant-velocity projection model is intentionally advisory.

**Primary verification:** TEST

---

### FSS-SYS-065 — Sole initial simulated-action trigger

**Requirement:** In the initial system version, a new `SIMULATED_SAFETY_ACTION_REQUEST` latch shall be created only when an active current-state envelope violation is confirmed persistent in accordance with `FSS-SYS-062` and the confirming navigation observation remains fresh, `HEALTHY`, action eligible, and `OUTSIDE`.

**Rationale:** The initial action path is intentionally limited to affirmative current-state evidence.

**Primary verification:** TEST

---

### FSS-SYS-066 — Non-triggering abnormal conditions

**Requirement:** Navigation staleness, absence of accepted navigation, interface timeout, rejected or invalid observations, duplicate or reordered observations, sequence gaps, `DEGRADED` or `INVALID` source health, projected envelope violation, logging failure, and rejected runtime events shall not independently create a new simulated safety-action latch.

**Rationale:** Loss, degradation, or advisory evidence shall remain observable without being silently promoted into action authority.

**Primary verification:** TEST

---

### FSS-SYS-067 — Rejected runtime-event evidence is separate

**Requirement:** A runtime event rejected before normal runtime evaluation shall produce runtime-event rejection evidence rather than a normal runtime decision record.

**Rationale:** An event that never entered normal evaluation must remain semantically distinct from an accepted runtime evaluation.

**Primary verification:** TEST

---

### FSS-SYS-068 — Rejected runtime-event evidence content

**Requirement:** Runtime-event rejection evidence shall identify, directly or by stable reference, the rejected event category, supplied event time, last accepted runtime-event time, rejection reason, and execution/configuration identity when available.

**Rationale:** Rejected external events must remain reconstructable without pretending that a normal monitor decision occurred.

**Primary verification:** INSPECTION + TEST

---

### FSS-SYS-069 — No required primary reason

**Requirement:** The initial system shall not require one applicable reason identifier to be designated as the authoritative primary reason; all applicable reasons shall remain observable in deterministic external order.

**Rationale:** No justified precedence hierarchy is required for initial monitor behavior.

**Primary verification:** TEST + INSPECTION

---

### FSS-SYS-070 — Recovery after rejected observation

**Requirement:** Rejection of one navigation observation shall not create a separate recovery state that prevents a later observation satisfying all acceptance rules from being accepted normally.

**Rationale:** Input rejection is an event result, not an independently latched monitor condition.

**Primary verification:** TEST

---

### FSS-SYS-071 — Recovery from degraded action eligibility

**Requirement:** After action eligibility is lost because accepted source health is `DEGRADED`, action eligibility shall recover only when a later accepted fresh observation reports source health `HEALTHY`; elapsed violation duration from before the degradation shall not be restored.

**Rationale:** Recovery of trusted action evidence must not silently resurrect an interrupted persistence interval.

**Primary verification:** TEST

---

### FSS-SYS-072 — Explicit reset scope

**Requirement:** Within an initialized execution, explicit reset shall clear the simulated safety-action latch and shall not by itself clear or replace accepted navigation state, runtime-event ordering history, qualifying interface-activity history, or an active violation interval. Reinitialization shall remain a distinct new-execution operation.

**Rationale:** Resetting the simulated-action indication must not silently destroy unrelated execution evidence or historical state. This preserves the distinction between an in-execution latch reset and creation of a new execution.

**Primary verification:** TEST

---

### FSS-SYS-073 — Post-reset re-latch requires new confirming navigation

**Requirement:** After explicit reset clears a latched simulated action, pre-reset navigation evidence and evaluation ticks alone shall not create a new latch. A new simulated-action latch may be created only when a navigation observation accepted after the reset supplies new confirming evidence and all otherwise applicable persistent-violation and action-eligibility conditions are satisfied. If persistence is interrupted after reset, the normal interval-restart rules shall apply.

**Rationale:** Clearing a latch must create a real confirmation boundary. Otherwise unchanged pre-reset evidence could recreate the latch immediately and make explicit reset ineffective.

**Primary verification:** TEST

---

## 15. Requirements Intentionally Deferred

The following remain intentionally unbaselined after Phase 1B:

- exact synthetic navigation freshness duration;
- exact synthetic interface-timeout duration;
- exact synthetic persistence duration;
- exact synthetic projection horizon;
- exact synthetic envelope dimensions;
- monitor operating-state names and transitions beyond the information/eligibility semantics already defined;
- whether an initialization acquisition deadline is required;
- detailed malformed/truncated transport diagnostics;
- concrete serialization and type names for runtime-event rejection evidence;
- the stable external reason-code ordering table;
- complete numeric representation and integer widths;
- complete configuration valid ranges;
- sequence-number rollover behavior.

No implementation shall invent these behaviors merely because this document defers them.

---

## 16. Preliminary Requirement Review Findings

### 16.1 Requirements ready for baseline consideration

The combined Phase 1A and Phase 1B requirements now define:

- deterministic modeled-time handling;
- configuration validity and immutability;
- runtime-event and evaluation cardinality;
- navigation-state acceptance;
- sequence/source-time ordering;
- future-source-time rejection;
- interface-activity semantics;
- freshness and interface-timeout boundaries;
- health-specific monitoring eligibility;
- current and projected envelope semantics;
- action-eligible persistence;
- interruption across stale evidence gaps;
- confirming-navigation requirements;
- advisory projection behavior;
- the sole initial simulated-action trigger;
- latch persistence and reset;
- explicit condition-specific recovery;
- normal decision evidence;
- rejected runtime-event evidence;
- deterministic multi-reason evidence.

### 16.2 Requirements that remain deferred

The project should not yet invent:

- a ceremonial monitor operating-state machine where information-status semantics are already sufficient;
- an initialization acquisition deadline without a justified use case;
- transport serialization details;
- numeric storage widths;
- operationally derived threshold values;
- sequence rollover rules;
- reason-ordering constants before the reason-code catalog is defined.

These remain design or later-requirement work rather than implementation assumptions.

---

## 17. Phase 1B Synchronization Exit Criteria

The system-requirements candidate is synchronized with the reviewed failure policy when:

- `FSS-SYS-001` through `FSS-SYS-071` are unique and stable;
- no Phase 1B requirement contradicts the Phase 0 semantic model;
- future-dated navigation cannot become accepted state;
- interface activity remains distinct from navigation usability;
- freshness and timeout boundary behavior is explicit;
- degraded information may remain diagnostically classifiable without receiving action authority;
- stale or otherwise ineligible information cannot accumulate action persistence;
- persistence cannot bridge a stale interval between runtime events;
- elapsed ticks alone cannot confirm persistent violation;
- a newer accepted action-eligible outside observation is required for persistence confirmation;
- projection remains advisory;
- the initial simulated-action trigger is narrowly defined;
- rejected runtime events remain distinct from normal runtime evaluations;
- recovery behavior is explicit for stale, timeout, degraded, and rejected-observation cases;
- the hazard log contains corresponding failure conditions and mitigations;
- no real operational aerospace thresholds or procedures have entered the requirements.

No implementation work shall begin until the synchronized requirements and hazard analysis pass review.

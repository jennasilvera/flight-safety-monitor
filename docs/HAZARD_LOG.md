# Flight Safety Monitor & Verification Platform
## Preliminary Software Hazard Log

**Status:** Draft for engineering review
**Lifecycle phase:** Phase 1 — Requirements Baseline
**Scope:** Synthetic software failure conditions only
**Certification status:** Educational/safety-oriented analysis; not a formal aerospace system-safety assessment

---

## 1. Purpose

This hazard log records software failure conditions that could undermine the correctness or interpretability of the synthetic monitoring system.

The purpose is not to assign real-world launch safety consequence classifications.

The purpose is to force explicit reasoning about:

- how the monitor could be wrong;
- what synthetic consequence would result;
- what requirements reduce the risk;
- how those requirements can be verified;
- what uncertainty remains.

No real vehicle, public-safety, personnel-safety, or range-safety severity classification is asserted.

---

## 2. Analysis Rules

Each hazard entry contains:

- hazard ID;
- failure condition;
- possible software causes;
- synthetic consequence;
- current mitigations;
- associated requirement IDs;
- verification approach;
- residual limitations/open questions.

Severity/probability scoring is intentionally deferred.

Without a defined synthetic consequence classification scheme, numeric or categorical risk scores would create false precision.

---

## HAZ-001 — Stale navigation treated as current

**Failure condition:** Navigation state older than permitted by the eventual freshness requirement is treated as current usable state.

**Possible software causes:**

- wrong time basis;
- source time confused with receive time;
- stale-state check omitted;
- evaluation occurs only on packet arrival;
- arithmetic/order defect in age calculation.

**Synthetic consequence:**

The monitor may evaluate envelope or projection conditions using state that no longer represents the intended scenario time.

**Current mitigations:**

- explicit modeled time;
- source/receive/evaluation time separation;
- evaluation tick capability;
- explicit navigation-age calculation.

**Associated requirements:**

- FSS-SYS-005
- FSS-SYS-009
- FSS-SYS-019
- FSS-SYS-021
- FSS-SYS-038
- FSS-SYS-040

**Verification approach:**

Boundary tests around the eventual freshness threshold, including advancement of modeled time without navigation arrival.

**Residual limitations / open questions:**

- exact synthetic freshness-limit value remains configuration data;
- final monitor operating-state representation, if any, remains a later design decision.

---

## HAZ-002 — Invalid or corrupted navigation becomes accepted state

**Failure condition:** Malformed, non-finite, duplicate, reordered, or otherwise unacceptable navigation data replaces valid accepted state.

**Possible software causes:**

- incomplete validation;
- validation performed after state mutation;
- sequence/timestamp checks implemented inconsistently;
- non-finite values not rejected.

**Synthetic consequence:**

Later envelope, persistence, projection, or state-machine behavior may operate from invalid state.

**Current mitigations:**

- explicit rejection semantics;
- rejected-state preservation;
- finite-value rule;
- timestamp and sequence progression rules.

**Associated requirements:**

- FSS-SYS-012
- FSS-SYS-013
- FSS-SYS-014
- FSS-SYS-015
- FSS-SYS-016
- FSS-SYS-017
- FSS-SYS-018
- FSS-SYS-044
- FSS-SYS-045
- FSS-SYS-049
- FSS-SYS-070

**Verification approach:**

Invalid-input, NaN/infinity, duplicate, reordering, and source-time regression tests.

**Residual limitations / open questions:**

- conflicting timestamp/sequence cases still need detailed acceptance requirements;
- malformed serialization behavior is not yet defined.

---

## HAZ-003 — Interface timeout cannot be detected after input stops

**Failure condition:** The monitor cannot recognize prolonged input silence because evaluation only occurs when new navigation arrives.

**Possible software causes:**

- event model tied exclusively to packet arrival;
- wall-clock polling hidden inside I/O code;
- no explicit modeled-time advancement.

**Synthetic consequence:**

The monitor may continue to present obsolete interface status indefinitely in a scenario where communication has stopped.

**Current mitigations:**

- explicit evaluation-tick event;
- separate interface-silence concept;
- modeled-time control.

**Associated requirements:**

- FSS-SYS-005
- FSS-SYS-009
- FSS-SYS-020
- FSS-SYS-021
- FSS-SYS-040
- FSS-SYS-041
- FSS-SYS-042
- FSS-SYS-053
- FSS-SYS-054
- FSS-SYS-055
- FSS-SYS-056

**Verification approach:**

Provide one qualifying interface event, then advance modeled time using ticks with no further navigation arrivals.

**Residual limitations / open questions:**

- exact synthetic timeout-limit value remains configuration data;
- malformed or unrecognized transport diagnostics remain interface-design work.

---

## HAZ-004 — Regressive modeled time mutates monitor state

**Failure condition:** An event with modeled time earlier than the previous accepted runtime event changes normal monitor state or participates in normal evaluation.

**Possible software causes:**

- event-time validation omitted;
- validation occurs after mutation;
- separate modules maintain inconsistent time histories.

**Synthetic consequence:**

Freshness, persistence, timeout, and event ordering can become internally inconsistent.

**Current mitigations:**

- monotonic event-time semantics;
- reject-before-normal-evaluation rule;
- state-preservation requirement.

**Associated requirements:**

- FSS-SYS-006
- FSS-SYS-007
- FSS-SYS-008
- FSS-SYS-038

**Verification approach:**

Inject regressive-time navigation and tick events and verify no normal monitor-state advancement.

**Residual limitations / open questions:**

- external evidence format for rejected runtime events remains open.

---

## HAZ-005 — Persistent envelope result depends on packet rate

**Failure condition:** The same continuous boundary excursion produces different persistent-violation results solely because navigation update frequency differs.

**Possible software causes:**

- persistence implemented as packet count;
- timer reset tied to arrival count;
- source rate implicitly assumed.

**Synthetic consequence:**

Scenario outcome changes even when the underlying modeled excursion duration is identical.

**Current mitigations:**

- elapsed modeled-time persistence;
- explicit evaluation time;
- transient-return semantics.

**Associated requirements:**

- FSS-SYS-005
- FSS-SYS-026
- FSS-SYS-027
- FSS-SYS-038
- FSS-SYS-058
- FSS-SYS-059
- FSS-SYS-060
- FSS-SYS-061
- FSS-SYS-062
- FSS-SYS-063

**Verification approach:**

Run equivalent excursion durations using different navigation update rates and compare logical results.

**Residual limitations / open questions:**

- exact synthetic persistence duration remains configuration data;
- verification must cover interruption at a freshness boundary occurring between runtime events.

---

## HAZ-006 — Projection uses unusable source state

**Failure condition:** Synthetic projected position is calculated from rejected, invalid, stale, or otherwise ineligible navigation state.

**Possible software causes:**

- projection invoked before eligibility check;
- stale accepted state not distinguished from eligible state;
- validation result not propagated.

**Synthetic consequence:**

The monitor may report a projected synthetic envelope condition that is not supported by valid source information.

**Current mitigations:**

- projection eligibility requirement;
- constant-velocity model;
- decision evidence records projection status.

**Associated requirements:**

- FSS-SYS-019
- FSS-SYS-028
- FSS-SYS-029
- FSS-SYS-030
- FSS-SYS-034
- FSS-SYS-046
- FSS-SYS-047
- FSS-SYS-048
- FSS-SYS-050
- FSS-SYS-051
- FSS-SYS-064

**Verification approach:**

Attempt projection with valid, invalid, rejected, stale, and boundary-condition source states.

**Residual limitations / open questions:**

- exact synthetic freshness-limit and projection-horizon values remain configuration data.

---

## HAZ-007 — Simulated action latch clears unintentionally

**Failure condition:** A previously latched simulated safety action disappears without explicit reset or reinitialization.

**Possible software causes:**

- latch stored as transient decision output only;
- nominal navigation overwrites latch state;
- explicit-reset scope or post-reset re-latch semantics implemented incorrectly.

**Synthetic consequence:**

Decision history becomes internally inconsistent and the meaning of "latched" is violated.

**Current mitigations:**

- explicit latch persistence semantics;
- explicit reset clearing behavior.

**Associated requirements:**

- FSS-SYS-036
- FSS-SYS-037
- FSS-SYS-038
- FSS-SYS-065
- FSS-SYS-066
- FSS-SYS-072
- FSS-SYS-073

**Verification approach:**

Verify subsequent nominal and abnormal events cannot clear a valid latch before explicit reset. Verify explicit reset clears the latch without implicitly clearing accepted navigation, runtime-event history, interface-activity history, or an active persistence interval.

**Residual limitations / open questions:**

- the sole initial latch trigger is now defined as a confirmed persistent action-eligible current-state envelope violation;
- exact synthetic persistence and envelope configuration values remain test configuration data.

---

## HAZ-008 — Simultaneous abnormal conditions lose evidence

**Failure condition:** When multiple abnormal conditions coexist, one condition suppresses evidence of the others.

**Possible software causes:**

- single mutable reason field;
- early-return logic;
- output order tied to implementation traversal order;
- primary reason treated as exclusive reason.

**Synthetic consequence:**

An engineer cannot reconstruct all conditions that influenced or coexisted with the decision.

**Current mitigations:**

- multiple-reason capability;
- stable reason identifiers;
- deterministic reason ordering;
- structured decision evidence.

**Associated requirements:**

- FSS-SYS-031
- FSS-SYS-032
- FSS-SYS-033
- FSS-SYS-034
- FSS-SYS-069

**Verification approach:**

Construct scenarios containing simultaneous independent abnormal conditions and verify all applicable reasons remain observable.

**Residual limitations / open questions:**

- the initial system intentionally does not designate an authoritative primary reason;
- the stable external reason-code ordering table remains to be defined.

---

## HAZ-009 — Invalid configuration enters monitoring

**Failure condition:** Monitoring begins using incomplete, contradictory, or invalid configuration.

**Possible software causes:**

- partial validation;
- lazy validation during evaluation;
- invalid bounds accepted;
- configuration changed after initialization.

**Synthetic consequence:**

Every downstream monitoring result may become unreliable or irreproducible.

**Current mitigations:**

- configuration validation before monitoring;
- invalid configuration blocks monitoring;
- configuration immutability;
- envelope ordering rule;
- configuration identity in evidence.

**Associated requirements:**

- FSS-SYS-001
- FSS-SYS-002
- FSS-SYS-003
- FSS-SYS-004
- FSS-SYS-023

**Verification approach:**

Invalid/missing/inverted configuration tests plus attempts to mutate active configuration.

**Residual limitations / open questions:**

- complete valid-range rules for configuration are not yet defined.

---

## HAZ-010 — Decision cannot be reconstructed

**Failure condition:** The system produces a decision but records insufficient evidence to determine why it occurred.

**Possible software causes:**

- free-form logs used as authoritative evidence;
- configuration identity omitted;
- prior/resulting state omitted;
- rejected-input evidence lost;
- reason identifiers unstable.

**Synthetic consequence:**

Verification failures and unexpected decisions cannot be reliably diagnosed or audited.

**Current mitigations:**

- one decision record per runtime evaluation;
- stable machine-readable reasons;
- structured reconstructability requirement;
- configuration identity requirement.

**Associated requirements:**

- FSS-SYS-004
- FSS-SYS-010
- FSS-SYS-011
- FSS-SYS-031
- FSS-SYS-032
- FSS-SYS-033
- FSS-SYS-034
- FSS-SYS-067
- FSS-SYS-068

**Verification approach:**

Scenario-level evidence review plus automated completeness checks once record schema is baselined.

**Residual limitations / open questions:**

- rejected runtime events now use separate rejection-evidence semantics;
- concrete serialization and type names for both evidence forms remain design work.

---

## HAZ-011 — Incorrect synthetic envelope classification

**Failure condition:** A valid current or projected position is incorrectly classified relative to the configured synthetic envelope.

**Possible software causes:**

- incorrect minimum/maximum comparison;
- treating an exact boundary point as outside;
- axis mix-up;
- incorrect use of inclusive/exclusive comparison;
- undocumented numerical tolerance;
- inconsistent current-state and projected-state boundary rules.

**Synthetic consequence:**

The monitor may report an envelope condition that does not match the configured synthetic geometry.

**Current mitigations:**

- deliberately simple axis-aligned envelope;
- explicit minimum/maximum ordering;
- explicit exact-boundary semantics;
- shared boundary semantics for current and projected state.

**Associated requirements:**

- FSS-SYS-022
- FSS-SYS-023
- FSS-SYS-024
- FSS-SYS-025
- FSS-SYS-030

**Verification approach:**

Exercise, for every Cartesian axis:

- strictly inside;
- exactly at minimum;
- exactly at maximum;
- immediately below minimum using representable test values;
- immediately above maximum using representable test values;
- multi-axis violations;
- equivalent projected-state cases.

**Residual limitations / open questions:**

- final numeric representation is not yet defined;
- any future computation-specific tolerance would require separate justification and dedicated tests.

---

## HAZ-012 — Diagnostic path alters monitoring behavior

**Failure condition:** Logging, diagnostic recording, or other observability behavior changes the logical monitoring decision sequence.

**Possible software causes:**

- logging mutates monitor state;
- exception/failure in diagnostic output interrupts normal decision logic;
- decision behavior depends on log success;
- diagnostic timestamps are reused as modeled safety time;
- observability introduces uncontrolled ordering or timing dependency.

**Synthetic consequence:**

Identical semantic inputs may produce different monitoring decisions depending on diagnostic-system behavior.

**Current mitigations:**

- logging is excluded from core decision semantics;
- modeled safety time is externally controlled;
- deterministic decision-sequence requirement.

**Associated requirements:**

- FSS-SYS-005
- FSS-SYS-038
- FSS-SYS-039
- FSS-SYS-040

**Verification approach:**

Execute equivalent monitor scenarios with diagnostic output:

- enabled;
- disabled;
- successfully recorded;
- deliberately failed at the diagnostic boundary where testable;

and verify that logical runtime decision records remain semantically equivalent.

**Residual limitations / open questions:**

- final logging architecture is not yet defined;
- failure-injection mechanism for diagnostic output will depend on later architecture.

---

## HAZ-013 — Future-dated navigation becomes accepted state

**Failure condition:** A navigation observation whose source timestamp is later than its receive time becomes accepted navigation state.

**Possible software causes:**

- source/receive time comparison omitted;
- comparison direction reversed;
- temporal validation occurs after state mutation.

**Synthetic consequence:**

Navigation age may become negative and freshness-dependent behavior may be evaluated from temporally invalid state.

**Current mitigations:**

- explicit future-source-time rejection;
- modeled source and receive times share one synthetic monotonic scenario-time domain.

**Associated requirements:**

- FSS-SYS-019
- FSS-SYS-044
- FSS-SYS-045
- FSS-SYS-050

**Verification approach:**

Test source time less than, equal to, and greater than receive time, including cases where sequence number otherwise advances normally.

**Residual limitations / open questions:**

- concrete timestamp numeric representation remains design work.

---

## HAZ-014 — Persistence bridges a stale evidence gap

**Failure condition:** A violation interval continues across modeled time during which the previously accepted navigation state was no longer fresh.

**Possible software causes:**

- staleness checked only when a tick occurs;
- later outside observation reuses the original interval start;
- freshness loss between events is ignored.

**Synthetic consequence:**

A simulated action may be based on claimed continuous evidence that did not actually remain action eligible.

**Current mitigations:**

- loss of action eligibility ends persistence;
- staleness between events interrupts the interval at the freshness boundary;
- interrupted persistence cannot resume.

**Associated requirements:**

- FSS-SYS-050
- FSS-SYS-051
- FSS-SYS-059
- FSS-SYS-060
- FSS-SYS-061

**Verification approach:**

Start an outside interval, allow the accepted state to become stale without an intervening event, then provide a later fresh outside observation and verify that a new interval begins.

**Residual limitations / open questions:**

- exact synthetic freshness and persistence durations remain configuration data.

---

## HAZ-015 — Elapsed ticks create a false persistent violation

**Failure condition:** Evaluation ticks alone cause one isolated outside observation to become persistent and create a new simulated action.

**Possible software causes:**

- elapsed duration treated as sufficient confirmation;
- triggering event category ignored;
- confirming-observation requirement omitted.

**Synthetic consequence:**

The system may create a simulated action without newer affirmative navigation evidence.

**Current mitigations:**

- ticks may age an interval but cannot confirm persistence;
- newer accepted action-eligible outside navigation is required at or after the threshold.

**Associated requirements:**

- FSS-SYS-058
- FSS-SYS-062
- FSS-SYS-063
- FSS-SYS-065

**Verification approach:**

Start an outside interval and advance beyond the persistence boundary using ticks only; verify no new latch, then provide qualifying newer outside navigation and verify confirmation.

**Residual limitations / open questions:**

- exact synthetic persistence duration remains configuration data.

---

## HAZ-016 — Simulated action derives from degraded, stale, invalid, or advisory information

**Failure condition:** Information lacking action eligibility independently contributes to creation of a new simulated safety-action latch.

**Possible software causes:**

- health eligibility omitted;
- stale state reused;
- rejected state leaked into evaluation;
- projected violation treated as action-authoritative;
- timeout or data loss directly mapped to action.

**Synthetic consequence:**

A simulated action may occur without the affirmative evidence required by the reviewed failure policy.

**Current mitigations:**

- explicit current/action/projection eligibility levels;
- stale-state restrictions;
- advisory-only projection;
- enumerated non-triggering conditions.

**Associated requirements:**

- FSS-SYS-046
- FSS-SYS-047
- FSS-SYS-048
- FSS-SYS-049
- FSS-SYS-051
- FSS-SYS-055
- FSS-SYS-057
- FSS-SYS-064
- FSS-SYS-065
- FSS-SYS-066

**Verification approach:**

Exercise every non-triggering condition individually and in combinations while confirming that applicable diagnostic reasons remain visible.

**Residual limitations / open questions:**

- final stable reason-code catalog remains to be defined.

---

## HAZ-017 — Valid confirmed persistent violation fails to latch action

**Failure condition:** A qualifying current-state violation remains action eligible through the required evidence interval, receives the required newer outside confirmation at or after the persistence boundary, but the simulated action does not latch.

**Possible software causes:**

- equality boundary implemented incorrectly;
- persistence start time reset unexpectedly;
- confirming observation not associated with active interval;
- latch transition omitted.

**Synthetic consequence:**

The system fails to produce the simulated action required by its own synthetic policy.

**Current mitigations:**

- explicit interval-start semantics;
- confirming-observation requirement;
- equality-boundary requirement;
- sole action-trigger requirement.

**Associated requirements:**

- FSS-SYS-058
- FSS-SYS-062
- FSS-SYS-063
- FSS-SYS-065
- FSS-SYS-036

**Verification approach:**

Test confirmation immediately before, exactly at, and immediately after the persistence boundary, plus subsequent evaluations verifying latch persistence.

**Residual limitations / open questions:**

- exact synthetic persistence duration remains configuration data.

---

## HAZ-018 — Recovery occurs before the required evidence exists

**Failure condition:** A stale, degraded, timed-out, or rejected-input condition is treated as recovered without satisfying its defined recovery rule.

**Possible software causes:**

- one generic recovery flag used for unrelated conditions;
- interface arrival mistaken for navigation recovery;
- degraded health cleared without a newer healthy accepted observation;
- rejected input clears prior condition by side effect.

**Synthetic consequence:**

Monitoring functions may resume using information that has not actually regained the required eligibility.

**Current mitigations:**

- condition-specific recovery rules;
- interface recovery separated from navigation recovery;
- stale recovery requires accepted fresh state;
- degraded action eligibility requires accepted fresh `HEALTHY` state.

**Associated requirements:**

- FSS-SYS-052
- FSS-SYS-056
- FSS-SYS-070
- FSS-SYS-071

**Verification approach:**

For each recoverable condition, provide near-miss inputs that fail one recovery criterion, followed by an input satisfying all criteria.

**Residual limitations / open questions:**

- final monitor operating-state representation remains a later design decision.

---

## HAZ-019 — Interface activity is conflated with navigation-state acceptance

**Failure condition:** Interface availability is refreshed only by accepted vehicle state, or unusable navigation is incorrectly treated as proof that no interface activity occurred.

**Possible software causes:**

- shared boolean used for communication and navigation validity;
- interface receive time updated only on state replacement;
- malformed and recognized-but-rejected inputs not distinguished.

**Synthetic consequence:**

Interface timeout and navigation-usability evidence become semantically incorrect.

**Current mitigations:**

- qualifying interface activity is based on accepted recognizable navigation-arrival events;
- contained observation acceptance is separate;
- unrecognized malformed transport does not qualify.

**Associated requirements:**

- FSS-SYS-041
- FSS-SYS-042
- FSS-SYS-043
- FSS-SYS-053
- FSS-SYS-054
- FSS-SYS-056

**Verification approach:**

Compare accepted usable observations, recognized arrival events with rejected observations, evaluation ticks, and unrecognized malformed transport.

**Residual limitations / open questions:**

- detailed transport parsing and malformed-input diagnostics remain interface-design work.

---

## HAZ-020 — Rejected runtime event is represented as a normal monitor decision

**Failure condition:** An event rejected before normal evaluation is emitted as though a normal runtime evaluation occurred.

**Possible software causes:**

- rejection and decision evidence share indistinguishable semantics;
- event-time validation performed inside normal decision generation;
- rejected event incorrectly increments logical evaluation identity.

**Synthetic consequence:**

Verification evidence falsely implies that the monitor evaluated and transitioned on an event that should have been rejected at the runtime boundary.

**Current mitigations:**

- rejected runtime-event evidence is distinct from normal decision records;
- minimum rejection-evidence content is specified.

**Associated requirements:**

- FSS-SYS-007
- FSS-SYS-008
- FSS-SYS-010
- FSS-SYS-011
- FSS-SYS-067
- FSS-SYS-068

**Verification approach:**

Inject regressive-time runtime events and verify separate rejection evidence, no normal runtime evaluation, and no normal state advancement.

**Residual limitations / open questions:**

- final evidence serialization and concrete type names remain design work.

---

## HAZ-021 — Simulated action re-latches after reset without new confirming evidence

**Failure condition:** After an explicit in-execution reset clears the simulated-action latch, the latch is recreated solely from pre-reset navigation evidence, previously accumulated persistence, or an evaluation tick without a newly accepted post-reset navigation observation.

**Possible software causes:**

- reset clears the latch but establishes no post-reset confirmation boundary;
- a previously persistent violation automatically reasserts the latch on the next evaluation;
- an evaluation tick is incorrectly treated as new confirming navigation evidence;
- persistence history and latch-confirmation evidence are conflated.

**Synthetic consequence:**

Explicit reset becomes ineffective or misleading because the simulated action can immediately reappear without new confirming navigation evidence.

**Current mitigations:**

- explicit reset has narrow in-execution scope;
- persistence state and latch state remain distinct;
- ticks cannot confirm persistent violation;
- post-reset re-latch requires newly accepted navigation evidence;
- normal persistence-interruption rules still apply after reset.

**Associated requirements:**

- FSS-SYS-037
- FSS-SYS-062
- FSS-SYS-065
- FSS-SYS-066
- FSS-SYS-072
- FSS-SYS-073

**Verification approach:**

Create a valid latched persistent violation, issue explicit reset, and verify the latch clears while unrelated execution state remains intact. Verify evaluation ticks cannot recreate the latch. Then provide a navigation observation accepted after reset and verify re-latching occurs only when all otherwise applicable persistent-violation and action-eligibility conditions are satisfied.

**Residual limitations / open questions:**

- concrete reset API representation remains Phase 2 interface-design work;
- reinitialization remains a separate new-execution operation.

---

## HAZ-022 — Projection arithmetic overflow produces misleading evidence

**Failure condition:** Projection is eligible and all required inputs are finite, but constant-velocity arithmetic produces a non-finite projected Cartesian position that is then treated as a valid projected envelope result.

**Possible software causes:**

- multiplication or addition overflows binary64 range;
- projected-position finiteness is not checked after arithmetic;
- infinity is allowed to flow into ordinary envelope comparisons;
- projection failure is accidentally coupled into persistence or action logic.

**Synthetic consequence:**

The monitor may emit misleading projected-envelope evidence or allow a failed advisory calculation to influence behavior outside projection.

**Current mitigations:**

- projection remains advisory;
- projection requires eligible finite source inputs;
- computed projected position is checked for finiteness before envelope classification;
- a non-finite computed projection produces `NOT_EVALUATED` projected-envelope status and a stable reason identifier;
- failed projection does not contribute to current-state persistence or create a simulated-action latch.

**Associated requirements:**

- FSS-SYS-028
- FSS-SYS-030
- FSS-SYS-034
- FSS-SYS-048
- FSS-SYS-057
- FSS-SYS-064
- FSS-SYS-066
- FSS-SYS-074

**Verification approach:**

Use finite synthetic position, velocity, and projection-horizon values chosen to force binary64 projection arithmetic beyond finite range. Verify the projected result is `NOT_EVALUATED`, the stable failure reason is present, and current-state persistence and simulated-action behavior are unchanged by the failed projection.

**Residual limitations / open questions:**

- no arbitrary operational position, velocity, or projection-horizon maximum is introduced solely to avoid this case;
- complete application-specific configuration policy ranges remain deferred.

---

## 3. Cross-Cutting Open Hazard Questions

The principal Phase 1B failure-policy questions are now resolved.

The following questions remain for later requirements/design work:

1. Does the system need an explicit monitor operating-state machine beyond the information-status and eligibility model?
2. Is an initialization acquisition deadline required before the first qualifying navigation activity?
3. What exact malformed/truncated transport diagnostics shall the external interface expose?
4. What stable ordering shall the final reason-code catalog use?
5. What numeric representations and valid configuration ranges shall be used?
6. How, if at all, shall sequence-number rollover be supported?
7. What concrete serialization shall represent normal decisions and rejected runtime-event evidence?
8. What diagnostic behavior is required if external evidence output itself cannot be recorded?

None of these open questions authorizes implementation to invent operational aerospace thresholds or procedures.

---

## 4. Preliminary Review Finding

The synchronized Phase 1 hazard analysis now supports requirements governing:

- controlled modeled time;
- navigation-state validation and temporal ordering;
- rejection of future-dated navigation;
- separation of interface activity from navigation usability;
- freshness and timeout semantics;
- health-specific monitoring eligibility;
- explicit envelope-boundary classification;
- action-eligible persistence;
- interruption across stale evidence gaps;
- confirming-navigation evidence;
- advisory-only projection;
- containment of non-finite projection arithmetic results;
- the sole initial simulated-action trigger;
- latch persistence;
- condition-specific recovery;
- structured normal and rejected-event evidence;
- deterministic multi-reason handling;
- diagnostic isolation from decision semantics;
- configuration validation.

The hazard analysis still does not justify inventing:

- operational aerospace thresholds;
- a monitor operating-state machine solely for architectural convenience;
- an initialization acquisition deadline without a defined need;
- transport serialization details;
- numeric representations or sequence rollover rules without interface requirements.

The next lifecycle step may define architecture only after the synchronized Phase 1 requirements, failure policy, and hazard analysis pass final traceability review.

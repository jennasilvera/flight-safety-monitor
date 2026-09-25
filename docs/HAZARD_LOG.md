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

- exact freshness threshold not yet defined;
- stale-navigation monitor consequence not yet defined.

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

**Verification approach:**

Provide one qualifying interface event, then advance modeled time using ticks with no further navigation arrivals.

**Residual limitations / open questions:**

- qualifying interface activity is not yet defined;
- timeout threshold and timeout consequence are not yet defined.

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

**Verification approach:**

Run equivalent excursion durations using different navigation update rates and compare logical results.

**Residual limitations / open questions:**

- behavior during loss of usable navigation while a persistence interval is active remains unresolved.

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

**Verification approach:**

Attempt projection with valid, invalid, rejected, stale, and boundary-condition source states.

**Residual limitations / open questions:**

- exact projection eligibility freshness rule remains open.

---

## HAZ-007 — Simulated action latch clears unintentionally

**Failure condition:** A previously latched simulated safety action disappears without explicit reset or reinitialization.

**Possible software causes:**

- latch stored as transient decision output only;
- nominal navigation overwrites latch state;
- reset semantics unclear.

**Synthetic consequence:**

Decision history becomes internally inconsistent and the meaning of "latched" is violated.

**Current mitigations:**

- explicit latch persistence semantics;
- explicit reset clearing behavior.

**Associated requirements:**

- FSS-SYS-036
- FSS-SYS-037
- FSS-SYS-038

**Verification approach:**

Once a future requirement defines a valid latch trigger, verify subsequent nominal and abnormal events cannot clear the latch before explicit reset.

**Residual limitations / open questions:**

- latch-trigger conditions are intentionally undefined;
- no test can be complete until at least one valid latch trigger is baselined.

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

**Verification approach:**

Construct scenarios containing simultaneous independent abnormal conditions and verify all applicable reasons remain observable.

**Residual limitations / open questions:**

- primary-reason precedence is not yet defined.

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

**Verification approach:**

Scenario-level evidence review plus automated completeness checks once record schema is baselined.

**Residual limitations / open questions:**

- rejected runtime-event evidence schema remains undefined;
- final decision-record serialization remains undefined.

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

## 3. Cross-Cutting Open Hazard Questions

Before state-machine and simulated-action requirements are baselined, review must answer:

1. What monitor consequence follows stale navigation?
2. What monitor consequence follows interface timeout?
3. What combination of failures, if any, requires a simulated safety-action request?
4. Which conditions are recoverable?
5. What evidence is required before leaving a degraded condition?
6. What happens to active violation persistence when usable navigation disappears?
7. Which simultaneous conditions require distinct primary versus contributing reasons?
8. Does a separate `DEGRADED` monitor operating state provide necessary behavior?
9. What behavior is required before the first accepted navigation observation?
10. What evidence format represents rejected runtime events?

---

## 4. Preliminary Review Finding

The hazard analysis supports baselining the requirements that govern:

- controlled time;
- invalid-state rejection;
- state preservation;
- explicit timeout detectability;
- elapsed-time persistence;
- projection eligibility;
- explicit envelope-boundary classification;
- structured evidence;
- deterministic reason handling;
- diagnostic isolation from decision semantics;
- configuration validation.

The hazard analysis does **not** yet justify selecting:

- a final operating-state machine;
- stale-navigation recovery behavior;
- interface-timeout recovery behavior;
- simulated safety-action triggers;
- primary reason precedence.

Those behaviors require a dedicated next review increment rather than implementation inference.

# Flight Safety Monitor & Verification Platform
## Phase 1B — Failure and Decision Policy

**Status:** Draft for engineering review
**Lifecycle phase:** Phase 1 — Requirements Baseline
**Scope:** Synthetic failure handling, recovery, monitoring eligibility, and simulated-action policy
**Implementation status:** No implementation authorized by this document

---

## 1. Purpose

This document resolves the principal failure-policy questions deliberately deferred by the Phase 0 semantic model and the Phase 1A requirements/hazard analysis.

It defines policy for:

- qualifying interface activity;
- navigation acceptance and usability;
- stale navigation;
- interface timeout;
- source-reported health;
- sequence gaps and ordering failures;
- persistence interruption;
- projection eligibility;
- simulated safety-action triggering;
- latch behavior;
- recovery;
- rejected runtime-event evidence;
- simultaneous conditions.

The document continues to describe a synthetic, non-operational monitor.

No real flight-termination thresholds, range-safety criteria, protected operational parameters, or hazardous hardware behavior are defined.

---

## 2. Policy Design Principles

### 2.1 Preserve information distinctions

The policy shall not collapse the following into one condition:

- interface activity;
- observation acceptance;
- navigation freshness;
- source-reported health;
- current envelope status;
- projected envelope status;
- persistence status;
- simulated action state.

Different combinations of these facts may coexist.

### 2.2 Do not infer knowledge during data loss

When usable navigation is unavailable, the monitor shall not pretend that an earlier outside-envelope state is known to have continued.

Likewise, it shall not pretend that the earlier condition resolved.

Loss of usable information is represented as loss of monitoring evidence.

### 2.3 Simulated action requires affirmative synthetic evidence

For the initial system version, a new simulated safety-action request shall require affirmative, usable synthetic vehicle-state evidence.

Loss of data, invalid data, interface timeout, or projection alone shall not create a new simulated safety-action latch.

This is a project policy for the synthetic system, not a claim about operational aerospace practice.

### 2.4 Recovery must be explicit

A condition clears only when its defined recovery criterion is met.

No condition may disappear merely because later code overwrites state.

---

## 3. Runtime-Event Acceptance Policy

A runtime event is either:

- accepted for normal runtime processing; or
- rejected at the runtime-event boundary.

A runtime event whose modeled time is earlier than the previously accepted runtime-event time is rejected.

A rejected runtime event:

- does not advance normal monitor state;
- does not perform a normal runtime evaluation;
- does not produce a normal runtime `DecisionRecord`;
- does not count as qualifying interface activity.

Rejected runtime-event evidence is handled separately as defined in Section 16.

---

## 4. Qualifying Interface Activity

For the initial system version, a navigation-arrival runtime event counts as qualifying interface activity when:

1. the runtime event itself is accepted;
2. it is recognized as a navigation-arrival event.

The contained navigation observation does **not** need to be accepted as vehicle state for the event to demonstrate interface activity.

Therefore an accepted arrival event containing:

- invalid navigation validity;
- degraded source health;
- invalid source health;
- a duplicate sequence number;
- a regressive sequence number;
- a regressive source timestamp;
- non-finite physical state;

may still demonstrate that the interface is active.

This distinction is intentional.

### 4.1 Rationale

Interface availability answers:

> Is the monitor receiving recognizable navigation-arrival activity?

Navigation usability answers:

> Does the monitor currently possess vehicle-state information permitted for the relevant monitoring function?

Those are different questions.

### 4.2 Malformed transport input

Bytes or transport input that cannot be recognized as a navigation-arrival runtime event are outside the deterministic core event model.

Later interface requirements shall define diagnostics for malformed or truncated transport input.

Such malformed transport input shall not qualify as navigation interface activity unless a later reviewed interface requirement explicitly establishes otherwise.

---

## 5. Navigation Observation Acceptance

A navigation observation is eligible to replace the last accepted navigation state only when all applicable acceptance conditions are satisfied.

For the initial version, these conditions include:

- accepted runtime-event time;
- supported observation version under the eventual interface definition;
- finite physical numeric values;
- navigation validity equal to `VALID`;
- source health not equal to `INVALID`;
- source timestamp not later than the navigation-arrival event's receive time;
- source timestamp strictly greater than the last accepted source timestamp, if an accepted observation exists;
- sequence number strictly greater than the last accepted sequence number, if an accepted observation exists.

### 5.1 Sequence gaps

A forward sequence-number gap does not by itself prevent observation acceptance.

If all other acceptance conditions are satisfied, the newer observation may be accepted.

The gap remains observable as missing-update evidence.

### 5.2 Duplicate and regressive observations

An observation with:

- equal sequence number;
- lower sequence number;
- equal source timestamp;
- lower source timestamp

is not eligible to replace the last accepted navigation state.

### 5.3 Future-dated source time

A navigation observation whose source timestamp is later than its navigation-arrival receive time is not eligible to replace accepted navigation state.

Such an arrival may still count as qualifying interface activity when the runtime event itself is accepted.

Accepted navigation state therefore cannot produce negative navigation age under the modeled-time definitions.

### 5.4 Conflicting sequence and source time

Both sequence progression and source-time progression are required.

Therefore:

- newer sequence with non-newer source time is rejected for state replacement;
- newer source time with non-newer sequence is rejected for state replacement.

This avoids silently choosing one ordering field as authoritative.

---

## 6. Source-Reported Health Policy

The source health domain remains:

- `HEALTHY`
- `DEGRADED`
- `INVALID`

These values describe synthetic source-reported health, not monitor operating state.

### 6.1 HEALTHY

A `HEALTHY` observation may be accepted and, if otherwise usable, may participate in:

- current-state envelope evaluation;
- persistence accumulation;
- projection;
- simulated-action triggering.

### 6.2 DEGRADED

A `DEGRADED` observation may be accepted as the newest navigation state when all other acceptance checks pass.

A fresh accepted degraded observation may participate in:

- current-state envelope classification;
- diagnostic decision evidence.

A degraded observation shall not participate in:

- action-eligible persistence accumulation;
- synthetic state projection;
- creation of a new simulated safety-action latch.

### 6.3 INVALID

An `INVALID` observation shall not replace accepted navigation state.

Its arrival may still count as qualifying interface activity when the runtime event itself is accepted.

### 6.4 Rationale

This gives `DEGRADED` behavior distinct from both `HEALTHY` and `INVALID` without inventing a monitor operating state merely to reuse the same word.

---

## 7. Navigation Freshness Policy

When an accepted navigation state exists:

`navigation_age = evaluation_time - last_accepted_source_time`

Configuration defines a synthetic `navigation_freshness_limit`.

### 7.1 Boundary semantics

Navigation is:

- **fresh** when `navigation_age <= navigation_freshness_limit`;
- **stale** when `navigation_age > navigation_freshness_limit`.

The exact configured duration remains synthetic and is not defined by this document.

### 7.2 No accepted state

Before the first navigation observation is accepted, usable navigation state does not exist.

This is distinct from stale navigation.

### 7.3 Consequence of stale navigation

A stale accepted navigation state remains historical evidence but is not eligible for:

- current-state envelope classification as a current monitoring result;
- persistence accumulation;
- projection;
- creation of a new simulated safety-action latch.

A stale condition shall be observable in the runtime decision record.

### 7.4 Recovery from stale navigation

Stale navigation recovers when a later navigation-arrival event provides an observation that:

- is accepted as the newest navigation state;
- is fresh at the evaluation time associated with that event.

No arbitrary multi-sample recovery count is introduced in the initial version.

If later discontinuity requirements justify a confirmation interval, recovery policy shall be revised through change control.

---

## 8. Interface-Timeout Policy

Interface silence is measured from the receive time of the last qualifying interface activity.

Configuration defines a synthetic `interface_timeout_limit`.

### 8.1 Boundary semantics

The interface is:

- **available by silence criterion** when `interface_silence <= interface_timeout_limit`;
- **timed out** when `interface_silence > interface_timeout_limit`.

### 8.2 Before first qualifying activity

Before any qualifying navigation-arrival event has occurred, the interface is considered **not yet observed**, not timed out.

A later requirement may define an initialization acquisition deadline if one is needed.

### 8.3 Consequence of interface timeout

Interface timeout shall:

- remain distinct from navigation staleness;
- be observable in the runtime decision record;
- prevent any assumption that new source information is arriving.

Interface timeout shall not by itself:

- create a new simulated safety-action latch;
- prove that the synthetic vehicle is inside the envelope;
- prove that the synthetic vehicle is outside the envelope;
- invalidate an otherwise fresh, accepted, action-eligible navigation state.

Navigation freshness and source-health rules independently determine whether the last accepted state remains action eligible.

A later confirming navigation observation is still required before persistence can create a new simulated safety-action latch.

### 8.4 Recovery from interface timeout

Interface timeout recovers on the next qualifying interface activity.

Recovery of interface availability does not imply recovery of navigation usability.

For example, an accepted arrival event may restore interface activity while containing a navigation observation that is rejected.

---

## 9. Monitoring Eligibility

For the initial system version, the document distinguishes three levels of navigation use.

### 9.1 Accepted state

An observation has passed state-acceptance rules and is stored as the newest accepted navigation state.

### 9.2 Current-state evaluation eligible

Accepted navigation is eligible for current-state envelope classification when:

- it is fresh;
- navigation validity is `VALID`;
- source health is `HEALTHY` or `DEGRADED`;
- required numeric state is finite.

### 9.3 Action eligible

Accepted navigation is action eligible when:

- it is current-state evaluation eligible;
- source health is `HEALTHY`.

Only action-eligible information may contribute to action-eligible persistence.

### 9.4 Projection eligible

Accepted navigation is projection eligible when:

- it is action eligible;
- all projection inputs are finite;
- the configured projection horizon is valid.

The initial policy intentionally makes projection stricter than current-state classification.

---

## 10. Current Envelope Policy

Current-state envelope classification is performed only when current-state evaluation is eligible.

Possible semantic results are:

- `INSIDE`
- `OUTSIDE`
- `NOT_EVALUATED`

`NOT_EVALUATED` means the system lacks information permitted for current-state envelope classification.

It does not mean inside or outside.

### 10.1 Boundary rule

A coordinate exactly equal to a configured minimum or maximum boundary remains inside.

### 10.2 Degraded source health

A fresh accepted `DEGRADED` observation may produce `INSIDE` or `OUTSIDE` classification.

The degraded-health condition shall remain observable alongside that classification.

However, the classification is not action eligible.

---

## 11. Persistence Policy

The monitor maintains an **action-eligible current-envelope violation interval**.

The interval begins only when an action-eligible evaluation first classifies current position as `OUTSIDE`.

### 11.1 Continuation

Once started, the violation interval may remain active while the last accepted navigation state remains action eligible and classified `OUTSIDE`.

Evaluation ticks may advance the elapsed modeled duration of the active interval.

However, evaluation ticks alone shall not confirm a persistent violation for simulated-action purposes.

Elapsed modeled time, not packet count, determines the persistence duration, but persistence confirmation requires newer accepted navigation evidence as defined in Section 12.

### 11.2 Return inside

If an action-eligible evaluation classifies current position as `INSIDE`, the active violation interval ends.

### 11.3 Loss of action eligibility

If action eligibility is lost while a violation interval is active, the active interval ends and shall not continue accumulating.

Examples include:

- navigation becomes stale;
- source health becomes `DEGRADED`;
- no usable accepted state exists.

Loss of action eligibility is a modeled-time fact and does not require a runtime event to occur at the exact instant eligibility is lost.

When a later runtime event is processed, the monitor shall determine whether the previously accepted navigation state remained action eligible continuously through the elapsed modeled interval.

If the previous accepted state would have become stale before the later event, the prior violation interval is considered interrupted at that freshness boundary.

A newly accepted outside observation after such an interruption shall begin a new violation interval and shall not retroactively bridge the stale period.

The interrupted interval remains historical evidence but is not resumed automatically.

### 11.4 Later outside state after interruption

If action eligibility is later restored and current state is `OUTSIDE`, a new violation interval begins at that later evaluation.

This rule applies even when no runtime event occurred at the exact modeled time at which the preceding accepted navigation state became stale.

Elapsed time from the earlier interrupted interval does not carry into the new interval.

### 11.5 Rationale

This policy avoids claiming continuous outside-envelope evidence across a period in which the monitor lacked action-eligible information.

The tradeoff is explicit: data loss can prevent a persistent violation from reaching the action threshold.

That limitation must remain visible in the hazard analysis.

---

## 12. Persistent Violation

Configuration defines a synthetic `violation_persistence_duration`.

A current-state violation is **persistent** for simulated-action purposes when:

- an action-eligible violation interval exists;
- elapsed modeled time since the interval start is greater than or equal to the configured persistence duration;
- a newer accepted navigation observation than the observation that started the interval is action eligible;
- that newer accepted observation is evaluated at or after the persistence-duration boundary;
- that newer accepted observation classifies current position as `OUTSIDE`.

Evaluation ticks may advance interval age but cannot, by themselves, satisfy the confirming-observation condition.

If a confirming accepted observation at or after the persistence boundary classifies the current position as `INSIDE`, the interval ends without becoming persistent.

At exact elapsed-time equality, an otherwise qualifying newer outside observation may confirm persistence.

This boundary rule shall be verified explicitly.

---

## 13. Projection Policy

The initial projection remains constant velocity:

`projected_position = current_position + current_velocity * projection_horizon`

Projection occurs only when projection eligibility is satisfied.

Projected envelope status is:

- `INSIDE`
- `OUTSIDE`
- `NOT_EVALUATED`

### 13.1 Projected violation consequence

A projected `OUTSIDE` result:

- is recorded as decision evidence;
- may produce a stable projected-violation reason code;
- does not create action-eligible persistence;
- does not by itself create a simulated safety-action latch.

### 13.2 Rationale

The initial constant-velocity projection is intentionally simple.

Treating it as an action trigger would give the simplified predictive model authority not justified by the project objective.

---

## 14. Initial Simulated Safety-Action Trigger Policy

For the first system version, the only condition authorized to create a **new** `SIMULATED_SAFETY_ACTION_REQUEST` latch is:

> a persistent action-eligible current-state envelope violation.

Therefore a new action latch requires:

1. an action-eligible accepted navigation observation that begins an `OUTSIDE` violation interval;
2. continuous preservation of action eligibility for that interval;
3. elapsed modeled violation duration greater than or equal to the configured persistence duration;
4. a newer accepted navigation observation evaluated at or after that persistence boundary;
5. fresh navigation for the confirming observation;
6. source health `HEALTHY` for the confirming observation;
7. current-state envelope classification `OUTSIDE` for the confirming observation.

An evaluation tick alone shall not create the new latch.

### 14.1 Conditions that do not independently trigger a new latch

The following do not independently create a new action latch in the initial version:

- navigation stale;
- no accepted navigation state;
- interface timeout;
- invalid observation;
- duplicate observation;
- reordered observation;
- sequence gap;
- source health `DEGRADED`;
- source health `INVALID`;
- projected envelope violation;
- logging failure;
- rejected runtime event.

These conditions remain observable and may affect monitoring eligibility.

### 14.2 Existing latch

Once the simulated action is latched, later loss of navigation, health degradation, return inside the envelope, or interface timeout does not clear it.

Explicit reset within the current execution clears the latch.

Reinitialization begins a new execution and does not carry the prior execution's latch into that new execution.

### 14.3 Explicit reset and post-reset re-latch

Explicit reset is an in-execution latch operation.

It clears the simulated-action latch and does not by itself clear or replace:

- accepted navigation state;
- runtime-event ordering history;
- qualifying interface-activity history;
- an active violation interval.

Reset does not itself establish new navigation evidence and does not itself interrupt an otherwise continuous action-eligible violation interval.

The reset establishes a confirmation boundary for later latch creation.

Navigation observations accepted before reset may contribute historical interval context, but they cannot themselves serve as the post-reset confirming observation that creates a new latch.

A new simulated-action latch after reset requires:

1. a navigation observation accepted after reset;
2. that observation to provide new confirming navigation evidence;
3. that observation to be action eligible;
4. current envelope classification `OUTSIDE`;
5. all otherwise applicable persistent-violation conditions to be satisfied.

An evaluation tick after reset cannot by itself re-latch the simulated action.

If action eligibility or violation continuity is lost after reset, the normal persistence-interruption and interval-restart rules apply.

---

## 15. Recovery Policy Summary

### 15.1 Invalid/rejected observation

A rejected observation requires no special recovery state.

A later acceptable observation may be accepted normally.

Historical rejection evidence remains available.

### 15.2 Sequence gap

A forward sequence gap is observable but does not itself require recovery.

Later strictly increasing observations may continue normally.

### 15.3 Duplicate/reordered observation

The rejected observation does not alter accepted state.

A later observation satisfying ordering rules may be accepted.

### 15.4 Stale navigation

Recovery occurs on the next accepted observation that is fresh at its evaluation time.

### 15.5 Degraded source health

Action eligibility recovers when a later accepted fresh observation reports `HEALTHY`.

No earlier violation-persistence duration is restored.

### 15.6 Invalid source health

A later otherwise acceptable observation with `HEALTHY` or `DEGRADED` source health may be accepted.

### 15.7 Interface timeout

Interface availability recovers on the next qualifying interface activity.

Navigation usability is evaluated independently.

### 15.8 Persistent action latch

The latch does not recover automatically.

Explicit reset clears the latch within the current execution according to Section 14.3.

Reinitialization begins a new execution without carrying the prior execution's latch state.

---

## 16. Rejected Runtime-Event Evidence

A runtime event rejected before normal evaluation shall produce a separate **runtime-event rejection evidence record** rather than a normal runtime `DecisionRecord`.

The semantic record shall contain enough information to determine:

- rejected event category;
- supplied event time;
- last accepted runtime-event time;
- rejection reason;
- execution/configuration identity when available.

The exact serialization and type name are deferred to interface/design work.

### 16.1 Rationale

A normal `DecisionRecord` represents an accepted runtime evaluation.

Using the same record semantics for an event that never entered normal evaluation would blur that distinction.

---

## 17. Simultaneous Conditions

Multiple conditions may coexist.

Examples include:

- interface available while navigation is stale;
- sequence gap while current navigation remains usable;
- degraded source health while current position is outside;
- interface timeout while a prior simulated action is already latched.

The system shall preserve all applicable stable reason identifiers.

### 17.1 No primary reason in the initial version

The initial system shall not require one reason to be designated as the authoritative "primary" reason.

All applicable reasons are recorded.

Their external ordering remains deterministic according to a stable reason-code ordering defined later.

### 17.2 Rationale

Deferring a primary reason avoids inventing a precedence hierarchy that is not required for behavior.

---

## 18. Decision Semantics Under Key Conditions

This section summarizes policy combinations.

### 18.1 Fresh healthy state inside envelope

- current envelope: `INSIDE`
- persistence: inactive
- projection: evaluated when eligible
- new simulated action: no

### 18.2 Fresh healthy state outside but not yet persistent

- current envelope: `OUTSIDE`
- persistence: active, non-persistent
- projection: evaluated when eligible
- new simulated action: no

### 18.3 Fresh healthy state outside reaches persistence duration without newer confirmation

- current envelope: `OUTSIDE`
- persistence interval age: persistence duration reached
- persistence confirmation: pending newer accepted outside observation
- new simulated action: no

### 18.4 Newer fresh healthy outside observation confirms persistence

- current envelope: `OUTSIDE`
- persistence: persistent
- new simulated action: latch `SIMULATED_SAFETY_ACTION_REQUEST`

### 18.5 Fresh degraded state outside

- current envelope: `OUTSIDE`
- action eligibility: false
- persistence: not accumulating
- projection: `NOT_EVALUATED`
- new simulated action: no

### 18.6 Stale accepted state

- current envelope: `NOT_EVALUATED`
- persistence: inactive
- projection: `NOT_EVALUATED`
- new simulated action: no

### 18.7 Interface timeout with stale state

- interface: timed out
- navigation: stale
- current envelope: `NOT_EVALUATED`
- projection: `NOT_EVALUATED`
- new simulated action: no

### 18.8 Projected outside, current inside

- current envelope: `INSIDE`
- projected envelope: `OUTSIDE`
- new simulated action: no

### 18.9 Existing latched action followed by nominal state

- current monitoring facts: evaluated normally when possible
- simulated action: remains latched
- latch-clear operation: none without explicit reset

### 18.10 Explicit reset during an active or persistent violation

- explicit-reset effect: simulated-action latch clears
- accepted navigation state: unchanged solely because of reset
- runtime-event ordering history: unchanged solely because of reset
- interface-activity history: unchanged solely because of reset
- active violation interval: retained unless normal eligibility or continuity rules interrupt it
- evaluation tick after reset: cannot by itself create a new latch
- post-reset latch creation: requires a navigation observation accepted after reset and all otherwise applicable persistent-violation conditions
- interrupted persistence after reset: normal restart rules apply

---

## 19. Configuration Relationships

The initial configuration shall eventually include at least:

- navigation freshness limit;
- interface timeout limit;
- current-envelope persistence duration;
- projection horizon;
- Cartesian envelope bounds.

All configured durations shall be finite and non-negative or positive as required by their semantic meaning.

The exact valid ranges remain to be specified in requirements.

### 19.1 No hidden threshold sourcing

No threshold may be copied from operational flight-safety material.

Synthetic verification configurations shall be chosen for test clarity and boundary coverage.

---

## 20. Failure-Policy Hazards and Tradeoffs

### 20.1 Data loss does not trigger action

This reduces false simulated actions caused solely by missing or invalid data.

It also means that prolonged loss of action-eligible information can prevent a synthetic envelope violation from becoming persistent.

That is an explicit limitation, not an accidental side effect.

### 20.2 Persistence requires confirming navigation evidence

Elapsed modeled time may advance an active violation interval, but a newer accepted action-eligible outside observation is required to confirm persistence.

This prevents one isolated outside observation from becoming action-authoritative merely because evaluation ticks elapsed.

Loss of action eligibility ends the interval entirely.

These choices can delay or prevent a simulated action when confirming navigation evidence is unavailable.

The hazard log shall capture this tradeoff.

### 20.3 Degraded data remains classifiable but not action eligible

This preserves diagnostic value while withholding action authority from degraded source health.

The distinction must be verified explicitly.

### 20.4 Projection is advisory

The project avoids making the intentionally simplified constant-velocity model an action authority.

If future project goals require projection-triggered actions, that would require a separate reviewed change to assumptions, requirements, hazards, and verification.

---

## 21. Requirements to Derive From This Policy

The next requirements update should cover at least:

- qualifying interface activity;
- combined sequence/source-time acceptance;
- rejection of source timestamps later than receive time;
- health-specific acceptance and eligibility;
- freshness boundary;
- timeout boundary;
- current-state `NOT_EVALUATED` semantics;
- action-eligible persistence start/continue/reset;
- persistence interruption when prior navigation becomes stale between runtime events;
- confirming-observation requirement for persistent violation;
- persistent-violation equality boundary;
- advisory-only projection;
- sole initial latch trigger;
- non-triggering fault conditions;
- explicit recovery rules;
- runtime-event rejection evidence;
- no-primary-reason policy.

Stable requirement IDs shall be appended rather than renumbering the existing Phase 1A requirements.

---

## 22. Hazard Log Updates Required

The hazard analysis shall be updated to include or refine at least:

- loss of violation continuity because usable navigation disappears;
- inappropriate simulated action from degraded/unusable information;
- failure to latch after a valid confirmed persistent current-state violation;
- false latch caused by elapsed ticks without confirming navigation evidence;
- failure to reset persistence after evidence continuity is lost;
- improper persistence bridging across an unevaluated stale interval;
- acceptance of future-dated navigation state;
- invalid recovery that restores monitoring too early;
- ambiguity between interface recovery and navigation recovery.

These hazard updates shall occur before implementation.

---

## 23. Critical Review

### 23.1 The policy deliberately favors evidentiary certainty over continuity assumption

When navigation becomes unusable, the system does not extrapolate a current violation interval through the unknown period.

That is defensible for this synthetic verification platform because the project emphasizes explicit evidence.

It would not be appropriate to claim that this policy represents operational range-safety doctrine.

### 23.2 One fresh accepted observation can recover stale navigation

This is simple and testable.

It may later prove too permissive if the project introduces state-discontinuity requirements.

The policy therefore treats this as an initial decision subject to controlled revision.

### 23.3 DEGRADED is useful only because its behavior differs

The degraded source-health value is retained because it allows current-state diagnostic classification while withholding projection, persistence, and new action authority.

If later requirements do not need that distinction, the health model should be simplified instead of retaining ceremonial states.

### 23.4 Interface activity is intentionally broader than usable navigation

A stream of invalid navigation observations can keep the interface "active" while navigation remains unusable.

That is not contradictory.

It represents two different facts and should produce simultaneous reason evidence.

### 23.5 The initial action trigger is intentionally narrow

Only a persistent, healthy, fresh, current-state synthetic envelope violation can create a new simulated action latch.

This keeps the action path inspectable and avoids assigning action authority to the simplified projection model or to absence of evidence.

### 23.6 No primary reason reduces premature policy

Stable multi-reason evidence is sufficient for initial verification.

A primary-reason hierarchy should be added only if a later interface or operator-facing use case actually requires it.

---

## 24. Phase 1B Exit Criteria

This failure-policy increment is acceptable when engineering review agrees that:

- interface activity and navigation usability are distinct;
- both sequence and source-time progression are required for state replacement;
- accepted navigation source time cannot be later than receive time;
- forward sequence gaps remain observable without automatic rejection;
- `HEALTHY`, `DEGRADED`, and `INVALID` have distinct behavior;
- freshness and timeout boundary semantics are explicit;
- stale navigation cannot drive current-state action decisions;
- loss of action eligibility interrupts persistence;
- stale transition between runtime events interrupts persistence even without an event at the exact transition time;
- interrupted persistence does not silently resume;
- elapsed ticks alone cannot confirm persistent violation;
- a newer accepted action-eligible outside observation is required to confirm persistence;
- projection is advisory only;
- the initial simulated-action trigger is explicitly limited to persistent action-eligible current-state violation;
- loss of data does not independently create a new simulated action;
- latch persistence remains explicit;
- recovery semantics are defined per condition;
- rejected runtime events remain separate from normal decision records;
- simultaneous conditions remain observable;
- no operational aerospace thresholds or procedures have entered the design.

After these criteria pass review, the system requirements and hazard log may be updated to encode this policy using new stable identifiers.

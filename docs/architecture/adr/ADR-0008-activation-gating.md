# ADR-0008 — Activation Gating as the Primary False-Activation Control

**Status:** Accepted
**Date:** 2026-11-06
**Decider:** Team Lead
**Consulted:** Member 4
**Supersedes:** —

---

## Context

The most common failure of gesture interfaces is not misclassification. It is **correct classification of an unintended movement**. A user gestures while speaking, reaches for a cup, or adjusts their posture, and the system acts.

Three control strategies exist:

1. Raise the confidence threshold until false activations become rare — which also makes real commands unreliable.
2. Add temporal stability requirements — helps with flicker but not with deliberate-looking unintended motion.
3. Require an explicit activation signal so the system knows the user intends to command it.

## Decision

Adopt **explicit activation gating** as the primary control, with three modes:

| Mode | Arm trigger | Disarm trigger | Status |
|---|---|---|---|
| `wake_gesture` | `open_palm` held for 800 ms | Inactivity timeout (30 s) or explicit stop | **Default** |
| `hold_pose` | Pose held during the command | Pose released | Supported |
| `push_to_talk` | Key held while camera is live | Key released | Supported |
| `always_on` | Session start | Session stop | Research only; labelled `research_unsafe` |

The system starts in `IDLE` and cannot execute anything until armed (`FR-001`).

## Consequences

**Positive**

- False activations during ordinary movement are reduced structurally, not statistically. The system is not listening when the user is not commanding.
- The state machine produces an explicit, visible `ARMED`/`IDLE` indication, which is itself a safety feature.
- Thresholds can be set for usability rather than being pushed to extremes.
- The activation gesture is a natural place to demonstrate intent to an audience.

**Negative**

- Every command sequence costs an extra activation step, which reduces the fluidity that makes gesture interfaces attractive.
- The wake gesture itself can be misdetected, either arming unintentionally or failing to arm.
- Users must learn the model. This is a usability cost that must be measured, not dismissed.

**Neutral**

- The `always_on` mode exists so the research can quantify what gating buys. It is deliberately not the default.

## Alternatives Considered

| Alternative | Rejected because |
|---|---|
| High threshold only | Trades recall for precision; real commands become unreliable |
| Temporal stability only | Does not prevent deliberate-looking but unintended motion |
| Face or gaze activation | Adds a second model and a second privacy surface |
| Continuous always-on with a spotting model | Solves the wrong problem and is the riskiest option for a live demo |
| Hardware button | Works, but abandons the hands-free premise |

## Measurement Commitment

The research design measures false activations **with gating enabled** and, in an ablation, **without it** (`AB-06`). The claim "gating reduces false activations" must be supported by that measurement, not asserted from design intent.

This is recorded as a hypothesis under test, not a settled fact.

## Verification

| Check | Method |
|---|---|
| No execution while `IDLE` | `TC-020`, 100-trial protocol |
| Only the wake pose arms | `AC-003` negative test across all classes |
| False activations per minute | `FR-108` protocol |
| Gating ablation | `AB-06` |
| Visible state indication | UI test |

## Review Trigger

- Measured false activation rate remains above the budget despite gating.
- Usability feedback shows the activation step is a significant burden.
- A reliable intent-spotting model becomes available and is evaluated.

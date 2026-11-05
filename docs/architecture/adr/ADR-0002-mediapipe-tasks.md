# ADR-0002 — MediaPipe Tasks for Hand Landmark Extraction

**Status:** Accepted
**Date:** 2026-11-06
**Decider:** Member 2
**Consulted:** Team Lead, Member 3
**Supersedes:** —

---

## Context

Every downstream result depends on landmark quality. The system must produce 21 hand landmarks per frame at ≥ 24 FPS on integrated graphics, without training a detector.

Three broad options exist: use a pretrained landmark model, train one, or avoid landmarks entirely and classify images.

## Decision

Use **MediaPipe Tasks `HandLandmarker`** as the landmark extractor, wrapped behind the `IF-002` protocol so the implementation is replaceable.

Configuration:

| Parameter | Value |
|---|---|
| Model | Hand landmarker bundle shipped with the pinned MediaPipe version |
| Max hands | 1 |
| Detection confidence | 0.5 (configurable) |
| Tracking confidence | 0.5 (configurable) |
| Input | 640×480 BGR frame converted to RGB |

## Consequences

**Positive**

- No training data or training time required for perception.
- CPU-friendly; runs without a GPU, satisfying the Tier A constraint.
- Stable landmark semantics (wrist, finger joints, tips) across users, which makes normalisation straightforward.
- Widely used, so failures are diagnosable.

**Negative**

- Wheel availability lags CPython releases; the runtime must be pinned to a supported version (`OQ-01`).
- Accuracy degrades with occlusion, extreme distance, and poor lighting. The mitigation is protocol control, not model work.
- The vendor controls the model; there is no way to fine-tune it for this specific domain.
- Exact-version pinning is mandatory because landmark behaviour can shift between releases, which would invalidate comparisons.

**Neutral**

- Vendor lock-in is bounded by the `IF-002` abstraction.

## Alternatives Considered

| Alternative | Rejected because |
|---|---|
| Train a custom keypoint model | A separate research project; needs labelled hand data and tuning time the team does not have |
| OpenPose hand model | Much heavier, GPU-oriented; no advantage at 21 points |
| YOLO-based hand pose | Requires training and a labelled dataset before any gesture work can begin |
| Classical CV (contour + convexity defects) | Fragile under rotation, occlusion, and varying skin tone and lighting |
| OS-provided hand tracking | Platform-specific and unavailable on Windows desktops |

## Verification

| Check | Method |
|---|---|
| 21 landmarks per hand with visibility | `TC-010` |
| Landmark continuity across frames | `QC-08` during capture |
| Per-frame extraction ≤ 12 ms on Tier A | `TC-060` |
| Replaceability | A stub extractor can be substituted in tests without touching recognition |

## Consequences if This Decision Fails

If accuracy proves inadequate at the required distance:

1. Narrow the capture protocol (distance band, framing, lighting).
2. Downscale or crop to the hand region before extraction.
3. Only then consider a different extractor, which requires a new ADR and a re-collection of data.

**Explicitly not a fallback:** silently lowering the distance requirement without documenting it, which would invalidate `NFR-042`.

## Review Trigger

- MediaPipe's supported Python version diverges from the project's runtime.
- Measured extraction latency exceeds the budget on the demo machine.
- Detection rate at 2–3 m falls below the protocol threshold.

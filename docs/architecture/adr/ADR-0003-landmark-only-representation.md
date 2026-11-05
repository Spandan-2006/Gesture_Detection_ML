# ADR-0003 — Landmark-Only Model Input

**Status:** Accepted
**Date:** 2026-11-06
**Decider:** Team Lead
**Consulted:** Member 2, Member 3
**Supersedes:** —

---

## Context

A gesture classifier can consume either raw image pixels, cropped hand images, or extracted landmarks. Each choice changes accuracy potential, data volume, privacy exposure, and generalisation behaviour.

The project's stated priorities are safety, reproducibility, and generalisation to unseen users — not maximum accuracy on a single benchmark.

## Decision

Models consume **only normalised landmarks** (21 × 3 per hand, plus velocities). No pixel data reaches any classifier.

The recorded dataset stores landmarks, not frames. The runtime never persists frames.

## Consequences

**Positive**

- Feature vectors are 126 numbers per frame instead of tens of thousands of pixels; training is fast and works on small datasets.
- Models are largely invariant to skin tone, clothing, and background, which improves cross-user generalisation.
- No images are stored or transmitted, which materially reduces the privacy and consent burden.
- Dataset size is kilobytes per sample, so the full dataset fits comfortably in local storage.
- Replay for testing and CI is trivial and deterministic.

**Negative**

- Appearance cues are discarded, so gestures distinguishable only by hand shape or object interaction cannot be learned.
- MediaPipe's landmark errors become the model's ceiling; there is no way for the classifier to compensate from pixels.
- The approach cannot extend to object manipulation without a new decision.

**Neutral**

- Requires a robust normalisation pipeline; that work is required regardless.

## Alternatives Considered

| Alternative | Rejected because |
|---|---|
| Raw frame CNN | Large data requirement, slower training, weaker cross-user generalisation, and storing frames creates a privacy liability |
| Cropped hand image CNN | Better than full frames but still requires storing images and far more data |
| Hybrid (landmarks + appearance) | Doubles the data requirement and prevents attributing results to the temporal factor |
| Pose estimation on the whole body | Unnecessary for hand gestures and adds a dependency |

## Trade-off Explicitly Accepted

A landmark-only model will likely underperform a well-trained image model on a large dataset. This project does not have a large dataset, does not need to beat state of the art, and does value privacy and generalisation. The trade-off is intentional, not an oversight.

## Verification

| Check | Method |
|---|---|
| No image data reaches models | Code review + `TR-020` payload test |
| No frames persisted at runtime | Security test: file-system assertion during a session |
| Dataset contains only landmarks and metadata | Schema validation (`ENT-03`) |
| Cross-user performance reported honestly | LOSO evaluation (`AC-016`) |

## Review Trigger

- Two-hand or body-pose extension is approved.
- Gesture vocabulary expands to classes that depend on hand shape relative to objects.
- Available data grows beyond what landmark models can exploit.

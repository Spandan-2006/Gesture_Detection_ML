# ADR-0010 — Late Fusion as the v1 Default

**Status:** Accepted
**Date:** 2026-11-06
**Decider:** Team Lead
**Consulted:** Member 3
**Supersedes:** —

---

## Context

Gesture, speech, and application context produce evidence of different kinds, at different times, with different reliability. They must be combined into an intent.

Three fusion strategies are available:

| Strategy | How it works |
|---|---|
| **Early fusion** | Concatenate features from all modalities before classification |
| **Late fusion** | Classify per modality, then combine decision scores |
| **Learned fusion** | Train a model to combine modality outputs |

Each has different data requirements and different explainability properties.

## Decision

Adopt **configurable late fusion with explicit abstention** as the v1 default.

```
score(intent) = Σ_s w_s · conf_s · match_s(intent)   for s ∈ {gesture, speech, context}
normalised by Σ_s w_s over sources present
```

Default weights: gesture 0.55, speech 0.30, context 0.15.

Abstain when:

- best score < `min_fused_confidence` (0.75), or
- margin to runner-up < `min_margin`, or
- only one source present and that source is below its own threshold.

## Consequences

**Positive**

- Weights are configuration, so the three research configurations (A, B, C) are the same binary with different weights. This is the design property that makes the comparison fair.
- Any subset of sources can be missing and the system still produces a result or an explicit abstention.
- Every decision can be explained by citing the contributing sources and their weights, which feeds the dashboard's decision trace.
- No training data is required for the fusion layer, so it can be built before the interaction dataset exists.
- Deterministic and property-testable.

**Negative**

- Cannot learn cross-modal interactions. For example, "a small hand motion plus the word *stop*" might be better learned than weighted.
- Equal-weight assumptions are naive; the default weights are a starting point, not a tuned result.
- Confidence values from different modalities are not naturally comparable. Gesture softmax confidence and speech engine confidence are not on the same scale, so the weighted sum is an approximation.

**Neutral**

- The fusion stage is behind `IF-006`, so a learned replacement does not affect other components.

## Alternatives Considered

| Alternative | Rejected because |
|---|---|
| Early fusion | Requires synchronised, aligned feature vectors across modalities; breaks when speech is absent; makes ablation nearly impossible |
| Learned fusion (stacked) | Needs interaction data at a volume the project will not have; opaque; cannot be built before Phases 7–9 complete |
| Rule-only fusion (if speech says X then X) | Ignores confidence; brittle; not measurable as a weighted contribution |
| Gesture-only with speech as a modifier flag | Discards the quantitative comparison the research question needs |

## Consequences for the Research

**The default weights are not a finding.** They are a starting configuration. The report must state that the weights were fixed before evaluation and were not tuned on test data.

**Ablation is first-class.** `AB-03`, `AB-04`, `AB-05` remove sources by setting weight to zero or disabling the subsystem. Because the code path is identical, the measured difference is attributable to the evidence, not to an implementation branch.

**Known measurement caveat.** Because confidences are not calibrated across modalities, a source with systematically higher confidence will dominate. The calibration of per-source confidences is recorded as a limitation and as a candidate ablation.

## Verification

| Check | Method |
|---|---|
| Deterministic output | `TC-095` property test |
| Order-independent | `TC-095` property test |
| Disabling a source never increases confidence | `TC-095` property test |
| Abstention on weak evidence | `TC-053` |
| Three configurations from config only | `TC-052` |
| Weights recorded in the event | `TC-040` |

## Review Trigger

- Interaction data becomes large enough to train a fusion model and validate it on held-out subjects.
- Ablation shows that one source dominates so strongly that the weighted sum is misleading.
- Per-source calibration is implemented, changing the interpretation of the weights.

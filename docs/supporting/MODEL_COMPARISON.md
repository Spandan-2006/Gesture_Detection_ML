# Model Comparison

**Project:** Gesture-Controlled AI Computer Interface
**Version:** 1.0.0
**Owner:** Member 3 (Models) with Team Lead
**Last Updated:** 2026-11-06

---

## 1. Purpose and Scope

This document compares the five candidate classifier families on **architectural and methodological grounds only**. It contains no benchmark scores. Any numeric result must come from a recorded experiment run referenced by its `run_id` in [BACKEND_SCHEMA](../core/BACKEND_SCHEMA.md) `ENT-06`.

**Prohibited in this document.** Accuracy claims, latency claims, "best model" declarations, or extrapolations from published literature presented as project results.

---

## 2. The Comparison Problem

The research question requires a fair comparison across architectures with very different inductive biases. Fairness here means:

1. **Same data.** One dataset release, one frozen split manifest.
2. **Same features.** Where architectures can share a representation, they do. Where they cannot, the difference is stated as part of the hypothesis, not hidden.
3. **Same evaluation.** Identical metric definitions and aggregation.
4. **Same tuning budget.** A comparable, pre-registered search budget per family.
5. **Same latency measurement.** Measured on the same hardware tier with the same boundaries.

The three input contracts from [TRD](../core/TRD.md) §8.1 exist to make point 2 explicit.

---

## 3. Input Representation

| Contract | Shape | Contents | Used by |
|---|---|---|---|
| `IC-FRAME` | `(126,)` | One frame: 63 normalised positions + 63 velocities | Frame-level baseline (H1 only) |
| `IC-FLAT` | `(3780,)` | 30 frames concatenated | Random Forest, MLP |
| `IC-SEQ` | `(30, 126)` | 30 time steps × 126 features | LSTM, GRU, Transformer |

**Why the split is defensible.**

- Random Forest and MLP are not sequence models. Flattening gives them temporal information without pretending they have sequence semantics. This tests whether *any* temporal window helps, independent of architecture.
- LSTM, GRU, and Transformer receive the same underlying features in a form that encodes order explicitly. The difference between `IC-FLAT` and `IC-SEQ` results isolates the value of *sequential inductive bias* from the value of *temporal context*.
- The frame-level baseline isolates the value of temporal context entirely (H1).

**This is the single most important design decision in the comparison, and it must not be altered mid-study.** If it changes, all prior runs are invalidated for comparison.

---

## 4. Family-by-Family Analysis

### 4.1 Random Forest

| Aspect | Detail |
|---|---|
| Architecture | Ensemble of decision trees; bootstrap aggregation; Gini or entropy splits |
| Input | `IC-FLAT` (3780 features) |
| Temporal capability | **None explicit.** Order is encoded only as feature position; the model can split on "velocity at frame 12" but has no notion of sequence. |
| Training complexity | Low. Seconds to minutes on CPU. Handles 3780 features without difficulty. |
| Inference | Single-sample prediction on the order of a few milliseconds; no GPU; thread-safe across trees. |
| Data requirements | Can fit small datasets; prone to overfitting with high dimensionality and few samples. |
| Generalisation | Sensitive to camera framing and participant-specific feature distributions; random splits would overstate performance. |
| Expected strengths | Strong tabular baseline; robust to irrelevant features; no hyperparameter fragility; interpretable feature importances. |
| Limitations | Cannot learn compositional or long-range motion patterns; memory grows with tree count; prediction cost scales with forest size. |
| Role in the study | **Lower bound on temporal modelling.** If it matches the sequence models, the temporal hypothesis is not supported. |
| Tuning budget | Tree count, max depth, min samples per leaf. |

### 4.2 MLP

| Aspect | Detail |
|---|---|
| Architecture | Fully connected feedforward network with non-linear activations |
| Input | `IC-FLAT` (3780 features) |
| Temporal capability | **None explicit.** Same limitation as Random Forest, with learned non-linear feature combinations instead of axis-aligned splits. |
| Training complexity | Low to moderate. Minutes on CPU with early stopping. |
| Inference | Sub-millisecond to low milliseconds; GPU unnecessary. |
| Data requirements | More data than a Random Forest to avoid overfitting on 3780 inputs; regularisation is essential. |
| Generalisation | Similar to Random Forest; weight initialisation adds variance across seeds. |
| Expected strengths | Learns interactions across frames and joints; fast inference; simple deployment. |
| Limitations | Fixed input size; parameter count on 3780 inputs is large relative to sample count; no weight sharing across time. |
| Role in the study | **Tests whether learned non-linearity alone closes the gap** without sequence structure. |
| Tuning budget | Hidden layers (1–2), width, dropout, weight decay, learning rate. |

### 4.3 LSTM

| Aspect | Detail |
|---|---|
| Architecture | Recurrent network with input, forget, and output gates plus cell state |
| Input | `IC-SEQ` (30 × 126) |
| Temporal capability | **Explicit.** Gating allows selective retention of motion phases across the window. |
| Training complexity | Moderate. Sequential computation limits parallelism; CPU training is slow relative to Transformers of similar size. |
| Inference | Sequential dependency means latency scales with window length; single-sample CPU inference is typically the slowest of the five. |
| Data requirements | Moderate to high; sensitive to small datasets without regularisation. |
| Generalisation | Weight sharing across time improves invariance to speed and phase; still sensitive to participant-specific kinematics. |
| Expected strengths | Models order and variable-speed motion; gate structure suits "hold then move" gestures. |
| Limitations | Vanishing gradients on long sequences (not a major concern at 30 steps); harder to parallelise; more hyperparameters. |
| Role in the study | **Primary temporal candidate** and the reference implementation for the class. |
| Tuning budget | Hidden size, layers, dropout, bidirectional on/off, learning rate. |

### 4.4 GRU

| Aspect | Detail |
|---|---|
| Architecture | Recurrent network with update and reset gates; no separate cell state |
| Input | `IC-SEQ` (30 × 126) |
| Temporal capability | **Explicit**, comparable in principle to LSTM with fewer parameters |
| Training complexity | Moderate; faster per epoch than an equivalently sized LSTM |
| Inference | Same sequential constraint as LSTM; fewer parameters often means slightly lower latency |
| Data requirements | Slightly lower than LSTM due to reduced parameter count |
| Generalisation | Often similar to LSTM; differences are dataset-dependent and must be measured |
| Expected strengths | Efficient temporal baseline; good accuracy-to-parameter ratio |
| Limitations | Less expressive gating; the same sequential inference constraint |
| Role in the study | **Efficiency reference.** Distinguishes "temporal modelling helps" from "LSTM specifically helps". |
| Tuning budget | Hidden size, layers, dropout, learning rate |

### 4.5 Transformer Encoder

| Aspect | Detail |
|---|---|
| Architecture | Multi-head self-attention over the sequence, plus position encoding and feedforward blocks |
| Input | `IC-SEQ` (30 × 126) |
| Temporal capability | **Explicit and global.** Every frame can attend to every other frame in one layer. |
| Training complexity | Moderate; parallel across time, so faster per epoch than recurrent models at this length |
| Inference | Single-sample inference includes full attention over 30 positions; typically higher cost than GRU at equal width |
| Data requirements | **Highest of the five.** Attention has little inductive bias; small datasets tend to underfit or overfit unpredictably |
| Generalisation | Positional encoding preserves order; no built-in locality or recurrence prior, which can hurt with few subjects |
| Expected strengths | Captures non-local relationships, such as "start pose → direction → release"; parallel training |
| Limitations | Data-hungry; attention maps are not automatically explanations; positional encoding choice affects results |
| Role in the study | **Upper-capacity candidate.** Tests whether attention capacity is warranted at this data scale. |
| Tuning budget | Layers (1–2), heads, model width, dropout, learning rate, warmup |

**Deliberate constraint.** The Transformer is kept small (≤ 2 encoder layers, ≤ 4 heads) because the dataset is small. A large Transformer would make the comparison uninformative: it would fail for data reasons rather than architectural ones, and that failure would be misread as a finding.

---

## 5. Cross-Family Comparison

| Dimension | Random Forest | MLP | LSTM | GRU | Transformer |
|---|---|---|---|---|---|
| Temporal modelling | None explicit | None explicit | Explicit | Explicit | Explicit, global |
| Input contract | `IC-FLAT` | `IC-FLAT` | `IC-SEQ` | `IC-SEQ` | `IC-SEQ` |
| Parameters | N/A (trees) | High | Moderate | Lower | Moderate–high |
| Training speed (CPU) | Fast | Fast | Slow | Medium | Medium |
| Inference speed (CPU) | Fastest | Fast | Slowest | Slow | Medium |
| Data appetite | Low | Medium | Medium | Medium | High |
| Overfitting risk | Medium | High | Medium | Medium | High |
| Speed invariance | None | None | Learned | Learned | Learned |
| Interpretability | Feature importance | Low | Low | Low | Attention (weak) |
| Deployment complexity | Low | Low | Medium | Medium | Medium |
| Failure mode | Ignores motion order | Memorises training subjects | Slow inference | Underfits | Overfits small data |

**No winner is declared.** The comparison exists to produce evidence, and the evidence may show that a classical model is sufficient at this data scale. That would be a legitimate and reportable outcome.

---

## 6. Evaluation Methodology

### 6.1 Splitting

| Protocol | Purpose | Role |
|---|---|---|
| **Leave-one-subject-out (LOSO)** | Measures generalisation to unseen users | **Primary** — all headline claims |
| Within-subject, session-disjoint | Upper bound under known-user conditions | Secondary |
| Cross-session | Robustness to re-recording on a different day | Secondary |
| Random window split | **Prohibited for reporting** | Diagnostic only, and must be labelled as optimistic |

**Leakage rule.** Splits are made at the *sample* or *session* level, never at the window level. Overlapping windows derived from one recorded sample must never appear on both sides of a split. This is the most common way gesture-recognition results are inflated, and it is explicitly forbidden.

### 6.2 Metrics

| Metric | Definition | Why |
|---|---|---|
| Accuracy | Correct / total | Intuitive, but misleading under class imbalance |
| Macro precision | Unweighted mean of per-class precision | Treats every gesture as equally important |
| Macro recall | Unweighted mean of per-class recall | Surfaces ignored classes |
| Macro F1 | Harmonic mean of the above | **Primary accuracy metric** |
| Per-class precision / recall / F1 | Per class | Identifies which gestures confuse |
| Confusion matrix | Row = true, column = predicted | Diagnoses systematic errors |
| Inference latency p50 / p95 | Model input ready → scores produced | Responsiveness |
| End-to-end latency p50 / p95 | Gesture completion → action result | Usability |
| False activations per minute | Unintended executions in a non-command protocol | The metric users actually feel |
| Abstention rate | Fraction of interactions with insufficient evidence | Calibration of trust |
| Action success rate | Executed actions that achieved the intended effect | End-to-end correctness |

### 6.3 Reporting Rules

1. Every number cites a `run_id`.
2. Report mean and dispersion across LOSO folds, not a single aggregate.
3. Report per-subject results, not only the mean. A high mean with one catastrophic subject is a different finding from a uniformly moderate one.
4. Never compare runs with different dataset versions or split manifests.
5. State the hardware tier for every latency number.
6. Report negative and null results explicitly.

### 6.4 Hyperparameter Search

| Rule | Detail |
|---|---|
| Budget parity | Each family gets the same number of configurations, pre-registered before running |
| Search set | Validation folds only; the test fold is untouched until final evaluation |
| Selection metric | Macro F1 on validation |
| Recording | Every configuration and its validation score is logged, including failures |

**Anti-pattern explicitly rejected.** Tuning on the test set and reporting the best result. If it happens accidentally, the run is marked invalid and re-run.

---

## 7. Threats to Validity

| Threat | Effect | Mitigation |
|---|---|---|
| Small participant pool | LOSO variance is high; one outlier can dominate the mean | Report per-subject results; state N; avoid over-generalising |
| Single camera model | Results may not transfer across devices | Record device metadata; state as a limitation |
| Controlled capture conditions | Real-world robustness unknown | Report the protocol; do not claim field performance |
| Overlapping windows | Inflated accuracy | Sample-level splitting enforced by validator |
| Class imbalance | Accuracy misleading | Macro metrics as primary |
| Hyperparameter budget differences | Unfair comparison | Pre-registered equal budget |
| Hardware variation between members | Latency numbers incomparable | One designated benchmark machine; tier recorded |
| Model selection on test data | Optimistic bias | Frozen test folds; selection on validation only |

---

## 8. Decision Criteria for the Interactive Model

The model used in the live system is not necessarily the most accurate one. Selection weighs:

| Criterion | Weight rationale |
|---|---|
| Macro F1 under LOSO | Primary but not sole |
| Inference latency p95 | Determines perceived responsiveness |
| Model size | Deployment simplicity |
| Robustness across subjects | Avoids a demo that works for one person |
| Abstention behaviour | A model that is confidently wrong is worse than one that abstains |

**Pre-registered rule.** If two families are within a small, pre-declared margin on macro F1, the one with lower p95 inference latency is selected. The margin is declared before evaluation and recorded in [EXPERIMENT_DESIGN](../research/EXPERIMENT_DESIGN.md).

---

## 9. Relationship to Hypotheses

| Hypothesis | Comparison that tests it |
|---|---|
| `H1` Temporal information improves recognition | Frame-level baseline (`IC-FRAME`) vs any window model (`IC-FLAT` or `IC-SEQ`) |
| `H2` Speech improves intent recognition | `gesture_only` vs `gesture_speech` |
| `H3` Context improves disambiguation | `gesture_speech` vs `gesture_speech_context` |
| `H4` Multimodal gains without unacceptable latency | Accuracy gain alongside measured end-to-end latency |
| `H5` User-independent evaluation reveals differences | LOSO vs within-subject comparison per family |

Full design: [RESEARCH_METHODOLOGY](./RESEARCH_METHODOLOGY.md) and [EXPERIMENT_DESIGN](../research/EXPERIMENT_DESIGN.md).

---

## 10. Related Documents

- [TRD](../core/TRD.md) §8 — recognition architecture and contracts
- [DATASET_SPECIFICATION](./DATASET_SPECIFICATION.md) — input data definition
- [EVALUATION_METRICS](../research/EVALUATION_METRICS.md) — metric definitions
- [RESEARCH_METHODOLOGY](./RESEARCH_METHODOLOGY.md) — design and controls

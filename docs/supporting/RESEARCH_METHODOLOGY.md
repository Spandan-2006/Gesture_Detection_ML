# Research Methodology

**Project:** Gesture-Controlled AI Computer Interface
**Version:** 1.0.0
**Owner:** Team Lead, with Member 3
**Last Updated:** 2026-11-06

---

## 1. Purpose

This document defines how the project will produce evidence for its research question, how that evidence will be analysed, and what would count as a refutation. It is written **before** data collection so that the analysis is not shaped around the results.

**Status of claims.** Nothing in this document is a finding. All numeric expectations are hypotheses or design targets.

---

## 2. Research Question

> **Does adding temporal information, speech, and application context improve human-computer intent recognition compared with gesture-only interaction?**

Three sub-questions make it testable:

| ID | Sub-question | Factor isolated |
|---|---|---|
| `RQ-1` | Does a temporal model outperform frame-level gesture classification? | Temporal information |
| `RQ-2` | Does adding speech to gesture improve intent recognition? | Speech |
| `RQ-3` | Does adding application context to gesture+speech improve disambiguation? | Context |

A fourth, cross-cutting question:

| ID | Sub-question |
|---|---|
| `RQ-4` | Do these gains survive a latency budget and generalise to users not present in training? |

---

## 3. Hypotheses

Each hypothesis is falsifiable and mapped to a specific comparison.

| ID | Hypothesis | Comparison | Primary metric | Falsified if |
|---|---|---|---|---|
| `H1` | Temporal models improve dynamic gesture recognition compared with frame-level classification | `IC-FRAME` baseline vs `IC-SEQ` models, same split | Macro F1 | No improvement beyond the pre-declared margin, or improvement disappears under LOSO |
| `H2` | Combining speech and gesture improves intent recognition compared with gesture alone | `gesture_only` vs `gesture_speech` | Intent accuracy, macro F1 | No improvement, or improvement only in conditions where speech is artificially clean |
| `H3` | Application context improves intent disambiguation | `gesture_speech` vs `gesture_speech_context` | Intent accuracy on ambiguous subset | No improvement on the ambiguous subset |
| `H4` | Multimodal interaction improves intent accuracy without unacceptable latency | Best multimodal vs gesture-only, with measured latency | Accuracy and end-to-end p95 | Accuracy gain accompanied by latency above the stated usability ceiling |
| `H5` | User-independent evaluation reveals meaningful differences in generalisation across architectures | LOSO vs within-subject, per family | Δ macro F1 and per-subject variance | Ranking is identical and variance is negligible |

**Directional expectation.** The project expects H1–H4 to hold. This is stated openly so that a negative result is recognised as informative rather than treated as a failure to be re-analysed.

---

## 4. Variables

### 4.1 Independent Variables

| Variable | Levels | Type |
|---|---|---|
| Temporal representation | Frame-level (`IC-FRAME`), flattened window (`IC-FLAT`), sequence (`IC-SEQ`) | Categorical |
| Model family | Random Forest, MLP, LSTM, GRU, Transformer Encoder | Categorical |
| Evidence configuration | `gesture_only`, `gesture_speech`, `gesture_speech_context` | Categorical |
| Temporal context length | 10, 20, 30, 45 frames | Ordinal (ablation) |
| Evaluation protocol | Within-subject, leave-one-subject-out | Categorical |

### 4.2 Dependent Variables

| Variable | Definition | Unit |
|---|---|---|
| Gesture accuracy | Correct classifications / total | ratio |
| Macro F1 | Unweighted mean of per-class F1 | ratio |
| Per-class precision / recall / F1 | Standard definitions | ratio |
| Intent accuracy | Correct intents / total interactions | ratio |
| Abstention rate | Abstentions / total interactions | ratio |
| Inference latency p50 / p95 | Model input ready → scores produced | ms |
| End-to-end latency p50 / p95 | Gesture completion → action result | ms |
| False activations per minute | Unintended executions in a non-command protocol | count/min |
| Action success rate | Executed actions achieving the intended effect | ratio |
| Cross-subject variance | Standard deviation of per-fold macro F1 | ratio |

### 4.3 Control Variables

| Variable | Control method |
|---|---|
| Camera hardware | One designated capture device; model recorded |
| Capture resolution and FPS | Fixed at 640×480, 30 FPS requested |
| Distance | Fixed band per protocol block; recorded per sample |
| Lighting | Recorded band; primary protocol uses "normal" |
| Participant instruction | Scripted, identical across participants |
| Randomised class order | Per participant, to remove order effects |
| Random seeds | Fixed per run and recorded |
| Hyperparameter budget | Equal configuration count per family |
| Evaluation code | One harness for all families |
| Hardware for latency | One designated benchmark machine, tier recorded |
| Preprocessing version | Frozen per dataset release |

### 4.4 Uncontrolled Variables (declared, not hidden)

| Variable | Why it cannot be fully controlled | Handling |
|---|---|---|
| Participant anatomy and style | Individual differences are the point of LOSO | Reported as variance, not "noise" |
| Fatigue across a session | 60-minute protocol | Practice block, rest, randomised order |
| Ambient conditions | No lab-grade control | Recorded per sample; reported as a limitation |
| Software environment | Updates during the study | Pinned versions; re-run affected experiments |

---

## 5. Experimental Design

### 5.1 Three Interaction Configurations

| Config ID | Evidence available | Weights (default) |
|---|---|---|
| `A` `gesture_only` | Gesture | gesture 1.0 |
| `B` `gesture_speech` | Gesture + speech | gesture 0.55, speech 0.45 (renormalised) |
| `C` `gesture_speech_context` | Gesture + speech + context | gesture 0.55, speech 0.30, context 0.15 |

**Critical design property.** These are the *same binary* with different configuration. No branch of code exists per configuration, so the comparison measures the contribution of evidence, not the contribution of implementation effort.

### 5.2 Design Matrix

| | Random Forest | MLP | LSTM | GRU | Transformer | Frame baseline |
|---|---|---|---|---|---|---|
| Gesture-only | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Gesture+speech | ✓ | ✓ | ✓ | ✓ | ✓ | — |
| Gesture+speech+context | ✓ | ✓ | ✓ | ✓ | ✓ | — |

18 gesture-level cells plus ablations. Every cell is evaluated under LOSO and within-subject.

### 5.3 Conditions and Order

| Aspect | Decision |
|---|---|
| Order of configurations | Randomised per participant for the *interactive* evaluation; fixed (A→B→C) for offline evaluation |
| Within-participant repetitions | 20 per class for offline; 3 scripted task sequences per configuration for interactive |
| Between-participant design | LOSO folds; every participant acts as a test subject exactly once |
| Counterbalancing | Configuration order randomised to control learning effects in the interactive study |

### 5.4 Offline vs Interactive Evaluation

| | Offline | Interactive |
|---|---|---|
| Input | Recorded landmark sequences | Live camera, microphone, foreground app |
| Measures | Accuracy, F1, confusion, inference latency | End-to-end latency, false activations, action success, usability |
| Purpose | Answers `RQ-1`, `RQ-2` (model side) | Answers `RQ-3`, `RQ-4` (system side) |
| Reproducibility | Full | Recorded traces, not bit-identical |

Both are required. Offline-only results cannot support a claim about interaction; interactive-only results cannot isolate architecture.

---

## 6. Procedure

### 6.1 Phase A — Offline Model Study

1. Freeze `ds-vX` and `split-loso-v1`.
2. Train every family on every fold with the pre-registered search budget.
3. Select hyperparameters on the validation participant of each fold.
4. Evaluate each selected model once on the held-out test participant.
5. Repeat for the within-subject protocol.
6. Run the window-length ablation.
7. Export per-fold and aggregate metrics.

### 6.2 Phase B — Interaction Study

1. Recruit participants (may overlap with Phase A participants; recorded).
2. Brief and consent.
3. For each configuration A, B, C in randomised order:
   - Complete a scripted task list (e.g. 10 slide advances, 5 reversals, 5 black-screen toggles).
   - Complete the non-command protocol: 10 minutes of ordinary movement and talking with no intent to command.
4. Record every interaction, denial, and latency sample.
5. Debrief.

### 6.3 Phase C — Analysis

1. Compute metrics per participant, per configuration, per model.
2. Aggregate across LOSO folds with dispersion.
3. Test hypotheses with the pre-declared statistical procedure.
4. Run the ablations.
5. Produce the results tables and figures.
6. Write the report, explicitly separating measured results from proposed targets.

---

## 7. Statistical Analysis Plan

### 7.1 Unit of Analysis

| Question | Unit |
|---|---|
| Model performance | Sample-level predictions, aggregated to fold |
| Generalisation | Participant (LOSO fold) |
| Latency | Interaction |
| False activations | Participant-session |

**No pseudo-replication.** Windows derived from the same sample are never treated as independent observations in a significance test. Where a test is run over predictions, it is run over per-participant aggregates, not raw windows.

### 7.2 Tests

| Comparison | Test | Rationale |
|---|---|---|
| Two configurations, same folds | Wilcoxon signed-rank | Paired, non-parametric, small N |
| Multiple families across folds | Friedman test, with post-hoc pairwise Wilcoxon and Holm correction | Non-parametric repeated measures |
| Effect size | Rank-biserial correlation, plus median difference with bootstrap CI | Significance alone is uninformative at small N |
| Latency distributions | Bootstrap percentile CIs for p50/p95 | No normality assumption |

**Sample size caveat, stated in advance.** With 8–12 participants, statistical power is limited. Results are reported as effect sizes with confidence intervals, and the report will state that the study is exploratory rather than confirmatory where power is insufficient. A non-significant result will not be described as evidence of no effect.

### 7.3 Multiple Comparisons

The design has 18 primary cells. Family-wise error is controlled with Holm–Bonferroni across the primary hypothesis tests. Ablations are explicitly labelled exploratory and are not corrected, but are reported with effect sizes.

### 7.4 Pre-Registration

The following are frozen before Phase A begins and recorded in [EXPERIMENT_DESIGN](../research/EXPERIMENT_DESIGN.md):

- Hypotheses and their direction.
- Primary metrics and the "meaningful improvement" margin.
- Splitting protocol.
- Hyperparameter search budget.
- Statistical tests.
- Exclusion rules.

**Change rule.** A post-hoc change is permitted but must be recorded with the reason and reported alongside the original plan, so that the reader can judge.

---

## 8. Ablation Studies

| ID | Ablation | Question answered |
|---|---|---|
| `AB-01` | Frame-level vs flattened vs sequence | Does temporal structure matter, or only temporal context? |
| `AB-02` | Window length 10 / 20 / 30 / 45 frames | How much context is useful before latency cost dominates? |
| `AB-03` | Remove speech from `gesture_speech` | Isolates speech contribution |
| `AB-04` | Remove context from `gesture_speech_context` | Isolates context contribution |
| `AB-05` | Remove both, keep gesture | Replicates the `gesture_only` baseline under identical conditions |
| `AB-06` | Stability window 1 / 2 / 3 / 5 | Trade-off between false activations and responsiveness |
| `AB-07` | Confidence threshold sweep | False activation vs coverage curve |
| `AB-08` | Temporal context length in fusion | Whether longer evidence windows help disambiguation |

**Rule.** Ablations are run on the same frozen splits as the primary study. An ablation run on different data is not an ablation.

---

## 9. Validity Threats and Mitigations

| Threat type | Specific threat | Mitigation | Residual risk |
|---|---|---|---|
| Internal | Overlapping-window leakage | Sample-level splitting, manifest validator | Low |
| Internal | Hyperparameter tuning on test data | Frozen test folds; selection on validation | Low |
| Internal | Implementation differences between configurations | Same binary, configuration-driven | Low |
| Internal | Order/learning effects | Randomised configuration order | Medium |
| Construct | Macro F1 may not reflect user-perceived quality | Report false activations and latency alongside | Medium |
| Construct | "Intent recognition" measured by scripted tasks may not reflect natural use | Document scripted nature; report as limitation | Medium–High |
| External | Small participant pool | Report per-subject results; state N | High |
| External | Single camera and room | Record conditions; state as limitation | High |
| External | One language, one vocabulary | State scope explicitly | Medium |
| Conclusion | Small N limits statistical power | Effect sizes, CIs, exploratory framing | High |
| Conclusion | Multiple comparisons | Holm–Bonferroni on primary tests | Low |

**Honesty requirement.** The report must contain a threats-to-validity section that names these risks without softening them. A demonstration-quality system with weak external validity is still a valid finding, provided the scope is stated.

---

## 10. Ethics

| Principle | Implementation |
|---|---|
| Informed consent | Written/digital, versioned, revocable |
| Minimisation | Landmarks only; no video or audio retained |
| Right to withdraw | Honoured at any time; data deleted |
| No deception | Participants are told exactly what is measured |
| No vulnerable populations | Minors excluded absent institutional process |
| Data security | Local storage, access-controlled, no cloud |
| Publication | Aggregates and pseudonymous examples only |
| Dual use | The system performs OS actions; destructive actions require confirmation and are excluded from default mappings |

Detailed policy: [SECURITY_AND_PRIVACY](./SECURITY_AND_PRIVACY.md).

---

## 11. Reporting Structure

The final report follows this order, which mirrors the analysis plan:

1. Research question and hypotheses.
2. System description (what was built, not how good it is).
3. Dataset: participants, protocol, class distribution, limitations.
4. Model comparison results (per family, per protocol).
5. Interaction configuration results (A, B, C).
6. Ablations.
7. Latency analysis.
8. False activation analysis.
9. Threats to validity.
10. Discussion and conclusion.

**Separator rule.** Measured results and proposed targets appear in separate tables with different headings. No table mixes them.

---

## 12. What Would Falsify the Project's Premise

Stated in advance, to keep the analysis honest:

| Observation | Interpretation |
|---|---|
| Temporal models do not beat the frame baseline under LOSO | H1 unsupported; temporal modelling adds cost without benefit at this data scale |
| Speech adds nothing once gesture confidence is high | H2 unsupported; speech may only help in ambiguous cases, which should then be reported as a conditional effect |
| Context changes behaviour but not accuracy | H3 unsupported as an accuracy claim; context may still improve safety and reduce false activations |
| Multimodal accuracy improves but latency becomes unusable | H4 unsupported; the trade-off is the finding |
| All families rank identically under LOSO and within-subject | H5 unsupported; architecture choice may be irrelevant at this scale |

Each of these is a publishable, defensible outcome. None is a project failure.

---

## 13. Related Documents

- [EXPERIMENT_DESIGN](../research/EXPERIMENT_DESIGN.md) — concrete matrix and run plan
- [EVALUATION_METRICS](../research/EVALUATION_METRICS.md) — metric definitions and instrumentation points
- [DATASET_SPECIFICATION](./DATASET_SPECIFICATION.md) — data and splitting rules
- [MODEL_COMPARISON](./MODEL_COMPARISON.md) — architecture rationale
- [PRD](../core/PRD.md) §19 — success metrics

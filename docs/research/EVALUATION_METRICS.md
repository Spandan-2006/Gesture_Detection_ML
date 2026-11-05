# Evaluation Metrics

**Project:** Gesture-Controlled AI Computer Interface
**Version:** 1.0.0
**Owner:** Member 3 with Team Lead
**Last Updated:** 2026-11-06

---

## 1. Purpose

Precise definitions for every metric the project reports. Ambiguity in a metric definition is a defect: two runs that use different definitions cannot be compared, and a reported number without a definition cannot be reproduced.

---

## 2. Rules

| Rule | Detail |
|---|---|
| Every metric has a formula or an instrumentation point | No informal definitions |
| Every reported value cites a `run_id` | Auditability |
| Latency always names its definition | Four distinct latencies exist |
| Accuracy is never reported alone | Macro F1 accompanies it |
| Aggregates always carry dispersion | Mean without variance hides a bad fold |

---

## 3. Classification Metrics

### 3.1 Accuracy

```
accuracy = correct_predictions / total_predictions
```

**Caveat.** Misleading under class imbalance. Reported for familiarity, never as the sole classification metric.

### 3.2 Per-Class Precision, Recall, F1

For class `c`:

```
precision(c) = TP(c) / (TP(c) + FP(c))
recall(c)    = TP(c) / (TP(c) + FN(c))
f1(c)        = 2 · precision(c) · recall(c) / (precision(c) + recall(c))
```

**Edge case.** A class with zero predictions has undefined precision. It is reported as `0.0` with a note, not omitted.

### 3.3 Macro-F1

```
macro_f1 = (1 / |C|) · Σ_c f1(c)
```

**Primary classification metric.** Treats every gesture as equally important, which matches the product need: a rarely used gesture that never works is a broken feature.

### 3.4 Weighted F1

```
weighted_f1 = Σ_c (n_c / N) · f1(c)
```

Reported for comparison with literature conventions. Not used for decisions, because it lets a dominant class mask a broken one.

### 3.5 Confusion Matrix

| Property | Value |
|---|---|
| Orientation | Row = true class, column = predicted class |
| Values | Counts (normalised version may accompany, clearly labelled) |
| Storage | `experiment_results.confusion_matrix_json` |
| Reporting | Always shown for the selected model; available for all |

### 3.6 Abstention Rate

```
abstention_rate = interactions_marked_abstained / total_interactions
```

A model that abstains constantly is safe and useless. This metric makes that trade-off visible.

---

## 4. Intent Metrics

### 4.1 Intent Accuracy

```
intent_accuracy = correct_intents / total_commanded_interactions
```

**Exclusion rule.** Non-commanded interactions (false activations) are not in the denominator. They are reported separately.

### 4.2 Disambiguation Accuracy

Measured only on the **ambiguous subset**: interactions where the gesture alone maps to more than one candidate intent in the active application's vocabulary.

```
disambiguation_accuracy = correct_intents / ambiguous_interactions
```

This is the metric that most directly tests H3. Reporting overall intent accuracy alone would dilute the context effect.

### 4.3 Evidence Contribution

For each configuration, the marginal contribution over the previous:

```
ΔB = accuracy(B) − accuracy(A)
ΔC = accuracy(C) − accuracy(B)
```

Reported with a paired confidence interval across folds. Never reported as a single number without dispersion.

---

## 5. Latency Metrics

Four distinct definitions. **Never mixed.**

| Metric | Start | End | Excludes |
|---|---|---|---|
| **Inference latency** | Input tensor ready | Score vector produced | Capture, window fill, fusion, dispatch |
| **End-to-end interaction latency** | Gesture completion (window close) | `ActionResult` received | Window fill before completion |
| **Time to commit** | Gesture onset | `ActionResult` | Nothing |
| **Speech latency** | Utterance end (VAD) | `SpeechObservation` emitted | Gating delay |

### 5.1 Percentiles

```
p50 = median
p95 = 95th percentile (nearest-rank or linear interpolation, stated)
```

**Sample size rule.** p95 requires at least 100 samples. Below that, report the maximum and label it `max (insufficient n for p95)`.

### 5.2 Aggregation

| Level | Aggregation |
|---|---|
| Per interaction | Single measurement |
| Per session | Percentiles over interactions |
| Per participant | Percentiles over interactions |
| Across participants | Median of per-participant percentiles, plus range |

**Never** compute a global percentile by pooling raw samples across participants; participant-level structure would be lost.

### 5.3 Warm-up

The first 50 inferences after model load are excluded and the exclusion is documented. Results that include cold-start are labelled as such.

---

## 6. Reliability Metrics

### 6.1 False Activations

```
false_activations_per_minute = dispatched_unintended_actions / protocol_minutes
```

| Variant | Definition | Use |
|---|---|---|
| **Dispatched** | An action reached the application | Headline metric |
| **Proposed** | The gate approved but dry-run prevented dispatch | Diagnostic |
| **Attempted** | Evidence exceeded threshold but gate denied | Diagnostic only |

Only the dispatched variant is the headline number. The others explain *why*, and must always be labelled.

### 6.2 Action Success Rate

```
action_success_rate = successful_dispatches / attempted_dispatches
```

Success is determined by observing the application state before and after, not by the adapter's self-report. An adapter that returns `success` while changing nothing is a failure.

### 6.3 System Uptime

```
uptime = (session_duration − unavailable_duration) / session_duration
```

Measured over a 60-minute soak, with unavailability defined as any period where status is `ERROR`.

---

## 7. Generalisation Metrics

| Metric | Definition |
|---|---|
| LOSO macro F1 | Mean of per-fold macro F1 when each subject is held out once |
| Cross-subject variance | Standard deviation of per-fold macro F1 |
| Worst-fold F1 | Minimum per-fold macro F1 — surfaces a model that fails for one person |
| Generalisation gap | Within-subject F1 − LOSO F1 |

**Reporting rule.** A mean of 0.85 with one fold at 0.40 is a different result from 0.85 with all folds at 0.83–0.87. Report the distribution, not only the mean.

---

## 8. Instrumentation Points

| Metric | Source | Field |
|---|---|---|
| Inference latency | Model wrapper timer | `EV-001.inference_latency_ms` |
| End-to-end latency | Trace start and `action.executed` | `interaction_logs.end_to_end_latency_ms` |
| Speech latency | VAD end to observation | `EV-002.transcription_latency_ms` |
| Fusion delay | First evidence to intent | Derived from timestamps |
| Gate duration | Gate timer | `EV-009` metadata |
| False activation | Corrected human review | `interaction_logs.decision` + review log |

**Rule.** Timestamps come from one monotonic clock for intervals, and UTC for wall-clock correlation. Mixing them produces negative latencies, which have been observed in more than one student project.

---

## 9. Statistical Reporting

| Element | Requirement |
|---|---|
| Central tendency | Median, not mean, for latency |
| Dispersion | IQR for latency; SD or range for F1 |
| Confidence interval | Bootstrap percentile, 10 000 resamples, reported with the estimate |
| Paired comparison | Wilcoxon signed-rank with effect size |
| Multiple comparisons | Holm–Bonferroni across primary tests |
| Null results | Reported explicitly, with the effect size and interval |

### 9.1 Effect Size

Rank-biserial correlation for Wilcoxon, plus the median difference in the underlying metric. A p-value without an effect size is not reported.

---

## 10. Reporting Format

Every results table follows this shape.

| Cell | Metric | Median | IQR | n | Run |
|---|---|---|---|---|---|
| `LSTM-C-LOSO` | Macro F1 | — | — | — | — |

**Column rules.**

- `n` is the number of folds or interactions, never omitted.
- `Run` is the `run_id` or a semicolon-separated list.
- Missing values are `—`, never `0`.
- Units are in the column header.

### 10.1 Prohibited Reporting

| Practice | Why prohibited |
|---|---|
| Accuracy without macro F1 | Hides class imbalance |
| Single latency number without definition | Incomparable |
| Best fold reported as the result | Cherry-picking |
| Percentage without denominator | Uninterpretable |
| "Significant" without a test and effect size | Unsupported claim |
| Target values presented in a results table | Fabrication |

---

## 11. Metric-to-Hypothesis Map

| Hypothesis | Primary metric | Secondary |
|---|---|---|
| `H1` | Macro F1: `FRAME` vs `SEQ` | Confusion matrix, per-class F1 |
| `H2` | Intent accuracy: A vs B | Disambiguation accuracy, abstention rate |
| `H3` | Disambiguation accuracy: B vs C | Abstention rate, false activations |
| `H4` | Accuracy gain vs end-to-end p95 | Action success rate |
| `H5` | LOSO vs within-subject macro F1 | Cross-subject variance, worst fold |

---

## 12. Related Documents

- [EXPERIMENT_DESIGN](./EXPERIMENT_DESIGN.md) — what is run
- [RESEARCH_METHODOLOGY](../supporting/RESEARCH_METHODOLOGY.md) — why
- [TRD](../core/TRD.md) §20 — latency definitions
- [BACKEND_SCHEMA](../core/BACKEND_SCHEMA.md) `ENT-07`, `ENT-12` — storage

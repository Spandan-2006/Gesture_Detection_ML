# Experiment Log Template

**Copy this file to `docs/reports/` for each experiment campaign.** One file per campaign, not per run. Individual runs live in the database and in `experiments/results/`.

---

## Campaign Header

| Field | Value |
|---|---|
| Campaign ID | `camp-YYYYMMDD-NN` |
| Title | |
| Owner | |
| Start date | |
| End date | |
| Status | `planned` / `running` / `complete` / `abandoned` |
| Related hypothesis | `H1` / `H2` / `H3` / `H4` / `H5` |
| Dataset version | `ds-` |
| Split manifest | `split-` |
| Code revision at start | |
| Code revision at end | |

---

## 1. Objective

One paragraph. What question this campaign answers and why it is being run now.

---

## 2. Pre-Registration

Completed **before** the first run. Do not edit after results exist; append to §7 instead.

| Item | Value |
|---|---|
| Primary metric | |
| Secondary metrics | |
| Meaningful improvement margin | |
| Statistical test | |
| Sample size (folds × participants) | |
| Exclusion rules | |
| Hyperparameter budget per family | |

---

## 3. Environment

| Component | Version |
|---|---|
| OS | |
| Python | |
| MediaPipe | |
| PyTorch | |
| scikit-learn | |
| faster-whisper | |
| Hardware tier | `A` / `B` |
| CPU / GPU | |
| Camera | |
| Microphone | |

---

## 4. Runs

| Run ID | Cell | Protocol | Status | Macro F1 | Inference p95 | Notes |
|---|---|---|---|---|---|---|
| `run-` | | | | | | |

**Status values.** `completed`, `failed`, `invalid`, `aborted`. An `invalid` run is listed, not deleted — silent removal is a form of fabrication.

---

## 5. Results

### 5.1 Primary Comparison

| Cell | Metric | Median | IQR | n | Run |
|---|---|---|---|---|---|
| | | | | | |

### 5.2 Confusion Summary

Qualitative description of the dominant confusions. Full matrices are in `experiments/results/`.

### 5.3 Ablations

| Ablation | Condition | Metric | Δ vs baseline | Interpretation |
|---|---|---|---|---|
| `AB-01` | | | | |

---

## 6. Observations

Free text. Facts only, separated from interpretation.

---

## 7. Deviations

Anything that differed from the pre-registration. Each entry records what changed, why, and when.

| Date | Deviation | Reason | Impact on validity |
|---|---|---|---|
| | | | |

---

## 8. Threats and Limitations

Specific to this campaign, in addition to the general list in [RESEARCH_METHODOLOGY](../supporting/RESEARCH_METHODOLOGY.md) §9.

---

## 9. Conclusion

| Hypothesis | Supported? | Evidence |
|---|---|---|
| `H-` | `yes` / `no` / `inconclusive` | |

**Rules.**

- `inconclusive` is an acceptable and honest answer.
- A hypothesis is not "supported" by a direction alone; it requires the pre-registered test and effect size.
- Negative results are recorded with the same care as positive ones.

---

## 10. Artefacts

| Artefact | Path |
|---|---|
| Results export | |
| Confusion matrices | |
| Latency data | |
| Analysis notebook | |
| Charts | |

---

## 11. Follow-Up

| Item | Owner | Trigger |
|---|---|---|
| | | |

---

## Checklist Before Closing

- [ ] Every run has a valid row and a `run_id`
- [ ] Pre-registration untouched
- [ ] Deviations recorded with reasons
- [ ] Negative results included
- [ ] Every reported number cites a run
- [ ] No target value appears in a results table
- [ ] Artefacts path-verified
- [ ] Threats section written

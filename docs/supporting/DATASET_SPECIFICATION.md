# Dataset Specification

**Project:** Gesture-Controlled AI Computer Interface
**Version:** 1.0.0
**Owner:** Member 3 (Dataset) with Team Lead
**Last Updated:** 2026-11-06

---

## 1. Purpose

Defines the gesture vocabulary, landmark format, capture protocol, annotation schema, quality rules, splitting strategy, versioning, and consent handling for the project dataset.

**Scope boundary.** This document specifies *what is collected and how it is structured*. It does not specify model architecture ([MODEL_COMPARISON](./MODEL_COMPARISON.md)) or experimental design ([RESEARCH_METHODOLOGY](./RESEARCH_METHODOLOGY.md)).

**Privacy boundary.** Raw video and audio are **not** collected. Only derived landmarks and metadata are retained. This is a deliberate design choice (`NFR-021`) that reduces consent burden and storage risk.

---

## 2. Gesture Vocabulary

Twelve classes: six static, six dynamic. The split is not cosmetic — it drives preprocessing, window requirements, and the temporal hypothesis.

### 2.1 Static Classes

| # | Label | Description | Hold duration | Discriminating feature |
|---|---|---|---|---|
| 1 | `open_palm` | All five fingers extended, palm toward camera | 1.0 s | Maximum finger extension |
| 2 | `closed_fist` | All fingers curled into palm | 1.0 s | Minimum finger extension |
| 3 | `thumbs_up` | Fist with thumb extended upward | 1.0 s | Thumb extension, others curled |
| 4 | `peace` | Index and middle extended, others curled | 1.0 s | Two-finger configuration |
| 5 | `pinch` | Thumb and index tips in contact | 1.0 s | Thumb–index tip distance ≈ 0 |
| 6 | `point` | Index extended, others curled | 1.0 s | Single extended digit |

### 2.2 Dynamic Classes

| # | Label | Description | Typical duration | Discriminating feature |
|---|---|---|---|---|
| 7 | `swipe_left` | Hand moves laterally toward the user's left | 0.5–1.0 s | Dominant velocity direction: −x |
| 8 | `swipe_right` | Hand moves laterally toward the user's right | 0.5–1.0 s | Dominant velocity direction: +x |
| 9 | `swipe_up` | Hand moves vertically upward | 0.5–1.0 s | Dominant velocity direction: −y |
| 10 | `swipe_down` | Hand moves vertically downward | 0.5–1.0 s | Dominant velocity direction: +y |
| 11 | `circle_clockwise` | Hand traces a clockwise circle in the image plane | 1.2–2.5 s | Rotational trajectory, consistent winding |
| 12 | `wave` | Hand oscillates laterally at least twice | 1.0–2.0 s | Direction reversals ≥ 2 |

### 2.3 Why These Classes

| Consideration | Decision |
|---|---|
| Distinguishable by landmarks alone | All twelve are separable from joint positions; none requires appearance |
| Culturally neutral | No symbolic or offensive configurations |
| Covers both motion types | Enables the temporal hypothesis test |
| Maps to real actions | Each has a natural intent in at least one target application |
| Small enough to collect | 12 × 20 repetitions × 8 participants = 1 920 samples, feasible |

**Excluded classes and why.** Finger-spelling, numeric counts, and two-hand poses are excluded: they multiply collection cost and add little to the research question. They remain documented extension points.

---

## 3. Landmark Representation

### 3.1 Raw Recorded Format

Stored per sample as a NumPy archive.

| Array | Shape | dtype | Meaning |
|---|---|---|---|
| `landmarks` | `(T, 21, 3)` | float32 | MediaPipe normalised coordinates: `x`, `y` in image space, `z` relative depth |
| `visibility` | `(T, 21)` | float32 | Per-landmark presence/confidence |
| `handedness` | `(T,)` | int8 | `0` = left, `1` = right |

`T` varies per sample (raw capture length).

### 3.2 Why Raw Landmarks Are Stored, Not Derived Features

Derived features (`IC-FRAME`/`IC-FLAT`/`IC-SEQ`) are computed from stored landmarks at preprocessing time. Storing derived features would:

- prevent recomputation if the normalisation improves,
- freeze one `feature_spec_version` into the dataset,
- make it impossible to compare two preprocessing strategies on the same raw data.

`feature_spec_version` is therefore recorded as metadata, and the preprocessing code version is recorded per release.

### 3.3 Canonical Feature Derivation

Applied at preprocessing time ([TRD](../core/TRD.md) §7):

1. Translate to wrist origin (`landmark[0]`).
2. Scale by maximum wrist-to-landmark distance in that frame.
3. Mirror left-hand samples horizontally to a right-handed canonical form.
4. Compute velocities as first differences.
5. Concatenate → 126 features per frame.

**Invariance achieved:** translation, scale, handedness.
**Invariance deliberately not achieved:** rotation. Directional classes depend on image-plane orientation; rotation invariance would destroy them.

---

## 4. Sampling and Sequence Parameters

| Parameter | Value | Rationale |
|---|---|---|
| Requested capture rate | 30 FPS | Balance between motion resolution and CPU cost |
| Minimum accepted rate | 24 FPS | Below this, dynamic motion is undersampled |
| Resampling | Linear to a uniform 30 Hz timeline | Camera timestamps drift |
| Classification window | 30 frames (~1 s) | Covers the shortest dynamic class |
| Window stride | 3 frames at inference | Decouples inference cost from capture |
| Minimum valid-frame ratio | 80 % | Below this the sample is unusable |
| Static sample length | 45 frames (1.5 s) after stabilisation | Contains a stable hold |
| Dynamic sample length | Variable, clipped to 90 frames (3 s) | Covers the longest class |

**Resampling rule.** Samples captured below 30 FPS are resampled to 30 Hz rather than dropped, provided the native rate is ≥ 24 FPS. The native rate is recorded.

---

## 5. Annotation Format

### 5.1 Per-Sample Files

```
datasets/raw/<participant_id>/<session_id>/<class_label>/<sample_id>.npz
datasets/raw/<participant_id>/<session_id>/<class_label>/<sample_id>.json
```

**Metadata JSON:**

```json
{
  "sample_id": "smp-000123",
  "gesture_class": "swipe_right",
  "participant_id": "p-007",
  "session_id": "s-2026-11-02-a",
  "repetition_index": 4,
  "feature_spec_version": "1.0.0",
  "capture": {
    "camera_device": "USB2.0 HD UVC WebCam",
    "requested_fps": 30,
    "effective_fps": 29.4,
    "width": 640,
    "height": 480,
    "distance_m": 2.0,
    "lighting_condition": "normal",
    "handedness_observed": "right"
  },
  "timing": {
    "recorded_at": "2026-11-02T14:12:07.220Z",
    "frame_count": 34,
    "duration_ms": 1156
  },
  "quality": {
    "status": "accepted",
    "valid_frame_ratio": 0.97,
    "rejection_reason": null
  },
  "checksum_sha256": "9c1f…"
}
```

**Rules.**

- `participant_id` is a pseudonymous code, never a name.
- `distance_m` is recorded as a band (`1.0`, `1.5`, `2.0`, `2.5`, `3.0`) rather than an exact measurement.
- No field may contain free text that could identify a person.

### 5.2 Release Manifest

```
datasets/annotations/releases/ds-2026.10.json
```

```json
{
  "dataset_version": "ds-2026.10",
  "created_at": "2026-10-28T18:00:00Z",
  "feature_spec_version": "1.0.0",
  "preprocessing_git_revision": "a1b2c3d",
  "participants": [
    { "participant_id": "p-001", "sessions": 2, "samples": 240, "consent_version": "1.0" }
  ],
  "class_counts": { "swipe_right": 192, "open_palm": 192 },
  "total_samples": 1920,
  "split_manifests": ["split-loso-v1", "split-within-v1"],
  "files_checksum": "sha256:…"
}
```

### 5.3 Split Manifest

```json
{
  "split_manifest_id": "split-loso-v1",
  "protocol": "leave_one_subject_out",
  "dataset_version": "ds-2026.10",
  "created_at": "2026-10-28T18:05:00Z",
  "folds": [
    {
      "fold_index": 0,
      "held_out_participant": "p-001",
      "train_samples": ["smp-000001"],
      "validation_samples": ["smp-000014"],
      "test_samples": ["smp-000020"],
      "train_participants": ["p-002", "p-003"],
      "validation_participants": ["p-004"],
      "test_participants": ["p-001"]
    }
  ]
}
```

**Immutability.** A split manifest is written once and never edited. Corrections produce `split-loso-v2` with a documented reason.

---

## 6. Participant Metadata

Recorded in `users` ([BACKEND_SCHEMA](../core/BACKEND_SCHEMA.md) `ENT-01`), not in the sample files.

| Field | Required | Purpose |
|---|---|---|
| `participant_id` | Yes | Pseudonymous key |
| `dominant_hand` | Yes | Generalisation covariate |
| `experience_level` | Yes | Covariate; affects gesture consistency |
| `consent_granted` | Yes | Legal prerequisite |
| `consent_version` | Yes | Which consent text |
| `consent_granted_at` | Yes | Audit |
| Age band | Optional | Covariate only if consented; use bands, never exact age |
| Vision correction | Optional | Could affect pose consistency |
| Sessions completed | Derived | Provenance |

**Not collected.** Names, contact details, student IDs, photographs, video, audio, precise age, gender (unless a specific analysis requires it and is consented). The dataset must be usable without any of these.

---

## 7. Splitting Strategy

### 7.1 Protocol Hierarchy

| Priority | Protocol | Use |
|---|---|---|
| 1 | **Leave-one-subject-out** | All headline claims (`H5`) |
| 2 | Within-subject, session-disjoint | Upper bound reference |
| 3 | Cross-session | Robustness reference |
| 4 | Random sample split | Diagnostic only, clearly labelled optimistic |
| — | Random window split | **Prohibited** |

### 7.2 The Leakage Problem

A single recorded sample yields overlapping windows (stride 3 over 30 frames). If windows from one sample land in both train and test, the model has effectively memorised the sample. Accuracy looks excellent and means nothing.

**Enforced rules.**

1. Splits are defined over **samples**, then windows are derived within a split side.
2. Recorded samples are assigned to exactly one side.
3. The manifest validator asserts that no `sample_id` and no `participant_id` appears on both sides where the protocol forbids it.
4. Validation runs in CI and blocks experiment creation on failure (`FR-102`, `AC-009`).

### 7.3 Validation Slice

Within each LOSO fold, one training participant is held out as a validation set for hyperparameter selection. This is distinct from the test participant. The test fold is used exactly once per run.

### 7.4 Class Balance

If a class is underrepresented, the response is **not** to duplicate samples across splits. Options, in order:

1. Collect more data for that class.
2. Apply class weighting in the loss.
3. Report per-class metrics so the imbalance is visible.

Oversampling is permitted only within the training side, never across the split boundary.

---

## 8. Data Quality Checks

| ID | Check | Threshold | Action on failure |
|---|---|---|---|
| `QC-01` | Valid-frame ratio | ≥ 0.80 | Reject sample |
| `QC-02` | Effective capture rate | ≥ 24 FPS | Reject sample |
| `QC-03` | Duration within class bounds | See §2 | Reject sample |
| `QC-04` | Hand detected for the whole dynamic motion | ≥ 90 % of motion frames | Reject sample |
| `QC-05` | Motion energy for dynamic classes | Above calibrated floor | Reject (likely static) |
| `QC-06` | Motion energy for static classes | Below calibrated ceiling | Reject (likely moving) |
| `QC-07` | Duplicate checksum | Unique | Reject duplicate |
| `QC-08` | Landmark continuity | No frame-to-frame jump > 0.35 normalised units | Flag, then reject if repeated |
| `QC-09` | Class balance per participant | ≥ 15 usable samples per class | Warn; collect more |
| `QC-10` | Timestamp monotonicity | Strictly increasing | Reject sample |

**Rejection is retained, not deleted.** Rejected samples remain with `quality_status = 'rejected'` and a reason, so that collection quality can be analysed. They are excluded from training by query, not by deletion.

---

## 9. Data Augmentation

Augmentation is applied **only to the training side**, after splitting.

| Technique | Applies to | Parameter | Rationale |
|---|---|---|---|
| Time warping | Dynamic | Scale duration by 0.85–1.15 | Speed invariance |
| Temporal jitter | Both | Shift window by ±2 frames | Reduce phase sensitivity |
| Spatial jitter | Both | Translate landmarks by ±0.02 normalised units | Camera framing variance |
| Scale jitter | Both | Scale by 0.95–1.05 | Distance variance |
| Gaussian noise | Both | σ = 0.005 on normalised coordinates | Sensor noise |
| Small rotation | **Prohibited** | — | Destroy directional semantics |
| Horizontal flip | **Prohibited** | — | Handedness already normalised; flipping would create impossible left-hand samples |
| Class mixing / cutmix | **Prohibited** | — | Creates physically meaningless sequences |

**Augmentation policy.** No augmentation is applied to validation or test data. Every augmented sample derives from exactly one original, and augmentation never crosses the split boundary.

---

## 10. Dataset Versioning

| Element | Convention | Example |
|---|---|---|
| Version | `ds-YYYY.MM[-suffix]` | `ds-2026.10`, `ds-2026.11-pilot` |
| Split manifest | `split-<protocol>-v<N>` | `split-loso-v1` |
| Preprocessing | `feature_spec_version` | `1.0.0` |

**Release rules.**

1. A release is immutable. Any change produces a new version.
2. A release records the preprocessing revision that produced it.
3. A release records per-participant and per-class counts.
4. A release is checksummed; the manifest is the source of truth.
5. Models record the dataset version they trained on; a mismatch with the evaluation dataset is a leakage warning (`FR-111`, `INV-03`).

**Pilot datasets.** `ds-*-pilot` releases are for protocol tuning and are never used for reported results.

---

## 11. Consent and Privacy

| Requirement | Implementation |
|---|---|
| Informed consent before capture | Recorded in `users.consent_granted` with version and timestamp |
| Purpose stated | Research on gesture recognition; no commercial use |
| Data collected | Landmarks and non-identifying metadata only |
| Data not collected | Video, audio, names, contact details, photographs |
| Storage | Local, access-controlled, gitignored |
| Retention | Until withdrawal or project completion, then per institutional policy |
| Withdrawal | Honoured: files deleted, rows marked, affected releases flagged |
| Withdrawal effect on results | Results retained but annotated; deleting published results would falsify the record |
| Publication | Only aggregate metrics and pseudonymous examples |
| Minors | Excluded unless the institution's guardian-consent process is followed |

**Anonymity limit, stated honestly.** Hand kinematics may be weakly identifying in principle. The mitigation is pseudonymisation and aggregate reporting, not a claim of mathematical anonymity.

Full policy: [SECURITY_AND_PRIVACY](./SECURITY_AND_PRIVACY.md).

---

## 12. Collection Protocol

The protocol exists to minimise two things: measurement noise and participant leakage.

### 12.1 Environment

| Factor | Control |
|---|---|
| Distance | 2.0 m from camera (band recorded); 1.0 and 3.0 m for a robustness subset |
| Camera height | Approximately eye level, 1.4–1.6 m |
| Lighting | Even, no strong backlight; band recorded as bright/normal/dim |
| Background | Plain wall preferred; clutter recorded as a note |
| Audio | Not relevant to gesture capture |

### 12.2 Session Procedure

1. **Briefing.** Purpose, duration, what is recorded, right to withdraw. *(5 min)*
2. **Consent.** Written or digital signature; version recorded. *(5 min)*
3. **Metadata.** Dominant hand, experience level, vision correction. *(2 min)*
4. **Calibration.** Stand at 2.0 m; confirm landmarks track reliably. *(3 min)*
5. **Practice.** Three repetitions of each class, discarded. Reduces learning effects. *(5 min)*
6. **Capture — static block.** Six classes × 20 repetitions, randomised order, 2 s rest between. *(~10 min)*
7. **Rest.** *(3 min)*
8. **Capture — dynamic block.** Six classes × 20 repetitions, randomised order, 3 s rest. *(~12 min)*
9. **Robustness subset.** 5 repetitions per class at 1.0 m and 3.0 m. *(~8 min)*
10. **Debrief.** Questions, withdrawal reminder. *(3 min)*

Total: approximately 55–60 minutes.

### 12.3 Anti-Leakage Measures

| Measure | Why |
|---|---|
| One participant at a time | Prevents cross-contamination of session metadata |
| Session ID includes date and sequence | Enables session-disjoint splits |
| Class order randomised per participant | Prevents order-based artefacts |
| Practice block excluded | Reduces early-learning bias |
| Minimum 3 s between dynamic captures | Prevents adjacent-sample contamination |
| Recorded distance band per sample | Enables distance-held-out analysis |
| Participant code assigned before capture | Prevents name/ID mapping in raw files |

### 12.4 Minimum Viable Dataset

| Parameter | Minimum | Target |
|---|---|---|
| Participants | 8 | 12 |
| Repetitions per class per participant | 15 | 20 |
| Classes | 12 | 12 |
| Usable samples | 1 440 | 2 880 |
| LOSO folds | 8 | 12 |

Below 8 participants, generalisation claims are reported as preliminary and the limitation is stated in the report.

---

## 13. Directory Layout

```
datasets/
├── raw/
│   └── <participant_id>/
│       └── <session_id>/
│           └── <class_label>/
│               ├── <sample_id>.npz
│               └── <sample_id>.json
├── processed/
│   └── <dataset_version>/
│       ├── features/
│       └── indexes/
└── annotations/
    ├── releases/
    │   └── <dataset_version>.json
    └── splits/
        └── split-<protocol>-v<N>.json
```

**Git policy.** `datasets/raw/` and `datasets/processed/` are ignored. Release manifests and split manifests **are** tracked, because they are the reproducibility record and contain no personal data.

---

## 14. Related Documents

- [BACKEND_SCHEMA](../core/BACKEND_SCHEMA.md) `ENT-03` — sample metadata
- [TRD](../core/TRD.md) §7 — preprocessing contract
- [RESEARCH_METHODOLOGY](./RESEARCH_METHODOLOGY.md) — experimental use
- [SECURITY_AND_PRIVACY](./SECURITY_AND_PRIVACY.md) — consent and retention
- [MODEL_COMPARISON](./MODEL_COMPARISON.md) — how data feeds each architecture

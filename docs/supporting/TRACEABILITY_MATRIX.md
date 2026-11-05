# Traceability Matrix

**Project:** Gesture-Controlled AI Computer Interface
**Version:** 1.0.0
**Owner:** Team Lead
**Last Updated:** 2026-11-06

---

## 1. Purpose

Proves that every requirement has an owner component, a defined contract, a delivery phase, and at least one verification. It is the document that detects orphans: a requirement with no test, a component with no requirement, a phase that delivers nothing traceable.

**Columns.**

| Column | Meaning |
|---|---|
| **Req** | Requirement identifier from [PRD](../core/PRD.md) |
| **Component** | Architecture component from [TRD](../core/TRD.md) §5 |
| **Contract** | API endpoint (`EP`), event (`EV`), interface (`IF`), or entity (`ENT`) |
| **Phase** | Delivery phase from [IMPLEMENTATION_PLAN](../core/IMPLEMENTATION_PLAN.md) |
| **Test** | Test case from [TESTING_STRATEGY](./TESTING_STRATEGY.md) §14 |

---

## 2. Functional Requirements

### 2.1 System Control and Activation

| Req | Requirement | Component | Contract | Phase | Test |
|---|---|---|---|---|---|
| `FR-001` | Start in `IDLE`, execution disabled | C-01 | `EV-008` | PH-04 | `TC-020` |
| `FR-002` | Start/stop via API and dashboard | C-11, C-12 | `EP-002`, `EP-003` | PH-11 | `TC-058` |
| `FR-003` | Single status value from six states | C-01 | `EP-001`, `EV-008` | PH-04 | `TC-020` |
| `FR-004` | Three activation modes | C-01 | `ENT-10` | PH-04 | Gate: `TC-020` |
| `FR-005` | Inactivity returns to `IDLE` | C-01 | `EV-008` | PH-04 | `TC-020` |
| `FR-006` | Emergency stop < 100 ms | C-09 | `EP-006` | PH-10 | `TC-065` |
| `FR-007` | Stop via hotkey, API, dashboard | C-09, C-11, C-12 | `EP-006`, `EV-009` | PH-10 | `TC-085` |
| `FR-008` | Stop leads to `PAUSED`, explicit re-arm | C-01, C-09 | `EP-007` | PH-10 | `TC-086` |
| `FR-009` | Reason recorded per state transition | C-01 | `EV-008` | PH-04 | `TC-042` |

### 2.2 Perception

| Req | Requirement | Component | Contract | Phase | Test |
|---|---|---|---|---|---|
| `FR-010` | Configurable capture | C-02 | `IF-001` | PH-02 | `TC-063` |
| `FR-011` | Recover from camera loss | C-02 | `IF-001` | PH-02 | Fault injection §4.3 |
| `FR-012` | 21 landmarks + handedness + visibility | C-03 | `IF-002` | PH-02 | `TC-010` |
| `FR-013` | One hand, flag for two-hand | C-03 | `IF-002` | PH-02 | `TC-010` |
| `FR-014` | Translation/scale/handedness invariance | C-03 | `IF-002` | PH-02 | `TC-010`–`TC-012` |
| `FR-015` | Ring buffer ≥ 60 frames | C-03 | `IF-002` | PH-02 | `TC-013` |
| `FR-016` | Expose FPS and drops | C-02 | `EP-060` | PH-02 | `TC-063` |
| `FR-017` | No raw frame persistence | C-02 | — | PH-02 | Security §17 |

### 2.3 Gesture Recognition

| Req | Requirement | Component | Contract | Phase | Test |
|---|---|---|---|---|---|
| `FR-020` | No label from partial window | C-06 | `IF-003` | PH-04 | `TC-013` |
| `FR-021` | Label + confidence + score vector | C-06 | `IF-003`, `EV-001` | PH-04 | `TC-041` |
| `FR-022` | 12-class vocabulary | C-06 | `ENT-02` | PH-05 | `TC-090` |
| `FR-023` | Artefacts loaded via registry | C-06 | `IF-003`, `ENT-05` | PH-05 | `TC-045` |
| `FR-024` | Five families behind one interface | C-06 | `IF-003` | PH-05 | `TC-091` |
| `FR-025` | Model selected by configuration | C-06 | `EP-022` | PH-05 | `TC-058` |
| `FR-026` | Reject contract mismatch | C-06 | `E-3002` | PH-05 | `TC-045` |
| `FR-027` | Temporal stability required | C-06, C-09 | `IF-007` | PH-04 | `TC-025` |
| `FR-028` | Duplicate suppression | C-09 | `IF-007` | PH-04 | `TC-026` |
| `FR-029` | Emit `gesture.detected` | C-11 | `EV-001` | PH-04 | `TC-040` |
| `FR-030` | Single-hand only in v1 | C-06 | `IF-002` | PH-04 | `TC-010` |

### 2.4 Speech

| Req | Requirement | Component | Contract | Phase | Test |
|---|---|---|---|---|---|
| `FR-031` | Local transcription | C-04 | `IF-004` | PH-07 | `TC-092` |
| `FR-032` | Gated, not continuous | C-04 | `IF-004` | PH-07 | `TC-092` |
| `FR-033` | Transcript + confidence + timestamps | C-04 | `EV-002` | PH-07 | `TC-040` |
| `FR-034` | Close-vocabulary fuzzy normalisation | C-04 | Vocab §7.3 | PH-07 | `TC-093` |
| `FR-035` | Maximum 5 s utterance | C-04 | Config bound | PH-07 | `TC-092` |
| `FR-036` | No raw audio persistence | C-04 | — | PH-07 | Security §17 |
| `FR-037` | Replaceable engine | C-04 | `IF-004` | PH-07 | `TC-092` |
| `FR-038` | Engine identity in event | C-04, C-11 | `EV-002` | PH-07 | `TC-040` |

### 2.5 Application Context

| Req | Requirement | Component | Contract | Phase | Test |
|---|---|---|---|---|---|
| `FR-040` | Foreground app at ≥ 5 Hz | C-05 | `IF-005` | PH-08 | Context timing test |
| `FR-041` | Map app to adapter or `unknown` | C-05 | `ENT-08` | PH-08 | `TC-094` |
| `FR-042` | Focus change event < 300 ms | C-05 | `EV-003` | PH-08 | Context timing test |
| `FR-043` | Adapter-reported state | C-05, C-10 | `IF-008` | PH-08 | `TC-094` |
| `FR-044` | Verify focus before execution | C-09, C-10 | `IF-007` | PH-10 | `TC-083` |
| `FR-045` | `unknown` is conservative | C-05 | `EV-003` | PH-08 | `TC-094` |

### 2.6 Fusion and Intent

| Req | Requirement | Component | Contract | Phase | Test |
|---|---|---|---|---|---|
| `FR-050` | Associate evidence within window | C-07 | `IF-006` | PH-09 | `TC-095` |
| `FR-051` | Accept any subset of sources | C-07 | `IF-006` | PH-09 | `TC-095` |
| `FR-052` | Explicit abstention | C-07 | `EV-004` | PH-09 | `TC-053` |
| `FR-053` | Configurable weights, recorded | C-07 | `EV-004` | PH-09 | `TC-095` |
| `FR-054` | Per-application intent vocabulary | C-08 | `ENT-11` | PH-09 | `TC-051` |
| `FR-055` | Ranked alternatives | C-08 | `EV-004` | PH-09 | `TC-041` |
| `FR-056` | Three configurations by config | C-07 | Config §7.2 | PH-09 | `TC-052` |
| `FR-057` | Deterministic fusion | C-07 | `IF-006` | PH-09 | `TC-095` |
| `FR-058` | Emit `intent.predicted` incl. abstain | C-11 | `EV-004` | PH-09 | `TC-040` |

### 2.7 Safety

| Req | Requirement | Component | Contract | Phase | Test |
|---|---|---|---|---|---|
| `FR-060` | Nothing executes without the gate | C-09 | `IF-007` | PH-10 | `TC-080` |
| `FR-061` | Confidence thresholds per source | C-09 | Config §7.2 | PH-10 | `TC-023`, `TC-024` |
| `FR-062` | Action rate limit | C-09 | Config §7.2 | PH-10 | `TC-027` |
| `FR-063` | Adapter allowlist | C-09 | `ENT-08` | PH-10 | `TC-082` |
| `FR-064` | Dry-run mode | C-09 | Config §7.2 | PH-10 | `TC-056` |
| `FR-065` | Action classes + confirmation | C-09 | `IF-008` | PH-10 | `TC-031` |
| `FR-066` | Record rules evaluated | C-09 | `EV-009` | PH-10 | `TC-024` |
| `FR-067` | Deny on focus mismatch | C-09 | `E-7011` | PH-10 | `TC-083` |
| `FR-068` | Refuse invalid configuration | C-09 | `E-1001` | PH-10 | `TC-059` |

### 2.8 Adapters and Execution

| Req | Requirement | Component | Contract | Phase | Test |
|---|---|---|---|---|---|
| `FR-070` | Common adapter interface | C-10 | `IF-008` | PH-06 | `TC-070` |
| `FR-071` | PowerPoint actions | C-10 | `EP-031` | PH-06 | `TC-050` |
| `FR-072` | Media actions | C-10 | `EP-031` | PH-10 | `TC-051` |
| `FR-073` | IDE actions | C-10 | `EP-031` | PH-10 | Adapter harness |
| `FR-074` | Unsupported action fails explicitly | C-10 | `E-8002` | PH-10 | Adapter harness |
| `FR-075` | No pipeline change for a new adapter | C-10 | `IF-008` | PH-10 | `TC-070` |
| `FR-076` | Result with status and duration | C-10 | `EV-006` | PH-10 | Adapter harness |
| `FR-077` | No execution on unhealthy adapter | C-10 | `E-7010` | PH-10 | `TC-029` |
| `FR-078` | Execution off the inference thread | C-10 | Threading §18.1 | PH-10 | Concurrency test |

### 2.9 Backend

| Req | Requirement | Component | Contract | Phase | Test |
|---|---|---|---|---|---|
| `FR-080` | Versioned HTTP API | C-11 | `EP-001`–`EP-071` | PH-11 | `TC-041` |
| `FR-081` | WebSocket event stream | C-11 | `EP-090` | PH-11 | `TC-040` |
| `FR-082` | Persist per schema | C-11 | `ENT-01`–`ENT-13` | PH-11 | `TC-044` |
| `FR-083` | Validate every request | C-11 | `E-9001` | PH-11 | `TC-041` |
| `FR-084` | Expose subsystem health | C-11 | `EP-001` | PH-06 | `TC-058` |
| `FR-085` | Runtime configuration updates | C-11 | `EP-040` | PH-11 | `TC-058` |
| `FR-086` | Read-only model/experiment metadata | C-11 | `EP-020`, `EP-050` | PH-11 | `TC-041` |
| `FR-087` | Correlation id links stages | C-11 | Envelope §2 | PH-11 | `TC-042` |
| `FR-088` | Aggregated metrics | C-11 | `EP-060` | PH-11 | `TC-062` |

### 2.10 Dashboard

| Req | Requirement | Component | Contract | Phase | Test |
|---|---|---|---|---|---|
| `FR-090` | Status, app, model version | C-12 | `EP-001` | PH-11 | `TC-057` |
| `FR-091` | Live gesture/speech/intent feed | C-12 | `EV-001`, `EV-002`, `EV-004` | PH-11 | `TC-057` |
| `FR-092` | Decision trace | C-12 | `EV-009`, `EP-071` | PH-11 | `TC-024` |
| `FR-093` | Start/stop/arm/estop controls | C-12 | `EP-002`–`EP-006` | PH-11 | `TC-085` |
| `FR-094` | Latency and false-activation view | C-12 | `EP-060` | PH-11 | `TC-062` |
| `FR-095` | Mapping and threshold editing | C-12 | `EP-040`, `ENT-11` | PH-11 | `TC-058` |
| `FR-096` | Experiment result views | C-12 | `EP-050`, `EP-051` | PH-11 | `TC-041` |

### 2.11 Research Tooling

| Req | Requirement | Component | Contract | Phase | Test |
|---|---|---|---|---|---|
| `FR-100` | Guided capture with metadata | Tooling | `ENT-03` | PH-03 | `TC-090` |
| `FR-101` | Validate sample quality | Tooling | QC §8 | PH-03 | `TC-090` |
| `FR-102` | Subject-disjoint split manifests | Tooling | Split manifest | PH-03 | `TC-091` |
| `FR-103` | Five families from one harness | Tooling | `ENT-04` | PH-05 | `TC-091` |
| `FR-104` | Full metric reporting | Tooling | `ENT-07` | PH-05 | `TC-091` |
| `FR-105` | Provenance per run | Tooling | `ENT-06` | PH-05 | `TC-091` |
| `FR-106` | Inference latency p50/p95 | Tooling | `EV-001` | PH-12 | `TC-061` |
| `FR-107` | End-to-end latency p50/p95 | Tooling | `ENT-12` | PH-12 | `TC-062` |
| `FR-108` | False activations per minute | Tooling | `ENT-12` | PH-12 | False-activation protocol |
| `FR-109` | Ablations | Tooling | `ENT-06` | PH-12 | Ablation runs |
| `FR-110` | Machine-readable export | Tooling | `EP-052` | PH-12 | `TC-041` |
| `FR-111` | No test influence on training | Tooling | `INV-03` | PH-05 | `TC-091` |

### 2.12 Configuration

| Req | Requirement | Component | Contract | Phase | Test |
|---|---|---|---|---|---|
| `FR-120` | Versioned config with env overrides | C-11 | `ENT-13` | PH-04 | `TC-058` |
| `FR-121` | Validate config on load | C-11 | `E-1001` | PH-11 | `TC-059` |
| `FR-122` | Per-app mappings as data | C-11 | `ENT-11` | PH-11 | `TC-051` |
| `FR-123` | Adjustable thresholds without code | C-09 | `EP-040` | PH-11 | `TC-058` |
| `FR-124` | Rollback to last known-good | C-11 | `EP-043` | PH-11 | `TC-059` |

---

## 3. Non-Functional Requirements

| Req | Requirement | Component | Contract | Phase | Test |
|---|---|---|---|---|---|
| `NFR-001` | Landmark per frame ≤ 15 ms | C-02, C-03 | `EV-013` | PH-02 | `TC-060` |
| `NFR-002` | Inference p50 ≤ 15 / p95 ≤ 30 ms | C-06 | `EV-001` | PH-05 | `TC-061` |
| `NFR-003` | End-to-end p50 ≤ 400 / p95 ≤ 700 ms | All | `ENT-12` | PH-12 | `TC-062` |
| `NFR-004` | Speech latency p50 ≤ 900 ms | C-04 | `EV-002` | PH-07 | `TC-092` |
| `NFR-005` | Context refresh ≤ 200 ms | C-05 | `EV-003` | PH-08 | Context timing |
| `NFR-006` | UI propagation ≤ 150 ms | C-11, C-12 | `EP-090` | PH-11 | `TC-066` |
| `NFR-007` | Sustained ≥ 24 FPS | C-02 | `EV-013` | PH-02 | `TC-063` |
| `NFR-010` | Subsystem failure is isolated | C-01 | `EV-008` | PH-11 | Fault injection §4.3 |
| `NFR-011` | No unbounded memory growth | All | `EV-013` | PH-12 | `TC-064` |
| `NFR-012` | Bounded queues, drop-oldest | All | Threading §18.1 | PH-04 | `TC-064` |
| `NFR-013` | Every action logged or rejected | C-09 | `ENT-12` | PH-10 | `TC-042` |
| `NFR-014` | Automatic device recovery | C-02, C-04 | `IF-001` | PH-02 | Fault injection |
| `NFR-020` | No action outside the gate | C-09 | `IF-007` | PH-10 | `TC-080` |
| `NFR-021` | No raw media persistence | C-02, C-04 | — | PH-02 | Security §17 |
| `NFR-022` | Explicit per-session device access | C-01 | `EP-002` | PH-04 | `TC-058` |
| `NFR-023` | Visible capture indicator | C-12 | `EP-001` | PH-11 | UI test |
| `NFR-024` | Destructive actions require confirmation | C-09 | `IF-008` | PH-10 | `TC-031` |
| `NFR-025` | Local processing only | All | — | PH-02 | `TC-089` |
| `NFR-026` | Consent before dataset capture | Tooling | `ENT-01` | PH-03 | `TC-090` |
| `NFR-030` | Loopback binding | C-11 | `TR-001` | PH-11 | `TC-089` |
| `NFR-031` | Path traversal prevention | C-11 | `E-1003` | PH-11 | `TC-088` |
| `NFR-032` | No dynamic command strings | C-10 | `TR-002` | PH-10 | `TC-087` |
| `NFR-033` | No credentials in the repository | All | — | PH-01 | Pre-commit scan |
| `NFR-040` | Persistent state indicator | C-12 | `EP-001` | PH-11 | UI test |
| `NFR-041` | Human-readable rejection reasons | C-12 | `EV-009` | PH-11 | `TC-024` |
| `NFR-042` | Operable at 1–3 m, typical lighting | C-02, C-03 | — | PH-02 | Distance/lighting protocol |
| `NFR-043` | No colour-only state encoding | C-12 | — | PH-11 | Accessibility test |
| `NFR-044` | Three-step documented onboarding | Docs | README | PH-06 | Manual |
| `NFR-050` | Adapter addition is localised | C-10 | `IF-008` | PH-10 | `TC-070` |
| `NFR-051` | Model addition is one interface | C-06 | `IF-003` | PH-05 | `TC-091` |
| `NFR-052` | Type-annotated public interfaces | All | — | PH-02 | mypy CI |
| `NFR-053` | Modules unit-testable without devices | All | — | PH-02 | `TR-028` |
| `NFR-054` | Docs are the source of truth | All | — | PH-01 | Docs review |
| `NFR-060` | Windows reference, others for non-adapter code | All | — | PH-02 | CI matrix |
| `NFR-061` | Runs without a discrete GPU | All | — | PH-02 | Tier A benchmark |
| `NFR-062` | CI runs without hardware | All | — | PH-02 | CI |
| `NFR-063` | Deterministic replay mode | All | `IF-001` | PH-04 | `TC-013` |
| `NFR-070` | Every stage emits typed events | All | Envelope §2 | PH-04 | `TC-040` |
| `NFR-071` | Structured log fields | All | Logging §4 | PH-02 | Log schema test |
| `NFR-072` | Per-stage latency recorded | All | `EV-001`, `ENT-12` | PH-02 | `TC-060` |
| `NFR-073` | Session trace export | C-11 | `EP-071` | PH-11 | `TC-042` |

---

## 4. Technical Requirements Coverage

| Req | Component | Phase | Test |
|---|---|---|---|
| `TR-001`–`TR-009` | C-02, C-04, C-05, C-09, C-10, C-11 | PH-10, PH-11 | `TC-080`–`TC-089` |
| `TR-010`–`TR-014` | C-06, C-07, C-09 | PH-04, PH-09 | `TC-013`, `TC-045`, `TC-095` |
| `TR-015`–`TR-020` | All | PH-04, PH-11 | `TC-013`, `TC-040`, `TC-043` |
| `TR-021`–`TR-025` | C-06, C-11 | PH-05, PH-11 | `TC-045`, `TC-057`, `TC-059` |
| `TR-026`–`TR-030` | All, Tooling | PH-02, PH-05, PH-10 | `TC-070`, `TC-091`, import-graph test |

---

## 5. Acceptance Criteria Coverage

| AC | Criterion (short) | Verified by | Phase |
|---|---|---|---|
| `AC-001` | 15+ FPS landmark stability | `TC-063` | PH-02 |
| `AC-002` | 6 MVP gestures 9/10 | Capture session | PH-06 |
| `AC-003` | Only wake pose arms | Negative test across classes | PH-04 |
| `AC-004` | Swipe advances one slide | `TC-050` | PH-06 |
| `AC-005` | No action while idle | `TC-020` | PH-04 |
| `AC-006` | 10-min session, no leak | `TC-064` | PH-12 |
| `AC-007` | 12 classes via registry | `TC-045` | PH-05 |
| `AC-008` | Five families, one harness | `TC-091` | PH-05 |
| `AC-009` | Subject-disjoint manifest | `TC-091` | PH-03 |
| `AC-010` | Comparison report complete | Review | PH-05 |
| `AC-011` | Complete run provenance | `TC-091` | PH-05 |
| `AC-012` | Frame vs temporal comparison | `TC-091` | PH-05 |
| `AC-013` | Deterministic replay | `TC-013` | PH-04 |
| `AC-014` | False activations under budget | Protocol §8 | PH-12 |
| `AC-015` | No unauthorised execution | `TC-080`–`TC-086` | PH-10 |
| `AC-016` | LOSO completes for all configs | Experiment run | PH-12 |
| `AC-017` | Same gesture, different apps | `TC-051` | PH-09 |
| `AC-018` | Three configs on same splits | `TC-052` | PH-09 |
| `AC-019` | Ablation reported, positive or null | Review | PH-12 |
| `AC-020` | Estop under 100 ms | `TC-065` | PH-10 |
| `AC-021` | Latency p50/p95 with boundaries | `TC-062` | PH-12 |
| `AC-022` | Adapter added without pipeline change | `TC-070` | PH-10 |
| `AC-023` | Invalid config rejected, old retained | `TC-059` | PH-11 |
| `AC-024` | Decision trace shows all rules | `TC-024` | PH-11 |

---

## 6. Component Coverage

Every component must be justified by at least one requirement. An orphan component indicates over-engineering.

| Component | Requirements served | Count |
|---|---|---|
| C-01 Perception Layer | `FR-001`–`FR-005`, `FR-009`, `NFR-010`, `NFR-022` | 8 |
| C-02 Camera | `FR-010`, `FR-011`, `FR-016`, `FR-017`, `NFR-001`, `NFR-021` | 6 |
| C-03 Landmarks | `FR-012`–`FR-015`, `FR-030`, `NFR-001` | 6 |
| C-04 Speech | `FR-031`–`FR-038`, `NFR-004` | 9 |
| C-05 Context | `FR-040`–`FR-045`, `NFR-005` | 7 |
| C-06 Recognition | `FR-020`–`FR-030`, `FR-106`, `NFR-002` | 12 |
| C-07 Fusion | `FR-050`–`FR-058` | 9 |
| C-08 Intent | `FR-054`–`FR-055` | 2 |
| C-09 Safety | `FR-006`–`FR-008`, `FR-060`–`FR-068`, `NFR-013`, `NFR-020` | 12 |
| C-10 Adapters | `FR-070`–`FR-078`, `NFR-032` | 10 |
| C-11 Backend | `FR-080`–`FR-088`, `NFR-030`–`NFR-031` | 11 |
| C-12 Dashboard | `FR-090`–`FR-096`, `NFR-023`, `NFR-040`–`NFR-043` | 11 |

**Observation.** C-08 (Intent) has only two direct requirements because its behaviour is largely specified through `FR-054`–`FR-055` plus the mapping entities. It is retained as a separate component because it has an independent failure mode (wrong vocabulary resolution) and a distinct test surface. This is recorded rather than silently merging it into C-07.

---

## 7. Phase Coverage

| Phase | Requirements delivered | Acceptance criteria |
|---|---|---|
| PH-01 | `NFR-033`, `NFR-054` | — |
| PH-02 | `FR-010`–`FR-017`, `NFR-001`, `NFR-007`, `NFR-021`, `NFR-042`, `NFR-052`–`NFR-053`, `NFR-060`–`NFR-063`, `NFR-070`–`NFR-072` | `AC-001` |
| PH-03 | `FR-100`–`FR-102`, `NFR-026` | `AC-009` |
| PH-04 | `FR-001`, `FR-003`–`FR-005`, `FR-009`, `FR-020`, `FR-021`, `FR-027`–`FR-030`, `FR-120`, `NFR-012`, `NFR-022`, `NFR-063` | `AC-003`, `AC-005`, `AC-013` |
| PH-05 | `FR-022`–`FR-026`, `FR-103`–`FR-105`, `FR-111`, `NFR-002`, `NFR-051` | `AC-007`, `AC-008`, `AC-010`–`AC-012` |
| PH-06 | `FR-070`–`FR-071`, `FR-084`, `NFR-044` | `AC-002`, `AC-004` |
| PH-07 | `FR-031`–`FR-038`, `NFR-004` | — |
| PH-08 | `FR-040`–`FR-045`, `NFR-005` | — |
| PH-09 | `FR-050`–`FR-058` | `AC-017`, `AC-018` |
| PH-10 | `FR-006`–`FR-008`, `FR-060`–`FR-068`, `FR-072`–`FR-078`, `NFR-013`, `NFR-020`, `NFR-024`, `NFR-032`, `NFR-050` | `AC-015`, `AC-020`, `AC-022` |
| PH-11 | `FR-002`, `FR-080`–`FR-083`, `FR-085`–`FR-088`, `FR-090`–`FR-096`, `FR-121`–`FR-124`, `NFR-006`, `NFR-010`, `NFR-023`, `NFR-030`–`NFR-031`, `NFR-040`–`NFR-041`, `NFR-043`, `NFR-073` | `AC-023`, `AC-024` |
| PH-12 | `FR-106`–`FR-110`, `NFR-003`, `NFR-011` | `AC-006`, `AC-014`, `AC-016`, `AC-019`, `AC-021` |

---

## 8. Orphan Analysis

| Check | Result |
|---|---|
| Requirements with no component | None |
| Requirements with no phase | None |
| Requirements with no test or verification | None |
| Components with no requirement | None |
| Phases with no deliverable requirement | None |
| Acceptance criteria with no test | None |
| Acceptance criteria with no requirement | None |

**Gaps deliberately accepted.**

| Gap | Reason |
|---|---|
| `NFR-042` (distance/lighting operability) has a protocol, not an automated test | Physical conditions cannot be asserted in CI; verified once by protocol and declared a limitation |
| `NFR-044` (onboarding) verified manually | Setup instructions cannot be automatically validated without a clean-machine CI runner |
| `AC-010` and `AC-019` verified by report review | Report completeness is a human judgement |

These are recorded rather than hidden so that an examiner can see what is machine-verified and what is reviewed.

---

## 9. Maintenance

| Trigger | Update required |
|---|---|
| New requirement | Row in §2, component count, phase coverage |
| Requirement removed | Row removed, justification recorded |
| Component renamed | All references, `C-` IDs preserved |
| Phase reordering | §7 and [IMPLEMENTATION_PLAN](../core/IMPLEMENTATION_PLAN.md) §9 |
| New test case | §14 of [TESTING_STRATEGY](./TESTING_STRATEGY.md) and the relevant row |
| Contract change | Contract column, plus the owning document |

**Review cadence.** The matrix is reviewed at every milestone gate. A requirement delivered without a row update is treated as undocumented.

---

## 10. Related Documents

- [PRD](../core/PRD.md) — requirement source
- [TRD](../core/TRD.md) — component definitions
- [API_AND_INTEGRATION](../core/API_AND_INTEGRATION.md) — contract identifiers
- [BACKEND_SCHEMA](../core/BACKEND_SCHEMA.md) — entity identifiers
- [IMPLEMENTATION_PLAN](../core/IMPLEMENTATION_PLAN.md) — phase definitions
- [TESTING_STRATEGY](./TESTING_STRATEGY.md) — test case register

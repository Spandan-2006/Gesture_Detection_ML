# Repository Structure

**Project:** Gesture-Controlled AI Computer Interface
**Version:** 1.0.0
**Owner:** Team Lead
**Last Updated:** 2026-11-06

---

## 1. Purpose

This document defines the target repository layout, the purpose and ownership of every directory, and — critically — **when** each directory is created. It is a blueprint, not an inventory.

## 2. What Exists Today

At the documentation baseline, the repository contains only documentation and repository-level configuration. There is no application code. This is intentional: the project brief prohibits generating placeholder modules during the documentation phase, and empty packages tend to be filled with invented structure before the design is settled.

```
gesture-ai-interface/
├── LICENSE
├── README.md
├── CONTRIBUTING.md
├── .gitignore
├── .env.example
└── docs/
    ├── core/
    │   ├── PRD.md
    │   ├── TRD.md
    │   ├── BACKEND_SCHEMA.md
    │   ├── IMPLEMENTATION_PLAN.md
    │   └── API_AND_INTEGRATION.md
    ├── supporting/
    │   ├── REPOSITORY_STRUCTURE.md
    │   ├── TECH_STACK.md
    │   ├── MODEL_COMPARISON.md
    │   ├── RESEARCH_METHODOLOGY.md
    │   ├── DATASET_SPECIFICATION.md
    │   ├── TESTING_STRATEGY.md
    │   ├── SECURITY_AND_PRIVACY.md
    │   ├── DEVELOPMENT_GUIDELINES.md
    │   └── TRACEABILITY_MATRIX.md
    ├── architecture/
    │   ├── SYSTEM_ARCHITECTURE.md
    │   ├── diagrams/
    │   │   └── README.md
    │   └── adr/
    │       ├── ADR-0001 … ADR-0010
    ├── research/
    │   ├── EXPERIMENT_DESIGN.md
    │   ├── EVALUATION_METRICS.md
    │   └── EXPERIMENT_LOG_TEMPLATE.md
    └── reports/
        └── .gitkeep
```

## 3. Target Structure

```
gesture-ai-interface/
│
├── README.md                      # Entry point, navigation, status
├── LICENSE                        # MIT
├── CONTRIBUTING.md                # Contribution workflow
├── .gitignore                     # Artefact exclusions
├── .env.example                   # Environment variable template
├── pyproject.toml                 # Package metadata, deps, tool config   [created PH-02]
│
├── docs/                          # AUTHORITATIVE DOCUMENTATION
│   ├── core/                      # 5 source-of-truth documents
│   ├── supporting/                # Elaborations, never contradicting core
│   ├── architecture/              # System architecture, diagrams, ADRs
│   │   ├── diagrams/
│   │   └── adr/
│   ├── research/                  # Experiment design, metrics, logs
│   └── reports/                   # Generated reports, phase logs
│
├── src/                           # APPLICATION SOURCE                    [created PH-02]
│   ├── perception/
│   │   ├── camera/                # C-02
│   │   ├── hand_tracking/         # C-03
│   │   ├── speech/                # C-04
│   │   └── pipeline.py            # C-01
│   ├── preprocessing/             # Normalisation, windowing, features
│   ├── contracts/                 # DTOs and protocol definitions
│   ├── recognition/
│   │   ├── classical/             # Random Forest
│   │   ├── mlp/
│   │   ├── lstm/
│   │   ├── gru/
│   │   ├── transformer/
│   │   └── registry.py            # Model registry
│   ├── fusion/                    # C-07
│   ├── context/                   # C-05
│   ├── intent/                    # C-08
│   ├── execution/
│   │   ├── adapters/              # C-10
│   │   └── safety/                # C-09
│   ├── backend/                   # C-11
│   └── dashboard/                 # C-12
│
├── datasets/                      # S5 RESEARCH DATA                   [created PH-03, gitignored]
│   ├── raw/
│   ├── processed/
│   └── annotations/
│
├── models/                        # S4 BINARY ARTEFACTS                [created PH-05]
│   ├── checkpoints/
│   └── configs/
│
├── experiments/                   # TRAINING / EVALUATION              [created PH-05]
│   ├── configs/
│   ├── notebooks/
│   └── results/
│
├── tests/                         # TEST SUITE                         [created PH-02]
│   ├── unit/
│   ├── integration/
│   ├── performance/
│   └── end_to_end/
│
├── scripts/                       # OPERATIONAL SCRIPTS                [created PH-02]
│   ├── setup/
│   ├── data/
│   └── training/
│
├── config/                        # DEFAULT CONFIGURATION              [created PH-04]
│   └── default.yaml
│
└── logs/                          # RUNTIME OUTPUT                     [created PH-02, gitignored]
```

## 4. Directory Ownership

| Directory | Owner | Purpose | Git policy |
|---|---|---|---|
| `docs/core/` | Lead | Authoritative specifications | Tracked |
| `docs/supporting/` | Lead, with domain owners | Elaborations | Tracked |
| `docs/architecture/` | Lead | Architecture and decision records | Tracked |
| `docs/research/` | Lead + M3 | Experimental design and metrics | Tracked |
| `docs/reports/` | All | Phase logs, generated reports | Tracked (small text only) |
| `src/perception/camera/` | M2 | Frame acquisition | Tracked |
| `src/perception/hand_tracking/` | M2 | Landmark extraction | Tracked |
| `src/perception/speech/` | M2 | Speech recognition | Tracked |
| `src/preprocessing/` | M2 + M3 | Normalisation and feature building | Tracked |
| `src/contracts/` | Lead | DTOs and protocols — change requires review | Tracked |
| `src/recognition/` | M3 | Models and registry | Tracked |
| `src/fusion/` | Lead | Multimodal fusion | Tracked |
| `src/context/` | M4 | Application context | Tracked |
| `src/intent/` | Lead | Intent resolution | Tracked |
| `src/execution/adapters/` | M4 | Application automation | Tracked |
| `src/execution/safety/` | M4 + Lead | Decision gate | Tracked |
| `src/backend/` | M4 | API and persistence | Tracked |
| `src/dashboard/` | M5 | Web client | Tracked |
| `datasets/` | M3 | Landmark samples and manifests | **Ignored** (except `.gitkeep` and manifests) |
| `models/` | M3 | Checkpoints and model cards | **Ignored** (model cards may be tracked) |
| `experiments/` | M3 | Configs, notebooks, results | Configs tracked; results ignored |
| `tests/` | M5 | All test levels | Tracked |
| `scripts/` | All | Automation | Tracked |
| `config/` | M4 | Defaults | Tracked |
| `logs/` | — | Runtime output | **Ignored** |

## 5. Creation Order by Phase

| Phase | Directories created | Rationale |
|---|---|---|
| `PH-01` | `docs/**` | Documentation baseline |
| `PH-02` | `src/perception/`, `src/preprocessing/`, `src/contracts/`, `tests/`, `scripts/`, `logs/`, `pyproject.toml` | First code |
| `PH-03` | `datasets/**` | Data collection begins |
| `PH-04` | `src/recognition/`, `src/execution/safety/`, `config/` | Recognition and gate |
| `PH-05` | `models/**`, `experiments/**`, `src/execution/adapters/` | Training artefacts |
| `PH-07` | `src/perception/speech/` | Speech channel |
| `PH-08` | `src/context/` | Context channel |
| `PH-09` | `src/fusion/`, `src/intent/` | Fusion |
| `PH-11` | `src/backend/`, `src/dashboard/` | Integration surface |

**Rule.** A directory is created when its first real file is written, not in advance. No `.gitkeep` is used inside `src/` because an empty package is an invitation to invent structure.

## 6. What Is Not in This Repository

| Excluded | Why | Where it lives |
|---|---|---|
| Virtual environments | Machine-specific | `.venv/` (gitignored) |
| Node modules | Reproducible from lockfile | `src/dashboard/node_modules/` (gitignored) |
| Raw video | Privacy (`NFR-021`) | Nowhere |
| Raw audio | Privacy (`NFR-021`) | Nowhere |
| Large datasets | Size and privacy | `datasets/` (gitignored), released separately if needed |
| Model weights | Size | `models/checkpoints/` (gitignored) |
| Secrets | Security (`NFR-033`) | Environment |
| Generated reports | Regenerable | `docs/reports/` for small text; `experiments/results/` for data |

## 7. Naming Conventions

| Artefact | Convention | Example |
|---|---|---|
| Documentation | `SCREAMING_SNAKE_CASE.md` | `BACKEND_SCHEMA.md` |
| Python modules | `snake_case.py` | `landmark_extractor.py` |
| Python packages | `snake_case/` | `hand_tracking/` |
| Test files | `test_<module>.py` | `test_decision_gate.py` |
| Dataset releases | `ds-YYYY.MM[-suffix]` | `ds-2026.10` |
| Model checkpoints | `<model_id>/<version>/` | `lstm/1.3.0/` |
| Experiment runs | `run-YYYYMMDD-NNNN` | `run-20261101-0007` |
| ADRs | `ADR-NNNN-title.md` | `ADR-0006-fastapi-eventbus.md` |
| Config versions | `cfg-NNNN` | `cfg-0007` |

## 8. Related Documents

- [TRD](../core/TRD.md) §27 — module-to-directory mapping
- [DEVELOPMENT_GUIDELINES](./DEVELOPMENT_GUIDELINES.md) — coding and commit conventions
- [DATASET_SPECIFICATION](./DATASET_SPECIFICATION.md) — dataset layout inside `datasets/`

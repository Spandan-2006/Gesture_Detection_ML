# Technology Stack

**Project:** Gesture-Controlled AI Computer Interface
**Version:** 1.0.0
**Owner:** Lead, with M4 (backend) and M5 (frontend)
**Last Updated:** 2026-11-06

---

## 1. How to Read This Document

Every technology is assessed on the same axes: purpose, alternatives, advantages, limitations, integration considerations, rationale, and **decision status**.

| Status | Meaning |
|---|---|
| **Selected** | Committed for the indicated phase. Changing it requires an ADR. |
| **Proposed** | Directionally chosen but not yet validated by a spike. May change on evidence. |
| **Deferred** | Deliberately not adopted in v1. Revisit only under stated conditions. |

No technology is installed at the documentation baseline. Installation begins in `PH-02`.

---

## 2. Decision Summary

| Layer | Technology | Status | Phase |
|---|---|---|---|
| Language | Python 3.11 / 3.12 | Proposed | `PH-02` |
| Vision capture | OpenCV | Selected | `PH-02` |
| Landmarks | MediaPipe Tasks | Selected | `PH-02` |
| Numerical | NumPy | Selected | `PH-02` |
| Deep learning | PyTorch | Selected | `PH-05` |
| Classical ML | scikit-learn | Selected | `PH-05` |
| API | FastAPI + Uvicorn | Selected | `PH-11` |
| Validation | Pydantic v2 | Selected | `PH-11` |
| Real-time transport | WebSocket | Selected | `PH-11` |
| Database | SQLite | Selected | `PH-11` |
| ORM / migrations | SQLAlchemy 2.x + Alembic | Selected | `PH-11` |
| Speech | faster-whisper | Proposed | `PH-07` |
| OS automation | Win32 + pyautogui (fallback) | Proposed | `PH-10` |
| Frontend | React + Vite + TypeScript | Selected | `PH-11` |
| Testing | pytest | Selected | `PH-02` |
| Experiment tracking | MLflow | **Deferred** | — |
| PostgreSQL | PostgreSQL 15+ | **Deferred** | — |

---

## 3. Language — Python 3.11 / 3.12

| Aspect | Detail |
|---|---|
| Purpose | Sole implementation language for the pipeline, backend, and tooling |
| Alternatives | C++ (performance), JavaScript/TypeScript (unified stack), Rust (safety) |
| Advantages | Native ecosystem for CV, ML, and speech; fastest path for a five-person research team; single language across pipeline and tooling |
| Limitations | GIL constrains CPU-parallel inference; packaging on Windows is awkward; startup time is non-trivial |
| Integration | MediaPipe, PyTorch, and faster-whisper all publish CPython wheels; CPython ABI matters more than the language version |
| Rationale | The project is a research prototype, not a shipping product. Ecosystem depth dominates micro-optimisation. |
| **Status** | **Proposed** — 3.11/3.12 pending MediaPipe wheel verification (`OQ-01`). The development machine currently has 3.13.7, which must be validated before pinning. |
| Risk | If MediaPipe lacks a 3.13 wheel, install 3.12 via the official installer rather than a third-party build |

**Constraint recorded.** Do not adopt 3.13 merely because it is installed. Pin the interpreter that the vision stack supports, and document it in `pyproject.toml`.

---

## 4. Computer Vision

### 4.1 OpenCV

| Aspect | Detail |
|---|---|
| Purpose | Camera capture, colour conversion, resizing, preview overlay |
| Alternatives | `picamera2` (Raspberry Pi only), DirectShow via `pygrabber`, `imageio` |
| Advantages | Mature Windows capture backends (MSMF/DirectShow); `cv2.VideoCapture` handles device indexing; drawing primitives for the preview |
| Limitations | Backend selection is opaque across machines; occasional buffer latency; no audio |
| Integration | Capture runs in its own thread, timestamps at acquisition, publishes `FramePacket` |
| Rationale | Zero viable alternatives for cross-backend Windows capture with in-process drawing |
| **Status** | **Selected** |
| Known pitfall | Default backend may buffer several frames, inflating latency. Set `CAP_PROP_BUFFERSIZE` to 1 and verify empirically. |

### 4.2 MediaPipe (Tasks API)

| Aspect | Detail |
|---|---|
| Purpose | 21-point hand landmark extraction plus handedness |
| Alternatives | YOLO-based hand pose (custom training), OpenPose (heavy), Apple Vision (platform-locked), raw `cv2` contour heuristics |
| Advantages | No training required; CPU-friendly; stable point semantics across users; well-documented landmark indices |
| Limitations | Vendor wheel availability lags CPython releases; model size and accuracy trade-off is fixed by the vendor; limited guidance on extreme occlusion |
| Integration | Wrapped behind `IF-002` so the extractor is replaceable without touching recognition or fusion |
| Rationale | Training a landmark model is a separate research project. MediaPipe converts a hard CV problem into a solved dependency. |
| **Status** | **Selected**, with the version pinned and recorded in `model_versions.environment_json` |
| Risk | If accuracy at 3 m proves inadequate, the mitigation is protocol change (distance, framing), not a model rewrite |

### 4.3 NumPy

| Aspect | Detail |
|---|---|
| Purpose | Landmark arrays, normalisation, windowing, feature construction |
| Alternatives | PyTorch tensors throughout; plain Python lists |
| Advantages | Vectorised, zero-copy views for windowing, universal interop |
| Limitations | No GPU; float64 default can silently double memory |
| Integration | Feature arrays are float32 at the boundary; models accept NumPy and convert internally |
| **Status** | **Selected** |

---

## 5. Machine Learning

### 5.1 PyTorch

| Aspect | Detail |
|---|---|
| Purpose | LSTM, GRU, MLP, Transformer Encoder training and CPU inference |
| Alternatives | TensorFlow/Keras, JAX, ONNX Runtime only |
| Advantages | Eager debugging; simple custom sequence models; strong Windows CPU support; `eval()` + `no_grad()` gives predictable inference latency |
| Limitations | Large install footprint; CPU inference is slower than optimised runtimes; version pinning matters |
| Integration | Models are exported to `state_dict` plus a JSON model card. The runtime loads only through the registry. |
| Rationale | Debuggability matters more than raw throughput at this data scale |
| **Status** | **Selected** |

### 5.2 scikit-learn

| Aspect | Detail |
|---|---|
| Purpose | Random Forest baseline, metrics, confusion matrices, splitting utilities |
| Alternatives | XGBoost/LightGBM (not needed at this scale), statsmodels |
| Advantages | Random Forest with a two-line API; metrics module is the reference implementation; stable |
| Limitations | No GPU; pickled models are version-sensitive |
| Integration | Exported via `joblib` with a pinned scikit-learn version recorded in the model card |
| Rationale | The classical baseline must be trustworthy and boring. scikit-learn is both. |
| **Status** | **Selected** |

### 5.3 MLflow — Deferred

| Aspect | Detail |
|---|---|
| Purpose | Experiment tracking and artefact lineage |
| Advantages | Standard UI, metric comparison, artefact storage |
| Limitations | A server or SQLite backend to operate; duplicates the custom `experiments` schema already specified in [BACKEND_SCHEMA](../core/BACKEND_SCHEMA.md) |
| Deferral rationale | The project already defines a purpose-built experiment schema with the exact fields the research needs. Introducing MLflow would create two overlapping sources of truth. |
| Revisit when | More than three people run experiments concurrently, or hyperparameter sweeps exceed what the custom harness can enumerate |
| **Status** | **Deferred** |

---

## 6. Backend

### 6.1 FastAPI + Uvicorn

| Aspect | Detail |
|---|---|
| Purpose | REST API, WebSocket hub, static dashboard serving |
| Alternatives | Flask (+ flask-sock), Django REST, Litestar, aiohttp |
| Advantages | Native async for WebSockets; Pydantic-based validation; automatic OpenAPI for contract testing; small surface |
| Limitations | Async/sync boundary is a real footgun — a blocking call stalls all clients; no built-in auth |
| Integration | Blocking work is pushed to worker threads. Handlers never call inference or automation directly. |
| Rationale | WebSocket + typed request models in one framework, with the least ceremony |
| **Status** | **Selected** |
| Guardrail | A single blocked event loop is a project-ending demo bug. The rule is enforced in review: no CPU-bound work inside a route. |

### 6.2 Pydantic v2

| Aspect | Detail |
|---|---|
| Purpose | Request/response validation, configuration schema |
| Alternatives | `attrs` + `cattrs`, `marshmallow`, dataclasses with manual checks |
| Advantages | Rust-backed validation; JSON Schema export; nested models; clear error paths |
| Limitations | v1→v2 API differences cause confusion; strict mode requires explicit configuration |
| Integration | API schemas mirror `src/contracts/`; configuration validation uses the same models |
| **Status** | **Selected** |

### 6.3 SQLite

| Aspect | Detail |
|---|---|
| Purpose | Default persistence for all relational data |
| Alternatives | PostgreSQL, DuckDB, JSON files |
| Advantages | Zero installation; single file; sufficient for single-user write volume; trivial backup |
| Limitations | One writer at a time; no network access; limited concurrent analytics |
| Integration | SQLAlchemy abstracts the dialect. WAL mode enabled. Writes batched off the inference path. |
| Rationale | Documented in [BACKEND_SCHEMA](../core/BACKEND_SCHEMA.md) §3 and ADR-0005 |
| **Status** | **Selected** |

### 6.4 PostgreSQL — Deferred

| Aspect | Detail |
|---|---|
| Purpose | Optional production-grade store |
| Advantages | MVCC, concurrent writers, strong analytics |
| Limitations | Service dependency for every demo and every teammate setup |
| Deferral rationale | No v1 requirement needs it. The schema is written to be portable so adoption is a migration, not a rewrite. |
| Revisit trigger | Concurrent writers, or sustained writes above ~50/s |
| **Status** | **Deferred** |

### 6.5 SQLAlchemy 2.x + Alembic

| Aspect | Detail |
|---|---|
| Purpose | ORM, schema definition, migrations |
| Alternatives | Raw `sqlite3`, Django ORM, Peewee, Tortoise |
| Advantages | Dialect portability; typed ORM; Alembic gives versioned, reviewable migrations |
| Limitations | Abstraction can hide dialect-specific performance; migration discipline is required |
| Integration | Single declarative base; migrations run at startup in dev, explicitly in demo |
| **Status** | **Selected** |

---

## 7. Speech

### 7.1 faster-whisper — Proposed

| Aspect | Detail |
|---|---|
| Purpose | Local speech-to-text for a closed command vocabulary |
| Alternatives | `openai-whisper` (slower, same models), Vosk (smaller, dictation-oriented), Windows SAPI (weak accuracy, no control), cloud STT (violates local-only) |
| Advantages | CPU INT8 inference; streaming-friendly; no network; multiple model sizes to trade accuracy for latency |
| Limitations | Model download at first use; latency grows with segment length; hallucination on silence |
| Integration | Behind `IF-004`; gated by push-to-talk; VAD before transcription |
| Rationale | The vocabulary is small and the grammar is constrained, so a small model plus fuzzy matching is sufficient |
| **Status** | **Proposed** — model size selected empirically in `PH-07` |
| Fallback | If latency or accuracy misses, reduce vocabulary further and rely on gesture+context; speech remains an optional channel by design |

---

## 8. OS Automation

### 8.1 Win32 API + pyautogui fallback — Proposed

| Aspect | Detail |
|---|---|
| Purpose | PowerPoint control, media keys, IDE keystrokes, foreground window detection |
| Alternatives | `pywinauto` (UI Automation), `pywin32` directly, AutoHotkey subprocess, OS-level accessibility APIs |
| Advantages | Direct COM for PowerPoint is the most reliable path; media keys are global and simple; `GetForegroundWindow` is trivial |
| Limitations | COM requires the app installed and may block if a modal dialog is open; global media keys affect any player; UI automation breaks on app updates |
| Integration | Each mechanism lives inside one adapter and is hidden by `IF-008`. Foreground verification guards every dispatch. |
| Rationale | Reliability per application matters more than framework uniformity. The adapter boundary allows a different mechanism per app. |
| **Status** | **Proposed** — validated per adapter in `PH-10` |
| Risk | The highest-risk dependency in the project. Mitigation: a spike in `PH-04`, and LibreOffice as a documented alternative target (`OQ-03`). |

---

## 9. Frontend

### 9.1 React + Vite + TypeScript

| Aspect | Detail |
|---|---|
| Purpose | Live dashboard: status, event feed, decision trace, controls, metrics |
| Alternatives | Svelte, Vue, plain HTML + htmx, Electron |
| Advantages | Component model fits the event-driven UI; Vite dev server is fast; TypeScript catches event-shape errors at build time |
| Limitations | Bundler/toolchain overhead for a small app; Node required at build time only |
| Integration | Consumes the REST API and one WebSocket; types generated from the OpenAPI schema where practical |
| Rationale | The dashboard must render a live, correlated event stream. That is a state-management problem React handles conventionally. |
| **Status** | **Selected** |

### 9.2 Charting — Deferred decision

| Option | Trade-off |
|---|---|
| Recharts | Declarative, simple, good for percentile and bar charts |
| Chart.js | Smaller, imperative, widely used |
| Plain SVG/D3 | No dependency, more code |

Not decided at baseline. The choice depends on whether the metrics views need anything beyond bars and lines. Recorded as an implementation-time decision in `PH-11`.

---

## 10. Testing and Tooling

### 10.1 pytest

| Aspect | Detail |
|---|---|
| Purpose | Unit, integration, performance, and end-to-end tests |
| Alternatives | `unittest`, `nose2` |
| Advantages | Fixtures, parametrisation, plugins (`pytest-cov`, `pytest-benchmark`, `pytest-asyncio`); readable failures |
| Limitations | Fixture magic can obscure setup; plugin sprawl |
| Integration | Tests must run without camera, microphone, or GUI (`TR-028`) using recorded fixtures |
| **Status** | **Selected** |

### 10.2 Ruff + mypy

| Aspect | Detail |
|---|---|
| Purpose | Linting, formatting, static typing |
| Alternatives | Flake8 + Black + isort, Pylint |
| Advantages | One fast tool replaces three; mypy catches contract drift at boundaries |
| Limitations | mypy on NumPy/PyTorch can require stubs and ignores |
| **Status** | **Selected** (`PH-02`) |

### 10.3 Git and GitHub

| Aspect | Detail |
|---|---|
| Purpose | Version control, review, CI |
| Alternatives | GitLab, Bitbucket, local-only git |
| Advantages | Free private repos; Actions for CI; pull-request review workflow matches the five-person split |
| Limitations | Binary artefacts must stay out of history; large files require LFS or external storage |
| Integration | Branch-per-task, PR required for `main`, CI runs tests and docs link checks |
| **Status** | **Selected** |
| Repository | `https://github.com/Spandan-2006/Gesture_Detection_ML` |

---

## 11. Explicit Non-Selections

| Technology | Why not |
|---|---|
| Docker | Adds a layer between the process and the camera/microphone on Windows; debugging device access through a container is disproportionate for a research prototype |
| Kubernetes / cloud infra | No deployment requirement |
| Redis / RabbitMQ | In-process event bus is sufficient at this volume |
| Electron | The dashboard is a browser client; packaging adds nothing |
| TensorFlow | Two DL frameworks would fragment the team's model work |
| OpenPose | Heavier, GPU-oriented, no advantage for single-hand landmarks |
| Whisper via cloud API | Violates local-only processing (`NFR-025`) |
| MongoDB | The data is relational and small |
| GraphQL | The query surface is fixed and small; REST is simpler to document and test |

---

## 12. Version Pinning Policy

| Artefact | Pin level | Reason |
|---|---|---|
| Python | Minor version | ABI compatibility with MediaPipe |
| MediaPipe | Exact | Landmark behaviour changes across versions |
| PyTorch | Minor, exact at release | Numerical reproducibility |
| faster-whisper | Minor | Model compatibility |
| FastAPI / Pydantic | Minor | Schema behaviour |
| Frontend build tools | Lockfile | Reproducible builds |
| Everything else | Compatible range | Development flexibility |

**Rule.** Exact pins for components whose behaviour affects reported metrics. A version bump that could change a measured number requires re-running the affected experiments, and that fact is recorded in the experiment log.

---

## 13. Related Documents

- [TRD](../core/TRD.md) §24 — prerequisites and §25 — trade-offs
- [MODEL_COMPARISON](./MODEL_COMPARISON.md) — model-level decisions
- [SECURITY_AND_PRIVACY](./SECURITY_AND_PRIVACY.md) — dependency and privacy implications
- [ADR-0005](../architecture/adr/ADR-0005-sqlite-default.md), [ADR-0006](../architecture/adr/ADR-0006-fastapi-event-bus.md)

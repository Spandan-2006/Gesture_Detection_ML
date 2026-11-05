# Contributing

Thanks for working on the Gesture-Controlled AI Computer Interface. This project has five contributors working in parallel on one pipeline, so a small amount of process prevents a large amount of integration pain.

---

## Before You Start

1. Read [docs/core/PRD.md](docs/core/PRD.md) for scope and requirements.
2. Read [docs/core/TRD.md](docs/core/TRD.md) for the component you are touching.
3. Read [docs/core/IMPLEMENTATION_PLAN.md](docs/core/IMPLEMENTATION_PLAN.md) for your phase and its exit gate.
4. Read [docs/supporting/DEVELOPMENT_GUIDELINES.md](docs/supporting/DEVELOPMENT_GUIDELINES.md) for code, branch, and review rules.

If a task is not covered by a requirement, it is not in scope. Raise it rather than starting it.

---

## Workflow

```
1. Pick a task from your phase in the Implementation Plan
2. Create a short-lived branch:  feat/perception-normalise-landmarks
3. Implement with tests
4. Run lint, types, and tests locally
5. Open a pull request with a linked requirement
6. Address review
7. Squash merge to main
```

**Branch naming**

| Type | Pattern | Example |
|---|---|---|
| Feature | `feat/<area>-<short>` | `feat/fusion-weight-scoring` |
| Fix | `fix/<area>-<short>` | `fix/safety-focus-check` |
| Docs | `docs/<short>` | `docs/adr-cleanup` |
| Experiment | `exp/<short>` | `exp/loso-baseline` |

**Commit messages** follow Conventional Commits with a component scope:

```
feat(recognition): add GRU classifier behind GestureClassifier protocol
fix(safety): re-check foreground before adapter dispatch
docs(core): resolve latency definition conflict
```

Reference requirements where they apply:

```
Refs: FR-027, AC-020
```

---

## Pull Requests

A pull request is ready for review when:

- [ ] It links at least one `FR` or `TR`
- [ ] Tests are added or updated
- [ ] Lint, types, and unit tests pass locally
- [ ] Documentation is updated if a contract, schema, or behaviour changed
- [ ] The diff is under roughly 400 lines, or the size is justified
- [ ] The author has reviewed their own diff

**Reviewers.** One approval is required. Two are required for changes under `src/contracts/`, `src/execution/safety/`, or any database migration.

**Review turnaround.** Within one working day. A stalled review blocks the critical path.

---

## Documentation Changes

Documentation is code. The same review standard applies.

| If you change… | Update… |
|---|---|
| A requirement | `docs/core/PRD.md` |
| A component or its responsibility | `docs/core/TRD.md` |
| An endpoint, event, or payload | `docs/core/API_AND_INTEGRATION.md` |
| A table, column, or constraint | `docs/core/BACKEND_SCHEMA.md` + Alembic migration |
| Phase scope or ordering | `docs/core/IMPLEMENTATION_PLAN.md` |
| An architectural decision | New ADR in `docs/architecture/adr/` |
| A metric definition | `docs/research/EVALUATION_METRICS.md` |

**Conflict rule.** The five documents in `docs/core/` are authoritative. If two disagree, do not silently pick one — record the conflict and its resolution (see `TRD` §30 for precedents).

---

## Testing Requirements

| Change | Required |
|---|---|
| New logic | Unit test |
| New gate rule | Positive and negative test |
| New adapter | Shared adapter harness |
| Bug fix | Regression test that fails before the fix |
| Model change | Harness run with recorded metrics |

No production code merges without a test that would fail if the code were removed.

Tests must run without a camera, microphone, or GUI. Use recorded fixtures.

---

## Code Standards

- Python 3.11/3.12, PEP 8, Ruff formatting, 100-character lines
- Full type annotations on public functions
- Typed contracts at module boundaries
- Structured logging with component ID and correlation ID
- No `except Exception: pass`, no `eval`, no `shell=True`, no `os.system`

Full rules: [Development Guidelines](docs/supporting/DEVELOPMENT_GUIDELINES.md).

---

## What Not to Commit

The following are gitignored and must stay that way:

- Datasets, model weights, logs, and virtual environments
- Secrets, tokens, and `.env` files
- Raw video or audio from any source

A commit containing participant data is a privacy incident, not a mistake to be fixed quietly. Report it.

---

## Reporting Problems

| Problem | Where |
|---|---|
| Bug in the code | GitHub issue with reproduction steps |
| Documentation contradiction | GitHub issue tagged `docs` |
| Scope change | Discussion with the Team Lead before any code |
| Safety concern | Immediately to the Team Lead; do not wait for a scheduled review |
| Privacy incident | Immediately to the Team Lead; follow [Security and Privacy](docs/supporting/SECURITY_AND_PRIVACY.md) §16 |

---

## Definition of Done

1. Merged to `main` through review
2. Tests cover it and pass
3. Lint and types clean
4. Relevant documentation updated
5. A teammate could understand it from the diff and description alone

---

## Code of Conduct

Be direct about code and generous about people. Critique the design, not the author. Resolve disagreements with evidence — a benchmark, a test, or an ADR — not seniority.

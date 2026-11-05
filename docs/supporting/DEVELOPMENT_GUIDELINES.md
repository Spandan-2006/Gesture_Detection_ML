# Development Guidelines

**Project:** Gesture-Controlled AI Computer Interface
**Version:** 1.0.0
**Owner:** Team Lead
**Last Updated:** 2026-11-06

---

## 1. Purpose

These rules exist so that five people can work independently on one system and integrate without renegotiation. Where a rule feels bureaucratic, the alternative is a merge conflict at a milestone gate.

**Enforcement philosophy.** Automated where possible, reviewed where not, documented where neither. A rule that is neither automated nor reviewed does not exist.

---

## 2. Coding Conventions

### 2.1 General

| Rule | Detail |
|---|---|
| Language | Python 3.11/3.12 for application code; TypeScript for the dashboard |
| Style | PEP 8, enforced by Ruff |
| Line length | 100 characters |
| Quotes | Double quotes |
| Formatting | Ruff formatter; no manual alignment |
| Imports | Absolute imports from `src`; no wildcard imports |

### 2.2 Type Hints

| Rule | Detail |
|---|---|
| Public functions | Full annotations required |
| Private helpers | Annotations strongly preferred |
| `Any` | Requires a comment explaining why |
| Return types | Always explicit, including `-> None` |
| Protocols | Used for component boundaries (`IF-001`–`IF-009`) |

**Boundary rule.** Every function that crosses a module boundary accepts and returns a typed contract from `src/contracts/`. No component passes raw dictionaries across its boundary.

### 2.3 Structure

| Rule | Detail |
|---|---|
| Module size | Prefer under 400 lines; split by responsibility, not by line count |
| Function size | Prefer under 50 lines; extract when the purpose stops fitting in the name |
| Nesting | Maximum three levels; use early returns |
| Pure functions | Required for gate rules, scoring, and feature construction |
| Global state | Prohibited except for the event bus and configuration singleton, both injected |

### 2.4 Naming

| Element | Convention | Example |
|---|---|---|
| Modules | `snake_case` | `landmark_extractor.py` |
| Classes | `PascalCase` | `DecisionGate` |
| Functions | `snake_case`, verb-first | `resolve_intent()` |
| Constants | `UPPER_SNAKE` | `MAX_WINDOW_FRAMES` |
| Private | Leading underscore | `_normalise()` |
| Contract types | Domain noun | `GestureObservation` |
| Booleans | `is_` / `has_` / `should_` | `is_stable` |

**Rule.** Names describe intent, not implementation. `process_data` is rejected in review; `normalise_landmarks` is accepted.

---

## 3. Prohibited Patterns

| Pattern | Why | Alternative |
|---|---|---|
| `except Exception: pass` | Hides failure | Catch specifically, log, re-raise or return a typed failure |
| Blocking I/O in an async route | Stalls every client | `run_in_executor` or a worker thread |
| CPU-bound work in the event loop | Same | Worker thread |
| `os.system`, `shell=True`, `eval`, `exec` | Injection | Enumerated actions |
| Mutable default arguments | Shared-state bug | `None` sentinel |
| Bare `print` for diagnostics | Unstructured, unredacted | Logger |
| Wildcard imports | Ambiguous namespace | Explicit imports |
| Circular imports between components | Coupling | Depend on contracts |
| Model inference inside an adapter | Layer violation | Adapters receive authorised commands only |
| Time-dependent gate decisions | Non-reproducible | Inject a clock where needed |

---

## 4. Logging

### 4.1 Structure

Every log record carries:

| Field | Source |
|---|---|
| `timestamp` | UTC, ISO-8601 ms |
| `level` | DEBUG / INFO / WARNING / ERROR / CRITICAL |
| `component` | `C-01`…`C-12` |
| `correlation_id` | When part of an interaction |
| `message` | Human-readable, single line |
| `context` | Structured key-value, redacted |

### 4.2 Levels

| Level | Use |
|---|---|
| DEBUG | Development detail; off in demo |
| INFO | State transitions, action outcomes, startup/shutdown |
| WARNING | Degradation that does not stop the system |
| ERROR | Operation failed; system continues |
| CRITICAL | System cannot continue |

### 4.3 Rules

- Log at boundaries, not inside tight loops.
- Never log raw frames, audio, or full landmark arrays.
- Never log credentials, tokens, or plaintext window titles.
- Every denial is logged at INFO with its code and deciding rule.
- Every unexpected exception is logged at ERROR with a stack trace to the log file, never to the API response.

---

## 5. Exception Handling

| Situation | Pattern |
|---|---|
| Expected failure (device missing, unsupported action) | Return a typed failure object |
| Programming error (contract violated) | Raise; the boundary converts to `E-9003` |
| Third-party call | Wrap, add context, convert to a domain error |
| Retryable failure | Bounded retry with backoff, declared explicitly |
| Shutdown | Catch, release devices, log, exit non-zero |

**Rule.** An exception that crosses a component boundary is a defect. Boundaries return values, not exceptions.

---

## 6. Branching Strategy

Trunk-based with short-lived branches.

| Branch | Purpose | Lifetime |
|---|---|---|
| `main` | Always releasable; protected | Permanent |
| `feat/<area>-<short>` | New capability | ≤ 5 days |
| `fix/<area>-<short>` | Defect | ≤ 2 days |
| `docs/<short>` | Documentation | ≤ 3 days |
| `exp/<short>` | Experiment configuration | ≤ 1 week |

**Rules.**

- No direct commits to `main`.
- Rebase onto `main` before requesting review; avoid merge commits in feature branches.
- A branch that lives longer than a week is split or abandoned.
- `main` must pass lint, types, unit, and contract tests at all times.
- Large binary artefacts never enter git history; history rewriting to remove them is expensive and avoidable.

---

## 7. Commit Conventions

Conventional Commits, scoped by component.

```
<type>(<scope>): <imperative summary>

<body: what and why, wrapped at 72 characters>

<footer: refs, breaking changes>
```

| Type | Use |
|---|---|
| `feat` | New capability |
| `fix` | Defect correction |
| `docs` | Documentation only |
| `test` | Tests only |
| `refactor` | Behaviour-preserving change |
| `perf` | Performance change |
| `chore` | Tooling, dependencies |
| `exp` | Experiment configuration or results |

**Scopes.** `perception`, `recognition`, `fusion`, `intent`, `safety`, `adapters`, `backend`, `dashboard`, `contracts`, `docs`, `ci`.

**Examples.**

```
feat(recognition): add GRU classifier behind GestureClassifier protocol
fix(safety): re-check foreground before adapter dispatch
docs(core): record CF-04 latency definition conflict
```

**Rules.**

- Summary in imperative mood, ≤ 72 characters.
- One logical change per commit.
- The body explains *why*, not *what*; the diff shows what.
- Reference requirement IDs where applicable: `Refs: FR-027, AC-020`.

---

## 8. Pull Request Process

### 8.1 Requirements

| Requirement | Detail |
|---|---|
| Title | Same convention as commits |
| Description | What changed, why, how it was tested, risk |
| Linked requirement | At least one `FR` or `TR` |
| Tests | New or updated tests included |
| Docs | Updated if a contract, schema, or behaviour changed |
| Size | Prefer under 400 changed lines; larger PRs need justification |
| Reviewers | One reviewer minimum; two for `contracts/`, `safety/`, or schema changes |

### 8.2 Review Checklist

| # | Check |
|---|---|
| 1 | Does it satisfy the linked requirement? |
| 2 | Are the contracts respected, or explicitly and consistently changed? |
| 3 | Are failure paths handled, or do they raise across a boundary? |
| 4 | Are there tests that would fail if the change were reverted? |
| 5 | Is anything blocking the event loop or inference thread? |
| 6 | Is any new dependency justified? |
| 7 | Are logs structured and redacted? |
| 8 | Does the documentation still describe reality? |

### 8.3 Self-Review Rule

The author reviews their own diff before requesting review. This catches the majority of obvious issues and respects the reviewer's time.

### 8.4 Merge

- Squash merge to `main` with a conventional commit message.
- Delete the branch after merge.
- No merge without green CI.

---

## 9. Testing Requirements

| Change type | Required tests |
|---|---|
| New function with logic | Unit test |
| New contract field | Contract test + schema test |
| New gate rule | Positive and negative test |
| New adapter | Shared adapter harness |
| Bug fix | Regression test that fails before the fix |
| Model change | Harness run with recorded metrics |
| Performance-sensitive change | Before/after measurement |

**Minimum.** No production code merges without a test that would fail if the code were removed. Exceptions require a reviewer's explicit approval recorded in the PR.

Full strategy: [TESTING_STRATEGY](./TESTING_STRATEGY.md).

---

## 10. Documentation Update Requirements

| Change | Documents to update |
|---|---|
| New or changed requirement | `PRD.md` |
| New component or changed responsibility | `TRD.md` |
| New or changed contract | `API_AND_INTEGRATION.md` |
| Schema change | `BACKEND_SCHEMA.md` + Alembic migration |
| Task reordering or scope change | `IMPLEMENTATION_PLAN.md` |
| New architectural decision | ADR + `SYSTEM_ARCHITECTURE.md` |
| Metric definition change | `EVALUATION_METRICS.md` + `RESEARCH_METHODOLOGY.md` |
| Dependency change | `TECH_STACK.md` |
| Security-relevant change | `SECURITY_AND_PRIVACY.md` |

**Rules.**

- Documentation changes land in the same PR as the code, not a follow-up.
- A change that contradicts a core document requires an explicit recorded resolution, not a silent edit.
- Documentation is reviewed like code.
- Stale documentation is a defect, tracked like any other.

---

## 11. Configuration and Secrets

| Rule | Detail |
|---|---|
| Defaults in `config/default.yaml` | Reviewable in git |
| Overrides via environment | `GESTURE_AI__SECTION__KEY` |
| Secrets never in git | `.env` is ignored; `.env.example` is tracked |
| Validation at load | Invalid configuration prevents startup |
| No magic numbers | Thresholds live in configuration, with defaults documented |

---

## 12. Code Review Culture

| Principle | Practice |
|---|---|
| Critique the code, not the author | "This blocks the event loop" not "you blocked the event loop" |
| Ask questions before asserting | "What happens if the window is empty here?" |
| Approve when it is better, not perfect | Blocking on style that the formatter handles wastes time |
| Disagreement is resolved by evidence | Benchmark, test, or ADR — not seniority |
| Review within one working day | A stalled PR blocks the critical path |

---

## 13. Definition of Done

A change is done when:

1. It is merged to `main` through review.
2. Tests cover it and pass.
3. Lint and types are clean.
4. The relevant documentation is updated.
5. A teammate not involved in the change could understand it from the diff and the PR description.

---

## 14. Related Documents

- [REPOSITORY_STRUCTURE](./REPOSITORY_STRUCTURE.md) — where code goes
- [TESTING_STRATEGY](./TESTING_STRATEGY.md) — what must be tested
- [API_AND_INTEGRATION](../core/API_AND_INTEGRATION.md) — contracts and protocols
- [IMPLEMENTATION_PLAN](../core/IMPLEMENTATION_PLAN.md) §5 — definition of done per phase

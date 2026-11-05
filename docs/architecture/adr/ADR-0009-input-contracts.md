# ADR-0009 — Three Input Contracts for a Fair Model Comparison

**Status:** Accepted
**Date:** 2026-11-06
**Decider:** Team Lead
**Consulted:** Member 3
**Supersedes:** —

---

## Context

The central experiment compares five classifier families. They are not interchangeable: Random Forest and MLP expect fixed-length vectors, while LSTM, GRU, and Transformer expect sequences.

The naive approach — give every model a flattened window — would make the comparison easy but meaningless: it would test only parameter count, not temporal inductive bias. The opposite naive approach — give sequence models raw sequences and classical models hand-crafted statistics — would confound representation with architecture.

The research question also requires a **frame-level baseline** to test whether temporal information helps at all (H1).

## Decision

Define **three explicit input contracts** and assign families to them deliberately.

| Contract | Shape | Consumers | Purpose |
|---|---|---|---|
| `IC-FRAME` | `(126,)` | Frame-level baseline (majority vote over window) | Isolates the value of temporal context |
| `IC-FLAT` | `(3780,)` | Random Forest, MLP | Temporal context without sequential bias |
| `IC-SEQ` | `(30, 126)` | LSTM, GRU, Transformer | Temporal context with sequential bias |

All three derive from the same normalised per-frame features. The difference is arrangement, not information content — except for `IC-FRAME`, which genuinely lacks temporal context.

## Consequences

**Positive**

- The comparison answers three distinct questions: does context help (`IC-FRAME` vs `IC-FLAT`), does sequence structure help (`IC-FLAT` vs `IC-SEQ`), and does architecture matter within `IC-SEQ`.
- Results remain interpretable even if the temporal hypothesis fails.
- A frame-level baseline exists, which many gesture papers omit, making the contribution defensible.

**Negative**

- Random Forest on 3 780 features is a large tabular problem for a small dataset; it may underperform for representation reasons rather than architectural ones. This must be acknowledged when interpreting results.
- Three feature builders must stay consistent, which is additional test surface.
- The comparison is not "all models, same input", so reviewers may challenge its fairness. The response is that equal representation is impossible without distorting one side.

**Neutral**

- The contracts are cheap to compute from the same buffer.

## Alternatives Considered

| Alternative | Rejected because |
|---|---|
| All models on flattened windows | Removes the variable under study |
| All models on sequences (pad classical models with statistics) | Misrepresents what a Random Forest does |
| Classical models on hand-crafted statistics (mean, variance, energy) | Confounds representation with architecture; different information than sequence models receive |
| Only compare within `IC-SEQ` | Cannot test H1 |

## Interpretation Rules

These are binding on the analysis, not advisory.

| Observation | Interpretation |
|---|---|
| `IC-SEQ` beats `IC-FLAT` | Sequential inductive bias contributes |
| `IC-FLAT` beats `IC-FRAME` | Temporal context contributes, regardless of architecture |
| `IC-FRAME` matches window models | Temporal information does not help at this data scale |
| RF underperforms LSTM badly | Must be checked against RF on `IC-FRAME` before attributing to temporal modelling |

## Verification

| Check | Method |
|---|---|
| Shapes exactly match the contract | `TC-010` family |
| All contracts derive from one normalisation | `TC-011` invariance tests |
| Contract mismatch blocks model load | `TC-045` |
| Same harness evaluates all contracts | `TC-091` |

## Review Trigger

- A fourth representation is proposed (for example hand-crafted motion statistics as a separate condition).
- The dataset grows enough that the RF-on-3 780-features concern becomes material.
- A reviewer demonstrates that the chosen assignment systematically disadvantages one family.

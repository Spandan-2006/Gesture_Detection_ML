# Diagram Index

**Version:** 1.0.0
**Owner:** Team Lead

---

## 1. Why Diagrams Are Inline

All Mermaid diagrams live inside the document that owns the subject. This is a deliberate decision.

| Approach | Problem |
|---|---|
| Separate `.mmd` files referenced by docs | Diagrams drift from the text that describes them; reviewers do not see them in the diff context |
| Duplicated diagrams (inline + file) | Two sources of truth; one is always stale |
| Images exported from a tool | Not reviewable in a pull request; binary in git |

**Rule.** A diagram's source and its surrounding prose are edited in the same commit. A standalone `.mmd` file is created only if a diagram is reused verbatim in more than one document, and then the document references the file rather than re-embedding it.

---

## 2. Diagram Register

| ID | Diagram | Type | Location |
|---|---|---|---|
| `D-01` | Proposed solution layers | Flowchart | [PRD](../../core/PRD.md) §5 |
| `D-02` | Command workflow state machine | State diagram | [PRD](../../core/PRD.md) §13.2 |
| `D-03` | UC-01 interaction sequence | Sequence | [PRD](../../core/PRD.md) §12.1 |
| `D-04` | System context | Flowchart (C4 L1) | [TRD](../../core/TRD.md) §3 |
| `D-05` | Container view | Flowchart (C4 L2) | [TRD](../../core/TRD.md) §4 |
| `D-06` | Computer vision pipeline | Sequence | [TRD](../../core/TRD.md) §6 |
| `D-07` | Fusion pipeline | Flowchart | [TRD](../../core/TRD.md) §11 |
| `D-08` | Gate data flow | Flowchart | [TRD](../../core/TRD.md) §12 |
| `D-09` | Adapter routing | Flowchart | [TRD](../../core/TRD.md) §14 |
| `D-10` | Deployment topology | Flowchart | [TRD](../../core/TRD.md) §23.1 |
| `D-11` | Entity relationship | ER diagram | [BACKEND_SCHEMA](../../core/BACKEND_SCHEMA.md) §2.2 |
| `D-12` | Dataset sample lifecycle | State diagram | [BACKEND_SCHEMA](../../core/BACKEND_SCHEMA.md) §10.1 |
| `D-13` | Model version lifecycle | State diagram | [BACKEND_SCHEMA](../../core/BACKEND_SCHEMA.md) §10.2 |
| `D-14` | Experiment lifecycle | State diagram | [BACKEND_SCHEMA](../../core/BACKEND_SCHEMA.md) §10.3 |
| `D-15` | Phase dependency graph | Flowchart | [IMPLEMENTATION_PLAN](../../core/IMPLEMENTATION_PLAN.md) §1.3 |
| `D-16` | Parallel tracks after M3 | Flowchart | [IMPLEMENTATION_PLAN](../../core/IMPLEMENTATION_PLAN.md) §1.4 |
| `D-17` | Event lifecycle | State diagram | [API_AND_INTEGRATION](../../core/API_AND_INTEGRATION.md) §5.4 |
| `D-18` | Dashboard session flow | Sequence | [API_AND_INTEGRATION](../../core/API_AND_INTEGRATION.md) §10.1 |
| `D-19` | C4 context (architecture view) | Flowchart | [SYSTEM_ARCHITECTURE](../SYSTEM_ARCHITECTURE.md) §3 |
| `D-20` | C4 containers (architecture view) | Flowchart | [SYSTEM_ARCHITECTURE](../SYSTEM_ARCHITECTURE.md) §4 |
| `D-21` | Pipeline components | Flowchart (C4 L3) | [SYSTEM_ARCHITECTURE](../SYSTEM_ARCHITECTURE.md) §5 |
| `D-22` | Startup sequence | Sequence | [SYSTEM_ARCHITECTURE](../SYSTEM_ARCHITECTURE.md) §6.1 |
| `D-23` | Multimodal interaction sequence | Sequence | [SYSTEM_ARCHITECTURE](../SYSTEM_ARCHITECTURE.md) §6.2 |
| `D-24` | Denial path | Sequence | [SYSTEM_ARCHITECTURE](../SYSTEM_ARCHITECTURE.md) §6.3 |
| `D-25` | Failure and recovery | State diagram | [SYSTEM_ARCHITECTURE](../SYSTEM_ARCHITECTURE.md) §6.4 |
| `D-26` | Concurrency view | Flowchart | [SYSTEM_ARCHITECTURE](../SYSTEM_ARCHITECTURE.md) §7 |
| `D-27` | Demo deployment | Flowchart | [SYSTEM_ARCHITECTURE](../SYSTEM_ARCHITECTURE.md) §8 |
| `D-28` | Trust boundaries | Flowchart | [SYSTEM_ARCHITECTURE](../SYSTEM_ARCHITECTURE.md) §9 |
| `D-29` | Test pyramid | Flowchart | [TESTING_STRATEGY](../../supporting/TESTING_STRATEGY.md) §2 |
| `D-30` | Experiment design matrix | Flowchart | [EXPERIMENT_DESIGN](../../research/EXPERIMENT_DESIGN.md) §3 |

---

## 3. Diagram Conventions

| Rule | Detail |
|---|---|
| Node labels | Always quoted: `A["Human readable label"]` |
| Line breaks in labels | `<br/>` |
| Identifier prefixes | Component IDs where a diagram shows components (`C-06`) |
| Direction | Left-to-right or top-down; never mixed in one diagram |
| Colour | Not used to carry meaning; state is in the label |
| Size | If a diagram exceeds roughly 20 nodes, split it |
| Ambiguous edges | Label every edge whose meaning is not obvious, for example `-.->|"events"|` |

### 3.1 Why Labels Are Quoted

Unquoted Mermaid labels break on parentheses, colons, slashes, and ampersands. Quoting every label removes a class of rendering failure that only appears in the browser, not in the editor.

---

## 4. Validation

| Check | Tool | When |
|---|---|---|
| Syntax parses | Mermaid CLI or a render check in CI | Every pull request |
| No unquoted special characters | Review | Every pull request |
| Diagram matches described behaviour | Review | Every change that touches the described flow |
| No duplicate diagram with divergent content | Review | Every pull request |

**Rule.** A diagram that contradicts the prose is a defect. When behaviour changes, the diagram changes in the same commit.

---

## 5. Related Documents

- [SYSTEM_ARCHITECTURE](../SYSTEM_ARCHITECTURE.md)
- [TRD](../../core/TRD.md)
- [DEVELOPMENT_GUIDELINES](../../supporting/DEVELOPMENT_GUIDELINES.md) §10

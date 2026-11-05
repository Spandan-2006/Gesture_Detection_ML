# ADR-0001 — Modular Monolith Over Microservices

**Status:** Accepted
**Date:** 2026-11-06
**Decider:** Team Lead
**Consulted:** All members
**Supersedes:** —

---

## Context

The system has twelve logical components (perception, recognition, fusion, intent, safety, adapters, backend, dashboard) with clearly different responsibilities. That decomposition invites a microservice architecture: one service per stage, communicating over a network.

The deployment reality is the opposite. The entire system runs on one workstation for one user. The end-to-end latency budget is 400 ms at p50. A 30-frame window already consumes ~1000 ms before inference begins, so any avoidable latency is costly.

The team is five part-time contributors. Every additional deployment unit adds configuration, startup ordering, observability, and failure-mode work that does not advance the research question.

## Decision

Build a **modular monolith**: one Python process containing all pipeline components, with strict module boundaries enforced in code, plus one browser client for the dashboard.

Boundaries are enforced by:

1. A shared `src/contracts/` package containing DTOs and protocols.
2. A rule that no module imports another component's concrete implementation — only its protocol.
3. An import-graph test in CI that fails on cross-stage backward imports.

## Consequences

**Positive**

- No network hop in the interaction path; latency budget is spent on computation, not transport.
- One process to start, observe, and debug. A failed demo has one log file.
- Shared memory avoids serialising landmark arrays.
- Testing is straightforward: components are plain objects with injectable dependencies.

**Negative**

- A crash takes everything down. Mitigated by fail-closed design and subsystem health reporting, not by process isolation.
- Modules can bypass boundaries if reviewers are inattentive. Mitigated by the import-graph test.
- One language runtime must host CPU-heavy inference and an async web server. Mitigated by the threading model in [TRD](../../core/TRD.md) §18.

**Neutral**

- The dashboard is already a separate process because it is a browser client.

## Alternatives Considered

| Alternative | Rejected because |
|---|---|
| Microservices per stage | Network latency in the critical path; five-person team cannot operate it |
| Two processes (pipeline + backend) | Requires IPC for landmark-rate data; adds complexity without fault-isolation benefit |
| Serverless | No persistent camera or microphone access |
| Separate inference server (e.g. TorchServe) | Adds a hop and a deployment unit for a model that runs in 15 ms |

## Verification

| Check | Method |
|---|---|
| No cross-stage backward imports | CI import-graph test (`TR-026`) |
| Contracts package has no internal dependencies | Dependency test |
| Modules testable in isolation | Unit tests with fakes |

## Review Trigger

Revisit if any of the following becomes true:

- More than one user or machine must be served concurrently.
- A component must run on a different OS (for example a Linux inference host).
- A crash in one component must not affect others as a hard requirement.
- Inference latency grows beyond the point where CPU contention with the API loop matters.

# ADR-0006 — FastAPI with an In-Process Event Bus

**Status:** Accepted
**Date:** 2026-11-06
**Decider:** Team Lead
**Consulted:** Member 4, Member 5
**Supersedes:** —

---

## Context

Two communication needs exist:

1. **Observation.** Every pipeline stage must publish events to the dashboard, the persistence writer, and the metrics aggregator.
2. **Control.** The dashboard must start, stop, arm, configure, and stop the system.

Options for observation include an external message broker, an in-process publish/subscribe hub, or direct calls to each consumer.

A broker (Redis, RabbitMQ, Kafka) would be appropriate if multiple processes or machines needed the stream. None do.

## Decision

1. **FastAPI + Uvicorn** for HTTP and WebSocket.
2. **An in-process event bus** with typed envelopes, bounded per-subscriber queues, and drop-oldest overflow.
3. **No external broker** in v1.

The bus is the single mechanism for observation. Persistence and the dashboard are both subscribers of the same stream, so the dashboard shows exactly what is recorded ([API_AND_INTEGRATION](../../core/API_AND_INTEGRATION.md) §2).

## Consequences

**Positive**

- One envelope definition serves WebSocket clients, persistence, and metrics, eliminating a common class of inconsistency.
- No broker to install, monitor, or explain.
- Publishing never blocks the pipeline: a slow consumer loses old events, not new frames.
- Async HTTP and WebSocket in one framework reduces the moving parts.

**Negative**

- Events are lost on process restart; there is no replay log. Mitigated by REST resynchronisation on reconnect.
- Only in-process subscribers are possible. A future external consumer would need the bus promoted to a broker.
- FastAPI's async model makes blocking calls catastrophic. Requires discipline and review.

**Neutral**

- The dashboard is a remote client over WebSocket, which is the same as it would be with a broker.

## Alternatives Considered

| Alternative | Rejected because |
|---|---|
| Redis Pub/Sub | External dependency, no replay, added operational surface |
| RabbitMQ / Kafka | Massively disproportionate for one process and tens of events per second |
| Direct callbacks from each stage to each consumer | Couples every stage to every consumer; order becomes implicit |
| Server-Sent Events instead of WebSocket | Simpler one-way, but control commands still need HTTP; WebSocket keeps one live channel for two-way if needed |
| Flask + flask-sock | Weaker async story; blocking risk is higher |

## Implementation Rules

| Rule | Reason |
|---|---|
| No CPU-bound work inside a route | A blocked loop stalls all clients |
| Publishing is non-blocking | The pipeline must never wait for a consumer |
| Per-client bounded queue (256) | A slow browser must not degrade the system |
| Unknown event types are ignored by consumers | Forward compatibility |
| Persistence batches on 250 ms or 50 events | Database I/O stays off the inference path |

## Verification

| Check | Method |
|---|---|
| Slow client does not affect others | `TC-057`, multi-client load test |
| Publish never blocks | Concurrency test with a stalled subscriber |
| Event schema conformance | `TC-040` |
| Correlation preserved | `TC-042` |
| No event loop stall under load | Profiling during `TC-062` |

## Review Trigger

- A second process or machine must consume events.
- Event volume exceeds what one process can fan out (not expected; interactions are capped at 12 per minute).
- Durable event replay becomes a requirement.

# ADR-0005 — SQLite Default with a PostgreSQL-Portable Schema

**Status:** Accepted
**Date:** 2026-11-06
**Decider:** Member 4
**Consulted:** Team Lead, Member 3
**Supersedes:** —

---

## Context

The project needs durable storage for configuration, models, experiments, applications, mappings, and interaction outcomes. Volume is low: at most 12 actions per minute by design, tens of thousands of context rows per session, and a dataset index in the thousands.

The project brief suggests PostgreSQL. The operational reality is that every demonstration runs on a laptop, every teammate sets up their own machine, and a dead database service during a live demo is a preventable failure.

## Decision

Use **SQLite** as the default store, with a schema written to be **PostgreSQL-portable** through SQLAlchemy 2.x and Alembic.

Portability rules:

| Rule | Reason |
|---|---|
| No `ENUM` types | Not portable; use `VARCHAR` + `CHECK` |
| No `SERIAL` / `AUTOINCREMENT` PKs | Use UUID strings |
| No arrays; JSON stored as `TEXT` | Application-level validation, dialect-neutral |
| No query depends on `JSONB` operators | PostgreSQL may use `JSONB`; nothing may rely on it |
| Timezone-aware UTC timestamps only | Consistent ordering across dialects |
| Partial unique indexes allowed | Supported by both |

## Consequences

**Positive**

- Zero setup: the application creates the database on first run.
- Backup is a file copy.
- Demonstrations cannot fail because a database service is not running.
- Schema portability is proven by construction, not aspiration.

**Negative**

- One writer at a time. Batched writes (250 ms interval) mitigate this; at 12 actions per minute it is not a constraint.
- No network access to the database, so a second machine cannot read it live.
- Analytical performance degrades beyond a few million rows; the retention policy prevents that.

**Neutral**

- PostgreSQL remains available as a connection-string change plus a migration run.

## Alternatives Considered

| Alternative | Rejected because |
|---|---|
| PostgreSQL only | Service dependency for every demo and every new machine |
| JSON files on disk | No constraints, no joins, no concurrent-write safety, painful analytics |
| DuckDB | Excellent for analytics, weaker for transactional config updates; would be a second system |
| MongoDB | Data is relational; no benefit |

## Verification

| Check | Method |
|---|---|
| Migration runs on SQLite | CI with a fresh database |
| Migration runs on PostgreSQL | CI job against a containerised instance |
| No SQLite-only SQL | Dialect lint + review |
| Write latency does not affect inference | Profiling test (`TR-024`) |
| One active configuration enforced | Partial unique index test |
| One active model version per model | Partial unique index test |

## Migration Trigger

Adopt PostgreSQL when any of the following holds:

- More than one process writes concurrently.
- Sustained writes exceed roughly 50 per second.
- A multi-machine deployment is required.
- Analytical queries exceed acceptable runtime at the retained data volume.

Adoption is a connection-string change plus `alembic upgrade head`, and requires no application code change if the portability rules were respected.

## Review Trigger

Re-evaluate at each milestone gate if the volume estimates in [BACKEND_SCHEMA](../../core/BACKEND_SCHEMA.md) §7 are exceeded by more than an order of magnitude.

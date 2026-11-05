# ADR-0007 — Adapter Allowlist and Enumerated Actions

**Status:** Accepted
**Date:** 2026-11-06
**Decider:** Member 4
**Consulted:** Team Lead
**Supersedes:** —

---

## Context

The system sends keystrokes, media keys, and COM commands to other applications. This is the highest-impact capability in the project: a wrong keystroke can close an unsaved document or move a slide during a critical moment.

Two failure sources matter:

1. The system decides to act on the wrong intent.
2. The system acts correctly but on the wrong target, or with an unintended command.

The decision gate addresses the first. This ADR addresses the second and the general question of what the system is *allowed* to do at all.

## Decision

1. **Every adapter action is enumerated.** Actions are declared in `supported_actions` with a class (`safe`, `reversible`, `destructive`). There is no free-form command path.
2. **Only allowlisted adapters execute.** `applications.is_enabled` is the allowlist; the gate denies anything else with `E-7009`.
3. **No dynamic command construction.** No `os.system`, no `shell=True`, no `eval`/`exec`, no string interpolation into automation calls.
4. **Focus is verified immediately before dispatch.** Context is re-read after the gate approves and before the adapter acts.
5. **Destructive actions are excluded from default mappings** and require per-invocation confirmation.

## Consequences

**Positive**

- The set of possible actions is finite, reviewable, and testable. A security review can enumerate it.
- Injection through transcripts, window titles, or configuration becomes structurally impossible.
- A new application is added by declaring its actions, not by granting general input control.
- Wrong-window delivery is caught by a check that is independent of the model.

**Negative**

- Adapters are more work than a generic "send keys" helper. That work is the safety feature.
- Users cannot script arbitrary commands through the system. This is intended.
- Focus verification adds latency (one context read, ~1 ms) to every dispatch.

**Neutral**

- The allowlist is configuration, so enabling a new application does not require a code change.

## Alternatives Considered

| Alternative | Rejected because |
|---|---|
| Free-form command mapping (user supplies keystrokes) | Unbounded surface; injection risk; cannot be reviewed |
| Trust the model's confidence instead of an allowlist | Model confidence is not an authorisation signal |
| OS-level sandboxing of the whole app | Would prevent the app from performing its function |
| Require elevation for all actions | Contradicts the standard-user requirement and adds friction without reducing the specific risk |

## Implementation Rules

| Rule | Enforcement |
|---|---|
| Adapter exposes a fixed action list | Adapter harness test |
| Unknown action returns `E-8002`, never raises | Adapter harness test |
| Pipeline stages never call adapters directly | Import-graph test |
| Focus re-check before dispatch | `TC-083` |
| Static analysis bans shell/eval patterns | CI lint rule |
| Each adapter declares idempotency | Adapter harness test |

## Verification

| Check | Method |
|---|---|
| Unlisted adapter denied | `TC-082` |
| Shell metacharacters rejected | `TC-087` |
| Focus change aborts dispatch | `TC-083` |
| Destructive requires confirmation | `TC-031` |
| New adapter requires no pipeline change | `TC-070` |

## Known Limitation

Process-name matching can be spoofed by a process that renames itself. Context is a **safety aid, not an authentication mechanism**. It prevents accidents between legitimate applications; it does not defend against an attacker who already has code execution on the machine. This is stated rather than implied away.

## Review Trigger

- A user requires a free-form command capability.
- An adapter needs an action that cannot be expressed as an enumerated command.
- A security review identifies a path from untrusted data to an automation call.

# ADR-0004 — Local-Only Processing and No Raw Media Retention

**Status:** Accepted
**Date:** 2026-11-06
**Decider:** Team Lead
**Consulted:** Member 4, Member 2
**Supersedes:** —

---

## Context

The system observes through a camera and microphone and controls the user's computer. Both capabilities are invasive by nature. Options range from full local processing to cloud inference for speech and vision.

Cloud processing would improve accuracy and reduce local compute requirements. It would also mean continuous video or audio leaving the machine, which is unacceptable for a system that may be used in a home or lecture setting and which must recruit research participants.

## Decision

1. **All processing is local.** No inference, transcription, or recognition request leaves the machine during operation.
2. **Raw video and audio are never persisted** in default configuration.
3. Model downloads happen once, during explicit setup, not at runtime.
4. Any future network egress requires a new ADR and a privacy review.

## Consequences

**Positive**

- Consent conversations are dramatically simpler: participants are told no video or audio is retained.
- No third-party data processor, no data processing agreement, no cross-border transfer question.
- Works offline, which matters for demonstrations in rooms with unreliable Wi-Fi.
- Removes an entire class of threats (interception, provider retention, account compromise).

**Negative**

- Speech accuracy is limited to what local models achieve on this hardware. The mitigation is a closed command vocabulary and fuzzy matching.
- Local compute must carry the full load, which constrains model size (`NFR-061`).
- Model weights must be downloaded and stored locally, consuming disk.

**Neutral**

- The dashboard is served over loopback, so "network" traffic exists but never leaves the host.

## Alternatives Considered

| Alternative | Rejected because |
|---|---|
| Cloud speech-to-text | Sends user speech off-device; contradicts the privacy position and complicates consent |
| Cloud vision inference | Highest privacy cost for the least benefit; landmarks are already cheap to compute locally |
| Hybrid with opt-in cloud fallback | Creates a configuration in which data leaves the device; the decision boundary becomes unclear |
| Local processing with optional telemetry | Telemetry would need its own consent and threat analysis; no v1 benefit |

## Implementation Rules

| Rule | Enforcement |
|---|---|
| No HTTP client in the runtime path | Dependency test; network monitor during a session |
| No raw frame or audio writes | Code review; file-system assertion test |
| Audio buffers zeroed after transcription | `TR-005` unit test |
| Redaction in the logger, not at call sites | `TR-004` review |

**Honest limitation.** Zeroing a buffer in a managed runtime is best-effort, not cryptographic erasure. The documentation must not claim otherwise.

## Verification

| Check | Method |
|---|---|
| No egress during a session | `TC-089` + manual monitor |
| No media files written | Security test |
| Speech model loaded from local path | Startup test with networking disabled |

## Review Trigger

- A requirement emerges for cross-device operation that cannot be met locally.
- Local speech accuracy proves insufficient even with a closed vocabulary.
- Institutional policy explicitly requires on-premises central processing rather than per-machine processing.

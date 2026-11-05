# Security and Privacy

**Project:** Gesture-Controlled AI Computer Interface
**Version:** 1.0.0
**Owner:** Member 4 (Execution Safety) with Team Lead
**Last Updated:** 2026-11-06

---

## 1. Scope and Risk Position

This system has two unusual properties that shape its entire security posture:

1. **It observes the user continuously** through camera and microphone.
2. **It acts on the user's computer** with the user's own privileges.

Both are necessary for the product to exist. Neither can be eliminated, so both must be bounded.

**Risk position.** The system is a local research prototype operated by its owner on their own machine. The primary risks are therefore *self-inflicted* (an unintended action destroying work) and *privacy* (inadvertent capture or retention), not remote compromise. Remote attack surface is deliberately minimal, and that minimalism is maintained rather than assumed.

**What this document is not.** It is not a claim of certification, compliance, or formal security audit. It is an engineering baseline proportionate to a research prototype.

---

## 2. Threat Model

| ID | Threat | Actor | Likelihood | Impact | Primary control |
|---|---|---|---|---|---|
| `T-01` | Unintended action destroys work | System itself | High | High | Decision gate, action classes, confirmation |
| `T-02` | Command sent to the wrong application | System itself | High | High | Foreground verification before dispatch |
| `T-03` | Camera captures bystanders or a private space | User | Medium | High | Explicit activation, visible indicator, no storage |
| `T-04` | Audio captured beyond the intended utterance | User | Medium | Medium | Push-to-talk gating, VAD, 5 s cap, no storage |
| `T-05` | Dataset leaks identifying information | Team | Medium | High | Landmarks only, pseudonyms, no video |
| `T-06` | Credential committed to the repository | Team | Medium | High | `.gitignore`, environment-only secrets, review |
| `T-07` | Another host on the LAN controls the API | Local network | Low | High | Loopback binding, optional token |
| `T-08` | Malicious input reaches an OS command | Attacker or malformed data | Low | Critical | Enumerated actions, no string interpolation |
| `T-09` | Model file replaced with a malicious artefact | Local attacker | Low | High | Checksums, allowlisted model root |
| `T-10` | Path traversal reads or overwrites arbitrary files | Malformed input | Low | High | Allowlisted roots |
| `T-11` | Stale context causes an action in the wrong window | Race condition | High | High | Re-verification immediately before dispatch |
| `T-12` | Bystander inference from hand kinematics | Researcher | Low | Medium | Pseudonymisation, aggregate reporting |

**Out of scope.** Remote code execution via network, supply-chain compromise of PyPI packages, physical access to an unlocked machine, and OS-level privilege escalation. These are properties of the host environment, not of this application, and are not mitigated here beyond dependency pinning.

---

## 3. Camera and Microphone Permissions

| Requirement | Implementation |
|---|---|
| Explicit user initiation | Devices open only on `POST /system/start` with the corresponding flag |
| Per-session authorisation | Closing the session releases the devices |
| Visible indicator | Persistent UI badge and preview window while capture is active |
| No silent restart | Automatic reconnect applies only within an active session |
| OS-level permission | Respect the Windows camera/microphone privacy settings; do not attempt to bypass |
| Device enumeration | List available devices; never open a device not selected by the user |

**Rule.** Starting the system with `enable_speech: true` yet no microphone permission SHALL fail with `E-4001` and a human-readable explanation. It SHALL NOT silently continue pretending speech is available.

---

## 4. Explicit Activation

Three layers of activation, each addressing a different failure:

| Layer | Mechanism | Prevents |
|---|---|---|
| Session | `start` is an explicit user command | Always-on surveillance |
| State | `ARMED` required before any gesture is considered | Accidental commands while configuring |
| Command | Optional wake pose or push-to-talk | Commands during ordinary movement |

**Default configuration.** `wake_gesture` with an 800 ms hold. An always-on mode exists for research comparison and is labelled `research_unsafe` in configuration to make its risk explicit.

**Rule.** The system SHALL NOT persist an "always armed" preference across sessions. Arming is per session, every time.

---

## 5. Local vs Remote Processing

| Data | Processing location | Leaves the machine |
|---|---|---|
| Video frames | Local | No |
| Landmarks | Local | No |
| Audio | Local | No |
| Transcripts | Local | No |
| Context (process name, title hash) | Local | No |
| Interaction logs | Local | No |
| Dataset | Local | No |
| Model inference | Local | No |
| Dashboard traffic | Loopback | No |

**Default egress policy: none.** The application makes no outbound network requests during normal operation. Dependency installation is a build-time activity, not a runtime one.

**Verification.** `TC-089` and a manual check with a network monitor confirm no egress during a 10-minute session. Any future feature requiring egress requires an ADR and a privacy review.

**Model downloads.** The speech model is downloaded once, at setup, not at runtime. The download step is explicit, documented, and separate from the application loop.

---

## 6. Audio and Video Retention

| Data | In memory | On disk | Configurable |
|---|---|---|---|
| Video frames | Transient (ring buffer, ≤ 60 frames) | Never | Only for a consented dataset-capture tool, never the runtime |
| Audio | Transient (≤ 5 s) | Never | No |
| Landmarks | Transient (≤ 60 frames) | Only for consented dataset capture | Dataset capture only |
| Transcripts | Event payload | Logged in redacted form | Yes, default off for raw text at INFO |
| Interaction traces | Event bus | JSONL for 90 days | Yes |

**Zeroing.** Audio buffers are overwritten after transcription (`TR-005`). This is a best-effort control in a managed runtime, not a guarantee; the report must not claim cryptographic erasure.

**What is never collected.** No screenshots, no full-frame video, no continuous audio, no keystroke content. The system sends keystrokes; it does not record them.

---

## 7. Dataset Consent

| Element | Requirement |
|---|---|
| Consent form | Written or digital, versioned, retained separately from the data |
| Stated purpose | Research on gesture recognition and multimodal intent |
| Scope | What is collected (landmarks, metadata), what is not (video, audio, identity) |
| Voluntary | Participation refusable at any point without consequence |
| Withdrawal | At any time, no reason required, honoured within a documented interval |
| Duration | Stated retention period |
| Contact | A named responsible person |
| Guardian consent | Required for minors; minors excluded by default |

**Withdrawal procedure** (from [BACKEND_SCHEMA](../core/BACKEND_SCHEMA.md) §11):

1. Delete the participant's files under `datasets/`.
2. Mark their `gestures` rows as rejected with reason `consent_withdrawn`.
3. Set `consent_withdrawn_at`.
4. Flag affected dataset releases.
5. Annotate affected experiment results rather than deleting them, because deleting published results would falsify the record.
6. Record the action in the audit log.

**Honest limitation.** Hand kinematics may be weakly identifying. The mitigation is pseudonymisation and aggregate reporting, not a claim of anonymity.

---

## 8. Credential Management

| Rule | Detail |
|---|---|
| No secrets in the repository | Enforced by `.gitignore` and review |
| Environment variables only | Prefixed `GESTURE_AI__` |
| No secrets in the database | Configuration rows must never contain tokens |
| No secrets in logs | Logger redacts known-sensitive keys |
| `.env.example` contains keys only | Placeholder values, never real ones |
| Token generation | Generated locally, not shared in chat or documentation |

**Pre-commit check.** A scan for high-entropy strings and known token prefixes runs before merge. Any hit blocks the merge until cleared.

---

## 9. API Validation and Network Exposure

| Control | Implementation |
|---|---|
| Loopback binding | `127.0.0.1` default (`TR-001`) |
| Explicit opt-in for LAN | `bind_public: true` plus a token; both required |
| Schema validation | Every request validated; unknown keys rejected |
| Field bounds | Declared ranges; out-of-range rejected |
| Path safety | All filesystem parameters resolved and checked against allowlisted roots |
| Rate limiting | Control endpoints limited to prevent accidental command storms |
| CORS | Restricted to the dashboard origin |
| Error detail | No stack traces or internal paths in responses |

**Why loopback matters.** The API can start the pipeline and issue control commands. Exposed on a LAN, it becomes a remote-control surface for anyone on the network. The default must therefore be closed, and opening it must be a deliberate, two-part action.

---

## 10. OS Command Authorisation

The most dangerous capability in the system. Four independent controls apply.

| Control | Implementation |
|---|---|
| Enumerated actions | Adapters declare a fixed action list. No dynamic command construction. |
| No string interpolation | Commands are dictionary lookups, never formatted strings (`TR-002`) |
| Allowlist | Only adapters with `is_enabled = true` may execute (`E-7009`) |
| Decision gate | Thirteen ordered rules, any of which can deny (`FR-060`) |

**Specifically prohibited.**

| Prohibited | Reason |
|---|---|
| `os.system(...)` with any variable content | Shell injection |
| `subprocess` with `shell=True` | Shell injection |
| `eval` / `exec` on configuration or model output | Arbitrary code execution |
| Sending keystrokes derived from transcript text | User speech must never become literal input |
| Sending keystrokes derived from window titles | Untrusted content |
| Free-text command entry mapped to automation | Unbounded surface |

**Verification.** Static analysis in CI flags `os.system`, `shell=True`, `eval`, and `exec`. Any exception requires an ADR.

---

## 11. Application Allowlisting

| Rule | Detail |
|---|---|
| Default allowlist | PowerPoint, media player, IDE only |
| Unknown application | Context resolves to `unknown`; gate denies (`E-5002`, `E-7011`) |
| Disabling an application | Removes its intents from resolution immediately |
| Adapter health failure | Removes its intents; no dead mappings |
| Process-name spoofing | Accepted risk: a process can rename itself. Context is a safety aid, not an authentication mechanism. |

**Stated limitation.** Process-name matching is not a security boundary. It prevents accidents, not a determined local attacker who already has code execution. This is documented rather than glossed over.

---

## 12. Emergency Stop

| Property | Requirement |
|---|---|
| Availability | Global hotkey, REST endpoint, dashboard control |
| Latency | Halt dispatch within 100 ms (`FR-006`, `AC-020`) |
| Check points | Before authorisation and again immediately before dispatch (`TR-019`) |
| State after | `PAUSED`, not `IDLE`; requires explicit re-arm (`FR-008`) |
| Clearing | Requires `confirm: true`; no keyboard shortcut clears it |
| Persistence | The flag survives configuration reloads within a session |
| Visibility | Persistent banner until re-armed |

**Rule.** Emergency stop is never behind a menu, never requires confirmation to engage, and never has a countdown.

---

## 13. Prevention of Unintended Destructive Actions

| Control | Detail |
|---|---|
| Action classification | Every action is `safe`, `reversible`, or `destructive` |
| Default mappings | Contain no destructive actions |
| Confirmation | Destructive actions require explicit confirmation per invocation |
| Dry-run mode | Full pipeline, no side effects (`FR-064`) |
| Cooldown | Prevents rapid repeated execution |
| Rate limit | Caps actions per minute |
| Focus verification | Prevents delivery to the wrong window |
| Reversibility preference | Where an action has an inverse, prefer it (for example mute rather than set volume to zero) |

**Design rule.** If an action cannot be undone and cannot be confirmed, it is not implemented.

---

## 14. Logging and Redaction

| Data | Logged as |
|---|---|
| Window title | SHA-256 hash only (`TR-004`) |
| Transcript at `INFO` | Redacted; full text only at `DEBUG`, which is off by default |
| File paths | Relative to allowlisted root |
| Participant identifiers | Pseudonymous codes |
| Credentials | Never |
| Model input | Counts and shapes, not values |

**Rule.** Redaction is implemented in the logger, not at each call site. Per-call-site redaction is forgotten eventually.

---

## 15. Dependency and Supply Chain

| Control | Detail |
|---|---|
| Approved versions | Pinned in `pyproject.toml` and the lockfile |
| Review on upgrade | A dependency bump is a reviewed change, not a background action |
| No unvetted packages | A new dependency requires justification in the pull request |
| Model artefact checksums | Verified at load (`model_versions.artefact_checksum`) |
| Download at setup only | No runtime model fetching |

**Stated residual risk.** A compromised PyPI package is not mitigated beyond pinning and review. This is accepted for a research prototype and recorded here rather than implied away.

---

## 16. Incident Response

| Situation | Immediate action |
|---|---|
| Unintended destructive action | Emergency stop; assess damage; disable the mapping; record the incident |
| Participant withdraws | Execute the withdrawal procedure immediately |
| Credential exposed | Rotate; purge history; record |
| Privacy breach | Stop collection; notify the responsible person; document |
| Demo failure with side effects | Switch to dry-run for the remainder of the session |

Incidents are recorded in `docs/reports/` with date, cause, impact, and corrective action. An incident without a corrective action is not closed.

---

## 17. Security Testing

| Test | Reference |
|---|---|
| No network egress during a session | `TC-089`, manual monitor |
| API unreachable from LAN by default | `TC-089` |
| Shell metacharacter injection rejected | `TC-087` |
| Path traversal rejected | `TC-088` |
| Emergency stop latency | `TC-065`, `TC-085` |
| Unlisted adapter denied | `TC-082` |
| Focus mismatch aborts | `TC-083` |
| Destructive action requires confirmation | `TC-081`, `TC-031` |
| No raw media written | File-system assertion during a session |

Full suite: [TESTING_STRATEGY](./TESTING_STRATEGY.md) §11.

---

## 18. Residual Risks

Accepted, documented, and not silently carried.

| Risk | Why accepted | Compensating control |
|---|---|---|
| Process-name spoofing | Requires prior local code execution | Context is a safety aid; gate still applies |
| Audio zeroing is best-effort | Managed runtime | Buffers are short-lived and never written |
| Small participant pool may be re-identifiable in combination | Research necessity | Pseudonyms; aggregate publication |
| Dependency compromise | Cannot be fully mitigated | Pinning and review |
| A user can disable safety controls deliberately | It is their machine | Defaults are safe; changes are logged and audited |
| Model can be confidently wrong | Inherent | Gate thresholds and abstention |

---

## 19. Related Documents

- [TRD](../core/TRD.md) §22 — security requirements `TR-001`–`TR-009`
- [BACKEND_SCHEMA](../core/BACKEND_SCHEMA.md) §11 — retention and withdrawal
- [DATASET_SPECIFICATION](./DATASET_SPECIFICATION.md) §11 — consent and data minimisation
- [TESTING_STRATEGY](./TESTING_STRATEGY.md) §11 — safety tests
- [ADR-0007](../architecture/adr/ADR-0007-adapter-allowlist.md) — allowlist decision

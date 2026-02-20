## Agentic Architecture & Modular Skills Design (v1)

Status: Target design (builds on `docs/agentic_architecture_v0.md`)

---

# 1. v1 Goals

v1 keeps the v0 architecture shape and hardens system guarantees around:

- Reproducibility
- Operational reliability
- Abuse resistance
- Auditability and transparency

---

# 2. Deterministic Runtime Contract

- Pin container image digest, runtime versions, and model versions
- Fix locale, timezone, and numeric precision defaults
- Persist every run input and parameter set
- Store a `reproducibility_fingerprint` for each run

---

# 3. Skill Execution Contract (Required for Every Skill)

Each mutating skill must define:

- `idempotency_key`
- Retry policy (`max_attempts`, backoff, failure class)
- Timeout and cancellation behavior
- Compensating action or dead-letter route
- Required emitted evidence events

---

# 4. Queueing, Ordering, and Backpressure

- At-least-once delivery support for webhooks
- Deduplication on `message_id` + source
- Dead-letter queue for poison messages
- Worker concurrency limits per skill category
- Backpressure policy for surge windows

---

# 5. State Machines for Critical Entities

## Submission

`received -> pending -> canonicalized -> clustered -> published | disputed`

## Candidate

`generated -> validated -> accepted | rejected`

## Cycle

`scheduled -> open -> closed -> tallied -> published`

## Dispute

`opened -> queued -> rerun_requested -> resolved`

---

# 6. Evidence Integrity Upgrades

- Add signature metadata: `signing_key_id`, `signature`
- Run nightly chain verification with alerting
- Anchor Merkle root externally on daily or weekly cadence
- Generate immutable audit report export per cycle

---

# 7. Sybil and Abuse Defense Upgrades

- Progressive trust tiers for new accounts
- Behavioral anomaly scoring (velocity, duplication, burst patterns)
- Quarantine review workflow with evidence-backed reason codes
- Measure false-positive and false-negative rates

---

# 8. Observability and SLOs

- End-to-end latency SLOs (`ingest -> canonicalize -> cluster -> publish`)
- Error budget for skill failures
- Cost SLO per cycle and per active user
- Drift alerts for canonicalization and clustering instability

---

# 9. v1 Additional Skills (Planned)

38. `Queue.DedupeAndAck`  
39. `Queue.DeadLetterReplay`  
40. `Run.ReproducibilityFingerprint`  
41. `Evidence.SignEvent`  
42. `Evidence.VerifyNightly`  
43. `Risk.ScoreBehavior`  
44. `Ops.EmitSLOMetrics`

---

# 10. Threat Model Snapshot

| Threat | Primary control | Detection signal | Residual risk |
| --- | --- | --- | --- |
| Duplicate webhook replay | `Queue.DedupeAndAck` | Rising dedupe ratio | Medium |
| Coordinated sybil voting | Trust tiers + anomaly scoring | Vote burst by related accounts | Medium |
| Silent operator manipulation | Rerun-only dispute policy + evidence signatures | Hash/signature mismatch | Low |
| LLM output drift | Version pinning + variance checks | Instability flag trend | Medium |
| PII leakage to LLM | `Privacy.RedactionGuard` + text-only payload | Redaction violations | Low |

---

# 11. Design Intent

Collective Will v1 remains:

- Transparent
- Reproducible
- Privacy-preserving
- Deterministic where possible
- Explicit about uncertainty
- Resistant to silent manipulation

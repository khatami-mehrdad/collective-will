## Agentic Architecture & Modular Skills Design

Version: v0 baseline (accepted) + v1 hardening plan

---

# 1. Architectural Philosophy (v0)

Collective Will follows a **small orchestration core + modular skills**
architecture.

## Core principles

- Minimal orchestration brain
- Strict separation of identity and public data
- Text-only external LLM interaction
- Deterministic clustering where feasible
- Append-only evidence ledger
- No hidden editorial overrides
- Human-in-the-loop only via reproducible reruns

---

# 2. High-Level Architecture (v0)

```text
WhatsApp (Evolution API)
      -> Channel Adapter Skill
      -> Conversation.Router (Orchestration Core)
      -> Modular Skills
      -> PostgreSQL + Evidence Ledger
      -> Public Dashboard
```

The orchestration core handles only routing and coordination. Domain
logic lives in composable skills.

---

# 3. Orchestration Core (v0)

## Responsibilities

- Route incoming messages
- Trigger the correct skill
- Manage session state
- Enforce skill boundaries
- Log state transitions through `Evidence.Append`

## Core services

1. `Conversation.Router`
2. `Evidence.Append`
3. `Sandbox.ExecuteLLMCall`
4. `Sandbox.ExecuteBatchJob`

Everything else is modular.

---

# 4. Skill Model (v0)

Each skill:

- Has one responsibility
- Has explicit input/output schema
- Emits evidence events
- Cannot directly modify other domains
- Calls other skills only through the orchestrator

---

# 5. Skill Catalog (v0)

## A. Channel skills (WhatsApp)

### 1. `Channel.IngestWhatsAppMessage`

- Purpose: Normalize webhook payload
- Output: `UnifiedMessage` with `text`, `tokenized_account_ref`,
  `timestamp`, `message_id`

### 2. `Channel.SendWhatsAppMessage`

- Purpose: Outbound messaging
- Emits: `message_sent`

## B. Identity and sybil resistance skills

### 3. `Identity.StartEmailMagicLink`

- Generate magic-link token and send email

### 4. `Identity.CompleteEmailMagicLink`

- Verify token and mark `emailVerified=true`

### 5. `Identity.LinkWhatsAppAccount`

- Link verified user to WhatsApp ID
- Store only `HMAC(wa_id, pepper)`

### 6. `Sybil.EnforceEligibility`

- Rules: account age >= 48h, voting requires >= 1 accepted submission

### 7. `Abuse.RateLimitAndQuarantine`

- Enforce rate limits and quarantine rules

## C. Submission intake skills

### 8. `Submission.AcceptRaw`

- Store raw text
- Compute SHA-256 hash
- Set `status=pending`
- Emit evidence entry

### 9. `Submission.AckUser`

- Confirm receipt in Farsi

## D. Canonicalization and embedding skills

### 10. `Privacy.StripMetadataForLLM`

- Remove identifiers
- Shuffle batch
- Send text only

### 11. `NLP.DetectLanguage`

- Return language and confidence

### 12. `LLM.CanonicalizeToPolicyCandidate`

- Output: `title`, `summary`, `domain`, `stance`, `entities`,
  `confidence`, `flags`

### 13. `Embed.ComputeMistralEmbed`

- Generate embeddings in batches

### 14. `Candidate.PersistAndLink`

- Store candidates and link to submission

### 15. `User.NotifyCanonicalForm`

- Send canonicalized result link

## E. Clustering and summarization skills

### 16. `Cluster.LoadCandidatesForRun`

- Load cycle dataset

### 17. `Cluster.RunHDBSCAN`

- Deterministic seed
- `min_cluster_size=5`

### 18. `Cluster.MultiRunVarianceCheck`

- Run clustering 3x and set instability flag

### 19. `LLM.SummarizeCluster`

- Produce representative summary (Farsi primary)

### 20. `Cluster.PersistRun`

- Store `run_id`, parameters, seed, membership, variance flag

## F. Agenda and voting skills

### 21. `Agenda.Build`

- Include clusters above threshold

### 22. `Voting.OpenCycle`

- Open 48-hour window

### 23. `Voting.SendBallotPrompt`

- Send numbered list

### 24. `Voting.ParseApprovalBallot`

- Parse `"1,3,5"` format

### 25. `Voting.Cast`

- Store vote and emit `vote_cast`

### 26. `Voting.CloseAndTally`

- Compute totals and emit `cycle_closed`

### 27. `Voting.Remind`

- Send reminder at 24h

## G. Dispute and human review skills

### 28. `Dispute.Open`

- Create dispute record

### 29. `Dispute.ReviewQueue`

- Operator dashboard queue

### 30. `Dispute.ResolveByRerun`

- Resolution only through parameter change + pipeline rerun
- No manual output editing

## H. Evidence and transparency skills

### 31. `Evidence.Append`

- Append-only log fields: `event_type`, `entity_snapshot`, `prev_hash`,
  `current_hash`, `timestamp`

### 32. `Evidence.VerifyChain`

- Verify integrity

### 33. `Evidence.ExportDailyMerkleRoot` (optional)

- Export root for external anchoring

## I. Privacy and deletion skills

### 34. `Privacy.DeleteAccountMapping`

- Destroy email <-> WhatsApp mapping while preserving evidence integrity

### 35. `Privacy.RedactionGuard`

- Prevent identifiers from leaving trusted boundary

## J. Sandbox and safety skills

### 36. `Sandbox.ExecuteLLMCall`

- Allowlisted models
- Schema validation
- Token limits
- Cost logging

### 37. `Sandbox.ExecuteBatchJob`

- Pinned dependencies
- Deterministic seeds
- Resource caps

---

# 6. Evidence Event Types (v0)

- `submission_received`
- `candidate_created`
- `cluster_created`
- `cluster_updated`
- `vote_cast`
- `cycle_opened`
- `cycle_closed`
- `dispute_open`
- `dispute_resolved`
- `identity_verified`
- `rate_limited`

---

# 7. Minimal Core for v0 Delivery

Start v0 implementation with:

- `Conversation.Router`
- `Evidence.Append`
- `Sandbox.ExecuteLLMCall`
- `Sandbox.ExecuteBatchJob`
- `Submission.AcceptRaw`
- `LLM.CanonicalizeToPolicyCandidate`
- `Cluster.RunHDBSCAN`
- `Voting.Cast`

Everything else is layered incrementally.

---

# 8. v1 Hardening Plan

v1 keeps the v0 shape and adds stronger reliability, reproducibility, and
governance guarantees.

## 8.1 Deterministic runtime contract

- Pin image digest, Python/runtime versions, and model versions
- Fix locale, timezone, and numeric precision defaults
- Persist every run input and parameter set
- Store a `reproducibility_fingerprint` per run

## 8.2 Skill execution contract (required for every skill)

- `idempotency_key` on every mutating command
- Retry policy: max attempts, exponential backoff, failure class
- Timeout and cancellation behavior
- Compensating action or dead-letter route
- Required emitted evidence events

## 8.3 Queueing, ordering, and backpressure

- At-least-once delivery support for webhooks
- Deduplication on `message_id` + source
- Dead-letter queue for poison messages
- Worker concurrency limits per skill category
- Backpressure policy during surge windows

## 8.4 State machines for critical entities

### Submission

`received -> pending -> canonicalized -> clustered -> published | disputed`

### Candidate

`generated -> validated -> accepted | rejected`

### Cycle

`scheduled -> open -> closed -> tallied -> published`

### Dispute

`opened -> queued -> rerun_requested -> resolved`

## 8.5 Evidence integrity upgrades

- Add signature metadata (`signing_key_id`, `signature`)
- Nightly chain verification job with alerting
- External anchoring cadence (daily or weekly Merkle root)
- Immutable audit report exports for each cycle

## 8.6 Sybil and abuse defense upgrades

- Progressive trust tiers for new accounts
- Behavioral anomaly scoring (velocity, duplication, burst patterns)
- Quarantine review workflow with evidence-backed reasons
- Measurable false-positive/false-negative tracking

## 8.7 Observability and SLOs

- Pipeline latency SLOs (ingest -> canonicalize -> cluster -> publish)
- Error budget for skill failures
- Cost SLO per cycle and per active user
- Drift alerts for canonicalization/clustering instability

## 8.8 v1 additional skills (planned)

38. `Queue.DedupeAndAck`  
39. `Queue.DeadLetterReplay`  
40. `Run.ReproducibilityFingerprint`  
41. `Evidence.SignEvent`  
42. `Evidence.VerifyNightly`  
43. `Risk.ScoreBehavior`  
44. `Ops.EmitSLOMetrics`

---

# 9. Threat Model Snapshot (v1)

| Threat | Primary control | Detection signal | Residual risk |
| --- | --- | --- | --- |
| Duplicate webhook replay | `Queue.DedupeAndAck` | Rising dedupe ratio | Medium |
| Coordinated sybil voting | Trust tiers + anomaly scoring | Vote burst by related accounts | Medium |
| Silent operator manipulation | Rerun-only dispute policy + evidence signatures | Hash/signature mismatch | Low |
| LLM output drift | Version pinning + variance checks | Instability flag trend | Medium |
| PII leakage to LLM | `Privacy.RedactionGuard` + text-only payload | Redaction violations | Low |

---

# 10. Design Intent

Collective Will must be:

- Transparent
- Reproducible
- Privacy-preserving
- Deterministic where possible
- Explicit about uncertainty
- Resistant to silent manipulation

This modular architecture scales by adding capabilities, not by expanding
core complexity.

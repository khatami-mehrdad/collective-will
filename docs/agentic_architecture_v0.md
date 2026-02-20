## Agentic Architecture & Modular Skills Design (v0)

Status: Accepted baseline

---

# 1. Architectural Philosophy

Collective Will follows a **small orchestration core + modular skills**
architecture.

## Core principles

- Minimal orchestration brain
- Strict separation of identity and public data
- Text-only external LLM interaction
- Deterministic clustering where feasible
- Append-only evidence ledger
- No hidden editorial overrides
- No per-item human adjudication; autonomous rerun/escalation policies only

---

# 2. High-Level Architecture

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

# 3. Orchestration Core (Minimal Brain)

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

# 4. Skill Model

Each skill:

- Has one responsibility
- Has explicit input/output schema
- Emits evidence events
- Cannot directly modify other domains
- Calls other skills only through the orchestrator

---

# 5. Skill Catalog

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
- Store only random opaque account refs in core tables; keep raw `wa_id` only in sealed mapping

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
- Config-backed `min_cluster_size` (pilot can start lower, then graduate)

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

## G. Dispute adjudication skills

### 28. `Dispute.Open`

- Create dispute record

### 29. `Dispute.EscalateByConfidence`

- Route low-confidence disputes through stronger fallback / ensemble policy

### 30. `Dispute.ResolveByRerun`

- Resolution through scoped rerun + model escalation policy
- No manual output editing or per-item human override

## H. Evidence and transparency skills

### 31. `Evidence.Append`

- Append-only log fields: `event_type`, `entity_snapshot`, `prev_hash`,
  `current_hash`, `timestamp`

### 32. `Evidence.VerifyChain`

- Verify integrity

### 33. `Evidence.ExportDailyMerkleRoot` (config-driven publish)

- Compute local daily Merkle root is required
- External publication is optional by config

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

# 6. Evidence Event Types

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

# 8. Design Intent

Collective Will must be:

- Transparent
- Reproducible
- Privacy-preserving
- Deterministic where possible
- Explicit about uncertainty
- Resistant to silent manipulation

This modular architecture scales by adding capabilities, not by expanding
core complexity.

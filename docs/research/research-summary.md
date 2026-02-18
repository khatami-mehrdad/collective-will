# Research Summary: Pre-MVP Findings

This document summarizes all research conducted to inform MVP definition for Collective Will, with a focus on a pilot targeting Iranian users.

**Last Updated:** February 2026

---

## Research Areas Completed

| Area | Document | Key Finding |
|------|----------|-------------|
| Pilot Community | [research-iran-pilot-community.md](research-iran-pilot-community.md) | 89% of Iranians support democracy; 81% use VPNs; phone verification is unsafe |
| Canonicalization | [research-llm-canonicalization-evaluation.md](research-llm-canonicalization-evaluation.md) | Qwen3-8B likely sufficient; need 200+ human-evaluated items; BERTScore + NLI for automation |
| Clustering | (in agent output) | HDBSCAN recommended; v0 uses Mistral embed API (cloud); local embeddings at scale |
| Voting Mechanisms | (in agent output) | Approval voting + bridging analysis for v0; Community Notes achieves consensus 11.5% of time |
| Action Templates | [research-action-types-templates.md](research-action-types-templates.md) | "Draft and copy" model; inside-Iran users vote-only; email to US/EU officials as primary |
| Identity/Sybil | [research-identity-sybil-resistance.md](research-identity-sybil-resistance.md) | Multi-signal scoring without KYC; phone verification dangerous for Iran; progressive trust |
| Evidence Store | [research-transparency-log-implementations.md](research-transparency-log-implementations.md) | PostgreSQL append-only hash-chain for v0 (decision updated); external anchoring optional; migrate to Trillian Tessera at scale |
| Legal/Regulatory | [research-legal-regulatory.md](research-legal-regulatory.md) | OFAC GL D-2 allows serving Iranians; 501(c)(4) structure; Resistbot model for compliance |
| LLM Costs | [research-llm-cost-analysis.md](research-llm-cost-analysis.md) | $5-15/month API cost realistic; cloud-first for MVP simplicity |
| Success Metrics | [research-success-metrics-failure-conditions.md](research-success-metrics-failure-conditions.md) | >90% of civic platform users are one-time; 50+ users completing pipeline = minimum viable |

---

## Key Decisions Informed by Research

### 1. Iran Pilot Design

**Do:**
- Target diaspora for action execution (safety) — action deferred to v1
- Allow inside-Iran users to submit and vote only
- Use WhatsApp as only channel for v0 (no VPN required in Iran since Dec 2024)
- Support Farsi + Azerbaijani Turkish + Kurdish
- Partner with trusted diaspora media (Iran International, BBC Persian)

**Don't:**
- Require phone verification (SIAM surveillance system)
- Require real identity anywhere
- Enable traceable actions for inside-Iran users
- Store IP addresses or device fingerprints
- Add Telegram/Signal in v0 (prove trust loop with one channel first)

### 2. Identity Verification (v0)

Per [v0 Frozen Decisions](../mvp-specification.md#v0-frozen-decisions), v0 uses a single identity path:

**For Submissions:**
- Email magic-link verification + WhatsApp account linking
- Account age >= 48 hours before first submission
- Rate limiting (5 submissions/day/account)

**For Voting:**
- Same verification as above + at least 1 accepted submission

**Rationale:** One simple path. No OAuth, no vouching, no phone verification. Minimizes linkage risk and governance complexity while providing enough Sybil resistance for pilot scale.

### 3. AI Pipeline (Cloud-First for MVP)

| Stage | Model | Location | Notes |
|-------|-------|----------|-------|
| Canonicalization | Claude Haiku | Cloud | Farsi -> structured English; text-only sent (no user IDs) |
| Embeddings | Mistral `mistral-embed` | Cloud | Anonymized canonicalized text; switch to local at scale |
| Clustering | HDBSCAN | Local | Runs on returned vectors; no cloud dependency |
| Summarization | DeepSeek V3.2 | Cloud | Anonymized cluster data |

**Strategy:** Cloud-first for MVP simplicity. GPU infrastructure deferred until ~10K+ submissions/month. Data separation ensures user identity never leaves local infrastructure. See [MVP Specification §7.3](mvp-specification.md#73-why-cloud-first-for-mvp) for details.

### 4. Voting Mechanism

**v0:** Approval voting + bridging analysis

- Universal comprehension
- No vote splitting
- Simple mobile UX
- Community Notes-style matrix factorization to identify cross-group consensus

**Add later:** Quadratic voting, delegation, time-weighting

### 5. Action Types

**Phase 1 (launch):**
1. Email to US Congress (draft + copy flow)
2. Email to EU officials (draft + copy flow)
3. UN Special Rapporteur submissions

**Execution model:** Platform generates draft → user personalizes → user sends from their own email. No bulk sending from platform.

### 6. Evidence Store

**v0:** PostgreSQL append-only hash-chain table (per [v0 Frozen Decisions](../mvp-specification.md#v0-frozen-decisions))
- Embedded in the same Postgres instance as core data — simpler operations, unified backups
- External anchoring (Witness.co / Ethereum) optional for v0 launch
- Sufficient for communities up to ~100k entries

**v1:** Consider migration to Trillian Tessera when scale requires Merkle proofs; require external anchoring

### 7. Legal Structure

- **Recommended:** US 501(c)(4) nonprofit (allows unlimited lobbying/advocacy)
- OFAC General License D-2 explicitly allows serving Iranians
- Position platform as facilitation tool (Resistbot model)
- Explicit consent for political data processing (GDPR Article 9)

---

## v0 Success Criteria

### Minimum Viable Validation

> 50+ users complete full pipeline, >70% report satisfaction ≥4/5, >50% would use again, at least 1 official acknowledges receiving collective input.

### 3-Month Kill Criteria

- <100 registered users
- <50 submissions total
- >40% negative satisfaction ratings
- Zero official engagement
- AI clustering accuracy <60%

### Key Metrics to Track

| Category | Metric | Target |
|----------|--------|--------|
| Participation | Registered users | >100 |
| Participation | Submissions/month | >50 |
| Participation | Voting participation | >30% of registered |
| Trust | Dispute rate (AI outputs) | <10% |
| Trust | Pipeline completion rate | >70% |
| Trust | Return usage (30 days) | >20% |
| Quality | Canonicalization accuracy (human eval) | >80% |
| Quality | Cluster coherence (human eval) | >75% |
| Impact | Action completion | >50% of approved |
| Impact | Official responses | ≥1 |

---

## Cost Estimates (Updated for Cloud-First MVP)

### Monthly Costs at Pilot Scale (1,000 submissions)

| Component | Cost |
|-----------|------|
| VPS (Njalla 4GB) | ~$32 |
| LLM API (Claude Haiku + DeepSeek + Mistral embed) | ~$6 |
| Domain (Njalla, amortized) | ~$1 |
| DNS/CDN (Cloudflare) | Free |
| **Total** | **~$40-50/month** |

No GPU required for MVP. See [Infrastructure Guide](infrastructure-guide.md) for complete setup.

### Future Hardware (When Scaling Past 10K/month)

| Scale | GPU | Monthly Server Cost |
|-------|-----|---------------------|
| 10K-50K/month | RTX 4060 8GB | ~$80-100 |
| 50K-100K/month | RTX 4090 24GB | ~$150-200 |

---

## Open Questions Remaining

### High Priority (Block MVP)

1. **Pilot community outreach:** How to reach Iranian diaspora? Which organizations to partner with?
2. **Template localization:** Who translates templates? How to ensure Farsi matches English intent?
3. **Vouching bootstrap:** Who are the initial trusted users for vouching network?

### Medium Priority (Can iterate)

4. **Clustering granularity:** How many clusters per voting cycle? Dynamic or fixed?
5. **Voting window:** 24 hours? 48 hours? Variable?
6. **Action completion verification:** Trust self-reporting or skip entirely?

### Lower Priority (v1+)

7. **Federation:** When do independent nodes become necessary?
8. **Cryptographic verification:** When to add TEEs or zkML?
9. **Economic mechanisms:** Staking, quadratic funding?

---

## Next Steps

1. **Finalize MVP scope document** based on this research
2. **Draft action templates** for US Congress and EU officials (in Farsi + English)
3. **Design identity flow** with multi-signal scoring
4. **Prototype canonicalization** with sample submissions
5. **Identify launch partners** in Iranian diaspora community

---

## Documents Index

### Core Documents

| Document | Purpose |
|----------|---------|
| [mvp-specification.md](mvp-specification.md) | Complete MVP specification and architecture |
| [infrastructure-guide.md](infrastructure-guide.md) | Step-by-step server setup and operations guide |
| [operational-security.md](operational-security.md) | Contributor anonymity and safety (SENSITIVE) |

### Research Documents

| Document | Purpose |
|----------|---------|
| [research-iran-pilot-community.md](research-iran-pilot-community.md) | Iran context, trust, risks, safety |
| [research-llm-canonicalization-evaluation.md](research-llm-canonicalization-evaluation.md) | How to evaluate AI text structuring |
| [research-action-types-templates.md](research-action-types-templates.md) | What actions to support, template design |
| [research-identity-sybil-resistance.md](research-identity-sybil-resistance.md) | Identity verification approaches |
| [research-transparency-log-implementations.md](research-transparency-log-implementations.md) | Evidence store technology options |
| [research-legal-regulatory.md](research-legal-regulatory.md) | Legal compliance requirements |
| [research-llm-cost-analysis.md](research-llm-cost-analysis.md) | Cost model validation (includes local GPU analysis) |
| [research-success-metrics-failure-conditions.md](research-success-metrics-failure-conditions.md) | How to measure success and failure |

# Research Summary: Pre-MVP Findings

This document summarizes all research conducted to inform MVP definition for Collective Will, with a focus on a pilot targeting Iranian users.

**Last Updated:** February 2026

---

## Research Areas Completed

| Area | Document | Key Finding |
|------|----------|-------------|
| Pilot Community | [research-iran-pilot-community.md](research-iran-pilot-community.md) | 89% of Iranians support democracy; 81% use VPNs; phone verification is unsafe |
| Canonicalization | [research-llm-canonicalization-evaluation.md](research-llm-canonicalization-evaluation.md) | Qwen3-8B likely sufficient; need 200+ human-evaluated items; BERTScore + NLI for automation |
| Clustering | (in agent output) | BERTopic + HDBSCAN recommended; multilingual embeddings; 80%+ of tokens stay local |
| Voting Mechanisms | (in agent output) | Approval voting + bridging analysis for v0; Community Notes achieves consensus 11.5% of time |
| Action Templates | [research-action-types-templates.md](research-action-types-templates.md) | "Draft and copy" model; inside-Iran users vote-only; email to US/EU officials as primary |
| Identity/Sybil | [research-identity-sybil-resistance.md](research-identity-sybil-resistance.md) | Multi-signal scoring without KYC; phone verification dangerous for Iran; progressive trust |
| Evidence Store | [research-transparency-log-implementations.md](research-transparency-log-implementations.md) | SQLite + hash chain + Witness.co for v0; migrate to Trillian Tessera at scale |
| Legal/Regulatory | [research-legal-regulatory.md](research-legal-regulatory.md) | OFAC GL D-2 allows serving Iranians; 501(c)(4) structure; Resistbot model for compliance |
| LLM Costs | [research-llm-cost-analysis.md](research-llm-cost-analysis.md) | $5-15/month estimate is realistic; DeepSeek 5-25× cheaper than competitors |
| Success Metrics | [research-success-metrics-failure-conditions.md](research-success-metrics-failure-conditions.md) | >90% of civic platform users are one-time; 50+ users completing pipeline = minimum viable |

---

## Key Decisions Informed by Research

### 1. Iran Pilot Design

**Do:**
- Target diaspora for action execution (safety)
- Allow inside-Iran users to submit and vote only
- Distribute through Telegram/VPN channels
- Support Farsi + Azerbaijani Turkish + Kurdish
- Partner with trusted diaspora media (Iran International, BBC Persian)

**Don't:**
- Require phone verification (SIAM surveillance system)
- Require real identity anywhere
- Enable traceable actions for inside-Iran users
- Store IP addresses or device fingerprints

### 2. Identity Verification (v0)

**For Submissions:**
- Email verification with disposable blocking
- 24-hour account age before first submission
- Basic rate limiting

**For Voting:**
- Email verification PLUS one of:
  - Aged social OAuth (Twitter/GitHub >90 days)
  - OR 3+ approved contributions
  - OR vouching from trusted early user

**Rationale:** Multiple paths to voting without requiring KYC or phone numbers.

### 3. AI Pipeline

| Stage | Model | Tier | Tokens/Submission |
|-------|-------|------|-------------------|
| Canonicalization | Qwen3-8B | Local | ~880 |
| Embeddings | multilingual-e5-large | Local | ~100 |
| Clustering | BERTopic + HDBSCAN | Local | ~70 (amortized) |
| Action Planning | DeepSeek V3.2 | Cloud | ~14 (amortized) |

**89% of tokens stay local.** Cloud costs negligible at pilot scale.

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

**v0:** SQLite + hash chain + Witness.co anchoring
- Zero dependencies, single file
- External tamper evidence via Ethereum anchoring
- Sufficient for communities up to ~100k entries

**v1:** Migrate to Trillian Tessera when scale requires Merkle proofs

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

## Cost Estimates (Validated)

### Monthly Costs at Pilot Scale (1,000 submissions)

| Component | Cost |
|-----------|------|
| LLM inference (local) | $0 (hardware owned) |
| Cloud API (DeepSeek) | $0.06 |
| Witness.co anchoring | $3-5 |
| VPS (if not self-hosted) | $10-20 |
| **Total** | **$13-25/month** |

Original estimate of $5-15/month is achievable with self-hosted infrastructure.

### Hardware Requirements

| Scale | GPU | Cost |
|-------|-----|------|
| Dev/testing | RTX 4060 8GB | $300 |
| <10K/month | RTX 3060 12GB | $275 (used) |
| >20K/month | RTX 4090 24GB | $1,800 |

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

## Research Documents Index

| Document | Purpose |
|----------|---------|
| [research-iran-pilot-community.md](research-iran-pilot-community.md) | Iran context, trust, risks, safety |
| [research-llm-canonicalization-evaluation.md](research-llm-canonicalization-evaluation.md) | How to evaluate AI text structuring |
| [research-action-types-templates.md](research-action-types-templates.md) | What actions to support, template design |
| [research-identity-sybil-resistance.md](research-identity-sybil-resistance.md) | Identity verification approaches |
| [research-transparency-log-implementations.md](research-transparency-log-implementations.md) | Evidence store technology options |
| [research-legal-regulatory.md](research-legal-regulatory.md) | Legal compliance requirements |
| [research-llm-cost-analysis.md](research-llm-cost-analysis.md) | Cost model validation |
| [research-success-metrics-failure-conditions.md](research-success-metrics-failure-conditions.md) | How to measure success and failure |

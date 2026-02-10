# Research: LLM Cost Analysis for Civic Action Pipeline

Detailed cost modeling for the three-stage AI pipeline: canonicalization, clustering, and action planning.

**Last Updated:** February 2026

---

## Executive Summary

| Metric | Finding |
|--------|---------|
| **Original estimate** | $5-15/month for small community |
| **Revised estimate** | **$3-12/month** for 1,000 submissions/month (realistic) |
| **Key insight** | DeepSeek V3.2's pricing makes cloud viable even for MVP |
| **Break-even for local** | ~5,000-10,000 submissions/month justifies dedicated GPU |

The project's original $5-15/month estimate is **realistic and possibly conservative** for a small pilot using the proposed hybrid architecture.

---

## 1. Token Cost Estimation

### Typical Submission Length

Based on civic engagement research and similar platforms:

| Submission Type | Words | Tokens (est.) | Notes |
|-----------------|-------|---------------|-------|
| Short feedback | 50-100 | 70-140 | "I support rent control because..." |
| Standard submission | 100-300 | 140-400 | Typical civic comment |
| Detailed proposal | 300-500 | 400-700 | Multi-issue, structured |
| **Assumed average** | **200 words** | **~280 tokens** | Conservative estimate |

**Token estimation rule:** ~1.4 tokens per word for English text.

### Stage 1: Canonicalization

**Task:** Convert freeform text → structured PolicyCandidate JSON

```
Prompt template (~400 tokens):
- System instructions: 150 tokens
- Schema definition: 100 tokens  
- Few-shot examples: 100 tokens
- Task framing: 50 tokens

Input: ~280 tokens (user submission)
Output: ~200 tokens (structured JSON)

Total per submission: ~880 tokens
```

| Component | Tokens |
|-----------|--------|
| System prompt | 400 |
| User submission | 280 |
| Output (JSON) | 200 |
| **Total** | **880** |

### Stage 2: Clustering

**Task:** Group similar items, produce cluster summaries

Clustering has two sub-operations:

#### 2a. Embedding Generation
- Input: Title + summary from canonicalized output (~100 tokens)
- Processing: Vector similarity (no LLM tokens, just embedding)

#### 2b. Cluster Summarization
- Runs once per cluster, not per submission
- Assuming 10-20 items per cluster, ~50 clusters per 1,000 submissions

```
Per cluster summary:
- System prompt: 200 tokens
- Cluster items (10 × 100 tokens): 1,000 tokens
- Output summary: 150 tokens
Total: ~1,350 tokens per cluster

For 50 clusters: 67,500 tokens total
Per submission (amortized): 67.5 tokens
```

| Component | Tokens/submission |
|-----------|-------------------|
| Embedding input | 100 (embedding tokens) |
| Cluster summary (amortized) | 68 |
| **Total LLM tokens** | **~70** (excl. embeddings) |

### Stage 3: Action Planning

**Task:** Map winning items to templates, draft personalized content

This is the most token-intensive stage but runs only for winning items (not every submission).

```
Per action draft:
- System prompt: 300 tokens
- Winning item context: 200 tokens
- Template + instructions: 400 tokens
- Generated draft: 500 tokens
Total: ~1,400 tokens

Assuming 5-10 winning items per cycle from 1,000 submissions:
Per winning item: 1,400 tokens
Amortized per submission: 7-14 tokens
```

| Component | Tokens |
|-----------|--------|
| Full draft | 1,400 |
| Per submission (amortized @ 10 wins) | 14 |

### Total Tokens Per Submission

| Stage | Tokens | Model Tier |
|-------|--------|------------|
| Canonicalization | 880 | Local |
| Embedding | 100 | Local (free) |
| Cluster summary (amortized) | 70 | Local |
| Action planning (amortized) | 14 | Cloud |
| **Total** | **~1,064** | |

**Cloud tokens only:** ~14 tokens/submission (action planning)  
**Local tokens:** ~950 tokens/submission (canonicalization + clustering)

---

## 2. Cloud API Costs (Current Pricing - February 2026)

### Comparison Table

| Provider | Model | Input $/1M | Output $/1M | Notes |
|----------|-------|------------|-------------|-------|
| **DeepSeek** | V3.2 | $0.28 | $0.42 | Cache hit: $0.028 input |
| **DeepSeek** | V3.2 (cache) | $0.028 | $0.42 | 10x savings on repeated prompts |
| OpenAI | GPT-4o | $2.50 | $10.00 | Cached: $1.25 input |
| OpenAI | GPT-4o-mini | $0.15 | $0.60 | Best budget option |
| Anthropic | Claude Haiku 4.5 | $1.00 | $5.00 | Fast, affordable |
| Anthropic | Claude Sonnet 4.5 | $3.00 | $15.00 | Balanced |
| Anthropic | Claude Opus 4.5 | $5.00 | $25.00 | Flagship |

### Cost Multipliers vs DeepSeek V3.2

| Model | Input Cost Multiplier | Output Cost Multiplier |
|-------|----------------------|------------------------|
| DeepSeek V3.2 | 1× (baseline) | 1× (baseline) |
| GPT-4o-mini | 0.5× | 1.4× |
| Claude Haiku 4.5 | 3.6× | 12× |
| GPT-4o | 9× | 24× |
| Claude Sonnet 4.5 | 11× | 36× |

**Key insight:** DeepSeek V3.2 is **5-25× cheaper** than comparable models for output tokens.

### Action Planning Cost (Cloud Only)

Using DeepSeek V3.2 for action planning (1,400 tokens per winning item):

```
Per winning item:
- Input: 900 tokens × $0.28/1M = $0.000252
- Output: 500 tokens × $0.42/1M = $0.000210
- Total: $0.000462 per action draft

With caching (system prompt cached):
- Input: 300 non-cached × $0.28/1M + 600 cached × $0.028/1M = $0.000101
- Output: 500 tokens × $0.42/1M = $0.000210
- Total: $0.000311 per action draft (33% savings)
```

---

## 3. Embedding Costs

### Comparison Table

| Provider | Model | $/1M Tokens | Notes |
|----------|-------|-------------|-------|
| **Local** | BGE-small-en | $0 | Requires compute |
| **Voyage AI** | voyage-4-lite | $0.02 | 200M free tokens |
| **Voyage AI** | voyage-4 | $0.06 | Best quality/cost |
| OpenAI | text-embedding-3-small | $0.02 | Good default |
| OpenAI | text-embedding-3-large | $0.13 | Higher dimension |
| OpenAI | ada-002 (legacy) | $0.10 | Not recommended |
| Cohere | embed-v3 | $0.10 | Enterprise focus |

### Embedding Cost Per Submission

At 100 tokens per embedding:

| Provider | Cost per 1,000 submissions |
|----------|---------------------------|
| Local | $0 (compute only) |
| Voyage AI (lite) | $0.002 |
| OpenAI (3-small) | $0.002 |
| OpenAI (3-large) | $0.013 |

**Recommendation:** Use local embeddings (BGE-small-en-v1.5) for v0. Cloud embeddings add minimal cost but introduce dependency.

---

## 4. Local Inference Costs

### Hardware Options

| GPU | VRAM | Price (Feb 2026) | Best For |
|-----|------|------------------|----------|
| RTX 4060 | 8GB | ~$300-350 | Qwen3-8B (Q4), dev/testing |
| RTX 3060 12GB | 12GB | ~$250-300 | Qwen3-8B (Q8), comfortable |
| RTX 4090 | 24GB | $1,730-2,000 | MiMo-V2-Flash, production |
| RTX 3090 | 24GB | ~$800-1,000 (used) | Budget 24GB option |

### Inference Speed (Tokens/Second)

| GPU | Qwen3-8B (Q4) | Qwen3-8B (Q8) | MiMo-V2-Flash |
|-----|---------------|---------------|---------------|
| RTX 4060 | ~60-80 | ~40-50 | N/A (needs 12GB+) |
| RTX 3060 12GB | ~70-90 | ~50-60 | ~30-40 (tight) |
| RTX 4090 | 140-175 | 100-120 | 70-90 |

### Power Consumption

| GPU | TDP | Inference Load | Est. Usage |
|-----|-----|----------------|------------|
| RTX 4060 | 115W | ~60-80W | ~0.07 kWh/hr |
| RTX 3060 | 170W | ~100-120W | ~0.11 kWh/hr |
| RTX 4090 | 450W | ~200-300W | ~0.25 kWh/hr |

**Electricity cost** (@ $0.15/kWh US average):
- RTX 4060: ~$0.01/hour of inference
- RTX 4090: ~$0.04/hour of inference

### Processing Time Estimates

At 950 local tokens per submission:

| GPU | Tokens/sec | Time per submission | Throughput |
|-----|------------|---------------------|------------|
| RTX 4060 | 60 | 16 seconds | 225/hour |
| RTX 4090 | 150 | 6 seconds | 600/hour |

**1,000 submissions/month:**
- RTX 4060: ~4.4 hours total inference time → ~$0.05 electricity
- RTX 4090: ~1.7 hours total inference time → ~$0.07 electricity

### Cloud GPU Rental (If Not On-Premise)

| Provider | GPU | Price/Hour | Monthly (730 hrs) |
|----------|-----|------------|-------------------|
| **RunPod** | RTX 4090 | $0.59-0.74 | $430-540 |
| **RunPod** | A100 40GB | $1.79 | $1,300 |
| **Lambda Labs** | A100 40GB | $1.29 | $940 |
| **Vast.ai** | RTX 4090 | $0.30-0.50 | $220-365 |

**Key insight:** Cloud GPU rental only makes sense for burst workloads. For always-on inference, buying hardware pays back in 3-4 months.

### On-Demand vs Always-On Analysis

For 1,000 submissions/month (4.4 hours on RTX 4060):

| Approach | Monthly Cost | Notes |
|----------|--------------|-------|
| Own RTX 4060 | ~$0.05 electricity | Requires upfront $300 |
| RunPod on-demand | $2.60-3.30 | 4.4 hrs × $0.59-0.74 |
| Vast.ai on-demand | $1.30-2.20 | 4.4 hrs × $0.30-0.50 |

**Break-even for RTX 4060:** ~100 months at 1,000 submissions/month  
**But:** Hardware provides flexibility, privacy, no API dependency

---

## 5. Scale Scenarios

### Scenario A: 1,000 Submissions/Month (Small Pilot)

**Token breakdown:**
- Local (canonicalization + clustering): 950,000 tokens
- Cloud (action planning, 10 wins): 14,000 tokens
- Embeddings: 100,000 tokens

**Cost calculation:**

| Component | Method | Cost |
|-----------|--------|------|
| Canonicalization | Local (Qwen3-8B) | $0 (compute) |
| Clustering | Local | $0 (compute) |
| Embeddings | Local (BGE-small) | $0 |
| Action planning | DeepSeek V3.2 | $0.005 |
| Electricity | RTX 4060, 4.4 hrs | $0.05 |
| **Total** | | **~$0.06/month** |

**If using cloud for everything (DeepSeek V3.2):**
- Input: 650K tokens × $0.28/1M = $0.18
- Output: 400K tokens × $0.42/1M = $0.17
- **Total: ~$0.35/month**

**If using GPT-4o-mini for everything:**
- Input: 650K × $0.15/1M = $0.10
- Output: 400K × $0.60/1M = $0.24
- **Total: ~$0.34/month**

### Scenario B: 10,000 Submissions/Month (Medium Community)

| Component | Hybrid (Local + DeepSeek) | All-Cloud (DeepSeek) | All-Cloud (GPT-4o-mini) |
|-----------|---------------------------|----------------------|-------------------------|
| Canonicalization | $0 | $1.80 | $1.00 |
| Clustering | $0 | $0.20 | $0.15 |
| Action planning | $0.05 | $0.05 | $0.30 |
| Embeddings | $0 | $0.02 | $0.02 |
| Electricity | $0.50 | $0 | $0 |
| **Total** | **~$0.55/month** | **~$2.07/month** | **~$1.47/month** |

### Scenario C: 100,000 Submissions/Month (Scaled Deployment)

| Component | Hybrid | All-Cloud (DeepSeek) | All-Cloud (GPT-4o) |
|-----------|--------|----------------------|---------------------|
| Canonicalization | $0 | $18 | $180 |
| Clustering | $0 | $2 | $20 |
| Action planning | $0.50 | $0.50 | $15 |
| Embeddings | $0 | $0.20 | $1.30 |
| Electricity | $5 | $0 | $0 |
| GPU depreciation | ~$15 | $0 | $0 |
| **Total** | **~$20/month** | **~$21/month** | **~$216/month** |

**Note:** At 100K/month, dedicated RTX 4090 (~45 hours inference) pays for itself.

### Scale Summary Table

| Scale | Hybrid (Recommended) | All-DeepSeek | All-GPT-4o |
|-------|---------------------|--------------|------------|
| 1,000/month | $0.06 | $0.35 | $3.50 |
| 10,000/month | $0.55 | $2.07 | $35 |
| 100,000/month | $20 | $21 | $350 |

---

## 6. Hidden Costs

### Retries and Error Handling

| Issue | Frequency | Cost Impact |
|-------|-----------|-------------|
| API timeouts | 1-3% | +1-3% tokens |
| Malformed JSON output | 2-5% | +2-5% retry tokens |
| Rate limiting delays | Variable | Latency, not cost |
| Model hallucinations requiring re-run | 1-2% | +1-2% tokens |

**Recommended buffer:** Add 10% to token estimates for retries.

### Multi-Run Variance Analysis

For clustering stability, you may want to run clustering multiple times:

| Runs | Purpose | Token Multiplier |
|------|---------|------------------|
| 1 | Production default | 1× |
| 3 | Variance measurement | 3× (cluster summaries only) |
| 5 | High-confidence consensus | 5× (cluster summaries only) |

At 67K cluster tokens for 1,000 submissions:
- 3-run variance: +134K tokens (~$0.05 with DeepSeek)
- 5-run variance: +268K tokens (~$0.10 with DeepSeek)

### Development and Testing

| Phase | Estimated Usage | Cost (DeepSeek) |
|-------|-----------------|-----------------|
| Prompt engineering | 500K-2M tokens | $0.15-0.60 |
| Integration testing | 200K-500K tokens | $0.06-0.15 |
| Evaluation runs | 1M-5M tokens | $0.30-1.50 |
| Monthly dev usage | 500K tokens | $0.15 |

**Recommendation:** Budget $5-10 for initial development, $1-2/month for ongoing dev.

### Monitoring and Logging

| Item | Cost |
|------|------|
| Token usage logging | Negligible (local storage) |
| LLM output auditing | Storage only |
| Quality sampling | 5-10% of production = 5-10% more tokens |
| Error rate tracking | Free (instrumentation) |

### Infrastructure (Not Token-Related)

| Item | Monthly Cost | Notes |
|------|--------------|-------|
| VPS for inference server | $10-50 | If not self-hosted |
| Database (PostgreSQL) | $0-15 | Supabase free tier works |
| File storage | $0-5 | For audit logs |
| Domain/SSL | $1-2 | Cloudflare free tier |

---

## 7. Cost Optimization Strategies

### Caching

**Prompt caching:** DeepSeek offers 10× discount on cached input tokens.

| Cacheable Component | Tokens | Savings per 1K Submissions |
|---------------------|--------|---------------------------|
| System prompts | 400 | $0.10 |
| Schema definitions | 100 | $0.025 |
| Few-shot examples | 100 | $0.025 |

**Response caching:** Cache canonical forms of common submissions.
- If 10% of submissions are near-duplicates: 10% token savings on canonicalization.

### Batching

**Cluster summarization:** Process multiple clusters in single API call.

```
Single cluster call: 1,350 tokens × 50 clusters = 67,500 tokens
Batched (5 clusters per call): 5,000 tokens × 10 calls = 50,000 tokens
Savings: 26%
```

**Embedding batching:** Always batch embeddings (most APIs support up to 2,048 texts per call).

### Prompt Optimization

| Technique | Token Savings | Quality Impact |
|-----------|---------------|----------------|
| Shorter system prompts | 20-30% | Test required |
| Remove few-shot examples | 10-20% | May degrade |
| Structured output formatting | 5-10% | Often improves |
| Dynamic prompt selection | Variable | Task-dependent |

**Example:** A 400-token system prompt reduced to 250 tokens saves 37.5% on input costs.

### When to Upgrade from Cloud to Local

| Trigger | Recommendation |
|---------|----------------|
| >5,000 submissions/month | Consider RTX 4060 ($300) |
| >20,000 submissions/month | RTX 4090 justified ($1,800) |
| Privacy requirements | Local regardless of scale |
| Latency requirements (<2s) | Local (no network) |
| Offline operation needed | Local required |

**ROI calculation:**
- RTX 4060 ($300) vs cloud: Break-even at ~3,000 submissions/month after 3 months
- RTX 4090 ($1,800) vs cloud: Break-even at ~10,000 submissions/month after 6 months

---

## 8. Comparison with Original Estimates

### Original Estimate from llm-strategy.md

> "Small community (e.g. 1k submissions/month): mostly local inference + limited DeepSeek API → on the order of **$5–15/month** vs **$50–200+** if everything used a premium cloud API."

### Validated Analysis

| Claim | Actual Finding | Assessment |
|-------|----------------|------------|
| $5-15/month for 1K | $0.06-0.55 hybrid, $0.35-2 all-cloud | **Conservative (actual is lower)** |
| $50-200+ for premium cloud | $3.50 (GPT-4o), $35 (GPT-4o at 10K) | **Overstated for current pricing** |
| "Mostly local inference" | 89% of tokens local in hybrid model | **Accurate** |
| "Limited DeepSeek API" | ~14 tokens/submission for action planning | **Accurate** |

### Why Original Estimate Is Higher

The $5-15 estimate likely includes:

1. **Infrastructure costs** (VPS, database): $10-20/month
2. **Development/testing buffer**: $2-5/month
3. **Error retry buffer**: +10%
4. **Safety margin**: 2-3×

**Revised realistic budget (1,000 submissions/month):**

| Component | Low | High |
|-----------|-----|------|
| Token costs (hybrid) | $0.06 | $0.60 |
| Infrastructure | $0 (self-hosted) | $15 (VPS) |
| Dev/testing | $1 | $3 |
| Buffer (2×) | $2 | $6 |
| **Total** | **$3** | **$25** |

### Conclusion

The **$5-15/month estimate is realistic and achievable** for a small pilot. The actual token costs are lower than estimated, but infrastructure and development add to the total.

For a minimal self-hosted setup: **$3-5/month**  
For a managed infrastructure setup: **$10-20/month**

---

## 9. Recommended Configuration for MVP

### Hardware (Self-Hosted)

| Component | Recommendation | Cost |
|-----------|----------------|------|
| GPU | RTX 4060 (8GB) or used RTX 3060 12GB | $250-350 |
| System | Any modern desktop with PCIe x16 | Existing or $500 |
| RAM | 16GB minimum, 32GB recommended | $60-100 |

**Alternative:** Use Vast.ai on-demand for first 3 months ($1-3/month), then buy hardware if scaling.

### Model Configuration

| Stage | Model | Tier | Notes |
|-------|-------|------|-------|
| Canonicalization | Qwen3-8B (Q4/Q8) | Local | 5-6GB VRAM |
| Embeddings | BGE-small-en-v1.5 | Local | CPU-capable |
| Cluster summaries | Qwen3-8B | Local | Same model |
| Action planning | DeepSeek V3.2 | Cloud | $0.28/$0.42 per 1M |

### Monthly Budget Tiers

| Tier | Submissions | Infrastructure | Tokens | Total |
|------|-------------|----------------|--------|-------|
| Minimal | 1,000 | Self-hosted | $0.10 | $0.10 + electricity |
| Standard | 1,000 | $15 VPS | $0.10 | $15-20 |
| Comfortable | 10,000 | $30 VPS | $1.00 | $30-35 |
| Scale | 100,000 | Dedicated GPU server | $20 | $100-150 |

---

## 10. Key Findings Summary

1. **Token costs are minimal** — The hybrid architecture keeps cloud costs under $1/month even at 10,000 submissions.

2. **DeepSeek V3.2 changes the economics** — At $0.28/$0.42 per million tokens, "all-cloud" is viable for MVPs. The 5-15× cost advantage over GPT-4o is real.

3. **Local inference is nearly free** — Electricity costs are negligible (~$0.05/month at 1K scale). Hardware pays for itself quickly.

4. **Original estimate is conservative** — $5-15/month is achievable; actual token costs are lower. Budget $10-20/month for a comfortable margin including infrastructure.

5. **Embeddings are cheap** — Whether local (free) or cloud ($0.02/1M), embedding costs are negligible.

6. **Hidden costs matter more than tokens** — Infrastructure, development, and error handling likely exceed token costs for small pilots.

7. **Scale gracefully** — Start with cloud for everything (simpler), move to hybrid at 5K+ submissions, fully local canonicalization at 20K+.

---

## Appendix: Quick Reference

### Per-Submission Token Costs

| Stage | Tokens | DeepSeek Cost | GPT-4o-mini Cost |
|-------|--------|---------------|------------------|
| Canonicalization | 880 | $0.00031 | $0.00032 |
| Clustering (amortized) | 70 | $0.000025 | $0.000026 |
| Action planning (amortized) | 14 | $0.000005 | $0.000006 |
| **Total** | **964** | **$0.00034** | **$0.00035** |

### API Pricing Quick Reference (Feb 2026)

```
DeepSeek V3.2:    $0.28 in / $0.42 out  (cache: $0.028 in)
GPT-4o-mini:      $0.15 in / $0.60 out
GPT-4o:           $2.50 in / $10.00 out
Claude Haiku 4.5: $1.00 in / $5.00 out
Claude Sonnet 4.5:$3.00 in / $15.00 out
```

### GPU Quick Reference

```
RTX 4060 (8GB):  $300,  ~60 tok/s Qwen3-8B,  115W TDP
RTX 3060 (12GB): $275,  ~75 tok/s Qwen3-8B,  170W TDP
RTX 4090 (24GB): $1800, ~150 tok/s Qwen3-8B, 450W TDP
```

### Cloud GPU Rental

```
RunPod RTX 4090:  $0.59-0.74/hr
Vast.ai RTX 4090: $0.30-0.50/hr
Lambda A100 40GB: $1.29/hr
```

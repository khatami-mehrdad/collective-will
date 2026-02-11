# LLM Strategy & Model Selection

How we keep the first MVP cheap while staying ready for the latest open-source models. Local small models for easy tasks; cloud or larger local models for hard tasks.

This doc summarizes the current model landscape (February 2026), the hybrid architecture, and the action plan for the future.

---

## The Short Version

- **Goal:** Minimize cost while handling Farsi well. Use the right model for each task.
- **Architecture:** A single LLM abstraction with task-based routing. No vendor lock-in.
- **Language strategy:** Farsi is the primary user language; English is the internal processing language. Models are chosen based on Farsi support needs.
- **Farsi-touching tasks:** Claude Haiku — excellent Farsi comprehension, handles translation + understanding in one step.
- **English-only reasoning:** DeepSeek V3.2 — cheap, high quality for English, poor Farsi (avoid for user-facing).
- **Local tier:** Embeddings (BGE-small) + Qwen3-8B for clustering summaries. Run on CPU/low VRAM.
- **Action plan:** Start with Claude Haiku + DeepSeek V3.2; add local models as volume grows.

---

## Why This Matters for Collective Will

The pipeline has three AI-dependent stages:

| Agent | Task | Language | Model | Why |
|-------|------|----------|-------|-----|
| **Canonicalization** | Freeform Farsi → structured English | Farsi in | Claude Haiku | Excellent Farsi, translation + understanding in one step |
| **Clustering** | Group similar items, produce summaries | English | Local (BGE embeddings + Qwen3-8B) | High volume, English-only, cost-sensitive |
| **Action planning** | Map voted items to action templates, draft content | English | DeepSeek V3.2 | Complex reasoning, English-only, cheap |
| **User messages** | Bot responses, confirmations, vote prompts | Farsi out | Claude Haiku | User-facing Farsi quality matters |

**Key insight:** DeepSeek is ~10x cheaper than Claude but has poor Farsi/RTL support (issues closed as "not planned"). We use Claude only where Farsi quality matters, DeepSeek for English-heavy reasoning.

---

## Latest Open-Source Models (February 2026)

### Top-Ranked (Cloud / Large Local)

| Model | Params (active) | Notes |
|-------|------------------|-------|
| **Kimi K2.5** | 32B (1T total MoE) | #1 on open-source rankings; 85% LiveCodeBench, 96% AIME. Best via API; full model needs 240GB+ RAM. |
| **GLM-4.7 (Thinking)** | — | #2; 89% LiveCodeBench, 95% AIME. Zhipu AI. |
| **DeepSeek V3.2** | 37B (671B MoE) | #3; 86% LiveCodeBench, 92% AIME. Very cheap API, open weights. |

### Best Small Models for Local (Under 20B / Low VRAM)

| Model | Params (active) | Context | VRAM (Q4) | Best for |
|-------|------------------|---------|-----------|----------|
| **Qwen3-8B** | 8.2B | 131K | ~5GB | #1 under 20B; dual thinking/fast mode; math, code, 100+ languages. |
| **GLM-Z1-9B** | 9B | 128K | ~6GB | Strong reasoning; Zhipu’s latest small model. |
| **Qwen3-14B** | 14B | 128K | ~9GB | Strong all-rounder; 36T training tokens. |
| **MiMo-V2-Flash** | 15B (309B MoE) | 256K | 12GB min | #1 SWE-Bench (73.4%); 150 tok/s; MIT; hybrid attention, multi-token prediction. |
| **Llama 4 Scout** | 17B (16 experts) | 10M | ~12GB | Best multimodal in class; single H100. |
| **Qwen3-Coder-30B-A3B** | 3.3B (30B MoE) | 256K→1M | ~17GB | Coding specialist; only 3.3B active — runs like a small model. |

### Hardware requirements: best local (MiMo-V2-Flash)

The best quality/cost local option in the table above is **MiMo-V2-Flash**. Recommended hardware:

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| **GPU VRAM** | 8GB (heavy quantization) | **24GB** (e.g. RTX 3090, RTX 4090, RTX 5090) |
| **System RAM** | 16GB | 32–64GB |
| **Storage** | 100GB | 200GB+ NVMe SSD |
| **GPU** | NVIDIA (CUDA); 12GB for comfortable single-GPU | Single 24GB card (no multi-GPU required) |

- **8GB VRAM:** Possible with aggressive quantization; expect slower inference and lower quality.
- **12GB VRAM:** Usable (e.g. RTX 3060 12GB); may need quantized weights and reduced batch/context.
- **24GB VRAM:** Sweet spot for full context (256K) and good throughput with vLLM or SGLang.

For **Qwen3-8B** (cheapest viable local): ~5GB VRAM (Q4), 8GB RAM; runs on RTX 3060/4060 or similar.

### DeepSeek V4 (Upcoming)

- **Expected:** Mid-February 2026 (around Feb 17, Lunar New Year). Not officially confirmed.
- **Focus:** Coding; internal benchmarks claim gains over Claude/GPT on code.
- **Tech:** Engram memory (1M+ token context), mHC (Manifold-Constrained Hyper-Connections).
- **Expected policy:** Open-source (e.g. MIT), 5–15× cheaper than competitors. Small or distilled variants TBD but likely, given DeepSeek’s history (e.g. R1 distilled 1.5B–32B).

When V4 is released, we reassess cloud default and any new small distilled options for local.

### Why Newer Models Help

Newer models bring algorithm and data advantages at the same size: better reasoning (thinking/reasoning modes), MoE (large total params, small active params), longer context, and stronger instruction following. Prefer 2025–2026 models over older families (e.g. DeepSeek-R1 distilled) for the same VRAM budget.

---

## Hybrid Architecture

```
┌─────────────────────────────────────────────────────────┐
│                  LLM abstraction + router               │
│            (task-based: by agent / language)            │
└─────┬─────────────────┬─────────────────┬───────────────┘
      │                 │                 │
┌─────▼─────┐    ┌──────▼──────┐   ┌──────▼──────┐
│  FARSI    │    │   LOCAL     │   │  ENGLISH    │
│  TIER     │    │   TIER      │   │  REASONING  │
│           │    │             │   │             │
│ • Canon.  │    │ • Embeddings│   │ • Action    │
│ • User    │    │ • Cluster   │   │   planning  │
│   messages│    │   summaries │   │ • Complex   │
│           │    │             │   │   drafts    │
│ Model:    │    │ Models:     │   │             │
│ Claude    │    │ BGE-small   │   │ Model:      │
│ Haiku     │    │ Qwen3-8B    │   │ DeepSeek    │
│           │    │ (CPU/6GB)   │   │ V3.2        │
└───────────┘    └─────────────┘   └─────────────┘
```

- **Router:** Task-based routing by agent name. Farsi-in or Farsi-out → Claude. English reasoning → DeepSeek. Embeddings/summaries → Local.
- **Embeddings:** BGE-small-en-v1.5 for clustering similarity; runs on CPU.
- **Abstraction:** All agents call the same interface (e.g. `complete(prompt, model_tier)`); implementation swaps backend.

---

## Action Plan for the Future

### Phase 1 — Now (design / pre-MVP)

- Define the **LLM abstraction**: single interface for completion, with parameters for model tier, max tokens, temperature.
- Document **which agent uses which tier** (canonicalization → local, clustering → local, action planning → cloud by default).
- Optionally prototype with **RouteLLM** or a simple rule-based router (e.g. by agent name).

### Phase 2 — MVP (first deploy)

- **Farsi tier:** Claude Haiku for canonicalization (Farsi → structured English) and user-facing messages (Farsi responses).
- **Embeddings:** BGE-small-en-v1.5 for clustering similarity; runs on CPU.
- **Local summaries:** Qwen3-8B for cluster summaries (English-only, optional — can use DeepSeek if no GPU).
- **English reasoning:** DeepSeek V3.2 API for action planning and complex drafts.
- **Target hardware:** CPU-only is fine for MVP. Add GPU later for local Qwen3 if cost matters at scale.

### Phase 3 — After DeepSeek V4 (Feb 2026+)

- Evaluate **DeepSeek V4 API** as default cloud backend (and any small/distilled V4 models for local).
- If local quality matters more: add **MiMo-V2-Flash** (or Qwen3-Coder-30B-A3B for code-heavy tasks) for the local tier.
- Revisit **Kimi K2.5** for agentic or tool-heavy action planning if needed.

### Phase 4 — Scale and cost tuning

- Use router metrics to see what fraction of requests stay local; tune thresholds or router model.
- Consider **multiple local model sizes** (e.g. 8B for latency, 14B for quality) and route by workload.
- Re-evaluate new small/open models (e.g. Llama 4 family, new Qwen/GLM/Zhipu releases) as they appear.

### Cost ballpark (MVP)

**1k submissions/month estimate:**

| Task | Model | Volume | Cost |
|------|-------|--------|------|
| Canonicalization | Claude Haiku | 1k submissions | ~$2 |
| Clustering embeddings | BGE-small (local) | 1k items | $0 |
| Cluster summaries | Qwen3-8B (local) or DeepSeek | 50 clusters | ~$0.50 |
| Action planning | DeepSeek V3.2 | 10 actions | ~$0.50 |
| User messages | Claude Haiku | 3k messages | ~$3 |
| **Total** | | | **~$6/month** |

Compare: ~$50–100/month if using Claude for everything, or ~$150+ if using GPT-4.

---

## References

- [RouteLLM](https://github.com/lm-sys/RouteLLM) — drop-in OpenAI-compatible router for cost-efficient routing.
- [LLMRouter](https://ulab-uiuc.github.io/LLMRouter/) — routing strategies and benchmarks.
- [MiMo-V2-Flash](https://github.com/XiaomiMiMo/MiMo-V2-Flash) — 15B active, 309B MoE, MIT, local deployment.
- [Qwen3](https://qwen-ai.com/qwen-3/) — Qwen3-8B/14B, dual-mode reasoning.
- DeepSeek V4 coverage (e.g. The Information, Spectrum AI Lab) — release timing and features (as of Feb 2026).

---

## Open Questions

- Should the router be rule-based (by agent) at v0, or should we train/use a learned router from day one?
- What are the exact latency and throughput requirements for canonicalization and clustering in the first pilot?
- Do we want to support multiple local model backends (e.g. vLLM, SGLang, llama.cpp) behind the same abstraction, and how do we version them in the audit log?
- When DeepSeek V4 ships, do we add a “small/local V4” slot to the strategy as soon as distilled or small variants are available?

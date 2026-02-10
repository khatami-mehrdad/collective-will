# LLM Strategy & Model Selection

How we keep the first MVP cheap while staying ready for the latest open-source models. Local small models for easy tasks; cloud or larger local models for hard tasks.

This doc summarizes the current model landscape (February 2026), the hybrid architecture, and the action plan for the future.

---

## The Short Version

- **Goal:** Minimize token cost for the MVP by routing easy work to local small LLMs and hard work to cloud or capable local models.
- **Architecture:** A single LLM abstraction with a router. Each agent declares task difficulty; the router picks local or cloud. No vendor lock-in.
- **Local tier:** Latest small open-source models (Qwen3-8B, MiMo-V2-Flash, etc.) handle canonicalization, clustering summaries, and simple drafts. Run on a single consumer GPU (8–24GB VRAM).
- **Cloud tier:** DeepSeek V3.2 (and V4 when it ships) or Kimi K2.5 for action planning and complex drafting. DeepSeek API is ~5–15× cheaper than GPT-4.
- **Action plan:** Design the abstraction now; start with Qwen3-8B + DeepSeek API; upgrade to MiMo-V2-Flash and DeepSeek V4 as they stabilize.

---

## Why This Matters for Collective Will

The pipeline has three AI-dependent stages:

| Agent | Task | Complexity | Target tier |
|-------|------|------------|-------------|
| **Canonicalization** | Freeform text → structured policy candidates | Medium | Local |
| **Clustering** | Group similar items, produce summaries | Easy–Medium | Local (embeddings + small LLM) |
| **Action planning** | Map voted items to action templates, draft content | Hard | Cloud or capable local |

Research shows ~80% of real-world queries can be handled by local models (≤20B parameters), with large cost and energy savings. We design for that split from day one.

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
│         (RouteLLM, or custom difficulty-based)          │
└──────────────┬──────────────────────────┬────────────────┘
               │                          │
        ┌──────▼──────┐            ┌──────▼──────┐
        │ LOCAL TIER  │            │ CLOUD TIER  │
        │             │            │             │
        │ • Canonical.│            │ • Action    │
        │ • Clustering│            │   planning  │
        │ • Simple    │            │ • Complex   │
        │   drafts    │            │   drafting  │
        │             │            │             │
        │ Models:     │            │ Models:     │
        │ Qwen3-8B    │            │ DeepSeek    │
        │ MiMo-V2-    │            │ V3.2 / V4   │
        │ Flash       │            │ Kimi K2.5   │
        │ (6–12GB     │            │ (fallback)  │
        │  VRAM)      │            │             │
        └─────────────┘            └─────────────┘
```

- **Router:** Chooses local vs cloud per request (e.g. by agent, by estimated difficulty, or via a trained router like RouteLLM).
- **Embeddings:** Local embedding model (e.g. BGE-small) for clustering similarity; no GPU required.
- **Abstraction:** All agents call the same interface (e.g. `complete(prompt, options)`); implementation swaps backend.

---

## Action Plan for the Future

### Phase 1 — Now (design / pre-MVP)

- Define the **LLM abstraction**: single interface for completion, with parameters for model tier, max tokens, temperature.
- Document **which agent uses which tier** (canonicalization → local, clustering → local, action planning → cloud by default).
- Optionally prototype with **RouteLLM** or a simple rule-based router (e.g. by agent name).

### Phase 2 — MVP (first deploy)

- **Local:** Qwen3-8B (or Qwen3-14B if hardware allows). Handles canonicalization and clustering-side summarization.
- **Embeddings:** Small local model (e.g. BGE-small-en-v1.5) for clustering.
- **Cloud:** DeepSeek V3.2 API for action planning and complex drafts.
- **Target hardware:** Single GPU with 8–12GB VRAM (e.g. RTX 3060/4060) or 24GB (e.g. RTX 4090) for larger local model.

### Phase 3 — After DeepSeek V4 (Feb 2026+)

- Evaluate **DeepSeek V4 API** as default cloud backend (and any small/distilled V4 models for local).
- If local quality matters more: add **MiMo-V2-Flash** (or Qwen3-Coder-30B-A3B for code-heavy tasks) for the local tier.
- Revisit **Kimi K2.5** for agentic or tool-heavy action planning if needed.

### Phase 4 — Scale and cost tuning

- Use router metrics to see what fraction of requests stay local; tune thresholds or router model.
- Consider **multiple local model sizes** (e.g. 8B for latency, 14B for quality) and route by workload.
- Re-evaluate new small/open models (e.g. Llama 4 family, new Qwen/GLM/Zhipu releases) as they appear.

### Cost ballpark (MVP)

- **Small community** (e.g. 1k submissions/month): mostly local inference + limited DeepSeek API → on the order of **$5–15/month** vs **$50–200+** if everything used a premium cloud API.

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

# Collective Will

**Collective action, without central control.**

A system where people submit what they care about, AI agents organize and surface the common ground, and coordinated action happens only when users vote for it — transparently, auditably, and without anyone controlling the outcome.

---

## The Problem

Civic action is broken in a specific way: people care, but acting together costs too much.

- Coordinating with others requires time, expertise, and trust in whoever is organizing
- Agenda-setting is controlled by platforms, institutions, or whoever has the loudest voice
- Repetitive tasks — writing emails, making calls, following up — exhaust the people who care most
- Individual action doesn't scale, and collective action requires surrendering control to an organizer

The result: most people opt out. Not because they don't care, but because the systems for acting together are either too hard to use or too easy to capture.

## The Idea

What if coordination was automated but intent was not?

1. **You say what you care about** — submit a concern in plain text
2. **AI organizes, not editorializes** — agents structure and group similar submissions without deciding what matters
3. **The agenda is transparent** — everyone sees what surfaced and why
4. **You vote** — only items the community selects move forward
5. **Action happens with your consent** — the system drafts; you approve and send
6. **Everything is auditable** — every step, from your submission to the final action, is publicly verifiable

The system coordinates effort without controlling intent.

## Why Now

Three things changed:

- **LLMs can do the grunt work.** Canonicalizing freeform text, clustering similar concerns, drafting templated actions — these are now tractable with AI, at a quality level that didn't exist three years ago.
- **Trust in institutions is low.** People are skeptical of platforms that claim to act in their interest. A system that proves its behavior through transparency has an opening.
- **Open-source AI tooling is mature enough.** You can run capable models on commodity hardware. Decentralized agent execution is no longer a theoretical exercise.

## Principles

- The system expresses the will of its users — not the preferences of its builders
- Coordination should be easy
- Legitimacy should be verifiable
- Power should remain distributed

---

## Current Status

This is a **design-phase project** transitioning to implementation. The MVP specification is complete.

**Pilot focus**: Iran — surfacing what Iranians collectively want.

**v0 goal**: Consensus visibility. Make collective preferences visible. Action execution is v1.

See **[MVP Specification](docs/mvp-specification.md)** for the full technical design.

---

## Context & Prior Art

What exists, what we can learn from, and where this project fits in the landscape.

- **[Landscape & Prior Art](docs/landscape.md)** — existing tools and projects that overlap with parts of this system (Polis, Talk to the City, DCAN, Resistbot, Decidim, and others), what they do well, where they stop, and what it means for this project.
- **[Roadmap](docs/roadmap.md)** — what v0 includes, what's deliberately excluded and why, the direction after v0, and open questions about what success looks like.

---

## Technical Design

How the system works, how trust is built, and how it could be implemented.

- **[MVP Specification](docs/mvp-specification.md)** — the complete v0 technical design: architecture, modules, data models, user flows, technology stack, and implementation milestones.
- **[Agent Design](docs/agents.md)** — how the canonicalization, clustering, and action planning agents work, what their input/output contracts look like, and the hard unsolved problems around bias, quality evaluation, and schema design.
- **[Architecture](docs/architecture.md)** — the full pipeline from submission to action, the evidence store, a worked example following a user through the entire lifecycle, and open questions about storage, scaling, and processing models.
- **[Governance and Trust](docs/governance-and-trust.md)** — the trust model, separation of power between roles, the threat model, decentralization approach, and open questions about identity, voting mechanisms, and protocol governance.
- **[Visibility and Trust](docs/visibility-and-trust.md)** — how to actually build verifiable trust: transparency logs, signed agent receipts, verifiable AI, sybil-resistant identity, end-to-end verifiable voting, and why most of this doesn't require blockchain.
- **[OpenClaw as Infrastructure](docs/openclaw-as-infrastructure.md)** — how OpenClaw's multi-agent architecture, channel integrations, skills system, and memory model could serve as the foundation for a v0, what we'd get for free, and what we'd still need to build.
- **[LLM Strategy & Model Selection](docs/llm-strategy.md)** — latest open-source models (Feb 2026), hybrid local/cloud architecture for low-cost MVP, and action plan (Qwen3-8B, MiMo-V2-Flash, DeepSeek V3.2/V4).

---

## Pre-MVP Research

Research conducted to inform MVP definition. All documents have detailed findings with citations.

- **[Research Summary](docs/research-summary.md)** — consolidated findings, key decisions, cost estimates, success criteria, and next steps.
- **[Iran Pilot Community](docs/research-iran-pilot-community.md)** — why Iran, trust landscape, messaging apps, safety considerations, and design implications.
- **[Canonicalization Evaluation](docs/research-llm-canonicalization-evaluation.md)** — how to evaluate LLM text structuring, metrics, schema design, failure modes.
- **[Action Types & Templates](docs/research-action-types-templates.md)** — what civic actions to support, template design, diaspora advocacy, platform constraints.
- **[Identity & Sybil Resistance](docs/research-identity-sybil-resistance.md)** — verification tiers, multi-signal scoring, authoritarian context safety.
- **[Transparency Log Implementations](docs/research-transparency-log-implementations.md)** — Trillian vs Rekor vs SCITT vs simpler approaches.
- **[Legal & Regulatory](docs/research-legal-regulatory.md)** — OFAC, GDPR, platform ToS, FARA, legal structure.
- **[LLM Cost Analysis](docs/research-llm-cost-analysis.md)** — validated cost model, local vs cloud, scale scenarios.
- **[Success Metrics & Failure Conditions](docs/research-success-metrics-failure-conditions.md)** — what to measure, kill criteria, prior art lessons.

---

## Contributing

Read **[CONTRIBUTING.md](CONTRIBUTING.md)**.

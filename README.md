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

This project is **transitioning from design to implementation**. v0 scope is frozen.

**Pilot focus**: Iran — surfacing what Iranians collectively want.

**v0 goal**: Consensus visibility + approval voting. No action execution (deferred to v1).

**v0 key constraints**:
- WhatsApp only (single channel)
- Email magic-link + WhatsApp account linking (no phone verification, no OAuth)
- Privacy-first infrastructure (Njalla/1984.is; WHOIS hides owner)
- Postgres hash-chain evidence store (external anchoring optional)

See **[MVP Specification — v0 Frozen Decisions](docs/mvp-specification.md#v0-frozen-decisions)** for the complete locked scope.

---

## Technical Design

Core documents for understanding and building the MVP.

- **[MVP Specification](docs/mvp-specification.md)** — the complete v0 technical design including **[v0 Frozen Decisions](docs/mvp-specification.md#v0-frozen-decisions)**: architecture, modules, data models, user flows, technology stack, and implementation milestones.
- **[Roadmap](docs/roadmap.md)** — what v0 includes, what's deliberately excluded and why, pilot gates.
- **[Agent Design](docs/agents.md)** — how the canonicalization, clustering, and action planning agents work.
- **[LLM Strategy](docs/llm-strategy.md)** — which LLMs for which tasks, cost estimates, hybrid architecture.
- **[Infrastructure Guide](docs/infrastructure-guide.md)** — deployment setup, Docker, privacy-first hosting.

---

## Pre-MVP Research (Archive)

Research conducted to inform MVP decisions. Decisions are now captured in the main docs above.

See **[docs/research/](docs/research/)** for:
- Landscape & prior art analysis
- Channel integration research (OpenClaw evaluation)
- Iran pilot community research
- Identity & Sybil resistance options
- Transparency log implementations
- Legal & regulatory considerations
- LLM cost analysis
- Success metrics & failure conditions

---

## Contributor Safety

**This project targets Iran.** Contributors may have family in sensitive countries. To protect everyone:

- **Use pseudonymous identities** for all public-facing work (git commits, GitHub account, communications)
- **Never use your real name** in commits, code comments, or public discussions
- **Read the [Operational Security Guide](docs/operational-security.md)** before contributing

If you're setting up infrastructure, the **[Infrastructure Guide](docs/infrastructure-guide.md)** includes anonymous hosting options.

---

## Contributing

Read **[CONTRIBUTING.md](CONTRIBUTING.md)**.

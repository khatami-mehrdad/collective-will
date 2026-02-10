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

This system proposes a pipeline where:

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

## Current Status

This is a **design-phase project**. No production code. The goal right now is to explore feasibility, identify risks, and pressure-test the governance model before writing implementation.

Every document has open questions at the bottom. Those questions are the real work.

---

## Navigate by Interest

### "I'm interested in AI and NLP"

Start with **[Agent Design](docs/agents.md)** — how the canonicalization, clustering, and action planning agents work, what their input/output contracts look like, and the hard unsolved problems around bias, quality evaluation, and schema design.

### "I care about trust, governance, and power dynamics"

Start with **[Governance and Trust](docs/governance-and-trust.md)** — the trust model, separation of power between roles, the threat model, decentralization approach, and open questions about identity, voting mechanisms, and protocol governance.

### "I want to think about system design and infrastructure"

Start with **[Architecture](docs/architecture.md)** — the full pipeline from submission to action, the evidence store, a worked example following a user through the entire lifecycle, and open questions about storage, scaling, and processing models.

### "I want to understand the big picture and where this is going"

Start with **[Roadmap](docs/roadmap.md)** — what v0 includes, what's deliberately excluded and why, the direction after v0, and open questions about what success looks like.

### "I want to contribute"

Read **[CONTRIBUTING.md](CONTRIBUTING.md)**.

---

## Principles

- The system expresses the will of its users — not the preferences of its builders
- Coordination should be easy
- Legitimacy should be verifiable
- Power should remain distributed

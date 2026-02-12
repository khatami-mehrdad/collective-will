# Governance and Trust

This system is designed to coordinate collective action. That means it has power — and power requires accountability. The governance model is built on one core principle: **trust emerges from visibility, not authority.**

No one — not the developers, not the infrastructure operators, not any single user — should be able to control what the system says or does on behalf of its users.

## Core Principles

**Deterministic rules over human judgment.** The system operates by protocol, not discretion. Agenda formation, voting mechanics, and action execution follow published rules that anyone can verify.

**User consent by default.** Nothing happens on a user's behalf without their knowledge and approval. In v0, all consent is explicit. Delegated automation comes later, with strict limits.

**No silent changes.** Every modification to the agenda, to cluster summaries, to action plans — every change is logged and publicly visible. If something changed, you can see who or what changed it, and when.

**Publicly verifiable records.** The system maintains an append-only evidence store. Every submission, every canonicalization, every cluster assignment, every vote, every action plan, and every execution result is recorded with hashes and timestamps.

## Separation of Power

The system defines three roles, deliberately separated:

**App Users** — The people. They submit concerns, vote on the agenda, approve actions. They are the only ones who determine what the system acts on.

**Nodes** — Infrastructure operators. They run AI agents, provide compute, serve the platform. They have no influence over what gets prioritized, how it's worded, or what actions are taken.

**Architects** — Protocol designers. They define the rules, maintain the system, and evolve the design. They do not participate in agenda-setting, voting, or execution. Their power is structural, not operational. Architects operate under pseudonyms — no real identities are publicly associated with this role. See [operational-security.md](operational-security.md).

This separation exists to prevent capture. No single role can control both the rules and the outcomes.

## Decentralization Approach

Decentralization isn't a binary — it's a spectrum, and jumping to the deep end before the design is proven is a risk in itself.

**v0: Centralized coordination, decentralized verification.**
The system runs on coordinated infrastructure, but everything it does is publicly auditable. Anyone can verify that the rules were followed. Trust the math, not the operator.

**Future: Distributed execution.**
As the design matures, agent execution moves to independent compute nodes. No single operator controls the pipeline. The protocol, not the platform, becomes the source of truth.

Decentralization is introduced gradually to preserve safety and clarity. Moving too fast here creates complexity that undermines the transparency guarantees.

## Threat Model

Honest threat assessment builds credibility. Here's what can go wrong:

**Sybil attacks.** Fake accounts flooding the system with coordinated submissions to manipulate the agenda. Mitigation: rate limiting, identity verification tiers, statistical anomaly detection on submission patterns.

**Clustering bias.** The clustering agent subtly shaping what "counts" as a topic by how it groups submissions. Mitigation: published grouping logic, user-challengeable clusters, multiple clustering runs with variance analysis.

**Agent output bias.** The AI agents introducing editorial slant through canonicalization or action planning. Mitigation: deterministic templates for actions, original submissions always preserved, output diffing against inputs.

**Platform bans.** Automated action execution (emails, social posts) being flagged as spam or violating terms of service. Mitigation: human-in-the-loop execution in v0, rate limiting, per-user authentication rather than centralized bot accounts.

**Malicious compute nodes.** Nodes running tampered agents that alter outputs. Mitigation: reproducible agent execution, output verification across multiple nodes, hash-based integrity checks.

**Capture by organized groups.** A coordinated minority gaming the voting system to dominate the agenda. Mitigation: this is an open design problem — see open questions.

### Open Questions

- What's the right identity model? Full anonymity enables Sybil attacks. KYC kills participation. What's the middle ground?
- How do you audit AI agent behavior in a way that non-technical users can understand?
- What voting mechanism best resists strategic manipulation while remaining simple? (Majority? Quadratic? Conviction voting?)
- How do you handle the tension between transparency (everything is public) and privacy (user submissions may be sensitive)?
- At what point does decentralization become necessary vs. aspirational? What's the trigger?
- How do you govern changes to the protocol itself? Who decides when the rules change?

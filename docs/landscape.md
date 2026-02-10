# Landscape & Prior Art

What already exists, what overlaps with this project, and where the gaps are.

This is not a competitive analysis in the business sense — most of these projects are open-source, non-profit, or academic. The goal is to understand what's been tried, what works, and what Collective Will would need to do differently.

---

## The Short Version

The landscape has strong individual pieces, but no one has assembled the full pipeline. Existing tools cover parts of the problem — clustering opinions, decentralized coordination, automated civic action — but nothing combines freeform submission, AI-driven clustering, transparent agenda-setting, community voting, automated action, and end-to-end auditability into a single system.

That integration is the core bet of this project.

---

## AI Opinion Clustering & Deliberation

### Polis (pol.is)

Open-source (AGPL-3.0). Uses ML and statistical analysis to cluster participants by opinion similarity and surface consensus statements. Famous for powering **vTaiwan**, where 80% of 26 issues deliberated led to government action. Used globally by governments and organizations.

- **Overlap:** Clustering opinions, surfacing common ground, open source, proven at national scale.
- **Gap:** No freeform text canonicalization — participants agree/disagree on pre-written statements, so the agenda is still set by whoever writes the seeds. No automated action pipeline. No LLM-based understanding of intent.
- **Link:** [pol.is](https://pol.is/home) | [GitHub](https://github.com/compdemocracy/polis)

### Talk to the City (T3C)

Open-source, by the AI Objectives Institute. Uses LLMs (GPT-4o) to extract arguments from freeform text, cluster them hierarchically, and produce interactive reports where every theme is grounded in individual quotes.

- **Overlap:** LLM-based clustering of freeform input, traceability back to individual submissions, transparency, open source.
- **Gap:** It's a sense-making and reporting tool, not an action platform. No voting, no coordinated action, no governance model. Analyzes what people said — doesn't help them do anything about it.
- **Link:** [talktothe.city](https://talktothe.city/) | [GitHub](https://github.com/aiobjectives/talk-to-the-city-reports)

### Generative Social Choice (Academic)

Research by Procaccia et al. (ACM EC 2023, ICML 2025). Uses LLMs combined with social choice theory to distill free-text opinions into representative "slates" of statements with mathematical fairness guarantees (Proportional Slate Engine / PROSE).

- **Overlap:** Directly addresses the problem of aggregating freeform text into representative positions with formal fairness properties.
- **Gap:** Academic framework, not a product. No action pipeline, no governance, no user-facing system. But the theoretical grounding is relevant to how Collective Will's clustering agents should behave.
- **Link:** [ACM paper](https://dl.acm.org/doi/10.1145/3670865.3673547) | [ICML 2025 follow-up](https://arxiv.org/abs/2505.22939)

### Deliberation.io

Combines citizen engagement with AI facilitation to identify common ground in large-scale dialogues. Used by Washington DC for an AI policy listening session in collaboration with MIT and Stanford.

- **Overlap:** AI-facilitated large-scale deliberation, finding common ground.
- **Gap:** Facilitation-focused, not action-focused. Designed for structured consultations run by institutions, not emergent bottom-up coordination.
- **Link:** [deliberation.io](https://deliberation.io/)

---

## Decentralized Coordination & Governance

### DCAN (Distributed Collective Action Network)

Built on Holochain. Uses "participation-contingent agreements" — people only commit to actions when enough others do too. No central decision-making, no coercion, no voting. Relies on free information flow and consensus-building.

- **Overlap:** Decentralized, no central control, collective action through consensus, cryptographic auditability. Philosophically the closest to Collective Will's goals.
- **Gap:** No AI layer. No opinion clustering or canonicalization. Coordination is manual and consensus-based, not AI-assisted. Early stage, small community, niche Holochain ecosystem.
- **Link:** [dcan.app](https://dcan.app/)

### Loomio

Mature platform for structured proposals, consent/consensus processes, and voting. Used by cooperatives, boards, and self-managing teams.

- **Overlap:** Collective decision-making, voting, structured processes, transparency.
- **Gap:** No AI. No freeform input clustering. No automated action. Requires someone to write proposals — the agenda is still set by individuals. Designed for organizations with existing membership, not open civic action.
- **Link:** [loomio.com](https://www.loomio.com/)

### Decidim

Open-source participatory democracy platform used by Barcelona, New York City, Helsinki, and others. Supports participatory budgeting, assemblies, strategic planning, and citizen initiatives.

- **Overlap:** Large-scale citizen participation, structured democratic processes, open source, proven at city scale.
- **Gap:** No AI. Heavy institutional setup — designed for government-run processes, not bottom-up emergent coordination. Someone (usually a government body) still sets the agenda.
- **Link:** [decidim.org](https://decidim.org/)

### X/Twitter Community Notes

Uses a "bridging algorithm" that requires notes to be rated helpful by people who typically disagree with each other. Open-source algorithm, publicly available data.

- **Overlap:** Bridging across perspectives, transparent and auditable algorithm, crowdsourced consensus at massive scale, manipulation resistance.
- **Gap:** Content moderation tool, not a coordination platform. No action pipeline. But the bridging algorithm design is a relevant precedent for how Collective Will might surface genuine consensus vs. faction-driven outcomes.
- **Link:** [GitHub](https://github.com/twitter/communitynotes)

---

## Automated Civic Action

### Resistbot

Text-based advocacy platform. Text 50409 to write and send letters to elected officials via email, fax, or postal mail. AI-assisted drafting, petitions, campaigns, voter registration tools.

- **Overlap:** Lowering the barrier to civic action, AI-assisted drafting, multi-channel delivery.
- **Gap:** Fully centralized. No collective agenda-setting — each individual decides what to write about independently. No clustering, no emergent consensus. The "what" is decided by each user alone or by campaign organizers.
- **Link:** [resist.bot](https://resist.bot/)

### Legisletter

Generates AI-powered personalized advocacy letters, auto-targets correct representatives, provides campaign pages and analytics. Reports 280,000+ letters sent.

- **Overlap:** AI-generated civic action, reducing friction for contacting representatives.
- **Gap:** Campaign-driven — someone else sets the agenda. Centralized platform. No collective deliberation or consensus mechanism.
- **Link:** [legisletter.org](https://legisletter.org/)

---

## Traditional Petition & Advocacy Platforms

### Change.org / Avaaz

Massive scale (Change.org: 75M+ users). Petition-based model. For-profit (Change.org sells preferential access to its user base).

- **What Collective Will is reacting against:** Centralized control, opaque agenda-setting, profit-driven incentives, optimized for "clicktivism" over deep engagement. Complex systemic issues don't fit the bumper-sticker format. Action = signing a petition, which is low-effort but also low-impact.
- **Link:** [change.org](https://www.change.org/)

---

## Other Relevant Projects

### Policy Synth (Citizens Foundation)

TypeScript library for multi-scale AI agent logic flow for policy analysis and decision-making. Open source.

- **Link:** [GitHub](https://github.com/citizensfoundation/policy-synth)

### CollectiVAI

European initiative building ethical AI infrastructure for civic use — serving cities, universities, NGOs. Focuses on climate dialogues, policy analysis, and participatory processes.

- **Link:** [collectivai.org](https://collectivai.org/)

### Nesta Civic AI Toolkit

Helps communities use data and AI to tackle local challenges. Supports civil society organizations and local authorities.

- **Link:** [nesta.org.uk/toolkit/civicai](https://www.nesta.org.uk/toolkit/civicai/)

### Metagov Interoperable Deliberative Tools

Grant program funding ~14 projects to build interoperable deliberation tools with shared data formats (JSON, JSON-LD, CSV). Relevant to the question of how Collective Will might integrate with existing tools.

- **Link:** [metagov.github.io/interop](https://metagov.github.io/interop/)

---

## Comparison Matrix

| Project | Freeform Input | AI Clustering | Transparent Agenda | Voting | Automated Action | Auditable | Decentralized |
|---|---|---|---|---|---|---|---|
| **Collective Will** | Yes | Yes (LLM) | Yes | Yes | Yes | Yes | Yes (goal) |
| **Polis** | Partial | Yes (ML) | Yes | Yes (agree/disagree) | No | Partial | No |
| **Talk to the City** | Yes | Yes (LLM) | Yes | No | No | Yes | No |
| **DCAN** | No | No | Yes | Consensus-based | Yes (manual) | Yes (Holochain) | Yes |
| **Loomio** | No | No | Partial | Yes | No | Partial | No |
| **Decidim** | Partial | No | Yes | Yes | No | Partial | No |
| **Resistbot** | Yes | No | No | No | Yes | No | No |
| **Legisletter** | Partial | No | No | No | Yes | No | No |
| **Change.org** | Yes | No | No | Yes (sign) | Partial | No | No |
| **Community Notes** | Yes | Bridging algo | Yes | Yes (ratings) | N/A | Yes | No |
| **Gen. Social Choice** | Yes | Yes (LLM) | Yes | Theoretical | No | Theoretical | N/A |

---

## What This Tells Us

1. **The "listen and cluster" problem is partially solved.** Polis and Talk to the City demonstrate that AI-assisted opinion clustering works and can operate at national scale. The open question is whether LLM-based canonicalization is reliable enough to replace the human-authored seed statements that Polis requires.

2. **The "act on it" problem is solved for individuals, not groups.** Resistbot and Legisletter make it easy for one person to contact a representative. But there's no system that turns collective consensus into collective action — where the group decides what to say and the system executes it with each person's consent.

3. **Decentralized coordination exists but without AI.** DCAN proves the philosophical model works on Holochain. The question is whether you can add an AI layer without reintroducing centralization (since LLMs currently require significant compute).

4. **Auditability is rare.** Community Notes and DCAN take it seriously. Most civic tech platforms treat transparency as "we publish our process" rather than "every step is cryptographically verifiable."

5. **Nobody owns the full pipeline.** The gap between "what do people care about" and "coordinated action happens" remains unbridged. That's the space Collective Will occupies.

---

## Open Questions

- Should Collective Will integrate with existing tools (e.g., use Polis for the voting layer, Resistbot for the action layer) or build the full stack?
- What can we learn from Polis's deployment in Taiwan about what makes adoption work at scale?
- How does the generative social choice research inform the design of the clustering agents — should we aim for formal fairness guarantees?
- Is Holochain (or similar) the right foundation for decentralization, or is that premature complexity?
- Community Notes' bridging algorithm is battle-tested against manipulation at massive scale — is there a version of that idea that applies here?

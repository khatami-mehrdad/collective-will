# Voting Mechanisms Research for Civic Agenda-Setting

Research on voting mechanisms suitable for a civic agenda-setting system where clustered policy concerns are presented to users who vote on which should trigger collective action. The system needs to resist manipulation while remaining understandable.

---

## 1. Voting Mechanism Options

### Simple Majority / Plurality

**Mechanism:** Each voter picks one option; the option with the most votes wins.

**Strengths:**
- Maximum simplicity—universal comprehension
- Minimal cognitive load
- Battle-tested across centuries of elections

**Weaknesses:**
- Vote splitting: multiple similar options dilute support
- No intensity expression: someone who cares deeply counts the same as someone indifferent
- Tyranny of the majority: 51% can override 49% on every issue
- Vulnerable to strategic voting (voting for "lesser evil" vs true preference)

**Best for:** Binary decisions, low-stakes polls, situations where simplicity trumps all.

### Approval Voting

**Mechanism:** Voters approve all options they find acceptable; highest approval count wins.

**Strengths:**
- Simple to understand ("check all you like")
- No vote splitting—supporting a second choice cannot hurt your first
- Produces richer preference data than plurality
- Low ballot spoilage rates compared to RCV [Election Science, 2024]

**Weaknesses:**
- Requires threshold decisions ("how many should I approve?")
- Strategic uncertainty with multiple viable candidates [RCVTheory.com]
- Research finds approval voting among *most* vulnerable to strategic manipulation in comparative studies [Laslier, 2010]

**Real-world evidence:** St. Louis 2025 mayoral election—32.8% of voters approved multiple candidates (avg 1.4 approvals/ballot). Non-front-runner supporters were far more likely (84%) to approve multiple candidates vs front-runner supporters (~40%) [Sargent, 2025].

**Best for:** Multi-option selection where simplicity matters and strategic voting risk is acceptable.

### Ranked Choice / Instant Runoff (IRV/RCV)

**Mechanism:** Voters rank options; lowest vote-getter eliminated iteratively; votes transfer to next preference.

**Strengths:**
- Most resistant to strategic voting among common methods [RCVTheory.com analysis]
- Ranking second choice cannot hurt first choice
- 500+ elections across 50+ U.S. jurisdictions
- Handles vote splitting through elimination rounds

**Weaknesses:**
- **Voter confusion is significant:** 16% of Santa Fe voters reported confusion (2018); Hispanic voters significantly more confused than white voters [Social Science Quarterly, 2024]
- **Ballot errors:** ~4.8% of voters improperly mark ballots; error rate 10× higher than non-RCV races [Springer, 2025]
- **Disparate impact:** Higher error rates in areas with more racial minorities, lower income, and lower education [Springer, 2025]
- **Ballot exhaustion:** NYC 2022 mayoral primary saw 15% ballot exhaustion (vs 10% national RCV average) [Gothamist]
- Complex tabulation (iterative elimination) reduces transparency

**Best for:** Multi-candidate elections where strategic resistance outweighs complexity costs.

### Quadratic Voting (QV)

**Mechanism:** Voters have credit budget; casting x votes on an issue costs x² credits. Expresses preference intensity.

**Strengths:**
- Addresses "tyranny of the majority" by allowing intensity expression [Posner & Weyl, 2018]
- Welfare-optimal pricing structure in theory
- Enables minority victories when minorities care more intensely [NBER, 2019]
- Recent research confirms robustness against strategic manipulation [arXiv:2409.06614]

**Weaknesses:**
- **Cognitive load is a barrier:** Traditional QV interfaces impose substantial cognitive burden [arXiv:2503.04114, 2025]
- When voters face quality uncertainty, quadratic costs can discourage informed voting, potentially underperforming linear voting [SSRN:4416748, 2024]
- Not all voters use the quadratic cost structure equally in practice [Harvard, 2017]
- Requires budget management across issues—complex mental model

**Mitigations:**
- Two-phase "organize-then-vote" interface design reduces cognitive burden [arXiv:2503.04114]
- Effects improve with increased participation (asymptotically optimal)

**Implementation advances:** QV-net (2025) introduces decentralized self-tallying QV with ballot secrecy for DAOs—millisecond voting/tallying, no trusted authority [IACR:2025/1146].

**Best for:** Prioritization tasks where intensity matters; requires careful UX design.

### Conviction Voting

**Mechanism:** Continuous voting where support accumulates over time ("signal processing"). Votes can be changed; multiple proposals supported simultaneously with varying intensity.

**Strengths:**
- Asynchronous—no voting periods; real-time visibility into likely outcomes
- Coalition building: see support trends before final outcome
- Reduces attack surface from flash decisions
- Time-weighting resists last-minute manipulation

**Weaknesses:**
- Complex mental model (signal processing vs discrete voting)
- Less mature; primarily used in DAO contexts (1Hive, Commons Stack)
- Requires persistent engagement rather than one-time participation

**Origin:** Developed by Michael Zargham (BlockScience); implemented in Aragon ecosystem [Aragon Forum].

**Best for:** Ongoing resource allocation decisions in engaged communities.

### Liquid Democracy / Delegation

**Mechanism:** Voters can vote directly or delegate to trusted representatives, with delegations potentially transitive.

**Strengths:**
- Combines direct democracy with representative efficiency
- Enables expertise-based delegation
- Flexible—voters can delegate on some issues, vote directly on others

**Weaknesses:**
- **Low real-world participation:** Only 17% of voting tokens delegated on average across 18 crypto projects; participation "relatively low among both voters and delegates" [Stanford, 2024]
- **Power concentration:** Delegation creates power structures enabling manipulation including bribery and control that don't exist in direct democracy [arXiv:2403.07558]
- **Delegation cycle paradox:** Circular chains require careful handling [TU Wien, 2024]
- Complexity of transitive trust

**Positive finding:** Coordination hubs for delegations significantly increase both delegation rates and overall participation [Stanford, 2024].

**Best for:** Ongoing governance with topic-specialized delegates; requires trust infrastructure.

---

## 2. Manipulation Resistance

### Strategic Voting Resistance by Mechanism

| Mechanism | Strategic Resistance | Key Vulnerability |
|-----------|---------------------|-------------------|
| **Plurality** | Low | Vote splitting, lesser-evil voting |
| **Approval** | Low-Medium | Threshold decisions, strategic uncertainty |
| **RCV/IRV** | **High** | Most resistant in comparative studies |
| **Quadratic** | Medium-High | Budget gaming, collusion |
| **Conviction** | Medium | Time-manipulation, last-minute swings |
| **Liquid** | Medium | Delegation bribery, power concentration |

Comparative analysis of eight voting methods found approval voting among the most vulnerable and RCV the least vulnerable to strategic manipulation [Laslier, MPRA:32200].

### Sybil Attack Resistance (Mechanism Design)

Beyond identity verification, mechanism design can limit Sybil impact:

1. **Quadratic costs:** QV's x² pricing naturally diminishes returns from splitting votes across fake identities—100 fake accounts with 1 vote each have less impact than 1 account with 10 votes (10² = 100 credits vs 100 × 1² = 100 credits for same credits but only 10 votes of impact).

2. **Time-weighting (Conviction):** Requiring votes to accumulate over time raises Sybil costs—attackers must maintain fake identities persistently.

3. **Bridging requirements (Community Notes):** Requiring cross-group consensus makes Sybil armies ineffective unless they infiltrate multiple opinion clusters.

4. **Participation history:** Weighting by engagement history (not just identity) raises the cost of fresh Sybils.

**Fundamental constraint:** Research shows the only non-wasteful, symmetric, incentive-compatible, and Sybil-proof direct mechanism is a second-price auction with symmetric tie-breaking—creating tension with broader participation goals [arXiv:2407.14485].

### Community Notes Bridging Algorithm

Community Notes uses **matrix factorization** to find content that receives cross-partisan support:

**Core formula:** `r̂ᵤₙ = μ + iᵤ + iₙ + fᵤ · fₙ`

Where:
- `μ`: Global intercept
- `iᵤ`: User intercept (how picky each rater is)
- `iₙ`: Note intercept (general note quality)
- `fᵤ · fₙ`: Single factor product capturing ideological alignment

The algorithm learns latent ideological positions from rating patterns. Notes that score well with users across ideological divides—not just with like-minded raters—are surfaced [X Community Notes, GitHub].

**Applicability to agenda-setting:**
- **Yes for consensus finding:** Could identify policy concerns that resonate across political divides
- **Limitation:** Only 11.5% of notes reach consensus; 69% show persistent dissensus [arXiv:2510.12559]
- **Caveat:** System "fails by design to moderate the most polarizing content" [arXiv:2506.15168]

**Implementation:** Fully open source with reproducible results from published data [github.com/twitter/communitynotes].

### Research on Online Voting Manipulation

Key findings from 2024 research:

1. **Coercion is underestimated:** Modern threats using blockchains, delay encryption, and trusted hardware exceed traditional coercion-resistance models [IACR:2024/1167]

2. **Fake credentials work but have limits:** 96% user comprehension, but 10% accidentally used fake credentials [arXiv:2404.12075]

3. **SyRA (Sybil-Resilient Anonymous Signatures):** Enables context-specific pseudonyms preventing multiple identities per user while maintaining anonymity [IACR:2024/379]

4. **zkVoting:** Zero-knowledge proofs with nullifiable commitments achieve coercion resistance with end-to-end verifiability [IACR:2024/1003]

---

## 3. Fairness Properties

### Minority Representation

**Problem:** Majority voting can systematically override minority preferences even when minorities care more intensely.

**Mechanisms that help:**

1. **Quadratic Voting:** Mathematically enables "minority victories" when minorities care more—empirically validated on California ballot propositions [NBER:25510]

2. **Proportional allocation:** Rather than winner-take-all, allocate resources/attention proportionally to vote share

3. **Bridging requirements:** Community Notes approach surfaces content supported across divides, not just by largest faction

4. **District magnitude in PR:** In proportional systems, minority representation depends critically on the number of seats per district [Cambridge University Press]

**Key insight from PR research:** In open-list proportional systems, minorities achieve proportional representation only when voters can cast a *limited* number of preferential votes. More preferential votes create a "multiplier effect" favoring majorities [Springer:10.1007/s00355-017-1084-2].

### Avoiding Tyranny of the Majority

| Approach | Mechanism |
|----------|-----------|
| Intensity expression | QV, conviction voting |
| Cross-group consensus | Community Notes bridging |
| Proportional outcomes | Allocate agenda slots proportionally |
| Supermajority thresholds | Require >50% for certain actions |
| Minority veto rights | Constitutional constraints |

### Surfacing Genuine Consensus vs Faction-Driven Outcomes

**Polis approach (vTaiwan):**
- Cluster opinions using PCA/dimensionality reduction
- Identify statements with cross-cluster agreement
- Surface "consensus statements" that majorities across groups endorse
- Result: 80% of 28+ national issues led to government action [info.vtaiwan.tw]

**Community Notes approach:**
- Learn latent ideology from rating patterns
- Surface notes that overcome predicted ideological bias
- Explicitly designed to find "bridging" content

**Design principle:** Consensus ≠ majority. True consensus means acceptance across diverse viewpoints. Algorithms should distinguish "60% of Group A + 10% of Group B support this" from "40% of Group A + 40% of Group B support this."

---

## 4. Practical Considerations

### Voter Comprehension

| Mechanism | Comprehension | Evidence |
|-----------|--------------|----------|
| **Plurality** | Universal | N/A |
| **Approval** | High | Simple "check all" instruction |
| **RCV** | Medium | 16% confusion in Santa Fe; disparate impact on minorities [SSQ, 2024] |
| **Quadratic** | **Low-Medium** | Substantial cognitive burden; requires scaffolded interfaces [arXiv:2503.04114] |
| **Conviction** | Low | "Signal processing" mental model unfamiliar |
| **Liquid** | Medium | Delegation concept familiar; transitivity confusing |

**QV UX finding:** Two-phase "organize-then-vote" interface significantly reduced cognitive burden and improved engagement depth [arXiv:2503.04114, 2025].

### Participation Rates

- **Liquid democracy:** Only 17% token delegation in DAOs; low participation among both voters and delegates [Stanford, 2024]
- **RCV:** Ballot exhaustion (15% NYC) means some votes don't count in final round
- **Approval:** St. Louis saw reasonable engagement; 32.8% used multi-approval feature
- **Quadratic:** No large-scale civic participation data; DAO usage growing

**Key factor:** Coordination tools and hubs significantly increase participation regardless of mechanism [Stanford, 2024].

### Mobile-Friendly Voting UX

Design considerations for mobile:

1. **Approval voting:** Easiest—simple checkbox list
2. **Plurality:** Trivial—single selection
3. **RCV:** Requires drag-and-drop or numbered selection; more complex
4. **Quadratic:** Requires budget visualization; slider or +/- controls; most complex
5. **Conviction:** Requires ongoing engagement interface; dashboard-style

**Recommendation:** Start with mechanisms that work well with simple touch interactions (approval, plurality) before introducing more complex UX requirements.

---

## 5. Implementations and Case Studies

### Gitcoin Quadratic Funding

**Scale:** $294,972 crowdfunded from 28,000 unique donors (GG22, Oct-Nov 2024) [Gitcoin Blog]

**Attack vectors encountered:**
- Sybil attacks: Fake identities claiming matching funds
- Collusion: Coordinated donations to inflate matching
- Round 9 saw "automated collusion and sybil attacks coordinated using bots" [Gitcoin]

**Defenses implemented:**
- **Gitcoin Passport:** Identity verification and uniqueness
- **Network science analysis:** cadCAD models identifying collusive fingerprints
- **Optimality Gap algorithm:** Detects collusion patterns in donation networks
- **Post-round sybil analysis:** Pattern detection for "spraying" donations

**Lesson:** QV/QF requires robust identity layer AND ongoing fraud detection; mechanism alone is insufficient.

### Snapshot.org Governance

**Scale:** Major off-chain governance platform for DAOs

**Findings from research:**
- **Power concentration:** Voting power highly concentrated across Aave, Compound, Lido, Uniswap [arXiv:2407.10945]
- **Minimal quorum size:** Small number of active voters can swing most votes
- **Flash loan attacks:** Time-weighted snapshots needed to prevent governance token manipulation [arXiv:2505.00888]

**Voting types supported:** Single choice, approval, quadratic, ranked choice, weighted [Snapshot docs]

**Lesson:** Token-weighted voting concentrates power regardless of mechanism; Sybil resistance via tokens creates plutocracy.

### Polis / vTaiwan

**Scale:** 200,000 participants on mailing list by 2020; 28+ national issues discussed

**Results:** 80% of cases led to decisive government action [info.vtaiwan.tw]

**Mechanism:**
- Collect open-ended statements
- Users vote agree/disagree/pass on statements
- Cluster users by opinion patterns (PCA)
- Identify cross-cluster consensus statements
- Combine with offline deliberation

**Key success factors:**
- Government commitment to act on results
- Trained "Participation Officers" in government agencies
- Combination of online scale with offline depth

**Lesson:** Mechanism matters less than institutional commitment; Polis succeeds because government acts on results.

### ElectionGuard

**Purpose:** Cryptographic toolkit for end-to-end verifiable elections

**Design:** Modular—separates cryptography from voting system mechanics and UI [Microsoft Research, 2024]

**Deployments:** Wisconsin, California, Idaho, Utah, Maryland (US); France; Switzerland/Denmark

**Features:**
- Homomorphic tallying
- Cast-as-intended verification via confirmation codes
- Runs alongside existing infrastructure

**Relevance:** For civic agenda-setting, provides template for verifiable tallying without replacing existing participation infrastructure.

### Helios / Belenios

**Purpose:** Complete internet voting systems with cryptographic protocols

**Key difference from ElectionGuard:** Full voting systems vs modular toolkit

**Security:** Formal verification identified security trade-offs; automated proofs and attacks published [IEEE, 2021; IACR:2024/915]

**Relevance:** More applicable to formal elections than ongoing civic participation.

---

## 6. Recommendation for v0 Civic System

### The Simplest Mechanism with Reasonable Manipulation Resistance

**Recommendation: Approval Voting with Bridging Analysis**

**Why approval voting for v0:**

1. **Simplicity:** "Check all you support"—universal comprehension, minimal cognitive load
2. **No vote splitting:** Supporting multiple concerns doesn't penalize any
3. **Rich data:** Produces preference intensity signal (more approvals = more engaged voter)
4. **Mobile-friendly:** Simple checkbox list
5. **Low ballot errors:** Significantly lower than RCV
6. **Implementation:** Trivial to build, explain, and audit

**Why add bridging analysis:**

Apply Community Notes-style matrix factorization to identify concerns with cross-group support:
- Cluster users by approval patterns
- Score concerns by cross-cluster support
- Surface "bridging concerns" that resonate across divides
- Distinguish genuine consensus from faction-driven outcomes

This adds manipulation resistance without complicating the voting UX.

**Why not other mechanisms for v0:**

| Mechanism | v0 Rejection Reason |
|-----------|---------------------|
| RCV | Ballot complexity, disparate impact on minorities, 10× error rate |
| Quadratic | High cognitive burden, requires careful UX scaffolding |
| Conviction | Unfamiliar mental model, requires persistent engagement |
| Liquid | Low participation, power concentration, complex trust model |

### What Must Be Right from the Start

1. **Identity layer:** Basic Sybil resistance (one person, one vote) is prerequisite. Can start simple (email verification, phone) but must exist.

2. **Audit trail:** All votes must be logged with timestamps for later analysis. Can't retrofit transparency.

3. **Bridging/consensus definition:** Decide what "cross-group support" means for your context. The Community Notes formula is a starting point.

4. **Commitment to act:** Polis/vTaiwan works because government acts on results. Define clear thresholds for when votes trigger action.

### What Can Be Added Later

1. **Quadratic voting:** Once you have UX learnings and user trust, introduce QV for prioritization within approved items. Use two-phase interface design.

2. **Delegation:** Add optional delegation after establishing direct participation patterns. Requires trust graph infrastructure.

3. **Time-weighting:** If manipulation patterns emerge, add conviction-style time accumulation.

4. **Cryptographic verification:** ElectionGuard-style verification if stakes increase or trust decreases.

5. **Sophisticated Sybil detection:** Network analysis, behavior fingerprinting (Gitcoin Passport model) as user base grows.

### Proposed v0 Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     VOTING LAYER                            │
│                                                             │
│  ┌─────────────┐    ┌──────────────────────────────────┐   │
│  │ Approval    │    │ Bridging Analysis                │   │
│  │ Voting      │───▶│ (Matrix factorization on         │   │
│  │             │    │  approval patterns)              │   │
│  │ "Check all  │    │                                  │   │
│  │  you        │    │ Outputs:                         │   │
│  │  support"   │    │ • Cross-group consensus concerns │   │
│  │             │    │ • Faction-specific concerns      │   │
│  │             │    │ • Opinion cluster visualization  │   │
│  └─────────────┘    └──────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                   ACTION TRIGGERING                         │
│                                                             │
│  Threshold rules:                                           │
│  • Absolute threshold: N total approvals                    │
│  • Bridging bonus: Lower threshold for cross-group items    │
│  • Time decay: Recent votes weighted more                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Implementation Priority

**Phase 1 (MVP):**
- Approval voting UI
- Basic identity (email/phone verification)
- Simple majority threshold for action
- Audit log of all votes

**Phase 2 (Post-launch learning):**
- Bridging analysis on accumulated data
- Cluster visualization for users
- Adjusted thresholds based on observed patterns

**Phase 3 (Scaling):**
- Gitcoin Passport integration for stronger Sybil resistance
- Optional quadratic voting for prioritization
- Delegation features for power users

---

## References

### Academic Papers
- Posner & Weyl (2018). "Quadratic Voting: How Mechanism Design Can Radicalize Democracy." AEA P&P.
- Laslier (2010). "Strategic voting and nomination." MPRA:32200.
- Casella et al. (2019). "Storable Votes and Quadratic Voting." NBER:25510.
- arXiv:2503.04114 (2025). "Two-phase QV interfaces and cognitive load."
- arXiv:2407.14485 (2024). "On Sybil-Proof Mechanisms."
- arXiv:2510.12559 (2024). "Timeliness, Consensus, and Composition: Community Notes on X."
- SSRN:4416748 (2024). "Balancing Power in Decentralized Governance."
- Social Science Quarterly (2024). "The impact of voter confusion in ranked choice voting."
- Springer (2025). "Overvotes, Overranks, and Skips: Mismarked and Rejected Votes in RCV."

### Implementation Resources
- Community Notes algorithm: https://communitynotes.x.com/guide/en/under-the-hood/ranking-notes
- Community Notes code: https://github.com/twitter/communitynotes
- ElectionGuard: https://www.electionguard.vote/
- Polis: https://pol.is/
- vTaiwan: https://info.vtaiwan.tw/
- Gitcoin: https://gitcoin.co/
- Snapshot: https://snapshot.org/

### Case Study Sources
- Gitcoin GG22 Results: https://gitcoin.co/blog/gg22-results-recap
- St. Louis Approval Voting Analysis: https://felixsargent.com/democracy/2025/08/29/st-louis-approval-voting.html
- Stanford Liquid Democracy Study: https://gsbpreserve.stanford.edu/view/4220/

---

*Research compiled February 2026. Sources include academic papers, implementation documentation, and practitioner reports.*

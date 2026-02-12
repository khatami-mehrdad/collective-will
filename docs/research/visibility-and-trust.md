# Visibility and Trust

How do you prove a system did what it claims? Not with promises — with math.

This document surveys the landscape of technologies, standards, and approaches for building verifiable trust into a platform where no single party controls the outcome. The goal is to answer the open questions from the [Governance and Trust](governance-and-trust.md) doc with concrete options.

---

## The Core Problem

Collective Will asks users to trust that:

1. **Their submission was recorded faithfully** — not altered, dropped, or duplicated
2. **The AI organized honestly** — clustering and canonicalization reflect what people actually said
3. **The agenda is legitimate** — what surfaced is what the community actually cares about, not what someone wanted them to care about
4. **Votes were counted correctly** — no stuffing, no suppression, no manipulation
5. **Actions match what was approved** — the letter that was sent is the letter you consented to
6. **No one can tamper after the fact** — the historical record is immutable

Each of these requires a different kind of trust guarantee. Blockchain is one tool. It's not the only one, and often not the best one. Here's what's out there.

---

## Part 1: Proving Things Happened (Verifiable Audit Trails)

The simplest question: how do you prove that a sequence of events happened in a specific order and wasn't changed afterward?

### Transparency Logs (The Certificate Transparency Model)

The most battle-tested approach doesn't use blockchain at all. **Certificate Transparency (CT)** is an internet standard (RFC 9162) that has logged over 2.5 billion TLS certificates since 2013. It uses:

- **Append-only Merkle tree logs** — each entry is hashed into a tree where any tampering is cryptographically detectable
- **Signed timestamps** — the log operator signs a promise that an entry was included at a specific time
- **Inclusion proofs** — anyone can verify that a specific entry exists in the log without downloading the whole thing
- **Consistency proofs** — anyone can verify that the log has only grown (entries were never removed or modified)

The key insight: **you don't need to trust the log operator**. The math lets anyone verify the log's integrity independently. Multiple independent monitors watch the logs and flag inconsistencies.

**Why this matters for Collective Will:** Every submission, canonicalization, cluster assignment, vote, and action could be an entry in a transparency log. Anyone could verify the full pipeline independently. No blockchain needed — just Merkle trees and signatures.

**Open-source implementations:**
- [certificate-transparency-go](https://github.com/google/certificate-transparency-go) (Google)
- [Trillian](https://github.com/google/trillian) — a general-purpose transparency log (powers CT, but designed for any data)
- [Sigstore Rekor](https://github.com/sigstore/rekor) — a transparency log for software supply chains, built on Trillian

### Sigstore / Rekor

**Rekor** is Sigstore's immutable transparency log for recording signed metadata. It's the Certificate Transparency model applied to software supply chains. Entries are indexed by hash, and anyone can verify inclusion proofs via a REST API.

**Why this matters:** Rekor is a proven, open-source, production-grade transparency log with a public instance (rekor.sigstore.dev, 99.5% SLO). Collective Will could use the same data structure — or even the same public infrastructure — to log pipeline events.

### SCITT (IETF Standard for Supply Chain Transparency)

**SCITT (Supply Chain Integrity, Transparency and Trust)** is an emerging IETF standard that generalizes the transparency log pattern. It defines:

- **Signed Statements** — any claim, signed by the issuer
- **Transparency Services** — logs that accept and register statements
- **Receipts** — cryptographic proofs that a statement was registered
- **Registration Policies** — rules governing what a transparency service accepts

SCITT uses COSE (Concise Signing and Encryption) for its cryptographic operations and Merkle trees for its verifiable data structures. It was submitted for standardization in October 2025.

**Why this matters:** SCITT is the emerging standard for exactly this kind of problem — "prove that this thing was said, by this entity, at this time, and hasn't been changed." If Collective Will uses SCITT-compatible receipts, it gets interoperability with a growing ecosystem for free.

### PEAC Protocol (For AI Agent Interactions)

**PEAC (Portable Evidence for Agent Coordination)** is an open standard specifically designed for verifiable records of AI agent interactions. It uses:

- **Ed25519 signatures** — compact (64-byte), verify in microseconds, work offline
- **PEAC-Receipt headers** — signed JWS receipts attached to every interaction
- **Dispute bundles** — ZIP packages containing receipts, policy, and verification reports
- **Policy declarations** — machine-readable terms published at `/.well-known/peac.txt`

PEAC is designed for exactly the scenario where AI agents act on behalf of users and you need to prove what happened. It's Apache-2.0 licensed with TypeScript and Go SDKs.

**Why this matters:** This is purpose-built for auditing AI agent actions — the canonicalization agent's output, the clustering agent's decisions, the action planner's drafts. Every agent interaction gets a signed receipt that anyone can verify independently without a central authority.

---

## Part 2: Proving AI Behaved Correctly (Verifiable AI)

Even if you log everything, how do you know the AI did what it was supposed to? This is the hardest problem — LLMs are non-deterministic by nature.

### The Determinism Problem

LLMs produce different outputs for the same input depending on sampling parameters, hardware, and random seeds. You can't simply re-run a prompt and expect the same output. This means traditional "reproducible computation" approaches don't directly apply.

**Approaches that exist:**

**1. Log everything, verify the trace.** Record the full input (prompt, system instructions, model version, parameters) and the full output. Don't try to reproduce — instead, make the evidence rich enough that anyone can judge whether the output was reasonable given the input. This is what Talk to the City does: every theme links back to individual quotes, so you can check the work.

**2. Structured reasoning traces.** Train or prompt models to produce reasoning in a structured format (tagged steps, named inputs/outputs) that can be audited programmatically. Research on Semi-Structured Reasoning Models (SSRMs) shows this enables automatic audits through domain-specific rules and learned models.

**3. Multiple independent runs.** Run the same input through multiple models or multiple instances and compare outputs. Disagreement flags potential bias or hallucination. The Collective Will architecture doc already mentions "multiple clustering runs with variance analysis" — this is the same idea.

**4. Attestable AI using Trusted Execution Environments (TEEs).** Run AI inference inside hardware secure enclaves (Intel SGX/TDX, AMD SEV-SNP, NVIDIA H100 TEE, AWS Nitro). The enclave produces a cryptographic attestation that the correct model ran on the correct input. This is real and production-grade — used for AI safety benchmarks in 2025.

**5. Zero-knowledge proofs for AI inference (zkML).** Mathematically prove that a specific model produced a specific output without revealing the model weights. Verification takes under 200ms. This is computationally expensive for large models but is being actively developed (JSTprove, Verifiable Fine-Tuning research).

### What's Practical Today

For Collective Will v0, the realistic approach is **#1 + #3**: log everything (inputs, outputs, model version, parameters) to the transparency log, and run critical steps (especially clustering) through multiple independent runs to detect variance. TEEs (#4) become viable when the system scales. zkML (#5) is promising but not ready for production LLM inference.

---

## Part 3: Proving People Are Real (Identity and Sybil Resistance)

The governance doc asks: "What's the right identity model? Full anonymity enables Sybil attacks. KYC kills participation. What's the middle ground?"

This is an active and unsolved research area. Here are the main approaches:

### Proof of Personhood

**Worldcoin / World ID** — Uses custom biometric hardware ("Orbs") to scan irises and generate zero-knowledge proofs of human uniqueness. Privacy-preserving (the proof doesn't reveal identity), but requires trusting specialized hardware and a centralized foundation. Controversial — many people refuse to scan their eyes.

**Gitcoin Passport** — Aggregates identity "stamps" from multiple sources (GitHub, Google, Twitter, ENS, on-chain activity) into a composite score. No single source is sufficient, but the combination provides sybil resistance. More accessible than biometrics, but gameable with enough effort.

**Personhood Credentials (PHC)** — Emerging research recommends time-bound, multi-factor, geo-restricted credentials with periodic re-verification. Government-issued for high-stakes contexts, private issuers for low-stakes. This is the direction the field is moving toward.

### W3C Verifiable Credentials and Decentralized Identifiers

The W3C has standardized two foundational technologies:

**Verifiable Credentials (VCs)** — Cryptographically signed claims (like a digital ID card) that can be verified without contacting the issuer. The holder controls what they share (selective disclosure).

**Decentralized Identifiers (DIDs)** — URIs that enable verifiable identity without centralized registries. The controller proves ownership via cryptographic keys, not a central authority.

Together, these allow a model where: a trusted issuer (government, university, employer, community organization) issues a credential saying "this is a real person in this jurisdiction." The user holds the credential and presents it to Collective Will without revealing their identity — only that they're a verified human in the relevant geography.

### What's the Middle Ground?

The spectrum:

| Approach | Sybil Resistance | Privacy | Participation Barrier | Trust Requirement |
|---|---|---|---|---|
| Full anonymity | None | Maximum | None | None |
| Email verification | Very low | Low | Low | Trust email provider |
| Phone number verification | Low | Low | Low | Trust telco |
| Gitcoin Passport-style (multi-stamp) | Medium | Medium | Medium | Trust attestation sources |
| Government ID (KYC) | High | None | High | Trust government + platform |
| Biometric (Worldcoin) | Very high | Medium (ZKP) | High (hardware) | Trust hardware + foundation |
| Verifiable Credentials (W3C) | Configurable | High (selective disclosure) | Medium | Trust issuer(s) |

For Collective Will, the most promising path may be **tiered identity**:
- **Submit concerns:** low barrier (email only — no phone for Iran pilot due to SIAM surveillance)
- **Vote:** higher bar (verifiable credential from any accepted issuer, or social account age)
- **Trigger action on behalf of others:** highest bar (strong identity + community standing)

This matches the principle that the cost of verification should scale with the power being exercised.

---

## Part 4: Proving Content Hasn't Been Tampered (Provenance)

### C2PA / Content Credentials

The **Coalition for Content Provenance and Authenticity (C2PA)** — founded by Adobe, Microsoft, BBC, Intel, and others — has created an open standard for content provenance. Content Credentials are cryptographically signed metadata that travel with digital content, recording:

- Who created/published it
- When it was created
- What tools were used (including AI)
- What edits were made
- Cryptographic hashes that break if anything is modified

This is now an industry standard used by major platforms and camera manufacturers.

**Why this matters:** When Collective Will's AI agents produce outputs (canonicalized text, cluster summaries, action drafts), those outputs could carry C2PA-style provenance metadata: which model, which version, what input, when, signed by which node. If the output is later changed, the signature breaks.

### Witness.co (Hybrid Provenance)

**Witness** provides a practical hybrid system: submit hashes to an API, which aggregates them into Merkle trees and periodically checkpoints the roots to Ethereum. Users get proofs without needing to interact with blockchain directly.

**Why this matters:** This gives blockchain-grade immutability with web API simplicity. A v0 could submit hashes of every pipeline step to Witness and get verifiable timestamps and inclusion proofs without running any blockchain infrastructure.

---

## Part 5: Verifiable Voting (What Actually Works)

The governance doc asks about voting mechanisms. The research on blockchain voting is sobering.

### The Case Against Blockchain Voting

A 2020 MIT paper ("Going from bad to worse: from Internet voting to blockchain voting") and security analyses of Voatz (the first blockchain voting app used in U.S. federal elections) found:

- Blockchain provides **no protection** against server-side attacks or client-side device compromise
- It introduces **new attack surfaces** that don't exist in paper voting
- There's **no evidence** that online voting increases turnout
- The convenience claim **doesn't justify** the security tradeoffs
- Proprietary systems lack transparency, making independent auditing difficult

The fundamental problem: blockchain can prove that a vote was recorded on the chain, but it can't prove that the vote on the chain matches the voter's actual intent (the "last mile" problem).

### What Does Work: End-to-End Verifiable Voting

**ElectionGuard** (Microsoft Research) is a cryptographic toolkit deployed in actual U.S. elections (Wisconsin, California, Idaho, Utah, Maryland) and civic voting in Paris. It provides:

- **Cast-as-intended verification** — voters get a code to verify their ballot was recorded correctly
- **Ballot challenge** — voters can challenge their ballot after verification to test the system
- **Counted-as-cast verification** — all encrypted artifacts and zero-knowledge proofs are published for independent third-party auditing

ElectionGuard runs alongside existing voting infrastructure. It doesn't replace anything — it adds a cryptographic verification layer.

**Belenios** — verifiable online voting used across France. Encrypts ballots, publishes the ballot box publicly, and produces publicly verifiable tallies. Privacy through encryption, verifiability through public proofs.

**Helios** — over 2 million votes cast. Voters get tracking numbers. Open source. Designed for low-stakes elections (organizations, clubs, student governments) — explicitly not for high-stakes political elections.

### What This Means for Collective Will

Collective Will's voting is closer to Helios territory (organizational/community decisions, not political elections) but with the added constraint that the system must resist strategic manipulation by organized groups. The three most relevant mechanisms from the research:

1. **Encrypted ballots + public tallying** (Belenios/ElectionGuard model) — voters can verify their own vote was counted without revealing it to others
2. **Quadratic voting** — the cost of additional votes grows quadratically, limiting the influence of passionate minorities without silencing them
3. **Conviction voting** — votes accumulate weight over time, rewarding sustained commitment over flash mobs

None of these require blockchain.

---

## Part 6: Putting It Together (A Trust Architecture)

Here's how these pieces could compose into Collective Will's trust stack:

### Layer 1: Append-Only Transparency Log
Every pipeline event (submission, canonicalization, cluster assignment, vote, action, execution receipt) is an entry in a Merkle tree transparency log (Trillian, Rekor, or SCITT-compatible).
- Anyone can verify inclusion and consistency
- No trust in the operator required
- Optionally checkpoint to a blockchain via Witness.co for additional immutability

### Layer 2: Signed Agent Receipts
Every AI agent interaction produces a PEAC-compatible signed receipt: input hash, output hash, model version, timestamp, agent identity.
- Receipts are independently verifiable offline
- Dispute bundles package related receipts for auditing
- C2PA-style provenance metadata travels with generated content

### Layer 3: Multi-Run Verification
Critical pipeline stages (clustering, canonicalization) run through multiple independent agents or models. Results are compared, and variance is published alongside the output.
- Disagreement between runs is flagged publicly
- Users can see confidence levels for every step

### Layer 4: Tiered Identity
Identity requirements scale with the power being exercised:
- Submit: low friction (email only — no phone for Iran pilot)
- Vote: verifiable credential (W3C VC from accepted issuer) or social account age
- Execute: strong identity + community standing

### Layer 5: Verifiable Voting
Votes are encrypted, the ballot box is public, and tallies are cryptographically verifiable (ElectionGuard/Belenios pattern).
- Voters can verify their own vote was counted
- Third parties can verify the tally without seeing individual votes
- No blockchain required

### Layer 6: Public Monitoring
Independent monitors (anyone who wants to run one) watch the transparency log, flag anomalies, and publish reports. The system's credibility comes from the ability of outsiders to verify it, not from the operators' claims.

---

## Open Questions

- Is SCITT mature enough to adopt now, or should v0 use Trillian/Rekor directly and migrate later?
- Can PEAC receipts scale to the volume of agent interactions in a busy campaign (thousands of submissions being canonicalized simultaneously)?
- What's the minimum viable identity tier for v0? Is phone number verification sufficient to start, with verifiable credentials added later?
- TEEs are production-ready for AI inference — but do we need them for v0, or is "log everything + multi-run" sufficient trust for early users?
- How do we make the transparency log understandable to non-technical users? A raw Merkle tree is mathematically trustworthy but socially opaque.
- Which verifiable voting implementation is the best fit? Helios is simplest, ElectionGuard is most proven, Belenios is most privacy-preserving.
- Should the transparency log be hosted (like Rekor's public instance) or self-hosted per community? What are the trust tradeoffs?
- How do verifiable credentials interact with the privacy requirement? Can a user submit a sensitive concern with a VC that proves "real person in California" without revealing who they are?

---

## References

- [Certificate Transparency](https://certificate.transparency.dev/) — the original transparency log model
- [Trillian](https://github.com/google/trillian) — general-purpose transparency log
- [Sigstore Rekor](https://github.com/sigstore/rekor) — transparency log for software supply chains
- [SCITT Architecture](https://datatracker.ietf.org/doc/html/draft-ietf-scitt-architecture-22) — IETF standard for signed statement transparency
- [PEAC Protocol](https://peacprotocol.org/) — verifiable interaction records for AI agents
- [C2PA](https://c2pa.org/) — content provenance and authenticity standard
- [Witness.co](https://docs.witness.co/) — hybrid provenance with blockchain checkpoints
- [ElectionGuard](https://www.electionguard.vote/) — end-to-end verifiable election cryptography
- [Belenios](https://www.belenios.org/) — verifiable online voting
- [Helios](https://vote.heliosvoting.org/) — open-source verifiable elections
- [W3C Verifiable Credentials](https://www.w3.org/TR/vc-data-model/) — cryptographic credential standard
- [W3C Decentralized Identifiers](https://www.w3.org/TR/did-1.1/) — decentralized identity standard
- [Gitcoin Passport](https://docs.passport.xyz/) — sybil resistance via attestation aggregation
- [TRUST Framework](https://openreview.net/forum?id=8jlQCOQp5J) — decentralized LLM reasoning auditing
- [Attestable Audits](https://openreview.net/forum?id=o0wbWJnCb1) — TEE-based AI safety verification

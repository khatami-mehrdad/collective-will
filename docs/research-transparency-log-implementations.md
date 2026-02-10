# Research: Transparency Log Implementations

How to build an append-only evidence store that lets anyone verify the pipeline—submissions, AI canonicalization, clustering, voting, actions—without trusting the operator.

This document evaluates concrete implementation options and provides recommendations for implementation order.

---

## Executive Summary

| Approach | Maturity | Ops Complexity | Best For | Recommendation |
|----------|----------|----------------|----------|----------------|
| **SQLite + hash chain** | High | Very low | v0 prototype | Start here |
| **Git as audit log** | High | Low | Small teams, markdown-native | Good v0 alternative |
| **Trillian Tessera** | Medium | Medium | Production at scale | v1 target |
| **Rekor (Sigstore)** | High | Low (hosted) | Supply chain, hosted option | Consider for v1 |
| **SCITT** | Low | Unknown | Standards-compliant future | Wait for RFC |
| **Kafka** | High | High | Event streaming, not audit | Not recommended |
| **Witness.co anchoring** | Medium | Very low | Blockchain timestamps | Add to any approach |

**Recommended path:**
1. **v0:** SQLite + hash chain (or Git), with Witness.co anchoring
2. **v1:** Migrate to Trillian Tessera when scale requires it
3. **v2:** Adopt SCITT when it becomes an RFC

---

## Part 1: Transparency Log Implementations

### 1.1 Google Trillian (Original)

**Status:** Maintenance mode. Google recommends Trillian Tessera for new deployments.

**Production track record:**
- Powers Certificate Transparency (2.5B+ certificates logged)
- Used by Sigstore Rekor, Go module checksum database
- Multiple production storage backends (MySQL, PostgreSQL, CloudSpanner)

**Operational complexity:** High
- Microservice architecture with multiple components
- Requires master election (etcd or Chubby)
- Needs separate sequencer, signer, and log server processes
- 4-500 entries/s on MySQL, 1000+ entries/s on CloudSpanner

**API/SDK quality:** Good
- Go SDK is comprehensive
- gRPC and REST APIs
- Well-documented but complex

**Verdict:** Don't start here. The maintenance-mode status and operational complexity make it unsuitable for a small team. Go to Tessera instead.

---

### 1.2 Trillian Tessera (Recommended for Production)

**Status:** Alpha (December 2024), actively developed. This is Google's recommended path for new transparency logs.

**Key improvements over Trillian v1:**
- **Library, not microservice** — embed directly into your application
- **Tile-based API** — static, cacheable responses reduce infrastructure demands
- **Multiple write frontends** — run concurrent frontends on the same storage for HA
- **Native multi-environment** — AWS, GCP, POSIX filesystem, MySQL
- **Fast sequencing** — entries get durable sequence numbers in seconds

**Operational complexity:** Medium (much lower than v1)
- Single binary deployment possible
- No master election required
- Can run on local filesystem for development

**Storage backends:**
- AWS S3 + DynamoDB (production)
- GCP Cloud Storage + Spanner (production)
- MySQL (production)
- POSIX filesystem (development/small scale)

**Inclusion/consistency proofs:** Yes, full Merkle tree support with efficient tile-based proof generation.

**Query capabilities:** Limited to index-based lookups. Not a database—you need a separate index for searching by content.

**Performance:** Designed for high throughput. Specific numbers TBD as it matures.

**Verdict:** Target for v1. Best balance of production-readiness, operational simplicity, and proper cryptographic guarantees. Wait for it to stabilize beyond alpha before betting on it.

---

### 1.3 Sigstore Rekor

**Status:** Production. Public instance at rekor.sigstore.dev with 99.5% SLO.

**What it is:** A transparency log for software supply chain, built on Trillian. Records signed metadata about software artifacts.

**Adaptability for Collective Will:**
- Designed for software signatures, not arbitrary data
- Entry types are schema-constrained (HashedRekord, JAR, RPM, etc.)
- You could use the HashedRekord type for arbitrary hashes, but it's a fit-in-a-box approach
- The public instance is free but focused on open-source software

**Operational options:**
1. **Use public instance** — zero ops, but schema constraints and software-focused governance
2. **Self-host** — full Trillian deployment complexity returns

**API/SDK quality:** Excellent
- REST API with OpenAPI spec
- Go, Python clients
- rekor-cli for command-line usage
- Well-documented

**Inclusion/consistency proofs:** Yes, full CT-style proofs via API.

**Query capabilities:**
- Search by hash, public key, email
- Real-time event stream via GCP Pub/Sub
- BigQuery dataset for bulk analysis

**Verdict:** Consider if you want a hosted transparency log with zero ops burden. The software supply chain framing is awkward but workable. Better to use Tessera if you're willing to run infrastructure.

---

### 1.4 SCITT (IETF Standard)

**Status:** Nearing RFC. The architecture document (draft-ietf-scitt-architecture-22) is in the RFC Editor Queue. The reference APIs (draft-ietf-scitt-scrapi-06) are in Working Group Last Call as of December 2025.

**What it is:** A standard for "Signed Statements" registered in "Transparency Services" with "Receipts" as proofs. Uses COSE for signatures and Merkle trees for verification.

**Current implementations:**
- **DataTrails** — production service, compatible with draft-10. Offers:
  - Verified third-party replication of logs
  - Non-falsifiability, non-repudiation, provability guarantees
  - GxP compliance features (ALCOA principles)
  - GitHub Action for CI/CD integration
- **Microsoft SCITT** — reference implementation, experimental

**Maturity assessment for 2026:**
- Architecture spec is almost an RFC (in editor queue)
- APIs spec still in working group review
- DataTrails is production but tied to specific draft versions
- Interoperability between implementations is improving
- New drafts emerging for AI and algorithmic trading audit trails (draft-kamimura-scitt-refusal-events, draft-kamimura-scitt-vcp)

**What's standardized:**
- REST API for registration and receipt retrieval
- COSE-based signed statements
- Merkle tree structure for inclusion proofs
- Optional key discovery and artifact repository integration

**Verdict:** Getting closer to ready. The architecture RFC is imminent (early-mid 2026). Still not recommended for v0 due to API flux, but timeline has accelerated. Design SCITT-compatible structures now; expect viable adoption by late 2026 or early 2027.

---

### 1.5 Plain Git as Append-Only Log

**Status:** Mature (Git is battle-tested), but not designed for this purpose.

**How it works:**
- Each commit is content-addressed by SHA-1 (SHA-256 in newer Git)
- Commit history forms a hash chain
- Signed commits provide non-repudiation
- Branches can model different log streams

**Advantages:**
- Zero new infrastructure—everyone has Git
- Built-in tooling for history, diff, blame
- Human-readable if you use markdown/JSON
- GitHub/GitLab provide hosting, collaboration, CI/CD
- RFC3161 timestamp chaining can add legal-grade provenance

**Limitations:**
- **Not a Merkle tree** — Git's DAG structure doesn't support efficient inclusion proofs for arbitrary entries
- **Rewriting history is possible** — force-push can alter the log (mitigated by branch protection, signed commits, and external witnesses)
- **No native consistency proofs** — you can verify the chain, but not prove "nothing was removed" cryptographically
- **Scale** — works for thousands of entries, struggles at millions
- **Query** — no indexing beyond filename/grep

**When it's sufficient:**
- Small communities (< 10k submissions/month)
- When human readability matters more than cryptographic proofs
- When you can trust GitHub/GitLab (or self-host) not to collude with operators
- When external witnesses (Witness.co) provide the tamper-evidence

**Implementation pattern:**
```
evidence/
  submissions/
    2026-02-10/
      abc123.json  # { original_text, timestamp, user_hash }
  canonicalized/
    2026-02-10/
      abc123.json  # { input_hash, output, model_version, ... }
  votes/
    2026-02-10-agenda-001/
      tally.json   # { votes, merkle_root, ... }
```

Each commit is signed. Each file includes the hash of its predecessor (if chained) or the hash of its input (if derived).

**Verdict:** Excellent for v0. Simple, human-readable, and good enough with external anchoring. Migrate to Tessera when you need proper Merkle proofs at scale.

---

### 1.6 Kafka-Style Append-Only Logs

**Status:** Production-grade for event streaming. Not designed for audit/transparency.

**What Kafka provides:**
- Append-only topic partitions
- Immutable records (once written, can't change)
- Consumer offsets for replay
- High throughput (millions of messages/second)
- Retention policies (time-based or size-based deletion)

**What Kafka lacks:**
- **No Merkle tree** — can't generate inclusion/consistency proofs
- **No cryptographic tamper evidence** — if a broker is compromised, records can be altered
- **Retention deletes old data** — not truly immutable
- **No built-in signing** — you'd need to add this layer
- **Operational complexity** — ZooKeeper (or KRaft), broker clusters, replication

**When to use Kafka:**
- As the *transport layer* feeding a transparency log
- For real-time event streaming between services
- When you need high throughput and can add verification elsewhere

**Verdict:** Wrong tool for the job. Kafka is excellent at event streaming but provides none of the cryptographic properties needed for a transparency log. Don't use it as your evidence store—use it to feed events into one.

---

## Part 2: Blockchain Anchoring Options

### 2.1 Why Anchor to Blockchain?

A transparency log proves internal consistency (nothing was removed or reordered). Blockchain anchoring adds:

- **External timestamp** — a third party (the blockchain) attests when a state existed
- **Immutability beyond the operator** — even if the operator colludes with all their infrastructure providers, they can't alter checkpoints on-chain
- **Public verifiability** — anyone can check the anchor without trusting the operator

The pattern: periodically post the Merkle root (or signed tree head) to a blockchain. If the log is later tampered with, the root won't match.

### 2.2 Witness.co (Recommended)

**What it is:** A hybrid Web2/Web3 service that aggregates hashes into Merkle trees and checkpoints roots to Ethereum.

**How it works:**
1. Submit a hash to the Witness API
2. Witness batches hashes into a Merkle tree
3. Periodically, Witness posts the Merkle root to Ethereum
4. You get an inclusion proof linking your hash to the on-chain root

**Advantages:**
- **Simple API** — REST endpoint, TypeScript/Solidity SDKs
- **Batching amortizes cost** — you don't pay per-hash; Witness handles the on-chain transaction
- **Multi-chain support** — Ethereum mainnet, L2s, potentially others
- **Recent funding ($58M)** — actively developed, not a side project

**Cost:** Pricing not publicly detailed, but the batching model means individual hashes are cheap. Order of magnitude: $0.01–$0.10 per hash depending on volume.

**Verification:** Anyone can verify a proof against the on-chain root without trusting Witness.

**Verdict:** Best option for anchoring. Zero blockchain infrastructure, simple API, amortized costs.

---

### 2.3 Direct Ethereum Writes

**What it is:** Post Merkle roots directly to an Ethereum smart contract.

**Costs (February 2026):**

| Network | Typical tx cost | Notes |
|---------|-----------------|-------|
| Ethereum L1 | $1–5 | Lower than 2024 due to EIP-4844 blobs |
| Arbitrum | $0.09–0.27 | Optimistic rollup, ETH transfer to swap range |
| Optimism | $0.09–0.18 | Optimistic rollup, ETH transfer to swap range |
| Base | $0.05–0.15 | Optimistic rollup (Coinbase), among lowest |
| zkSync Era | $0.05–0.15 | ZK rollup |
| Polygon zkEVM | $0.05–0.15 | ZK rollup |

*Note: L2 costs have dropped significantly since 2024. Both Arbitrum and Optimism remain "throttled while in beta"—fees may decrease further.*

**Frequency tradeoffs:**

| Frequency | Cost/month (L2) | Latency to finality |
|-----------|-----------------|---------------------|
| Every hour | $65–130 | 1 hour |
| Every 6 hours | $11–22 | 6 hours |
| Daily | $2.70–5.40 | 24 hours |
| Weekly | $0.40–0.80 | 7 days |

**Advantages:**
- Maximum decentralization
- No intermediary (like Witness)
- Full control over timing

**Disadvantages:**
- Requires running blockchain infrastructure (or trusting an RPC provider)
- Gas cost volatility
- Operational complexity (key management, transaction monitoring)

**Verdict:** Use if you need maximum trustlessness. Otherwise, Witness.co is simpler.

---

### 2.4 Chainlink (Oracle Pattern)

**What it is:** Use Chainlink oracles to post data to smart contracts.

**Relevance:** Chainlink is designed for bringing external data *into* smart contracts (price feeds, API results). Posting Merkle roots is the reverse direction—you're publishing data *from* your system.

**Verdict:** Not the right tool. Use direct writes or Witness.co.

---

### 2.5 Periodic vs. Continuous Anchoring

**Periodic anchoring (recommended for v0):**
- Checkpoint daily or when a significant event occurs (voting round closes, batch of submissions processed)
- Lower cost, simpler ops
- Acceptable latency for civic applications (users don't need sub-minute finality)

**Continuous anchoring:**
- Every entry anchored immediately
- Higher cost, more complex
- Needed for high-stakes real-time applications (financial trading, not civic deliberation)

**Verdict:** Daily anchoring is sufficient for Collective Will. The transparency log provides real-time verification; blockchain adds long-term immutability.

---

## Part 3: Simpler Alternatives

### 3.1 SQLite + Hash Chain (Recommended for v0)

**How it works:**
```sql
CREATE TABLE evidence (
  id INTEGER PRIMARY KEY,
  timestamp TEXT NOT NULL,
  event_type TEXT NOT NULL,
  payload TEXT NOT NULL,  -- JSON
  payload_hash TEXT NOT NULL,  -- SHA-256 of payload
  prev_hash TEXT NOT NULL,  -- hash of previous row
  signature TEXT NOT NULL  -- Ed25519 signature of (payload_hash || prev_hash)
);

-- Constraint: signature must verify
-- Constraint: prev_hash must match previous row's computed hash
```

**Verification:**
```python
def verify_chain(db):
    prev = "genesis"
    for row in db.execute("SELECT * FROM evidence ORDER BY id"):
        expected_prev = prev
        computed_hash = sha256(row.payload)
        if row.prev_hash != expected_prev:
            raise TamperDetected(f"Row {row.id}: prev_hash mismatch")
        if row.payload_hash != computed_hash:
            raise TamperDetected(f"Row {row.id}: payload_hash mismatch")
        if not verify_signature(row.signature, row.payload_hash + row.prev_hash):
            raise TamperDetected(f"Row {row.id}: invalid signature")
        prev = sha256(row.payload_hash + row.prev_hash + row.signature)
    return True
```

**Advantages:**
- Zero dependencies beyond SQLite
- Single file, easy to backup and replicate
- Full SQL query capabilities
- Human-readable with standard tools
- Trivial to implement in any language

**Limitations:**
- **Linear verification only** — must verify the entire chain, O(n)
- **No efficient inclusion proofs** — can't prove "entry X is in the log" without providing the whole chain up to X
- **No consistency proofs** — can't prove "the log only grew" without comparing full chains

**When linear verification is acceptable:**
- Small logs (< 100k entries) where full verification takes seconds
- When you publish the full log publicly (anyone can download and verify)
- When external anchoring (Witness.co) provides the tamper-evidence shortcut

**Verdict:** Excellent for v0. The lack of Merkle proofs matters less when the log is small and publicly downloadable. Add Witness.co anchoring for external verification.

---

### 3.2 Content-Addressed Storage (Git Model)

**How it works:**
- Every object is stored by its hash: `objects/ab/cdef123...`
- References (like Git branches) point to root hashes
- Changing any content changes its hash, breaking all references

**Implementations:**
- Git itself
- IPFS (distributed, with CIDs)
- Custom directory structure

**IPFS considerations:**
- Content-addressed by design (CIDs)
- Distributed storage and retrieval
- Public by default (privacy requires encryption)
- Pinning required for persistence (data can be garbage collected)
- No built-in Merkle log structure (you'd build it on top)

**Verdict:** Git is simpler for a single-operator system. IPFS adds complexity without clear benefit for Collective Will's use case. Consider IPFS if you want fully decentralized storage, but it's not necessary for v0.

---

### 3.3 When Is "Simple" Sufficient?

**Use hash chains (no Merkle tree) when:**
- Log size is small enough to verify in full (< 100k entries, ~1s verification)
- You can publish the entire log publicly
- External anchoring provides tamper evidence
- You don't need to prove individual entry inclusion to third parties

**Use Merkle trees when:**
- Log size is large (millions of entries)
- You need to prove inclusion without revealing the whole log
- Third parties need to verify specific entries efficiently
- You need consistency proofs between log states

**The threshold:**
- **< 10k entries:** Hash chain is fine, full verification takes milliseconds
- **10k–100k entries:** Hash chain still works, verification takes seconds
- **100k–1M entries:** Consider Merkle tree, verification without proofs becomes slow
- **> 1M entries:** Merkle tree required for practical verification

**For Collective Will:**
- A community with 1,000 submissions/month generates ~12k entries/year
- A very active community (10,000 submissions/month) generates ~120k entries/year
- Hash chains are sufficient for the first 1-2 years of most deployments
- Plan the migration to Merkle trees, but don't build it prematurely

---

## Part 4: SCITT Assessment (2026)

### Is SCITT Ready for Production?

**Not yet, but getting close.** Here's the detailed assessment:

| Factor | Status | Notes |
|--------|--------|-------|
| Specification stability | Improving | Architecture in RFC Editor Queue |
| RFC status | Imminent | Architecture RFC expected early-mid 2026 |
| Interoperability | Untested | Implementations tied to specific draft versions |
| Production deployments | Limited | DataTrails only, proprietary |
| SDK availability | Minimal | No widely-adopted open-source SDK |
| Community | Growing | New drafts for AI audit trails (draft-kamimura-scitt-refusal-events) |

### What Implementations Exist?

1. **DataTrails** — Commercial service, production-grade, SCITT-compatible
   - Verified log replication
   - GitHub Action for CI/CD
   - Focused on supply chain and compliance (GxP)
   - Pricing not public

2. **Microsoft SCITT** — Reference implementation
   - Experimental, not production-ready
   - Useful for understanding the spec

3. **IETF reference code** — Spec examples
   - Not a deployable implementation

### Migration Path from Simpler Approaches

The good news: SCITT's data model is straightforward:

```
Signed Statement = {
  issuer: "did:example:collective-will",
  content_type: "application/json",
  payload: { ... your evidence ... },
  signature: COSE_Sign1
}

Receipt = {
  transparency_service: "...",
  inclusion_proof: [ ... merkle path ... ],
  signed_tree_head: { ... }
}
```

**To prepare for SCITT:**
1. Structure your evidence entries as JSON with a consistent schema
2. Sign entries with Ed25519 (COSE-compatible)
3. Store inclusion proofs when you generate them
4. When SCITT stabilizes, wrap your entries in SCITT Signed Statements and register with a Transparency Service

**Estimated timeline:**
- SCITT Architecture RFC: Early-mid 2026 (in editor queue now)
- SCITT APIs RFC: Late 2026
- Mature open-source implementations: Late 2026-2027
- Safe to adopt: Late 2026 for early adopters, 2027 for conservative deployments

**Verdict:** SCITT timeline has accelerated significantly. Design SCITT-compatible structures now. Consider adoption once both RFCs are published and at least one non-DataTrails open-source implementation matures.

---

## Part 5: Practical Recommendations

### v0: Start Simple (Now)

**Evidence store:**
- SQLite database with hash chain
- Each entry: `{ id, timestamp, event_type, payload_json, payload_hash, prev_hash, signature }`
- Single table, append-only by application logic
- Full chain verification on startup and periodically

**Anchoring:**
- Integrate Witness.co
- Submit `signed_tree_head` (latest entry's hash + signature) daily
- Store Witness proofs alongside local log

**Export:**
- JSON Lines dump of entire log for public download
- Anyone can verify the chain locally

**Implementation effort:** 1-2 days for core, 1 day for Witness integration.

**Limitations you accept:**
- No efficient inclusion proofs (must provide chain up to entry)
- Verification is O(n) (acceptable at small scale)
- Trust model: trust Witness.co for timestamps, verify chain locally

---

### v1: Add Merkle Trees (When Scale Requires)

**When to migrate:**
- Log exceeds 100k entries
- Third parties need to verify individual entries without downloading everything
- You need consistency proofs for federated/replicated deployments

**Target implementation:** Trillian Tessera
- Mature enough by late 2026
- Library integration, not microservice
- MySQL or filesystem backend for small scale
- AWS/GCP backends for production scale

**Migration path:**
1. Export SQLite log to Tessera
2. Tessera assigns sequence numbers, builds Merkle tree
3. Old hashes remain valid (content unchanged)
4. New entries go to Tessera
5. Keep SQLite as read-only archive of pre-migration entries

**Implementation effort:** 1-2 weeks for migration, ongoing ops.

---

### v2: Standards Compliance (When SCITT Matures)

**When to migrate:**
- SCITT becomes an RFC
- Multiple interoperable implementations exist
- Ecosystem benefits (tooling, auditors, integrations) outweigh migration cost

**What changes:**
- Wrap entries as SCITT Signed Statements
- Register with SCITT Transparency Service (self-hosted or third-party)
- Issue SCITT Receipts as proofs

**Benefits:**
- Interoperability with supply chain security ecosystem
- Standardized verification tools
- Regulatory recognition (potentially)

---

### Making It Understandable to Non-Technical Users

The math is trustworthy but opaque. Here's how to bridge that gap:

**1. Visual verification status**
```
✓ Your submission was recorded at 2026-02-10 14:32 UTC
✓ The record is anchored to Ethereum (block 19,234,567)
✓ 847 independent monitors have verified the log
```

**2. "Verify it yourself" links**
- Link to raw JSON of the entry
- Link to the Merkle proof
- Link to the blockchain transaction
- One-click verification tool that runs locally

**3. Explainer content**
- "What does 'anchored to Ethereum' mean?" → short video or diagram
- "Why can't anyone change this?" → hash chain explanation with physical analogy (tamper-evident seals)

**4. Third-party auditor badges**
- Independent organizations run monitors
- Display "Verified by [Auditor Name]" badges
- Link to auditor's verification report

**5. Transparency reports**
- Monthly public report: entries added, anomalies detected (should be zero), verification stats
- Human-readable, not just JSON dumps

---

## Summary: Implementation Order

| Phase | What | Why | Effort |
|-------|------|-----|--------|
| **v0 (now)** | SQLite + hash chain + Witness.co | Simple, sufficient, anchored | 2-3 days |
| **v0.5** | Add JSON export + verification CLI | Public verifiability | 1 day |
| **v1 (100k+ entries)** | Migrate to Trillian Tessera | Merkle proofs, scale | 1-2 weeks |
| **v1.5** | Add third-party monitor infrastructure | Decentralized verification | 1 week |
| **v2 (post-RFC)** | SCITT compliance | Standards, ecosystem | TBD |

---

## References

- [Trillian Tessera](https://github.com/transparency-dev/trillian-tessera) — next-gen transparency log
- [Sigstore Rekor](https://docs.sigstore.dev/logging/overview) — software supply chain transparency
- [SCITT Architecture](https://datatracker.ietf.org/doc/draft-ietf-scitt-scrapi/) — IETF draft standard
- [Witness.co](https://docs.witness.co/) — hybrid blockchain anchoring
- [Certificate Transparency](https://certificate.transparency.dev/) — the original transparency log
- [RFC 9162](https://rfc-editor.org/rfc/rfc9162.html) — Certificate Transparency v2
- [DataTrails SCITT](https://docs.datatrails.ai/) — production SCITT implementation
- [SQLite hash chain tutorial](https://www.viget.com/articles/lets-make-a-hash-chain-in-sqlite/) — implementation reference

---

## Open Questions for Collective Will

1. **What's the expected submission volume for the first pilot?** This determines whether SQLite is sufficient or if we should start with Tessera.

2. **Do we need to prove individual entry inclusion to third parties?** If yes, Merkle trees are needed earlier. If users just verify their own submissions, hash chains suffice.

3. **What's the trust model for Witness.co?** They're a startup with recent funding—acceptable third-party or too much concentration risk?

4. **Should the evidence store be replicated across multiple operators?** If yes, consistency proofs become essential, pushing toward Merkle trees earlier.

5. **What's the litigation/regulatory context?** If evidence might be used in legal proceedings, RFC3161 timestamps and SCITT compliance may matter more than technical optimality.

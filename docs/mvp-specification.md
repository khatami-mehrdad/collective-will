# MVP Specification: Collective Will v0

**Goal**: Surface what Iranians collectively want. Make consensus visible.

**Non-goal for v0**: Action execution against policymakers (deferred to v1).

---

## Table of Contents

1. [Product Overview](#1-product-overview)
2. [System Architecture](#2-system-architecture)
3. [Module Breakdown](#3-module-breakdown)
4. [Data Models](#4-data-models)
5. [User Flows](#5-user-flows)
6. [Technology Stack](#6-technology-stack)
7. [Design Decisions](#7-design-decisions)
8. [What's In / Out of Scope](#8-whats-in--out-of-scope)
9. [Open Questions](#9-open-questions)

---

## 1. Product Overview

### Core Principle: Visibility and Trust

**Trust emerges from visibility, not authority.**

This is the foundational principle of the entire system. Every design decision in this MVP prioritizes:

1. **Visibility** — Users can see exactly what the system did with their input. Every submission, every canonicalization, every cluster assignment, every vote is traceable and publicly verifiable.

2. **Trust through transparency** — The system earns trust not by claiming to be fair, but by proving it. The evidence store, public analytics, and audit trail exist so anyone can verify the pipeline worked correctly.

3. **No hidden decisions** — The AI organizes, but doesn't editorialize. Clustering logic is published. Summaries link to source submissions. Nothing is suppressed.

4. **User sovereignty** — Users can dispute canonicalization, see which cluster their submission joined, verify their vote was counted, and trace the full history.

See [Visibility and Trust](visibility-and-trust.md) for the full technical approach to building verifiable trust.

### How Do I Know You're Not Manipulating the Results?

**Short answer: Don't trust us. Verify.**

| "What if you..." | Our answer |
|------------------|------------|
| ...change my submission? | Your original text is hashed and stored. The canonical form is separate. You can see both and flag if they don't match. |
| ...put me in the wrong cluster? | Every cluster links to its member submissions. Click through and judge for yourself. |
| ...don't count my vote? | Your vote is in the evidence store with a hash. You can verify it's there. The public tally includes it. |
| ...add fake votes? | Every vote links to a verified user. Registration requires email + messaging account verification. |
| ...delete things you don't like? | The evidence store is append-only with a hash chain. Deleting anything breaks the chain — publicly detectable. |
| ...bias the AI clustering? | We run clustering multiple times and flag variance. You can see the "why grouped" explanation for every cluster. |

**What we can't fully prevent in v0** (honest limitations):
- Operators with database access could theoretically insert fake users (solution: federation in v2)
- AI clustering could have subtle bias (solution: multi-run analysis, external audits)
- You have to trust our code is doing what we say (solution: open source, independent review)

**The v0 promise**: We publish everything. If we're cheating, you can catch us.

---

### Core Value Proposition

Iranians don't know what other Iranians want. The regime controls public discourse, diaspora is fragmented, and no neutral platform surfaces collective preferences. Collective Will v0 answers: **"What do we, as a people, actually want?"**

### User Story

> As an Iranian (inside or diaspora), I want to express what I care about and see what others care about, so I can understand where consensus exists and feel connected to a collective voice.

### Success Metrics (v0)

| Metric | Target |
|--------|--------|
| Registered users | >500 in first 3 months |
| Submissions | >200 unique policy concerns |
| Voting participation | >30% of registered users vote |
| Return visits (website) | >20% visit analytics weekly |
| **Trust: Fair representation** | >70% say clustering "fairly represents" their view |
| **Trust: Dispute rate** | <10% of users flag canonicalization as wrong |
| **Trust: Audit usage** | >5% of users check their submission trail |

---

## 2. System Architecture

### High-Level Overview

```
┌──────────────────────────────────────────────────────────────────┐
│                         PUBLIC WEBSITE                            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐               │
│  │  Landing    │  │  Analytics  │  │   User      │               │
│  │  + Subscribe│  │  Dashboard  │  │  Dashboard  │               │
│  └─────────────┘  └─────────────┘  └─────────────┘               │
└──────────────────────────────────────────────────────────────────┘
                              │
                              │ reads from
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│                        CORE DATABASE                              │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐            │
│  │  Users   │ │Submissions│ │ Clusters │ │  Votes   │            │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘            │
│                                                                   │
│  ┌──────────────────────────────────────────────────┐            │
│  │              Evidence Store (audit log)           │            │
│  └──────────────────────────────────────────────────┘            │
└──────────────────────────────────────────────────────────────────┘
                              ▲
                              │ writes to
                              │
┌──────────────────────────────────────────────────────────────────┐
│                      PROCESSING PIPELINE                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐           │
│  │Canonicalize  │─▶│   Cluster    │─▶│    Agenda    │           │
│  │   Agent      │  │    Agent     │  │   Builder    │           │
│  └──────────────┘  └──────────────┘  └──────────────┘           │
└──────────────────────────────────────────────────────────────────┘
                              ▲
                              │ receives messages
                              │
┌──────────────────────────────────────────────────────────────────┐
│                    MESSAGING GATEWAY                              │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                    OpenClaw Gateway                       │   │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐                   │   │
│  │  │WhatsApp │  │Telegram │  │ Signal  │                   │   │
│  │  │ Channel │  │ Channel │  │ Channel │                   │   │
│  │  └─────────┘  └─────────┘  └─────────┘                   │   │
│  └──────────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────┘
                              ▲
                              │
                           USERS
                    (WhatsApp primary)
```

### Module Boundaries

| Module | Responsibility | Talks To |
|--------|---------------|----------|
| **Messaging Gateway** | Receive/send messages, normalize across platforms | Pipeline, Database |
| **Processing Pipeline** | Canonicalize, cluster, build agenda | Database |
| **Core Database** | Store all data, provide queries | All modules |
| **Public Website** | Display analytics, user dashboard | Database (read-only) |
| **Evidence Store** | Append-only audit trail | Embedded in Database |

---

## 3. Module Breakdown

### 3.1 Messaging Gateway

**Purpose**: Interface with WhatsApp (primary), Telegram, Signal. Normalize messages into common format.

**Technology Choice**: **OpenClaw**

**Why OpenClaw**:
- Already has WhatsApp, Telegram, Signal integrations
- Multi-agent architecture fits our pipeline (canonicalizer, clusterer as separate agents)
- Hooks system enables audit logging at every step
- MIT licensed, can fork/extend
- Runs locally — no vendor dependency

**What we customize**:
- Add user verification flow (account linking)
- Add voting interaction flow
- Connect to our database instead of OpenClaw's memory system

**Key Components**:

```
messaging-gateway/
├── agents/
│   ├── intake/              # Receives submissions
│   │   ├── AGENT.md
│   │   └── tools/
│   ├── voting/              # Sends vote prompts, receives votes
│   │   ├── AGENT.md
│   │   └── tools/
│   └── notifications/       # Sends updates to users
│       ├── AGENT.md
│       └── tools/
├── channels/
│   ├── whatsapp/
│   ├── telegram/
│   └── signal/
├── hooks/
│   ├── message_received/    # Log to evidence store
│   └── message_sent/        # Log to evidence store
└── config.yaml
```

**Design Decisions**:

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Primary channel | WhatsApp | 81% of Iranians use VPNs; WhatsApp restrictions lifted Dec 2024 |
| Account verification | Link existing account (not phone number) | Telegram/WhatsApp account age provides some sybil resistance without phone exposure |
| Message format | Plain text, Farsi primary | Lowest friction; LLM handles translation |
| Session management | Stateless per-message | Simpler; state lives in database |

**WhatsApp Business API: Medium Risk (Acceptable)**

WhatsApp Business API requires Meta business verification, but this is **not a high-risk exposure**:
- Meta verification is NOT public record (unlike domain WHOIS)
- Iranian government cannot subpoena Meta (no jurisdiction)
- You're one of millions of Business API users
- No public link between your identity and the project

This is the same risk level as using Anthropic's API — acceptable for MVP.

**Setup options:**
1. **Direct registration** with Meta Business (credit card OK)
2. **Via BSP** like Twilio or MessageBird (one step removed)

See [operational-security.md](operational-security.md) for the full risk-tier framework.

---

### 3.2 Processing Pipeline

**Purpose**: Transform raw submissions into clustered policy positions.

**Sub-modules**:

#### 3.2.1 Canonicalization Agent

**Input**: Raw message text (Farsi or English)
**Output**: Structured `PolicyCandidate`

**Technology**: 
- **Model**: Anthropic Claude or Mistral API (cloud)
- **Data separation**: Only anonymous text sent to API; user IDs and metadata never leave local infrastructure

**Processing Steps**:
1. Store raw submission locally with user link (never sent to cloud)
2. Extract text only, strip all metadata
3. Batch submissions, shuffle order (breaks timing correlation)
4. Send anonymous text to cloud LLM for canonicalization
5. Detect language (Farsi, English, Kurdish, etc.)
6. Extract policy concern(s) — may yield multiple candidates from one message
7. Structure into `PolicyCandidate` schema
8. Compute semantic embedding locally (CPU)
9. Store structured result, link back to original submission

**Prompt Strategy**:
```
You are a policy structuring assistant. Given a user's freeform concern, 
extract structured policy positions WITHOUT editorializing.

Rules:
- Preserve the user's intent exactly
- Do not add opinions or framing
- If the message contains multiple distinct concerns, output multiple candidates
- Flag uncertainty rather than guessing

Output JSON schema: {PolicyCandidate}
```

**Design Decisions**:

| Decision | Choice | Rationale |
|----------|--------|-----------|
| LLM location | Cloud (Anthropic/Mistral) | Simpler infrastructure for MVP; revisit at scale |
| Data separation | Text-only to cloud, user links stay local | Protects user identity while enabling AI processing |
| Multi-issue handling | Split into separate candidates | Research shows joint prediction works; preserves granularity |
| Language | Process in original language, translate for display | Preserves nuance; translation happens at display time |
| Confidence threshold | <0.7 confidence → flag for review | Better to surface uncertainty than silently fail |
| Schema rigidity | Hybrid (required core + flexible extensions) | Balance between consistency and expressiveness |

#### 3.2.2 Clustering Agent

**Input**: Set of `PolicyCandidate` records with embeddings
**Output**: `Cluster` records with summaries

**Technology**:
- **Embeddings**: `multilingual-e5-large` (local, CPU) — stays local for privacy; CPU sufficient for batch processing at MVP scale
- **Clustering**: HDBSCAN (density-based, no need to specify K) — runs locally
- **Summarization**: Anthropic Claude or Mistral API (cloud) — receives only aggregated/anonymized cluster content, not individual submissions

**Processing Steps**:
1. Load all candidates from current cycle (or delta since last run)
2. Compute/retrieve embeddings
3. Run HDBSCAN clustering
4. For each cluster:
   - Generate representative summary via LLM
   - Compute cluster statistics (size, diversity)
   - Link member candidates
5. Store clusters with audit trail

**Run Schedule**: 
- **Batch**: Every 6 hours
- **Future**: Consider incremental updates as volume grows

**Design Decisions**:

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Clustering algorithm | HDBSCAN | No need to specify K; handles varying densities |
| Granularity control | `min_cluster_size=5` | Prevent micro-clusters; can tune based on feedback |
| Multi-run variance | Run 3x, flag high-variance clusters | Detect instability before publishing |
| Small clusters | Show all (no suppression) | Transparency principle; minorities visible |

#### 3.2.3 Agenda Builder

**Input**: Clusters from current cycle
**Output**: Voting agenda (subset of clusters)

**Logic for v0**: Simple — all clusters above minimum size go to voting. No editorial selection.

**Future considerations**:
- Diversity weighting (geographic, demographic)
- Recency vs. sustained interest
- Topic balancing

---

### 3.3 Core Database

**Purpose**: Single source of truth for all application data.

**Technology Choice**: **PostgreSQL**

**Why PostgreSQL**:
- Mature, reliable, well-understood
- JSONB for flexible schema fields
- Full-text search for Farsi (with `pg_trgm` or external index)
- Extensions: `pgvector` for embedding similarity search
- Easy to backup, replicate, audit

**Schema Overview**:

```sql
-- See Section 4 for full data models

-- Core tables
users
submissions
policy_candidates
clusters
votes
voting_cycles

-- Audit/evidence
evidence_log (append-only)
```

**Evidence Store Implementation**:

For v0, embed evidence store in PostgreSQL:

```sql
CREATE TABLE evidence_log (
    id BIGSERIAL PRIMARY KEY,
    timestamp TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    event_type TEXT NOT NULL,
    entity_type TEXT NOT NULL,
    entity_id UUID NOT NULL,
    payload JSONB NOT NULL,
    hash TEXT NOT NULL,           -- SHA-256 of payload
    prev_hash TEXT NOT NULL,      -- Hash of previous entry (chain)
    
    -- Immutability enforced by:
    -- 1. No UPDATE/DELETE permissions on this table
    -- 2. Trigger that validates hash chain on INSERT
);

-- Index for verification queries
CREATE INDEX idx_evidence_hash ON evidence_log(hash);
CREATE INDEX idx_evidence_entity ON evidence_log(entity_type, entity_id);
```

**Witness.co Integration** (optional for v0):
- Daily job: compute Merkle root of day's evidence entries
- Submit root to Witness.co for Ethereum anchoring
- Store anchor receipt in evidence_log

**Design Decisions**:

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Database | PostgreSQL | Mature, pgvector for embeddings, JSONB flexibility |
| Evidence store | Embedded hash-chain table | Simpler than separate system; sufficient for v0 scale |
| Blockchain anchoring | Optional via Witness.co | External tamper evidence without running infrastructure |
| Embedding storage | pgvector extension | Keep embeddings with data; similarity search built-in |

---

### 3.4 Public Website

**Purpose**: Display analytics, allow subscription, show user dashboard.

**Technology Choice**: **Next.js** (TypeScript)

**Why Next.js**:
- SSR for public analytics (SEO, fast initial load)
- React for interactive dashboards
- API routes for subscription flow
- TypeScript for type safety across data models
- Large ecosystem, easy to find developers

**Pages**:

```
/                           # Landing + subscribe CTA
/analytics                  # Public dashboard (no login required)
/analytics/clusters         # Cluster explorer with drill-down
/analytics/trends           # Time-series of policy interest
/dashboard                  # Logged-in user's personal view
/dashboard/submissions      # My submissions and their fate
/dashboard/votes            # My votes
/verify                     # Email/messaging verification flow
/about                      # Mission, methodology, team
/audit                      # Evidence explorer (technical users)
```

**Analytics Dashboard Components**:

1. **Policy Landscape** — treemap or bubble chart of clusters by size
2. **Top Policies** — ranked list with vote counts
3. **Recent Activity** — stream of anonymized submissions
4. **Demographic Breakdown** — if collected (optional)
5. **Cluster Deep-Dive** — click cluster → see summary, source submissions (anonymized), vote distribution

**Design Decisions**:

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Framework | Next.js (App Router) | SSR for public pages, React for dashboards |
| Styling | Tailwind CSS | Rapid development, consistent design |
| Charts | Recharts or D3 | Flexible, well-documented |
| i18n | next-intl | Farsi RTL support built-in |
| Auth | NextAuth.js with email magic links | Simple, no password management |

**Internationalization**:

- Primary: Farsi (RTL)
- Secondary: English
- UI chrome in both languages
- User-generated content displayed in original language with optional translation toggle

---

### 3.5 Voting Service

**Purpose**: Manage voting cycles, collect votes, tally results.

**Implementation**: Part of Messaging Gateway + Database

**Flow**:
1. Agenda Builder produces voting agenda
2. Voting Service creates `VotingCycle` record
3. Messaging Gateway sends vote prompts to all verified users
4. Users reply with votes (via messaging app)
5. Votes recorded in database
6. Cycle closes (time-based or threshold)
7. Results computed and published

**Voting Mechanism for v0**: **Approval Voting**

- User sees list of policies in current cycle
- User can approve any number (including zero)
- Final score = count of approvals
- Simple, understandable, no strategic complexity

**Message Format**:
```
🗳️ صندوق رای باز است!

این هفته، این سیاست‌ها مطرح شدند:

1. [Policy summary in Farsi]
2. [Policy summary in Farsi]
3. [Policy summary in Farsi]

برای رای دادن، شماره‌های موردنظر خود را بفرستید.
مثال: 1, 3

برای انصراف: "انصراف" بفرستید
```

**Design Decisions**:

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Mechanism | Approval voting | Simple, no vote-splitting, universally understood |
| Cycle duration | 48 hours | Long enough for global participation across timezones |
| Reminders | 1 reminder at 24h remaining | Balance engagement vs. annoyance |
| Minimum participation | No threshold for v0 | Learn what natural participation looks like first |

---

## 4. Data Models

### 4.1 User

```typescript
interface User {
  id: UUID;
  email: string;                    // For notifications, magic links
  emailVerified: boolean;
  
  // Messaging account (primary interaction)
  messagingPlatform: 'whatsapp' | 'telegram' | 'signal';
  messagingAccountId: string;       // Platform-specific ID (not phone number)
  messagingVerified: boolean;
  messagingAccountAge?: Date;       // For sybil scoring
  
  // Metadata
  createdAt: Date;
  lastActiveAt: Date;
  locale: 'fa' | 'en';              // Preferred language
  
  // Trust scoring (for future sybil resistance)
  trustScore: number;               // Computed from signals
  contributionCount: number;        // Approved submissions
  
  // Privacy
  isAnonymous: boolean;             // True = inside-Iran, extra care
}
```

### 4.2 Submission

```typescript
interface Submission {
  id: UUID;
  userId: UUID;
  
  // Content
  rawText: string;                  // Original message
  language: string;                 // Detected language code
  
  // Processing state
  status: 'pending' | 'processed' | 'flagged' | 'rejected';
  processedAt?: Date;
  
  // Audit
  hash: string;                     // SHA-256 of rawText
  createdAt: Date;
  evidenceLogId: number;            // Link to evidence store entry
}
```

### 4.3 PolicyCandidate

```typescript
interface PolicyCandidate {
  id: UUID;
  submissionId: UUID;               // Source submission
  
  // Structured content
  title: string;                    // 5-15 words
  titleEn?: string;                 // English translation
  domain: PolicyDomain;             // Enum: governance, economy, rights, etc.
  summary: string;                  // 1-3 sentences
  summaryEn?: string;
  stance: 'support' | 'oppose' | 'neutral' | 'unclear';
  
  // Extracted entities
  entities: string[];               // Named entities mentioned
  
  // For clustering
  embedding: number[];              // Vector from multilingual-e5-large
  
  // Quality signals
  confidence: number;               // LLM confidence 0-1
  ambiguityFlags: string[];         // e.g., 'sarcasm_possible', 'multi_issue'
  
  // Audit
  modelVersion: string;             // Which model produced this
  createdAt: Date;
  evidenceLogId: number;
}

enum PolicyDomain {
  GOVERNANCE = 'governance',        // System of government, elections
  ECONOMY = 'economy',              // Jobs, inflation, sanctions
  RIGHTS = 'rights',                // Human rights, women's rights
  FOREIGN_POLICY = 'foreign_policy',
  RELIGION = 'religion',            // Role of religion in state
  ETHNIC = 'ethnic',                // Minority rights
  JUSTICE = 'justice',              // Accountability, judiciary
  OTHER = 'other'
}
```

### 4.4 Cluster

```typescript
interface Cluster {
  id: UUID;
  cycleId: UUID;                    // Which clustering cycle
  
  // Content
  summary: string;                  // Representative summary (Farsi)
  summaryEn?: string;
  domain: PolicyDomain;
  
  // Members
  candidateIds: UUID[];             // PolicyCandidates in this cluster
  memberCount: number;
  
  // Clustering metadata
  centroidEmbedding: number[];
  cohesionScore: number;            // How tight is the cluster
  varianceFlag: boolean;            // True if multi-run showed instability
  
  // Voting
  approvalCount: number;            // Updated as votes come in
  
  // Audit
  createdAt: Date;
  evidenceLogId: number;
}
```

### 4.5 Vote

```typescript
interface Vote {
  id: UUID;
  userId: UUID;
  cycleId: UUID;
  
  // Vote content
  approvedClusterIds: UUID[];       // Clusters user approved
  
  // Audit
  createdAt: Date;
  evidenceLogId: number;
  
  // Privacy: votes are pseudonymous
  // User can see their own; aggregates are public
}
```

### 4.6 VotingCycle

```typescript
interface VotingCycle {
  id: UUID;
  
  // Timing
  startedAt: Date;
  endsAt: Date;
  status: 'active' | 'closed' | 'tallied';
  
  // Content
  clusterIds: UUID[];               // Clusters in this cycle's agenda
  
  // Results (populated after close)
  results?: {
    clusterId: UUID;
    approvalCount: number;
    approvalRate: number;           // approvals / total votes
  }[];
  totalVoters: number;
  
  // Audit
  evidenceLogId: number;
}
```

### 4.7 EvidenceLogEntry

```typescript
interface EvidenceLogEntry {
  id: number;                       // BIGSERIAL, monotonic
  timestamp: Date;
  
  eventType: 
    | 'submission_received'
    | 'candidate_created'
    | 'cluster_created'
    | 'cluster_updated'
    | 'vote_cast'
    | 'cycle_opened'
    | 'cycle_closed'
    | 'user_created'
    | 'user_verified';
  
  entityType: string;               // 'submission', 'cluster', etc.
  entityId: UUID;
  
  payload: object;                  // Full snapshot of entity at this point
  
  hash: string;                     // SHA-256(JSON.stringify(payload))
  prevHash: string;                 // Previous entry's hash (chain)
}
```

---

## 5. User Flows

### 5.1 Subscription Flow

```
User visits website
        │
        ▼
   Landing page
   "What do Iranians want? Help us find out."
        │
        ▼
   [Subscribe] button
        │
        ▼
   Enter email
        │
        ▼
   Receive magic link email
        │
        ▼
   Click link → email verified
        │
        ▼
   Prompt: "Connect WhatsApp"
   (Show QR code or deep link)
        │
        ▼
   User messages bot from WhatsApp
        │
        ▼
   Bot verifies account link
   (Checks account ID, age)
        │
        ▼
   ✅ User fully verified
   Bot: "شما آماده‌اید! نگرانی‌های خود را با ما به اشتراک بگذارید."
```

### 5.2 Submission Flow

```
User sends WhatsApp message
"وضعیت اقتصادی خیلی بد است. تورم را کنترل کنید."
        │
        ▼
   Gateway receives message
   → Log to evidence store
        │
        ▼
   Canonicalization Agent
   → Extracts: {
       title: "کنترل تورم",
       domain: "economy",
       summary: "درخواست کنترل تورم و بهبود شرایط اقتصادی",
       confidence: 0.92
     }
   → Log to evidence store
        │
        ▼
   Store PolicyCandidate
   Compute embedding
        │
        ▼
   Reply to user:
   "✅ دریافت شد! نظر شما: «کنترل تورم»
    می‌توانید وضعیت آن را در وبسایت ببینید."
```

### 5.3 Clustering Flow (Batch)

```
Cron trigger (every 6 hours)
        │
        ▼
   Load new PolicyCandidates
   since last run
        │
        ▼
   Compute/retrieve embeddings
        │
        ▼
   Run HDBSCAN clustering
   (on all candidates, not just new)
        │
        ▼
   For each cluster:
   → Generate summary via LLM
   → Compute statistics
   → Log to evidence store
        │
        ▼
   Update cluster table
   (Merge with existing or create new)
        │
        ▼
   Website auto-refreshes
   (Public analytics update)
```

### 5.4 Voting Flow

```
Voting cycle starts (manual or scheduled)
        │
        ▼
   Agenda Builder selects clusters
   → All clusters with size ≥ 5
        │
        ▼
   Create VotingCycle record
   Log to evidence store
        │
        ▼
   For each verified user:
   → Send vote prompt via WhatsApp
        │
        ▼
   User replies: "1, 3, 5"
        │
        ▼
   Gateway parses vote
   → Validate cluster IDs
   → Create Vote record
   → Log to evidence store
        │
        ▼
   Reply: "✅ رای شما ثبت شد!"
        │
        ▼
   (48 hours later)
   Cycle closes
        │
        ▼
   Tally results
   Update cluster approvalCounts
   Log to evidence store
        │
        ▼
   Notify users:
   "نتایج رای‌گیری: [link]"
```

### 5.5 User Dashboard Flow

```
User visits /dashboard
        │
        ▼
   Auth check (magic link session)
        │
        ▼
   Load user's submissions
   For each:
   → Show original text
   → Show canonical form
   → Show which cluster it's in
   → Show cluster's current vote count
        │
        ▼
   Load user's votes
   For each cycle:
   → Show what they voted for
   → Show final results
        │
        ▼
   User can:
   → Flag if canonical form is wrong
   → See full audit trail
```

---

## 6. Technology Stack

### 6.1 Summary

| Layer | Technology | Rationale |
|-------|------------|-----------|
| **Messaging Gateway** | OpenClaw (TypeScript) | Multi-platform, agent architecture, hooks |
| **AI Pipeline** | Python | Better ML ecosystem (transformers, HDBSCAN) |
| **Database** | PostgreSQL + pgvector | Mature, embeddings, JSONB |
| **Website** | Next.js (TypeScript) | SSR, React, good i18n |
| **Hosting** | Single VPS (Hetzner/DigitalOcean) | Simple, cheap, EU-based for GDPR |
| **Cloud LLM** | Anthropic Claude or Mistral API | EU/US providers with strong privacy posture |
| **Embeddings** | multilingual-e5-large (local, CPU) | Stays local for privacy; CPU sufficient at MVP scale |

**LLM Strategy**: Cloud-first for MVP. Local GPU infrastructure deferred until critical mass justifies the complexity. See [Section 7.3](#73-why-cloud-first-for-mvp) for rationale and data separation approach.

### 6.2 Language Choices

**TypeScript** for:
- Messaging Gateway (OpenClaw is TypeScript)
- Website (Next.js)
- Shared type definitions

**Python** for:
- AI Pipeline (canonicalization, clustering)
- Better ecosystem: transformers, sentence-transformers, hdbscan, scikit-learn

**Shared**:
- Data models defined in TypeScript, exported as JSON Schema
- Python validates against same schema
- Database is source of truth

### 6.3 TypeScript Security & Memory Guidelines

TypeScript is chosen for speed-to-MVP, but we treat it as a liability to manage, not a guarantee of safety.

#### Dependency Security

| Practice | Implementation |
|----------|----------------|
| **Minimal dependencies** | Every npm package is a risk; justify each one |
| **Lock versions** | Use `package-lock.json`; pin exact versions |
| **Audit regularly** | Run `npm audit` in CI; block deploys on critical vulns |
| **Avoid deep trees** | Prefer packages with few transitive dependencies |
| **No postinstall scripts** | Disable with `ignore-scripts=true` where possible |

#### Memory Management

| Practice | Implementation |
|----------|----------------|
| **Monitor heap usage** | Use `--max-old-space-size` flag; alert on threshold |
| **Avoid memory leaks** | No global caches that grow unbounded; use WeakMap where appropriate |
| **Stream large data** | Don't load full evidence store into memory; use cursors/pagination |
| **Profile under load** | Test with realistic message volume before launch |
| **Set timeouts** | All external calls (DB, API) have timeouts; prevent hung connections |

#### Secure Coding Practices

| Practice | Implementation |
|----------|----------------|
| **Strict TypeScript** | `"strict": true` in tsconfig; no `any` types |
| **Input validation** | Validate all user input with Zod or similar; never trust messaging payloads |
| **No eval/Function** | Never execute dynamic code |
| **Parameterized queries** | Use query builders (Kysely, Drizzle); no string concatenation for SQL |
| **Secrets management** | Environment variables only; never in code; rotate regularly |
| **Rate limiting** | On all endpoints; prevent abuse |

#### Crypto Operations

| Operation | Approach |
|-----------|----------|
| **Hashing (evidence store)** | Use Node.js built-in `crypto` module (C++ bindings), not npm packages |
| **Future: signatures** | Consider calling Go/Rust microservice for signing operations |
| **Random generation** | Use `crypto.randomUUID()`, not `Math.random()` |

#### Migration Path (If Needed at Scale)

If we reach scale where TypeScript becomes a bottleneck or security liability:

1. **Evidence store service** → Rewrite in Rust (crypto operations, append-only log)
2. **Message routing** → Rewrite in Go (high-concurrency, low-latency)
3. **Keep TypeScript for** → Web frontend, admin interfaces, non-critical paths

Design interfaces now (via API boundaries) so these rewrites are possible without full system redesign.

### 6.4 Monorepo Structure

```
collective-will/
├── packages/
│   ├── types/                    # Shared TypeScript types
│   │   ├── src/
│   │   │   ├── user.ts
│   │   │   ├── submission.ts
│   │   │   ├── cluster.ts
│   │   │   └── index.ts
│   │   └── package.json
│   │
│   ├── database/                 # Database schema, migrations
│   │   ├── migrations/
│   │   ├── schema.sql
│   │   └── package.json
│   │
│   └── shared/                   # Shared utilities
│       ├── src/
│       │   ├── hash.ts
│       │   └── evidence.ts
│       └── package.json
│
├── apps/
│   ├── gateway/                  # OpenClaw-based messaging gateway
│   │   ├── agents/
│   │   │   ├── intake/
│   │   │   ├── voting/
│   │   │   └── notifications/
│   │   ├── channels/
│   │   ├── hooks/
│   │   ├── src/
│   │   │   └── db.ts            # Database client
│   │   ├── config.yaml
│   │   └── package.json
│   │
│   ├── pipeline/                 # Python AI pipeline
│   │   ├── agents/
│   │   │   ├── canonicalize.py
│   │   │   ├── cluster.py
│   │   │   └── agenda.py
│   │   ├── models/
│   │   │   └── embeddings.py
│   │   ├── scheduler.py         # Cron-like job runner
│   │   ├── requirements.txt
│   │   └── pyproject.toml
│   │
│   └── web/                      # Next.js website
│       ├── app/
│       │   ├── page.tsx         # Landing
│       │   ├── analytics/
│       │   ├── dashboard/
│       │   └── api/
│       ├── components/
│       ├── lib/
│       ├── messages/            # i18n strings
│       │   ├── fa.json
│       │   └── en.json
│       └── package.json
│
├── docs/                         # Documentation (existing)
├── docker-compose.yml            # Local development
├── docker-compose.prod.yml       # Production
└── README.md
```

### 6.5 Development Setup

```bash
# Prerequisites
- Node.js 20+
- Python 3.11+
- PostgreSQL 15+
- Docker (optional, for easy setup)

# Quick start
docker-compose up -d postgres
pnpm install
pnpm db:migrate
pnpm dev                          # Starts all services

# Or individually:
pnpm --filter gateway dev
pnpm --filter web dev
cd apps/pipeline && python -m scheduler
```

### 6.6 Production Deployment

**Single-server deployment for v0** (simplicity over scale).

> **📖 See [Infrastructure Guide](infrastructure-guide.md)** for complete step-by-step setup instructions.

**Architecture Summary:**

```
Internet → Cloudflare (DNS/CDN) → Hetzner VPS
                                      │
                    ┌─────────────────┼─────────────────┐
                    │                 │                 │
                    ▼                 ▼                 ▼
                  nginx ──────► web (Next.js)     gateway
                    │               │            (OpenClaw)
                    │               │                 │
                    │               └────────┬────────┘
                    │                        ▼
                    │                    postgres
                    │                   (pgvector)
                    │                        │
                    └──► pipeline ───────────┘
                        (Python AI)
```

**Recommended Setup:**

| Component | Specification |
|-----------|---------------|
| Provider | **Hetzner Cloud** (EU jurisdiction, privacy-friendly) |
| Plan | CX32 (4 vCPU, 8GB RAM, 80GB SSD) |
| Location | Falkenstein or Helsinki (EU) |
| OS | Ubuntu 22.04 LTS |
| Deployment | Docker Compose |

**Why Hetzner over AWS:**
- **EU jurisdiction** (Germany) — strong privacy laws, no US CLOUD Act
- **5-10× cheaper** for equivalent specs
- **Simple pricing** — no surprise bills
- **Sufficient for MVP** — handles <1000 concurrent users easily

**Monthly Cost Breakdown:**

| Item | Cost |
|------|------|
| VPS (Hetzner CX32) | €8.50 (~$9) |
| Backup storage (100GB) | €3 (~$3) |
| LLM API (Anthropic/Mistral) | $5-15 |
| Domain (amortized) | ~$1 |
| DNS/CDN (Cloudflare) | Free |
| SSL (Let's Encrypt) | Free |
| **Total** | **~$20-30/month** |

**Infrastructure Guide covers:**
- [ ] Server provisioning and SSH setup
- [ ] Docker installation and configuration  
- [ ] Domain, DNS (Cloudflare), and HTTPS (Let's Encrypt)
- [ ] Database access methods (SSH tunnel, GUI tools)
- [ ] Automated backup strategy
- [ ] Security hardening (firewall, fail2ban, etc.)
- [ ] Monitoring and alerting

**Future scaling**: When volume exceeds ~10K submissions/month, consider dedicated server with GPU for local LLM inference.

---

## 7. Design Decisions

### 7.0 Visibility and Trust as Non-Negotiable

Every feature in v0 is filtered through one question: **"Can a user verify this?"**

| Principle | Implementation |
|-----------|----------------|
| **Every input is preserved** | Raw submissions stored with hash; never modified |
| **Every AI decision is logged** | Canonicalization, clustering logged to evidence store with model version |
| **Every cluster is traceable** | Click any cluster → see member submissions |
| **Every vote is verifiable** | User can see their vote in dashboard; aggregate tallies are public |
| **No suppression** | All clusters shown, even small ones; no editorial filtering |
| **Public by default** | Analytics visible to anyone; no login wall for transparency |

This is why we have an evidence store, why clustering shows "why these were grouped," why users can flag bad canonicalization, and why the audit explorer exists.

**If we can't make it visible, we don't build it.**

---

### 7.0.1 Contributor Safety and Anonymity

**The problem**: Contributors may have family in Iran. If identity is linked to this project, families could face pressure.

**The principle**: The key is **publicly discoverable** vs **requires subpoena**. Use privacy-focused providers; credit card is fine.

| Vector | Risk | Approach |
|--------|------|----------|
| Regular domain registrar | **High** | Your name in WHOIS — DON'T USE |
| Regular hosting (Hetzner) | **High** | Identity verified — DON'T USE |
| Git commits with real name | **High** | Permanent in history — USE PSEUDONYM |
| GitHub with real identity | **High** | Public profile — USE PSEUDONYM |
| Njalla domain | Medium | WHOIS shows Njalla — credit card OK |
| Njalla/1984.is hosting | Medium | Privacy jurisdiction — credit card OK |
| WhatsApp Business API | Medium | Not public — credit card OK |
| LLM API accounts | Medium | Not public — credit card OK |

**Key insight**: The protection comes from **using privacy-focused registrar/host** (Njalla, 1984.is), not from paying with crypto. Credit card is acceptable.

**MVP requirements**:
- Domain + hosting via Njalla or 1984.is (credit card OK)
- Pseudonymous identity for code (git, GitHub, public comms)
- Regular accounts OK for WhatsApp, LLM APIs, dev tools

See **[Operational Security Guide](operational-security.md)** for implementation details.

**Long-term goal**: Decentralize so project can run without any single identity exposure point.

---

### 7.1 Why OpenClaw for Messaging

| Alternative | Pros | Cons | Decision |
|-------------|------|------|----------|
| **Build custom** | Full control | Months of work on WhatsApp integration | ❌ |
| **OpenClaw** | Ready integrations, agent model fits | Learning curve, may need customization | ✅ |
| **Twilio + custom** | Well-documented API | Higher cost, still need agent logic | ❌ |
| **Matrix/Mautrix** | Open protocol | Worse WhatsApp support | ❌ |

### 7.2 Why PostgreSQL (not specialized stores)

| Alternative | Use Case | Why Not for v0 |
|-------------|----------|----------------|
| **Kafka** | Append-only log | Overkill; hash-chain in Postgres sufficient |
| **MongoDB** | Flexible schema | Postgres JSONB is enough; lose ACID |
| **Pinecone** | Vector search | pgvector sufficient; one less service |
| **Redis** | Caching | Premature optimization |

### 7.3 Why Cloud-First for MVP

| Approach | Infra Cost | LLM Cost (1K submissions) | Complexity |
|----------|------------|---------------------------|------------|
| **GPU server + local LLM** | ~$150-200/mo | ~$0.50 | High (GPU procurement, vLLM setup, model management) |
| **VPS + cloud LLM** | ~$50/mo | ~$5-15 | Low (standard VPS, API calls) |

**Decision**: Cloud-first for MVP. At 500 users / ~200 submissions, API costs are negligible (~$5-15/month). GPU infrastructure complexity is not justified until we reach critical mass.

**When to revisit**: Consider local LLM infrastructure when:
- Submission volume exceeds ~10K/month (cost crossover)
- Privacy requirements escalate (e.g., government partnership)
- Latency becomes critical (real-time processing needed)

#### Data Separation Strategy

The key concern with cloud LLMs is privacy for users inside Iran whose political views could be dangerous if exposed.

**What stays local (never sent to cloud):**
- User identifiers and account information
- Submission-to-user mapping
- Timing metadata that could correlate submissions
- Raw embeddings (computed locally on CPU)

**What goes to cloud LLM (anonymized):**
- Submission text only (stripped of all context)
- Batched and shuffled to break timing correlation
- Cluster summaries (aggregated content, not individual voices)

**Provider selection criteria:**
| Provider | Jurisdiction | Data Handling | Recommendation |
|----------|-------------|---------------|----------------|
| **Anthropic Claude** | US | No training on API data, SOC2 | ✅ Good for MVP |
| **Mistral** | France/EU | EU data residency, GDPR compliant | ✅ Good for MVP |
| **DeepSeek** | China | Unclear data handling | ❌ Avoid for sensitive content |
| **OpenAI** | US | Enterprise tier has strong privacy | ⚠️ Acceptable |

**Implementation pattern:**

```
User submission → Store locally with user_id
                         ↓
              Strip metadata, extract text only
                         ↓
              Batch submissions (every 6 hours)
                         ↓
              Shuffle batch order
                         ↓
              Send anonymous text[] to cloud LLM
                         ↓
              Receive structured PolicyCandidate[]
                         ↓
              Match results back to local submissions
                         ↓
              Store with audit trail
```

**Threat model acknowledgment**: Even anonymized political text in bulk could theoretically be valuable to adversaries, but it represents aggregate opinions without individual identification. The user-to-submission link is the critical secret, and that never leaves local infrastructure.

### 7.4 Why Approval Voting

| Mechanism | Comprehension | Strategic Resistance | Implementation |
|-----------|--------------|---------------------|----------------|
| **Plurality** | Universal | Low | Trivial |
| **Approval** | High | Medium | Simple |
| **Ranked choice** | Medium | High | Complex |
| **Quadratic** | Low-Medium | Medium-High | Complex + needs credits |

**Decision**: Approval voting. Users understand it, implementation is simple, no vote-splitting problem.

### 7.5 Why Batch Clustering (not real-time)

- **Stability**: Clusters don't jump around as users watch
- **Efficiency**: Embedding computation is batched
- **Simplicity**: No complex incremental clustering logic
- **Audit**: Clear cycle boundaries in evidence store

**Tradeoff**: 6-hour delay between submission and cluster assignment. Acceptable for v0.

### 7.6 Why Public Analytics (no login wall)

- **Purpose**: Make collective will visible to everyone
- **Trust**: Transparency builds credibility
- **Reach**: Journalists, policymakers can cite without account
- **Safety**: Aggregated data doesn't expose individuals

### 7.7 Identity: Why No Phone Numbers

Per research: Iranian SIAM system uses phone numbers for surveillance. Platform must not require phone verification.

**What we use instead**:
- Email verification (magic links)
- Messaging account linking (account age is signal)
- Contribution-based trust (approved submissions unlock voting)

---

## 8. What's In / Out of Scope

### In Scope (v0)

| Feature | Notes |
|---------|-------|
| WhatsApp submission intake | Primary channel |
| Telegram intake | Secondary |
| Email verification | Magic links |
| Messaging account verification | Account age check |
| Canonicalization (LLM) | Cloud API (Anthropic/Mistral) with data separation |
| Clustering (HDBSCAN) | Batch every 6 hours |
| Approval voting | Via messaging app |
| Public analytics dashboard | Anyone can view |
| User dashboard | See own submissions/votes |
| Evidence store | Hash-chain in Postgres |
| Farsi + English UI | RTL support |
| Basic audit explorer | For technical users |

### Out of Scope (v0)

| Feature | Why Deferred |
|---------|-------------|
| Action execution | v0 is about visibility, not action |
| Signal integration | Lower priority; add if demand |
| Quadratic/conviction voting | Start simple, iterate |
| Federation/decentralization | Single server sufficient for pilot |
| Blockchain anchoring | Optional; can add later |
| Mobile app | Website + messaging app sufficient |
| AI-generated translations | Start with human review |
| Demographic collection | Privacy concerns; add carefully later |
| Bridging algorithm | Can add after basic voting works |

---

## 9. Open Questions

### Must Resolve Before Build

1. **WhatsApp Business API access** — Need approved business account. What's the lead time? Cost? Alternatives if rejected?

2. **OpenClaw customization scope** — How much modification needed? Fork or plugin? Who maintains?

3. **Cluster summary quality** — Need sample data to test. Where do we get realistic Farsi policy submissions for development?

### Can Resolve During Build

4. **Voting cycle timing** — Start with 48 hours, adjust based on participation patterns.

5. **Cluster granularity** — Start with `min_cluster_size=5`, tune based on feedback.

6. **Trust scoring weights** — Start simple (account age + contributions), refine with data.

### Can Resolve After Launch

7. **Optimal clustering frequency** — 6 hours to start; could go to real-time if needed.

8. **Translation strategy** — LLM auto-translate vs. human review vs. community edit.

9. **Demographic insights** — If/when/how to collect and display.

---

## Appendix A: Evidence Store Events

| Event | Payload |
|-------|---------|
| `submission_received` | `{ submissionId, userId, rawText, hash, timestamp }` |
| `candidate_created` | `{ candidateId, submissionId, title, domain, summary, confidence, modelVersion }` |
| `cluster_created` | `{ clusterId, cycleId, summary, candidateIds, memberCount }` |
| `cluster_updated` | `{ clusterId, candidateIds, memberCount, reason }` |
| `vote_cast` | `{ voteId, userId, cycleId, approvedClusterIds }` |
| `cycle_opened` | `{ cycleId, clusterIds, startsAt, endsAt }` |
| `cycle_closed` | `{ cycleId, results, totalVoters }` |
| `user_created` | `{ userId, email, messagingPlatform }` |
| `user_verified` | `{ userId, verificationType, timestamp }` |

---

## Appendix B: API Endpoints (Website)

```
GET  /api/analytics/clusters          # Public: all clusters with stats
GET  /api/analytics/clusters/:id      # Public: cluster detail + members
GET  /api/analytics/trends            # Public: time-series data
GET  /api/cycles                      # Public: voting cycles
GET  /api/cycles/:id/results          # Public: cycle results

POST /api/auth/subscribe              # Start subscription flow
GET  /api/auth/verify                 # Verify magic link
POST /api/auth/link-messaging         # Initiate messaging link

GET  /api/user/me                     # Authenticated: user profile
GET  /api/user/submissions            # Authenticated: my submissions
GET  /api/user/votes                  # Authenticated: my votes
POST /api/user/flag                   # Authenticated: flag bad canonicalization

GET  /api/evidence/:hash              # Public: verify evidence entry
GET  /api/evidence/chain              # Public: verify hash chain integrity
```

---

## Appendix C: Messaging Commands

| User Input | Bot Response |
|------------|--------------|
| (any freeform text) | Process as submission → confirmation |
| `وضعیت` / `status` | Show pending submissions, active votes |
| `کمک` / `help` | Show available commands |
| `رای` / `vote` | Show current voting agenda (if active) |
| `1, 3, 5` (during voting) | Record vote → confirmation |
| `انصراف` / `skip` | Skip current voting cycle |
| `زبان` / `language` | Toggle Farsi/English |

---

## Appendix D: Milestones

### Milestone 1: Foundation (Week 1-2)
- [ ] Set up monorepo structure
- [ ] PostgreSQL schema + migrations
- [ ] Basic OpenClaw gateway with WhatsApp
- [ ] "Hello world" message round-trip

### Milestone 2: Pipeline (Week 3-4)
- [ ] Canonicalization agent (cloud API with data separation)
- [ ] Embedding computation (local, CPU)
- [ ] HDBSCAN clustering
- [ ] Batch scheduler

### Milestone 3: Website (Week 5-6)
- [ ] Next.js setup with Farsi/English
- [ ] Public analytics dashboard
- [ ] User authentication (magic links)
- [ ] User dashboard

### Milestone 4: Voting (Week 7-8)
- [ ] Voting cycle management
- [ ] Vote collection via WhatsApp
- [ ] Results display
- [ ] Evidence store integration

### Milestone 5: Polish (Week 9-10)
- [ ] End-to-end testing
- [ ] Security review
- [ ] Documentation
- [ ] Soft launch with test users

---

*This specification will evolve. All decisions are documented so they can be revisited as we learn from real usage.*

# Collective Will v0 — Shared Context

Every agent receives this file. It is the ground truth for the project.

---

## What This Project Is

Collective Will surfaces what Iranians collectively want. Users submit concerns via WhatsApp, AI agents organize and cluster them, and the community votes on priorities. Everything is transparent and auditable.

**v0 goal**: Consensus visibility + approval voting. No action execution (deferred to v1).

**Pilot**: Iran (diaspora + inside-Iran).

---

## v0 Frozen Decisions

These are locked. Do not deviate.

| Decision | Rule |
|----------|------|
| **Scope** | Consensus visibility + approval voting only. No action drafting or execution. |
| **Channel** | WhatsApp only (via Evolution API, self-hosted). No Telegram, no Signal. |
| **Canonicalization model** | Claude Haiku (Farsi → structured English) |
| **Embeddings** | Mistral `mistral-embed` API (cloud) |
| **Cluster summaries** | DeepSeek V3.2 API |
| **User-facing messages** | Claude Haiku (Farsi quality) |
| **Clustering** | HDBSCAN (runs locally), `min_cluster_size=5` |
| **Identity** | Email magic-link + WhatsApp account linking. No phone verification, no OAuth, no vouching. |
| **WhatsApp ID storage** | Tokenized via `HMAC(wa_id, SECRET_PEPPER)`. Raw IDs isolated in sealed mapping, stripped from logs/exports. |
| **Submission eligibility** | Verified account + account age >= 48 hours |
| **Vote eligibility** | Verified account + age >= 48h + at least 1 accepted submission |
| **Evidence store** | PostgreSQL append-only hash-chain. No UPDATE/DELETE. |
| **External anchoring** | Optional for v0 (Witness.co daily Merkle root) |
| **Infrastructure** | Njalla/1984.is VPS. Privacy-first. WHOIS hides owner. |

### Abuse Thresholds

| Control | Limit |
|---------|-------|
| Submissions per account per day | 5 |
| Accounts per email domain per day | 3 |
| Burst quarantine trigger | 10+ submissions/hour from one account |
| Vote changes per cycle | 1 |
| Failed verification attempts | 5 per email per 24h, then 24h lockout |

### Dispute Handling

- Users flag bad canonicalization or cluster assignment from their dashboard.
- Operator reviews within 72 hours.
- Disputed items tagged (`dispute_open`, `dispute_resolved`) but never removed or suppressed.
- Resolution logged to evidence store. Resolution is by re-running pipeline, not manual content override.

### Data Retention

| Data | Deletable on user request? |
|------|---------------------------|
| Evidence chain entries | No (chain integrity) |
| Account linkage (email ↔ wa_id mapping) | Yes (GDPR) |
| Tokenized user refs in evidence chain | No (but unlinkable after account deletion) |
| Raw submissions in evidence chain | No (user link severed; text preserved anonymously) |
| Votes | No (pseudonymous; user link severed on deletion) |

---

## Data Models

Implement as Pydantic `BaseModel` subclasses (Python) and SQLAlchemy ORM models for DB.

### User

```
id: UUID
email: str
email_verified: bool
messaging_platform: "whatsapp"
messaging_account_ref: str          # HMAC(wa_id, SECRET_PEPPER)
messaging_verified: bool
messaging_account_age: datetime | None
created_at: datetime
last_active_at: datetime
locale: "fa" | "en"
trust_score: float
contribution_count: int
is_anonymous: bool
```

### Submission

```
id: UUID
user_id: UUID
raw_text: str
language: str
status: "pending" | "processed" | "flagged" | "rejected"
processed_at: datetime | None
hash: str                           # SHA-256 of raw_text
created_at: datetime
evidence_log_id: int
```

### PolicyCandidate

```
id: UUID
submission_id: UUID
title: str                          # 5-15 words
title_en: str | None
domain: PolicyDomain
summary: str                        # 1-3 sentences
summary_en: str | None
stance: "support" | "oppose" | "neutral" | "unclear"
entities: list[str]
embedding: list[float]              # pgvector column
confidence: float                   # 0-1
ambiguity_flags: list[str]
model_version: str
prompt_version: str
created_at: datetime
evidence_log_id: int
```

### PolicyDomain (enum)

```
governance, economy, rights, foreign_policy, religion, ethnic, justice, other
```

### Cluster

```
id: UUID
cycle_id: UUID
summary: str                        # Farsi
summary_en: str | None
domain: PolicyDomain
candidate_ids: list[UUID]
member_count: int
centroid_embedding: list[float]
cohesion_score: float
variance_flag: bool
run_id: str
random_seed: int
clustering_params: dict
approval_count: int
created_at: datetime
evidence_log_id: int
```

### Vote

```
id: UUID
user_id: UUID
cycle_id: UUID
approved_cluster_ids: list[UUID]
created_at: datetime
evidence_log_id: int
```

### VotingCycle

```
id: UUID
started_at: datetime
ends_at: datetime
status: "active" | "closed" | "tallied"
cluster_ids: list[UUID]
results: list[{cluster_id, approval_count, approval_rate}] | None
total_voters: int
evidence_log_id: int
```

### EvidenceLogEntry

```
id: int (BIGSERIAL)
timestamp: datetime
event_type: str                     # submission_received, candidate_created, cluster_created,
                                    # cluster_updated, vote_cast, cycle_opened, cycle_closed,
                                    # user_created, user_verified
entity_type: str
entity_id: UUID
payload: dict                       # JSONB — full entity snapshot
hash: str                           # SHA-256 of payload
prev_hash: str                      # previous entry's hash (chain)
```

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| **Backend** | Python 3.11+, FastAPI, SQLAlchemy (async), Pydantic |
| **Database** | PostgreSQL 15+ with pgvector extension |
| **Website** | Next.js (App Router), TypeScript, Tailwind CSS, next-intl |
| **Dependency mgmt** | uv (Python), pnpm (Node) |
| **Migrations** | Alembic |
| **Testing** | pytest (Python), vitest or jest (TypeScript) |
| **Linting** | ruff (Python), eslint (TypeScript) |
| **Type checking** | mypy strict (Python) |
| **Containerization** | Docker Compose |

---

## Project Directory Structure

```
collective-will/
├── src/
│   ├── __init__.py
│   ├── config.py
│   ├── channels/
│   │   ├── __init__.py
│   │   ├── base.py              # Abstract channel interface
│   │   ├── whatsapp.py          # Evolution API client
│   │   └── types.py             # Unified message format
│   ├── handlers/
│   │   ├── __init__.py
│   │   ├── intake.py            # Receives submissions
│   │   ├── voting.py            # Vote prompts, receives votes
│   │   ├── notifications.py     # Sends updates
│   │   ├── identity.py          # Email magic-link, WhatsApp linking
│   │   ├── abuse.py             # Rate limiting, quarantine
│   │   └── commands.py          # Message command router
│   ├── pipeline/
│   │   ├── __init__.py
│   │   ├── llm.py               # LLM abstraction + router
│   │   ├── privacy.py           # Strip metadata for LLM
│   │   ├── canonicalize.py      # LLM canonicalization
│   │   ├── embeddings.py        # Mistral embed
│   │   ├── cluster.py           # HDBSCAN clustering
│   │   ├── summarize.py         # Cluster summaries
│   │   └── agenda.py            # Agenda building
│   ├── models/
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── submission.py
│   │   ├── cluster.py
│   │   └── vote.py
│   ├── db/
│   │   ├── __init__.py
│   │   ├── connection.py
│   │   ├── evidence.py          # Evidence store operations
│   │   └── queries.py
│   ├── api/
│   │   ├── __init__.py
│   │   ├── main.py              # FastAPI app
│   │   ├── routes/
│   │   │   ├── webhooks.py
│   │   │   ├── analytics.py
│   │   │   ├── user.py
│   │   │   └── auth.py
│   │   └── middleware/
│   │       ├── audit.py
│   │       └── auth.py
│   ├── scheduler.py
│   └── config.py
├── migrations/
│   ├── versions/
│   └── alembic.ini
├── web/                          # Next.js website
│   ├── app/
│   ├── components/
│   ├── lib/
│   ├── messages/
│   │   ├── fa.json
│   │   └── en.json
│   ├── package.json
│   └── tsconfig.json
├── tests/                        # Mirrors src/ structure
│   ├── test_channels/
│   ├── test_pipeline/
│   ├── test_handlers/
│   ├── test_api/
│   └── test_db/
├── docker-compose.yml
├── pyproject.toml
├── .env.example
├── .gitignore
└── README.md
```

---

## What's In Scope (v0)

- WhatsApp submission intake (Evolution API)
- Email magic-link verification
- WhatsApp account linking (HMAC tokenized)
- Canonicalization (Claude Haiku, cloud)
- Embeddings (Mistral embed, cloud)
- Clustering (HDBSCAN, local, batch every 6h)
- Approval voting via WhatsApp
- Public analytics dashboard (no login wall)
- User dashboard (submissions, votes, disputes)
- Evidence store (hash-chain in Postgres)
- Farsi + English UI (RTL support)
- Audit evidence explorer
- Abuse controls (rate limits, quarantine)

## What's Out of Scope (v0)

- Action execution / drafting
- Telegram, Signal
- Phone verification, OAuth, vouching
- Quadratic/conviction voting
- Federation / decentralization
- Blockchain anchoring (required)
- Mobile app
- Demographic collection

---

## Process Rules — EVERY AGENT MUST FOLLOW

1. **Test after every task**: When you finish implementing a task, write unit tests for what you just built. Tests go in `tests/` mirroring `src/` structure. Use pytest (Python) or vitest (TypeScript). Run tests and confirm they pass before moving to the next task.
2. **Type hints everywhere**: All Python functions have type annotations. Run mypy in strict mode.
3. **Pydantic for all models**: All data models are Pydantic BaseModel subclasses. SQLAlchemy models are separate but aligned.
4. **Parameterized queries only**: Use SQLAlchemy ORM. No string concatenation for SQL.
5. **Use `secrets` not `random`**: For any crypto/token generation.
6. **No eval/exec**: Never execute dynamic code.
7. **Ruff for formatting**: Run ruff before finishing.
8. **Never commit secrets**: `.env` is gitignored. No API keys, passwords, or tokens in code.
9. **OpSec**: No real names in commits/comments/code. No hardcoded paths containing usernames. Tokenize WhatsApp IDs with HMAC — never store raw wa_id in core tables or logs.

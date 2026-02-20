# Task: Core Data Models

## Depends on
- `database/01-project-scaffold` (project structure, config)
- `database/02-db-connection` (Base declarative class, session factory)

## Goal
Create SQLAlchemy ORM models and Pydantic schemas for all 6 core tables, plus basic CRUD query functions.

## Files to create/modify

- `src/models/user.py` — User ORM + Pydantic schemas
- `src/models/submission.py` — Submission + PolicyCandidate ORM + Pydantic schemas, PolicyDomain enum
- `src/models/cluster.py` — Cluster ORM + Pydantic schemas
- `src/models/vote.py` — Vote + VotingCycle ORM + Pydantic schemas
- `src/models/__init__.py` — re-export all models
- `src/db/queries.py` — basic CRUD functions

## Specification

### ORM models

Map exactly to the data models in CONTEXT-shared.md. Key details:

**User table**
- `id`: UUID primary key (use `uuid4` default)
- `email`: unique, indexed
- `messaging_account_ref`: unique, indexed (this is the HMAC token, NOT raw wa_id)
- `locale`: default `"fa"`
- `trust_score`: default `0.0`
- `contribution_count`: default `0`
- `is_anonymous`: default `False`

**Submission table**
- `id`: UUID primary key
- `user_id`: foreign key to users
- `hash`: SHA-256 of raw_text, indexed
- `status`: default `"pending"`

**PolicyCandidate table**
- `id`: UUID primary key
- `submission_id`: foreign key to submissions
- `domain`: use Python `enum.Enum` for PolicyDomain
- `embedding`: use pgvector `Vector` column type
- `confidence`: float, 0-1
- `model_version`, `prompt_version`: string, not null

**Cluster table**
- `id`: UUID primary key
- `cycle_id`: foreign key to voting_cycles
- `candidate_ids`: ARRAY of UUIDs (or use association table)
- `centroid_embedding`: pgvector Vector column
- `variance_flag`: boolean, default False
- `clustering_params`: JSONB

**Vote table**
- `id`: UUID primary key
- `user_id`: foreign key to users
- `cycle_id`: foreign key to voting_cycles
- `approved_cluster_ids`: ARRAY of UUIDs

**VotingCycle table**
- `id`: UUID primary key
- `status`: default `"active"`
- `results`: JSONB (nullable, populated after close)

### Pydantic schemas

For each model, create at minimum:
- `Create` schema (input for creating a new record)
- `Read` schema (output for API responses, includes id and timestamps)

Example: `UserCreate(email, locale)`, `UserRead(id, email, email_verified, ...)`

### PolicyDomain enum

```python
class PolicyDomain(str, Enum):
    GOVERNANCE = "governance"
    ECONOMY = "economy"
    RIGHTS = "rights"
    FOREIGN_POLICY = "foreign_policy"
    RELIGION = "religion"
    ETHNIC = "ethnic"
    JUSTICE = "justice"
    OTHER = "other"
```

### CRUD queries (src/db/queries.py)

Basic async functions:
- `create_user(session, data) -> User`
- `get_user_by_email(session, email) -> User | None`
- `get_user_by_messaging_ref(session, ref) -> User | None`
- `create_submission(session, data) -> Submission`
- `get_submissions_by_user(session, user_id) -> list[Submission]`
- `create_policy_candidate(session, data) -> PolicyCandidate`
- `create_cluster(session, data) -> Cluster`
- `create_vote(session, data) -> Vote`
- `create_voting_cycle(session, data) -> VotingCycle`

## Constraints

- Use `pgvector` for embedding columns. The pgvector SQLAlchemy integration requires the `pgvector` Python package.
- UUID columns must use `uuid.uuid4` as default.
- All timestamps use timezone-aware datetimes (`DateTime(timezone=True)`).
- Do NOT store raw WhatsApp IDs in any model. `messaging_account_ref` is always the HMAC token.

## Tests

Write tests in `tests/test_db/test_models.py` covering:
- Each ORM model can be created and saved to the test database
- Pydantic schemas validate correct input and reject invalid input (wrong types, missing required fields)
- PolicyDomain enum accepts valid values and rejects invalid ones
- `get_user_by_email` returns None for nonexistent user
- `get_user_by_messaging_ref` finds user by HMAC token
- Embedding column stores and retrieves a vector correctly (e.g., 1024-dim float list)
- Foreign key constraints work (e.g., Submission with invalid user_id fails)

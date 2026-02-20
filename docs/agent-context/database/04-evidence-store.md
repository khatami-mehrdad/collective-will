# Task: Evidence Store

## Depends on
- `database/02-db-connection` (engine, session, Base)

## Goal
Implement the append-only evidence log with hash-chain integrity, an append function, and a chain verification function.

## Files to create/modify

- `src/db/evidence.py` — EvidenceLogEntry ORM model, append and verify functions
- SQL for the `evidence_log` table, trigger, and indexes (can be inline or in a separate `.sql` file referenced by migrations later)

## Specification

### Table DDL

```sql
CREATE TABLE evidence_log (
    id BIGSERIAL PRIMARY KEY,
    timestamp TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    event_type TEXT NOT NULL,
    entity_type TEXT NOT NULL,
    entity_id UUID NOT NULL,
    payload JSONB NOT NULL,
    hash TEXT NOT NULL,
    prev_hash TEXT NOT NULL,
    
    -- Immutability enforced by:
    -- 1. No UPDATE/DELETE permissions on this table
    -- 2. Trigger that validates hash chain on INSERT
);

CREATE INDEX idx_evidence_hash ON evidence_log(hash);
CREATE INDEX idx_evidence_entity ON evidence_log(entity_type, entity_id);
```

### Hash computation

```python
import hashlib, json

def compute_hash(payload: dict) -> str:
    serialized = json.dumps(payload, sort_keys=True, default=str)
    return hashlib.sha256(serialized.encode()).hexdigest()
```

### Chain trigger (SQL)

On INSERT, validate that `prev_hash` matches the `hash` of the most recent existing entry. For the very first entry, `prev_hash` should be a known genesis value (e.g., `"0"` or `"genesis"`).

### append_evidence() function

```python
async def append_evidence(
    session: AsyncSession,
    event_type: str,
    entity_type: str,
    entity_id: UUID,
    payload: dict,
) -> EvidenceLogEntry:
```

Steps:
1. Get the hash of the last entry (or genesis value if empty)
2. Compute hash of the new payload
3. Insert new row with `prev_hash` = last entry's hash, `hash` = new payload hash
4. Return the created entry

This must be atomic — use a transaction with row-level locking to prevent race conditions on concurrent inserts.

### verify_chain() function

```python
async def verify_chain(session: AsyncSession) -> tuple[bool, int]:
    """Returns (is_valid, entries_checked)."""
```

Iterate through all entries in order. For each entry, verify:
1. `hash` matches `compute_hash(payload)`
2. `prev_hash` matches the preceding entry's `hash`

Return False immediately if any link is broken.

### Event types

Valid event types (enforce via validation):
```
submission_received, candidate_created, cluster_created, cluster_updated,
vote_cast, cycle_opened, cycle_closed, user_created, user_verified
```

## Constraints

- The evidence_log table must NEVER allow UPDATE or DELETE. Enforce this at the database permission level and in application code (no ORM update/delete methods exposed).
- Use `hashlib.sha256` from Python stdlib. Do NOT use `random` module.
- Hash computation must be deterministic — use `sort_keys=True` and `default=str` in JSON serialization.
- Concurrent appends must not corrupt the chain. Use database-level locking.

## Tests

Write tests in `tests/test_db/test_evidence.py` covering:
- Append a single evidence entry — hash and prev_hash are correct
- Append a chain of 5 entries — each prev_hash links to the previous hash
- `verify_chain()` returns True on a valid chain
- `verify_chain()` returns False when a payload is tampered with (modify payload in DB directly for test)
- Genesis entry has prev_hash = "genesis" (or chosen sentinel)
- All valid event types are accepted
- Invalid event type is rejected
- Concurrent appends (two near-simultaneous inserts) maintain chain integrity
- `compute_hash()` is deterministic — same payload always produces same hash

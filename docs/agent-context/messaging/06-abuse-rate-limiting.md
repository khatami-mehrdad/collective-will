# Task: Abuse Controls and Rate Limiting

## Depends on
- `database/03-core-models` (User, Submission models)
- `database/02-db-connection` (session)

## Goal
Implement rate limiting, burst detection, and quarantine logic as reusable functions that handlers call before processing.

## Files to create

- `src/handlers/abuse.py` — rate limiting and quarantine logic

## Specification

### Rate limit checks

All functions are async and take a database session + user/identifier.

```python
async def check_submission_rate(db: AsyncSession, user_id: UUID) -> RateLimitResult:
    """Check if user can submit. Returns allowed=True or reason for denial."""

async def check_domain_rate(db: AsyncSession, email_domain: str) -> RateLimitResult:
    """Check if email domain hasn't exceeded 3 accounts/day."""

async def check_burst(db: AsyncSession, user_id: UUID) -> RateLimitResult:
    """Check if user has > 10 submissions in the last hour. Triggers quarantine."""

async def check_vote_change(db: AsyncSession, user_id: UUID, cycle_id: UUID) -> RateLimitResult:
    """Check if user has already changed their vote this cycle."""
```

### RateLimitResult

```python
class RateLimitResult(BaseModel):
    allowed: bool
    reason: str | None = None    # Human-readable reason if denied
    quarantine: bool = False     # True if this check triggers quarantine
```

### Thresholds (from frozen decisions)

| Check | Threshold | Window |
|-------|-----------|--------|
| Submissions per user | 5 | 24 hours |
| Accounts per email domain | 3 | 24 hours |
| Burst detection | 10+ submissions | 1 hour |
| Vote changes per cycle | 1 | Per voting cycle |
| Failed verifications | 5 | 24 hours |

### Quarantine

When burst detection triggers:
1. Set a `quarantined_at` timestamp on the user (add this field to User model if needed)
2. All future submissions from this user are stored with `status="quarantined"` instead of `"pending"`
3. Quarantined submissions are NOT processed by the pipeline until manual review
4. User receives a message: "حساب شما برای بررسی موقت مسدود شده است." ("Your account has been temporarily suspended for review.")

### Unquarantine

For v0, unquarantine is manual (operator sets `quarantined_at = None`). No automated unquarantine.

## Constraints

- Rate limit queries must be efficient — use indexed timestamp columns and COUNT queries, not loading all rows.
- Do NOT use in-memory rate limiting (Redis, etc.). Query the database directly. At v0 scale this is fine.
- Quarantine does not delete any data. Submissions are preserved but held.
- Rate limit messages to users should be in Farsi.

## Tests

Write tests in `tests/test_handlers/test_abuse.py` covering:
- `check_submission_rate`: allows 5 submissions, denies 6th
- `check_submission_rate`: resets after 24 hours (submissions from yesterday don't count)
- `check_domain_rate`: allows 3 accounts on same domain, denies 4th
- `check_burst`: 10 submissions in 1 hour triggers quarantine
- `check_burst`: 9 submissions in 1 hour does not trigger
- `check_vote_change`: first vote change allowed, second denied
- Quarantined user's submissions are stored with `status="quarantined"`
- RateLimitResult correctly reports reason and quarantine flag
- Rate limit functions handle empty database (no prior submissions) gracefully

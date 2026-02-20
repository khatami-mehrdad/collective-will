# Task: Submission Intake Handler

## Depends on
- `messaging/03-webhook-endpoint` (message routing)
- `messaging/02-whatsapp-evolution-client` (WhatsAppChannel for sending confirmations)
- `database/03-core-models` (Submission model, User model, CRUD queries)
- `database/04-evidence-store` (append_evidence function)

## Goal
Implement the handler that receives a user's freeform text message and stores it as a submission, with evidence logging and user confirmation.

## Files to create

- `src/handlers/intake.py` — submission intake handler

## Specification

### handle_submission()

```python
async def handle_submission(
    message: UnifiedMessage,
    user: User,
    channel: BaseChannel,
    db: AsyncSession,
) -> None:
```

Steps:
1. **Check eligibility**: User must be verified (`email_verified=True` AND `messaging_verified=True`) AND account age >= 48 hours. If not eligible, send a Farsi message explaining why and return.
2. **Check rate limit**: Query submissions by this user in the last 24 hours. If >= 5, send "limit reached" message and return.
3. **Store submission**:
   - Compute `hash = sha256(raw_text)`
   - Create Submission record: `user_id`, `raw_text=message.text`, `language` (detect later), `status="pending"`, `hash`
   - Save to database
4. **Log to evidence store**: Append `submission_received` event with payload `{submission_id, user_id, raw_text, hash, timestamp}`
5. **Send confirmation**: Via channel, send a Farsi message:
   ```
   ✅ دریافت شد! نظر شما ثبت شد.
   می‌توانید وضعیت آن را در وبسایت ببینید.
   ```
6. **Update user stats**: Increment `user.contribution_count`

### Hash computation

```python
import hashlib

def hash_submission(text: str) -> str:
    return hashlib.sha256(text.encode("utf-8")).hexdigest()
```

## Constraints

- Do NOT process or canonicalize the submission here. That happens in the pipeline module (batch, every 6 hours).
- Do NOT send the user's raw text to any external API in this handler.
- The user_id in the evidence store payload is the internal UUID, NOT the WhatsApp ID.
- If any step fails (DB write, evidence append), the submission should not be partially saved. Use a transaction.

## Tests

Write tests in `tests/test_handlers/test_intake.py` covering:
- Valid submission from verified user: submission stored, evidence logged, confirmation sent
- Unverified user (email not verified): submission rejected, appropriate message sent
- Unverified user (messaging not verified): submission rejected
- Account too young (< 48h): submission rejected
- Rate limit: 5th submission accepted, 6th rejected with limit message
- Submission hash is correct SHA-256 of raw_text
- Evidence log entry has correct event_type and payload
- User contribution_count incremented
- Database transaction: if evidence append fails, submission is not saved (rollback)
- Mock the channel's send_message to verify confirmation message content

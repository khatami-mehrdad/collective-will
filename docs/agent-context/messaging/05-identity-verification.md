# Task: Identity Verification

## Depends on
- `messaging/03-webhook-endpoint` (FastAPI app and routes)
- `messaging/02-whatsapp-evolution-client` (HMAC tokenization)
- `database/03-core-models` (User model, CRUD queries)
- `database/04-evidence-store` (append_evidence)

## Goal
Implement the full identity verification flow: email magic-link signup, token verification, and WhatsApp account linking.

## Files to create/modify

- `src/handlers/identity.py` — identity/verification logic
- `src/api/routes/auth.py` — auth API endpoints
- `src/api/main.py` — register auth router

## Specification

### POST /api/auth/subscribe

Input: `{"email": "user@example.com"}`

Steps:
1. Validate email format
2. Check if email already registered. If so, send a new magic link (re-verify flow).
3. Check domain rate limit: max 3 accounts per email domain per day
4. Generate magic-link token using `secrets.token_urlsafe(32)`
5. Store token with expiry (15 minutes) — can use a `verification_tokens` table or store on user record
6. Send email with magic link: `https://yourdomain.com/verify?token={token}`
7. If user doesn't exist yet, create User record with `email_verified=False`
8. Return `{"status": "magic_link_sent"}`

For v0, email sending can be a stub (log the link) — actual email integration is an infrastructure concern.

### GET /api/auth/verify

Query param: `token`

Steps:
1. Look up token
2. If token doesn't exist or is expired → return 400
3. If token is valid → set `email_verified=True` on the user
4. Log `user_verified` event to evidence store
5. Delete/invalidate the token
6. Return success page/redirect with instructions to connect WhatsApp

### Failed attempt tracking

- Track failed verification attempts per email (IP-based tracking is optional for v0)
- After 5 failed attempts in 24 hours → 24-hour lockout on that email
- Return generic error (don't reveal whether email exists)

### WhatsApp account linking

When a verified user sends their first WhatsApp message to the bot:

```python
async def link_whatsapp_account(
    user: User,
    wa_hmac_ref: str,
    db: AsyncSession,
) -> None:
```

1. Check that no other user is already linked to this `wa_hmac_ref`
2. Set `user.messaging_account_ref = wa_hmac_ref`
3. Set `user.messaging_verified = True`
4. Set `user.messaging_account_age` to current time (or WhatsApp account creation time if available)
5. Log `user_verified` event (type: `whatsapp_linked`) to evidence store

The linking flow requires the user to send a linking code (provided on the website after email verification) via WhatsApp. The bot matches the code to the pending user.

### Linking code

- Generated after email verification: `secrets.token_urlsafe(8)` (short, easy to type)
- Stored with 1-hour expiry
- User sends this code as their first WhatsApp message
- Bot matches code → links account

## Constraints

- Use `secrets` module for all token generation. NEVER use `random`.
- Do NOT store raw WhatsApp IDs. Only HMAC tokens.
- Magic link tokens must expire (15 minutes). Linking codes must expire (1 hour).
- Generic error responses — do not reveal whether an email is registered.
- Email sending is a stub for now (print/log the link). Do not add email service dependencies.

## Tests

Write tests in `tests/test_handlers/test_identity.py` and `tests/test_api/test_auth.py` covering:
- `POST /api/auth/subscribe` with valid email returns 200
- `POST /api/auth/subscribe` with invalid email returns 422
- `POST /api/auth/subscribe` creates user with `email_verified=False`
- Domain rate limit: 4th account on same domain blocked
- `GET /api/auth/verify` with valid token sets `email_verified=True`
- `GET /api/auth/verify` with expired token returns 400
- `GET /api/auth/verify` with invalid token returns 400
- Lockout after 5 failed verification attempts (returns 429)
- WhatsApp linking: valid linking code sets `messaging_verified=True`
- WhatsApp linking: expired code rejected
- WhatsApp linking: code already used rejected
- WhatsApp linking stores HMAC ref, not raw ID
- Evidence logged for both email verification and WhatsApp linking

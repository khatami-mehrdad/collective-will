# Task: WhatsApp Evolution API Client

## Depends on
- `messaging/01-channel-base-types` (BaseChannel, UnifiedMessage, OutboundMessage)
- `database/01-project-scaffold` (config with EVOLUTION_API_URL, EVOLUTION_API_KEY, SECRET_PEPPER)

## Goal
Implement the WhatsApp channel using Evolution API. This handles sending/receiving WhatsApp messages via the self-hosted Evolution API gateway.

## Files to create

- `src/channels/whatsapp.py` — WhatsAppChannel implementation

## Specification

### WhatsAppChannel class

```python
class WhatsAppChannel(BaseChannel):
    def __init__(self, api_url: str, api_key: str, secret_pepper: str):
        self.api_url = api_url
        self.api_key = api_key
        self.secret_pepper = secret_pepper
        self.client = httpx.AsyncClient(
            base_url=api_url,
            headers={"apikey": api_key},
            timeout=30.0,
        )
```

### send_message()

POST to Evolution API endpoint to send a text message.
- Endpoint: `POST {api_url}/message/sendText/{instance}`
- Body: `{"number": wa_id, "text": message.text}`
- The recipient_ref in OutboundMessage is the HMAC token. You need a reverse lookup (or the raw ID from the sealed mapping) to send. For v0, store a mapping service that holds `hmac_ref -> wa_id` in a restricted table.

### parse_webhook()

Parse the Evolution API webhook payload into a `UnifiedMessage`:
- Extract sender phone number from payload
- Tokenize it: `hmac_ref = hmac.new(secret_pepper.encode(), wa_id.encode(), hashlib.sha256).hexdigest()`
- Extract message text (handle text messages only; ignore media, reactions, status updates)
- Return `None` for non-text-message payloads

Example Evolution API webhook payload structure:
```json
{
  "event": "messages.upsert",
  "instance": "default",
  "data": {
    "key": {
      "remoteJid": "989123456789@s.whatsapp.net",
      "fromMe": false,
      "id": "ABC123"
    },
    "message": {
      "conversation": "سلام، وضعیت اقتصادی خیلی بد است"
    },
    "messageTimestamp": 1707000000
  }
}
```

### HMAC tokenization

```python
import hmac, hashlib

def tokenize_wa_id(wa_id: str, pepper: str) -> str:
    return hmac.new(pepper.encode(), wa_id.encode(), hashlib.sha256).hexdigest()
```

This function must be deterministic — same wa_id + same pepper = same token every time.

### send_ballot()

Format a voting ballot as a numbered list in Farsi and send via `send_message()`:

```
🗳️ صندوق رای باز است!

این هفته، این سیاست‌ها مطرح شدند:

1. [Policy summary]
2. [Policy summary]
3. [Policy summary]

برای رای دادن، شماره‌های موردنظر خود را بفرستید.
مثال: 1, 3

برای انصراف: "انصراف" بفرستید
```

## Constraints

- NEVER log or store raw WhatsApp IDs (`wa_id`) in application tables, logs, or error messages. Only the HMAC token.
- The raw ID mapping (`hmac_ref -> wa_id`) is needed only for sending messages. Keep it in a separate, restricted table or service.
- Use `hmac` module from stdlib, NOT a custom hash.
- Handle Evolution API errors gracefully (connection timeout, 4xx, 5xx). Return `False` from `send_message()` on failure, do not crash.

## Tests

Write tests in `tests/test_channels/test_whatsapp.py` covering:
- `parse_webhook()` correctly extracts text and tokenized sender from a valid payload
- `parse_webhook()` returns None for status update payloads (no message text)
- `parse_webhook()` returns None for media messages
- HMAC tokenization is deterministic (same input = same output)
- HMAC tokenization produces different tokens for different wa_ids
- `send_message()` calls the correct Evolution API endpoint (mock httpx)
- `send_message()` returns False on HTTP error (mock 500 response)
- `send_ballot()` formats the ballot correctly with numbered policies in Farsi

# v0 Collaborator Setup Checklist

Use this document to onboard collaborators for v0 implementation.

Scope reflects current frozen decisions:
- WhatsApp only (Evolution API) for v0
- No action execution in v0
- Cloud embeddings via Mistral `mistral-embed`
- Privacy-first infra (Njalla/1984.is)

---

## 1) Identity and account separation (required)

### Public-facing project identity (pseudonymous)
Use a dedicated pseudonymous identity for:
- GitHub account
- Git commit name/email
- Project communications
- Domain/hosting accounts (Njalla/1984.is)

Recommended: dedicated ProtonMail address (example: `pseudo@protonmail.com`).

### Non-public service accounts (can be regular identity)
Use regular accounts for:
- Anthropic API
- Mistral API
- DeepSeek API
- Dev tooling accounts

Important:
- Do not use your pseudonymous public identity for LLM API billing accounts.
- Keep public pseudonymous identity and personal/billing identity separate.

---

## 2) API keys needed before coding

Each collaborator who runs the stack locally should have access to:
- `ANTHROPIC_API_KEY` (canonicalization + user-facing Farsi messages)
- `MISTRAL_API_KEY` (embeddings via `mistral-embed`)
- `DEEPSEEK_API_KEY` (cluster summaries)
- `EVOLUTION_API_KEY` (WhatsApp gateway auth)

Do not commit keys. Keep them in local `.env` only.

---

## 3) Local development prerequisites

- Python 3.11+
- Node.js 20+ (website work)
- Docker + Docker Compose
- `uv` (Python dependency manager)

---

## 4) Local environment template

Create a local `.env` (never commit):

```bash
# LLM/API
ANTHROPIC_API_KEY=
MISTRAL_API_KEY=
DEEPSEEK_API_KEY=

# WhatsApp gateway (Evolution API)
EVOLUTION_API_KEY=
EVOLUTION_API_URL=http://evolution:8080

# App
DATABASE_URL=postgres://collective:<password>@postgres:5432/collective_will
```

Also ensure `.env` is in `.gitignore`.

---

## 5) WhatsApp setup for v0

v0 uses:
- Evolution API (self-hosted Docker container)
- No Meta Business verification required for development/early pilot

v1 migration plan:
- Move to official WhatsApp Business API for stability/compliance at scale
- Keep channel adapter boundary clean so migration mostly affects `whatsapp.py`

---

## 6) What to set up now vs later

### Set up now (blocks coding)
- Pseudonymous git identity (for commits and GitHub activity)
- API keys: Anthropic, Mistral, DeepSeek
- Local dev environment (Python/Node/Docker/uv)
- Evolution API local container

### Set up before pilot launch (does not block coding)
- Domain via Njalla or 1984.is
- VPS via Njalla or 1984.is
- Cloudflare DNS + HTTPS
- Backup/monitoring hardening

---

## 7) Minimum security rules for collaborators

- Never put real names in commits, code comments, or public docs.
- Never commit secrets (`.env`, private keys, tokens).
- Use VPN for project account access.
- Do not store raw WhatsApp identifiers in logs or analytics exports.
- Treat account mapping data as sensitive; tokenize where possible.

---

## 8) Ready-to-start checklist

- [ ] Pseudonymous GitHub + git identity configured
- [ ] Required API keys available locally
- [ ] `.env` created and ignored by git
- [ ] Docker services run locally (Postgres + Evolution API)
- [ ] Team aligned on v0 boundaries (no action execution, WhatsApp only)


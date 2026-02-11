# Operational Security Guide

Protecting the identity of project contributors whose families may be at risk.

**Threat model**: Iranian government attempting to identify project architects/developers to pressure family members inside Iran.

**Goal**: Complete separation between real identity and project involvement until the project is decentralized and community-run.

---

## Table of Contents

1. [Principles](#1-principles)
2. [Identity Layers](#2-identity-layers)
3. [Infrastructure Anonymity](#3-infrastructure-anonymity)
4. [Code & Repository](#4-code--repository)
5. [Communications](#5-communications)
6. [Financial Privacy](#6-financial-privacy)
7. [Legal Entity Strategy](#7-legal-entity-strategy)
8. [Operational Hygiene](#8-operational-hygiene)
9. [What We Cannot Fully Hide](#9-what-we-cannot-fully-hide)
10. [Recommended Setup](#10-recommended-setup)

---

## 1. Principles

### Assume Breach
Design as if any single system could be compromised. No single point should expose your real identity.

### Compartmentalization
Project identity and personal identity must never touch:
- Separate devices
- Separate networks
- Separate accounts
- Separate payment methods

### Layered Defense
Multiple independent protections. If one fails, others still protect you.

### Minimize Trust
The fewer people who know your real identity, the better. Even trusted partners can be compromised.

---

## 2. Identity Layers

```
┌─────────────────────────────────────────────────────────────┐
│                     PUBLIC LAYER                             │
│  What the world sees:                                        │
│  - Project name: "Collective Will"                           │
│  - Public spokesperson (if any): NOT YOU                     │
│  - Website, social media                                     │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                  ORGANIZATIONAL LAYER                        │
│  Legal entity (if needed):                                   │
│  - Partner NGO as fiscal sponsor                             │
│  - Or: Privacy-jurisdiction entity with nominee directors    │
│  - YOU ARE NOT NAMED                                         │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                   OPERATIONAL LAYER                          │
│  Pseudonymous identity for project work:                     │
│  - Pseudonym (e.g., "Dara" or random name)                  │
│  - Dedicated email (ProtonMail)                              │
│  - Dedicated GitHub account                                  │
│  - VPN/Tor for all project work                             │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    REAL IDENTITY                             │
│  Your actual self:                                           │
│  - Never touches project directly                            │
│  - No financial links                                        │
│  - No account links                                          │
│  - No device sharing                                         │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. Infrastructure Anonymity

### Domain Registration

**DO NOT** use standard registrars with your real name.

| Option | Anonymity | Cost | Notes |
|--------|-----------|------|-------|
| **Njalla** | Excellent | ~$15/yr | They own domain on your behalf; you have usage rights |
| **1984.is** | Good | ~$30/yr | Icelandic, privacy-focused, accepts crypto |
| **Namecheap + WHOIS Privacy** | Medium | ~$10/yr | Privacy guard, but they know your identity |

**Recommendation**: **Njalla** (njal.la)
- Swedish company, strong privacy laws
- They are the legal registrant; your info never in WHOIS
- Accept Bitcoin and Monero
- Even if subpoenaed, they can only reveal payment method (use crypto)

### Hosting Provider

Standard providers (Hetzner, DigitalOcean, AWS) require identity verification.

**Privacy-focused alternatives:**

| Provider | Location | Accepts Crypto | Identity Required | Notes |
|----------|----------|----------------|-------------------|-------|
| **Njalla VPS** | Sweden | Yes (BTC, XMR) | No | Same company as domain |
| **1984.is** | Iceland | Yes | Minimal | Strong privacy laws |
| **OrangeWebsite** | Iceland | Yes | No | Offshore hosting |
| **Privex** | Sweden/Belize | Yes (BTC, XMR) | No | Privacy-focused |
| **Cockbox** | Romania | Yes | No | Cheap, less reliable |

**Recommendation**: **Njalla VPS** or **1984.is**
- Same provider for domain + hosting = fewer identity points
- Pay with Monero for maximum privacy
- Slightly more expensive than Hetzner, but worth it for your safety

**Tradeoff**: These providers are smaller, potentially less reliable than Hetzner. Acceptable for MVP.

### DNS & CDN

**Cloudflare** is excellent for DDoS protection but requires account verification for some features.

Options:
1. **Create Cloudflare account with pseudonym** - Use pseudonymous email, they don't strictly verify identity for free tier
2. **Skip Cloudflare** - Use Njalla's DNS directly (less DDoS protection)
3. **Use Njalla's DDoS protection** - They offer basic protection

For MVP, option 1 or 2 is fine. Add proper DDoS protection later if attacked.

### SSL Certificates

Let's Encrypt certificates don't contain personal info - just domain name. Safe to use.

---

## 4. Code & Repository

### Git Configuration

**CRITICAL**: Git embeds author name and email in EVERY commit permanently.

```bash
# Create project-specific git config
cd ~/collective-will

# Set pseudonymous identity FOR THIS REPO ONLY
git config user.name "Dara"
git config user.email "dara@protonmail.com"

# Verify it's not using global config
git config --list --local
```

**Never commit with your real name/email**. Check existing commits:

```bash
# See all authors in repo
git log --format='%an <%ae>' | sort -u

# If you already committed with real identity, you need to rewrite history
# (Only safe if repo is private and not shared yet)
git filter-branch --env-filter '
if [ "$GIT_AUTHOR_EMAIL" = "your.real@email.com" ]; then
    export GIT_AUTHOR_NAME="Dara"
    export GIT_AUTHOR_EMAIL="dara@protonmail.com"
fi
if [ "$GIT_COMMITTER_EMAIL" = "your.real@email.com" ]; then
    export GIT_COMMITTER_NAME="Dara"
    export GIT_COMMITTER_EMAIL="dara@protonmail.com"
fi
' --tag-name-filter cat -- --all
```

### GitHub Account

**Create a dedicated pseudonymous GitHub account:**

1. Use Tor or VPN when creating
2. Use pseudonymous ProtonMail for signup
3. Don't link to any personal accounts
4. Don't use same password as personal accounts
5. Enable 2FA with a dedicated authenticator
6. Don't star/follow repos from personal account

**Repository settings:**
- Keep repo private until launch
- Don't add personal GitHub as collaborator
- Use pseudonymous account for all interactions

### Code Hygiene

Remove any identifying information:

```bash
# Search for your real name/email in codebase
grep -r "your.real.name" .
grep -r "your@email.com" .
grep -r "your-company" .

# Check for hardcoded paths that might contain username
grep -r "/Users/" .
grep -r "/home/" .
```

**Watch for:**
- Comments mentioning personal details
- TODO comments with your name
- Error messages with paths
- Configuration files with personal info

---

## 5. Communications

### Email

**Dedicated pseudonymous email:**

| Provider | Jurisdiction | Free Tier | Notes |
|----------|--------------|-----------|-------|
| **ProtonMail** | Switzerland | Yes | Best overall, encrypted |
| **Tutanota** | Germany | Yes | Good alternative |
| **Disroot** | Netherlands | Yes | Community-run |

**Setup:**
1. Create account over VPN/Tor
2. Don't link to real phone number (use email-only verification)
3. Don't access from same IP as personal email
4. Use only for project communications

### Messaging

For team communication (if you have collaborators):

| Platform | Anonymity | Notes |
|----------|-----------|-------|
| **Signal** | Medium | Requires phone number (use VoIP number) |
| **Wire** | Good | Can sign up with email only |
| **Matrix/Element** | Good | Can be anonymous |
| **Session** | Excellent | No phone/email required |

**Recommendation**: **Session** or **Wire** for maximum anonymity

### Public Communications

**You should NOT be the public face of this project.**

Options:
1. **No public spokesperson** - Let the project speak for itself
2. **Partner organization** - An established NGO handles public comms
3. **Hired spokesperson** - Someone with no family in Iran
4. **Collective voice** - All communications from "The Collective Will Team"

---

## 6. Financial Privacy

### Key Insight: Credit Card is Fine for Most Services

The privacy protection comes from **which provider you use**, not from payment method.

| Service | Provider Choice | Payment | Why |
|---------|----------------|---------|-----|
| Domain | Njalla or 1984.is | **Credit card OK** | WHOIS shows registrar, not you |
| Hosting | Njalla VPS or 1984.is | **Credit card OK** | Privacy jurisdiction protects you |
| WhatsApp API | Direct or via Twilio | **Credit card OK** | Not publicly linked to project |
| LLM API | Anthropic/Mistral | **Credit card OK** | Not publicly linked to project |
| VPN | Any trusted provider | **Credit card OK** | Low risk service |

### When Crypto Adds Value (Optional)

Crypto payment adds an extra layer if Njalla/1984.is were ever compelled to reveal records. But this is already a difficult scenario (requires Swedish/Icelandic legal process).

**If you want the extra layer:**
- Njalla and 1984.is accept Bitcoin and Monero
- Monero is more private than Bitcoin
- But this is optional, not required

### What We're Actually Protecting Against

| Threat | Protection | Payment Method Matters? |
|--------|------------|------------------------|
| Public WHOIS lookup | Use Njalla (shows "Njalla, Sweden") | **No** |
| Casual investigation | Privacy-focused provider | **No** |
| Swedish/Icelandic legal process | Crypto payment | Slightly (extra layer) |
| Direct subpoena of US companies | N/A (no jurisdiction for Iran) | **No** |

**Bottom line**: Use Njalla or 1984.is. Credit card is fine. Crypto is optional extra protection.

---

## 7. Legal Entity Strategy

### The Problem

Running a visible project typically requires a legal entity for:
- Receiving donations
- Signing contracts (WhatsApp Business API, etc.)
- Legal liability protection

But legal entities create public records with officer names.

### Solutions

**Option 1: Partner with Existing Organization (Recommended)**

Find an established human rights or democracy NGO willing to:
- Act as fiscal sponsor
- Be the legal face of the project
- Hold contracts (WhatsApp API, etc.)
- You remain anonymous contributor

Potential partners:
- Access Now (digital rights)
- Electronic Frontier Foundation
- United for Iran
- Center for Human Rights in Iran
- National Endowment for Democracy programs

**Benefits:**
- Your name nowhere in legal records
- Established compliance/legal team
- Credibility by association
- They handle donations, taxes, etc.

**Option 2: Privacy-Jurisdiction Entity**

Create entity in privacy-respecting jurisdiction with nominee directors:

| Jurisdiction | Privacy | Cost | Notes |
|--------------|---------|------|-------|
| **Nevis** | Excellent | ~$2K/yr | Strong asset protection |
| **Seychelles** | Good | ~$1K/yr | Cheap, common |
| **Wyoming LLC** | Medium | ~$500/yr | US-based, some privacy |
| **Estonian e-Residency** | Low | ~$200/yr | Digital, but identity verified |

With nominee services:
- Nominee director's name on public records
- You control via power of attorney
- More expensive ($500-2000/year for nominees)

**Option 3: Operate Without Entity (MVP)**

For MVP, you may not need a legal entity:
- Accept no donations (self-fund from crypto)
- WhatsApp API: Use unofficial libraries or partner org
- No contracts in your name

This limits scale but maximizes anonymity.

---

## 7.1 Messaging Platform APIs

### WhatsApp Business API — Medium Risk (Acceptable)

The WhatsApp Business API requires Meta business verification, but this is **similar to Anthropic API** in terms of risk:

| Factor | Domain (HIGH) | WhatsApp API (MEDIUM) | Anthropic API (MEDIUM) |
|--------|---------------|----------------------|------------------------|
| Publicly discoverable? | Yes (WHOIS) | **No** | No |
| Requires subpoena? | Maybe | **Yes** | Yes |
| Links you to project publicly? | Yes | **No** | No |

**Why WhatsApp is acceptable (like Anthropic):**
- Meta verification is NOT public record
- Iranian government cannot subpoena Meta (no jurisdiction)
- You're one of millions of Business API users
- No public link between your identity and the project

**What you'll need:**
- Business name (can be generic: "Civic Tech Services" or similar)
- Address (can use PO Box or mail forwarding service)
- Phone number (can use VoIP number)
- Basic business documentation

**Alternative: Use a WhatsApp BSP (Business Solution Provider)**
- Twilio, MessageBird, 360dialog act as intermediaries
- You register with them (like registering with Anthropic)
- They handle the Meta relationship
- One step removed from direct Meta verification

### Telegram Bot API — Low Risk

Telegram requires even less verification:
- Create bot via @BotFather
- No business verification required
- No identity documents
- Can use pseudonymous Telegram account

**Note:** Telegram is blocked in Iran and requires VPN. WhatsApp is the primary channel for reaching users inside Iran.

---

## 8. Operational Hygiene

### Device Separation

**Ideal**: Dedicated laptop for project work
- Never log into personal accounts
- Different browser profiles
- Full disk encryption
- Wipe if traveling to sensitive countries

**Minimum**: Dedicated VM or user account
- Separate browser (Firefox with containers)
- VPN always on for project work
- Don't mix with personal browsing

### Network Separation

**Always use VPN or Tor for project work:**

```
Your home network
       │
       ▼
     VPN (Mullvad)
       │
       ▼
  Project websites/services
```

**Never access project services from:**
- Work network
- Identifiable home IP without VPN
- Mobile network (linked to your identity)

### Browser Hygiene

```
Personal browsing: Chrome/Safari with personal accounts
Project work: Firefox with:
  - Separate container for each service
  - VPN always on
  - No personal bookmarks/history
  - Different search engine
```

### Password Management

- Use separate password manager or separate vault for project
- Don't sync project passwords to personal devices
- Strong, unique passwords everywhere

### Physical Security

- Don't leave project work visible when others around
- Encrypted drives
- Screen lock always
- Consider plausible deniability (hidden volumes with VeraCrypt)

---

## 9. What We Cannot Fully Hide

### Honest Limitations

| Aspect | Difficulty to Hide | Notes |
|--------|-------------------|-------|
| Your existence as developer | Impossible if project succeeds | Accept this |
| Writing style (stylometry) | Very hard | Can analyze code/docs patterns |
| Timezone patterns | Hard | Commit times reveal approximate location |
| Technical fingerprint | Medium | Tech choices, code style |
| That someone with Iran connection runs it | Medium | Project focus reveals this |

### What We CAN Hide

| Aspect | Achievable | Method |
|--------|------------|--------|
| Legal name | Yes | Pseudonyms, nominees, partners |
| Physical address | Yes | No public records with address |
| Financial identity | Mostly | Crypto, partner org |
| Personal accounts | Yes | Strict separation |
| Family connection | Partially | Nothing public links you |

### The Goal

**Not**: Perfect anonymity (impossible)
**But**: No public records or easily discoverable links between the project and your real identity

An adversary would need to:
1. Compromise privacy-focused hosting provider (hard, requires legal process in Sweden/Iceland)
2. AND trace cryptocurrency (hard for Monero, possible for Bitcoin)
3. AND compromise your operational security (your responsibility)

This is a high bar for the Iranian government, who primarily target publicly visible activists.

---

## 10. Recommended Setup

### Risk-Tiered Approach for MVP

**Principle**: The key distinction is **publicly discoverable** vs **requires subpoena**. Use privacy-focused providers for domain/hosting; credit card payment is fine.

```
┌─────────────────────────────────────────────────────────────────┐
│ HIGH RISK — Publicly discoverable, must protect                │
├─────────────────────────────────────────────────────────────────┤
│ Regular domain registrar    → Your name in WHOIS (public!)     │
│ Regular hosting (Hetzner)   → Identity verified, discoverable  │
│ Git commits with real name  → Permanent in code history        │
│ GitHub with real identity   → Public profile                   │
│ Public communications       → Linked to real name              │
│ Legal entity with your name → Public records                   │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ MEDIUM RISK — Not public, requires subpoena (acceptable)       │
├─────────────────────────────────────────────────────────────────┤
│ Njalla domain (credit card OK)    → WHOIS shows Njalla, Sweden │
│ Njalla/1984.is VPS (credit card OK) → Privacy jurisdiction     │
│ WhatsApp Business API             → Meta has records, not public│
│ LLM API (Anthropic)               → They have records, not public│
│ Twilio/MessageBird                → They have records, not public│
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ LOW RISK — No special handling needed                          │
├─────────────────────────────────────────────────────────────────┤
│ VPN service           → Regular account OK                     │
│ Dev tools             → Regular accounts OK                    │
│ Cloud backups         → Encrypted, regular account OK          │
│ Telegram Bot API      → No verification required               │
└─────────────────────────────────────────────────────────────────┘
```

**Why Njalla/1984.is with credit card is acceptable:**
- **WHOIS shows the registrar**, not you (this is the key protection)
- Privacy-focused jurisdiction (Sweden/Iceland)
- Requires legal process in their jurisdiction to reveal customer
- Iranian government cannot easily subpoena Swedish/Icelandic companies
- Crypto payment is optional extra layer, not required

**Why WhatsApp/Anthropic are acceptable:**
- **Not publicly discoverable** — no one can look up "who runs this bot"
- **Requires legal process** — Iranian government cannot subpoena US companies
- **Provider only knows** you have an account, not what project it's for

### MVP-Optimized Setup

```
MUST USE PRIVACY-FOCUSED PROVIDERS (credit card OK)
───────────────────────────────────────────────────
DOMAIN
├── Registrar: Njalla or 1984.is
├── Payment: Credit card OK (crypto optional)
└── WHOIS: Shows registrar, not you ← KEY PROTECTION

HOSTING
├── Provider: Njalla VPS or 1984.is
├── Payment: Credit card OK (crypto optional)
└── Jurisdiction: Sweden/Iceland (privacy-friendly)

MUST USE PSEUDONYMOUS IDENTITY
───────────────────────────────────────────────────
CODE REPOSITORY
├── Platform: GitHub
├── Account: Pseudonymous (new account, new email)
└── Git config: Pseudonymous name/email in every commit

PUBLIC COMMUNICATIONS
├── Project email: ProtonMail (pseudonymous)
├── Public voice: "Collective Will Team"
└── No individual names anywhere public

REGULAR ACCOUNTS OK (not publicly linked to project)
───────────────────────────────────────────────────
├── WhatsApp Business API — credit card OK
├── LLM API (Anthropic/Mistral) — credit card OK
├── Twilio/MessageBird — credit card OK
├── VPN subscription — credit card OK
├── Telegram Bot — no verification needed
├── Dev tools — regular accounts
└── Cloud backups — encrypted, regular account
```

### Cost Estimate

| Item | Provider | Payment | Annual Cost |
|------|----------|---------|-------------|
| Domain | Njalla | Crypto | ~$15 |
| VPS | Njalla | Crypto | ~$180-360 |
| VPN | Any | Card OK | ~$60-100 |
| LLM API | Anthropic | Card OK | ~$60-180 |
| **Total** | | | **~$350-650/year** |

### What You Need to Do Now

### Checklist Before Launch

**Must do — Use privacy-focused providers:**
- [ ] Domain via Njalla or 1984.is (credit card OK)
- [ ] Hosting via Njalla VPS or 1984.is (credit card OK)

**Must do — Use pseudonymous identity:**
- [ ] Git history clean of real name/email
- [ ] Pseudonymous GitHub account created
- [ ] Pseudonymous ProtonMail for project communications
- [ ] No real name in any public-facing content (website, docs, social)

**Regular accounts OK — Not publicly linked to project:**
- [x] WhatsApp Business API
- [x] LLM API (Anthropic/Mistral)
- [x] Twilio/MessageBird
- [x] VPN subscription
- [x] Telegram Bot
- [x] Dev tools
- [x] Cloud backups

---

## Appendix: If Compromise is Suspected

If you believe your identity may have been exposed:

1. **Don't panic** - Exposure doesn't mean immediate danger
2. **Assess the leak** - What was exposed? How?
3. **Don't delete evidence** - May need for legal response
4. **Contact digital security experts** - Access Now has a helpline
5. **Consider family safety** - Do they need to take precautions?
6. **Plan transition** - Hand off project to trusted parties if needed

**Resources:**
- Access Now Digital Security Helpline: https://www.accessnow.org/help/
- EFF Surveillance Self-Defense: https://ssd.eff.org/
- Security in a Box: https://securityinabox.org/

---

*This document should be kept private and not committed to public repositories.*

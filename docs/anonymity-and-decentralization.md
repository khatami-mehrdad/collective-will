# Anonymity, Takedown Resilience, and Federation

Erase the identity link between the existing public repo and your real name, establish a pseudonymous development identity, and architect the software for takedown-resistant self-hosting with a path toward federation.

This plan has four phases:
1. **Identity Cleanup** — urgent, do before writing any code
2. **Self-Hostable Architecture** — design for cloneability
3. **Federation Design** — architect now, build in v1
4. **Ongoing Anonymity Measures** — habits and hygiene

---

## Table of Contents

1. [Phase 1: Identity Cleanup](#phase-1-identity-cleanup)
2. [Phase 2: Self-Hostable Architecture](#phase-2-self-hostable-architecture)
3. [Phase 3: Federation Design](#phase-3-federation-design)
4. [Phase 4: Ongoing Anonymity Measures](#phase-4-ongoing-anonymity-measures)
5. [Document Changes Required](#document-changes-required)
6. [Checklist](#checklist)

---

## Phase 1: Identity Cleanup

**Priority: URGENT. Do this before writing any code.**

### The Problem

The public repo at `github.com/khatami-mehrdad/collective-will` has:

- **14 commits** all authored by `Mehrdad <khatami-mehrdad@github.com>`
- **Commit timestamps** showing `-0800` (US Pacific timezone), which narrows location to US/Canada West Coast
- **GitHub username** containing a real name
- A README that explicitly mentions **Iran** as the pilot target

Even after deletion, GitHub forks, cached pages (Google, Wayback Machine, GitHub Archive), and anyone who already cloned the repo may retain this data.

---

### Step 1.1: Check for External Caches

Before deleting anything, check what's already out there:

- [ ] Search Google: `site:github.com khatami-mehrdad collective-will`
- [ ] Check Wayback Machine: `web.archive.org/web/*/github.com/khatami-mehrdad/collective-will`
- [ ] Check GitHub Archive / GH Torrent if applicable
- [ ] If cached, submit removal requests:
  - Google: [Search Console removal tool](https://search.google.com/search-console/removals)
  - Wayback Machine: email `info@archive.org` requesting exclusion
  - Google cache: use the "Remove outdated content" tool after the repo is deleted

---

### Step 1.2: Delete the Existing Public Repo

- [ ] Go to GitHub repo Settings > Danger Zone > Delete this repository
- [ ] Delete `khatami-mehrdad/collective-will` entirely

**Why delete, not make private**: Private repos still exist on GitHub servers and can be subpoenaed. Deleted repos are purged after 90 days (per GitHub's data retention policy).

**Do this today.**

---

### Step 1.3: Create a Pseudonymous Identity

All steps below should be done **over VPN** (Mullvad or similar).

- [ ] Pick a pseudonym with **no connection** to your real name
  - Not your initials
  - Not a translation or transliteration of your name
  - Not a nickname friends or family use
  - Something completely unrelated (e.g., a random word combination)

- [ ] Create a **ProtonMail** account with the pseudonym
  - Use VPN when creating
  - Do not link a real phone number (use email-only verification)
  - Do not access from the same IP/browser as personal email

- [ ] Create a **GitHub account** using the ProtonMail address
  - Use VPN when creating
  - Do NOT follow or star your real GitHub account
  - Do NOT follow repos that could link back to you
  - Enable 2FA with a dedicated TOTP authenticator app (not the same one as personal accounts)
  - Use a different avatar/profile than any personal accounts

---

### Step 1.4: Create a Clean Repo Under the New Identity

The existing git history contains your real name in every commit. **Do NOT push the existing history.** Start fresh.

```bash
# 1. Create a NEW directory (not inside the old repo)
mkdir collective-will-clean
cd collective-will-clean

# 2. Initialize a fresh git repo
git init

# 3. Set pseudonymous identity FOR THIS REPO ONLY
git config user.name "YourPseudonym"
git config user.email "pseudonym@proton.me"

# 4. Copy files from old repo (NOT the .git folder)
cp -r /path/to/old/collective-will/* .
cp /path/to/old/collective-will/.gitignore .
# Do NOT copy the .git directory

# 5. Commit with a controlled UTC timestamp (no timezone leak)
GIT_AUTHOR_DATE="2026-02-15T12:00:00+00:00" \
GIT_COMMITTER_DATE="2026-02-15T12:00:00+00:00" \
git add -A && git commit -m "initial commit"

# 6. Add the new GitHub remote
git remote add origin git@github.com:YourPseudonym/collective-will.git

# 7. Push (over VPN)
git push -u origin main
```

**Verify the clean repo:**

```bash
# Confirm no real identity in commits
git log --format='%an <%ae> %ai'

# Should show ONLY:
# YourPseudonym <pseudonym@proton.me> 2026-02-15 12:00:00 +0000
```

---

### Step 1.5: Set Up Commit Timestamp Discipline

Your old commits all showed `-0800` (Pacific time), which narrows your location. Prevent this going forward.

**Option A: Git alias that forces UTC (recommended)**

```bash
# Add to the new repo's local config
git config alias.ci '!GIT_AUTHOR_DATE="$(date -u +%Y-%m-%dT%H:%M:%S+00:00)" GIT_COMMITTER_DATE="$(date -u +%Y-%m-%dT%H:%M:%S+00:00)" git commit'

# Usage: instead of `git commit -m "msg"`, use:
git ci -m "msg"
```

**Option B: Post-commit hook that rewrites timestamps**

```bash
# Create .git/hooks/post-commit
cat > .git/hooks/post-commit << 'EOF'
#!/bin/bash
# Rewrite the commit to use UTC timestamp
CURRENT=$(git log -1 --format=%H)
GIT_AUTHOR_DATE="$(date -u +%Y-%m-%dT%H:%M:%S+00:00)" \
GIT_COMMITTER_DATE="$(date -u +%Y-%m-%dT%H:%M:%S+00:00)" \
git commit --amend --no-edit --date="$(date -u +%Y-%m-%dT%H:%M:%S+00:00)" > /dev/null 2>&1
EOF
chmod +x .git/hooks/post-commit
```

**Option C: Set TZ environment variable for project work**

```bash
# Add to your shell profile or run before project work
export TZ=UTC
```

---

### Step 1.6: Scrub Docs for Identifying Content

Before pushing to the new repo, check all files:

```bash
# Search for real name or identifiable strings
grep -ri "mehrdad" .
grep -ri "khatami" .
grep -ri "/Users/" .
grep -ri "/home/" .

# Check for any personal email addresses
grep -ri "@gmail\|@yahoo\|@hotmail\|@outlook" .

# Check for any hardcoded paths
grep -ri "khatami-mehrdad" .
```

- [ ] Verify no personal references exist in any file
- [ ] Verify `CONTRIBUTING.md` is clean (it is -- no identifying info)
- [ ] Verify `README.md` does not reference the old GitHub URL

---

## Phase 2: Self-Hostable Architecture

**Goal**: Anyone can `git clone` + `docker compose up` and have a working instance.

This is the PirateBay/Mastodon model -- the software is a commodity, instances are independent. If one goes down, others survive.

---

### Step 2.1: One-Command Deployment

The existing `docker-compose.yml` in the infrastructure guide is a good start. To make it truly self-hostable:

- [ ] Place a working `docker-compose.yml` at the **repo root** (not buried in docs)
- [ ] Create `.env.example` with all required variables documented
- [ ] Database migrations run **automatically** on first boot (via entrypoint script)
- [ ] Add a `/health` endpoint that reports system status
- [ ] Create a `Makefile` with common commands:

```makefile
setup:      ## First-time setup: copy env, build, migrate
run:        ## Start all services
stop:       ## Stop all services
backup:     ## Create database backup
export:     ## Export public data (clusters, votes, evidence chain)
import:     ## Import public data from another instance
logs:       ## Tail all logs
health:     ## Check system health
```

---

### Step 2.2: Instance Configuration

Each instance must be fully independent. No hardcoded assumptions about who runs it.

| Setting | Source | Notes |
|---------|--------|-------|
| Instance name | `.env` | Default: "Collective Will", operator can change |
| Domain | `.env` | Operator's own domain |
| Branding / logo | `config/` directory | Optional override |
| Messaging bot tokens | `.env` | Operator's own WhatsApp/Telegram bots |
| LLM API keys | `.env` | Operator's own Anthropic/Mistral keys |
| Database credentials | `.env` | Auto-generated on first setup |

**Critical**: The software must **never phone home**. No analytics, no update checks, no telemetry, no external calls except the ones the operator explicitly configures (LLM API, messaging APIs).

---

### Step 2.3: Data Portability

Operators should be able to export and import **public-facing data** (no user PII):

**What exports** (public, safe to share):
- Cluster summaries and metadata
- Vote tallies (aggregate counts, not individual votes)
- Evidence chain entries
- Voting cycle results

**What never exports** (stays on the instance):
- User accounts and identifiers
- Raw submissions with user links
- Individual votes
- Submission-to-user mappings

**Implementation:**

```bash
# Export public data to a portable format
make export
# Produces: export/collective-will-export-2026-02-15.json

# Import on a new instance to bootstrap
make import FILE=collective-will-export-2026-02-15.json
```

The evidence chain should be **independently verifiable** -- any instance that imports the data can verify the hash chain integrity without trusting the source instance.

---

### Step 2.4: Tor Hidden Service (.onion)

Optional but recommended. Provides access even if all clearnet domains are seized.

Add to `docker-compose.yml`:

```yaml
tor:
  image: goldy/tor-hidden-service:latest
  environment:
    SERVICE1_TOR_SERVICE_HOSTS: "80:nginx:80"
    SERVICE1_TOR_SERVICE_VERSION: "3"
  volumes:
    - ./data/tor:/var/lib/tor/hidden_service
  depends_on:
    - nginx
  restart: unless-stopped
```

After first boot:

```bash
# Find your .onion address
docker compose exec tor cat /var/lib/tor/hidden_service/hostname
```

- [ ] Add Tor service to docker-compose as optional (commented out by default)
- [ ] Document how to enable and find the `.onion` address
- [ ] Publish `.onion` address on social media / other instances so it survives domain takedown

---

### Step 2.5: Multiple Domain Strategy

Don't rely on a single domain:

- [ ] Register 2-3 domains via Njalla on **different TLDs** (e.g., `.org`, `.net`, `.io`)
- [ ] All point to the same instance (or different mirrors)
- [ ] If one domain is seized, others remain
- [ ] Publish all domains publicly so users know alternatives
- [ ] Consider a `.onion` address as the "canonical" address that can never be seized

---

## Phase 3: Federation Design

**Build timing**: Architect interfaces now, implement in v1.

Federation means instances share **aggregate results** while keeping **user data local**.

---

### Step 3.1: What Federates vs. What Stays Local

| Data | Federates? | Reason |
|------|-----------|--------|
| Cluster summaries | Yes | Public aggregate data |
| Vote tallies (aggregate) | Yes | Public results |
| Evidence chain | Yes | Enables cross-instance verification |
| Voting cycle metadata | Yes | Public schedule/results |
| User accounts | **No** | Private, instance-local |
| Raw submissions | **No** | Contains user link |
| Individual votes | **No** | Privacy-sensitive |
| User-to-submission mapping | **No** | The critical secret |

**Key principle**: An instance can be destroyed and the public data survives on other instances. User data is lost (users re-register on another instance). This is acceptable and by design.

---

### Step 3.2: Federation Protocol (Design Sketch)

Pull-based, no central registry:

```
Instance A                          Instance B
    |                                    |
    |--- GET /federation/manifest ------>|
    |<-- {instance_id, last_cycle, ...}--|
    |                                    |
    |--- GET /federation/cycles?since=N->|
    |<-- [{cycle_id, clusters, votes}]---|
    |                                    |
    |--- GET /federation/evidence?since->|
    |<-- [{evidence_chain_entries}]------|
    |                                    |
    |  (verify evidence chain locally)   |
```

- Instances expose a `/federation/` API namespace (read-only)
- Pull-based: instances periodically pull from peers they trust
- Evidence chain can be verified **cryptographically** without trusting the source
- No central registry -- instances discover each other via configuration or manual entry
- Peers are configured in `config.yaml`:

```yaml
federation:
  enabled: true
  instance_id: "randomly-generated-uuid"
  peers:
    - url: "https://other-instance.org"
      trusted: true
    - url: "http://xyz123.onion"
      trusted: true
  sync_interval_minutes: 60
```

---

### Step 3.3: What This Means for v0 Code

Even though federation is v1, make these design choices now to avoid painful refactors later:

- [ ] Add `instance_id` field to relevant database tables: `clusters`, `voting_cycles`, `evidence_log`
- [ ] Keep a clean separation in the schema between "instance-local" data (users, submissions, individual votes) and "public/federable" data (clusters, tallies, evidence)
- [ ] Reserve the `/federation/` API namespace in the route definitions
- [ ] Build `/federation/manifest` as a read-only endpoint early -- it costs nothing and proves the architecture
- [ ] Design the evidence store so that hash chain verification works independently of the instance that created it (include `instance_id` in evidence entries)

**Data model additions for v0:**

```typescript
// Add to Cluster
interface Cluster {
  // ... existing fields ...
  instanceId: string;           // UUID of the originating instance
}

// Add to VotingCycle
interface VotingCycle {
  // ... existing fields ...
  instanceId: string;           // UUID of the originating instance
}

// Add to EvidenceLogEntry
interface EvidenceLogEntry {
  // ... existing fields ...
  instanceId: string;           // UUID of the originating instance
}
```

**New API endpoints (read-only, safe to expose):**

```
GET  /federation/manifest            # Instance identity, last cycle, capabilities
GET  /federation/cycles?since=N      # Cycles with cluster summaries and vote tallies
GET  /federation/evidence?since=N    # Evidence chain entries for verification
```

---

## Phase 4: Ongoing Anonymity Measures

### Stylometry Awareness

Writing style in docs, code comments, commit messages, and variable naming can be fingerprinted.

- [ ] Consider having someone else review/rewrite public-facing docs
- [ ] Use an LLM to rephrase your writing in a neutral style before committing
- [ ] Be aware that commit message conventions, code comment style, and naming patterns are all analyzable signals
- [ ] Vary writing style across commits (some terse, some detailed)

### Development Hygiene

- [ ] **Always** use VPN when accessing the pseudonymous GitHub account
- [ ] **Never** access the pseudonymous account from the same browser session/profile as your real GitHub
- [ ] Use a **separate browser profile** (Firefox Multi-Account Containers or a dedicated browser) for all project work
- [ ] Set the pseudonymous git config as **repo-local** (not global) to prevent leaking into personal projects:

```bash
# In the project repo (NOT --global):
git config user.name "YourPseudonym"
git config user.email "pseudonym@proton.me"

# Verify it's local:
git config --local --list | grep user
```

- [ ] Periodically audit your commits for accidental identity leaks:

```bash
git log --format='%an <%ae> %ai' | sort -u
```

### Operational Compartmentalization

```
Personal identity          Project identity
─────────────────          ─────────────────
Personal GitHub      ←──→  NEVER cross-link
Personal browser     ←──→  Separate profile / container
Personal email       ←──→  ProtonMail pseudonym
Home IP              ←──→  Always behind VPN
Personal devices     ←──→  Ideally separate device or VM
```

---

## Document Changes Required

These existing docs need updates to reflect the decisions in this plan:

| Document | Change |
|----------|--------|
| [operational-security.md](operational-security.md) | Add commit timestamp discipline, stylometry awareness, Tor hidden service guidance |
| [infrastructure-guide.md](infrastructure-guide.md) | Add "Deploying Your Own Instance" section for third-party operators, Tor setup |
| [mvp-specification.md](mvp-specification.md) | Add `instance_id` to relevant data models, reserve `/federation/` API namespace |
| [roadmap.md](roadmap.md) | Add self-hosting as v0 requirement, federation in v1 description |
| **New**: `INSTALL.md` | Operator-focused deployment guide (non-developer audience) |
| **New**: `.env.example` | Template configuration file with all variables documented |

---

## Checklist

### Phase 1: Identity Cleanup (Do Immediately)

- [ ] Check Google cache for existing repo
- [ ] Check Wayback Machine for existing repo
- [ ] Submit removal requests if cached
- [ ] Delete `khatami-mehrdad/collective-will` on GitHub
- [ ] Create pseudonymous ProtonMail (over VPN)
- [ ] Create pseudonymous GitHub account (over VPN)
- [ ] Enable 2FA on pseudonymous GitHub
- [ ] Create clean repo with no git history
- [ ] Verify no real identity in any commit
- [ ] Set up UTC timestamp discipline (alias or hook)
- [ ] Scrub all docs for identifying content
- [ ] Push to new pseudonymous GitHub (over VPN)

### Phase 2: Self-Hostable Architecture (During v0 Build)

- [ ] `docker-compose.yml` at repo root, works with `.env.example`
- [ ] `.env.example` with all variables documented
- [ ] `Makefile` with setup/run/backup/export/import commands
- [ ] `INSTALL.md` for third-party operators
- [ ] Auto-running database migrations on first boot
- [ ] `/health` endpoint
- [ ] No phone-home behavior anywhere in codebase
- [ ] Data export (public data only, no PII)
- [ ] Data import (bootstrap from export file)
- [ ] Tor hidden service (optional, documented)
- [ ] Multiple domain strategy (2-3 TLDs via Njalla)

### Phase 3: Federation (Design in v0, Build in v1)

- [ ] `instance_id` in database schema (clusters, cycles, evidence)
- [ ] `/federation/manifest` read-only endpoint
- [ ] Clean separation of local vs. federable data in schema
- [ ] Federation protocol documented
- [ ] Peer configuration in `config.yaml`

### Phase 4: Ongoing Hygiene

- [ ] VPN always on for project work
- [ ] Separate browser profile for project
- [ ] Periodic commit audit for identity leaks
- [ ] Stylometry awareness for public-facing writing

---

*This document should be kept private and not committed to public repositories until identifying references to the old repo are removed.*

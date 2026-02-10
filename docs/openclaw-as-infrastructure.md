# OpenClaw as Infrastructure

How [OpenClaw](https://github.com/openclaw) — an open-source, local-first agentic AI assistant — could serve as foundational infrastructure for Collective Will, and what ideas from its architecture are worth stealing.

---

## Background

OpenClaw (formerly Clawdbot / Moltbot) is an open-source AI agent gateway that runs locally and connects to messaging platforms (WhatsApp, Telegram, Signal, Discord, Slack, iMessage, and more). It supports multiple isolated agents, each with their own persona, skills, tools, memory, and sandboxed execution. A single gateway process orchestrates everything.

The idea of **a group of specialized agents taking on different roles in a campaign pipeline** — one agent to understand submissions, another to cluster them, another to plan action — comes directly from OpenClaw's multi-agent architecture.

This document maps OpenClaw's design patterns onto Collective Will's needs, identifies what we'd get for free, and flags what we'd still need to build.

---

## Key Ideas to Borrow

### 1. Pipeline Stages as Isolated Agents

OpenClaw's core primitive is the **agent**: a fully scoped brain with its own workspace, persona files (`AGENTS.md`, `SOUL.md`), tool access, model configuration, and session store. Agents are isolated from each other by default — no shared state, no cross-talk unless explicitly enabled.

This maps directly to Collective Will's pipeline:

| Pipeline Stage | OpenClaw Agent | Model | Tools Allowed |
|---|---|---|---|
| Canonicalization | `canonicalizer` | Fast, cheap model | Read submissions, write canonical forms |
| Clustering | `clusterer` | Smarter model | Read canonical forms, write clusters |
| Action Planning | `action-planner` | Capable model | Read clusters + vote results, draft actions |
| Action Execution | `executor` | Fast model | Send emails/faxes, log delivery receipts |

Each agent gets only the tools it needs. The canonicalizer **cannot** send emails. The executor **cannot** modify clusters. This is separation of powers enforced at the infrastructure level, not by policy.

**Relevant OpenClaw concepts:**
- `agents.list[]` — defines each agent with its own `workspace`, `model`, `tools`, `sandbox`
- Per-agent `tools.allow` / `tools.deny` — restricts what each agent can do
- Per-agent `sandbox` — isolates execution environments

### 2. Multi-Channel Intake

One of the hardest UX problems is "how do people submit concerns?" OpenClaw already has production-grade integrations with every major messaging platform. Instead of building a custom submission UI, users could **text their concerns through whatever app they already use**.

The channel plugin system normalizes messages from WhatsApp, Telegram, Signal, Discord, Slack, and iMessage into a common format. The routing/bindings system means different communities or campaigns could use different channels, all feeding into the same pipeline.

This dramatically lowers the barrier to participation — meet people where they already are.

**Relevant OpenClaw concepts:**
- Channel plugin system — normalizes messages across platforms
- Bindings — routes messages to specific agents by channel, account, or peer
- `allowFrom` — controls who can submit

### 3. Sub-Agents for Parallel Campaign Work

OpenClaw's `sessions_spawn` tool lets a main agent spin up isolated background workers that run independently and report results back when done. This is non-blocking: the main agent keeps working while sub-agents execute in parallel.

For Collective Will:

- **Processing submissions**: spawn a sub-agent per batch of incoming concerns for parallel canonicalization
- **Executing campaign actions**: once a vote passes, spawn sub-agents to draft and send personalized letters for each opted-in user
- **Cross-agent delegation**: an orchestrator agent can spawn work under specialized agents (`subagents.allowAgents`), so a "campaign coordinator" can hand off to a "letter drafter" and a "phone script writer" simultaneously

**Relevant OpenClaw concepts:**
- `sessions_spawn` — non-blocking background agent runs
- Sub-agent queue lanes — separate from main agent, prevents blocking
- Cross-agent spawning — orchestrator delegates to specialists
- Announce flow — results posted back to requester

### 4. Skills as Civic Action Templates

OpenClaw skills are markdown files (`SKILL.md`) with instructions that teach an agent how to use tools. They're composable, auditable, and overridable. Each skill is just a directory with a markdown file and optional scripts.

For Collective Will, each type of civic action could be a skill:

- `write-to-representative` — templates and rules for drafting letters to elected officials
- `draft-public-comment` — how to write comments for regulatory proceedings
- `file-foia-request` — templates for FOIA requests with jurisdiction-specific rules
- `organize-phone-bank` — scripts and talking points for coordinated calling campaigns

Because skills are plain markdown, they are:
- **Auditable** — anyone can read what the agent will do
- **Community-editable** — submit a PR to improve a template
- **Overridable by community** — the precedence system (bundled < managed < workspace) lets local communities customize actions for their context

**Relevant OpenClaw concepts:**
- `SKILL.md` format — markdown with YAML frontmatter
- Skill precedence — workspace overrides managed overrides bundled
- ClawHub — public registry for sharing skills across communities
- Gating via `metadata.openclaw.requires` — skills only load when prerequisites are met

### 5. Memory and Hybrid Search as Evidence Store

Collective Will needs every step to be auditable — an "evidence store." OpenClaw's memory architecture has useful patterns:

- **Plain markdown as source of truth** — not locked in a database, just files on disk that anyone can read and verify
- **Hybrid search (vector + BM25)** — when checking "have we seen this concern before?" you need both semantic similarity (same concern, different words) and exact matching (specific policy numbers, bill names)
- **Per-agent memory isolation** — each agent's memory is scoped to its own directory, enforcing that the clustering agent can't see raw submissions, only canonicalized versions
- **Session transcripts as JSONL** — every interaction is logged in structured, parseable format — an audit trail by default

The memory system would need to be extended with cryptographic verification (hashing, signatures) for Collective Will's stronger auditability requirements, but the storage and retrieval patterns are directly reusable.

**Relevant OpenClaw concepts:**
- `memory/YYYY-MM-DD.md` + `MEMORY.md` — layered memory files
- Hybrid search — BM25 + vector similarity
- Session JSONL transcripts — structured interaction logs
- Per-agent SQLite memory stores

### 6. Hooks for Transparency

OpenClaw's hook system fires on lifecycle events: agent bootstrap, command execution, tool calls, message send/receive. Hooks are directories with a `HOOK.md` and a handler script — discoverable and auditable, just like skills.

For Collective Will, hooks could inject transparency at every step:

- **`before_tool_call` / `after_tool_call`** — log every AI action to the evidence store with timestamps and input/output
- **`tool_result_persist`** — annotate tool results with provenance metadata before they're saved
- **`message_received`** — timestamp and hash every incoming submission at intake
- **`agent_end`** — capture the full reasoning trace after each pipeline stage completes

This means auditability becomes a **cross-cutting concern** — injected by the event system rather than implemented separately in each agent.

**Relevant OpenClaw concepts:**
- Internal hooks — event-driven scripts for commands and lifecycle
- Plugin hooks — intercept tool calls, messages, session boundaries
- `tool_result_persist` — transform results before persistence

### 7. Cron for Campaign Lifecycle

Campaigns have temporal structure. OpenClaw's cron system supports:

- **One-time events** (`at`) — "voting closes February 15 at midnight"
- **Recurring intervals** (`every`) — "aggregate new submissions every hour"
- **Cron expressions with timezone** — "send weekly summary every Monday at 9am Eastern"
- **Isolated execution** — cron jobs run in their own sessions, preventing interference with live interactions

This covers submission windows, voting deadlines, reminder notifications, periodic aggregation runs, and campaign expiration.

**Relevant OpenClaw concepts:**
- Cron service with `at`, `every`, and `cron` schedule types
- `agentTurn` jobs — run an agent with a message on schedule
- `systemEvent` jobs — enqueue events into the main session
- Error handling with exponential backoff

### 8. Security Model as Trust Boundaries

OpenClaw's security architecture addresses concerns raised in the Collective Will governance doc:

- **Per-agent sandboxing** — each pipeline stage runs in its own container with restricted filesystem access
- **Tool allow/deny lists** — enforce what each agent can and cannot do
- **Audit framework** — automated checks for filesystem permissions, exposed secrets, channel access
- **Skill scanning** — validates that community-submitted action templates don't contain malicious patterns
- **Channel allowlists** — controls who can submit, per channel and per account

---

## What OpenClaw Gives Us (and What It Doesn't)

### What we'd get

- Multi-agent orchestration with per-agent isolation, tools, and models
- Multi-channel message intake (WhatsApp, Telegram, Signal, Discord, etc.)
- Sub-agent spawning for parallel background work
- Skills system for composable, auditable action templates
- Memory with hybrid search for deduplication and context retrieval
- Hooks for cross-cutting transparency and logging
- Cron for campaign lifecycle management
- Security model with sandboxing and tool restrictions
- Session management with structured transcripts
- A gateway that runs locally — no dependency on a hosted service

### What we'd still need to build

| Component | Why OpenClaw doesn't cover it |
|---|---|
| **LLM clustering logic** | OpenClaw orchestrates agents but doesn't include NLP-specific clustering algorithms. Build as an agent + skill. |
| **Voting mechanism** | No concept of user voting or quorum in OpenClaw. This is governance-hard and needs dedicated design. |
| **Transparent agenda dashboard** | OpenClaw has a chat UI, not a public-facing dashboard showing what surfaced and why. |
| **Cryptographic audit trail** | OpenClaw logs transcripts but doesn't hash/sign them. Extend hooks with cryptographic verification. |
| **Decentralization / federation** | OpenClaw runs a single gateway per host. Federating gateways across communities requires protocol work. |
| **Identity and Sybil resistance** | OpenClaw authenticates devices, not civic identity. Need a separate identity layer. |
| **Campaign-specific UX** | Submission, voting, and result-viewing need purpose-built interfaces beyond chat. |

---

## A Possible v0 Path

Build Collective Will's v0 as **a set of OpenClaw agents and skills** running on a single gateway:

1. **Intake**: users submit concerns via WhatsApp/Telegram/Signal (OpenClaw channels)
2. **Canonicalization**: a dedicated agent processes submissions using an LLM skill
3. **Clustering**: another agent groups canonicalized concerns using a clustering skill
4. **Agenda**: results published to a simple web page (built separately, reading from the evidence store)
5. **Voting**: minimal voting mechanism (could start as simple as reactions in a group chat, or a basic web form)
6. **Action**: action-planning agent drafts letters/emails using civic action skills; users approve via their messaging app
7. **Execution**: executor agent sends approved actions and logs receipts
8. **Auditability**: hooks log every step to markdown files with timestamps

This lets us validate the pipeline design with real users before committing to custom infrastructure. If it works, we can decide later whether to stay on OpenClaw, fork relevant pieces, or build something purpose-built.

---

## Open Questions

- Is OpenClaw's single-gateway model acceptable for v0, or does even a prototype need federation?
- How much of the clustering logic can be a skill (markdown instructions to an LLM) vs. needing custom code?
- OpenClaw skills are instructions to the model, not executable code — is that sufficient for civic action templates, or do we need deterministic execution?
- The memory system uses markdown files — is that durable and structured enough for an evidence store, or do we need a proper append-only log?
- OpenClaw's security model is designed for personal use (one person's agents). How much needs to change for a multi-user civic system where participants don't trust each other?
- What's the licensing situation? OpenClaw is MIT-licensed — does that align with Collective Will's goals for the commons?

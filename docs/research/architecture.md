# System Architecture

## Overview

The system moves a user's concern from raw text to coordinated action through a pipeline of auditable steps. Every transition between steps produces a record. Nothing is lossy — the original input is always reachable from the final output.

```
User submits concern
        │
        ▼
┌─────────────────┐
│ Evidence Store   │◄── append-only, hash-linked
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Canonicalization │── structures raw text into policy candidates
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Clustering       │── groups similar candidates, produces summaries
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Agenda Builder   │── selects daily topics from clusters
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Voting           │── users vote on agenda items
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Action Planning  │── maps winning items to action templates
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Execution        │── actions run with user consent
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Public Audit Log │── execution metadata published
└─────────────────┘
```

## Layers

### Ingestion

Users submit concerns through apps or messaging platforms. Submissions are plain text with optional metadata (location, topic tags). Each submission is hashed and stored before any processing begins.

### Evidence Store

Append-only. Every artifact the system produces — raw submissions, canonical forms, cluster assignments, votes, action plans, execution results — gets a hash-linked entry. This is the backbone of auditability. If it happened, it's in the evidence store. If it's not in the evidence store, it didn't happen.

### Agent Orchestration

The three AI agents (canonicalization, clustering, action planning) run as a pipeline. Each agent reads from the evidence store and writes back to it. Agents are stateless between runs — they process a batch, produce outputs, and those outputs become the next input.

See [agents.md](agents.md) for detailed agent design.

### Voting Service

Users vote on agenda items within a time window. The voting mechanism in v0 is deliberately simple — the design priority is legitimacy and understandability over sophistication. Results are published with full tallies, not just winners.

### Action Execution

Winning items get action plans. In v0, execution requires explicit user consent for each action. The system drafts; the user sends. No autonomous posting, no silent campaigns.

### Public Audit Log

A read-only view of the evidence store. Anyone — user or not — can verify that the pipeline ran correctly: that submissions were faithfully canonicalized, that clusters reflect their members, that votes were tallied correctly, that actions matched the voted intent.

## Policy Lifecycle (Worked Example)

**Alice** lives in Portland. She's frustrated that her rent went up 15% in one year.

1. **Submission.** Alice opens the app and types: "My landlord just raised rent by 15%. There should be a cap on how much rent can go up each year." The system stores her text, timestamps it, and generates a hash.

2. **Canonicalization.** The agent structures her input:
   - *Title:* Annual rent increase cap
   - *Domain:* Housing
   - *Summary:* Advocates for legislative limits on year-over-year residential rent increases
   - *Original hash:* `a3f8c1...`

   Alice can see both her original text and the canonical version. If the canonical version misrepresents her, she can flag it.

3. **Clustering.** Over the past day, 340 other people submitted concerns about housing costs. The clustering agent groups Alice's submission with 127 others under the summary: "Demand for rent stabilization and caps on annual rent increases." The cluster includes her hash. She can verify she's in it.

4. **Agenda.** The agenda builder selects the top clusters by volume and diversity. Rent stabilization makes the daily agenda.

5. **Voting.** All users see today's agenda and vote on which items should trigger action. Rent stabilization gets 4,200 votes. It passes the threshold.

6. **Action planning.** The action agent selects an email-to-representatives template. It fills in the specifics: target representatives for the relevant jurisdictions, a draft message based on the cluster summary, and a subject line. Alice sees the draft.

7. **Execution.** Alice reviews the email, optionally edits it, and hits send. The system records that she participated, what was sent, and to whom.

8. **Audit.** The full chain — her original text, the canonical form, the cluster she was placed in, the vote tally, the action plan, and the execution record — is visible in the public audit log. Anyone can trace the path from submission to action.

### Open Questions

- What database or storage layer is right for the evidence store? Append-only log (Kafka-style)? Merkle tree? Git-like content-addressed store?
- How does the agenda builder select from clusters? Pure volume? Weighted by diversity of geography or demographics? Some other signal?
- What's the voting time window? 24 hours? Does it vary?
- How do you handle real-time vs. batch processing? Are agents triggered on a schedule, or as submissions arrive?
- What are the latency requirements? Does a user expect to see their submission canonicalized in seconds, or is a daily batch acceptable?
- How does the system scale if submission volume grows 100x? Which layers are the bottleneck?

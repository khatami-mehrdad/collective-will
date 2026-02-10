# AI Agents

The system uses three specialized agents to turn raw user input into structured, actionable policy items. None of these agents set the agenda or choose what to act on — they process, organize, and plan. Users decide.

## Canonicalization Agent

**Job:** Turn messy, freeform user submissions into structured policy candidates.

People express concerns in wildly different ways. "Rent is too damn high" and "We need stronger tenant protections against above-inflation rent increases" may point at the same policy area. The canonicalization agent doesn't judge or filter — it normalizes.

**Input:** Raw text from a user submission.

**Output:** A structured policy candidate with:
- A short title
- A policy domain (e.g., housing, environment, labor)
- A plain-language summary of the concern
- Key entities mentioned (if any)
- The original submission hash (for auditability)

**Failure modes:**
- Misinterpreting intent (sarcasm, irony, vague language)
- Imposing framing that the user didn't intend
- Dropping nuance when normalizing

**Constraints:**
- The original submission is always preserved alongside the canonical form
- Users can dispute or correct their canonicalized version
- The agent must not editorialize — structure, don't rewrite

### Open Questions

- What schema should the canonical form follow? Should it be rigid (fixed fields) or flexible (tagged attributes)?
- How do you handle submissions that contain multiple distinct concerns?
- Should the agent attempt to link submissions to known legislation or policy frameworks, or stay domain-agnostic?
- How do you evaluate canonicalization quality at scale?

---

## Clustering Agent

**Job:** Group similar policy candidates together and produce a representative summary for each cluster.

After canonicalization, the system may have hundreds or thousands of structured submissions. The clustering agent finds natural groupings and surfaces what people collectively care about — without anyone manually curating the agenda.

**Input:** A set of canonicalized policy candidates.

**Output:** Per cluster:
- A representative summary
- The number of submissions in the cluster
- A list of member submission hashes
- An explanation of *why* these were grouped together

**Failure modes:**
- Merging distinct concerns that share surface-level language
- Splitting a unified concern into fragments
- Systematic bias in what gets clustered prominently (e.g., English-language bias, recency bias)
- Producing summaries that editorialize rather than represent

**Constraints:**
- Grouping logic must be explainable and published
- Cluster summaries must be traceable back to source submissions
- No cluster is suppressed — all are visible, even small ones

### Open Questions

- What similarity metric works for policy text? Embedding distance? Topic modeling? Something hybrid?
- How do you handle the tension between granularity (many small clusters) and readability (fewer, broader clusters)?
- Should users be able to challenge or split a cluster?
- How do you prevent the clustering step from becoming a subtle editorial filter — the thing that decides what "counts" as a topic?
- Does the clustering agent run once per cycle, or continuously as submissions arrive?

---

## Action Planning Agent

**Job:** Given a policy item that won the vote, produce a concrete action plan.

This agent doesn't decide *what* to do — the users already decided that by voting. It decides *how* to do it by selecting from predefined action templates and filling in the specifics.

**Input:** A voted-on policy item with its cluster summary and metadata.

**Output:** A machine-readable action plan containing:
- Action type (e.g., email campaign, public comment, social media push, petition)
- Target recipients or platforms
- Draft content (from templates, not freeform generation)
- Execution constraints (timing, volume, consent requirements)

**Failure modes:**
- Generating messaging that misrepresents the voted intent
- Selecting an action type that's disproportionate to the issue
- Producing content that violates platform terms of service

**Constraints:**
- Actions use templates, not open-ended generation — the agent selects and fills, not writes
- No action executes without user consent (in v0, explicit consent; later, delegated consent with limits)
- All action plans are published before execution

### Open Questions

- What action types should v0 support? Email to representatives is the obvious starting point — what else?
- How do you template action content without making every message identical (which platforms flag as spam)?
- Who defines the action templates? Are they part of the protocol, or community-contributed?
- How do you handle actions that require platform-specific authentication (e.g., posting on behalf of a user)?
- Should the system support escalation paths (if email doesn't work, try X next)?

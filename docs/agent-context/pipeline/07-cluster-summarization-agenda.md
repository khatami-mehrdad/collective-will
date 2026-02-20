# Task: Cluster Summarization and Agenda Builder

## Depends on
- `pipeline/01-llm-abstraction` (complete() with english_reasoning tier)
- `pipeline/05-hdbscan-clustering` (clusters exist with member candidates)
- `database/03-core-models` (Cluster, PolicyCandidate models)
- `database/04-evidence-store` (append_evidence)

## Goal
Generate human-readable summaries for each cluster using DeepSeek V3.2, then build the voting agenda from clusters meeting the size threshold.

## Files to create

- `src/pipeline/summarize.py` — cluster summarization
- `src/pipeline/agenda.py` — agenda builder

## Specification

### summarize_clusters()

```python
async def summarize_clusters(
    clusters: list[Cluster],
    db: AsyncSession,
) -> list[Cluster]:
    """Generate summaries for clusters that don't have one yet."""
```

Steps:
1. For each cluster without a summary:
   - Load member PolicyCandidates
   - Prepare aggregated content: combine all member titles and summaries into one text block
   - Do NOT include individual submission IDs, user references, or metadata
   - Call `complete()` with `tier="english_reasoning"` and summarization prompt
   - Parse response into `summary` (Farsi) and `summary_en` (English)
2. Update cluster records with summaries
3. Return updated clusters

### Summarization prompt

```
You are summarizing a group of related policy concerns from Iranian citizens.

The following policy positions were clustered together because they address similar concerns:

{aggregated_titles_and_summaries}

Write:
1. A concise summary (2-3 sentences) in Farsi that represents what this group collectively wants
2. An English translation of the same summary
3. A brief explanation (1 sentence) of why these items were grouped together

Output JSON:
{
  "summary_fa": "...",
  "summary_en": "...",
  "grouping_rationale": "..."
}
```

### build_agenda()

```python
async def build_agenda(
    cycle_id: UUID,
    db: AsyncSession,
    min_size: int = 5,
) -> list[UUID]:
    """Select clusters for voting. Returns list of cluster IDs."""
```

Steps:
1. Load all clusters for this cycle
2. Filter to clusters with `member_count >= min_size`
3. Include ALL qualifying clusters — no editorial selection, no ranking, no filtering by topic
4. Log `cycle_opened` preparation to evidence store (the actual cycle_opened event is logged by the voting service)
5. Return the list of qualifying cluster IDs

### Cluster domain assignment

When summarizing, also verify/update the cluster's `domain` field based on the majority domain of its member candidates:

```python
def determine_cluster_domain(candidates: list[PolicyCandidate]) -> PolicyDomain:
    domain_counts = Counter(c.domain for c in candidates)
    return domain_counts.most_common(1)[0][0]
```

## Constraints

- Only aggregated/anonymized content is sent to the LLM. Never individual submissions or user data.
- ALL clusters meeting the size threshold go to voting. No editorial filtering. This is a frozen decision.
- Small clusters (below threshold) are NOT deleted. They remain visible on the analytics dashboard but don't appear in the voting ballot.
- The summarization prompt asks for Farsi output. If DeepSeek's Farsi quality is poor, the pipeline can fall back to getting an English summary and translating via Claude Haiku. Implement this as a configurable option.

## Tests

Write tests in `tests/test_pipeline/test_summarize.py` and `tests/test_pipeline/test_agenda.py` covering:

**Summarization:**
- Cluster with members gets a summary generated (mock LLM response)
- Cluster that already has a summary is skipped
- Aggregated text sent to LLM contains member titles/summaries but no user IDs (mock, inspect prompt)
- LLM returning invalid JSON: cluster flagged, no crash
- Summary stored in both Farsi and English fields

**Agenda:**
- Clusters with member_count >= 5 included in agenda
- Clusters with member_count < 5 excluded
- Empty cluster set returns empty agenda
- All qualifying clusters included (no filtering beyond size)
- `determine_cluster_domain()` returns most common domain
- Domain tie-breaking is deterministic

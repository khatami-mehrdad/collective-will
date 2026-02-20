# Task: LLM Abstraction Layer

## Depends on
- `database/01-project-scaffold` (config with API keys)

## Goal
Create a unified LLM interface with task-based routing to Claude Haiku, DeepSeek V3.2, and Mistral embed. All AI calls go through this abstraction.

## Files to create

- `src/pipeline/llm.py` — LLM abstraction and router

## Specification

### Model tiers

| Tier | Model | Use case | API |
|------|-------|----------|-----|
| `farsi` | Claude Haiku | Canonicalization (Farsi→English), user-facing Farsi messages | Anthropic Messages API |
| `english_reasoning` | DeepSeek V3.2 | Cluster summaries, complex English reasoning | DeepSeek API (OpenAI-compatible) |
| `embedding` | Mistral `mistral-embed` | Compute vectors for clustering | Mistral Embeddings API |

### complete() function

```python
async def complete(
    prompt: str,
    tier: Literal["farsi", "english_reasoning"],
    system_prompt: str | None = None,
    max_tokens: int = 1024,
    temperature: float = 0.0,
) -> LLMResponse:
```

- Routes to the correct API based on tier
- Uses httpx async client
- Returns structured response

```python
class LLMResponse(BaseModel):
    text: str
    model: str                   # Actual model name used
    input_tokens: int
    output_tokens: int
    cost_usd: float              # Estimated cost
```

### embed() function

```python
async def embed(texts: list[str]) -> list[list[float]]:
```

- Calls Mistral `mistral-embed` API
- Batches if list is too long (Mistral has batch limits)
- Returns list of embedding vectors

### API clients

Use httpx async for all APIs:

**Anthropic (Claude Haiku)**:
- Endpoint: `https://api.anthropic.com/v1/messages`
- Headers: `x-api-key`, `anthropic-version: 2023-06-01`
- Model: `claude-3-haiku-20240307` (or latest haiku)

**DeepSeek V3.2**:
- Endpoint: `https://api.deepseek.com/v1/chat/completions` (OpenAI-compatible)
- Headers: `Authorization: Bearer {api_key}`
- Model: `deepseek-chat`

**Mistral embed**:
- Endpoint: `https://api.mistral.ai/v1/embeddings`
- Headers: `Authorization: Bearer {api_key}`
- Model: `mistral-embed`

### Cost tracking

Estimate cost per call based on token counts and known pricing. Log to stdout or a cost tracking table (simple for v0).

### Error handling

- Retry transient errors (429, 500, 502, 503) up to 3 times with exponential backoff
- Timeout: 60 seconds per call
- Raise clear exceptions for auth errors (401) and bad requests (400) — do not retry these

## Constraints

- All LLM calls go through this abstraction. No direct API calls from other modules.
- The abstraction must be easy to swap backends (e.g., switch DeepSeek to a local model later).
- Log the model name and version with every call (for evidence store reproducibility).
- Do NOT send user IDs, account references, or metadata to any LLM API. Only text content.

## Tests

Write tests in `tests/test_pipeline/test_llm.py` covering:
- `complete()` with `tier="farsi"` calls Anthropic API (mock httpx, verify correct endpoint and headers)
- `complete()` with `tier="english_reasoning"` calls DeepSeek API (mock httpx)
- `embed()` calls Mistral API (mock httpx)
- `embed()` with large batch splits into sub-batches
- LLMResponse contains correct model name and token counts
- Retry logic: transient 429 error retried, succeeds on 2nd attempt
- Auth error (401) not retried, raises immediately
- Timeout fires after configured duration
- Cost estimate is non-negative and reasonable

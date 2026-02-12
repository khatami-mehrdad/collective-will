# Research: Evaluating LLM-Based Canonicalization of Civic Submissions

Research findings on evaluating LLM normalization of freeform policy submissions into structured candidates.

---

## 1. Academic Research on Text Canonicalization/Normalization

### Intent Preservation Measurement

**Natural Language Inference (NLI)** is the primary academic framework for measuring whether meaning is preserved during text transformation. The Stanford NLI corpus (570K human-labeled sentence pairs) defines three relations: entailment, contradiction, or neutral. For canonicalization evaluation, the goal is **bidirectional entailment**—the original should entail the structured output and vice versa.

Key insight: 98% of SNLI cases achieve 3-annotator consensus, and 58% achieve unanimous 5-annotator agreement, indicating that humans can reliably judge intent preservation when properly prompted.

### Paraphrase Quality Metrics

Recent research (2024-2025) has identified critical problems with traditional metrics:

| Metric Type | Examples | Limitation |
|-------------|----------|------------|
| **N-gram based** | BLEU, ROUGE | Penalize valid paraphrases that use different words |
| **Reference-based** | Most traditional metrics | Reference-free can actually outperform |
| **Lexical distance** | Edit distance | Paraphrases *require* lexical divergence |

**Better approaches:**

- **ParaPLUIE** (2025): Uses LLM log-likelihood ratios to evaluate semantic proximity independent of surface-level lexical similarity. Provides interpretable thresholds between paraphrases and non-paraphrases.
  - *Citation: ACL 2025 COLING, "Paraphrase Generation Evaluation Powered by an LLM"*

- **PAM (Paraphrase AMR-Centric)** (2025): Uses Abstract Meaning Representation graphs for evaluation agnostic to surface form.
  - *Citation: ACL 2025 Findings, "PAM: Paraphrase AMR-Centric Evaluation Metric"*

- **BERTScore**: Computes token-level similarity using contextual BERT embeddings with IDF weighting. Better correlation with human judgments than BLEU/ROUGE. Robust against adversarial paraphrases.
  - *Citation: Zhang et al., ICLR 2020*

- **MPNet**: Empirically outperforms BERT and RoBERTa for semantic similarity (75.6% on MRPC), though optimal thresholds vary significantly (0.334–0.867).
  - *Citation: MDPI Computers 2024*

### Policy Position Extraction Research

**Wordscores** (Laver, Benoit, Garry) is the foundational computational method:
- Treats text as quantitative word data rather than discourse to interpret
- Language-blind word scoring replicates expert policy estimates
- Provides uncertainty measures for distinguishing real differences from measurement error
- Validated against Comparative Manifesto Project and expert surveys

*Citation: Laver, Benoit & Garry, APSR 2003; Evaluation by Hjorth et al. 2015*

**Comparative Manifesto Project (CMP)** is the standard schema in political science:
- Coding unit: "quasi-sentence" (single statement, ≤1 grammatical sentence)
- Categories organized into domains: Foreign Policy (101-110), Democracy (201+), Economy, Welfare, Social Groups, etc.
- Current version: 5th edition (2021)
- Available as CSV with codes, domains, titles, descriptions

---

## 2. Evaluation Approaches

### Human Evaluation Protocols

**HEDS 3.0 (Human Evaluation Datasheet)** is the current standard framework:

| Section | What to Document |
|---------|------------------|
| Reference info | Paper, date, contact |
| Evaluated systems | Models compared |
| Sample/evaluators/design | Sample size, rater count, design |
| Quality criteria | Rubric definitions |
| Ethics | Compensation, consent |

**Key findings on rater requirements:**

- **Inter-rater agreement**: Use ICC (intra-class correlation coefficient) rather than percent agreement
- **Sample size**: Acceptance sampling can reduce required samples by up to 50% with same statistical guarantees
- **Power analysis**: Most NLP studies are underpowered; small test sets make it hard to detect real differences
- **Ordinal data**: Don't treat rating scales as interval data—use ordinal mixed effects models
- *Citation: ACL 2024, "On Efficient and Statistical Quality Estimation for Data Annotation"*
- *Citation: EMNLP 2021, "What happens if you treat ordinal ratings as interval data?"*
- *Citation: EMNLP 2020, "With Little Power Comes Great Responsibility"*

**Practical rater guidelines:**
- Minimum 3 raters per item for agreement calculation
- 5 raters enables unanimous agreement tracking (used in SNLI)
- Use confidence intervals to determine minimum sample size for error estimation

### Automated Metrics

**For canonicalization, combine:**

1. **Semantic similarity** (meaning preserved):
   - BERTScore or sentence-transformers cosine similarity
   - Threshold: 0.75+ typically indicates strong similarity

2. **NLI-based entailment** (no information added/lost):
   - Use an NLI model to check: does original entail structured? Does structured entail original?
   - Bidirectional entailment = good canonicalization
   - Models: DeBERTa-v3-large-mnli-fever-anli, nli-roberta-large

3. **Schema compliance** (output validity):
   - JSON Schema validation
   - Required field completeness
   - Entity extraction accuracy (precision/recall if gold labels exist)

### Talk to the City Evaluation Approach

TTTC emphasizes **traceability over metrics**:
- Each cluster summary links to participant quotes
- Users can drill down from themes to original statements
- Design principle: "grounding" mitigates LLM inaccuracies
- Focus on auditable representation rather than automated scores

*Source: AI Objectives Institute documentation*

### Polis Approach

Polis uses a fundamentally different strategy:
- **No NLP on text content**—operates purely on the vote matrix (agree/disagree/pass)
- K-means clustering on voting patterns, not semantic similarity
- Dimensionality reduction (PCA/UMAP) for visualization
- Success measured by: consensus discovery, reduced trolling, practical deployment

*Citation: Small et al., RECERCA 2021*

---

## 3. Schema Design for Policy Positions

### Existing Schemas

**POLIANNA (Policy Design Annotations)**:
- Coding scheme translating policy design taxonomies to span annotations
- Enables supervised ML for policy analysis
- Challenges: non-textual elements, document structure, flexible annotation, global information integration
- *Citation: Sewerin & Kaack et al., Semantic Scholar*

**Legislative Data Standards:**

| Standard | Description | Use Case |
|----------|-------------|----------|
| **Akoma Ntoso** | XML standard for legislative acts | International legislative documents |
| **ELI (European Legislation Identifier)** | Ontology for legislation metadata | EU Member State legal publishing |
| **LegalRuleML** | XML for legal rules | Machine-readable legal logic |
| **Property Graphs** | Nodes=laws/articles, edges=citations/modifications | Legislative interdependency analysis |

### Rigid vs. Flexible Schema Tradeoffs

| Approach | Pros | Cons |
|----------|------|------|
| **Rigid schema** | Consistent, comparable, easier validation | Loses nuance, forces false categorization |
| **Flexible schema** | Captures edge cases, preserves intent | Harder to aggregate, cluster, compare |
| **Hybrid** | Core required fields + free-form extensions | Best for canonicalization |

**Recommendation for Collective Will:**

```typescript
interface PolicyCandidate {
  // Required (rigid)
  id: string;
  title: string;                    // 5-15 word summary
  domain: PolicyDomain;             // Enum: housing, transit, environment, etc.
  stance: 'support' | 'oppose' | 'neutral';
  
  // Required but flexible
  summary: string;                  // 1-3 sentence expansion
  
  // Optional structured
  entities?: string[];              // Named entities mentioned
  geographicScope?: 'local' | 'regional' | 'national';
  
  // Flexible extension
  originalText: string;             // Always preserve source
  confidence: number;               // LLM self-reported confidence
  ambiguityFlags?: string[];        // e.g., ["multiple_issues", "sarcasm_possible"]
}
```

### Handling Multi-Issue Submissions

Multi-label stance detection research shows:

1. **Split into atomic statements**: One submission → multiple PolicyCandidates
2. **Label dependencies matter**: Train models that understand correlations (e.g., "rent control" often co-occurs with "affordable housing")
3. **Joint prediction**: Predicting all stances together outperforms independent prediction

*Citation: Ferreira & Vlachos, EMNLP 2019, "Incorporating Label Dependencies in Multilabel Stance Detection"*

**Practical approach:**
- LLM identifies if submission contains multiple distinct policy positions
- If yes, generate separate PolicyCandidates with shared `originalText`
- Track which portion of original maps to which candidate

---

## 4. LLM Failure Modes for This Task

### Editorializing vs. Structuring

**The core risk**: LLMs make unnecessary changes beyond the stated task.

Research findings:
- When debiasing news, ChatGPT introduced unrelated changes affecting author style and potentially creating misinformation
- In Wikipedia neutrality editing, LLMs made extraneous corrections (grammar, style) beyond NPOV rules
- *Citation: arXiv 2404.06488, "Pitfalls of Conversational LLMs on News Debiasing"*

**Mitigations:**
- Explicit instructions: "Structure only. Do not rephrase, editorialize, or add context."
- Output original text alongside structured fields for comparison
- Automated checks for semantic drift (BERTScore < threshold = flag)

### Sarcasm, Irony, Ambiguity

LLMs struggle significantly with sarcasm:

| Challenge | Description |
|-----------|-------------|
| **Speaker/observer mismatch** | What speaker intends ≠ what observer perceives |
| **Perspective bias** | LLMs default to speaker's perspective |
| **Context dependency** | Sarcasm requires understanding normative expectations |

*Citation: ACL 2025 IWCS, "The Difficult Case of Intended and Perceived Sarcasm"*

**Best approaches for smaller models:**
- Chain-of-contradiction prompting
- Explicit inconsistency scoring
- Flag ambiguous cases for human review rather than guessing

**Practical mitigation:**
- Add `ambiguityFlags` field to schema
- If confidence < threshold, route to human review
- Never assume literal interpretation is correct for short, emphatic statements

### Bias in Framing

Critical finding: **LLM annotations can produce any result you want** with prompt manipulation.

- 31% incorrect conclusions even with SOTA models
- 50% incorrect with smaller models
- "With just a handful of prompt paraphrases, virtually anything can be presented as statistically significant"
- Bias can amplify post-edit, especially regarding demographics
- *Citation: arXiv 2509.08825, "LLM Hacking: Quantifying Hidden Risks"*

**Mitigations:**
- Use neutral, non-leading prompts
- Test prompt variations to ensure stability
- Compare LLM output distributions against human baselines
- Audit for demographic/political bias in outputs

---

## 5. Practical Recommendations

### Sample Size for Evaluation

**For human evaluation:**

| Purpose | Minimum Sample | Recommended |
|---------|----------------|-------------|
| Initial validation | 100 items | 200+ items |
| Per-rater reliability | 50 overlapping items | 100 overlapping |
| Production monitoring | 5-10% ongoing sample | 10% with stratification |

**Statistical power considerations:**
- Effect sizes in NLP are often small (d ≈ 0.2-0.5)
- Most published evaluations are underpowered
- Use power analysis tools: [github.com/dallascard/NLP-power-analysis](https://github.com/dallascard/NLP-power-analysis)

**Recommended evaluation rubric:**

| Criterion | 1 (Poor) | 2 (Fair) | 3 (Good) |
|-----------|----------|----------|----------|
| **Meaning preserved** | Meaning changed or lost | Minor nuance lost | Fully captured |
| **No editorializing** | Added opinion/framing | Minor additions | Purely structural |
| **Schema fit** | Wrong domain/fields | Debatable choices | Clear correct fit |
| **Completeness** | Missing key elements | Partial coverage | All relevant info |

### Model Capability Assessment (Qwen3-8B Class)

**Qwen3-8B benchmarks:**
- GSM8K: 82.7% (math reasoning)
- CEVAL: 81.5% (Chinese evaluation)
- Strong instruction-following, agent capabilities, 100+ language support
- Supports 32K context natively, 131K with YaRN extension

**Likely sufficient for canonicalization if:**
- Task is well-specified in prompt
- Schema is not overly complex (≤10 fields)
- Domain categories are clear and enumerable
- Multi-issue detection is explicitly prompted

**Potential limitations:**
- Sarcasm/irony detection will be weaker than frontier models
- Complex multi-step reasoning may degrade
- May need higher-tier model for ambiguous cases

**Recommendation:** Start with Qwen3-8B, establish baseline, compare against frontier model on difficult subset to quantify gap.

### Best Practices from Similar Projects

**From Talk to the City:**
- Always preserve link to original text
- Design for drilldown (summary → cluster → quotes)
- Assume LLMs will make mistakes; design for auditability

**From Brazil's National Participation Platform:**
- Use BERTopic + seed words for clustering
- LLM validation layer after clustering
- Scale: 50K+ proposals successfully processed

**From France's Grand Débat National:**
- Segment submissions into sentences before analysis
- Community detection algorithms for topic clustering
- 225K contributions analyzed

**From Polis:**
- Consider separating voting/ranking from content analysis
- Consensus discovery is often more valuable than perfect classification
- Simple approaches (vote matrix) can outperform NLP in practice

### Implementation Checklist

1. **Prompt engineering**
   - [ ] Test 5+ prompt variations for stability
   - [ ] Include examples spanning edge cases (sarcasm, multi-issue, ambiguous)
   - [ ] Explicit "do not editorialize" instructions

2. **Automated evaluation pipeline**
   - [ ] Schema validation (JSON Schema)
   - [ ] Semantic similarity check (BERTScore ≥ 0.75)
   - [ ] NLI entailment check (bidirectional)
   - [ ] Confidence threshold for human review routing

3. **Human evaluation baseline**
   - [ ] 200+ items annotated by 3+ raters
   - [ ] Inter-rater agreement ≥ 0.7 ICC
   - [ ] Error taxonomy documented

4. **Ongoing monitoring**
   - [ ] 10% sample for continuous evaluation
   - [ ] Drift detection on domain distribution
   - [ ] User feedback integration

---

## Key Citations

### Paraphrase and Semantic Similarity
- Zhang et al. (2020). BERTScore: Evaluating Text Generation with BERT. ICLR.
- ACL 2025 COLING. ParaPLUIE: LLM-based paraphrase evaluation.
- ACL 2025. PARAPHRASUS: Multi-dimensional paraphrase benchmark.

### Policy Position Extraction
- Laver, Benoit & Garry (2003). Extracting policy positions using words as data. APSR.
- Comparative Manifesto Project: [manifestoproject.wzb.eu](https://manifestoproject.wzb.eu)
- POLIANNA Dataset: [github.com/kueddelmaier/POLIANNA](https://github.com/kueddelmaier/POLIANNA)

### Human Evaluation
- HEDS 3.0 (2024). arXiv:2412.07940.
- Card et al. (2020). With Little Power Comes Great Responsibility. EMNLP.
- ACL 2024. On Efficient and Statistical Quality Estimation for Data Annotation.

### LLM Failure Modes
- arXiv 2509.08825. LLM Hacking: Quantifying Hidden Risks.
- arXiv 2404.06488. Pitfalls of Conversational LLMs on News Debiasing.
- ACL 2025 IWCS. The Difficult Case of Intended and Perceived Sarcasm.

### Civic Tech / Deliberation
- Talk to the City: [ai.objectives.institute/talk-to-the-city](https://ai.objectives.institute/talk-to-the-city)
- Small et al. (2021). Polis: Scaling Deliberation. RECERCA.
- Brazil's National Participation Platform: arXiv:2509.21292.

### Structured Output
- JSONSchemaBench (2025). arXiv:2501.10868.
- Cleanlab analysis: [cleanlab.ai/blog/structured-output-benchmark](https://cleanlab.ai/blog/structured-output-benchmark)

---

## Summary

For evaluating LLM canonicalization of civic submissions:

1. **Combine automated + human evaluation**: BERTScore/NLI for scaling, human rubrics for ground truth
2. **Preserve traceability**: Always link structured output to original text
3. **Design for failure**: Flag ambiguous cases, route low-confidence to humans
4. **Start with hybrid schema**: Required core fields + flexible extensions
5. **Budget for 200+ human-evaluated items** for reliable baseline
6. **Qwen3-8B likely sufficient** for straightforward cases; reserve frontier models for ambiguous routing
7. **Monitor for editorializing** as the primary failure mode

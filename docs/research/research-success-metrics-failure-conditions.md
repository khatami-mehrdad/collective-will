# Success Metrics and Failure Conditions for Civic Technology Platforms

Research compiled for Collective Will v0 planning.

---

## 1. Success Metrics from Prior Art

### Polis / vTaiwan

**What made it "successful":**
- **Policy impact**: 80% of 28+ cases discussed led to decisive government action
- **Scale**: 200,000+ on mailing list; companion platform JOIN reached >50% of Taiwan's population
- **Longevity**: Longest-running national-scale Polis deployment
- **Trust mechanism**: Mandatory "Participation Officers" (civil servants with domain expertise) ensured policy relevance

**Key insight**: Success = policy outcomes, not just participation numbers.

### Decidim (Barcelona)

**Metrics achieved:**
- 149,279 registered users (as of Sept 2022)
- 10,860 proposals → 18,191 comments → 165,087 support indications (2016 planning project)
- ~20 social organizations adopted it for their own processes

**What they learned:**
- Proposals from collaborative processes (civic associations, offline meetings) had **significantly higher acceptance rates** than individual citizen proposals
- Hybrid online/offline participation drives quality
- Network analysis showed genuine grassroots engagement clusters

### Community Notes (Twitter/X)

**Accuracy factors identified:**
- Notes linking to trustworthy sources rated more helpful
- Notes on posts from high-follower accounts yield **lower consensus** among raters
- Each contributor has tracked "writing impact" and "rating impact" scores
- Challenge: opinion speculation and polarization affect quality

**Key insight**: Credibility of sources matters more than note volume.

### Change.org

**Scale metrics:**
- 571 million users taking action globally
- 112,485 victories across 196 countries
- ~12 successful petitions per day
- ~1 victory per hour

**Critical insight**: They measure **civic participation rate** (1/3 of users signed a winning petition) rather than just victory count. Users who participate in wins are much more likely to re-engage.

### Resistbot

**Output metrics:**
- 30+ million letters sent to elected officials since 2017
- 12,000+ net votes generated in 2018 midterms (independent study)
- 10,000+ voter registration forms pre-filled/sent

**Delivery success tracking**: Weighted contactability scores (email 70%, fax 25%, phone 5%) with intelligent routing.

### Talk to the City

**Design metrics:**
- Can process input from 50 to 5+ million people
- Every theme grounded directly in participant quotes (auditability)
- Tested in Taiwan, Michigan, AI governance consultations

**Key insight**: Auditability (linking summaries to original quotes) is essential for trust.

---

## 2. Trust and Legitimacy Metrics

### OECD Five Drivers of Trust in Democratic Systems

| Driver | Measurement Approach |
|--------|---------------------|
| **Reliability** | Consistency in system performance |
| **Responsiveness** | Speed/quality of addressing user concerns |
| **Integrity** | Ethical conduct, adherence to stated principles |
| **Openness** | Transparency, access to information |
| **Fairness** | Equitable treatment across user groups |

### Behavioral Indicators

| Metric | What It Indicates | Target Range |
|--------|-------------------|--------------|
| **Return usage** | Trust that time is well-spent | >20% return within 30 days |
| **Referral rate** | Willingness to stake reputation | >5% organic referrals |
| **Completion rate** | Trust in process | >70% who start, finish |
| **Dispute/challenge rate** | AI output acceptance | <10% of outputs challenged |
| **Appeal resolution satisfaction** | Fair dispute handling | >80% satisfied with resolution |

### External Validation Approaches

- **Independent audits**: Third-party review of AI clustering/canonicalization
- **Academic partnerships**: Peer review of methodology
- **Stakeholder review**: Target officials/orgs assess output quality
- **Comparison studies**: Does output match manual analysis?

### Survey Metrics (periodic)

- "Do you trust this system to fairly represent your views?" (1-5 scale)
- "Would you recommend this to a friend?" (NPS)
- "Did you feel your input mattered?" (1-5 scale)
- "Was the AI summary accurate?" (1-5 scale)

---

## 3. Participation Metrics

### Submission Rates

| Metric | Calculation | Healthy Range |
|--------|-------------|---------------|
| **Submission rate** | Submissions / active users | >10% weekly |
| **Submission quality** | % meeting minimum criteria | >80% |
| **Unique submitters** | % of users who ever submit | >30% over lifetime |

### Voting Participation

| Metric | Benchmark (from research) |
|--------|---------------------------|
| **Voting participation** | 10-30% of registered users typical |
| **Participatory budgeting** | NYC averaged 30,700 votes/year citywide |
| **vTaiwan high-engagement** | >50% population reached (companion platform) |

**Warning sign**: <5% voting participation suggests insufficient engagement.

### Action Completion

| Stage | Target Conversion |
|-------|-------------------|
| View drafted action → Approve | >50% |
| Approve → Send | >80% |
| Overall (view → send) | >40% |

### Retention and Repeat Participation

**Baseline reality** (from research):
- >90% of online civic platform participants are one-time users
- <10% retention after one year is typical
- Hands-on programs achieve 54% retention after one year

**Targets for v0:**
| Timeframe | Retention Target |
|-----------|------------------|
| Week 1 | >30% return |
| Month 1 | >15% still active |
| Month 3 | >8% still active |
| Month 6 | >5% still active |

### Geographic/Demographic Diversity

**From participatory budgeting research:**
- PB increased voting probability by 8.4 percentage points
- Greater effects for underrepresented groups (young, lower-income, minorities)
- BUT: Typical participants skew highly educated (86% bachelor's+) and higher income

**Metrics to track:**
- Geographic distribution (zip codes, districts)
- Age distribution vs. target population
- First-time civic participants (% who've never engaged before)

---

## 4. Pipeline Quality Metrics

### Canonicalization Accuracy

| Metric | Method | Target |
|--------|--------|--------|
| **Semantic preservation** | Human eval: "Does summary capture original meaning?" | >85% agreement |
| **Completeness** | Human eval: "Are key points preserved?" | >90% |
| **Neutrality** | Human eval: "Is summary biased?" | <10% flagged |
| **LLM-human alignment** | Compare LLM ratings to human panel | r > 0.8 correlation |

**Evaluation protocol** (from Talk to the City):
- Every theme must be grounded in participant quotes
- Provide audit trail from summary → original submissions

### Clustering Coherence

| Metric | Method | Target |
|--------|--------|--------|
| **Intra-cluster similarity** | Are items in same cluster related? | >80% human agreement |
| **Inter-cluster distinctiveness** | Are clusters meaningfully different? | >75% can identify difference |
| **Granularity appropriateness** | Right number of clusters? | Subjective + stability test |
| **Stability** | Same inputs → similar clusters? | >85% overlap on re-run |

**From NLP research:**
- LLM-based evaluation proxies can be statistically indistinguishable from human annotators
- Traditional coherence metrics often correlate poorly with human preference
- "Consistency" metric (% of topic switches) correlates most with human judgment

### Action Effectiveness

| Metric | Measurement |
|--------|-------------|
| **Response rate** | % of letters that get any response |
| **Substantive response rate** | % that get non-form-letter response |
| **Time to response** | Average days from send to response |
| **Resolution rate** | % where stated concern was addressed |

**Resistbot benchmark**: Tracks delivery success by channel, with intelligent routing to maximize delivery.

### False Positive/Negative Rates

| Error Type | Definition | Target |
|------------|------------|--------|
| **False positive (clustering)** | Items grouped that shouldn't be | <15% |
| **False negative (clustering)** | Related items not grouped | <20% |
| **False positive (canonicalization)** | Summary adds content not present | <5% |
| **False negative (canonicalization)** | Summary omits key content | <10% |

---

## 5. Impact Metrics

### Official Response Tracking

| Metric | Measurement |
|--------|-------------|
| **Acknowledgment rate** | % of campaigns acknowledged by target |
| **Meeting rate** | % that result in constituent meetings |
| **Public statement rate** | % where official addresses the issue publicly |
| **Policy change rate** | % that contribute to observable policy change |

**Attribution challenge** (from advocacy research):
- Establishing causal links between advocacy and policy change is the greatest challenge
- Use **contribution analysis**: Evidence-based stories of how advocacy contributed
- Track "touchpoints" between campaign and outcome

### Media Coverage

| Metric | Purpose |
|--------|---------|
| **Mentions** | Raw visibility |
| **Sentiment** | Positive/negative framing |
| **Reach** | Audience of covering outlets |
| **Message accuracy** | Did coverage reflect campaign's core message? |

### How Advocacy Organizations Measure Impact

**Four levels of evaluation:**
1. **Strategy and direction** - Are we targeting the right issues?
2. **Management and outputs** - Are we executing well?
3. **Outcomes and impact** - What changed?
4. **Understanding causes** - Did we cause the change?

**American Heart Association framework**: Assess health/equity impact of legislation over time through researcher partnerships.

---

## 6. Failure Conditions

### Structural Failure Modes (from research)

| Failure Type | Description |
|--------------|-------------|
| **Timeline misalignment** | Research/funding cycles (3-5 years) don't match community needs (ongoing) |
| **Infrastructure integration failure** | Cannot integrate into government structures or secure stable funding |
| **Sociotechnical gaps** | Technical system doesn't fit organizational/social context |
| **Volunteer burnout** | Unsustainable reliance on unpaid labor |
| **Financial unsustainability** | Costs exceed revenue (see: Benefits Data Trust) |

### Benefits Data Trust Cautionary Tale

- 20 years of operation, ~300 employees
- Millions in philanthropic + government funding
- Shut down suddenly in 2024
- Root cause: Spent far more than brought in, sustained deficit
- Warning: Impact reports and audits showed no warning signs

### The Zombie Company Problem

Most insidious failure mode: **"Neither growing nor truly dead"**
- Requires no active decision
- Easy to continue doing bare minimum
- Harder to recognize than outright failure
- Drains resources without impact

### Trust Breakdown Scenarios

| Scenario | Warning Signs | Recovery Difficulty |
|----------|---------------|---------------------|
| **AI output consistently wrong** | Dispute rate >30%, accuracy <70% | Moderate (technical fix) |
| **Perceived bias** | Demographic skew in outputs, user complaints | High (perception sticky) |
| **Gaming/manipulation** | Coordinated submissions, astroturfing detected | Very high |
| **Data breach/privacy violation** | Any PII exposure | Critical (may be fatal) |
| **Official backlash** | Targets publicly dismiss/discredit platform | High |
| **Media narrative turns negative** | Coverage frames platform as ineffective/harmful | High |

---

## 7. Kill Criteria Examples

### From Startup Methodology

**STOP statements** (end the project):
- Core assumption invalidated: Users don't care about the problem
- Product-market fit impossible: No differentiation, no value delivery
- Continued investment without traction

**PIVOT statements** (change direction):
- Current approach shows declining metrics despite fixes
- Adjacent opportunity shows stronger signal
- External conditions changed (policy, technology, competition)

**INVEST statements** (continue despite challenges):
- Metrics improving, even if slowly
- Clear path to removing blockers
- Strong qualitative signals (user enthusiasm)

### Civic Tech Specific Kill Criteria

| Condition | Threshold | Timeframe |
|-----------|-----------|-----------|
| **Zero policy impact** | No official responses to any campaign | 6 months |
| **Participation collapse** | <100 active users | 3 months post-launch |
| **Trust crisis** | >50% negative survey responses | Any point |
| **Financial runway exhausted** | <3 months operating capital | Rolling |
| **Team departure** | Core team leaves, no succession | Any point |
| **Legal/regulatory block** | Platform operation prohibited | Any point |

### How Other Projects Defined Failure

**vTaiwan**: Continues as volunteer effort despite losing government support - different definition of "success" (civic lab vs. policy tool)

**Failed Yet Successful research**: Argues discontinued projects should be seen as learning opportunities, not failures. Key question: Did it generate knowledge/capacity that outlived the project?

---

## 8. Recommendations for Collective Will v0

### Minimum Metrics to Track from Day One

**Must track:**

| Category | Metric | Tool/Method |
|----------|--------|-------------|
| **Participation** | Daily/weekly active users | Analytics |
| **Participation** | Submissions per user | Database |
| **Participation** | Voting participation rate | Database |
| **Quality** | AI output dispute rate | User feedback button |
| **Quality** | Action completion rate | Database |
| **Trust** | Post-action satisfaction (1-5) | In-app survey |
| **Impact** | Letters sent | Database |
| **Impact** | Responses received | Manual tracking |

**Should track:**
- Referral source (how did you hear about us?)
- Time to complete each step
- Drop-off points in funnel
- Geographic distribution

**Nice to have:**
- Demographic survey (optional)
- Longitudinal trust survey
- External audit of AI quality

### Success Criteria by Timeframe

#### 3-Month Success (Proof of Concept)

| Metric | Target | Rationale |
|--------|--------|-----------|
| **Users registered** | >500 | Minimum for statistical validity |
| **Submissions** | >200 | Enough to test clustering |
| **Voting participation** | >20% of registered | Baseline engagement |
| **Action completion** | >30% of approved | Users trust the output |
| **At least 1 official response** | Yes/No | Impact signal |
| **User satisfaction** | >3.5/5 average | Trust indicator |
| **Return rate (30-day)** | >15% | Retention baseline |

#### 6-Month Success (Validation)

| Metric | Target | Rationale |
|--------|--------|-----------|
| **Users registered** | >2,000 | Growing |
| **Monthly active users** | >500 | Sustained engagement |
| **Campaigns completed** | >10 | Demonstrated pipeline |
| **Official response rate** | >20% | Platform taken seriously |
| **User satisfaction** | >4.0/5 average | Trust building |
| **Organic referral rate** | >10% | Word of mouth |
| **Media mention** | >1 | External validation |
| **AI accuracy (audited)** | >80% | Quality threshold |

### Red Line Failure Conditions

**Immediate shutdown triggers:**
- Data breach exposing user PII
- Platform used for harassment/abuse that causes real harm
- Legal cease-and-desist that cannot be resolved

**3-month failure (pivot or stop):**
- <100 registered users
- <50 submissions total
- >40% negative satisfaction ratings
- Zero official engagement with any campaign
- AI clustering accuracy <60% on human evaluation

**6-month failure (serious reconsideration):**
- <500 registered users
- <10% voting participation
- <5% action completion rate
- Zero substantive official responses
- Declining month-over-month metrics for 3+ consecutive months
- Operating costs unsustainable with current resources

### Warning Signs to Watch

| Signal | Interpretation | Response |
|--------|----------------|----------|
| High registration, low submission | Friction in submission flow | UX research |
| High submission, low voting | Agenda not compelling or voting UX broken | Test both |
| High voting, low action completion | Don't trust AI output or action quality | Quality audit |
| High action completion, zero responses | Targeting wrong officials or bad delivery | Channel audit |
| Declining return rate | Not providing value | User interviews |
| Geographic concentration | Not reaching diverse communities | Outreach strategy |
| Dispute rate rising | AI quality declining or expectations misaligned | Technical + comms review |

---

## Summary: The v0 "Earning Trust" Scorecard

For a system focused on earning trust, weight metrics accordingly:

| Priority | Metric Category | Weight |
|----------|-----------------|--------|
| **Highest** | Trust/satisfaction scores | 30% |
| **Highest** | AI accuracy (human-evaluated) | 25% |
| **High** | Action completion rate | 20% |
| **Medium** | Participation numbers | 15% |
| **Medium** | Impact (official responses) | 10% |

**Core thesis to validate:**
> "Users who go through the full pipeline (submit → vote → approve action → send) believe the system fairly represented their concerns and would use it again."

**Minimum viable validation:**
- 50+ users complete full pipeline
- >70% report satisfaction ≥4/5
- >50% say they'd use it again
- At least 1 official acknowledges receiving collective input

If these conditions aren't met after 3 months of active operation, the core value proposition needs fundamental rethinking.

---

## Sources

- Polis/vTaiwan case studies (compdemocracy.org)
- Decidim Barcelona research (UOC, TICTeC)
- Community Notes research (Twitter/GitHub, arxiv)
- Change.org impact reports
- Resistbot effectiveness data
- Talk to the City documentation (AI Objectives Institute)
- OECD Trust and Democracy reports
- Advocacy evaluation frameworks (BetterEvaluation, AHA)
- Civic tech discontinuation research (CHI 2023)
- Startup kill criteria (Y Combinator, Built In)
- Participatory budgeting studies (Cambridge, Springer)

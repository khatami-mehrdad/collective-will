# Research: Action Types and Templates for Civic Action System

This document captures research on action types and template design for a civic action system, with specific focus on a pilot targeting Iranians (inside Iran and diaspora).

**Last Updated:** February 2026

## 1. Action Types for Civic Engagement

### Tier 1: Low-Friction Actions (High Volume)

**Email campaigns to officials**
- Most common and effective digital advocacy method — FiscalNote's analysis of 585 million advocacy engagements confirms email as the #1 success driver
- Congressional Management Foundation research: 90% of staff say individualized postal mail and 88% say individualized email have "a lot of positive influence" on undecided Members
- Works best when: personalized, locally relevant, tied to specific legislation
- Challenge: Email deliverability requirements have tightened significantly (SPF, DKIM, DMARC required for bulk senders as of 2025)

**Petition signing**
- Resistbot model: users sign with a single keyword, platform handles delivery
- Effective for showing volume of support
- Less impactful per-signature than personalized letters
- Good for building initial engagement before escalating to higher-effort actions

**Social media amplification**
- Sharing pre-written content, coordinated hashtags
- Must carefully avoid "coordinated inauthentic behavior" flags (see Platform Constraints below)
- Effective for visibility and media pickup
- Works across borders without deliverability concerns

### Tier 2: Medium-Friction Actions (Higher Impact)

**Phone banking / call campaigns**
- Congressional staff report phone calls as highly influential (94-99% say "some" or "a lot" of influence)
- Requires more committed advocates
- Can be coordinated with scripts but calls themselves must be individual
- Limited applicability for Iran (targeting foreign officials requires different approach)

**Letter-writing campaigns (physical mail)**
- Still valued by legislative offices for demonstrating effort
- Resistbot handles delivery via fax and postal mail
- Higher friction but signals more commitment
- Useful for US Congress, EU Parliament targets

**Public comment submissions**
- Government regulatory processes often have comment periods
- Technical but high-impact when coordinated
- Relevant for US/EU policy on Iran sanctions, visa policies, etc.

### Tier 3: High-Friction Actions (Specialist)

**FOIA / Freedom of Information requests**
- Useful for transparency campaigns
- Requires legal knowledge to craft effective requests
- Could be templated for common request types
- Relevant for investigating government dealings with Iranian regime

**Coordinated media outreach**
- Press release distribution, journalist contact lists
- Op-ed placement coordination
- Higher complexity, requires editorial judgment
- Effective for major campaigns after building momentum

## 2. What Works for Diaspora Advocacy

### Key Mechanisms

Research on Cold War-era diaspora mobilization and contemporary movements identifies several effective mechanisms:

1. **Publicizing regime abuses** from safer positions abroad
2. **Assisting activists on the ground** with resources and visibility
3. **Pressuring host-country governments** to intervene
4. **Pursuing international justice** mechanisms
5. **Creating special-purpose human rights NGOs** that gain credibility for international engagement

### Target Categories for Diaspora Action

**Host country officials (US, EU, Canada, etc.)**
- Most accessible and responsive to constituent pressure
- Relevant for: sanctions policy, visa policies, asylum support, foreign policy positions
- Templates can follow standard domestic advocacy patterns

**International bodies**
- UN Human Rights Council Special Rapporteur on Iran (Mai Sato as of 2025)
- UN Independent Fact-Finding Mission on Iran (FFMI) — mandate significantly expanded in April 2025 to cover "all recent and ongoing serious human rights violations" (not just 2022 protests)
- FFMI has interviewed 300+ victims/witnesses, preserved 38,000+ evidence items as of October 2025
- These mechanisms have formal "Calls for Input" — submissions can be templated
- Submission portal: https://spsubmission.ohchr.org/ (no ECOSOC status required)
- Can also email indaba@ohchr.org if unclear which mandate holder to contact

**EU mechanisms**
- EU Global Human Rights Sanctions Regime (Magnitsky-style, adopted December 2020)
- Currently 216 individuals and 37 entities under Iran human rights sanctions
- Covers: genocide, crimes against humanity, torture, slavery, extrajudicial killings, enforced disappearances, arbitrary detention
- Advocacy targets: European External Action Service, national foreign ministries
- EU requires unanimity among Member States to add names — proposals come from Member States or High Representative
- Can advocate for additions to sanctions lists with documented evidence
- **Resource:** Justice for Iran and Safeguard Defenders have published a Persian-language guide on petitioning for Magnitsky designations (https://safeguarddefenders.com)

**Home country officials**
- Generally not responsive to diaspora pressure for authoritarian regimes
- Exception: some governments have consular services that can be pressured (e.g., passport renewals)
- More relevant for transitional justice advocacy (preparing for future accountability)

### Diaspora-Specific Challenges

**Transnational repression**
- Surveillance, targeting of family members, passport confiscation
- Iran actively surveils diaspora activists
- Safety considerations must inform action design (see Section 6)

**Internal diaspora disagreements**
- Significant debate on broad vs. targeted sanctions
- Some argue broad sanctions harm ordinary Iranians (medication shortages documented)
- Template content must navigate these sensitivities or allow community choice

## 3. Template Design Considerations

### Personalization vs. Uniformity

**Research findings:**
- Personalized messages are substantially more effective than form letters
- 91% of congressional staffers want information about local impact; only 9% receive it frequently
- 79% find personal stories helpful; only 18% receive them frequently
- Politicians are less responsive when they perceive mass-mailing list treatment

**Practical approach for v0:**
- Provide templates with required fill-in sections (personal story, local connection)
- Require at least one personalization before sending
- Display "personalization score" to encourage customization
- Store personalized versions to avoid repeat work

### Avoiding Spam Filters

Modern email deliverability requirements (2025-2026):

1. **Authentication required**: SPF, DKIM, DMARC must all pass
2. **Provider enforcement (all major providers now enforce for 5,000+ daily sends):**
   - **Microsoft (Outlook/Hotmail/Live)**: As of May 5, 2025, non-compliant messages go to junk; future rejection planned
   - **Google/Yahoo**: DMARC, SPF, DKIM required; one-click unsubscribe mandatory
3. **Spam complaint thresholds**: Google/Yahoo require <0.1%, Microsoft <0.3%
4. **Technical requirements**: Valid forward/reverse DNS, RFC 5321/5322 compliance
5. **Unsubscribe**: Must process within 2 days, both header and in-message options required

**Recommendation for v0:**
- Don't send bulk emails from platform domain
- Instead: Generate draft, user sends from their own email client (or `mailto:` link)
- This sidesteps deliverability issues entirely
- Matches v0 philosophy of explicit consent execution
- No sender reputation to manage, no spam complaint risk

### Multi-Language Templates

For Iran pilot:
- Primary languages: Persian (Farsi), English
- Secondary: Turkish (for Iran-Turkey diaspora), German, French (major EU diaspora communities)
- Templates should have parallel versions, not just translations
- Cultural context differs between diaspora and inside-Iran participants

### How Existing Tools Template Messages

**Resistbot approach:**
- Users can paste bill number (e.g., H.R. 123) or news article link
- AI Writer generates customized draft from 1-2 sentence position statement
- "Coauthor" keyword triggers AI letter generation
- Simple keywords for targeting: "all" (President + Congress + Governors), "federal", "state", "congress", "senate", "house", etc.
- Delivery handled across formats (web forms, email, fax, postal)
- Open Letters feature: users can make letters public for visibility
- Scale: 10+ million letters delivered, consistent interface via SMS/messaging apps
- Petitions: organizers create campaigns, users join with simple keywords

**VoterVoice approach:**
- Pre-written templates with editable sections
- Audience segmentation by topic interest — surveys to understand supporter interests
- Multiple action types offered (email, phone, social) as "tiered engagement"
- Tracking of deliverability and engagement
- Challenge: 44% of advocacy professionals struggle with driving advocates to take action amid inbox overload

**Action Network approach:**
- "Ladders" — automated email/mobile message sequences that guide supporters through engagement journeys
- Decision trees that send different messages based on supporter actions and location
- Customizable wait times to feel organic and personal
- Machine learning-based scoring to predict engagement (Boost feature)
- Ready-made templates for common flows (welcome new supporters, reengage lapsed ones)

## 4. Platform-Specific Constraints

### Email Deliverability

**For bulk sends from platform:**
- Requires SPF, DKIM, DMARC setup
- Need to warm up sender reputation gradually
- Risk of being flagged as spam

**Recommendation:** In v0, avoid this entirely. Draft messages that users copy/send themselves.

### X (Twitter) Rules

**Prohibited (Coordinated Inauthentic Behavior):**
- Operating multiple accounts that interact with same/similar content to inflate prominence
- Unauthorized automation (scripting website without API)
- Purchasing engagement or using engagement pods
- Hashtag hijacking and reply spam
- Fake personas using stock/stolen/AI-generated profile photos, copied bios
- Impersonation (falsely posing as individuals, groups, or organizations)

**Allowed:**
- Organic sharing of content by real users
- API-based posting from single accounts with proper authorization
- Coordinated *authentic* activity (real people independently choosing to share)
- Parody/commentary/fan accounts if clearly labeled in name and bio

**Key distinction:** Platform rules target *inauthentic* coordination. Real people independently choosing to share similar content is allowed. The platform looks for patterns of manufactured identities and coordinated manipulation.

**Recommendation for v0:**
- Don't automate posting
- Provide shareable content users can copy
- Don't orchestrate timing or create fake amplification
- Never create accounts on behalf of users or suggest sock-puppet tactics

### Instagram (Meta) Rules

**Prohibited:**
- Account networks that coordinate to violate policies
- Accounts created to evade previous removals
- Patterns indicating coordinated inauthentic behavior

**Enforcement:**
- AI detection of violating content
- Network-based enforcement (linked accounts restricted together)
- Proportional to violation severity and history

**Recommendation:** Same as X — provide content for organic sharing, avoid orchestrated campaigns.

## 5. Who Defines Templates

### Community-Contributed vs. Centrally Defined

**Tradeoffs:**

| Approach | Pros | Cons |
|----------|------|------|
| Central | Quality control, consistent voice, legally reviewed | Bottleneck, may miss community priorities |
| Community | Scales, reflects authentic voices, diverse tactics | Risk of low quality, malicious content, liability |
| Hybrid | Best of both | Complexity |

### Recommendation for v0: Hybrid with Central Review

1. **Core templates centrally defined** for common action types (email to Congress, UN submission, etc.)
2. **Community can propose templates** for specific campaigns
3. **Review process before publication:**
   - Check for factual accuracy
   - Check for spam/malicious content
   - Check for safety risks (especially for Iran)
   - Legal review for defamation/liability

### Version Control and Review Process

Drawing from Santa Clara Principles and co-design research:

1. **Template proposals** submitted with rationale and target
2. **Community comment period** (24-72 hours for urgent, longer for standard)
3. **Review by designated moderators** (could be rotating community role)
4. **Publication with version history**
5. **Ability to flag/report problematic templates**
6. **Transparent appeals process** if template rejected

### Preventing Malicious Templates

Risks:
- Content that could endanger inside-Iran participants
- Defamatory content creating legal liability
- Content designed to discredit the movement
- Spam/phishing content

Mitigations:
- Mandatory review before first publication
- Rate limiting on new template authors
- Reputation system for template contributors
- Automated scanning for obvious problems (links to known malicious sites, etc.)
- Clear liability disclaimers that users are responsible for what they send

## 6. Iran-Specific Considerations

### Relevant Action Types

**High relevance:**
1. **International advocacy** — contacting US Congress, EU Parliament, UN mechanisms
2. **Targeted sanctions advocacy** — advocating for individuals/entities to be added to EU/US sanctions lists
3. **Documentation of abuses** — contributing to IHRDC, Iran Digital Archive Coalition (2M+ artifacts preserved)
4. **Media attention campaigns** — diaspora media outreach, op-ed coordination
5. **Asylum/refugee support** — policy advocacy in host countries

**Medium relevance:**
- FOIA requests (US government dealings)
- Public comments on Iran-related regulations

**Lower relevance for this platform:**
- Direct contact to Iranian officials (ineffective, potentially dangerous)
- Inside-Iran civic engagement (different safety profile)

### Key Targets

**US:**
- Members of Congress (especially Foreign Affairs, Human Rights committees)
- State Department
- Treasury (OFAC for sanctions)

**EU:**
- European External Action Service
- National foreign ministries (Germany, France, UK particularly active)
- European Parliament Human Rights Subcommittee

**UN:**
- Special Rapporteur on Iran — Mai Sato (formal submission process: spsubmission.ohchr.org)
- Independent Fact-Finding Mission on Iran (FFMI) — expanded mandate April 2025
- Human Rights Council (through member state delegations)
- OHCHR Calls for Input — periodic deadlines for thematic reports

**Diaspora media:**
- BBC Persian, Iran International, Voice of America Persian
- Op-ed placement in major Western outlets

### Effective Coalition Tactics

The April 2025 expansion of the FFMI mandate demonstrates effective diaspora advocacy:

1. **Coalition building**: 43 international human rights organizations jointly urged UN Member States to support the renewal/expansion
2. **Coordinated messaging**: Unified public statements timed to HRC sessions
3. **Documentation emphasis**: Organizations stressed evidence preservation (38,000+ items) for future accountability
4. **Sustained engagement**: Consistent presence at HRC sessions, regular reporting cycles

**Key lesson:** Individual action is valuable, but coordinated coalition action achieves institutional outcomes.

### Safety Considerations for Inside-Iran Participants

**Critical risks — Iran's Transnational Repression:**
- Iran conducts comprehensive transnational repression across 9+ countries (Europe, Middle East, North America)
- Toolkit includes: assassinations, renditions, detentions, digital intimidation, spyware, mobility controls
- Diaspora activists targeted with Pegasus and similar spyware for complete device access
- Ethnic minorities particularly targeted: 17.5% of reported cyber-repression cases target Baluchi, Kurdish, Turkic activists

**Specific cyber tactics (per 2024 Miaan Group Threat Intelligence Report):**
- Account breaches and phishing campaigns
- Malware deployment
- Identity fraud
- "Quishing" (QR code phishing) — newer technique
- Online harassment campaigns and content manipulation
- False reporting to get activists' accounts suspended

**During detention:**
- Devices confiscated and set to Airplane Mode to prevent remote wipe
- Coerced disclosure of passwords and contacts
- Legal penalties under Computer Crimes Law include fines, imprisonment, potentially death

**Psychological impact:**
- Relentless online harassment
- Financial and emotional exhaustion
- Pressure on relatives inside Iran as leverage
- Knowledge that activities are monitored in real-time

**Platform design implications:**

1. **Minimize data collection** — don't require real names or location from inside-Iran users
2. **No action traceability** — actions should not create records linkable to specific inside-Iran individuals
3. **Separate participation modes:**
   - Diaspora mode: can take public-facing actions (letters with name)
   - Anonymous mode: contribute to voting/prioritization but don't execute traceable actions
4. **VPN/censorship guidance** — provide resources (United4Iran's Safe Activism guide)
5. **No inside-Iran action execution** — v0 should only execute actions targeting outside officials, from diaspora participants
6. **Warn about metadata** — email headers, account associations can leak identity
7. **Consider device security guidance** — warn about Quishing, phishing, and account security

**Recommendation:** For v0, inside-Iran users should be able to:
- Submit concerns (anonymized in evidence store)
- Vote on agenda items
- View action results

But should NOT be encouraged to:
- Send emails to officials (creates traceable record)
- Participate in social media campaigns (creates traceable record)
- Use their real identity anywhere in the system
- Click links in messages that could be phishing attempts

## 7. Practical Recommendations for v0

### Action Types to Include

**Phase 1 (launch):**
1. Email to US Congress members (template + send-yourself flow)
2. Email to EU officials (template + send-yourself flow)
3. UN Special Rapporteur submission (during call-for-input periods)

**Phase 2 (after initial feedback):**
4. Social media shareable content (no automation, just copy-paste)
5. Letter templates for physical mail
6. Media pitch templates (for diaspora journalists)

### Template Architecture

```
ActionTemplate {
  id: string
  version: number
  type: "email" | "letter" | "social" | "submission" | "media_pitch"
  
  // Targeting
  target_type: "us_congress" | "eu_official" | "un_mechanism" | "media"
  target_selection: "by_location" | "specific_list" | "user_choice"
  
  // Content
  languages: ["en", "fa", ...]
  subject_template: string  // with {{variables}}
  body_template: string
  required_personalizations: string[]  // fields user must fill
  optional_personalizations: string[]
  
  // Metadata
  author: string
  created_at: timestamp
  reviewed_by: string
  review_status: "draft" | "pending" | "approved" | "rejected"
  
  // Safety
  safe_for_inside_iran: boolean  // almost always false for v0
  requires_real_identity: boolean
}
```

### Execution Flow

```
1. Agenda item wins vote
2. System selects appropriate template(s) based on item topic/target
3. User sees draft with required personalizations highlighted
4. User fills in personalizations
5. User reviews complete message
6. User copies message to their email client (or system provides mailto: link)
7. User confirms they sent it
8. System records participation (anonymized for inside-Iran)
```

### Key v0 Constraints

| Constraint | Rationale |
|------------|-----------|
| No bulk sending from platform | Deliverability, spam risk, liability |
| No social media automation | Platform policy compliance |
| No inside-Iran action execution | Safety |
| Required personalization | Effectiveness research |
| Human review of templates | Quality, safety, liability |
| Send-yourself model | User consent, no impersonation |

## 8. Open Questions

1. **How to verify action completion?** User says "I sent it" but did they? Does it matter for v0?

2. **Multi-action campaigns?** Should a single winning agenda item trigger multiple action types simultaneously?

3. **Timing coordination?** Is there value in coordinated timing ("everyone email on Tuesday") without crossing into inauthentic behavior?

4. **Sanctions list advocacy specifics?** What evidence standards are needed to responsibly advocate for adding individuals to sanctions lists?

5. **Template localization workflow?** Who translates templates? How to ensure Persian version matches English intent?

6. **Inside-Iran participation value?** If they can only vote (not act), is their participation still valuable enough to justify the complexity/risk?

7. **Action effectiveness measurement?** How do you know if actions had impact? Response tracking? Policy outcome correlation?

## 9. Resources

### Existing Platforms to Study
- Resistbot (https://resist.bot/) — SMS-based civic action, AI letter generation
- VoterVoice — Enterprise advocacy platform, 585M+ engagements analyzed
- Action Network (https://actionnetwork.org/) — Progressive organizing, automation ladders
- Countable — Bill tracking and action

### Iran-Specific Organizations
- Iran Human Rights Documentation Center (IHRDC)
- Iran Digital Archive Coalition (Atlantic Council) — 2M+ artifacts preserved
- United4Iran Safe Activism Project
- Center for Human Rights in Iran (https://iranhumanrights.org/)
- Impact Iran (https://impactiran.org/) — UN advocacy coordination
- Justice for Iran — Magnitsky sanctions advocacy

### Research
- Congressional Management Foundation "Building Trust by Modernizing Constituent Engagement" report
- VoterVoice 2025 Advocacy Benchmark Report (FiscalNote)
- Miaan Group 2024 Iran Cyber Threat Intelligence Report (https://miaan.org/)
- UK Home Office Country Policy Note on Iran social media surveillance (April 2025)
- Freedom House Iran Transnational Repression Case Study
- Safeguard Defenders Persian guide to Magnitsky sanctions petitioning

### UN Mechanisms
- OHCHR Special Procedures submission portal: https://spsubmission.ohchr.org/
- General submissions: indaba@ohchr.org
- Special Rapporteur on Iran calls for input: https://www.ohchr.org/en/special-procedures/sr-iran
- FFMI information: https://www.ohchr.org/en/hr-bodies/hrc/ffmi-iran/index

### Email Deliverability Standards (2025)
- Microsoft bulk sender requirements: https://techcommunity.microsoft.com/t5/microsoft-defender-for-office-365-blog
- Google Postmaster Tools: https://postmaster.google.com/

## 10. v0 Implementation Summary

### Recommended Action Types for Launch

| Action Type | Target | Complexity | Inside-Iran Safe? |
|-------------|--------|------------|-------------------|
| Email to US Congress | Members of Congress | Low | No |
| Email to EU officials | EEAS, foreign ministries | Low | No |
| UN Special Rapporteur submission | OHCHR portal | Medium | Possibly (anonymized) |
| Social media shareable | Public | Low | No |

### Execution Model: "Draft and Copy"

For v0, avoid sending anything on behalf of users. Instead:

```
1. User selects action from winning agenda item
2. System generates personalized draft (with required fill-ins)
3. User reviews and customizes
4. System provides:
   - Copy-to-clipboard button
   - mailto: link (for email actions)
   - Social share buttons (for social)
5. User confirms completion (optional, not verified)
6. System logs participation (anonymized for inside-Iran)
```

### Template Personalization Requirements

Based on Congressional Management Foundation research:

| Required Field | Why |
|----------------|-----|
| Local impact statement | 91% of staff want this; only 9% receive frequently |
| Personal connection/story | 79% find helpful; only 18% receive frequently |
| Constituent identification | Address, relation to district |

Minimum viable template:
```
Subject: [Issue] - Constituent from [District/City]

Dear [Official],

[One sentence stating position]

[Required: How this affects me/my community locally]

[Optional: Personal story]

[Call to action: specific ask]

Sincerely,
[Name]
[City, State/Country]
```

### Safety Defaults for Iran Pilot

| Setting | Default | Rationale |
|---------|---------|-----------|
| Inside-Iran action execution | Disabled | Transnational repression risk |
| Real name required | No | Safety |
| Location tracking | No | Safety |
| Action completion verification | Optional self-report | No coercion |
| Data retention | Minimal | Reduce attack surface |

### Open Questions for v0 Decision

1. **Verification of completion**: Accept self-reporting? Or skip entirely?
2. **Multi-language flow**: Persian primary with English option? Or parallel?
3. **Sanctions advocacy**: Include or defer? Requires evidence standards
4. **Coalition integration**: Connect to existing orgs (Impact Iran, CHRI) or independent?
5. **UN submission timing**: Only during official call-for-input periods, or anytime?

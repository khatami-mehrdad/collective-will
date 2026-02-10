# Legal and Regulatory Research: Civic Action Platform

**Research Date:** February 2026  
**Context:** A system that collects political opinions, clusters them, facilitates voting, and helps users take collective action (emails to officials, social media campaigns). Pilot targets Iranian users (inside Iran and diaspora in US, EU, Canada).

---

## Table of Contents
1. [Sending Messages on Behalf of Users](#1-sending-messages-on-behalf-of-users)
2. [Data Privacy](#2-data-privacy)
3. [Platform Terms of Service](#3-platform-terms-of-service)
4. [Political Activity Regulations](#4-political-activity-regulations)
5. [Safety and Liability](#5-safety-and-liability)
6. [Iran-Specific Considerations](#6-iran-specific-considerations)
7. [Practical Recommendations](#7-practical-recommendations)

---

## 1. Sending Messages on Behalf of Users

### CAN-SPAM Act Requirements (US)

The CAN-SPAM Act applies to all commercial messages. Key requirements:

| Requirement | Details |
|-------------|---------|
| **Header accuracy** | Truthful "From," "To," and routing information |
| **Subject lines** | Non-deceptive, accurately reflecting content |
| **Ad identification** | Clear identification as advertisement (unless prior affirmative consent) |
| **Physical address** | Valid postal address required |
| **Opt-out mechanism** | Clear, conspicuous opt-out instructions; honor within 10 business days |
| **Penalties** | Up to $53,088 per violation |

**Important:** Political/advocacy messages may have different treatment than commercial messages, but best practice is to comply with CAN-SPAM requirements regardless.

### GDPR Email Marketing Rules (EU)

For EU users, additional requirements under GDPR and ePrivacy Directive:

- **Consent required** for electronic marketing communications
- **Legitimate interest** may apply for existing relationships, but political content is higher risk
- **Explicit consent** strongly recommended for political communications (special category data)

### When Does Collective Action Become "Spam"?

Key distinctions:

| Legitimate Advocacy | Potential Spam Issues |
|---------------------|----------------------|
| Users individually compose/customize messages | Automated identical messages without user involvement |
| Clear user intent and action | Messages sent without meaningful user choice |
| One message per user per official | High-volume repeated messages from same users |
| Authentic constituent communication | Artificial amplification or misrepresentation |

### Email Authentication Requirements (SPF, DKIM, DMARC)

**Critical for deliverability.** As of 2025, bulk senders (5,000+ emails/day) must implement:

1. **SPF (Sender Policy Framework)** - Authorize sending IPs
2. **DKIM (DomainKeys Identified Mail)** - Cryptographic signature verification
3. **DMARC** - At minimum `p=none` policy; `p=quarantine` or `p=reject` recommended

Microsoft Outlook and Gmail will route non-compliant messages to spam or reject them entirely.

### How Resistbot Handles This

**Legal Structure:** Resistbot operates as the Resistbot Action Fund, a Delaware-registered 501(c)(4) non-profit.

**Approach:**
- Users compose their own messages via text/web interface
- First-time users verify identity via email
- Multiple delivery methods: email, fax, postal mail
- Emphasizes quality over quantity (discourages excessive messaging)
- Users are clearly the sender; Resistbot is a facilitation tool
- 40+ million letters delivered since 2017

**Key Takeaway:** Resistbot positions itself as a communication tool that users actively operate, not a service sending messages "on behalf of" users. The user is the sender.

### Action Items for v0

- [ ] **Implement full email authentication** (SPF, DKIM, DMARC) before launch
- [ ] **Design user flow** where users clearly compose/approve each message
- [ ] **Include physical address** in all outgoing communications
- [ ] **Build opt-out mechanism** that honors requests within 10 days
- [ ] **Rate limit** to prevent abuse (e.g., max 1 message per user per official per week)
- [ ] **Identity verification** for first-time users

---

## 2. Data Privacy

### GDPR: Political Opinions as Special Category Data

**Critical:** Under GDPR Article 9, political opinions are **special category data** requiring enhanced protections.

**Processing Requirements:**
1. Must have a lawful basis under Article 6 (e.g., consent, legitimate interest)
2. Must **also** satisfy a condition under Article 9 for special category data
3. Most viable conditions:
   - **Explicit consent** (Article 9(2)(a)) - Strongest approach
   - **Data made manifestly public by data subject** (Article 9(2)(e))
   - **Processing by non-profit body** relating to members/regular contacts (Article 9(2)(d))

**Cannot target individuals with political messaging using inferred political opinions without explicit consent.**

### CCPA Requirements (California)

As of 2025:
- Applies to businesses with $26.625M+ revenue, data on 100k+ consumers, or 50%+ revenue from data sales
- Right to know, delete, and opt-out of sale
- **No explicit "special category" designation** for political data like GDPR
- Starting 2026: Enhanced requirements for minors, automated decision-making

**Note:** CCPA does not have GDPR's special category restrictions on political data, but general privacy principles apply.

### Right to Erasure vs. Append-Only Audit Logs

**The Tension:** GDPR requires data deletion on request, but audit logs may be legally required for security/compliance.

**Resolution Strategy:**

| Approach | Description |
|----------|-------------|
| **Pseudonymization** | Replace PII with irreversible tokens in audit logs |
| **Tiered retention** | Short retention for full data, longer for pseudonymized |
| **Suppression registers** | Hash-based suppression lists to prevent re-creation |
| **Document everything** | Demonstrate compliance efforts |

**Technical Implementation:**
1. Store user-facing data separately from audit logs
2. Use irreversible hashes/tokens in audit logs instead of PII
3. Implement automated deletion for live data
4. Backups can remain until overwritten but must be "beyond use"

### Data Minimization Principles

- Collect only what's necessary for stated purposes
- Don't store more detailed data than needed
- Aggregate where possible (cluster opinions without retaining individual raw text)
- Regular review and purging of unnecessary data

### Cross-Border Data Transfers

**EU-US:** The EU-US Data Privacy Framework (July 2023 adequacy decision) allows transfers to certified US organizations. First periodic review completed October 2024 with positive findings.

**Hosting Considerations:**

| Location | Pros | Cons |
|----------|------|------|
| **EU (e.g., Netherlands, Germany)** | GDPR compliant by default, no transfer issues for EU users | May complicate US nonprofit structure |
| **US** | Simpler for US nonprofit, Data Privacy Framework certified | Must certify under DPF, additional safeguards for transfers |
| **Multi-region** | Best of both, data residency options | Complexity, cost |

### Action Items for v0

- [ ] **Obtain explicit consent** for processing political opinions (GDPR Article 9(2)(a))
- [ ] **Design consent flow** clearly explaining what data is collected and why
- [ ] **Implement right to erasure** with pseudonymization for audit trails
- [ ] **Data minimization** - aggregate/anonymize where possible
- [ ] **Privacy policy** clearly explaining special category data handling
- [ ] **Data processing agreement** template for any third-party processors
- [ ] **Consider EU hosting** to simplify GDPR compliance

---

## 3. Platform Terms of Service

### Twitter/X Automation Rules

**Permitted:**
- Broadcasting helpful information
- Auto-replying to user engagement
- Responding to DMs

**Prohibited:**
- Abusing API or circumventing rate limits
- Non-API automation (scripting the website)
- Spam or unsolicited messages
- Deriving sensitive information (political affiliation) without opt-in consent

**Key Risk:** Coordinated posting by many users on the same topic could be flagged as coordinated inauthentic behavior, even if organic.

### Meta (Facebook, Instagram) Policies

**Coordinated Inauthentic Behavior (CIB)** is prohibited:
- Using fake accounts/pages to deceive about identity or origin
- Evading enforcement
- Misusing reporting systems

**Foreign Interference:** CIB where operators are outside the target country.

**Key Distinction:** Meta distinguishes between:
- Authentic coordinated activity (allowed) - Real people organizing around shared interests
- Inauthentic coordinated behavior (prohibited) - Fake accounts, deception about identity

**Advocacy organizations navigate this by:**
- Using authentic accounts representing real people
- Transparent organizational identity
- Not misrepresenting the origin of campaigns
- Following platform rules for political advertising

### When Does Collective Action Violate ToS?

| Likely Compliant | Likely Violation |
|------------------|------------------|
| Users share platform-generated content from their own accounts | Automated posting from fake accounts |
| Transparent campaign attribution | Concealing coordination or sponsorship |
| Users choose what/when to post | Platform posts automatically without user action |
| Authentic engagement | Artificial amplification, fake engagement |

### Action Items for v0

- [ ] **Design for authenticity** - Users manually share, not auto-post
- [ ] **Transparent attribution** - Clear that content comes from your platform
- [ ] **No fake accounts** - All activity from real, verified users
- [ ] **Comply with API terms** if integrating with social platforms
- [ ] **Monitor platform policy updates** - These change frequently

---

## 4. Political Activity Regulations

### US Lobbying Disclosure Requirements

**Lobbying Disclosure Act (LDA):**
- Registration required when lobbyists make lobbying contacts with federal officials
- Applies to paid lobbyists making direct contacts
- Most grassroots advocacy (constituent communications) does not trigger LDA

**IRS Rules for Nonprofits:**
- 501(c)(3): Lobbying cannot be "substantial part" of activities; 501(h) election provides clear spending limits
- 501(c)(4): Unlimited lobbying permitted

### EU Transparency Register

Organizations conducting interest representation activities in the EU should register:
- Meetings, consultations, communications campaigns
- Policy papers, social media influencer engagement

New 2025 rules address third-country lobbying with stricter requirements.

### FARA (Foreign Agent Registration Act)

**When registration is required:**
- Acting "at the order, request, or under direction/control" of a foreign principal
- Foreign principals include: foreign governments, political parties, individuals, entities
- Activities intended to influence US government or public on US policy

**Key risks for diaspora advocacy:**
- Coordination with foreign opposition groups could trigger registration
- Soliciting contributions on behalf of foreign principals
- Activities advancing foreign government/party interests

**Important Distinction:**
- Advocacy FOR regime change IN Iran by Iranian diaspora: Generally NOT FARA-triggering (advocating for US policy, not on behalf of foreign government)
- Advocacy ON BEHALF OF Iranian government/political parties: Would trigger FARA

**2025 Update:** DOJ is revising regulations; expect stricter interpretation of exemptions.

### Campaign Finance Implications

If the platform supports:
- Advocacy for/against specific candidates: FEC rules apply
- Issue advocacy (without candidate endorsement): Generally unrestricted for 501(c)(4)s

### 501(c)(3) vs 501(c)(4) Comparison

| Aspect | 501(c)(3) | 501(c)(4) |
|--------|-----------|-----------|
| **Tax-deductible donations** | Yes | No |
| **Lobbying** | Limited ("not substantial") | Unlimited |
| **Political campaign activity** | Prohibited | Permitted (not primary purpose) |
| **Voter registration** | Nonpartisan only | May be partisan |
| **Candidate endorsements** | Prohibited | Permitted |

**Recommendation:** 501(c)(4) provides more flexibility for political advocacy but donations are not tax-deductible.

### Action Items for v0

- [ ] **Legal review** of activities for FARA implications
- [ ] **Clear firewall** - Platform facilitates user expression, doesn't advocate on behalf of foreign principals
- [ ] **Avoid coordination** with entities that could be "foreign principals"
- [ ] **Choose appropriate nonprofit structure** (likely 501(c)(4) for flexibility)
- [ ] **Consider EU registration** if engaging EU institutions

---

## 5. Safety and Liability

### Section 230 Protections (US)

Section 230 of the Communications Decency Act provides:
- Platforms are not treated as "publisher or speaker" of user content
- Protection from civil liability for user-generated content
- Protection for good-faith content moderation

**What Section 230 covers:**
- Defamation claims based on user posts
- Most civil liability for user content

**What Section 230 does NOT cover:**
- Federal criminal law
- Intellectual property claims
- Communications privacy law
- Sex trafficking (FOSTA-SESTA carve-out)

### EU Digital Services Act (DSA)

**Effective February 2024** for all platforms serving EU users.

**Obligations by size:**

| Platform Size | Key Obligations |
|---------------|-----------------|
| **All intermediaries** | Notice and action for illegal content, transparency reporting |
| **Hosting services** | Content moderation, user notification of decisions |
| **Online platforms** | Trusted flaggers, complaint mechanisms, advertising transparency |
| **VLOPs (45M+ EU users)** | Risk assessments, annual audits, researcher access |

**For a small civic platform:**
- Implement notice-and-action procedures for illegal content
- Provide clear terms of service
- Transparency reporting on content decisions
- Micro/small company exemptions may apply initially

### Harassment and Illegal Activity

**If the platform is used for harassment:**
1. Section 230 generally protects against liability for user content
2. BUT: Must have and enforce terms of service
3. Must respond to valid legal process
4. DSA requires action on illegal content notices

**Recommended protections:**
- Clear terms of service prohibiting harassment, illegal activity
- Reporting mechanisms for abuse
- Moderation capacity to remove violating content
- Cooperation procedures for law enforcement
- User blocking/muting capabilities

### Terms of Service Best Practices

Based on similar platforms (Change.org, Resistbot, Speak4):

1. **User conduct rules** prohibiting harassment, illegal activity
2. **Content disclaimers** - Platform not responsible for user content
3. **Liability limitations** - Disclaim liability for third-party actions
4. **Arbitration clauses** - Consider for dispute resolution
5. **Account accuracy** - Require truthful registration
6. **Termination rights** - Right to remove violating users

### Action Items for v0

- [ ] **Draft Terms of Service** with clear conduct rules, liability disclaimers
- [ ] **Implement reporting mechanism** for abuse/harassment
- [ ] **Build moderation capacity** (at least manual review process)
- [ ] **DSA compliance** - Notice-and-action procedure for illegal content
- [ ] **Document content policies** transparently
- [ ] **Legal review** of terms and privacy policy before launch

---

## 6. Iran-Specific Considerations

### OFAC Sanctions and General License D-2

**Key Finding:** US entities CAN provide communications software to Iranians under General License D-2.

**General License D-2 (September 2022, codified May 2024) authorizes:**
- Instant messaging, chat, email services
- Social media and networking platforms
- Video conferencing
- Cloud-based services
- Collaboration platforms
- Anti-surveillance software development

**Important changes from earlier policy:**
- Removed "personal" communications requirement
- Software need only be "incident to" authorized services (not "necessary")
- Expanded support for anti-surveillance tools

**What's covered:**
- Providing the platform/software to Iranian users: **Authorized**
- Fee-based or no-cost services: **Both authorized**
- Cloud-based services: **Authorized**

**What may need separate review:**
- Any services to designated persons (e.g., IRGC, sanctioned entities)
- Hardware exports (different requirements)
- Financial services beyond what's incident to communications

### How Similar Projects Handle This

**GAMAAN (The Group for Analyzing and Measuring Attitudes in Iran):**
- **Structure:** Non-profit research foundation registered in Netherlands
- **Approach:** Online surveys distributed through multiple channels
- **Innovation:** Partners with VPN providers to reach respondents in Iran securely
- **Recognition:** Won MRS President's Medal (2022); director represents Iran at WAPOR

**Key lessons from GAMAAN:**
1. Netherlands registration provides institutional legitimacy
2. Digital-first approach enables reaching users in Iran
3. Partnership with VPN providers enhances security/access
4. Academic/research framing may provide additional protections

### Security Considerations for Iranian Users

Beyond legal compliance, must consider:
- **Anonymity protections** for users inside Iran
- **Secure communications** (E2E encryption where possible)
- **No data that could identify users to authorities** if compromised
- **VPN-friendly design** (users in Iran often use VPNs)
- **Minimal data collection** to reduce risk if breached

### Action Items for v0

- [ ] **Document GL D-2 compliance** - Ensure activities fall within authorization
- [ ] **OFAC counsel review** - Confirm specific activities are covered
- [ ] **Do not collect** information that could endanger users in Iran
- [ ] **Security-first design** - Assume adversarial environment
- [ ] **Consider Netherlands/EU entity** for European operations
- [ ] **No services to designated persons** - Screen for OFAC-listed entities

---

## 7. Practical Recommendations

### Recommended Legal Structure

**Option A: US 501(c)(4) (Recommended for v0)**

| Pros | Cons |
|------|------|
| Unlimited lobbying | Donations not tax-deductible |
| Can engage in political activity | US jurisdiction/discovery |
| Established advocacy framework | FARA scrutiny potential |
| GL D-2 clearly applies | |

**Option B: Netherlands Foundation (Stichting)**

| Pros | Cons |
|------|------|
| GDPR-compliant by default | Less familiar structure |
| European credibility (like GAMAAN) | Banking/operational complexity |
| Distance from US political dynamics | May need US entity for US operations |

**Option C: Dual Structure**

- US 501(c)(4) for advocacy activities
- EU foundation for research/data processing
- More complex but maximizes protections

**Recommendation:** Start with US 501(c)(4), consider EU entity for data processing/privacy compliance later.

### Required Disclaimers and Consent Flows

**Registration/Onboarding:**
1. Privacy policy acknowledgment
2. Terms of service acceptance
3. **Explicit consent for political data processing** (GDPR Article 9)
4. Explanation of how data will be used
5. Identity verification (email at minimum)

**Before each action:**
1. Clear preview of message content
2. Confirmation that user wants to send
3. Attribution statement (message comes from user, facilitated by platform)

**Ongoing:**
1. Easy opt-out/unsubscribe from platform communications
2. Data deletion request mechanism
3. Account deletion capability

### Legal Review Checklist Before Launch

| Area | Review Needed | Priority |
|------|---------------|----------|
| **OFAC compliance** | Confirm GL D-2 coverage | Critical |
| **Terms of Service** | Attorney review | Critical |
| **Privacy Policy** | GDPR/CCPA compliance review | Critical |
| **FARA analysis** | Confirm no registration required | High |
| **Email compliance** | CAN-SPAM, authentication setup | High |
| **Nonprofit formation** | 501(c)(4) application | High |
| **DSA compliance** | EU obligations review | Medium |
| **Platform ToS** | Review if social media integration | Medium |

### v0 Launch Checklist

**Legal Foundation:**
- [ ] Form 501(c)(4) nonprofit (Delaware recommended)
- [ ] OFAC counsel sign-off on GL D-2 coverage
- [ ] FARA analysis confirming no registration required
- [ ] Terms of Service drafted and reviewed
- [ ] Privacy Policy drafted and reviewed (GDPR-compliant)

**Technical Compliance:**
- [ ] Email authentication (SPF, DKIM, DMARC)
- [ ] Explicit consent flow for political data
- [ ] Right to erasure implementation
- [ ] Data minimization review
- [ ] Secure communications (TLS, consider E2E)

**Operational:**
- [ ] Content moderation procedures
- [ ] Abuse reporting mechanism
- [ ] Opt-out/unsubscribe system
- [ ] Identity verification for users
- [ ] Rate limiting to prevent abuse

**Documentation:**
- [ ] GL D-2 compliance documentation
- [ ] Data processing records (GDPR Article 30)
- [ ] Consent records
- [ ] Moderation decision logs

### Ongoing Compliance

- **Quarterly:** Review platform ToS changes
- **Annually:** GDPR/CCPA compliance audit
- **Annually:** OFAC sanctions list screening
- **As needed:** Monitor FARA regulatory changes
- **As needed:** Update policies for new features

---

## Summary of Key Findings

1. **You can serve Iranian users** under OFAC General License D-2 - this is the most critical finding for your use case.

2. **Political opinions are special category data under GDPR** - explicit consent is required.

3. **The Resistbot model works** - user composes message, platform facilitates delivery, user is the sender.

4. **501(c)(4) is likely the right structure** - allows unlimited lobbying and political activity.

5. **FARA is a risk to monitor** - ensure platform facilitates user expression, doesn't advocate on behalf of foreign principals.

6. **Section 230 provides protection** but requires good terms of service and content moderation.

7. **Email authentication is mandatory** - SPF, DKIM, DMARC required for deliverability.

8. **Netherlands entity (like GAMAAN) is an option** for enhanced GDPR compliance and European credibility.

---

## References

- OFAC Iran General License D-2 (31 CFR § 560.540)
- GDPR Article 9 (Special Categories of Personal Data)
- CAN-SPAM Act (15 U.S.C. § 7701 et seq.)
- Section 230 (47 U.S.C. § 230)
- EU Digital Services Act (Regulation 2022/2065)
- Foreign Agents Registration Act (22 U.S.C. § 611 et seq.)
- Lobbying Disclosure Act (2 U.S.C. § 1601 et seq.)
- EU-US Data Privacy Framework (EC Adequacy Decision, July 2023)

# Research: Iran as Pilot Community

Research on Iran as the target community for Collective Will—understanding what Iranians want, who they trust, how they communicate, and what a civic platform must consider.

**Last Updated:** February 2026

---

## Executive Summary

Iran represents both an urgent need and significant challenges for civic technology. With **70-89% of the population opposing the current regime**, **89% supporting democracy**, and **81% using VPNs** to access blocked platforms, there's strong demand for channels to express democratic preferences. However, severe surveillance, regime repression, and complex trust dynamics create substantial risks and design considerations.

---

## 1. Recent Political Issues and Protests (2024-2026)

### The 2025-2026 Protests (Current)

A new wave of massive protests erupted in **late December 2025**, initially triggered by economic collapse:

- Iranian rial collapsed to **1.4 million per dollar**
- Inflation exceeded **52%** (IMF projects 42.4% for 2025, remaining above 40% in 2026)
- **22-50% living below poverty line**; 57% face malnourishment
- 50% of males aged 25-40 unemployed

What began in Tehran's bazaar evolved into a mass movement against the theocratic government. By January 2026, **thousands reported dead** from government crackdowns, with the majority under age 30.

### Woman, Life, Freedom Movement (2022-Present)

- **Triggered**: September 2022 death of Mahsa (Zhina) Amini, arrested for "improper hijab"
- **Scope**: Nationwide protests in 26 of 31 provinces
- **Deaths**: At least 551 (including 49 women, 68 children) per UN investigation
- **Languages**: Slogans in Kurdish, Persian, Azeri, and Balochi—linguistic diversity matters

### Key Grievances

1. **Economic**: Sanctions impact, inflation, unemployment, poverty
2. **Political**: Authoritarian religious rule, election manipulation, corruption
3. **Social**: Mandatory hijab enforcement, women's rights suppression
4. **Ethnic**: Marginalization of Kurds, Baloch, Arabs, Azeris
5. **Human rights**: Arbitrary detention, torture, executions of protesters

---

## 2. Messaging App Usage and Circumvention

### Platform Status in Iran

| Platform | Status | Usage |
|----------|--------|-------|
| **Telegram** | Banned since 2018 | 65%+ have accounts despite ban; accessed via VPN |
| **WhatsApp** | Restrictions "lifted" December 2024 | Primary organizing tool during 2022 protests |
| **Signal** | Repeatedly blocked | Used by security-conscious activists; proxy workarounds |
| **Instagram** | Filtered | Major organizing platform during 2022 protests |
| **Facebook/TikTok/X/YouTube** | Blocked | Accessed via VPN |

### VPN Usage

- **81%** of Iranian internet users use VPNs
- 49.4% use free VPNs; 30.3% use paid services
- Only **2.4%** are "very satisfied" with internet quality
- **79%** are "not very satisfied" or "not satisfied at all"

**Key insight**: Government officials—including Supreme Leader Khamenei's office—continue using Telegram despite banning it for citizens, fueling public anger about hypocrisy.

---

## 3. The Trust Landscape

### Who Do Iranians Trust?

**Media Trust (2023 GAMAAN survey, 38,445 respondents)**

| Source | Following | Trust Level |
|--------|-----------|-------------|
| Iran International | 54% | 50% trust |
| Manoto TV | 42% | 44% trust |
| BBC Persian | 37% | 34% trust |
| State broadcaster IRIB | 36% | **21% trust** |

**Critical finding**: **59% express "no trust at all"** in state media.

### Institutional Trust Crisis

- Iran faces **widespread trust deficit** across all institutions
- Support for Supreme Leader and Islamic revolution: dropped from **18% (2022) to 11% (2024)**
- The regime has "sown seeds of distrust among Iranians," causing social atomization

### Political Preferences (June 2024 GAMAAN, 77,216 respondents)

- **70% oppose** continuation of Islamic Republic
- **89% support democracy**
- **66% oppose** religious law-based governance
- **40%** view regime change as precondition for reform
- Only **5%** support parties prioritizing traditional/religious values

### Preferred Alternatives

- **26%** prefer secular republic
- **21%** prefer constitutional monarchy
- **11%** say form matters less than achieving change
- **43%** remain open to authoritarian rule by a "strong leader" (higher in rural areas)

---

## 4. Existing Civic Tech Attempts

### GAMAAN (Netherlands)

- Independent research foundation conducting attitude surveys
- Innovative methodology for authoritarian contexts addressing "preference falsification"
- Large-scale surveys: 58,000-77,000 respondents from inside Iran
- **Model worth studying** for methodology

### Fordem (Germany)

- Censorship-resistant digital democracy platform
- Features: participation, deliberation, networking, survey tools
- Open-source, GDPR compliant
- Built specifically for Woman, Life, Freedom movement
- Currently work-in-progress

### United4Iran

- Iran Prison Atlas database
- **Gershad app**: crowdsourced mobile app helping citizens avoid morality police
- Safe activism guides

### 2022-2023 Protest Coordination Patterns

**Tools used**:
- Instagram and WhatsApp as primary organizing tools
- Telegram channels for coordination and news
- Signal, Tor for secure communication

**Government counter-tactics**:
- "Snitch lines" on Telegram
- "Mobile curfews" disabling networks 4pm-1am
- Targeted VPN blocking
- Device confiscation during arrests

---

## 5. What Would "Hearing the Voice of the Nation" Look Like?

### Key Unanswered Questions

1. **Transition preferences**: Iranians agree more on *opposing* the regime than *what comes next*
2. **Leadership vacuum**: No consensus on trusted leadership figures for transition
3. **Reform vs. revolution**: 40% insist on regime change as precondition
4. **Regional priorities**: Ethnic minorities may have different priorities than Persian majority
5. **Diaspora vs. domestic alignment**: Tensions exist about strategy (especially sanctions)

### Questions Worth Exploring

- What specific policy changes would improve daily life?
- How should ethnic minority rights be protected in a future system?
- What role should religion play in governance?
- How should economic reconstruction be prioritized?
- What justice/accountability mechanisms for past abuses?
- Attitudes toward regional foreign policy?

### Demographic Patterns

| Group | Opposition to Islamic Republic |
|-------|-------------------------------|
| University graduates | 74%+ |
| Non-university educated | 66% |
| Urban residents | ~75% |
| Rural residents | ~72% (28% support regime) |
| Youth | Highest opposition |

---

## 6. Risks and Safety Considerations

### Surveillance and Repression

**SIAM System Capabilities** (documented by The Intercept/Citizen Lab):
- Real-time location tracking
- Communication metadata monitoring
- SIM card deactivation as punishment
- "Not a surveillance system but a repression and control system"

**Cyber Tactics (2024 Miaan Group Report)**:
- Account breaches, phishing, malware
- "Quishing" (phishing via QR codes)
- 17.5% of attacks target ethnic minority activists
- Device confiscation and forced password disclosure

**Consequences**:
- Women human rights defenders face fines, lengthy prison sentences, death penalty
- Social media users posting critical content face harassment, arrest, torture
- Near-total internet shutdowns during crises

### Transnational Repression

Iran conducts transnational repression across 9+ countries:
- Pegasus spyware deployment
- Assassinations and renditions
- Family intimidation inside Iran
- Passport confiscations

**Diaspora journalists under attack**: BBC Persian journalists in UK face threats, family interrogations.

### Diaspora vs. Inside-Iran Dynamics

- Diaspora debates about sanctions have created internal divisions
- Those inside Iran may resent diaspora activists who face no physical risk
- Different access to information and different lived experiences

---

## 7. Design Implications for Collective Will

### Platform Design

1. **Security-first**: End-to-end encryption, no metadata storage, proxy support
2. **Pseudonymization**: Allow meaningful participation without real identity exposure
3. **Multi-language**: At minimum Farsi + Azerbaijani Turkish + Kurdish
4. **Low-bandwidth friendly**: Many users have poor connections
5. **Works with existing tools**: Integration with Telegram/Signal/WhatsApp

### Trust Building

1. **Partner with trusted diaspora media** (Iran International has highest trust at 50%)
2. **Transparency about methodology** (GAMAAN's approach is well-regarded)
3. **No ties to any political faction** (neither reformist nor monarchist nor MEK)
4. **Community oversight** for question design and result interpretation
5. **Open-source code** for verifiability

### Safety Measures

1. **Never collect identifying information**
2. **Allow participation via VPN/Tor**
3. **Distribute through trusted VPN providers** (like Psiphon, which GAMAAN uses)
4. **No phone number requirements** (SMS is surveillance vector)
5. **Clear guidance** on safe participation practices

### Participation Model

**Inside-Iran users should be able to:**
- Submit concerns (anonymized in evidence store)
- Vote on agenda items
- View action results

**Inside-Iran users should NOT:**
- Execute traceable actions (emails, social media)
- Use their real identity anywhere
- Be exposed to metadata that could identify them

### What Success Looks Like

- **Legitimacy**: Results broadly match independent polling (GAMAAN)
- **Representativeness**: Demographics roughly match Iran's population
- **Action**: Results inform diaspora advocacy, international policy
- **Safety**: No participant harm attributable to the platform

---

## 8. Language Requirements

| Language | % Population | Priority |
|----------|--------------|----------|
| Farsi (Persian) | ~60% | Primary |
| Azerbaijani Turkish | ~25% | High |
| Kurdish (Sorani, Kurmanji) | ~7% | High |
| Balochi | ~2% | Medium |
| Arabic | ~2% | Medium |
| Luri, Gilaki, Mazandarani | ~5% | Lower |

The Woman, Life, Freedom slogan being chanted in Kurdish, Persian, Azeri, and Balochi demonstrates that linguistic diversity matters for legitimacy.

---

## 9. Key Sources

- **GAMAAN** (gamaan.org): Gold standard for Iran polling
- **Freedom House** Freedom on the Net reports
- **Iran International**: Most trusted diaspora news source
- **Miaan Group**: Cyber threat intelligence and surveillance documentation
- **United4Iran**: Civic tech and activism tools
- **The Intercept / Citizen Lab**: SIAM surveillance system documentation
- **UN FFMI**: Independent Fact-Finding Mission on Iran reports

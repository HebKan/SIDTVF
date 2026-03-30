# SIDTVF Severity Classification Reference

> **Document Version:** 1.0 | **Status:** Active | **Last Updated:** 2026-03-30

---

## Overview

Every SIDTVF scenario carries a single severity rating: **Critical**, **High**, **Medium**, or **Low**. This rating reflects the combination of two independent assessments:

- **Impact** — how much harm results if the scenario plays out to completion
- **Likelihood** — how probable it is that this scenario occurs in a typical enterprise environment

Severity is not a property of the actor alone, the data type alone, or the detection difficulty alone. It is a structured judgment that accounts for all of these factors together. This document defines exactly how to make that judgment.

Use this document when:
- Authoring a new scenario card and selecting the `Severity` field
- Reviewing or challenging an existing severity assignment
- Explaining severity ratings to non-technical stakeholders
- Calibrating triage card escalation criteria

Full template reference: [scenarios/_template.md](../scenarios/_template.md)

---

## Part 1 — Impact Assessment

Impact is assessed across five dimensions. Rate each dimension independently, then use the highest dimension score as your impact anchor.

### Impact Dimensions

#### Dimension 1: Data Impact

What is the sensitivity and scale of data that could be exposed, stolen, or destroyed?

| Rating | Criteria |
|--------|----------|
| **Catastrophic** | >10,000 records of PII, PHI, or MNPI; complete exfiltration of trade secrets, core IP, or proprietary algorithms; full database or data warehouse contents; data that triggers mandatory regulatory breach notification |
| **Severe** | 1,000–10,000 records of sensitive data; significant confidential business data (financial forecasts, M&A plans, key customer contracts); partial but material IP exfiltration |
| **Moderate** | <1,000 records of sensitive data; internal-only confidential data with limited external value; one department's data with no regulated data types |
| **Minor** | Non-sensitive internal data only; publicly available or low-value information; no regulated data types (PII/PHI/MNPI/IP) involved |

#### Dimension 2: Financial Impact

What is the direct or reasonably foreseeable monetary consequence?

| Rating | Criteria |
|--------|----------|
| **Catastrophic** | Direct monetary loss or regulatory fine exposure exceeding $1M; MNPI misuse capable of triggering SEC enforcement; ransomware-class financial disruption; market manipulation enabling material gain |
| **Severe** | $100K–$1M in direct loss, legal costs, or regulatory fine exposure; material breach notification and remediation costs; significant contract penalties |
| **Moderate** | $10K–$100K; limited remediation costs; minor regulatory fine exposure; no material financial loss to the organization |
| **Minor** | Under $10K; cleanup and investigation costs only; no regulatory fine exposure; no direct financial loss |

#### Dimension 3: Operational Impact

What is the disruption to systems, services, or business processes?

| Rating | Criteria |
|--------|----------|
| **Catastrophic** | Critical production system(s) taken offline or permanently destroyed; business continuity threatened; multi-day recovery required; broad customer or partner-facing service disruption |
| **Severe** | Significant service degradation affecting a material user population; hours of recovery; a major business process (payroll, order processing, clinical operations) disrupted |
| **Moderate** | Limited service impact; a single non-critical system affected; recovery under one hour; one team or department disrupted |
| **Minor** | No service impact; investigation and cleanup only; no user-facing disruption |

#### Dimension 4: Legal and Regulatory Impact

What compliance obligations or legal consequences are triggered?

| Rating | Criteria |
|--------|----------|
| **Catastrophic** | Mandatory external regulatory reporting triggered (HIPAA breach notification, GDPR Art. 33/34 notification, SEC Form 8-K); potential criminal referral; class action exposure; multi-jurisdiction reporting obligations |
| **Severe** | Internal legal investigation required; potential civil liability; compliance audit or examination likely triggered; material contract breach |
| **Moderate** | Legal review required but no external reporting obligation likely; internal policy violation documented; potential for regulatory scrutiny if pattern continues |
| **Minor** | No regulatory trigger; internal HR action only; no legal counsel involvement required |

#### Dimension 5: Reputational Impact

What is the risk of reputational harm to the organization?

| Rating | Criteria |
|--------|----------|
| **Catastrophic** | Material risk of public disclosure; customer or patient data breach requiring external notification letters; high probability of press coverage or regulatory public censure |
| **Severe** | Customer or partner relationship risk; likely internal disclosure to executive leadership or board; potential for reputational harm if not contained |
| **Moderate** | Team or department-level awareness; no customer or external visibility; limited executive attention required |
| **Minor** | No material reputational risk; event is fully internal and containable |

---

### Deriving the Impact Score

Apply the following rule:

| If your highest dimension rating is... | ...and at least one other dimension is... | Impact Score |
|---------------------------------------|-------------------------------------------|--------------|
| Catastrophic | Any rating | **Catastrophic** |
| Severe | Severe or higher | **Severe** |
| Severe | Moderate or lower | **Severe** |
| Moderate | Moderate or higher | **Moderate** |
| Minor | Any rating | **Minor** |

In practice: **your Impact Score equals the highest single dimension rating.** The multi-dimension check is used to confirm whether to round up a borderline case.

---

## Part 2 — Likelihood Assessment

Likelihood is assessed across four dimensions. Unlike impact (where the highest dimension anchors the score), likelihood is an average judgment across all four dimensions.

### Likelihood Dimensions

#### Dimension 1: Actor Access Requirement

What level of organizational access does an actor need to execute this scenario?

| Rating | Criteria |
|--------|----------|
| **High** | Standard user access — any employee with a laptop and a corporate account can execute this scenario without needing elevated permissions |
| **Medium** | Departmental or functional access — a member of a specific team (finance, IT, HR, DevOps) with their normal access level can execute this |
| **Low** | Elevated or privileged access required — DBA, system administrator, security engineer, or someone with special approvals needed |
| **Very Low** | Highly restricted access — root-level, security-team-only, or access that requires multi-party approval to obtain |

#### Dimension 2: Technical Sophistication Required

What skill level does the actor need to successfully execute the scenario?

| Rating | Criteria |
|--------|----------|
| **High** | None — uses only standard business tools (web browser, email client, USB drive, SaaS application); no technical knowledge beyond normal job requirements |
| **Medium** | Low — basic technical knowledge sufficient (PowerShell or command-line familiarity, basic scripting, understanding of file transfers); available in general IT-literate population |
| **Low** | Medium — requires understanding of security controls, logging behavior, or system internals; most employees cannot do this without research |
| **Very Low** | High — requires advanced knowledge: detection evasion techniques, custom tool development, exploitation of vulnerabilities, or deep understanding of the target system |

#### Dimension 3: Detection Evasion Effort

How much deliberate effort is required to avoid triggering existing controls?

| Rating | Criteria |
|--------|----------|
| **High** | None required — behavior either blends with normal activity, is not currently monitored, or is structurally invisible to common controls (e.g., AI tool input, screen capture) |
| **Medium** | Minimal — basic behavioral adjustments sufficient (working after hours, breaking a large action into smaller ones, using a personal account) |
| **Low** | Moderate — requires staging data, using encrypted channels, choosing less-monitored paths, or exploiting monitoring gaps deliberately |
| **Very Low** | Significant — requires active counter-detection measures: disabling logs, living-off-the-land technique selection, deliberate log tampering, or operational security planning |

#### Dimension 4: Observed Frequency

How often does this category of behavior occur in real enterprise environments, based on industry reporting?

| Rating | Criteria |
|--------|----------|
| **High** | Very common — multiple incidents per year in mid-to-large enterprises; extensively documented in CERT, Verizon DBIR, or industry breach reports; well-understood by security teams |
| **Medium** | Common — observed regularly across the industry; specialist knowledge required to execute but threat actors are known to use it; documented in threat intelligence |
| **Low** | Occasional — occurs but is not routine; requires specific opportunity or motivation; fewer documented cases in public reporting |
| **Very Low** | Rare — few documented enterprise cases; requires highly unusual access, motivation, or opportunity; specialist threat actor profile |

---

### Deriving the Likelihood Score

Assess each dimension, then apply this rule:

| Result | Criteria |
|--------|----------|
| **High Likelihood** | Three or more dimensions rated High; OR all four dimensions rated Medium or above |
| **Medium Likelihood** | Two dimensions rated High and the rest Medium; OR three dimensions rated Medium and one rated Low |
| **Low Likelihood** | The majority of dimensions rated Low or Very Low; OR access requirement is Very Low |

---

## Part 3 — Severity Rating Matrix

Combine your Impact Score and Likelihood Score using this matrix:

|  | **High Likelihood** | **Medium Likelihood** | **Low Likelihood** |
|--|---------------------|-----------------------|--------------------|
| **Catastrophic Impact** | **Critical** | **Critical** | **High** |
| **Severe Impact** | **High** | **High** | **Medium** |
| **Moderate Impact** | **High** | **Medium** | **Low** |
| **Minor Impact** | **Medium** | **Low** | **Low** |

### Reading the Matrix

- A catastrophic impact scenario is never below **High** regardless of likelihood — the potential harm is too severe to treat as low priority.
- A moderate impact scenario can be **High** if the likelihood is high enough — common, easily executed scenarios warrant elevated response readiness even if each individual incident causes limited harm.
- A minor impact scenario peaks at **Medium** — even if it occurs frequently, limited harm caps the escalation response required.

---

## Part 4 — Severity Tiers Defined

### Critical

**Definition:** The scenario can result in catastrophic or severe irreversible harm across multiple dimensions, and is either highly likely or structurally difficult to prevent. Critical scenarios require the highest detection priority, pre-approved response playbooks, and direct executive awareness.

**Typical characteristics:**
- Mass regulated data exposure (PII, PHI, MNPI, or IP at scale)
- Regulatory breach notification obligations triggered
- Direct financial loss or enforcement risk exceeding $1M
- Operational destruction that threatens business continuity
- Executable by a standard-access employee using normal business tools

**Example scenarios:**

| Scenario | Why Critical |
|----------|-------------|
| **SB-001** — Targeted Database Deletion | Catastrophic operational impact (production data destroyed), catastrophic data impact (complete data loss), highly motivated malicious insider, detection often comes after damage is done |
| **DE-001** — Cloud Storage Upload Exfiltration | Catastrophic data impact (mass PII/IP at scale), High likelihood (any employee with cloud access), minimal evasion required, standard tools only |
| **FR-001** — Invoice and Vendor Payment Manipulation | Catastrophic financial impact ($1M+ in wire fraud schemes), Medium-High likelihood (any finance team member), exploits approval workflow gaps, directly enriches actor |
| **IP-001** — Source Code and Proprietary Algorithm Theft | Catastrophic data impact (core IP), legal exposure under DTSA, High actor motivation at offboarding, difficult to recover from once disclosed |
| **AC-001** — Insider Account Compromise by External Actor | Catastrophic across all dimensions if privileged account is compromised; external actor uses insider's full access scope; difficult to distinguish from legitimate activity |

---

### High

**Definition:** The scenario can result in severe harm with a realistic probability of occurrence, or catastrophic harm that requires elevated access or technical sophistication (reducing likelihood). High scenarios require active detection coverage, response playbook readiness, and management escalation on confirmation.

**Typical characteristics:**
- Significant data exposure (hundreds to thousands of records, or targeted confidential data)
- Regulatory audit risk or civil liability
- Financial loss in the $100K–$1M range
- Service degradation for significant user populations
- Requires departmental access or basic technical knowledge

**Example scenarios:**

| Scenario | Why High |
|----------|---------|
| **DE-002** — Email Auto-Forwarding Rule | Severe data impact (ongoing exfiltration of all email), Medium-High likelihood (any M365 user), minimal evasion (rule persists silently), often missed until departure |
| **UA-001** — After-Hours Unauthorized System Access | Moderate-Severe data impact, High likelihood (credential theft or sharing is common), can be precursor to higher-severity scenarios |
| **PM-001** — Privilege Escalation via Access Abuse | Severe impact if escalation is to admin-level systems, Medium likelihood, requires insider knowledge of access control gaps |
| **TP-001** — Vendor Credential Abuse | Severe data and operational impact, Medium likelihood (vendors frequently over-provisioned), often undetected due to monitoring gaps for third-party accounts |

---

### Medium

**Definition:** The scenario results in meaningful but contained harm, or has a moderate probability of occurring but with limited potential impact. Medium scenarios require documented detection coverage and response guidance, but do not require the same level of urgency as Critical or High.

**Typical characteristics:**
- Data exposure is limited in scale or sensitivity (<1,000 records, no regulated types)
- Financial impact under $100K
- No immediate regulatory reporting trigger
- Requires legal review but not mandatory external notification
- Often involves negligent rather than malicious actors
- Service impact limited to one team or department

**Example scenarios:**

| Scenario | Why Medium |
|----------|-----------|
| **DE-003** — AI Tool Data Leakage (negligent actor) | Moderate data impact (typically internal/confidential data pasted, not full PII databases), High likelihood (AI tool use is ubiquitous), but detection is structurally difficult — CASB coverage can address |
| **PV-001** — Shadow IT Data Storage | Minor-to-moderate data impact, High likelihood, no malicious intent in most cases; risk is data custody, not deliberate exfiltration |

---

### Low

**Definition:** The scenario results in minor harm that is unlikely to cause lasting organizational damage, involves no regulated data at risk, and typically involves negligent rather than malicious actors. Low scenarios still warrant policy enforcement and documented guidance, but do not typically justify SOC escalation beyond standard alert handling.

**Typical characteristics:**
- Non-sensitive, non-regulated data only
- No financial exposure or regulatory obligation
- No service impact
- Easily reversible or contained
- Primarily a policy violation rather than a security incident

**Example indicators of Low severity:**
- An employee installs unapproved personal software on their work laptop with no data transfer
- An employee accesses a coworker's public shared drive folder without permission, reads a non-sensitive document, and takes no further action
- An employee uses a personal USB drive to transfer their own non-sensitive documents when leaving a role

> **Note:** Low severity scenarios are still worth documenting if they represent a measurable behavior pattern that can be an early indicator of escalating risk. A single USB transfer is Low; a pattern of USB transfers correlated with departure notice is an indicator for a higher-severity scenario.

---

## Part 5 — Scoring Worksheet

Use this worksheet when authoring or reviewing a scenario's severity rating. Record your assessments for each dimension, derive the composite scores, and apply the matrix.

```
SCENARIO ID: __________   SCENARIO NAME: __________________________________

IMPACT ASSESSMENT
─────────────────────────────────────────────────────────────────────────────
Dimension 1 — Data Impact:          [ ] Catastrophic  [ ] Severe  [ ] Moderate  [ ] Minor
  Rationale: _________________________________________________________________

Dimension 2 — Financial Impact:     [ ] Catastrophic  [ ] Severe  [ ] Moderate  [ ] Minor
  Rationale: _________________________________________________________________

Dimension 3 — Operational Impact:   [ ] Catastrophic  [ ] Severe  [ ] Moderate  [ ] Minor
  Rationale: _________________________________________________________________

Dimension 4 — Legal/Regulatory:     [ ] Catastrophic  [ ] Severe  [ ] Moderate  [ ] Minor
  Rationale: _________________________________________________________________

Dimension 5 — Reputational Impact:  [ ] Catastrophic  [ ] Severe  [ ] Moderate  [ ] Minor
  Rationale: _________________________________________________________________

IMPACT SCORE (highest single dimension):  ___________________________________

LIKELIHOOD ASSESSMENT
─────────────────────────────────────────────────────────────────────────────
Dimension 1 — Actor Access Req.:    [ ] High  [ ] Medium  [ ] Low  [ ] Very Low
Dimension 2 — Technical Sophist.:   [ ] High  [ ] Medium  [ ] Low  [ ] Very Low
Dimension 3 — Evasion Effort:       [ ] High  [ ] Medium  [ ] Low  [ ] Very Low
Dimension 4 — Observed Frequency:   [ ] High  [ ] Medium  [ ] Low  [ ] Very Low

LIKELIHOOD SCORE (per rule in Part 2):  ____________________________________

SEVERITY RATING (matrix lookup):  __________________________________________

Assigned by: __________________   Date: _______________   Reviewed by: _______________
```

---

## Part 6 — Common Calibration Mistakes

| Mistake | Why It's Wrong | Correct Approach |
|---------|---------------|-----------------|
| Rating Critical based on data type alone | "PHI is involved, so it's Critical" ignores volume and likelihood | Rate Catastrophic data impact, then check likelihood — a single accidental email with one patient record may be Medium |
| Rating Low because no malicious intent | Negligent actors can cause Critical harm | Severity is about potential harm, not intent; PV-001 (shadow IT) is Medium not Low because the data custody risk is real |
| Rating High for all departing employee scenarios | Not all departures involve access to sensitive data | Apply the dimensions — a departing janitor with standard IT access is lower likelihood than a departing DBA with production credentials |
| Treating likelihood as "how often does insider threat happen" | Likelihood is scenario-specific, not a general program concern | Rate each dimension for this specific scenario: what access, what skill, what evasion, how often in industry |
| Upgrading severity based on actor motivation alone | A highly motivated actor with standard access executing a low-impact behavior is still not Critical | Motivation informs the actor profile field, not severity; a disgruntled employee who leaks a non-sensitive internal memo is Medium at most |
| Downgrading severity because "we have controls" | Severity reflects the harm if the scenario plays out; controls affect likelihood, not harm | If your DLP would stop the exfiltration, that reduces the likelihood dimension — but do not reduce the data impact rating based on the assumption that controls will work |

---

## Changelog

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-03-30 | Initial release — impact/likelihood dimensions, severity matrix, per-tier examples, scoring worksheet, calibration guide |

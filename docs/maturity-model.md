# SIDTVF Program Maturity Model

> **Document Version:** 1.0 | **Status:** Active | **Last Updated:** 2026-03-25

---

## Overview

The SIDTVF Maturity Model provides organizations with a structured way to assess the current state of their insider threat program, identify capability gaps, and build a prioritized roadmap for improvement. It is designed to be usable by organizations of any size, industry, or regulatory context.

The model defines five maturity levels across six capability domains. An organization's overall maturity is the lowest level at which all domain criteria are consistently met — not an average.

**Important:** Maturity levels are not a competitive ranking. A Level 2 program that consistently executes its defined activities is more effective than an aspirational Level 4 program that is inconsistently implemented.

---

## The Five Maturity Levels

### Level 1 — Initial

**Characteristics:**
- No formal insider threat program exists
- Detection of insider threats is accidental — an alert fires because a control happened to be in place, not because it was designed for this purpose
- Response is ad hoc, reactive, and uncoordinated
- No defined scenarios, no documented playbooks
- No metrics are tracked
- Legal and HR are involved after the fact, if at all

**Typical Organization Profile:**
Small to mid-size organizations without a dedicated security team. Security is managed by IT operations. Insider threats are not a documented risk.

**Risk of remaining at Level 1:**
High probability of undetected data exfiltration, prolonged dwell time, and inability to respond effectively when an incident occurs.

---

### Level 2 — Developing

**Characteristics:**
- Basic security monitoring tools are deployed (SIEM, DLP, endpoint protection)
- Some insider-threat-relevant policies exist (acceptable use, data classification) but may not be actively enforced or communicated
- Detection is rule-based and reactive — alerts fire based on static thresholds, not behavioral baselines
- Incident response exists but insider threat is not a distinct response category
- Some scenarios may be informally understood but are not documented
- Limited metrics; typically only counts (number of alerts, number of incidents)

**Typical Organization Profile:**
Mid-size organizations with a small security team. Security tools are deployed for compliance reasons. Insider threat awareness is growing.

**Key Gaps vs. Level 3:**
No documented scenarios, no formal validation of controls, no structured hunting program.

---

### Level 3 — Defined

**Characteristics:**
- Formal insider threat program with documented policies, scope, and governance
- Scenario library exists with documented behaviors, actor archetypes, and MITRE ATT&CK mappings
- Detections are tied to defined scenarios and have been at least minimally validated
- Response playbooks exist for major scenario categories
- Legal and HR are engaged as formal program stakeholders
- Threat hunting occurs on a defined cadence, using documented hypotheses
- Basic metrics are tracked: detection coverage, validation pass rate, time-to-detect

**Typical Organization Profile:**
Large organizations with a dedicated security team or a dedicated insider threat program team. Program exists as a recognized function. Often driven by regulatory requirements (HIPAA, CMMC, financial services regulations).

**Key Gaps vs. Level 4:**
Validation is not systematic; metrics are collected but not used to drive decisions; feedback loops are informal.

---

### Level 4 — Managed

**Characteristics:**
- All active scenarios have validated detections with documented pass/fail results
- Metrics are tracked, reported to leadership, and used to drive program decisions
- Validation cycles run on a defined quarterly schedule
- Feedback loop is formalized — incidents and hunt findings are routinely fed back into the scenario library
- Behavioral analytics and User and Entity Behavior Analytics (UEBA) are deployed and tuned to scenarios
- Maturity assessments are conducted annually
- Third-party risk is formally incorporated into the program
- Legal and privacy impact assessments are conducted for new monitoring capabilities

**Typical Organization Profile:**
Large enterprises and heavily regulated organizations (financial services, defense contractors, healthcare systems). Insider threat is a distinct program function with dedicated staff and budget.

**Key Gaps vs. Level 5:**
Program is reactive to the scenario library; does not yet anticipate novel threats. Limited cross-organizational intelligence sharing.

---

### Level 5 — Optimizing

**Characteristics:**
- Continuous improvement cycle operates without manual prompting — new scenarios are identified automatically from hunt findings, incident patterns, and external threat intelligence
- Predictive indicators are in use — program identifies at-risk individuals before harmful activity occurs, using behavioral, psychological, or contextual signals (with full legal review)
- Cross-organizational threat intelligence sharing (ISACs, government partnerships, peer organizations)
- Red team and purple team exercises are used to validate and stress-test the program on a regular cadence
- Automation handles routine validation and metric collection
- Program contributes scenarios and detections back to the wider community (open source, ISAC sharing)
- Privacy-preserving analytics are in use to minimize PII exposure in monitoring systems

**Typical Organization Profile:**
Intelligence community, defense, critical infrastructure operators, and mature financial institutions. Insider threat is a strategic capability, not just a compliance function.

---

## The Six Capability Domains

The self-assessment questionnaire and maturity levels are organized across six domains. Each domain must reach the target level for the overall program maturity to be assessed at that level.

| Domain | Code | Description |
|--------|------|-------------|
| Governance & Program Structure | GPS | Program ownership, policies, scope, legal framework |
| Scenario Coverage | SC | Documented scenarios, actor archetypes, ATT&CK mapping |
| Detection & Monitoring | DM | Tools, rules, behavioral analytics, data sources |
| Validation & Testing | VT | Test cases, validation cycles, gap analysis |
| Hunting & Proactive Discovery | HPD | Hypothesis development, hunt execution, finding documentation |
| Feedback & Continuous Improvement | FCI | Metrics, feedback loops, program evolution |

---

## Self-Assessment Questionnaire

### Instructions

Score each question from 0 to 4:
- **0** — Not in place. No evidence of this capability.
- **1** — Ad hoc. Attempted or partially in place but not consistent or documented.
- **2** — Developing. Documented and partially deployed but not consistently practiced.
- **3** — Defined. Documented, deployed, and consistently practiced.
- **4** — Managed/Optimizing. Measured, reviewed regularly, and continuously improved.

Your **domain score** is the average of your responses within that domain.
Your **overall maturity level** is determined by the domain score table below.

---

### Domain 1 — Governance & Program Structure (GPS)

| # | Question | Score (0–4) |
|---|----------|-------------|
| GPS-01 | Does a formal insider threat policy exist that defines scope, prohibited behaviors, and enforcement? | |
| GPS-02 | Is program ownership formally assigned (a named role or team)? | |
| GPS-03 | Are Legal and HR formally engaged as program stakeholders? | |
| GPS-04 | Has a legal review been conducted to ensure monitoring activities comply with applicable law? | |
| GPS-05 | Is the program scope documented (which users, systems, and data types are in scope)? | |
| GPS-06 | Are employees notified of monitoring activities as required by applicable law and policy? | |
| GPS-07 | Is the program reviewed at least annually for policy and scope alignment? | |
| GPS-08 | Are third-party vendors and contractors explicitly addressed in program scope? | |

**Domain Score:** _____ / 4

---

### Domain 2 — Scenario Coverage (SC)

| # | Question | Score (0–4) |
|---|----------|-------------|
| SC-01 | Does a documented scenario library exist with defined behaviors, not just threat categories? | |
| SC-02 | Are scenarios tagged to specific actor archetypes (MAL, NEG, COM, TP)? | |
| SC-03 | Are scenarios mapped to MITRE ATT&CK techniques? | |
| SC-04 | Does the scenario library cover all four actor archetypes? | |
| SC-05 | Are scenarios reviewed and updated at least annually? | |
| SC-06 | Are scenario severity and priority assigned using a documented method? | |
| SC-07 | Are new scenarios created when incidents or hunts reveal uncatalogued behaviors? | |
| SC-08 | Do scenarios include legal and privacy considerations specific to the behavior? | |

**Domain Score:** _____ / 4

---

### Domain 3 — Detection & Monitoring (DM)

| # | Question | Score (0–4) |
|---|----------|-------------|
| DM-01 | Is a SIEM or equivalent log aggregation platform in place and collecting relevant insider-threat data sources? | |
| DM-02 | Is DLP (Data Loss Prevention) deployed and tuned to insider threat scenarios? | |
| DM-03 | Is endpoint monitoring in place (EDR/UEM) for managed devices? | |
| DM-04 | Is User and Entity Behavior Analytics (UEBA) deployed and tuned? | |
| DM-05 | Are detection rules explicitly tied to documented scenarios? | |
| DM-06 | Is detection coverage tracked as a percentage of active scenarios? | |
| DM-07 | Are privileged accounts subject to enhanced monitoring? | |
| DM-08 | Is cloud storage and SaaS application activity monitored for exfiltration indicators? | |

**Domain Score:** _____ / 4

---

### Domain 4 — Validation & Testing (VT)

| # | Question | Score (0–4) |
|---|----------|-------------|
| VT-01 | Are documented test cases used to validate that detections fire against expected behaviors? | |
| VT-02 | Is validation conducted on a defined schedule (not just at initial deployment)? | |
| VT-03 | Are validation results documented with pass/fail classification? | |
| VT-04 | Are control gaps identified through validation tracked to remediation? | |
| VT-05 | Is a false positive rate tracked and managed? | |
| VT-06 | Are detection rules re-validated after significant configuration or tool changes? | |
| VT-07 | Are validation exercises authorized and documented with appropriate approvals? | |
| VT-08 | Are third-party detections (vendor-provided rules) validated before deployment? | |

**Domain Score:** _____ / 4

---

### Domain 5 — Hunting & Proactive Discovery (HPD)

| # | Question | Score (0–4) |
|---|----------|-------------|
| HPD-01 | Does a formal threat hunting function exist (dedicated resource or cadenced activity)? | |
| HPD-02 | Are hunt hypotheses documented and linked to defined scenarios? | |
| HPD-03 | Are hunt findings documented and fed back into the scenario library? | |
| HPD-04 | Are hunts conducted on a defined cadence (quarterly at minimum)? | |
| HPD-05 | Is hypothesis-driven hunting practiced (vs. only ad hoc or tool-directed hunting)? | |
| HPD-06 | Do hunts cover all major scenario categories in the library? | |
| HPD-07 | Are hunting queries reusable and documented for future use? | |
| HPD-08 | Are hunt findings integrated into the incident response workflow? | |

**Domain Score:** _____ / 4

---

### Domain 6 — Feedback & Continuous Improvement (FCI)

| # | Question | Score (0–4) |
|---|----------|-------------|
| FCI-01 | Are program KPIs defined and tracked? | |
| FCI-02 | Are metrics reported to security leadership at least quarterly? | |
| FCI-03 | Are incident outcomes formally fed back into the scenario library? | |
| FCI-04 | Are detection failures analyzed for root cause (not just counted)? | |
| FCI-05 | Is the maturity model assessment conducted at least annually? | |
| FCI-06 | Does the program have a documented improvement roadmap? | |
| FCI-07 | Are lessons from peer organizations or industry intelligence incorporated? | |
| FCI-08 | Are program changes impact-assessed for legal and privacy implications before implementation? | |

**Domain Score:** _____ / 4

---

## Scoring and Maturity Determination

### Domain Score to Maturity Level

| Average Domain Score | Domain Maturity Level |
|---------------------|-----------------------|
| 0.0 – 0.9 | Level 1 — Initial |
| 1.0 – 1.9 | Level 2 — Developing |
| 2.0 – 2.9 | Level 3 — Defined |
| 3.0 – 3.7 | Level 4 — Managed |
| 3.8 – 4.0 | Level 5 — Optimizing |

### Overall Maturity Level

Your overall maturity level is the **lowest individual domain maturity level**. A program with five domains at Level 4 and one domain at Level 2 is an overall Level 2 program, because the Level 2 domain represents a systemic gap.

This design is intentional: insider threat programs fail through their weakest domain, not their strongest.

**Example Assessment:**

| Domain | Score | Level |
|--------|-------|-------|
| GPS — Governance | 3.1 | 4 |
| SC — Scenario Coverage | 2.4 | 3 |
| DM — Detection | 3.0 | 4 |
| VT — Validation | 1.5 | 2 |
| HPD — Hunting | 2.1 | 3 |
| FCI — Feedback | 2.8 | 3 |

**Overall Maturity: Level 2** (limited by Validation domain)

---

## Roadmap by Maturity Level

### Level 1 → Level 2: Foundation

**Priority Actions:**

1. **Establish governance.** Assign program ownership to a named role. Draft an insider threat policy (even a one-page one). Engage HR and Legal at least once to define the monitoring boundaries.
2. **Inventory your data sources.** Identify what logs you already have: authentication, email, endpoint, cloud. You cannot detect what you cannot observe.
3. **Deploy basic monitoring.** If you do not have a SIEM, deploy one. If you have one, ensure authentication logs, endpoint logs, and email logs are being ingested.
4. **Define three starter scenarios.** Start with DE-001 (cloud storage exfiltration), DE-002 (email forwarding), and PM-001 (access abuse). These are high-value, well-logged scenarios.
5. **Document your acceptable use policy.** Users need to know what is and is not permitted before they can be held accountable for policy violations.

**Timeline:** 60–90 days

---

### Level 2 → Level 3: Structure

**Priority Actions:**

1. **Build a scenario library.** Document at least 10 scenarios using `scenarios/_template.md`. Cover all four actor archetypes. Map to MITRE ATT&CK.
2. **Tie detections to scenarios.** For each scenario, identify which alert (if any) would fire. If none, document the gap.
3. **Write playbooks.** Create response playbooks for your top five scenarios. Use `playbooks/_template.md`. Include Legal and HR decision gates.
4. **Formalize legal review.** Have Legal review your monitoring activities, data retention practices, and employee notification language.
5. **Establish a hunting cadence.** Commit to one threat hunt per quarter, using documented hypotheses from `hunting/_template.md`.

**Timeline:** 90–180 days

---

### Level 3 → Level 4: Measurement

**Priority Actions:**

1. **Implement structured validation.** For every active scenario, create a test case and run it. Document results. Fix gaps.
2. **Establish a metrics dashboard.** Track the eight KPIs defined in [metrics/kpis.md](../metrics/kpis.md). Report quarterly to leadership.
3. **Deploy UEBA.** Move from threshold-based detection to behavioral baseline detection for high-risk user populations.
4. **Formalize feedback loops.** After every incident and every hunt cycle, update the scenario library. Make this a standing agenda item.
5. **Incorporate third-party risk.** Audit third-party accounts. Remove stale access. Scope vendor monitoring explicitly.
6. **Conduct annual maturity assessment.** Use this document. Present results to leadership.

**Timeline:** 6–12 months

---

### Level 4 → Level 5: Optimization

**Priority Actions:**

1. **Automate validation.** Build or buy capability to run validation test cases on a schedule without manual execution.
2. **Develop predictive indicators.** In collaboration with Legal and HR, define and implement leading indicators of insider risk (with appropriate privacy protections).
3. **Engage in intelligence sharing.** Join relevant ISACs, government threat sharing programs, or peer networks. Contribute scenarios and detections.
4. **Run adversarial exercises.** Commission red team or purple team exercises specifically scoped to insider threat scenarios. Use findings to update the program.
5. **Implement privacy-preserving analytics.** Minimize raw PII in monitoring systems. Use tokenization, aggregation, and access controls to protect individual privacy while maintaining detection capability.

**Timeline:** 12–24 months

---

## Annual Maturity Review Process

1. **Schedule the assessment.** Block time annually (recommend aligning with fiscal year or audit cycle).
2. **Complete the questionnaire.** Have the program owner and at least one independent reviewer complete the questionnaire separately. Compare results.
3. **Calculate domain and overall scores.**
4. **Identify the priority gap domain.** This is the domain holding back your overall maturity level.
5. **Update the improvement roadmap.** Set specific, time-bound goals for the next 12 months.
6. **Report to leadership.** Present current maturity, gap domain, and roadmap. Get commitment for resources.
7. **Archive the assessment.** Store completed assessment results in version control for longitudinal comparison.

---

## References

- Carnegie Mellon University CERT — Insider Threat Program Manager's Guide
- NIST SP 800-53 Rev 5 — Program Management (PM) and Personnel Security (PS) control families
- CISA Insider Threat Mitigation Guide
- SANS Threat Hunting Maturity Model

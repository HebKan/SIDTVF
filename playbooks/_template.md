# Response Playbook Template

> Copy this file and rename it `PB-[SCENARIO-ID]-[short-slug].md`
> Place it in `playbooks/examples/` for completed playbooks.
> Remove all instructional comments (lines beginning with `>`) before publishing.

---

## Playbook Card

| Field | Value |
|-------|-------|
| **Playbook ID** | PB-[SCENARIO-ID] |
| **Name** | [Short descriptive name] |
| **Version** | 1.0 |
| **Status** | Draft / Active / Deprecated |
| **Created** | [YYYY-MM-DD] |
| **Last Updated** | [YYYY-MM-DD] |
| **Author** | [Name or team] |
| **Linked Scenario** | [SCENARIO-ID] — [Scenario Name] |
| **Trigger** | [What triggers this playbook — e.g., "Alert from DET-DE-001-SIGMA fires" or "SOC escalates after-hours access anomaly"] |
| **Primary Owner** | [Team — e.g., SOC, Incident Response, Legal] |
| **Stakeholders** | [Teams involved — e.g., SOC, IR, Legal, HR, Finance, Executive] |

---

## 1. Overview

> One to three sentences describing the purpose of this playbook, what threat it responds to, and what a successful response looks like.

[Overview here]

---

## 2. Prerequisites

> What must be in place for this playbook to be executable?

- [ ] [Prerequisite 1 — e.g., "Access to web proxy logs via SIEM"]
- [ ] [Prerequisite 2 — e.g., "Legal counsel available for on-call escalation"]
- [ ] [Prerequisite 3 — e.g., "HR point of contact identified and briefed on insider threat procedures"]

---

## 3. Severity Classification

> Provide criteria for classifying the severity of an incident matching this playbook on arrival.

| Severity | Criteria | Response SLA |
|----------|----------|-------------|
| Critical | [e.g., "Data volume > 1 GB or confirmed exfiltration of PII/PHI/IP"] | [e.g., 30 minutes to IR lead engagement] |
| High | [e.g., "Data volume 100 MB–1 GB or sensitive data category involved"] | [e.g., 2 hours] |
| Medium | [e.g., "Data volume < 100 MB, data classification unknown"] | [e.g., 4 hours] |
| Low | [e.g., "Policy violation only, no confirmed sensitive data"] | [e.g., Next business day] |

---

## 4. Response Steps

> Each step must identify the responsible team in brackets. Steps are numbered and sequential unless marked [PARALLEL].
> Decision gates (steps requiring a go/no-go decision before proceeding) are marked [DECISION GATE].

### Phase 1 — Triage and Verification

**Goal:** Determine whether this is a true positive and classify the initial severity.

**[DECISION GATE]** If the activity is confirmed legitimate, close the alert as false positive and document the justification. If unconfirmed or suspicious, proceed to Phase 2.

| Step | Action | Owner | Notes |
|------|--------|-------|-------|
| 1.1 | [First triage action] | [SOC] | |
| 1.2 | [Second triage action] | [SOC] | |
| 1.3 | Classify initial severity using the table in Section 3 | [SOC] | |

---

### Phase 2 — Containment

**Goal:** Stop or limit the ongoing threat activity while preserving evidence.

**[DECISION GATE]** Legal must advise on account action timing. Do not disable the account or take containment action before Legal approval if the investigation may result in criminal referral.

| Step | Action | Owner | Notes |
|------|--------|-------|-------|
| 2.1 | [Containment action 1] | [IR] | |
| 2.2 | [Containment action 2] | [IR / Legal] | |

---

### Phase 3 — Evidence Preservation

**Goal:** Secure all relevant evidence before it can be lost, overwritten, or tampered with.

| Step | Action | Owner | Notes |
|------|--------|-------|-------|
| 3.1 | [Evidence action 1] | [IR / Forensics] | |
| 3.2 | [Evidence action 2] | [IR] | |

---

### Phase 4 — Investigation

**Goal:** Establish the full scope of the incident — what was accessed, what was taken, when, and by whom.

| Step | Action | Owner | Notes |
|------|--------|-------|-------|
| 4.1 | [Investigation action 1] | [IR] | |
| 4.2 | [Investigation action 2] | [IR / Legal] | |

---

### Phase 5 — Escalation and Notification

**Goal:** Notify appropriate internal and external stakeholders based on severity and scope.

**[DECISION GATE]** Legal determines breach notification obligations. Do not notify external parties (regulators, customers, press) without Legal approval.

| Stakeholder | Notification Trigger | Owner | Channel |
|-------------|---------------------|-------|---------|
| [Internal team] | [When to notify] | [Owner] | [Email / Phone / Secure channel] |
| [Regulator] | [When required] | [Legal] | [As required by regulation] |

---

### Phase 6 — Recovery and Remediation

**Goal:** Return affected systems to normal operation and close the access pathway that enabled the incident.

| Step | Action | Owner | Notes |
|------|--------|-------|-------|
| 6.1 | [Remediation action] | [IT / IR] | |

---

### Phase 7 — Post-Incident

**Goal:** Document, learn, and improve.

| Step | Action | Owner | Notes |
|------|--------|-------|-------|
| 7.1 | Complete incident report | [IR Lead] | |
| 7.2 | Conduct post-incident review with all stakeholders | [IR Lead + all stakeholders] | |
| 7.3 | Update scenario card and detection rules based on findings | [Detection Engineering] | |
| 7.4 | Feed findings back into SIDTVF scenario library | [Scenario Author] | |

---

## 5. False Positive Considerations

> Document how to confirm this is a false positive and close the alert cleanly.

| Scenario | How to Confirm False Positive |
|----------|------------------------------|
| [FP scenario] | [What to check] |

---

## 6. Key Contacts

> Fill in during deployment — do not leave blank in production.

| Role | Name | Contact |
|------|------|---------|
| IR Lead | [Name] | [Phone / Slack] |
| Legal Counsel (On-Call) | [Name] | [Phone] |
| HR Business Partner | [Name] | [Phone / Email] |
| CISO | [Name] | [Phone] |
| Forensics Lead | [Name] | [Contact] |

---

## 7. Linked Artifacts

| Artifact | ID | File |
|----------|----|------|
| Scenario | [SCENARIO-ID] | [scenarios/.../SCENARIO-ID-slug.md] |
| Validation Test Case | TC-[SCENARIO-ID] | [validation/test-cases/TC-[SCENARIO-ID].md] |
| Detection | DET-[SCENARIO-ID] | [detections/...] |
| Hunt Hypothesis | HYP-[SCENARIO-ID] | [hunting/hypotheses/HYP-[SCENARIO-ID].md] |

---

## Changelog

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | [YYYY-MM-DD] | [Author] | Initial creation |

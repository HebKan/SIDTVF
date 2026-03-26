# Scenario Card: FR-001 — Invoice and Vendor Payment Manipulation

| Field | Value |
|-------|-------|
| **Scenario ID** | FR-001 |
| **Name** | Invoice and Vendor Payment Manipulation |
| **Version** | 1.0 |
| **Status** | Active |
| **Created** | 2026-03-26 |
| **Last Updated** | 2026-03-26 |
| **Author** | SIDTVF Core Team |
| **Category** | Fraud & Financial Crime |
| **Category Code** | FR |
| **Actor Archetype(s)** | MAL |
| **Severity** | Critical |
| **Minimum Maturity Level** | 3 |

---

## 1. Description

An insider with access to accounts payable or financial approval systems manipulates vendor banking details, invoice amounts, or payment approvals to redirect organizational funds to accounts they control or to accounts belonging to a co-conspirator. The insider exploits their legitimate access to financial systems and their knowledge of approval workflows to create fraudulent transactions that bypass normal controls.

---

## 2. Narrative Case Study

Carla was a senior accounts payable specialist at a regional logistics company. Her role gave her write access to the vendor master file — the database of authorized vendors, their banking details, and payment routing information — as well as the ability to create and approve invoices below $25,000 without a second approver.

Over an eight-month period, Carla made subtle changes to the banking details for three existing vendors in the vendor master file, substituting account numbers that routed to a personal account she had opened specifically for this purpose. She changed the routing information in small increments across multiple sessions, correctly reasoning that a sudden wholesale change would be more likely to be noticed during an audit than gradual modifications.

She also created two entirely fictitious vendors — "Apex Courier Services" and "DataStream Logistics" — with plausible business names and invoiced the company for logistics services that were never rendered. Each invoice was carefully kept below the $25,000 solo-approval threshold.

The scheme was discovered during an annual external audit when the auditors cross-referenced vendor banking details against a prior-year snapshot and identified the three changed accounts. The cumulative loss at discovery was $412,000. Forensic analysis of the financial system's audit log confirmed that all changes were made from Carla's account, predominantly during lunch hours and in the final 30 minutes of the business day — windows where supervisory oversight was minimal.

The investigation further revealed that the vendor master file had no change notification workflow — updates could be made without any alert to Finance management — and that the fictitious vendor creation process required only a single approver (the same person who later submitted invoices).

---

## 3. Actor Profile

| Field | Value |
|-------|-------|
| **Archetype** | MAL |
| **Typical Role** | AP Specialist, Finance Manager, Controller, Procurement Officer, CFO |
| **Access Level** | Standard to Power User within financial systems |
| **Technical Sophistication** | Low — the attack uses legitimate financial system features, not technical exploits |
| **Motivation** | Personal financial gain |
| **Insider Knowledge Used** | Knowledge of approval thresholds; knowledge of which changes trigger notifications vs. which are silent; knowledge of vendor onboarding processes |

---

## 4. Attack Phases

### Phase 1 — Target Selection and Planning
**Description:** Insider identifies the financial controls and approval workflows they will exploit, selecting methods that stay below detection thresholds.
**Observable Actions:**
- Access to vendor master file outside normal transactional workflow (browsing vs. processing)
- Review of financial approval policy documentation
- Queries against vendor payment history to identify high-value, regularly paid vendors suitable for banking detail substitution

### Phase 2 — Fraudulent Record Creation or Modification
**Description:** Insider modifies vendor banking details or creates fictitious vendor records.
**Observable Actions:**
- Vendor master file update: banking routing number or account number changed
- New vendor created by the same user who will later submit invoices to that vendor (segregation of duties violation)
- Vendor address or contact information changed to details the insider controls
- Changes made in small increments across multiple sessions (incremental fraud pattern)

### Phase 3 — Invoice Submission and Approval
**Description:** Insider submits fraudulent invoices and routes them through the approval process, exploiting their own approval authority or a weak approval chain.
**Observable Actions:**
- Invoice created for a vendor whose banking details were recently changed
- Invoice amount consistently kept just below the solo-approval threshold
- Invoice submitted and self-approved within the same session (same user creates and approves)
- Batch of invoices submitted in a short window — unusual for a single approver
- Invoices submitted for services with no corresponding purchase order or delivery confirmation

### Phase 4 — Payment Execution and Cover
**Description:** Payment is executed to the fraudulent account. Insider may modify records to obscure the audit trail.
**Observable Actions:**
- Payment batch processed to accounts with recently changed banking details
- GL entries modified post-payment to reclassify the transaction
- Supporting documents (purchase orders, delivery confirmations) absent or created retroactively

---

## 5. Indicators of Behavior (IOBs)

### Technical IOBs

| IOB | Data Source | Signal Strength |
|-----|-------------|----------------|
| Vendor banking detail change (routing number or account number) without a secondary approver or change notification | Financial system audit log | High |
| New vendor created and invoiced by the same user within the same week | Financial system audit log (user correlation) | High |
| Invoice amounts consistently close to but below the solo-approval threshold | Financial system analytics | High |
| Invoice submitted and approved by the same user (segregation of duties violation) | Financial system audit log | High |
| Payment to a vendor whose banking details changed within the past 30 days | Financial system payment log + vendor master audit | High |
| Multiple vendor master changes in a single session outside business hours | Financial system audit log | High |
| New vendor with no purchase order, contract, or business justification on file | Procurement system + financial system cross-reference | High |
| GL reclassification entries made by the same user who created the original invoice | Financial system audit log | Medium |

### Contextual IOBs

| IOB | Source | Notes |
|-----|--------|-------|
| Employee is experiencing financial hardship | HR / Management discretion | Low-weight; never act on alone |
| Employee has recently been passed over for a raise or promotion | HR | Low-weight contextual factor |
| AP or Finance employee whose role is being eliminated | HR | Medium — financial motivation may exist |

---

## 6. Detection Opportunities

| Phase | Detection Opportunity | Required Data Source | Confidence |
|-------|-----------------------|---------------------|-----------|
| Phase 2 | Alert on vendor banking detail change | Financial system audit log | High |
| Phase 2 | Alert on vendor creation by the same user who will submit invoices | Financial system (user correlation across create + submit) | High |
| Phase 3 | Invoice amount clustering just below solo-approval threshold | Financial system analytics / SIEM | High |
| Phase 3 | Same user creates and approves invoice (SOD violation) | Financial system audit log | High |
| Phase 4 | Payment to vendor with recently changed banking details | Financial system payment log + vendor master audit | High |

---

## 7. MITRE ATT&CK Mapping

| Tactic | Technique ID | Technique Name | Applies To |
|--------|-------------|----------------|------------|
| Impact | T1565.001 | Data Manipulation: Stored Data Manipulation | Phase 2 — vendor master modification |
| Defense Evasion | T1070.006 | Indicator Removal: Timestomp | Phase 4 — cover (if timestamps modified) |
| Impact | T1496 | Resource Hijacking | Phase 4 — payment redirection |

> Note: Financial fraud by insiders does not map cleanly to ATT&CK Enterprise (which focuses on IT compromise). The above are the closest applicable techniques. Consider supplementing with COSO Internal Control framework mapping for financial controls specifically.

---

## 8. Legal and Privacy Considerations

- **Financial fraud is criminal:** Invoice manipulation is typically wire fraud (18 U.S.C. § 1343), bank fraud (18 U.S.C. § 1344), or computer fraud (CFAA) in the US. Engage Legal immediately upon suspicion — law enforcement referral is usually warranted.
- **Segregation of duties evidence:** The investigation will need to demonstrate which controls were absent or bypassed. Document the absence of segregation-of-duties controls as both a finding and an enabling factor.
- **Financial system audit logs:** Many ERP systems (SAP, Oracle, NetSuite) require specific audit logging configurations to capture field-level changes to vendor master records. Verify that vendor master change logging is enabled before relying on this data.
- **Recovery:** If funds were transferred to a controlled account, rapid law enforcement notification may allow financial recovery through a bank hold. Time is critical — Legal must be notified within hours, not days.
- **SOX (public companies):** Vendor payment manipulation is a material internal control failure under Sarbanes-Oxley Section 404. External auditors must be notified.

---

## 9. Response Guidance

| Priority | Action | Owner |
|----------|--------|-------|
| 1 | Notify Legal and Finance leadership immediately | IR Lead / CISO |
| 2 | Freeze any pending payments to vendors with recently changed banking details | Finance / Treasury |
| 3 | Preserve all financial system audit logs and ERP change records | IR |
| 4 | Identify all transactions involving modified vendor accounts | IR + Finance |
| 5 | Engage law enforcement if criminal referral is warranted (wire fraud) | Legal |
| 6 | Notify bank(s) — time-critical for fund recovery | Legal + Finance |
| 7 | Notify external auditors if the company is SOX-governed | Legal + CFO |
| 8 | Engage HR for employment action | Legal + HR |

**Linked Playbook:** PB-FR-001 (create using `playbooks/_template.md`)

---

## 10. Linked Artifacts

| Artifact Type | ID | File |
|---------------|----|------|
| Validation Test Case | TC-FR-001 | validation/test-cases/TC-FR-001.md (create) |
| Detection | DET-FR-001 | detections/ (create) |
| Hunt Hypothesis | HYP-FR-001 | hunting/hypotheses/HYP-FR-001.md (create) |

---

## 11. Related Scenarios

| Scenario ID | Name | Relationship |
|-------------|------|-------------|
| UA-001 | After-Hours Unauthorized System Access | Financial fraud is often executed after hours |
| PM-001 | Privilege Escalation via Access Abuse | Escalated access may be needed to bypass approval thresholds |

---

## Changelog

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | 2026-03-26 | SIDTVF Core Team | Initial creation |

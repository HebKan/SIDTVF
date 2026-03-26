# Starter Pack: Healthcare and Life Sciences

> Use this starter pack if your organization is a hospital, health system, health plan, pharmaceutical company, medical device manufacturer, clinical research organization, or any entity subject to HIPAA.

---

## Why Healthcare Has Elevated Insider Threat Risk

Healthcare organizations face a distinct insider threat profile:
- **Workforce scale:** Large clinical and administrative workforces with broad access to EHR systems
- **Data value:** Protected health information (PHI) is worth 10–20x the value of a credit card record on criminal markets
- **Cultural factors:** Healthcare workers share a "care first" culture that can create resistance to monitoring — design your program with this in mind
- **Regulatory stakes:** A single insider-caused HIPAA breach of 500+ records triggers HHS notification, public breach portal listing, and potential OCR investigation
- **Dual risk:** PHI theft for financial fraud (healthcare identity fraud) and PHI snooping (curiosity about VIP patients, family members) are both common and both reportable

---

## Your Starting Scenarios

Start with these three scenarios in this order:

### Priority 1 — PM-001: Privilege Escalation via Access Abuse

**Why:** Inappropriate EHR access is the most commonly reported HIPAA breach category involving a workforce member. Nurses accessing records of patients not in their care, admins browsing celebrity records, and employees accessing records of family members are all PM-001 variants.

**Minimum technical requirement:** EHR audit logging (Epic, Cerner, Meditech, and most major EHR platforms have built-in audit logging). You do not need a SIEM to start — most EHR platforms have built-in inappropriate access reports.

**First detection to deploy:** Any access to patient records by a user whose job function has no clinical relationship to those patients (e.g., a billing clerk accessing ICU records).

**Healthcare-specific IOB:** Time-of-access patterns are particularly useful — EHR access at 3am by a day-shift nurse is highly anomalous.

### Priority 2 — DE-001: Cloud Storage Upload Exfiltration

**Why:** Patient lists, lab results, and insurance data are increasingly targeted for export to personal storage before a workforce member leaves to work for a competitor or starts a private practice. Clinical staff download entire patient panels.

**Minimum technical requirement:** Web proxy logging and a CASB or DLP solution capable of inspecting HTTPS traffic.

**Healthcare-specific note:** Distinguish between legitimate patient portal exports (authorized) and bulk downloads to personal storage (unauthorized). Focus your detection on the **volume and destination** rather than the act of download.

### Priority 3 — TP-001: Vendor Credential Abuse

**Why:** Healthcare IT environments rely heavily on third-party vendors for EHR support, imaging systems, and managed IT services. Vendor-caused breaches are the second most common type of HIPAA business associate (BA) breach.

**Minimum technical requirement:** Privileged access management (PAM) logging for vendor accounts; network segmentation monitoring.

**Healthcare-specific requirement:** Every vendor with PHI access must have a signed Business Associate Agreement (BAA). Insider threat monitoring of vendor accounts is required under HIPAA's BAA provisions.

---

## Healthcare-Specific Legal Requirements

Before deploying any monitoring:

| Requirement | Source | Action Required |
|-------------|--------|----------------|
| Employee monitoring notice | HIPAA Workforce Security (§164.308(a)(3)), state law | Issue updated monitoring notice before deploying new detection tools |
| Business Associate Agreement for monitoring tools | HIPAA §164.308(b) | Confirm your SIEM, CASB, and UEBA vendors have signed BAAs |
| Minimum Necessary standard | HIPAA §164.502(b) | Ensure monitoring collects only data necessary for the insider threat program; do not collect clinical data beyond what's needed for behavioral analysis |
| EHR audit log requirements | Meaningful Use / ONC certification | EHR systems must maintain access logs; confirm your EHR audit logs are complete and queryable |
| State privacy laws | Varies (CA, TX, NY, IL have additional requirements) | Review with Legal before deployment |

---

## Recommended 90-Day Plan

**Days 1–30:**
- Complete the maturity self-assessment
- Confirm EHR audit logging is complete and accessible
- Issue updated monitoring notice to all workforce members
- Run validation test for PM-001 using synthetic data

**Days 31–60:**
- Deploy inappropriate EHR access detection (PM-001) using your EHR's built-in audit reports
- Configure weekly review cadence for high-risk access patterns
- Review all current vendor BAAs — confirm monitoring tools are covered
- Begin DE-001 detection deployment

**Days 61–90:**
- Run first tabletop exercise using PM-001 scenario
- Establish KPI baseline (KPI-01 through KPI-04)
- Brief Privacy Officer and Compliance on program status
- Present to CISO using the executive briefing template

---

## Healthcare-Specific KPI Benchmarks

| KPI | Healthcare Benchmark | Notes |
|-----|---------------------|-------|
| EHR Inappropriate Access Rate | < 0.5% of monthly active users | Industry benchmark from AHIMA and OCR data |
| Days to detect EHR snooping | < 7 days | Many organizations still detect via complaint rather than monitoring |
| HIPAA breach incidents involving insiders | Downward trend YoY | Report to Compliance and Board annually |
| Vendor access review completion | 100% quarterly | All vendor accounts reviewed every 90 days |

---

## Common Healthcare Mistakes

- **Starting with DE-001 instead of PM-001:** EHR snooping is far more prevalent than cloud exfiltration in healthcare. Start with the highest-frequency scenario.
- **Treating all EHR access equally:** Not all access anomalies are equal — a physician accessing a patient in the next unit is different from an AP clerk accessing an oncology record. Build role-based baselines.
- **Forgetting the Break-the-Glass exception:** Emergency access overrides (Break-the-Glass) are legitimate but must be reviewed within 24–48 hours. Your detection must exclude these from alert queues but include them in audit review.
- **Not involving the Privacy Officer:** The HIPAA Privacy Officer must be a stakeholder in this program. Many healthcare organizations keep Privacy and Security in separate silos — unify them for insider threat response.

---

## Regulatory References

| Regulation | Relevant Provision | Insider Threat Implication |
|-----------|-------------------|-----------------------------|
| HIPAA Security Rule | 45 CFR §164.308(a)(1) | Risk analysis must include insider threat scenarios |
| HIPAA Security Rule | 45 CFR §164.312(b) | Audit controls required for ePHI access |
| HIPAA Security Rule | 45 CFR §164.308(a)(5) | Security awareness training requirement |
| HITECH Act | §13405(c) | Workforce member access to unsegregated data is a HIPAA violation |
| ONC Certification | EHR audit log requirements | Clinical systems must maintain user-level access logs |

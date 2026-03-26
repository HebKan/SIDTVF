# Starter Pack: Financial Services

> Use this starter pack if your organization is a bank, credit union, broker-dealer, investment adviser, insurance company, fintech, or any entity subject to SOX, GLBA, PCI DSS, SEC regulations, or state financial services regulations.

---

## Why Financial Services Has Elevated Insider Threat Risk

Financial services organizations face a distinct insider threat profile:
- **Data value:** Account credentials, credit card data, and customer financial records are immediately monetizable
- **Insider access scope:** Front-office staff have legitimate access to customer accounts and transaction data; back-office staff have access to payment processing and wire transfer systems
- **MNPI exposure:** Any employee with knowledge of unreleased earnings, M&A activity, or material regulatory matters is a potential MNPI leakage risk under securities law
- **Regulatory stakes:** A single SOX control failure or SEC disclosure violation can result in personal liability for senior officers; a GLBA breach triggers FTC enforcement
- **High-value targets:** Trading desks, M&A teams, corporate treasury, and client service teams are priority targets for insider threat actors and external actors seeking insider access (AC-001)
- **Separation of duties:** Financial fraud often involves circumventing internal controls — FR-001 scenarios are more prevalent in financial services than any other sector

---

## Your Starting Scenarios

### Priority 1 — UA-001: After-Hours Unauthorized System Access

**Why:** After-hours system access is particularly anomalous in financial services — most transaction processing occurs during business hours. Access to trading systems, customer databases, or payment processing outside normal business hours by a non-operations staff member is a high-confidence signal.

**Minimum technical requirement:** Authentication logs with timestamps (Active Directory, VPN, application logs), SIEM.

**Finance-specific IOB:** Look for after-hours access to wire transfer systems, trading platforms, and customer account databases — these are the highest-value targets.

### Priority 2 — FR-001: Invoice and Vendor Payment Manipulation

**Why:** Accounts payable fraud is consistently one of the most prevalent insider fraud types in financial services and at finance departments in all sectors. Segregation of duties failures (a single employee who can both add vendors and approve payments) create the opportunity. Departing employees with lingering access are a common vector.

**Minimum technical requirement:** ERP/accounting system audit logs (SAP, Oracle, NetSuite, QuickBooks Enterprise). Most enterprise accounting systems have built-in audit logging — you do not need a SIEM to start.

**Finance-specific detection:** Alert on any change to a vendor's bank account details within 30 days of a payment to that vendor. This single rule catches the majority of vendor payment fraud cases.

### Priority 3 — AC-001: Insider Account Compromise by External Actor

**Why:** Financial services employees are high-value targets for AiTM (adversary-in-the-middle) phishing attacks. External actors who compromise a trader's or banker's account can access client portfolios, execute unauthorized transactions, and exfiltrate MNPI. The AC-001 scenario is increasingly common in financial services.

**Minimum technical requirement:** Entra ID / Active Directory sign-in logs with conditional access policies; MFA enforcement.

**Finance-specific note:** Impossible travel detection is particularly effective in financial services because most employees have predictable travel patterns. A trader who is always in London appearing in Singapore 4 hours later is an immediate high-confidence signal.

---

## Financial Services-Specific Legal Requirements

| Requirement | Source | Action Required |
|-------------|--------|----------------|
| SOX Section 302/404 | Sarbanes-Oxley Act | Document insider threat controls as part of ICFR assessment; material weaknesses in access controls must be disclosed |
| GLBA Safeguards Rule | 16 CFR Part 314 (updated 2023) | Risk assessment must cover insider threats; incident response plan required; report qualifying events to FTC within 30 days |
| SEC Regulation S-P | 17 CFR Part 248 | Safeguards for customer financial records; amended 2023 rules tighten incident reporting requirements |
| PCI DSS v4.0 | Requirement 12.6 | Security awareness training; Requirement 10: audit log requirements; Requirement 7: access control |
| FINRA Rule 3110 | Financial Industry Regulatory Authority | Supervision requirements that overlap with insider threat monitoring for broker-dealers |
| State regulations | NY DFS 23 NYCRR 500, CA DFPI, others | State-level cybersecurity regulations for financial institutions — many have explicit insider threat provisions |

---

## MNPI-Specific Guidance

Material Nonpublic Information (MNPI) is a critical insider threat vector unique to publicly traded companies and their advisors:

**Who is at risk:** Employees with access to:
- Unreleased earnings or financial forecasts
- M&A transaction details (both buy-side and sell-side)
- Regulatory filings before public release
- Material contract wins/losses

**Detection approach:**
- Track access to financial model repositories, deal rooms, and board materials
- Alert on bulk downloads from M&A data rooms
- Correlate document access with stock trades in employee accounts (requires HR/compliance data feed)
- DE-003 (AI tool leakage) has special urgency for MNPI: pasting deal details into ChatGPT may constitute unauthorized disclosure under SEC Regulation FD

**Legal note:** Organizations should consult Legal and Compliance before designing MNPI-specific monitoring. The intersection of insider trading law, privilege, and employee monitoring is complex.

---

## Recommended 90-Day Plan

**Days 1–30:**
- Complete maturity self-assessment
- Confirm accounting system audit logging is complete and queryable
- Issue updated monitoring notice to all employees
- Deploy UA-001 after-hours detection using authentication logs

**Days 31–60:**
- Deploy FR-001 vendor payment change detection in accounting system
- Establish baseline for normal vendor master change frequency
- Begin AC-001 impossible travel detection in identity platform
- Review separation of duties gaps in payment approval workflows

**Days 61–90:**
- Run first tabletop exercise using FR-001 scenario (invite Finance, Audit, Legal)
- Deploy MFA for all financial systems if not already in place
- Establish KPI baseline and brief CISO
- Engage Compliance on SOX ICFR documentation for new controls

---

## Financial Services-Specific KPI Benchmarks

| KPI | Finance Benchmark | Notes |
|-----|------------------|-------|
| After-hours financial system access rate | Establish baseline; alert on 2SD above mean | Zero tolerance for unexplained access to wire transfer systems |
| Vendor master changes reviewed | 100% same-day | Every banking detail change must be reviewed |
| MFA enrollment rate | 100% for all financial system users | Non-negotiable for SOX compliance |
| Mean time to detect unauthorized account access | < 1 hour | Critical for limiting transaction exposure window |

---

## Common Finance Mistakes

- **Underestimating the FR-001 scenario:** Invoice fraud is often dismissed as "an accounting problem, not a security problem." The insider threat program should own the detection; Finance owns the controls.
- **Not covering cloud-based financial tools:** QuickBooks Online, Expensify, Concur, and similar cloud finance tools are often excluded from monitoring. These are high-value targets.
- **No departing employee protocol for finance staff:** Finance employees with lingering access to payment systems represent extreme risk. Access must be revoked at termination, not "when IT gets around to it."
- **Treating external audit as insider threat response:** SOX auditors review controls; they do not investigate insider incidents. The IR and Legal functions must be clearly differentiated from the audit function.

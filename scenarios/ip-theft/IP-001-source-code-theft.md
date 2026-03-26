# Scenario Card: IP-001 — Source Code and Proprietary Algorithm Theft

| Field | Value |
|-------|-------|
| **Scenario ID** | IP-001 |
| **Name** | Source Code and Proprietary Algorithm Theft |
| **Version** | 1.0 |
| **Status** | Active |
| **Created** | 2026-03-26 |
| **Last Updated** | 2026-03-26 |
| **Author** | SIDTVF Core Team |
| **Category** | Intellectual Property Theft |
| **Category Code** | IP |
| **Actor Archetype(s)** | MAL |
| **Severity** | Critical |
| **Minimum Maturity Level** | 3 |

---

## 1. Description

A departing engineer or developer systematically clones, downloads, or exports proprietary source code, machine learning models, algorithms, or technical architecture documents with the intent to use this intellectual property at a new employer or to commercialize independently. Unlike general data exfiltration, IP theft targets specific, high-value technical artifacts that represent the organization's core competitive advantage and may be used directly in a competing product.

---

## 2. Narrative Case Study

Nathan was a principal machine learning engineer at a fintech company specializing in credit risk modeling. His proprietary models, trained on six years of anonymized transaction data, represented the core of the company's competitive differentiation. After quietly interviewing with a direct competitor over a two-month period, Nathan accepted an offer that was contingent, in the recruiter's words, on him "bringing what you know."

In the four weeks between accepting the offer and his last day, Nathan systematically cloned the company's most sensitive model repositories from the internal GitLab instance. He also downloaded model weights, training pipeline configurations, and the feature engineering documentation that described the organization's proprietary data transformation methods. He used his authorized access to the ML platform to export three pre-trained models in a portable format.

To avoid detection, Nathan spread his activity across multiple tools: some repositories were cloned via CLI using his valid SSH keys, others were downloaded as zip archives through the GitLab web UI, and the model exports were initiated through the MLflow tracking server's export API — an operation that was legitimate in isolation, since model export was part of his normal workflow. He stored everything in a personal NAS device connected to his home network, which he had synced to his work laptop via a VPN connection during late evenings.

The exfiltration was not discovered until six months after Nathan's departure, when a competitor launched a product that performed remarkably well on the same credit risk benchmarks. A former colleague who had joined the competitor recognized the model's output behavior. A forensic investigation of GitLab and MLflow audit logs confirmed the cloning pattern: 34 repositories accessed and exported in a 22-day window, compared to Nathan's average of 2–3 repository clones per week in the prior year. The ML model exports had been individually logged by MLflow but never reviewed.

---

## 3. Actor Profile

| Field | Value |
|-------|-------|
| **Archetype** | MAL |
| **Typical Role** | Software Engineer, ML Engineer, Data Scientist, Architect, Research Scientist |
| **Access Level** | Standard to Power User — legitimate access to code and model repositories |
| **Technical Sophistication** | High — developers understand version control, APIs, and how to avoid obvious detection |
| **Motivation** | Competitive advantage at new employer; direct commercialization of stolen IP; fulfilling implicit or explicit expectation from a recruiting employer |
| **Insider Knowledge Used** | Knowledge of which repositories contain the most valuable IP; knowledge of which export methods are monitored vs. unmonitored; familiarity with the technical stack that makes IP portable |

---

## 4. Attack Phases

### Phase 1 — IP Inventory and Targeting
**Description:** Insider identifies the most valuable and portable IP assets to target before departure.
**Observable Actions:**
- Access to repositories, model registries, or document stores outside the user's current project assignment
- Search queries in code repositories targeting high-value terms (proprietary algorithm names, model architecture terms, patent-pending features)
- Access to architecture documents, design specs, or README files describing proprietary systems

### Phase 2 — Systematic Extraction
**Description:** Insider systematically extracts targeted IP using authorized but unusual methods or at unusual volumes.
**Observable Actions:**
- Repository clone volume significantly above the user's historical baseline
- Git clone / git archive commands executed against multiple repositories not associated with the user's current sprint or project
- Zip download of repository archives from web UI
- ML model export API calls (MLflow, DVC, SageMaker) for models not in the user's active projects
- Batch download of design documents, architecture diagrams, or specification files

### Phase 3 — Staging and Transfer
**Description:** Insider consolidates the extracted IP into a portable format and transfers it out of the organizational network.
**Observable Actions:**
- Large HTTPS transfers to personal cloud storage or NAS device IP
- USB mass storage write events on the developer workstation
- SCP or SFTP transfer to a personal server
- Compression of large repository collections into archives

### Phase 4 — Evidence Minimization
**Description:** Insider may attempt to minimize traces of the extraction.
**Observable Actions:**
- Shell history cleared or commands executed with no-history flags
- Local staging directories deleted after transfer
- Git credential cache cleared
- SSH key rotation or removal from repository platform (to reduce future attribution)

---

## 5. Indicators of Behavior (IOBs)

### Technical IOBs

| IOB | Data Source | Signal Strength |
|-----|-------------|----------------|
| Repository clone volume > 3× the user's 90-day daily average over a 2-week window | GitLab/GitHub audit log + SIEM | High |
| Cloning of repositories not associated with the user's active team or sprint board | GitLab/GitHub audit log + project management tool | High |
| Zip archive download of a repository containing proprietary algorithms | GitLab/GitHub audit log (DownloadRepository event) | High |
| ML model export API call for models outside the user's active experiments | MLflow / SageMaker / AzureML audit log | High |
| Large HTTPS POST or SCP transfer to a non-corporate destination following repository cloning | NGFW / proxy logs + EDR | High |
| USB write event on developer workstation in the period following high-volume repository access | EDR | High |
| SSH key added to a personal account on the same platform (e.g., GitHub) from a work device | EDR / SSH key management audit | Medium |
| Access to IP-sensitive repositories (labeled as proprietary, confidential, or patent-pending) by an account not on the authorized access list | Code repository RBAC + audit | High |
| Departure-correlated spike: repository access surge in the 30 days before a known departure date | Code repository audit + HR departure data | High |

### Contextual IOBs

| IOB | Source | Notes |
|-----|--------|-------|
| Employee is interviewing with a competitor (if known) | HR / Management | High risk indicator |
| Employee gave notice or is in a role being eliminated | HR | High risk — trigger immediate repository access review |
| Employee requested export permissions or asked about data portability | Manager / IT ticket | Medium |

---

## 6. Detection Opportunities

| Phase | Detection Opportunity | Required Data Source | Confidence |
|-------|-----------------------|---------------------|-----------|
| Phase 1 | Access to IP-sensitive repositories outside current project scope | GitLab/GitHub audit log + project-repo mapping | High |
| Phase 2 | Repository clone volume spike vs. 90-day baseline | GitLab/GitHub audit + SIEM/UEBA | High |
| Phase 2 | ML model export for non-active experiments | ML platform audit log | High |
| Phase 3 | Large outbound transfer following repository cloning session | Proxy + NGFW + EDR correlation | High |
| Phase 3 | USB write event following high-volume repository activity | EDR | High |

---

## 7. MITRE ATT&CK Mapping

| Tactic | Technique ID | Technique Name | Applies To |
|--------|-------------|----------------|------------|
| Collection | T1213 | Data from Information Repositories | Phase 1, Phase 2 |
| Collection | T1005 | Data from Local System | Phase 2 |
| Collection | T1560.001 | Archive Collected Data: Archive via Utility | Phase 3 |
| Exfiltration | T1567.002 | Exfiltration Over Web Service: Exfiltration to Cloud Storage | Phase 3 |
| Exfiltration | T1052.001 | Exfiltration Over Physical Medium: Exfiltration over USB | Phase 3 |
| Defense Evasion | T1070.003 | Indicator Removal: Clear Command History | Phase 4 |

---

## 8. Legal and Privacy Considerations

- **Trade secret law:** Source code and proprietary algorithms are typically trade secrets under the Defend Trade Secrets Act (DTSA, 18 U.S.C. § 1836) in the US, or equivalent laws in other jurisdictions. DTSA allows for civil action and criminal prosecution, including emergency injunctive relief to prevent the competitor from using the stolen IP.
- **Time to action:** Trade secret litigation is most effective when pursued quickly. The longer a competitor uses the stolen IP, the more difficult it becomes to unwind. Legal must be engaged within hours of confirmed IP theft, not days.
- **Employee agreements:** Confidentiality, invention assignment, and non-compete/non-solicitation agreements define the legal basis for action. Retrieve and review the employee's signed agreements before confronting them.
- **Competitive intelligence:** The new employer may face liability if they knew or should have known the employee brought stolen IP. Preserve all evidence carefully — it may be needed in civil litigation against the new employer.
- **Computer log admissibility:** Repository audit logs, SSH logs, and GitLab/GitHub events may be required as digital evidence. Engage Legal early to ensure evidence preservation meets admissibility standards.

---

## 9. Response Guidance

| Priority | Action | Owner |
|----------|--------|-------|
| 1 | Preserve all code repository audit logs, ML platform logs, and endpoint telemetry | IR |
| 2 | Notify Legal immediately — engage IP litigation counsel | Legal |
| 3 | Revoke all repository access, SSH keys, and API tokens | IAM / DevOps |
| 4 | Identify the full scope: which repositories, which models, what was exported | IR + Engineering |
| 5 | Review the employee's confidentiality and invention assignment agreements | Legal + HR |
| 6 | Assess whether the new employer is a competitor and whether emergency injunctive relief is appropriate | Legal |
| 7 | Engage HR for employment action | Legal + HR |

**Linked Playbook:** PB-IP-001 (create using `playbooks/_template.md`)

---

## 10. Linked Artifacts

| Artifact Type | ID | File |
|---------------|----|------|
| Validation Test Case | TC-IP-001 | validation/test-cases/TC-IP-001.md (create) |
| Detection | DET-IP-001 | detections/ (create) |
| Hunt Hypothesis | HYP-IP-001 | hunting/hypotheses/HYP-IP-001.md (create) |

---

## 11. Related Scenarios

| Scenario ID | Name | Relationship |
|-------------|------|-------------|
| DE-001 | Cloud Storage Upload Exfiltration | Often used as the final exfiltration channel after repository cloning |
| DE-002 | Email Auto-Forwarding Rule | May be used concurrently to capture design discussion emails |
| UA-001 | After-Hours Unauthorized System Access | IP theft often occurs after hours to avoid observation |

---

## Changelog

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | 2026-03-26 | SIDTVF Core Team | Initial creation |

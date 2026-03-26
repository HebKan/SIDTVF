# Scenario Card: DE-003

| Field | Value |
|-------|-------|
| **Scenario ID** | DE-003 |
| **Name** | AI Tool Data Leakage |
| **Category** | Data Exfiltration |
| **Version** | 1.0 |
| **Status** | Active |
| **Created** | 2026-03-26 |
| **Last Updated** | 2026-03-26 |
| **Severity** | High (Critical if PHI, MNPI, or trade secrets confirmed) |
| **Minimum Maturity Level** | 2 |
| **Actor Archetype(s)** | NEG, MAL |
| **MITRE ATT&CK Version** | Enterprise v15 |

---

## 1. Description

An employee uses a publicly accessible, third-party AI assistant (such as a commercial large language model chat interface) to assist with work tasks, pasting internal data — including sensitive documents, personal information, source code, financial projections, or clinical data — into the AI chat interface. The data leaves the organization's environment via encrypted HTTPS POST to the AI provider's servers, bypassing traditional DLP controls that inspect email and file transfers but not browser-based HTTPS payloads.

The primary actor profile is **Negligent (NEG)**: the employee believes they are acting productively, does not understand the data classification implications, and may genuinely not know that pasting data into a consumer AI tool constitutes an unauthorized disclosure to a third party.

A secondary **Malicious (MAL)** variant exists: a departing employee intentionally uses an AI tool to extract product roadmaps, source code, or customer data under the guise of legitimate AI-assisted work, exploiting CASB blind spots in environments that have not deployed deep packet inspection.

**What makes this scenario distinct:** Unlike most exfiltration scenarios, the primary risk is not intentional theft — it is systemic negligence enabled by a tooling gap. Organizations that have not authorized an enterprise AI tool create pressure for employees to use consumer alternatives, making this scenario a symptom of policy lag behind technology adoption.

---

## 2. Narrative Case Study

Alex Chen, a regulatory affairs specialist at a mid-sized pharmaceutical company, routinely uses AI assistants to help draft regulatory submissions, summarize clinical trial data, and research competitor filings. The company had recently deployed Microsoft 365 Copilot for a subset of users but Alex had not yet been provisioned access. Finding that the commercial ChatGPT interface was faster and more capable for her specific tasks, she began using her personal ChatGPT account daily from her corporate workstation.

Over a six-week period, Alex pasted sections of unreleased clinical trial data, patient outcome summaries, and proprietary drug formulation details into ChatGPT's web interface. She believed the data was appropriately anonymized based on the company's internal drug candidate coding conventions — the same codes used in published papers — and considered the practice equivalent to searching the internet for research assistance.

The behavior was detected when the company's newly deployed Cloud Access Security Broker (CASB) solution, configured with DLP policies for medical terminology and internal project codenames, flagged a series of large HTTP POST requests to `chat.openai.com` from Alex's workstation. Inspection of the request bodies — permissible under the company's monitoring policy and employee agreement — revealed content matching internal drug candidate naming conventions.

Forensic review of 90 days of proxy logs revealed 43 separate chat sessions over six weeks, with progressively larger upload payloads averaging 85 KB per session. A temporal correlation analysis showed that Alex's uploads to ChatGPT consistently occurred within five minutes of her accessing a restricted clinical data repository on the internal SharePoint site. This sequence — internal sensitive data access followed immediately by AI tool upload — became the primary detection signature.

Alex cooperated fully during the investigation, expressing genuine surprise that her activity had been visible to the security team. She had assumed that HTTPS encryption meant her sessions were private. Forensic analysis identified 12 patient records where the combination of drug candidate code, trial site identifier, and demographic information met the HIPAA definition of Protected Health Information (PHI), triggering a formal breach notification assessment. The company's legal team also identified that three of the submitted sessions contained information about an unreleased drug candidate with material nonpublic information (MNPI) implications for the company's publicly traded stock.

The investigation found no evidence of malicious intent. The root cause was identified as a policy gap: the company had a data classification policy but no specific guidance on AI tool usage, and the enterprise Copilot deployment had not been extended to Alex's department. The incident drove an organization-wide AI usage policy and CASB deployment.

---

## 3. Actor Profile

| Attribute | NEG Variant | MAL Variant |
|-----------|-------------|-------------|
| **Motivation** | Productivity, convenience, unaware of risk | Competitive advantage, pre-departure data extraction |
| **Technical Sophistication** | Low — uses standard browser, no evasion | Medium — may use API access, personal device, or VPN to evade CASB |
| **Employment Status** | Active, no disciplinary history | Active or pre-resignation |
| **Data Targets** | Work-relevant data matching current task | High-value IP, customer lists, financial projections |
| **Concealment** | None — activity is visible but misunderstood | May use personal device or home network to avoid corporate proxy |
| **Primary Signal** | Temporal correlation: sensitive system access → AI upload | Volume + targeting + timing relative to resignation |

---

## 4. Attack Phases

### Phase 1 — Tool Selection

| Observable Action | Log Source | IOB Type |
|------------------|------------|----------|
| Employee navigates to AI tool domain (chat.openai.com, gemini.google.com, claude.ai, copilot.microsoft.com with personal account) | Web proxy / DNS | Technical |
| Employee logs in with personal (non-corporate) account credentials | Web proxy (HTTPS POST to auth endpoint) | Technical |
| First-time access to AI domain on corporate network | CASB / proxy | Technical |

### Phase 2 — Data Access

| Observable Action | Log Source | IOB Type |
|------------------|------------|----------|
| Employee accesses internal data repository (SharePoint, Confluence, clinical database, code repository) | M365 audit log, SharePoint audit, application log | Technical |
| Employee downloads, copies, or opens a sensitive document | Endpoint agent (DeviceFileEvents), DLP endpoint agent | Technical |
| High clipboard activity: multiple copy operations in quick succession | Endpoint DLP agent (if deployed) | Technical |

### Phase 3 — Data Input (Exfiltration)

| Observable Action | Log Source | IOB Type |
|------------------|------------|----------|
| HTTP POST to AI tool API endpoint with large request body (> 10 KB payload) | Web proxy (if HTTPS inspection enabled) or CASB | Technical |
| CASB DLP match: request body matches sensitive data patterns (PII, PHI, code patterns, financial terms) | CASB policy engine | Technical |
| Upload occurs within 0–10 minutes of accessing the linked internal data source | SIEM correlation (temporal) | Technical |
| Multiple sessions over multiple days with increasing payload size | SIEM aggregation | Contextual |

### Phase 4 — Persistence in AI Platform

| Observable Action | Log Source | IOB Type |
|------------------|------------|----------|
| Employee returns to same AI conversation in subsequent sessions (session continuation) | Web proxy (cookie-based session tracking, if captured) | Contextual |
| No deletion of conversation history (data retained by AI provider per their data retention policy) | Not directly observable — relevant for breach assessment | Contextual |

---

## 5. Indicators of Behavior (IOBs)

### Technical IOBs

| IOB-ID | Indicator | Threshold / Context | Data Source |
|--------|-----------|--------------------|-----------:|
| IOB-T-001 | HTTP POST to known AI tool domain from corporate network | Any occurrence — establish baseline first | Web proxy |
| IOB-T-002 | Request payload to AI tool > 50 KB | Single session threshold | CASB with HTTPS inspection |
| IOB-T-003 | DLP match on sensitive content patterns (PII regex, PHI terms, internal project codenames) in AI tool upload | Any match | CASB DLP policy engine |
| IOB-T-004 | AI tool access occurs within 10 minutes of accessing a classified internal data source | Temporal correlation | SIEM correlation rule |
| IOB-T-005 | User accesses AI tool with personal (non-corporate) credentials while on corporate network | CASB user identity enrichment | CASB |
| IOB-T-006 | Total daily upload volume to AI tool domains > 500 KB | Aggregated daily threshold | CASB / proxy |
| IOB-T-007 | AI tool access from endpoint DLP-enriched browser process (clipboard data patterns passed to browser) | Any DLP match | Endpoint DLP agent |

### Contextual IOBs

| IOB-ID | Indicator | Significance |
|--------|-----------|-------------|
| IOB-C-001 | Employee does not have access to an enterprise-licensed AI tool | Policy gap indicator — increases NEG risk |
| IOB-C-002 | AI usage increases in volume over multiple weeks | Escalating pattern — behavior is becoming habitual |
| IOB-C-003 | AI tool access correlates with access to multiple sensitive data sources across departments | Broader scope than job function warrants |
| IOB-C-004 | Employee is in Finance, Legal, M&A, R&D, or HR — higher data sensitivity departments | Risk multiplier |
| IOB-C-005 | Employee is within 60 days of known resignation (MAL variant) | Departure-correlated — elevates to High/Critical |

---

## 6. Minimum Viable Detection

> **If you can only deploy one control:** Enable CASB with HTTPS inspection for AI tool domains and configure a DLP policy that alerts on uploads > 50 KB to AI tool URLs. This single control catches both the volume pattern and enables content inspection for sensitive data matches. Without HTTPS inspection, AI tool uploads are invisible to proxy-based detection.

---

## 7. Detection Blind Spots

| Blind Spot | Why It Evades Detection | Mitigating Control |
|-----------|------------------------|-------------------|
| Personal device on home network | Completely outside corporate network visibility | MDM policy prohibiting work data on personal devices; policy + training |
| Corporate device with split-tunnel VPN (AI traffic bypasses proxy) | AI tool traffic routed outside corporate proxy | Full-tunnel VPN policy or CASB agent-based detection |
| API key-based AI access (no browser) | Does not generate browser-visible HTTP traffic | Endpoint DLP monitoring of API call processes |
| Text typed manually rather than pasted (slow disclosure) | No clipboard event; low per-session volume | Policy and training are the primary control for this variant |
| AI tool accessed via mobile device on cellular | No corporate visibility | MDM enrollment and mobile CASB |
| Use of an enterprise AI tool with poor DLP integration | Sanctioned tool with weaker controls | Enterprise AI tool procurement must require DLP integration as a requirement |

---

## 8. MITRE ATT&CK Mapping

| Tactic | Technique | Sub-technique | Notes |
|--------|-----------|--------------|-------|
| Collection | T1213 | Data from Information Repositories | Phase 2 — internal data source access |
| Collection | T1005 | Data from Local System | Phase 2 — file download/copy to local staging |
| Exfiltration | T1567.002 | Exfiltration Over Web Service: Cloud Storage | Phase 3 — upload to AI platform |
| Exfiltration | T1048.003 | Exfiltration Over Unencrypted/Obfuscated Non-C2 Protocol | Phase 3 — when HTTPS inspection not deployed |

---

## 9. Legal and Privacy Considerations

### Data Classification Impact
The severity of breach notification obligations depends on what data type was disclosed:

| Data Type Exposed | Applicable Law | Notification Obligation |
|------------------|---------------|------------------------|
| PHI (patient records, diagnoses) | HIPAA (US) | Breach notification within 60 days (HHS + affected individuals) |
| PII (EU residents) | GDPR Art. 33-34 | Notification to supervisory authority within 72 hours if high risk |
| MNPI (public company financials) | SEC Regulation FD, Sarbanes-Oxley | Legal review required — may trigger insider trading disclosure analysis |
| Trade secrets (source code, formulas, models) | Defend Trade Secrets Act (DTSA) — 18 U.S.C. § 1836 | Civil remedy available; preserving evidence critical |
| California residents' PII | California Consumer Privacy Act (CCPA) | Consumer notification may be required |

### AI Provider Data Retention
Organizations must assess the AI provider's data retention policy:
- **OpenAI (personal accounts):** Conversations retained; may be used for model training unless opted out
- **OpenAI (enterprise API):** Data not used for training; retention configurable
- **Google Gemini (personal):** Activity saved; review Google's retention settings
- **Anthropic Claude.ai:** Retention governed by current privacy policy; enterprise contracts differ

Organizations should contact the AI provider's enterprise data protection team immediately following a confirmed disclosure event. Many providers have a data deletion request process for confirmed accidental disclosures.

### Legal Caution
Organizations should consult legal counsel before determining whether a disclosure event constitutes a reportable breach. Not all disclosures trigger mandatory notification — the risk of harm to data subjects is typically the threshold. However, disclosure to an AI provider constitutes transmission to an unauthorized third party under most data protection frameworks.

---

## 10. Response Guidance

### Immediate (0–4 hours)
1. Preserve CASB logs and proxy logs for the affected period
2. Notify Legal and Privacy teams — they must assess breach notification obligations
3. Identify the specific sessions: what data, what volume, what time window
4. Determine the AI provider and their applicable data retention policy
5. Do NOT notify the employee until Legal advises — preserve investigation integrity

### Short-term (4–72 hours)
6. Assess the data types disclosed — classify each session's content
7. Determine whether PHI, PII, MNPI, or trade secrets were included
8. Contact the AI provider's enterprise/privacy team to request conversation deletion
9. Legal determines breach notification obligations
10. HR and Legal coordinate on employee notification and appropriate response (training, written warning, or escalation depending on intent and severity)

### Longer-term
11. Deploy enterprise AI tool access so employees have a sanctioned, DLP-integrated alternative
12. Update the Acceptable Use Policy to explicitly address AI tool usage
13. Deploy CASB with HTTPS inspection for AI tool domains across the organization
14. Conduct security awareness training on AI tool data risks for all employees

### False Positive Considerations
Not all AI tool usage is a policy violation. Consider:
- Employees using enterprise-licensed AI tools (Microsoft 365 Copilot, corporate ChatGPT Enterprise) — verify whether the detected tool is sanctioned
- Employees pasting only public, non-sensitive information (the CASB DLP policy should not trigger on non-sensitive content)
- IT staff or AI/ML engineers with legitimate need to test AI APIs with sample data

---

## 11. Linked Artifacts

| Type | ID | Status | File |
|------|----|--------|------|
| Detection (Sigma) | DET-DE-003-SIGMA | Planned | detections/sigma/DE-003-ai-tool-upload.yml |
| Detection (KQL) | DET-DE-003-KQL | Planned | detections/queries/DE-003-kql.md |
| Playbook | PB-DE-003 | Planned | playbooks/examples/PB-DE-003-ai-data-leakage.md |
| Hunt Hypothesis | HYP-DE-003 | Planned | hunting/hypotheses/HYP-DE-003.md |
| Validation Test | TC-DE-003 | Planned | validation/test-cases/TC-DE-003.md |
| Simulation | SIM-DE-003 | Planned | simulation/tests/SIM-DE-003.md |

---

## 12. Related Scenarios

| Scenario ID | Relationship |
|-------------|-------------|
| DE-001 | Same exfiltration tactic (web service upload) but intentional; DE-003 is primarily negligent |
| PV-001 | Shadow IT root cause — DE-003 is often driven by the same tooling gap as PV-001 |
| IP-001 | MAL variant of DE-003 (source code theft via AI tool) overlaps with IP-001 |

---

## 13. Changelog

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | 2026-03-26 | SIDTVF Core Team | Initial creation |

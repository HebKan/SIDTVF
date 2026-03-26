# Acceptable Use Policy — AI Tools Addendum Template

> **Instructions:** This addendum supplements your organization's existing Acceptable Use Policy (AUP) to address AI tool usage. If your organization does not have an AUP, create one before using this addendum. Have Legal counsel review before publishing.
>
> **Why this addendum now:** Consumer AI tools (ChatGPT, Gemini, Claude.ai, Copilot personal accounts, etc.) are the fastest-growing unmanaged data exfiltration vector in enterprise environments. Most existing AUPs predate these tools and do not address them. See SIDTVF scenario DE-003 for the detection framework.

---

# Acceptable Use Policy — Addendum: Artificial Intelligence (AI) Tools

**[Organization Name]**
Effective Date: [Date] | Supersedes: [Prior version or "New addition to AUP"]
Applies to: All employees, contractors, and third parties with access to Company information

---

## 1. Purpose

This addendum establishes the Company's policy on the use of artificial intelligence (AI) tools and services, including generative AI assistants, large language model (LLM) chat interfaces, AI-powered code assistants, and AI content generation services.

The rapid proliferation of consumer AI tools has created new risks for the unauthorized disclosure of confidential, sensitive, and regulated information. This policy ensures that productivity benefits of AI tools are realized without creating data loss, regulatory, or legal exposure for the organization.

---

## 2. Definitions

**Sanctioned AI Tool:** An AI tool that has been formally evaluated, approved, and provisioned by [Organization Name] for business use. These tools have contractual data protection terms, DLP integration, and are accessed via corporate accounts. [List current sanctioned tools: e.g., Microsoft 365 Copilot, GitHub Copilot Enterprise, [other].]

**Consumer AI Tool:** An AI tool accessed via a personal account that does not have a corporate data processing agreement with [Organization Name]. Examples include: ChatGPT (personal account), Google Gemini (personal account), Claude.ai (personal account), Meta AI, Perplexity.ai personal tier.

**AI-Assisted Output:** Any document, code, email, analysis, or other content that was substantially generated or edited using an AI tool.

---

## 3. Permitted AI Tool Usage

### 3.1 Sanctioned AI Tools
Employees may use currently sanctioned AI tools for business purposes subject to the restrictions in Section 4.

To request approval of a new AI tool, submit a request to the IT Security team at [contact]. New tools are evaluated for: data processing terms, retention policies, DLP integration, and security certifications. Evaluation typically takes [N] business days.

### 3.2 Consumer AI Tools — Personal Use
Consumer AI tools may be used on personal devices with personal accounts for **non-work-related** personal tasks only. Do not use personal AI tools for any work-related task if that task requires you to input Company information.

### 3.3 Consumer AI Tools — Research and Awareness
Employees may use consumer AI tools to research industry topics, learn about new AI capabilities, or explore public information — **provided no Company information is entered into the tool**.

---

## 4. Prohibited Activities

The following activities are prohibited regardless of which AI tool is used:

**4.1 Prohibited data inputs — never enter the following into any AI tool that is not a sanctioned Company tool:**
- Customer names, contact information, account details, or any personally identifiable information (PII)
- Protected health information (PHI) as defined under HIPAA
- Payment card information (PCI data)
- Employee personal information (compensation, performance, health, disciplinary records)
- Material nonpublic information (MNPI) — financial projections, unreleased earnings data, M&A activity
- Proprietary source code, algorithms, models, or technical specifications
- Attorney-client privileged communications or confidential legal strategy
- Confidential business strategy, pricing, customer pipeline, or competitive intelligence
- Regulated research data, clinical trial data, or investigational drug information
- Any document classified as Confidential or above under the Company's data classification policy

**4.2 Additional prohibitions:**
- Using a consumer AI tool as a substitute for a sanctioned tool that has been made available for your role
- Sharing AI tool credentials or API keys with unauthorized users
- Using AI tools to generate content that could constitute fraud, harassment, or policy violations
- Representing AI-generated content as original human work in contexts where this distinction is material (e.g., regulatory filings, legal declarations, financial certifications)

---

## 5. Obligations When Using Sanctioned AI Tools

Even when using Company-sanctioned AI tools, employees must:

5.1 Follow data classification rules — do not enter data into an AI tool that is classified at a higher level than the tool's approved use scope

5.2 Review and validate AI-generated content before using it — AI tools can produce factually incorrect, biased, or incomplete information

5.3 Not disable or bypass DLP controls integrated with the sanctioned AI tool

5.4 Report any unexpected data disclosure (e.g., the AI tool surfaces data that appears to belong to another user) to the Security team immediately

---

## 6. Monitoring

The Company monitors AI tool usage on corporate networks and devices as described in the Employee Monitoring Notice. This includes:
- HTTP traffic to AI tool domains via web proxy and CASB
- Content inspection of uploads to AI tool interfaces where HTTPS inspection is deployed
- Detection of AI tool usage patterns that correlate with access to sensitive internal data sources

Monitoring is conducted for security and compliance purposes, not to evaluate employee productivity.

---

## 7. Disclosure and Approval for Regulated Use

Some use cases require additional approval before using AI tools, even sanctioned tools:

| Use Case | Approval Required |
|----------|------------------|
| Using AI to process PHI | Privacy Officer + Legal + HIPAA BAA confirmation with AI vendor |
| Using AI to process EU personal data | DPO review of data flows and transfer mechanisms |
| Using AI in financial reporting workflows | CFO + Legal review of AI-generated content disclosure requirements |
| Using AI to process classified government data (CMMC environments) | CMMC assessor consultation |

---

## 8. Consequences of Policy Violation

Violation of this policy may result in:
- Mandatory security awareness training
- Formal written warning
- Termination of employment, in cases of serious or repeated violation
- Legal liability where a violation results in a data breach with regulatory notification obligations
- Referral to law enforcement in cases of intentional, criminal misuse

The Company will assess violations in context, recognizing that many AI tool policy violations result from insufficient awareness rather than malicious intent. The Company's primary response to first-time, non-malicious violations is education and remediation.

---

## 9. Questions and Requests

| Contact | Purpose |
|---------|---------|
| IT Security | Request new AI tool approval, report a potential incident |
| Legal Counsel | Questions about regulatory or contractual requirements |
| Privacy / DPO | Questions about personal data processing |
| Your Manager | Guidance on whether a specific task requires Company information |

---

## Acknowledgment

By signing or electronically acknowledging this addendum, you confirm that you have read and will comply with this policy.

Employee Name: _______________________________
Signature: _______________________________
Date: _______________________________

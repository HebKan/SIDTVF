# Starter Pack: Technology and Software Companies

> Use this starter pack if your organization develops software, operates cloud platforms, provides technology services, or employs a primarily technical workforce. Also applicable to any organization with a substantial R&D or engineering function.

---

## Why Technology Companies Have Elevated Insider Threat Risk

Technology companies face a distinct insider threat profile:
- **IP concentration:** Source code, ML models, proprietary algorithms, and product roadmaps are often the organization's most valuable assets — and they are easily portable (a developer can clone a repository in minutes)
- **High mobility workforce:** Engineers, data scientists, and product managers move between companies frequently; IP theft often occurs before or immediately after resignation
- **Broad system access:** Developers often have access to production infrastructure, customer data, and intellectual property far beyond what is needed for their current task — access hygiene is frequently poor
- **Cloud-native complexity:** Multi-cloud, microservices, and CI/CD pipelines create thousands of potential exfiltration pathways (API keys, container registries, model artifact stores)
- **AI/LLM risk:** Technology companies are early adopters of AI tools; employees paste source code, model architectures, and proprietary data into consumer AI tools more often than in any other sector (DE-003 is your highest-frequency emerging risk)
- **Competitive intelligence:** Competitors actively recruit from technology companies specifically to gain access to IP — both through legitimate hiring and through deliberate placement

---

## Your Starting Scenarios

### Priority 1 — IP-001: Source Code and Proprietary Algorithm Theft

**Why:** This is the defining insider threat scenario for technology companies. Repository cloning, bulk file downloads from code repositories, and model weight exports before departure are extremely common. The IP-001 scenario is often discovered 6–12 months after the departure when a competitor product launches.

**Minimum technical requirement:** Git repository audit logging (GitHub Enterprise, GitLab, Bitbucket all have audit APIs); DLP or CASB for code platforms.

**Tech-specific detection:** Alert on any user cloning more than 3 repositories in a single day outside normal working patterns, especially when correlated with a known or anticipated departure date. The volume and variety of repositories (rather than depth in a single project) is the key signal.

### Priority 2 — DE-003: AI Tool Data Leakage

**Why:** Technology employees paste source code, system architecture diagrams, and proprietary algorithms into consumer AI tools (ChatGPT, Copilot, Gemini) constantly. This is the highest-volume emerging data leakage vector in tech. Unlike malicious exfiltration, most of these cases are negligent — employees genuinely do not understand that pasting code into ChatGPT discloses it to OpenAI.

**Minimum technical requirement:** CASB with HTTPS inspection for AI tool domains.

**Tech-specific note:** Enterprise GitHub Copilot, JetBrains AI Assistant with enterprise licensing, and similar enterprise AI coding tools have much stronger data protection terms than consumer alternatives. Providing engineers with enterprise-licensed AI tools and actively communicating the data protection difference dramatically reduces DE-003 risk.

### Priority 3 — DE-001: Cloud Storage Upload Exfiltration

**Why:** Developers have native comfort with cloud tools and frequently use personal Dropbox, Google Drive, or GitHub (personal accounts) to "back up" work. The line between legitimate personal backup and unauthorized IP exfiltration is often narrow.

**Minimum technical requirement:** Web proxy or CASB; endpoint DLP for code file types.

**Tech-specific note:** Standard DE-001 thresholds (50 MB/hour) need adjustment for developer workflows — source code repositories can be gigabytes. Tune your detection to focus on **file types** (`.py`, `.ipynb`, `.pkl`, `.h5`, `.json`, `.proto`, `.tf`) and **destination** (personal accounts) rather than pure volume.

---

## Technology-Specific Scenarios Not Yet in SIDTVF

The following scenarios are high-priority for technology companies and are candidates for future SIDTVF development:

| Scenario | Description | MITRE Technique |
|----------|-------------|----------------|
| Cloud credential exfiltration | Developer exports AWS access keys or service account credentials | T1552.005 |
| Container registry IP theft | Developer pushes proprietary container images to personal registry | T1537 |
| ML model weight exfiltration | Data scientist exports trained model files before departure | T1005, T1052.001 |
| API key hoarding | Employee creates personal API keys on corporate platforms before departure | T1098 |
| Private repository scraping | Automated script bulk-clones all accessible repositories | T1213, T1119 |

---

## Technology-Specific Legal Requirements

| Requirement | Source | Action Required |
|-------------|--------|----------------|
| DTSA trade secret protection | Defend Trade Secrets Act, 18 U.S.C. § 1836 | Document trade secrets; ensure employee agreements include IP assignment and confidentiality provisions; preserve evidence for DTSA civil claims |
| IP assignment agreements | State law (CA, DE, WA, NY vary) | Review employee IP assignment agreements with Legal; CA restricts overly broad IP assignments |
| CCPA (if CA users/employees) | California Consumer Privacy Act | Customer data accessed or exfiltrated may trigger notification obligations |
| CFAA | Computer Fraud and Abuse Act, 18 U.S.C. § 1030 | Access beyond authorization — relevant for PM-001 variants; recent case law (Van Buren) limits CFAA scope for insiders |
| Export control | EAR, ITAR (if defense tech) | Exfiltration of export-controlled technical data to foreign nationals requires special handling |
| NDAs and non-compete agreements | State law varies significantly | Coordinate IP theft response with Legal on enforceability of NDAs and non-competes in the relevant state |

---

## Developer-Specific Controls

Standard security controls often need adaptation for developer environments:

| Control | Standard Approach | Developer-Adapted Approach |
|---------|------------------|---------------------------|
| DLP | Block file types with sensitive content | Alert on code file types (.py, .ipynb, .pkl, model files) being sent to personal storage; do not block git operations to personal repos — alert instead |
| Repository access | Role-based access | Repository access should follow need-to-know; developers need access to their current project, not the entire codebase |
| API key management | Rotate regularly | Use OIDC/workload identity federation instead of long-lived keys; alert on creation of new long-lived API keys |
| CI/CD pipeline monitoring | Monitor build outputs | Detect unusual artifact exports or image pushes from pipelines |
| Remote access | VPN | Zero Trust Network Access (ZTNA) provides more granular logging for developer access patterns |

---

## Recommended 90-Day Plan

**Days 1–30:**
- Complete maturity self-assessment
- Audit repository access permissions — identify employees with access beyond current project scope
- Confirm GitHub/GitLab audit logging is complete and flowing to SIEM
- Issue updated monitoring notice and AUP addendum covering AI tools (use docs/policies/aup-addendum.md)

**Days 31–60:**
- Deploy IP-001 detection: repository clone volume alerting
- Deploy DE-003 detection: CASB alert for AI tool uploads from engineering team
- Begin DE-001 detection for code file types specifically
- Procure or expand enterprise AI tool licenses to give engineers a sanctioned alternative

**Days 61–90:**
- Run first tabletop exercise using IP-001 scenario (invite Engineering Lead, Legal, HR)
- Implement departing employee repository access freeze protocol: upon resignation, read access only, no new clones
- Establish KPI baseline
- Review contractor and vendor access to repositories — apply same monitoring controls

---

## Departing Employee Protocol for Tech Companies

The highest-risk period for IP theft in technology companies is the **2 weeks between resignation and last day**. The following protocol significantly reduces risk:

| Trigger | Action | Owner | Timeline |
|---------|--------|-------|----------|
| Resignation received | Add to DepartingEmployees watchlist | HR → SOC | Same day |
| Resignation received | Audit last 30 days of repository activity | Security | Within 24 hours |
| Resignation received | Remove access to production infrastructure (read-only for code review only) | IAM | Within 24 hours — coordinate with manager |
| 7 days before last day | Full access review: identify any access that wasn't removed in prior step | IAM | |
| Last day | Revoke all access within 1 hour of departure | IAM | |
| 30 days after departure | Run IP-001 hunt query for activity in the 90 days before departure | Threat Hunt | |

---

## Common Tech Mistakes

- **Over-relying on NDAs without monitoring:** An NDA is a legal remedy after theft occurs. Monitoring detects theft before it's complete.
- **Not monitoring personal GitHub accounts:** Employees pushing code to personal GitHub repos is extremely common and often undetected. CASB can detect GitHub.com traffic; endpoint DLP can detect git push commands.
- **Treating AI tool usage as an HR issue instead of a security issue:** AI tool data leakage is a DE-003 detection problem with legal consequences. The security team must own the detection; HR handles the response.
- **Access hygiene neglect:** Many tech companies give all developers access to all repositories "for collaboration." This dramatically increases the blast radius of any insider event. Need-to-know access controls for code repositories are essential.

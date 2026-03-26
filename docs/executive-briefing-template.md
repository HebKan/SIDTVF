# Executive Briefing Template

> **Purpose:** Use this template to prepare a 10–15 minute briefing on your insider threat program for the CISO, security committee, or board. Replace all `[bracketed]` sections with your organization's actual data.
>
> **Recommended frequency:** Quarterly

---

## Slide 1 — Program Summary

**Insider Threat Program — Quarterly Status**
*[Organization Name] | [Quarter] [Year]*

| Metric | Current | Target | Status |
|--------|---------|--------|--------|
| Program Maturity Level | [1–5] | [Target] | [On Track / At Risk / Behind] |
| Detection Coverage (KPI-01) | [X]% of [N] scenarios | ≥80% | [Status] |
| Validation Pass Rate (KPI-02) | [X]% | ≥90% | [Status] |
| Mean Time to Detect (KPI-03) | [X hours] | <24h | [Status] |
| False Positive Rate (KPI-04) | [X]:1 FP:TP | <10:1 | [Status] |

**Program Health:** [GREEN / YELLOW / RED]

---

## Slide 2 — What We're Protecting Against

**Our three highest-priority insider threat scenarios:**

| Priority | Scenario | Why It Matters to Us | Detection Status |
|----------|----------|---------------------|-----------------|
| 1 | [e.g., DE-001 — Cloud Storage Upload] | [e.g., Our IP is in SharePoint; 3 incidents industry-wide in Q3] | [Deployed / Planned] |
| 2 | [e.g., PM-001 — Privilege Escalation] | [e.g., We have 45 privileged admin accounts; limited monitoring] | [Deployed / Planned] |
| 3 | [e.g., AC-001 — Account Compromise] | [e.g., AiTM phishing campaigns targeting our sector] | [Deployed / Planned] |

**Selection basis:** Scenarios selected using SIDTVF prioritization criteria: [industry vertical starter pack / maturity-based recommendation / risk assessment].

---

## Slide 3 — What We Detected This Quarter

**Alert Summary**

| Period | Total Alerts | Investigated | True Positives | Policy Violations | False Positives |
|--------|-------------|-------------|----------------|------------------|----------------|
| This Quarter | [N] | [N] | [N] | [N] | [N] |
| Last Quarter | [N] | [N] | [N] | [N] | [N] |
| Change | [▲/▼] | [▲/▼] | [▲/▼] | [▲/▼] | [▲/▼] |

**Notable findings this quarter:**
- [Finding 1 — anonymized/sanitized: e.g., "One confirmed data staging event detected; contained before exfiltration. Employee in offboarding process."]
- [Finding 2]
- [Finding 3 — if none: "No confirmed insider threat incidents this quarter. [N] potential policy violations referred to HR."]

---

## Slide 4 — Program Gaps and Risks

**Top 3 gaps that increase our risk exposure:**

**Gap 1: [Gap Name]**
- *What we're missing:* [e.g., "No detection rule for email auto-forwarding. DE-002 scenario has no deployed Sigma rule."]
- *Risk if unaddressed:* [e.g., "Email forwarding is the second most common insider exfiltration vector per industry data. Our log data is available; rule has not been written."]
- *Effort to close:* [Low / Medium / High] | *Timeline:* [e.g., "2 weeks with existing staff"]

**Gap 2: [Gap Name]**
- *What we're missing:* [e.g., "No CASB deployed — AI tool data leakage (DE-003) is undetectable"]
- *Risk if unaddressed:* [e.g., "Employees routinely using ChatGPT for work. No visibility into what data is being shared."]
- *Effort to close:* [Medium] | *Resource required:* [e.g., "$45K/year CASB license + 2 weeks implementation"]

**Gap 3: [Gap Name]**
- *What we're missing:*
- *Risk if unaddressed:*
- *Effort to close:*

---

## Slide 5 — Roadmap to Next Maturity Level

**Current maturity:** Level [X] — [Name]
**Target maturity:** Level [X+1] — [Name]
**Estimated timeline:** [e.g., Q2 2026]

**Required actions to reach Level [X+1]:**

| Action | Owner | Timeline | Dependency |
|--------|-------|----------|-----------|
| [e.g., Deploy DE-002 detection rule] | [Detection Engineering] | [4 weeks] | [Exchange audit logging confirmed] |
| [e.g., Complete validation testing for DE-001] | [SOC] | [2 weeks] | [TC-DE-001 scheduled] |
| [e.g., Run first tabletop exercise] | [IR Lead] | [6 weeks] | [Playbook review complete] |
| [e.g., Implement HR data feed for departing employee watchlist] | [HR + SOC] | [8 weeks] | [HR system API access] |

---

## Slide 6 — Resource Request (if applicable)

**To close Priority Gap [N] and reach Level [X+1], we are requesting:**

| Item | Cost | Justification |
|------|------|---------------|
| [e.g., CASB license (300 users)] | [$X] | [Enables DE-003 AI tool detection and enhances DE-001 coverage] |
| [e.g., 0.5 FTE Detection Engineer] | [$X] | [To build out artifact coverage per the Coverage Matrix priority queue] |
| [e.g., Legal review — AI monitoring policy] | [$X] | [Required before deploying CASB/HTTPS inspection per legal-privacy.md guidance] |

**ROI context:** The median insider threat incident costs $[X] per incident ([cite source, e.g., Ponemon 2023 Cost of Insider Threats]). Investment in detection reduces mean time to detect and contain, directly reducing cost per incident.

---

## Slide 7 — Next Quarter Plan

**Q[N+1] Commitments:**

- [ ] Deploy detection rule for [scenario]
- [ ] Complete validation testing for [scenario]
- [ ] Run tabletop exercise on [date]
- [ ] Achieve Maturity Level [X+1] by [date]
- [ ] Reduce false positive rate to < [target]

**Date of next briefing:** [Date]
**Program owner:** [Name, Title]

---

## Appendix: Definitions

| Term | Definition |
|------|-----------|
| Maturity Level | 1–5 scale from the SIDTVF Maturity Model; see docs/maturity-model.md |
| Detection Coverage Rate | % of active scenarios with at least one deployed detection rule |
| True Positive | Alert confirmed as matching an insider threat scenario behavior |
| Policy Violation | Alert confirmed as a rule violation but not criminal/termination-worthy |
| False Positive | Alert confirmed as legitimate behavior; no policy violation |

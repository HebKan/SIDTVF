# SIDTVF Program KPIs and Measurement Framework

> **Document Version:** 1.0 | **Status:** Active | **Last Updated:** 2026-03-25

---

## Overview

Measuring the effectiveness of an insider threat program is essential for justifying investment, identifying gaps, and demonstrating value to leadership. SIDTVF defines eight core KPIs that are measurable, directly tied to program activities, and actionable when they fall short of target.

KPIs should be reviewed quarterly and reported to security leadership. An annual summary should accompany the maturity model assessment (see [docs/maturity-model.md](../docs/maturity-model.md)).

---

## KPI Summary Table

| KPI ID | Name | Formula | Target | Measurement Frequency |
|--------|------|---------|--------|----------------------|
| KPI-01 | Detection Coverage Rate | (Scenarios with ≥1 validated detection ÷ Total active scenarios) × 100 | ≥ 80% | Quarterly |
| KPI-02 | Validation Pass Rate | (Test cases PASS ÷ Total test cases executed) × 100 | ≥ 90% | Quarterly |
| KPI-03 | Mean Time to Detect (MTTD) | Mean(time from behavior onset to alert generation) across confirmed incidents | < 24 hours | Per incident + quarterly average |
| KPI-04 | False Positive Rate | Total alerts ÷ True positive alerts | < 10:1 | Monthly |
| KPI-05 | Scenario Currency Rate | (Scenarios reviewed or updated in past 12 months ÷ Total active scenarios) × 100 | 100% | Annually |
| KPI-06 | Mean Time to Respond (MTTR) | Mean(time from alert generation to containment decision) across responded incidents | < 4 hours | Per incident + quarterly average |
| KPI-07 | Hunt Hypothesis Conversion Rate | (Hypotheses resulting in findings ÷ Total hypotheses executed) × 100 | Track as baseline; no fixed target | Quarterly |
| KPI-08 | Control Gap Closure Rate | (Gaps addressed within 90 days ÷ Total gaps identified) × 100 | ≥ 70% | Quarterly |

---

## KPI Definitions

---

### KPI-01 — Detection Coverage Rate

**Definition:** The percentage of active SIDTVF scenarios for which at least one validated detection rule exists and has passed its validation test case.

**Formula:**
```
Detection Coverage Rate = (Scenarios with ≥ 1 validated detection / Total active scenarios) × 100
```

**What it measures:** How comprehensively the program's scenario library is backed by working detections. A scenario with an unvalidated or untested detection does not count toward coverage.

**Target:** ≥ 80%

**Interpretation:**
- **< 50%:** Critical gap — most threats have no validated detection. Prioritize detection development.
- **50–79%:** Developing — coverage exists for major scenarios but gaps remain. Assign coverage to the gap scenarios.
- **80–94%:** Healthy — strong coverage with some known gaps. Review gap scenarios for detection feasibility.
- **≥ 95%:** Optimizing — excellent coverage. Focus on maintaining currency and reducing false positive rates.

**Data source:** Scenario registry (count of Active scenarios) + Validation test case results (count of PASS outcomes linked to each scenario)

**Measurement instructions:**
1. List all scenarios with `Status: Active`
2. For each, check whether a linked detection exists with a `Status: Active` tag
3. For each active detection, check whether the linked test case has at least one `Result: PASS` entry
4. Count scenarios meeting both conditions; divide by total active scenarios

---

### KPI-02 — Validation Pass Rate

**Definition:** The percentage of validation test case executions that resulted in a PASS (the detection fired correctly with the expected fields within the expected time).

**Formula:**
```
Validation Pass Rate = (Test case executions with Result: PASS / Total test case executions) × 100
```

**What it measures:** The reliability of existing detection rules. High validation pass rates indicate that detections fire consistently under test conditions. Low rates indicate that rules are misconfigured, data sources are incomplete, or the test case is out of date.

**Target:** ≥ 90%

**Interpretation:**
- **< 70%:** Critical — most detection rules are not working as designed. Pause new scenario development and focus on fixing existing detections.
- **70–89%:** Concerning — significant failure rate. Investigate root causes systematically.
- **90–95%:** Target range — acceptable with active gap remediation for failures.
- **> 95%:** Excellent — maintain through regular re-validation cycles.

**Data source:** Validation test case files (`validation/test-cases/*.md`) — count of PASS vs. FAIL/PARTIAL results across all recorded executions

**Measurement instructions:**
1. Open all validation test case files
2. Count total executions (one row per Hunt Cycle entry in the results log)
3. Count executions with `Result: PASS`
4. Apply formula

---

### KPI-03 — Mean Time to Detect (MTTD)

**Definition:** The average elapsed time between when an insider's threat behavior begins and when a security alert is generated or the behavior is otherwise identified.

**Formula:**
```
MTTD = Mean(Alert Generation Time - Behavior Start Time) across all confirmed incidents
```

**What it measures:** How quickly your detection capabilities surface insider threat activity after it begins. Long MTTD values indicate that behaviors are persisting undetected — allowing greater data loss, damage, or financial harm.

**Target:** < 24 hours (for detected incidents)

**Interpretation:**
- **> 72 hours:** Critical — significant exposure window. Exfiltration and sabotage can be substantially completed in 72 hours.
- **24–72 hours:** Concerning — behaviors are persisting longer than necessary. Improve real-time detection rules and alert latency.
- **< 24 hours:** Target — good detection speed.
- **< 4 hours:** Excellent — near-real-time detection capability.

**Data source:** Incident records — requires documented `behavior start time` (from forensic timeline) and `alert generation time` (from SIEM alert timestamp)

**Note:** MTTD can only be accurately measured for incidents where forensic analysis establishes the behavior start time. For incidents discovered through hunting (no prior alert), record the hunt finding date as the detection time and note that the metric reflects hunt-based detection.

---

### KPI-04 — False Positive Rate

**Definition:** The ratio of alerts that do not represent genuine insider threat activity (false positives) to alerts that do (true positives).

**Formula:**
```
False Positive Rate = Total alerts generated / True positive alerts
```

**What it measures:** The signal-to-noise ratio of your detection environment. A high false positive rate degrades analyst trust in alerts, leads to alert fatigue, and ultimately causes real threats to be missed.

**Target:** < 10:1 (fewer than 10 false positive alerts per true positive)

**Interpretation:**
- **> 100:1:** Critical — almost all alerts are noise. Analysts will begin ignoring the alert queue. Immediately tune or disable the highest-noise rules.
- **50:1 – 100:1:** Severe — significant tuning required. Identify the 20% of rules generating 80% of false positives.
- **10:1 – 50:1:** Concerning — acceptable only if alert volume is low (< 20 alerts/week). Otherwise tune.
- **< 10:1:** Target — manageable noise level for a dedicated team.
- **< 5:1:** Excellent — high confidence alerts that analysts can act on quickly.

**Data source:** SIEM alert queue — count of alerts closed as false positive vs. alerts confirmed as true positive, per detection rule

**Note:** Track false positive rate per individual rule, not just overall. A single poorly tuned rule can mask the signal quality of the rest of the ruleset.

---

### KPI-05 — Scenario Currency Rate

**Definition:** The percentage of active scenarios that have been reviewed and confirmed (or updated) within the past 12 months.

**Formula:**
```
Scenario Currency Rate = (Active scenarios with Last Updated ≤ 12 months ago / Total active scenarios) × 100
```

**What it measures:** Whether the threat scenario library remains relevant to current attacker behavior, organizational changes, and the evolving technology environment. Stale scenarios may reference outdated attack techniques, retired systems, or controls that have changed.

**Target:** 100%

**Interpretation:**
- **< 80%:** Concerning — a significant portion of the program is operating on outdated threat assumptions.
- **80–99%:** Developing — almost current but overdue scenarios exist. Prioritize review by severity.
- **100%:** Target — all scenarios have been reviewed within the last year.

**Data source:** Scenario files (`scenarios/**/*.md`) — `Last Updated` field vs. current date

**Note:** "Reviewed" means a human reviewer confirmed the scenario is still accurate, relevant, and well-linked to current detections. Simply incrementing the date without review does not count.

---

### KPI-06 — Mean Time to Respond (MTTR)

**Definition:** The average elapsed time between when a detection alert fires (or a hunt finding is escalated) and when a containment decision is made and executed by the response team.

**Formula:**
```
MTTR = Mean(Containment Decision Time - Alert Time) across all confirmed incidents
```

**What it measures:** The effectiveness and speed of the incident response process. Long MTTR allows the insider to continue harmful activity after detection.

**Target:** < 4 hours (initial containment decision)

**Interpretation:**
- **> 24 hours:** Critical — too slow for meaningful containment. Review response process, on-call coverage, and Legal/HR decision-making speed.
- **4–24 hours:** Developing — acceptable for lower-severity incidents but too slow for Critical or High incidents.
- **< 4 hours:** Target for all confirmed insider threat incidents.
- **< 1 hour:** Excellent — rapid response capability.

**Data source:** Incident records — `alert time` (from SIEM alert) vs. `containment decision time` (from incident ticket)

**Note:** Distinguish between time to containment decision and time to full remediation. MTTR measures the decision point, not complete remediation, because containment timing is often under Security and Legal control while remediation timing depends on IT operations.

---

### KPI-07 — Hunt Hypothesis Conversion Rate

**Definition:** The percentage of executed threat hunting hypotheses that result in a confirmed or suspicious finding (as opposed to "clean" results).

**Formula:**
```
Hunt Conversion Rate = (Hypotheses resulting in Suspicious or True Threat Activity findings / Total hypotheses executed) × 100
```

**What it measures:** The productivity and effectiveness of the threat hunting program. A low conversion rate over time may indicate that hypotheses are not well-targeted to the actual threat landscape, or that existing detections are already catching most activity.

**Target:** No fixed target — track as a baseline over time. A rate of 10–20% is common in mature programs.

**Interpretation:** This metric should be trended rather than measured against an absolute target. An increasing conversion rate may indicate a genuine increase in insider threat activity, improved hypothesis quality, or gaps in detection coverage (threats that were previously missed). A decreasing rate may indicate improved detection coverage (fewer threats evading detection to be found in hunts) or declining hypothesis quality.

**Data source:** Hunt hypothesis files (`hunting/hypotheses/*.md`) — count of hypothesis executions with `Finding Classification: Suspicious` or `True Threat Activity` vs. `Clean`

---

### KPI-08 — Control Gap Closure Rate

**Definition:** The percentage of control gaps identified through validation testing that are addressed (detection implemented, control fixed, or risk accepted) within 90 days of identification.

**Formula:**
```
Gap Closure Rate = (Gaps addressed within 90 days / Total gaps identified) × 100
```

**What it measures:** How effectively the program converts detection failures into improved controls. A low closure rate indicates that gaps are being identified but not fixed — which means the feedback loop (Layer 6) is broken.

**Target:** ≥ 70%

**Interpretation:**
- **< 50%:** Critical — gaps are accumulating faster than they are being addressed. The detection environment is deteriorating.
- **50–69%:** Concerning — many gaps remain open beyond 90 days. Review resourcing and prioritization.
- **≥ 70%:** Target — most gaps are addressed within the quarter.
- **≥ 90%:** Excellent — rapid and consistent gap closure.

**Data source:** Validation test case files (gap analysis sections) — track gap identification date and closure date; Detection Engineering ticket system

---

## Reporting Template

Use this template for quarterly KPI reporting to security leadership.

```
SIDTVF Program KPI Report
Reporting Period: [Quarter] [Year]
Prepared By: [Name/Team]
Date: [Date]

┌────────────────────────────────────────────────────────────────────┐
│ KPI                          │ Target │ This Quarter │ Last Quarter │
├────────────────────────────────────────────────────────────────────┤
│ KPI-01 Detection Coverage    │ ≥ 80%  │ [Value]      │ [Value]      │
│ KPI-02 Validation Pass Rate  │ ≥ 90%  │ [Value]      │ [Value]      │
│ KPI-03 MTTD                  │ < 24h  │ [Value]      │ [Value]      │
│ KPI-04 False Positive Rate   │ < 10:1 │ [Value]      │ [Value]      │
│ KPI-05 Scenario Currency     │ 100%   │ [Value]      │ [Value]      │
│ KPI-06 MTTR                  │ < 4h   │ [Value]      │ [Value]      │
│ KPI-07 Hunt Conversion Rate  │ Track  │ [Value]      │ [Value]      │
│ KPI-08 Gap Closure Rate      │ ≥ 70%  │ [Value]      │ [Value]      │
└────────────────────────────────────────────────────────────────────┘

KEY FINDINGS:
1. [Most important positive finding]
2. [Most important area of concern]
3. [Priority recommendation for next quarter]

INCIDENTS THIS QUARTER:
- Confirmed insider threat incidents: [Count]
- Hunting findings escalated to IR: [Count]
- Near-misses identified and remediated: [Count]

PROGRAM CHANGES THIS QUARTER:
- New scenarios added: [List with IDs]
- Scenarios updated: [List with IDs]
- New detections deployed: [List with IDs]
- Validation cycles completed: [Count]
- Hunt cycles completed: [Count]
```

---

## Maturity Level vs. Expected KPI Targets

As programs mature, KPI targets should be adjusted. The table below shows typical achievable targets at each maturity level.

| KPI | Level 1 | Level 2 | Level 3 | Level 4 | Level 5 |
|-----|---------|---------|---------|---------|---------|
| Detection Coverage | Not tracked | < 30% | 50–80% | ≥ 80% | ≥ 95% |
| Validation Pass Rate | Not tracked | Not tracked | 70–85% | ≥ 90% | ≥ 95% |
| MTTD | Unknown | > 7 days | 24–72 hours | < 24 hours | < 4 hours |
| False Positive Rate | Not tracked | > 100:1 | 20:1–50:1 | < 10:1 | < 5:1 |
| Scenario Currency | Not tracked | Not tracked | 80–100% | 100% | 100% |
| MTTR | Unknown | > 24 hours | 4–24 hours | < 4 hours | < 1 hour |
| Hunt Conversion | Not tracked | Not tracked | Track only | Track + trend | Benchmark externally |
| Gap Closure | Not tracked | Not tracked | 50–70% | ≥ 70% | ≥ 90% |

---

## Changelog

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-03-25 | Initial KPI definitions — eight core metrics |

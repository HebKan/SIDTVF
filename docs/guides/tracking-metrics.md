# Guide: Tracking Metrics and KPIs

SIDTVF defines a set of KPIs that measure program health, detection effectiveness, and operational performance. This guide explains how to collect, calculate, and report each metric.

---

## Full KPI Definitions

All KPI formulas, data sources, and targets are in `metrics/kpis.md`. This guide focuses on how to operationalize them — what to collect, when to collect it, and what to do with the results.

---

## Core Metrics and Data Sources

### Detection Coverage Rate
**What:** Percentage of Active scenarios that have at least one validated detection rule deployed.

**How to calculate:**
```
Active scenarios with a validated detection
─────────────────────────────────────────── × 100
Total Active scenarios
```

**Where to get the data:** `docs/coverage-matrix.md` — the coverage matrix tracks which scenarios have which artifacts. Count rows where the "Detection (Sigma/KQL/SPL)" column shows a validated artifact.

**Target:** ≥ 80%. Below 60% means significant coverage gaps requiring prioritized detection builds.

---

### Validation Pass Rate
**What:** Percentage of test cases where the detection fired as expected during the most recent validation run.

**How to calculate:**
```
Test cases with result = Pass
───────────────────────────── × 100
Total test cases executed
```

**Where to get the data:** Your validation test run records (linked from each test case file in `validation/test-cases/`).

**Target:** ≥ 90%. A pass rate below 80% means your detection rules are either misconfigured or your log sources are unreliable.

---

### Mean Time to Detect (MTTD)
**What:** Average time from when an insider behavior begins (per the scenario's attack phase definition) to when an alert fires.

**How to calculate:** This requires simulation test data. Record the time of the first simulated action and the time the alert fired. Average across runs.

**Where to get the data:** Simulation test run logs (`simulation/tests/SIM-[SCENARIO_ID].md` — run log section).

**Target:** < 24 hours for Critical scenarios; < 72 hours for High.

---

### False Positive Rate
**What:** Ratio of false positive alerts to true positive alerts for each scenario's detection rules.

**How to calculate:**
```
Alerts closed as False Positive (30-day window)
──────────────────────────────────────────────── : 1
Alerts confirmed as True Positive
```

**Where to get the data:** Your SIEM/ticketing system — filter to alert rules tagged with SIDTVF scenario IDs.

**Target:** < 10:1 (less than 10 false positives per confirmed true positive). High false positive rates cause alert fatigue and missed detections.

---

### Control Gap Closure Rate
**What:** Percentage of detection gaps identified in validation testing that have been remediated within 90 days.

**How to calculate:**
```
Gaps identified ≥ 90 days ago that are now closed
─────────────────────────────────────────────────── × 100
Total gaps identified ≥ 90 days ago
```

**Where to get the data:** Your detection improvement backlog and the coverage matrix.

**Target:** ≥ 70%. Gaps that stay open longer than 90 days indicate a resourcing or prioritization problem.

---

### Scenario Currency
**What:** Percentage of Active scenarios that have been reviewed and updated within the last 12 months.

**How to calculate:** Count scenario files where `Last Updated` is within 12 months.

**Where to get the data:** `scenarios/index.yaml` — check the `last_updated` field for each Active scenario.

**Target:** 100%. Stale scenarios may reference outdated attack methods, log sources, or MITRE mappings.

---

## Reporting Cadence

| Metric | Collection frequency | Reporting audience |
|--------|---------------------|-------------------|
| Detection Coverage Rate | Monthly | Security leadership, program manager |
| Validation Pass Rate | Per validation cycle (quarterly minimum) | Detection engineering, SOC manager |
| Mean Time to Detect | Per simulation run | Detection engineering, IR lead |
| False Positive Rate | Monthly | SOC manager, detection engineering |
| Control Gap Closure Rate | Monthly | Program manager, security leadership |
| Scenario Currency | Quarterly | Framework owner, scenario authors |

---

## Presenting Metrics to Leadership

For quarterly CISO or board reporting, use the executive briefing template: `docs/executive-briefing-template.md`

Key principles for leadership reporting:
- Lead with **program health** (coverage rate, pass rate) before operational details
- Frame gaps as **prioritized investments**, not failures
- Show **trend** — is coverage improving, stable, or declining?
- Connect metrics to **business risk** using scenario severity and compliance mapping

A Detection Coverage Rate of 65% is meaningless to a board. "We can detect 8 of our 11 highest-risk insider threat scenarios; the 3 gaps are in IP theft, sabotage, and vendor abuse, which represent our highest-severity unmitigated risks" is actionable.

---

## Common Measurement Mistakes

- **Counting deployed rules as validated** — a rule that has never been tested against a known behavior is not validated. Only count rules with a passed TC or SIM result.
- **Measuring MTTD only in lab conditions** — lab detection times are faster than production. Use simulation tests in near-production environments for realistic MTTD figures.
- **Reporting coverage without separating severity tiers** — 80% coverage across all scenarios but 40% coverage for Critical scenarios is a problem. Report coverage by severity tier.

---

## Related

- KPI definitions and formulas: [metrics/kpis.md](../../metrics/kpis.md)
- Coverage matrix: [docs/coverage-matrix.md](../coverage-matrix.md)
- Executive briefing template: [docs/executive-briefing-template.md](../executive-briefing-template.md)
- Maturity model: [docs/maturity-model.md](../maturity-model.md)

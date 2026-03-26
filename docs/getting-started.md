# Getting Started with SIDTVF

> This guide walks you through your first 90 days with SIDTVF.
> Start here before reading any other document in this repository.

---

## Before You Begin: Two Questions

**Question 1 — What is your current program maturity?**
Take the 10-minute self-assessment in [docs/maturity-model.md](maturity-model.md) before doing anything else. Your maturity score determines which activities are realistic right now and which will require prerequisite work.

**Question 2 — Are you legally authorized to monitor?**
Review [docs/legal-privacy.md](legal-privacy.md) for your jurisdiction before deploying any detection. In most countries, employee monitoring requires documented policy, employee notice, and legal review. Do not deploy monitoring without completing this step.

---

## Day 1 (2–3 hours)

### 1. Complete the Maturity Assessment
Read [docs/maturity-model.md](maturity-model.md) and answer the 48-question self-assessment. Your score is a number from 1–5. Write it down.

**Typical starting scores:**
- **Score 1:** No formal monitoring. This guide starts at Step 2 below.
- **Score 2:** Basic tools deployed (SIEM, proxy logs) but no insider-specific program. Start at Step 3.
- **Score 3+:** Formal program exists. Use the Priority Build Queue in [docs/coverage-matrix.md](coverage-matrix.md) to identify gaps.

### 2. Read the Legal Landscape (30 minutes)
Open [docs/legal-privacy.md](legal-privacy.md). Read Section 1 (Universal Principles) and the section for your primary jurisdiction. Note any requirements that apply to your organization before you proceed.

**Key action:** Identify whether you need to update your employee monitoring notice before deploying any new detections. Many jurisdictions require this.

### 3. Choose Your First Two Scenarios
Based on your maturity score, select your starting scenarios:

| Your Maturity | Recommended Starting Scenarios |
|--------------|-------------------------------|
| 1 (Initial) | DE-001 (Cloud Storage Upload), DE-002 (Email Forwarding) — basic logging required |
| 2 (Developing) | DE-001, PM-001 — you have the logs; now write the rules |
| 3+ (Defined) | Review [docs/coverage-matrix.md](coverage-matrix.md) and fill your largest gaps |

For each selected scenario, read the full scenario file (e.g., `scenarios/data-exfiltration/DE-001-cloud-storage-upload.md`).

---

## Week 1 (4–8 hours)

### 4. Run a Validation Test Before Deploying Detections
Before writing a single detection rule, run the validation test for your chosen scenario. This tells you whether you even have the log data needed.

**For DE-001:** Run [validation/test-cases/TC-DE-001.md](../validation/test-cases/TC-DE-001.md)
- If the test **passes**: you have the log data. Proceed to Step 5.
- If the test **fails**: you have a log gap. Fix the logging before writing detection rules.

This is the most important step most organizations skip. Detection rules written against logs that don't exist will never fire.

### 5. Deploy Your First Detection
Use the detection files for your chosen scenario:
- **Sigma** (platform-agnostic): `detections/sigma/DE-001-cloud-storage-upload.yml` — compile to your SIEM
- **KQL** (Microsoft Sentinel): `detections/queries/DE-001-kql.md` — deploy Query 1 first
- **SPL** (Splunk): `detections/queries/DE-001-spl.md` — deploy Query 1 first

Deploy one query first. Do not deploy all variants simultaneously — you won't be able to tune them independently.

### 6. Read the Response Playbook
Before your detection fires, make sure you know what to do when it does. Read the playbook for your scenario:
- **DE-001:** [playbooks/examples/PB-DE-001-cloud-storage-exfil.md](../playbooks/examples/PB-DE-001-cloud-storage-exfil.md)

Share the playbook with your SOC, IR, Legal, and HR teams. Confirm who owns each phase.

---

## Month 1 (ongoing)

### 7. Run a Threat Hunt
Once your detection is deployed, conduct a proactive hunt for historical evidence of the behavior:
- **DE-001 Hunt:** [hunting/hypotheses/HYP-DE-001.md](../hunting/hypotheses/HYP-DE-001.md)
- **DE-002 Hunt:** [hunting/hypotheses/HYP-DE-002.md](../hunting/hypotheses/HYP-DE-002.md)

Hunting often surfaces incidents that pre-date your detection deployment. It also helps you tune thresholds before the detection fires on a real event.

### 8. Run a Simulation Test
After 2–3 weeks of operating your detection, run a simulation test to confirm the full pipeline works end-to-end:
- **DE-001 Simulation:** [simulation/tests/SIM-DE-001.md](../simulation/tests/SIM-DE-001.md)
- Read [simulation/authorization-checklist.md](../simulation/authorization-checklist.md) first

The simulation confirms that a real insider behaving in the scenario's pattern would actually trigger your alert.

### 9. Add Your Second Scenario
Repeat Steps 4–8 for your second scenario. Building one scenario at a time is intentional — it forces you to complete the full artifact stack before adding complexity.

### 10. Measure Your Program
Set up the KPI tracking in [metrics/kpis.md](../metrics/kpis.md). At minimum, track:
- **KPI-01:** Detection Coverage Rate (how many of your scenarios have deployed detection rules?)
- **KPI-02:** Validation Pass Rate (what % of your validation tests pass?)
- **KPI-04:** False Positive Rate (how many alerts are genuine vs. noise?)

---

## Quarter 1 (30–60 days in)

### 11. Run a Tabletop Exercise
Use [docs/tabletop-exercise-guide.md](tabletop-exercise-guide.md) to run a 2-hour cross-team tabletop using your first scenario. This is how you discover whether your playbook is actually executable — before a real incident.

Invite: SOC lead, IR lead, Legal, HR, and CISO. Use the scenario narrative as the exercise scenario.

### 12. Assess Your Maturity Again
Retake the maturity self-assessment. Most organizations move from Level 1 to Level 2 in their first quarter. If your score hasn't moved, review the Level 2 prerequisites in [docs/maturity-model.md](maturity-model.md) to identify blockers.

### 13. Present to Leadership
Use [docs/executive-briefing-template.md](executive-briefing-template.md) to prepare a 10-minute briefing for your CISO or security committee. Present:
- Your current maturity score and target
- The two scenarios you've deployed
- Your KPI baseline
- The two biggest gaps and what resources are needed to close them

---

## Common Mistakes

| Mistake | Why It's a Problem | What to Do Instead |
|---------|-------------------|--------------------|
| Writing detection rules before validating log sources | Rules will never fire; you won't know until an incident | Always run TC-XXX before writing DET-XXX |
| Deploying all detection variants simultaneously | Can't tune independently; alert fatigue risk | Deploy one query; tune it; then add more |
| Skipping the legal review | Monitoring may be unauthorized; evidence inadmissible | Read legal-privacy.md and consult counsel before deploying |
| Treating high alert volume as "working" | High FP rate wastes SOC time and causes alert fatigue | Target < 10:1 FP:TP ratio (KPI-04) |
| Selecting scenarios based on technical interest | May miss highest-risk scenarios for your industry | Use the industry starter packs in docs/starter-packs/ |
| Running simulation tests in production without authorization | Legal and compliance risk; SOC disruption | Always complete simulation/authorization-checklist.md first |

---

## Recommended Reading Order

1. This guide (you're here)
2. [docs/maturity-model.md](maturity-model.md) — take the self-assessment
3. [docs/legal-privacy.md](legal-privacy.md) — understand your legal context
4. Your first scenario file (e.g., `scenarios/data-exfiltration/DE-001-cloud-storage-upload.md`)
5. The corresponding validation test case
6. The corresponding detection files
7. The response playbook
8. The hunt hypothesis
9. The simulation test
10. [docs/actor-archetypes.md](actor-archetypes.md) — deepens your understanding of who you're detecting

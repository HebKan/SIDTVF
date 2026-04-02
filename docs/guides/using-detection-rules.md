# Guide: Using Detection Rules

SIDTVF provides three detection artifact types for each scenario: Sigma rules (platform-agnostic), KQL queries (Microsoft Sentinel), and SPL queries (Splunk). This guide covers how to deploy and tune each.

---

## Detection Artifact Types

| Type | File location | Use when |
|------|--------------|---------|
| Sigma rules | `detections/sigma/[SCENARIO_ID]-*.yml` | You want to convert to any SIEM or use as a platform-neutral specification |
| KQL queries | `detections/queries/[SCENARIO_ID]-kql.md` | Your SIEM is Microsoft Sentinel |
| SPL queries | `detections/queries/[SCENARIO_ID]-spl.md` | Your SIEM is Splunk |

Each detection file names the required data source at the top. **Check this before attempting to deploy.** A detection without its required log source will either fail silently or never fire.

---

## Deploying a Sigma Rule

Sigma rules require conversion to your SIEM's native query language before deployment.

1. **Validate the rule** — Open the `.yml` file and confirm the `logsource` and `detection` fields match log sources you have ingested.
2. **Convert** — Use [sigma-cli](https://github.com/SigmaHQ/sigma-cli) or a tool like uncoder.io:
   ```
   sigma convert -t splunk detections/sigma/DE-001-cloud-storage-upload.yml
   sigma convert -t sentinel detections/sigma/DE-001-cloud-storage-upload.yml
   ```
3. **Review converted output** — The conversion is a starting point. Review field names against your actual log schema before deploying.
4. **Deploy as a scheduled alert** — Set the query interval to the recommended schedule in the rule's `status` and `level` fields. Critical rules should run every 5–15 minutes; Medium/Low can run hourly.
5. **Set alert actions** — Route to your ticketing system or SOAR workflow. See [docs/soar-integration.md](../soar-integration.md) for SOAR connection guidance.

---

## Deploying a KQL Query (Microsoft Sentinel)

1. Open the `.md` file in `detections/queries/`. Each query section has a comment block explaining what it detects and what data source it requires.
2. Copy the query into Sentinel's **Logs** blade and run it against your data to confirm it returns results (or expected empty results in a clean environment).
3. Create a **Scheduled Analytics Rule** with the query. Recommended settings are noted in the query file comments.
4. Map the rule to the MITRE ATT&CK techniques listed in the scenario card's detection section.
5. Link the alert's incident creation to the matching triage card (`detections/triage-cards/TRIAGE-[SCENARIO_ID].md`).

---

## Deploying an SPL Query (Splunk)

1. Open the `.md` file in `detections/queries/`. Each query includes inline comments explaining field names and thresholds.
2. Run the query in Splunk Search to verify it executes against your data. Adjust index names and sourcetype values to match your environment.
3. Save as a **Saved Search** or **Correlation Search** (Splunk ES). Set the schedule and suppression window per the query file recommendations.
4. Create a notable event rule and map it to the relevant MITRE technique.

---

## Required Data Sources

Before deploying any detection, verify you have the required data source ingested and queryable:

| Scenario | Primary data source required |
|----------|------------------------------|
| DE-001 | Proxy/web gateway logs with URL and bytes transferred |
| DE-002 | Exchange/M365 audit log (mailbox rule creation events) |
| DE-003 | CASB session logs or proxy logs for AI tool domains |
| SB-001 | SQL Server Audit or database activity monitoring |
| UA-001 | Authentication logs (Entra ID, AD, or equivalent) |
| PM-001 | Privileged access logs, directory change logs |

---

## Tuning Guidance

Detection rules ship with **false positive** sections. Read these before deploying to production.

Common tuning approaches:
- **Allowlist known-good destinations** — add your corporate OneDrive tenant or approved SaaS apps to exclusion lists
- **Threshold adjustments** — volume-based rules use conservative defaults; adjust to your environment's baseline
- **Time windows** — after-hours rules define business hours broadly; narrow to your actual work schedule
- **User scope** — some rules should be scoped to specific populations (e.g., only contractors, only finance team)

After tuning, re-run the linked validation test case (`validation/test-cases/TC-[SCENARIO_ID].md`) to confirm the detection still fires for the intended behavior.

---

## Related

- SOC triage after alert fires: [docs/guides/using-triage-cards.md](using-triage-cards.md)
- Validating that a detection works: [docs/guides/running-validation-tests.md](running-validation-tests.md)
- SOAR automation: [docs/soar-integration.md](../soar-integration.md)
- Detection authoring process: [docs/processes/detection-development-guide.md](../processes/detection-development-guide.md)

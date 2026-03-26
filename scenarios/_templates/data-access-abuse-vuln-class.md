# Scenario Template: Data Access Abuse Vulnerability Class

> Pre-populated for scenarios where the threat behavior centers on accessing data
> beyond the actor's authorized scope — including querying sensitive databases,
> accessing files outside job function, bulk downloading, or browsing restricted repositories.
> Copy to the appropriate category directory, rename, and fill in `[PLACEHOLDER]` fields.

---

## Scenario Card

| Field | Value |
|-------|-------|
| **Scenario ID** | [e.g., PM-002] |
| **Name** | [Short name] |
| **Version** | 1.0 |
| **Status** | Draft |
| **Created** | [YYYY-MM-DD] |
| **Last Updated** | [YYYY-MM-DD] |
| **Author** | [Name] |
| **Category** | [Category] |
| **Actor Archetype(s)** | [MAL / NEG / COM / TP] |
| **Severity** | [Critical / High / Medium / Low] |
| **Minimum Maturity Level** | 3 |
| **Vulnerability Class** | Data Access Abuse |
| **Data Repository Type** | [Database / File Share / Document Management / Data Warehouse / Cloud Storage / API] |
| **Data Classification** | [PHI / PII / PCI / Confidential / Trade Secret / Financial] |

---

## 1. Description

[Describe the data access abuse scenario. Specify: what repository is being accessed, what data classification is involved, how the access exceeds the actor's legitimate authorization, and what the intended use of the data is.]

---

## 2. Narrative Case Study

[3–6 paragraphs. Include how the actor accessed the data, what technical or procedural gaps enabled the access, how much data was involved, and what the discovery mechanism was.]

---

## 3. Actor Profile

| Field | Value |
|-------|-------|
| **Archetype** | [MAL / NEG / COM / TP] |
| **Typical Role** | [Roles with broad read access: analyst, developer, auditor, report writer, data engineer] |
| **Access Level** | [Standard User — but with broad read permissions due to over-provisioning] |
| **Technical Sophistication** | [Low to Medium — data access often requires only knowledge of the query interface] |
| **Motivation** | [Financial gain / curiosity / competitive intelligence / revenge / negligence] |
| **Data Knowledge Used** | [e.g., "Knows which database tables contain the most valuable customer records"; "Aware that the data warehouse has no query volume monitoring"] |

---

## 4. Attack Phases

### Phase 1 — Target Data Identification
**Description:** Actor identifies the repository and tables/directories containing the data of interest.
**Observable Actions:**
- Schema enumeration queries (SHOW TABLES, SELECT * FROM INFORMATION_SCHEMA)
- Directory or SharePoint site browsing outside normal work scope
- Search queries for sensitive keywords in a document management system
- Access to data catalog or metadata system to locate high-value datasets

### Phase 2 — Data Query or Access
**Description:** Actor queries or directly accesses the target data, often in volumes significantly beyond their role requirements.
**Observable Actions:**
- Database query returning a large number of rows from a sensitive table
- SELECT * FROM [sensitive_table] with no WHERE clause (bulk harvest)
- SharePoint or file share: high-volume file open or download events
- API calls returning large paginated datasets from a sensitive endpoint
- Data warehouse query exporting full customer or employee tables

### Phase 3 — Local Staging or Export
**Description:** Actor stages the queried data locally or exports it through available channels.
**Observable Actions:**
- Query results saved to local CSV or spreadsheet file
- Report export function used to generate a file containing sensitive records
- Data warehouse "Download Results" function used on a large query
- File containing query results moved to a personal folder or removable media

### Phase 4 — [Exfiltration]
**Description:** [How the data is removed from the organization]
**Observable Actions:**
- [Cloud upload / email attachment / USB transfer of the exported file]

---

## 5. Indicators of Behavior (IOBs)

### Technical IOBs — Data Access Specific

| IOB | Data Source | Signal Strength |
|-----|-------------|----------------|
| Database query returning > [threshold] rows from a sensitive table by a non-DBA account | Database Activity Monitoring (DAM) | High |
| SELECT * or bulk query with no filtering on a sensitive table | DAM | High |
| User queries a database table they have never queried in the past 90 days | DAM + UEBA | High |
| File access volume on a file share significantly above the user's 90-day baseline | SIEM (file audit logs) + UEBA | High |
| Report export from a BI tool containing sensitive columns | BI platform audit log (Tableau, Power BI, Looker) | High |
| Data warehouse large query result downloaded (rows > threshold or bytes > threshold) | Data warehouse audit log (Snowflake, Redshift, BigQuery) | High |
| API call returning paginated sensitive dataset with all pages retrieved | API gateway logs | Medium |
| Access to a data classification label: Confidential or Restricted by an account with no prior access to that label | DLP / file classification system | High |
| Multiple queries to the same sensitive table from the same account within a short window | DAM | Medium |

### Contextual IOBs

| IOB | Source | Notes |
|-----|--------|-------|
| User's role does not require access to the queried data category | Role documentation / RBAC policy | Signals access is over-provisioned or being abused |
| User has a pending role change or is leaving a project requiring this data access | HR / project tracking | Access may no longer be appropriate |

---

## 6. Detection Opportunities

| Phase | Detection Opportunity | Required Data Source | Confidence |
|-------|-----------------------|---------------------|-----------|
| Phase 1 | Schema enumeration queries from non-DBA account | DAM | High |
| Phase 2 | Bulk SELECT on sensitive table exceeding row threshold | DAM | High |
| Phase 2 | File access volume spike above personal baseline | SIEM + UEBA | High |
| Phase 3 | Large query result downloaded from BI or data warehouse tool | BI/DW audit log | High |
| Phase 4 | File matching sensitive data classification uploaded externally | DLP | High |

---

## 7. MITRE ATT&CK Mapping

| Tactic | Technique ID | Technique Name | Applies To |
|--------|-------------|----------------|------------|
| Discovery | T1087 | Account Discovery | Phase 1 (if schema includes account data) |
| Collection | T1213 | Data from Information Repositories | Phase 1, Phase 2 |
| Collection | T1005 | Data from Local System | Phase 3 |
| Collection | T1530 | Data from Cloud Storage Object | Phase 2 (cloud data store variant) |
| Exfiltration | T1567.002 | Exfiltration to Cloud Storage | Phase 4 |
| [Add additional techniques] | | | |

---

## 8. Legal and Privacy Considerations

- **DAM deployment:** Database Activity Monitoring tools intercept database traffic. Confirm that DAM deployment is covered by the acceptable use policy and has been reviewed for applicable wiretapping or interception laws in your jurisdiction.
- **Data minimization in DAM:** DAM tools may log query results. Ensure DAM is configured to log query metadata (query text, row count, user, timestamp) rather than full query results containing PII — logging PII in a security tool creates a secondary privacy risk.
- **PHI/HIPAA:** If the accessed data contains PHI, HIPAA breach notification analysis is required regardless of whether the actor had some degree of legitimate access.
- **RBAC documentation:** For cases to be actionable, you must be able to demonstrate that the actor's role did not authorize the queried data. Maintain up-to-date RBAC documentation.

---

## 9. Response Guidance

| Priority | Action | Owner |
|----------|--------|-------|
| 1 | Preserve DAM logs and file access logs for the affected user and data repository | IR |
| 2 | Identify full scope: which tables, how many rows, over what period | IR + Data Owner |
| 3 | Assess data classification of accessed records — determines breach obligations | IR + Legal + Data Owner |
| 4 | Restrict or revoke the data access in question (after Legal approval if employment action likely) | IAM / DBA |
| 5 | Notify Legal — assess HIPAA, GDPR, or other breach obligations | Legal / Privacy |
| 6 | Conduct access review for other users with similar over-provisioned database access | IAM + DBA |

**Required Configuration for Detection:**
- Database Activity Monitoring (DAM) must be deployed on all databases containing sensitive data categories
- BI tools must have audit logging enabled (Tableau, Power BI, Looker, Snowflake all support this natively)
- Row-count and byte-volume thresholds must be baselined per data store before alert deployment

**Linked Playbook:** [PB-XX-NNN — create using `playbooks/_template.md`]

---

## Changelog

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | [YYYY-MM-DD] | [Author] | Initial creation |

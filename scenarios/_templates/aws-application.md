# Scenario Template: Amazon Web Services (AWS)

> Pre-populated for scenarios where the threat behavior occurs within an AWS environment.
> Assumes CloudTrail is enabled organization-wide with log delivery to S3 and SIEM.
> Copy to the appropriate category directory, rename, and fill in `[PLACEHOLDER]` fields.

---

## Scenario Card

| Field | Value |
|-------|-------|
| **Scenario ID** | [e.g., DE-008] |
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
| **Attack Vector** | Application — Amazon Web Services |
| **AWS Services Involved** | [S3 / IAM / EC2 / RDS / Lambda / CloudFormation / Other] |

---

## 1. Description

[Describe the AWS-specific threat behavior. Include which AWS services are targeted, what IAM permissions make the scenario possible, and what data or infrastructure is at risk.]

---

## 2. Narrative Case Study

[3–6 paragraphs. Include which AWS APIs were called, why the activity wasn't caught by default AWS security controls (e.g., GuardDuty gaps, lack of S3 server access logging), and how CloudTrail revealed the activity.]

---

## 3. Actor Profile

| Field | Value |
|-------|-------|
| **Archetype** | [MAL / NEG / COM / TP] |
| **Typical Role** | [Cloud Engineer, DevOps, Data Engineer, SRE, Security Engineer — roles with AWS IAM permissions] |
| **Access Level** | [Developer / PowerUser / Administrator / Root (critical)] |
| **Technical Sophistication** | [Medium to High — AWS attacks require API/CLI familiarity] |
| **AWS Knowledge Used** | [e.g., "Knows S3 server access logging is separate from CloudTrail"; "Aware that certain regions have CloudTrail disabled"; "Can create IAM access keys for exfiltration staging"] |

---

## 4. Attack Phases

### Phase 1 — IAM and Environment Reconnaissance
**Description:** Insider surveys the AWS environment for valuable resources or misconfigured permissions.
**Observable Actions:**
- ListBuckets, ListObjects, DescribeInstances, DescribeDBInstances API calls from non-service accounts
- GetAccountAuthorizationDetails, ListRoles, ListPolicies (IAM enumeration)
- GetBucketPolicy, GetBucketAcl (checking S3 access controls)
- DescribeRegions queries (to identify regions with logging gaps)

### Phase 2 — Access Key or Credential Staging
**Description:** Insider creates or retrieves long-lived credentials for persistent access outside the normal session.
**Observable Actions:**
- CreateAccessKey API call (CloudTrail: iam.amazonaws.com / CreateAccessKey)
- PutUserPolicy or AttachUserPolicy to self-grant additional permissions
- AssumeRole into a higher-privileged role not normally accessed

### Phase 3 — [Data Collection or Resource Action]
**Description:** [How data is collected from AWS resources or how infrastructure is targeted]
**Observable Actions:**
- GetObject API calls on S3 buckets outside normal scope (high volume or sensitive bucket)
- RDS database snapshot creation followed by share or export
- SSM GetParameter retrieving secrets from Parameter Store
- Secrets Manager GetSecretValue for credentials outside the user's application scope

### Phase 4 — [Exfiltration or Impact]
**Description:** [How data leaves AWS or how infrastructure is harmed]
**Observable Actions:**
- PutBucketAcl / PutBucketPolicy making S3 bucket publicly accessible
- S3 CopyObject to an external AWS account
- EC2 instance snapshot shared to external AWS account ID
- Lambda function created to exfiltrate data to external endpoint

---

## 5. Indicators of Behavior (IOBs)

### Technical IOBs — AWS-Specific

| IOB | Data Source | Signal Strength |
|-----|-------------|----------------|
| CreateAccessKey called for a user that already has active access keys | CloudTrail (IAM events) | High |
| GetObject API calls on an S3 bucket outside the user's historical access scope | CloudTrail + S3 server access logs | High |
| IAM enumeration API calls (ListRoles, GetAccountAuthorizationDetails) from non-automation account | CloudTrail | High |
| PutBucketAcl changing a private bucket to public-read | CloudTrail | High |
| AssumeRole into a role not used by this identity in the past 30 days | CloudTrail (STS: AssumeRole) | High |
| S3 CopyObject to a destination bucket in a different AWS account | CloudTrail | High |
| High volume of GetObject requests from a single IAM identity in a short window | CloudTrail + S3 access logs | High |
| RDS CreateDBSnapshot without an associated change ticket | CloudTrail + change management | High |
| API calls originating from a non-AWS IP (personal workstation, VPN) via access key | CloudTrail (sourceIPAddress field) | Medium |
| API calls outside the user's established working hours and region | CloudTrail + UEBA | High |

### Contextual IOBs

| IOB | Source | Notes |
|-----|--------|-------|
| User has AWS IAM permissions far exceeding their current project scope | IAM access review | Signals hygiene gap |
| Departing employee holds active AWS access keys | HR + IAM | Revoke keys immediately at departure |

---

## 6. Detection Opportunities

| Phase | Detection Opportunity | Required Data Source | Confidence |
|-------|-----------------------|---------------------|-----------|
| Phase 1 | IAM enumeration from non-automation account | CloudTrail → SIEM | High |
| Phase 2 | CreateAccessKey for account with existing active keys | CloudTrail (IAM) | High |
| Phase 3 | High-volume GetObject from S3 bucket not in user's historical access | CloudTrail + S3 access logs | High |
| Phase 4 | PutBucketAcl or PutBucketPolicy changing bucket to public | CloudTrail | High |
| Phase 4 | CopyObject to external AWS account | CloudTrail | High |

---

## 7. MITRE ATT&CK Mapping

| Tactic | Technique ID | Technique Name | Applies To |
|--------|-------------|----------------|------------|
| Discovery | T1580 | Cloud Infrastructure Discovery | Phase 1 |
| Discovery | T1619 | Cloud Storage Object Discovery | Phase 1 |
| Privilege Escalation | T1078.004 | Valid Accounts: Cloud Accounts | Phase 2 |
| Credential Access | T1552.005 | Unsecured Credentials: Cloud Instance Metadata API | Phase 2 (variant) |
| Collection | T1530 | Data from Cloud Storage Object | Phase 3 |
| Exfiltration | T1537 | Transfer Data to Cloud Account | Phase 4 |
| Impact | T1485 | Data Destruction | Phase 4 (sabotage variant) |

---

## 8. Legal and Privacy Considerations

- **CloudTrail coverage:** CloudTrail must be enabled in all AWS regions and for management, data, and Insights events. Gaps in CloudTrail coverage are exploited by insiders who understand which regions or services are unlogged. Audit CloudTrail configuration quarterly.
- **S3 server access logging:** CloudTrail records S3 API calls at the management event level; S3 server access logging is required for per-object access detail. Both must be enabled for complete visibility.
- **AWS root account:** Any activity from the AWS root account is a critical alert regardless of context — root should never be used for day-to-day operations.
- **Cross-account attribution:** If data was copied to an external AWS account, recovering that data requires legal process (AWS subpoena). Engage Legal immediately.

---

## 9. Response Guidance

| Priority | Action | Owner |
|----------|--------|-------|
| 1 | Disable or delete the IAM identity's active access keys | Cloud Admin / IAM |
| 2 | Revoke any active console sessions (IAM → Security credentials → Sign out) | Cloud Admin |
| 3 | Preserve CloudTrail logs (enable CloudTrail log file integrity validation) | IR + Cloud Admin |
| 4 | Reverse any ACL changes that made resources publicly accessible | Cloud Admin |
| 5 | Notify Legal — assess whether data was exposed externally | Legal |

**Required AWS Configuration for Detection:**
- CloudTrail: Enabled in all regions, all event types (management + data events for S3, RDS)
- CloudTrail log integrity validation: Enabled
- S3 server access logging: Enabled for all sensitive buckets
- GuardDuty: Enabled organization-wide (supplements but does not replace SIEM-based detection)
- AWS Config: Enabled for configuration change tracking

**Linked Playbook:** [PB-XX-NNN — create using `playbooks/_template.md`]

---

## Changelog

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | [YYYY-MM-DD] | [Author] | Initial creation |

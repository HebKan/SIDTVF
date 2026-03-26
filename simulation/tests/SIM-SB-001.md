# Simulation Test: SIM-SB-001

| Field | Value |
|-------|-------|
| **Simulation ID** | SIM-SB-001 |
| **Linked Scenario** | SB-001 — Targeted Database Deletion |
| **Linked Validation** | TC-SB-001 |
| **Version** | 1.0 |
| **Status** | Active |
| **Created** | 2026-03-26 |
| **Last Updated** | 2026-03-26 |
| **Estimated Duration** | 30 minutes + 30 minute observation window |
| **Required Environment** | Dedicated test database — NEVER run in production |
| **Minimum Maturity Level** | 3 |

---

## Objective

Reproduce the behavioral pattern of a privileged insider performing targeted database deletion: reconnaissance of database objects, backup system access or modification, and DROP/DELETE commands on tables. The simulation must generate database audit log events sufficient to trigger SB-001 detection rules — without affecting any production data.

---

## Authorization

> **CRITICAL:** This simulation involves executing `DROP TABLE` commands on a test database. These commands are IRREVERSIBLE without backup. This simulation must NEVER be run against a production database. A dedicated test database with synthetic data is absolutely required. Complete the [Authorization Checklist](../authorization-checklist.md) and obtain explicit sign-off from the DBA lead and system owner before proceeding.

---

## Prerequisites

### Environment
- [ ] A **dedicated test database** exists on a non-production server (e.g., `sim-test-db.corp.local`)
- [ ] The test database is named `SIM_TEST_DB` and contains only synthetic data
- [ ] Database audit logging is enabled and flowing to SIEM: SQL Server Audit, Oracle Unified Auditing, or PostgreSQL pgaudit
- [ ] The test user account `sim_testdba` exists with DBA-equivalent privileges on `SIM_TEST_DB` only
- [ ] SIEM detection rules for SB-001 (DROP TABLE, DELETE DATABASE, backup system access) are deployed
- [ ] The DBA lead has been notified and has confirmed this is a test database
- [ ] SOC has been notified of simulation window

### Test Data Setup
- [ ] `SIM_TEST_DB` contains at least 3 test tables with 1000+ rows of synthetic data each
- [ ] A test backup of `SIM_TEST_DB` exists (to enable cleanup)

```sql
-- Setup script: run as DBA before simulation
CREATE DATABASE SIM_TEST_DB;
USE SIM_TEST_DB;

CREATE TABLE employees_FAKE (id INT, name VARCHAR(50), dept VARCHAR(50));
CREATE TABLE customers_FAKE (id INT, email VARCHAR(100), region VARCHAR(50));
CREATE TABLE transactions_FAKE (id INT, amount DECIMAL, date DATE);

-- Populate with synthetic rows (1000 rows each)
INSERT INTO employees_FAKE SELECT TOP 1000 ROW_NUMBER() OVER (ORDER BY (SELECT NULL)),
    'Employee_' + CAST(ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) AS VARCHAR),
    'Dept_' + CAST(ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) % 10 AS VARCHAR)
FROM sys.all_objects;

-- Similar for other tables...
PRINT 'SIM_TEST_DB setup complete. This database contains only synthetic data.';
```

---

## Simulation Steps

### Phase 1 — Database Reconnaissance

Reproduces the enumeration behavior seen in SB-001 Phase 1. This generates `SELECT` queries against system catalog tables — a behavioral signal that precedes targeted deletion.

| Step | Action | Expected Log Source | Expected Log Event |
|------|--------|--------------------|--------------------|
| 1.1 | Connect to `sim-test-db.corp.local` as `sim_testdba` from test workstation | DB audit log | Successful connection: user=sim_testdba, database=SIM_TEST_DB |
| 1.2 | Run: `SELECT name, size FROM sys.databases;` | DB audit log | AuditEvent: SELECT on sys.databases |
| 1.3 | Run: `SELECT schema_name(schema_id), name FROM sys.tables;` | DB audit log | AuditEvent: SELECT on sys.tables |
| 1.4 | Run: `SELECT name FROM sys.backup_devices;` | DB audit log | AuditEvent: SELECT on sys.backup_devices |
| 1.5 | Run: `EXEC sp_helpdb 'SIM_TEST_DB';` | DB audit log | AuditEvent: sp_helpdb execution |

**Expected artifact:** Sequence of catalog queries (sys.databases, sys.tables, sys.backup_devices) by a single user in a short time window — matches reconnaissance IOB pattern.

### Phase 2 — Backup Manipulation Simulation

Reproduces the backup targeting behavior. **Does not delete real backups.** Only marks the backup device as inaccessible via permissions, then restores.

| Step | Action | Expected Log Source | Expected Log Event |
|------|--------|--------------------|--------------------|
| 2.1 | Run: `EXEC sp_dropdevice 'SIM_TEST_BACKUP_DEVICE';` — note: if no test backup device exists, skip to 2.2 | DB audit log | AuditEvent: sp_dropdevice or PERMISSION DENIED |
| 2.2 | Run: `REVOKE EXECUTE ON sp_addumpdevice FROM PUBLIC;` | DB audit log | AuditEvent: REVOKE permission change |
| 2.3 | Attempt to create a new backup (should fail): `BACKUP DATABASE SIM_TEST_DB TO DISK='NUL'` | DB audit log | AuditEvent: BACKUP DATABASE — success or failure |

**Expected artifact:** Backup-related DDL/DCL events within the same session as Step 1 reconnaissance — matches "backup system targeting" IOB.

### Phase 3 — Data Destruction (DROP TABLE)

> **Final check:** Confirm you are connected to `SIM_TEST_DB` on `sim-test-db.corp.local`. Verify: `SELECT DB_NAME();` should return `SIM_TEST_DB`. Do not proceed if the result is any other database name.

| Step | Action | Expected Log Source | Expected Log Event |
|------|--------|--------------------|--------------------|
| 3.1 | Run: `SELECT DB_NAME();` — VERIFY result = `SIM_TEST_DB` | — | Must equal SIM_TEST_DB |
| 3.2 | Run: `DROP TABLE SIM_TEST_DB.dbo.employees_FAKE;` | DB audit log | AuditEvent: DROP OBJECT, ObjectName=employees_FAKE |
| 3.3 | Run: `DROP TABLE SIM_TEST_DB.dbo.customers_FAKE;` | DB audit log | AuditEvent: DROP OBJECT, ObjectName=customers_FAKE |
| 3.4 | Run: `DROP TABLE SIM_TEST_DB.dbo.transactions_FAKE;` | DB audit log | AuditEvent: DROP OBJECT, ObjectName=transactions_FAKE |
| 3.5 | Run: `DELETE FROM SIM_TEST_DB.dbo.audit_log_FAKE WHERE date < GETDATE();` (if table exists) | DB audit log | AuditEvent: DELETE, ObjectName=audit_log_FAKE |

**Expected artifact:** Multiple `DROP OBJECT` / `DROP TABLE` events from `sim_testdba` within a 5-minute window — primary trigger for SB-001 detection rules.

### Phase 4 — Log Deletion Simulation (Optional)

Tests detection of log tampering IOB.

| Step | Action | Expected Log Source | Expected Log Event |
|------|--------|--------------------|--------------------|
| 4.1 | Run: `EXEC sp_cycle_errorlog;` | SQL Server error log | Error log cycle event |
| 4.2 | In Windows Event Log: run `wevtutil cl Application` from cmd.exe as `sim_testdba` (requires elevated privileges) | Windows Security | Event 1102: Audit log cleared |

---

## Expected Log Artifacts

| Log Source | Event Type | Key Field | Expected Value |
|------------|------------|-----------|----------------|
| SQL Server Audit | Schema Object Access | object_name | `employees_FAKE`, `customers_FAKE`, `transactions_FAKE` |
| SQL Server Audit | Schema Object Access | action_id | `DR` (DROP) |
| SQL Server Audit | Server Object Access | object_name | `sys.databases`, `sys.backup_devices` |
| SQL Server Audit | Database Principal Management | — | REVOKE events |
| Windows Security | Event ID 1102 | — | Audit log cleared (Phase 4) |

---

## Expected Detection Output

| Detection | Expected Trigger | Time to Alert |
|-----------|-----------------|---------------|
| SB-001 Sigma rule (DROP TABLE) | 3+ DROP TABLE from same user in 5 min | < 10 minutes |
| SB-001 reconnaissance pattern | sys.databases + sys.backup_devices queries in same session | < 10 minutes |
| Windows Event 1102 | Audit log cleared | < 5 minutes |

---

## Observation Window

| Detection Type | Observation Window |
|---------------|-------------------|
| DROP TABLE rule (real-time) | 10 minutes |
| Behavioral correlation (sequence detection) | 15 minutes |
| Scheduled DB audit sweep | Up to 1 hour |

---

## Pass / Fail Criteria

| Result | Criteria |
|--------|----------|
| **PASS** | SIEM alert fires within 10 minutes of Phase 3 steps, showing: user=sim_testdba, event=DROP TABLE, 3+ tables in the alert |
| **PARTIAL** | Alert fires for individual DROP events but not as a correlated pattern; OR reconnaissance phase not detected |
| **FAIL** | No SIEM alert fires within 15 minutes of Phase 3 |

---

## Cleanup Steps

| Step | Action | Verified By | Date |
|------|--------|-------------|------|
| C1 | Restore `SIM_TEST_DB` from the pre-simulation backup | | |
| C2 | Verify all three tables are restored: `SELECT COUNT(*) FROM employees_FAKE;` etc. | | |
| C3 | Revoke `sim_testdba` account connection to the test database server | | |
| C4 | Restore any revoked permissions from Phase 2 | | |
| C5 | Mark all SIEM alerts as "SIM-SB-001 — simulation test" | | |
| C6 | Notify DBA lead and SOC that simulation window is closed | | |

---

## Run Log

| Run Date | Analyst | Environment | Result | Alerts Fired | Notes |
|----------|---------|-------------|--------|-------------|-------|
| | | | | | |

---

## Atomic Red Team Reference

| Technique | Atomic Test # | Notes |
|-----------|--------------|-------|
| T1485 | T1485-1 (Data Destruction) | ART covers file deletion; this simulation covers database-specific destruction that ART does not fully address |
| T1490 | T1490-1 (Inhibit System Recovery) | ART has backup deletion tests; this simulation adds the behavioral context of DBA account + reconnaissance sequence |
| T1070.004 | T1070.004-1 | ART has indicator removal tests |

---

## Linked Artifacts

| Type | ID | File |
|------|----|------|
| Scenario | SB-001 | scenarios/sabotage/SB-001-targeted-database-deletion.md |
| Playbook | PB-SB-001 | playbooks/examples/PB-SB-001-database-deletion.md |

---

## Changelog

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | 2026-03-26 | SIDTVF Core Team | Initial creation |

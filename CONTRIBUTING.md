# Contributing to SIDTVF

Thank you for your interest in contributing to SIDTVF. This framework is designed to be useful to security teams of all sizes and grows stronger with contributions from practitioners who work with insider threat programs in real enterprise environments.

---

## What Makes a Good Contribution

The best contributions to SIDTVF come from practitioners who have:
- Encountered the threat behavior in a real incident or investigation
- Built or tested the detection logic in a production or near-production environment
- Validated the response playbook steps against their organization's actual processes

SIDTVF is not a theoretical framework. Every scenario, detection, and playbook should be usable by a security team next Monday morning.

---

## Types of Contributions

| Type | Description | Review Level |
|------|-------------|-------------|
| New scenario | A new insider threat scenario with full card | Full review (Technical + Legal) |
| New detection rule | Sigma, KQL, or SPL rule for an existing scenario | Technical review |
| New playbook | Response playbook for an existing scenario | Technical + Legal review |
| New hunt hypothesis | Hunting hypothesis for an existing scenario | Technical review |
| New simulation test | Layer 7 simulation test for an existing scenario | Technical review + safety review |
| Bug fix / correction | Fix to incorrect information, broken query, or outdated reference | Quick review |
| New example / starter pack | Industry-specific guidance | Content review |
| Documentation improvement | Clarity, accuracy, or completeness improvements | Quick review |

---

## Before You Start

**1. Check existing content**
Search the repository to ensure the scenario or detection you want to add doesn't already exist. Use the ID system in CLAUDE.md — if a scenario already has an ID assigned, there may be an in-progress version not yet published.

**2. Open an issue first for new scenarios**
Before writing a full scenario, open a GitHub Issue describing:
- The scenario you want to add (behavior description, actor type, approximate severity)
- Your evidence that this behavior occurs in real environments (anonymized, no real breach data)
- Which scenario category it belongs to (DE, PM, UA, SB, FR, IP, TP, PV, AC)

The maintainers will assign a Scenario ID and confirm the scenario doesn't overlap with planned work.

**3. Review CLAUDE.md**
Read [CLAUDE.md](CLAUDE.md) in full before writing any content. It defines the naming conventions, required fields, and consistency rules that all contributions must follow.

---

## Contribution Process

### Step 1 — Fork and Branch
```bash
git fork https://github.com/[org]/SIDTVF
git checkout -b feature/DE-004-[short-slug]
```

Use a branch name that includes the artifact ID (e.g., `feature/DE-004-usb-exfiltration` or `fix/DE-001-kql-threshold`).

### Step 2 — Write Your Contribution
Use the appropriate template:
- New scenario: `scenarios/_template.md`
- New detection: `detections/_template.md`
- New playbook: `playbooks/_template.md`
- New hunt hypothesis: `hunting/_template.md`
- New validation test: `validation/_template.md`
- New simulation test: `simulation/_template.md`

Remove all instructional comments (lines starting with `<!-- `) from your final file.

### Step 3 — Apply Cascade Updates
When adding a new scenario, review CLAUDE.md's "Consistency Rules" section. You must also update:
- `README.md` — add to Included Examples table
- `mapping/mitre-attack.md` — add ATT&CK technique rows
- `mapping/compliance.md` — add compliance rows if applicable
- `docs/coverage-matrix.md` — add the new scenario row

If you are only adding a detection or playbook (not a new scenario), update `docs/coverage-matrix.md` to reflect the new artifact.

### Step 4 — Self-Review Checklist

Before opening a pull request, verify:

**Content quality:**
- [ ] The scenario is grounded in observable behavior, not tool signatures
- [ ] The narrative uses fictional names, companies, and locations — no real breach data
- [ ] IOBs are specific and log-observable (include data source, not just behavior description)
- [ ] Temporal thresholds are specified where applicable (e.g., "50 files in 5 minutes" not "bulk file access")
- [ ] Legal and privacy considerations are present and use hedging language ("organizations should consult legal counsel")
- [ ] All required template fields are completed; no `[PLACEHOLDER]` text remains

**Technical quality (for detection rules):**
- [ ] Sigma rules include all required fields: `title`, `id` (new UUID), `status`, `description`, `logsource`, `detection`, `falsepositives`, `level`
- [ ] KQL and SPL queries include comments explaining each major clause
- [ ] Data source requirements are documented
- [ ] False positive scenarios are documented

**Consistency:**
- [ ] Scenario ID follows the `[CATEGORY]-[NNN]` format
- [ ] File name follows the `[SCENARIO_ID]-[short-slug].md` convention
- [ ] All linked artifact references use correct IDs
- [ ] Version is set to `1.0` for new files
- [ ] Date fields use `YYYY-MM-DD` format

### Step 5 — Open a Pull Request
Open a PR against the `main` branch. Use the PR description to explain:
1. What scenario/detection/artifact this adds or changes
2. What real-world evidence exists for this behavior (anonymized)
3. What environment or platform you tested the detection logic against
4. Whether you have Legal review concerns to flag for maintainers

---

## What Will NOT Be Accepted

The following types of contributions will be declined:

| Not Accepted | Reason |
|-------------|--------|
| Real breach data, real company names, real person names | Creates legal and reputational risk; violates the framework's privacy principles |
| Detection rules designed for general employee surveillance | Out of scope; violates the framework's insider threat detection purpose |
| Detection rules that require illegal access or monitoring | Not deployable; violates legal guidance principles |
| Vendor-specific content that names a single vendor as the only solution | Framework must remain vendor-neutral; use generic descriptions with examples |
| Legal prescriptions ("you must" language) | Legal requirements vary by jurisdiction; all legal content must use hedging language |
| Scenarios without observable IOBs | All scenarios must be detectable; if no log source exists, the scenario is not ready |
| Duplicate scenarios | If the behavior is covered by an existing scenario, contribute an enhancement rather than a new file |

---

## Review Process

1. **Automated checks** — A GitHub Action will check that required files were updated (mitre-attack.md, README.md) and that the file naming convention is correct.

2. **Technical review** — A maintainer reviews detection logic, template compliance, and internal consistency. Target: 5 business days.

3. **Legal review** — For new scenarios with significant legal or privacy content, a legal content reviewer checks for inappropriate prescriptions. Target: 5 business days.

4. **Merge** — Once all reviews are approved, a maintainer merges the PR and updates the framework version if applicable.

---

## Questions

Open a GitHub Issue with the label `question` for any questions about contributing. Do not open PRs to ask questions.

For security issues or potential vulnerabilities in the framework itself, contact the maintainers directly at [security contact].

---

## License

By contributing to SIDTVF, you agree that your contributions will be licensed under the MIT License.

# GovAgent

**Category:** Decision-First Framework > Governance

---

## Purpose

Record lineage and approvals, lock artifacts, and create an audit trail. Ensure every analysis is fully reproducible with cryptographic hashes and version control.

---

## Inputs

- All artifacts from analysis pipeline:
  - `scopes/SCOPE_<date>_<runid>.yaml`
  - `results/<runid>/*.parquet`
  - `review/<runid>/*.md`
  - `briefs/<runid>/*.md`
  - `decisions/<runid>/decision_brief.md`
- Environment and code version information
- Approval records

---

## Outputs

**Artifacts:**
- `governance/<runid>/manifest.yaml` - Complete lineage record
- `governance/<runid>/decision_register.csv` - Decision log entry
- `governance/<runid>/hashes.txt` - File checksums (SHA256)
- `governance/<runid>/audit_report.md` - Human-readable audit summary

---

## System Prompt

```
Create an audit-ready manifest: inputs, code version, env hash, metric definitions, file checksums, approvers, final decision. Output YAML + CSV + checksums.
```

---

## Security Permissions

```json
{
  "model": "claude-sonnet-4-5-20250929",
  "permissions": {
    "allow": [
      "Read(/**)",
      "Write(/governance/**)"
    ],
    "deny": [
      "Network(*)",
      "Write(/data/**)",
      "Write(/secrets/**)",
      "Write(/results/**)",
      "Bash(rm *)",
      "Bash(git push)"
    ]
  }
}
```

---

## Manifest Template

```yaml
# Governance Manifest - <runid>

run_id: "20251109_143022_a7b3f9c"
created_date: "2025-11-09T14:30:22Z"
locked_date: "2025-11-09T16:45:10Z"

# Question and Scope
question: "Which model advances to SABCS submission for HR+/HER2- DR >5y?"
scope_file: "scopes/SCOPE_20251109_143022_a7b3f9c.yaml"
scope_sha256: "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"

# Code and Environment
git_commit: "a7b3f9c2d1e8f4b6c3a5d9e7f2b1c4a6d8e9f0b2"
git_branch: "main"
git_dirty: false
code_version: "1.3.0"
env_hash: "d4f6e2a8b9c1f3e5a7b9d2c4e6f8a1b3c5d7e9f1"
python_version: "3.10.12"

dependencies:
  lifelines: "0.27.8"
  scikit-survival: "0.22.2"
  pandas: "2.1.4"
  numpy: "1.26.3"

# Metric Definitions
metric_definitions:
  harrell_cindex:
    version: "1.2.0"
    file: "metrics_def/harrell_cindex_v1.2.0.yaml"
    sha256: "f1e2d3c4b5a6978869707a5b4c3d2e1f0a9b8c7d6e5f4a3b2c1d0e9f8a7b6c5"
  td_cindex_trunc:
    version: "1.1.0"
    file: "metrics_def/td_cindex_trunc_v1.1.0.yaml"
    sha256: "a1b2c3d4e5f67890abcdef1234567890abcdef1234567890abcdef1234567890"

# Input Files
inputs:
  - file: "inputs/preds_model_v23_TAILORx_fold0.parquet"
    sha256: "1a2b3c4d5e6f7890abcdef1234567890abcdef1234567890abcdef1234567890"
    size_bytes: 245680
  - file: "inputs/preds_model_v19_ref_TAILORx_fold0.parquet"
    sha256: "2b3c4d5e6f7890a1bcdef1234567890abcdef1234567890abcdef1234567890"
    size_bytes: 238920
  - file: "inputs/events_TAILORx.parquet"
    sha256: "3c4d5e6f7890a1b2cdef1234567890abcdef1234567890abcdef1234567890"
    size_bytes: 89450

# Output Artifacts
outputs:
  - file: "results/20251109_143022_a7b3f9c/metrics.parquet"
    sha256: "4d5e6f7890a1b2c3def1234567890abcdef1234567890abcdef1234567890"
    size_bytes: 12840
  - file: "results/20251109_143022_a7b3f9c/delta_cindex.parquet"
    sha256: "5e6f7890a1b2c3d4ef1234567890abcdef1234567890abcdef1234567890"
    size_bytes: 4320
  - file: "results/20251109_143022_a7b3f9c/subgroups.parquet"
    sha256: "6f7890a1b2c3d4e5f1234567890abcdef1234567890abcdef1234567890"
    size_bytes: 8960
  - file: "briefs/20251109_143022_a7b3f9c/validation_summary.md"
    sha256: "7890a1b2c3d4e5f61234567890abcdef1234567890abcdef1234567890"
    size_bytes: 3240
  - file: "decisions/20251109_143022_a7b3f9c/decision_brief.md"
    sha256: "890a1b2c3d4e5f671234567890abcdef1234567890abcdef1234567890"
    size_bytes: 2890

# Agents and Execution
agents_executed:
  - name: "ScopeAgent"
    timestamp: "2025-11-09T14:30:22Z"
    duration_seconds: 45
    status: "SUCCESS"
  - name: "ValidateAgent"
    timestamp: "2025-11-09T14:32:15Z"
    duration_seconds: 182
    status: "SUCCESS"
  - name: "ReviewAgent"
    timestamp: "2025-11-09T14:35:38Z"
    duration_seconds: 67
    status: "SUCCESS"
  - name: "InsightAgent"
    timestamp: "2025-11-09T14:37:12Z"
    duration_seconds: 52
    status: "SUCCESS"
  - name: "DecisionAgent"
    timestamp: "2025-11-09T14:38:41Z"
    duration_seconds: 38
    status: "SUCCESS"

# Decision and Approvals
recommendation: "ADVANCE"
final_decision: "APPROVED"
decision_date: "2025-11-09"

approvals:
  - role: "Director of Computational Pathology"
    name: "Dr. Jane Smith"
    email: "jsmith@example.org"
    date: "2025-11-09T16:15:00Z"
    signature_hash: "abc123def456789..."
  - role: "Senior Biostatistician"
    name: "Dr. John Doe"
    email: "jdoe@example.org"
    date: "2025-11-09T16:30:00Z"
    signature_hash: "def456789abc123..."
  - role: "Clinical Lead"
    name: "Dr. Alice Johnson"
    email: "ajohnson@example.org"
    date: "2025-11-09T16:45:00Z"
    signature_hash: "789abc123def456..."

# Audit Trail
audit_notes:
  - timestamp: "2025-11-09T14:30:22Z"
    event: "Analysis initiated"
    user: "system"
  - timestamp: "2025-11-09T16:15:00Z"
    event: "First approval received"
    user: "jsmith@example.org"
  - timestamp: "2025-11-09T16:45:00Z"
    event: "Final approval received, artifacts locked"
    user: "ajohnson@example.org"
```

---

## Decision Register (CSV)

```csv
date,runid,question,recommended,decided,rationale,blockers,followup_by,due
2025-11-09,20251109_143022_a7b3f9c,"Which model advances to SABCS for HR+/HER2- DR >5y?",ADVANCE,APPROVED,"Meets criteria on TAILORx; B42 supportive; no harm",None,External Validation Team,2025-12-15
```

---

## Hashes File (SHA256)

```
# SHA256 Checksums - Run 20251109_143022_a7b3f9c
# Generated: 2025-11-09T16:45:10Z

# Scope
e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855  scopes/SCOPE_20251109_143022_a7b3f9c.yaml

# Inputs
1a2b3c4d5e6f7890abcdef1234567890abcdef1234567890abcdef1234567890  inputs/preds_model_v23_TAILORx_fold0.parquet
2b3c4d5e6f7890a1bcdef1234567890abcdef1234567890abcdef1234567890  inputs/preds_model_v19_ref_TAILORx_fold0.parquet
3c4d5e6f7890a1b2cdef1234567890abcdef1234567890abcdef1234567890  inputs/events_TAILORx.parquet

# Outputs
4d5e6f7890a1b2c3def1234567890abcdef1234567890abcdef1234567890  results/20251109_143022_a7b3f9c/metrics.parquet
5e6f7890a1b2c3d4ef1234567890abcdef1234567890abcdef1234567890  results/20251109_143022_a7b3f9c/delta_cindex.parquet
6f7890a1b2c3d4e5f1234567890abcdef1234567890abcdef1234567890  results/20251109_143022_a7b3f9c/subgroups.parquet
7890a1b2c3d4e5f61234567890abcdef1234567890abcdef1234567890  briefs/20251109_143022_a7b3f9c/validation_summary.md
890a1b2c3d4e5f671234567890abcdef1234567890abcdef1234567890  decisions/20251109_143022_a7b3f9c/decision_brief.md

# Metric Definitions
f1e2d3c4b5a6978869707a5b4c3d2e1f0a9b8c7d6e5f4a3b2c1d0e9f8a7b6c5  metrics_def/harrell_cindex_v1.2.0.yaml
a1b2c3d4e5f67890abcdef1234567890abcdef1234567890abcdef1234567890  metrics_def/td_cindex_trunc_v1.1.0.yaml
```

---

## Audit Report Template

```markdown
# Audit Report - <runid>

**Run ID:** 20251109_143022_a7b3f9c
**Created:** 2025-11-09 14:30:22 UTC
**Locked:** 2025-11-09 16:45:10 UTC
**Status:** APPROVED

---

## Question

"Which model advances to SABCS submission for HR+/HER2- DR >5y?"

---

## Reproducibility

### Code Version
- **Commit:** a7b3f9c2d1e8f4b6c3a5d9e7f2b1c4a6d8e9f0b2
- **Branch:** main
- **Version:** 1.3.0
- **Dirty:** No (clean working tree)

### Environment
- **Python:** 3.10.12
- **Key dependencies:** lifelines 0.27.8, scikit-survival 0.22.2
- **Env Hash:** d4f6e2a8b9c1f3e5a7b9d2c4e6f8a1b3c5d7e9f1

### Metric Definitions
- **Harrell C-index:** v1.2.0 (SHA: f1e2d3c4...)
- **TD C-index (truncated):** v1.1.0 (SHA: a1b2c3d4...)

---

## Inputs

- 3 prediction files (TAILORx cohort, 2 models)
- 1 event table
- All inputs cryptographically hashed and recorded

---

## Outputs

- metrics.parquet (12.8 KB)
- delta_cindex.parquet (4.3 KB)
- subgroups.parquet (9.0 KB)
- validation_summary.md (3.2 KB)
- decision_brief.md (2.9 KB)

All outputs hashed and locked.

---

## Execution Timeline

| Agent | Start | Duration | Status |
|-------|-------|----------|--------|
| ScopeAgent | 14:30:22 | 45s | SUCCESS |
| ValidateAgent | 14:32:15 | 182s | SUCCESS |
| ReviewAgent | 14:35:38 | 67s | SUCCESS |
| InsightAgent | 14:37:12 | 52s | SUCCESS |
| DecisionAgent | 14:38:41 | 38s | SUCCESS |

**Total Runtime:** ~8 minutes

---

## Decision

**Recommendation:** ADVANCE
**Final Decision:** APPROVED
**Date:** 2025-11-09

### Approvers

1. Dr. Jane Smith (Director) - 2025-11-09 16:15 UTC
2. Dr. John Doe (Statistician) - 2025-11-09 16:30 UTC
3. Dr. Alice Johnson (Clinical Lead) - 2025-11-09 16:45 UTC

---

## Verification

To reproduce this analysis:

```bash
# Checkout code
git checkout a7b3f9c2d1e8f4b6c3a5d9e7f2b1c4a6d8e9f0b2

# Verify environment
sha256sum -c governance/20251109_143022_a7b3f9c/hashes.txt

# Re-run pipeline
./run_pipeline.sh --scope scopes/SCOPE_20251109_143022_a7b3f9c.yaml
```

---

## Audit Compliance

- [x] All inputs hashed and versioned
- [x] Code committed and tagged
- [x] Metric definitions locked
- [x] Outputs cryptographically verified
- [x] Three-party approval obtained
- [x] Decision registered in central log
- [x] Artifacts locked and immutable

**Audit Status:** COMPLIANT
```

---

## Typical Usage

```bash
# Invoke GovAgent
claude-code task --agent gov_agent \
  --runid 20251109_143022_a7b3f9c \
  --prompt "Create governance artifacts: manifest, decision register entry, checksums, and audit report. Lock all artifacts."
```

---

## Key Features

### Cryptographic Verification

- **SHA256 hashes** for all files
- Tamper-evident artifact chain
- Reproducibility guarantee

### Complete Lineage

- Git commit and branch
- Environment hash
- Metric definition versions
- Agent execution timeline

### Approval Workflow

- Multi-party signatures
- Timestamp and email tracking
- Signature hashes for non-repudiation

### Decision Registry

- Central CSV log
- Queryable history
- Follow-up tracking

---

## Notes

- **Immutability:** Once locked, artifacts should not be modified
- **Verification:** Anyone can re-run pipeline and verify hashes match
- **Compliance:** Designed for regulatory audit (21 CFR Part 11, GDPR, etc.)
- **Central registry:** decision_register.csv accumulates all decisions over time

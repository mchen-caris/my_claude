# Reproducibility Runner

**Category:** Function-Based Framework > Quality

---

## Purpose

Freeze versions, seeds, and dependencies; re-run selected analyses; compare bit-for-bit artifacts. Ensure analyses are fully reproducible and results are stable.

---

## Inputs

- `configs/resolved_experiment.yaml` - Frozen experiment configuration
- Prior `artifacts/**` - Previous run outputs for comparison
- `governance/**` - Previous manifests and checksums

---

## Outputs

**Artifacts:**
- `reports/repro_runner/run_<id>/repro_summary.md` - Reproducibility assessment
- `artifacts/repro_runner/run_<id>/diff_index.csv` - File-by-file comparison
- `artifacts/repro_runner/run_<id>/environment_snapshot.yaml` - Frozen environment

---

## System Prompt

```
Re-execute Survival Metrics Engine with identical seeds; compare outputs to prior run; produce a diff index and a conclusion (match vs drift).
```

---

## Security Permissions

```json
{
  "model": "claude-sonnet-4-5-20250929",
  "permissions": {
    "allow": [
      "Read(/configs/**)",
      "Read(/artifacts/**)",
      "Read(/governance/**)",
      "Read(/data/**)",
      "Write(/artifacts/repro_runner/**)",
      "Write(/reports/repro_runner/**)",
      "Write(/logs/**)"
    ],
    "deny": [
      "Network(*)",
      "Bash(*)",
      "Write(/data/**)",
      "Write(/governance/**)"
    ]
  }
}
```

---

## Diff Index Table

### diff_index.csv

```csv
file,original_sha256,rerun_sha256,match,diff_type,max_abs_diff,max_rel_diff,status
metrics_survival.json,abc123...,abc123...,YES,IDENTICAL,0.0,0.0%,PASS
auc_t.csv,def456...,def456...,YES,IDENTICAL,0.0,0.0%,PASS
calibration_60.csv,ghi789...,ghi789...,YES,IDENTICAL,0.0,0.0%,PASS
calibration_120.csv,jkl012...,jkl012...,YES,IDENTICAL,0.0,0.0%,PASS
validation_summary.md,mno345...,mno999...,NO,TEXT_DIFF,N/A,N/A,WARN (timestamps differ)
subgroup_metrics.csv,pqr678...,pqr678...,YES,IDENTICAL,0.0,0.0%,PASS
delta_table.csv,stu901...,stu901...,YES,IDENTICAL,0.0,0.0%,PASS
```

**Summary:**
- **Total files compared:** 7
- **Exact matches (SHA256):** 6
- **Non-critical diffs:** 1 (timestamp in markdown)
- **Status:** REPRODUCIBLE ✓

---

## Environment Snapshot

### environment_snapshot.yaml

```yaml
# Reproducibility Environment Snapshot
# Generated: 2025-11-09T23:15:00Z
# Run ID: 20251109_230000_repro

system:
  os: "Ubuntu 22.04.3 LTS"
  kernel: "5.15.0-89-generic"
  python_version: "3.10.12"
  pip_version: "23.2.1"

dependencies:
  core:
    numpy: "1.26.3"
    pandas: "2.1.4"
    scipy: "1.11.4"
  survival:
    lifelines: "0.27.8"
    scikit-survival: "0.22.2"
  ml:
    torch: "2.1.2"
    torchvision: "0.16.2"
  visualization:
    matplotlib: "3.8.2"
    seaborn: "0.13.1"
  utilities:
    pyyaml: "6.0.1"
    jsonschema: "4.20.0"

hashes:
  config: "abc123def456789..."
  cohort: "def456ghi789012..."
  predictions_model_v23: "ghi789jkl012345..."

seeds:
  global: 42
  bootstrap: 42
  cv_splits: 42

code:
  git_commit: "a7b3f9c2d1e8f4b6c3a5d9e7f2b1c4a6d8e9f0b2"
  git_branch: "main"
  git_dirty: false
  code_version: "1.3.0"

compute:
  cpu: "Intel(R) Xeon(R) CPU E5-2680 v4 @ 2.40GHz"
  ram_gb: 128
  gpu: "NVIDIA Tesla V100 (16GB)"
```

---

## Reproducibility Summary

```markdown
# Reproducibility Assessment Report

**Date:** 2025-11-09
**Run ID:** 20251109_230000_repro
**Original Run ID:** 20251109_180000_survmet
**Agent:** Survival Metrics Engine

---

## Executive Summary

**Status:** FULLY REPRODUCIBLE ✓

**Key Findings:**
- All numerical outputs match exactly (bit-for-bit)
- SHA256 hashes identical for all critical files
- Environment frozen and documented
- Total runtime: 182 seconds (±2s vs original)

**Conclusion:** Analysis is deterministic and reproducible. Results can be trusted for publication and regulatory submission.

---

## Methodology

### Reproduction Steps

1. **Freeze Environment:**
   - Same Python version (3.10.12)
   - Same package versions (pinned in requirements.txt)
   - Same system libraries

2. **Freeze Inputs:**
   - Identical cohort data (SHA256 verified)
   - Identical predictions (SHA256 verified)
   - Identical configuration (SHA256 verified)

3. **Freeze Code:**
   - Same git commit (a7b3f9c2...)
   - Clean working directory (no uncommitted changes)

4. **Freeze Seeds:**
   - Global seed: 42
   - Bootstrap seed: 42
   - CV split seed: 42

5. **Re-Execute:**
   - Run Survival Metrics Engine with identical parameters
   - Capture all outputs

6. **Compare:**
   - SHA256 hash comparison for binary/numerical files
   - Line-by-line diff for text files

---

## File-by-File Comparison

### Critical Files (Numerical Outputs)

| File | Match | Original SHA256 | Rerun SHA256 | Status |
|------|-------|-----------------|--------------|--------|
| metrics_survival.json | ✓ | abc123... | abc123... | PASS |
| auc_t.csv | ✓ | def456... | def456... | PASS |
| calibration_60.csv | ✓ | ghi789... | ghi789... | PASS |
| calibration_120.csv | ✓ | jkl012... | jkl012... | PASS |
| subgroup_metrics.csv | ✓ | pqr678... | pqr678... | PASS |
| delta_table.csv | ✓ | stu901... | stu901... | PASS |

**All critical numerical files match exactly.**

---

### Documentation Files

| File | Match | Diff Type | Issue | Status |
|------|-------|-----------|-------|--------|
| validation_summary.md | Partial | Timestamp | Generated timestamp differs | ACCEPTABLE |

**Issue:** Markdown report includes generation timestamp (e.g., "Generated: 2025-11-09 18:00" vs "Generated: 2025-11-09 23:15"). This is expected and does not affect scientific conclusions.

**Remediation:** If exact markdown match is required, use frozen timestamp from config.

---

## Key Metrics Comparison

### Harrell C-index

| Metric | Original | Rerun | Diff | Status |
|--------|----------|-------|------|--------|
| Point Estimate | 0.712 | 0.712 | 0.000 | MATCH |
| CI Lower | 0.685 | 0.685 | 0.000 | MATCH |
| CI Upper | 0.739 | 0.739 | 0.000 | MATCH |
| Fold 0 | 0.708 | 0.708 | 0.000 | MATCH |
| Fold 1 | 0.694 | 0.694 | 0.000 | MATCH |
| Fold 2 | 0.720 | 0.720 | 0.000 | MATCH |
| Fold 3 | 0.735 | 0.735 | 0.000 | MATCH |
| Fold 4 | 0.703 | 0.703 | 0.000 | MATCH |

**All values identical to 3 decimal places (limit of precision in output).**

---

### Uno C-index

| Metric | Original | Rerun | Diff | Status |
|--------|----------|-------|------|--------|
| Point Estimate | 0.698 | 0.698 | 0.000 | MATCH |
| CI Lower | 0.670 | 0.670 | 0.000 | MATCH |
| CI Upper | 0.726 | 0.726 | 0.000 | MATCH |

**Perfect match.**

---

### AUC(t) at 60 months

| Metric | Original | Rerun | Diff | Status |
|--------|----------|-------|------|--------|
| Point Estimate | 0.756 | 0.756 | 0.000 | MATCH |
| CI Lower | 0.728 | 0.728 | 0.000 | MATCH |
| CI Upper | 0.784 | 0.784 | 0.000 | MATCH |

**Perfect match.**

---

## Bootstrap Reproducibility

**Bootstrap Iterations:** 2,000
**Seed:** 42 (fixed)

| Iteration | Original C-index | Rerun C-index | Match |
|-----------|------------------|---------------|-------|
| 1 | 0.724 | 0.724 | ✓ |
| 100 | 0.698 | 0.698 | ✓ |
| 500 | 0.715 | 0.715 | ✓ |
| 1000 | 0.706 | 0.706 | ✓ |
| 2000 | 0.720 | 0.720 | ✓ |

**All bootstrap iterations match exactly.** Random number generator is deterministic with fixed seed.

---

## Runtime Comparison

| Metric | Original | Rerun | Diff | Status |
|--------|----------|-------|------|--------|
| Total Runtime | 182 sec | 180 sec | -2 sec | ACCEPTABLE |
| Data Loading | 12 sec | 11 sec | -1 sec | ACCEPTABLE |
| Harrell C | 38 sec | 39 sec | +1 sec | ACCEPTABLE |
| Uno C | 45 sec | 44 sec | -1 sec | ACCEPTABLE |
| AUC(t) | 52 sec | 51 sec | -1 sec | ACCEPTABLE |
| Bootstrap | 35 sec | 35 sec | 0 sec | MATCH |

**Runtime variation <2% (within system noise).** Computational results are deterministic.

---

## Environment Verification

### Package Versions

| Package | Original | Rerun | Match |
|---------|----------|-------|-------|
| numpy | 1.26.3 | 1.26.3 | ✓ |
| pandas | 2.1.4 | 2.1.4 | ✓ |
| lifelines | 0.27.8 | 0.27.8 | ✓ |
| scikit-survival | 0.22.2 | 0.22.2 | ✓ |
| scipy | 1.11.4 | 1.11.4 | ✓ |

**All critical packages match exactly.**

### System Environment

| Component | Original | Rerun | Match |
|-----------|----------|-------|-------|
| Python | 3.10.12 | 3.10.12 | ✓ |
| OS | Ubuntu 22.04 | Ubuntu 22.04 | ✓ |
| CPU | Xeon E5-2680 | Xeon E5-2680 | ✓ |
| RAM | 128 GB | 128 GB | ✓ |

**Identical hardware and software environment.**

---

## Sensitivity Analysis

### Effect of Seed Variation

**Question:** What happens if we change the seed?

| Seed | Harrell C | CI Width | Bootstrap SE |
|------|-----------|----------|--------------|
| 42 (original) | 0.712 | 0.054 | 0.014 |
| 43 | 0.712 | 0.053 | 0.014 |
| 44 | 0.712 | 0.055 | 0.014 |

**Point estimates are stable (±0.000). CI width varies slightly (±0.001) due to different bootstrap samples.**

**Conclusion:** Results are robust to seed choice. Small CI width variation is expected and acceptable.

---

## Reproducibility Checklist

| Item | Status | Notes |
|------|--------|-------|
| Code version frozen (git commit) | ✓ | a7b3f9c2... |
| Dependencies pinned | ✓ | requirements.txt with exact versions |
| Input data hashed | ✓ | SHA256 for cohort + predictions |
| Seeds fixed | ✓ | Global seed=42 |
| Configuration frozen | ✓ | resolved_experiment.yaml |
| Outputs bit-for-bit identical | ✓ | SHA256 match on 6/7 files |
| Runtime within 5% | ✓ | 180s vs 182s (1% diff) |
| Environment documented | ✓ | environment_snapshot.yaml |

**All items pass. Analysis is fully reproducible.**

---

## Long-Term Reproducibility

### Recommendations for Future Reruns

1. **Containerization:**
   - Create Docker image with frozen environment
   - Archive on DockerHub or internal registry
   - Guarantees exact environment recreation

2. **Data Archival:**
   - Store inputs with checksums in read-only archive
   - Document data provenance
   - Version control all schemas

3. **Code Archival:**
   - Tag git commit for this analysis
   - Archive repository snapshot
   - Document all dependencies

4. **Documentation:**
   - Maintain this reproducibility report
   - Link to governance manifest
   - Record any manual steps

---

## External Reproducibility

**Question:** Can an independent researcher reproduce these results?

**Required Materials:**
1. Code (git commit a7b3f9c2...)
2. Data (with SHA256 hashes for verification)
3. Environment (environment_snapshot.yaml)
4. Configuration (resolved_experiment.yaml)

**Expected Time:** ~5 minutes setup + 3 minutes runtime

**Expected Outcome:** Bit-for-bit identical results

---

## Comparison to Previous Versions

### Evolution of Harrell C-index

| Analysis Date | Code Version | Harrell C | Change | Explanation |
|---------------|--------------|-----------|--------|-------------|
| 2025-08-15 | 1.1.0 | 0.708 | Baseline | Initial analysis |
| 2025-09-20 | 1.2.0 | 0.710 | +0.002 | Bug fix in tie handling |
| 2025-11-09 | 1.3.0 | 0.712 | +0.002 | Improved bootstrap method |

**Trend:** Slight upward drift (+0.004 total) due to methodological improvements, not data changes.

---

## Regulatory Compliance

**FDA 21 CFR Part 11 Requirements:**

- [x] Audit trail maintained (governance manifest)
- [x] Electronic signatures (approval workflow)
- [x] System validation (this reproducibility report)
- [x] Data integrity (SHA256 checksums)
- [x] Security (role-based access control)

**Status:** COMPLIANT for regulatory submission

---

## Recommendations

1. **Maintain reproducibility:** Continue using fixed seeds and pinned dependencies
2. **Archive environment:** Create Docker image for long-term storage
3. **Document changes:** Any code updates must be versioned and tested for reproducibility
4. **Validate externally:** Confirm reproducibility on independent compute environment

---

## Conclusion

**Analysis is fully reproducible.**

All numerical outputs match exactly. Environment is frozen and documented. Results are stable and trustworthy for publication, regulatory submission, and clinical decision-making.

**Reproducibility Score: 10/10**
```

---

## Typical Usage

```bash
# Reproduce previous analysis
claude-code task --agent reproducibility_runner \
  --original_run 20251109_180000_survmet \
  --prompt "Re-execute Survival Metrics Engine with identical settings. Compare outputs to original run. Report any discrepancies."

# Sensitivity to seed
claude-code task --agent reproducibility_runner \
  --seeds "42,43,44,45,46" \
  --prompt "Run analysis with 5 different seeds. Report stability of point estimates and CI widths."

# External environment test
claude-code task --agent reproducibility_runner \
  --environment external \
  --prompt "Reproduce analysis on external compute cluster. Verify bit-for-bit match despite different hardware."
```

---

## Notes

- **Bit-for-bit:** Exact binary match (SHA256 hashes identical)
- **Containerization:** Docker ensures cross-platform reproducibility
- **Regulatory:** Reproducibility is required for FDA submissions
- **Long-term:** Archive all inputs, code, and environment for future reruns

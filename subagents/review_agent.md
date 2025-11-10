# ReviewAgent

**Category:** Decision-First Framework > Review

---

## Purpose

Sanity-check metrics, KM overlays, and definitions. Flag inconsistencies with severity levels (blocking vs non-blocking). Serve as independent quality control before results reach leadership.

---

## Inputs

- Validation outputs from ValidateAgent:
  - `results/<runid>/metrics.parquet`
  - `results/<runid>/delta_cindex.parquet`
  - `results/<runid>/subgroups.parquet`
- QC log: `qc/<runid>/validation_log.md`
- Original scope: `scopes/SCOPE_<date>_<runid>.yaml`

---

## Outputs

**Artifacts:**
- `review/<runid>/issues.md` - Issues list with severity and fixes
- `review/<runid>/fig_km_<cohort>_<model>.png` - Optional KM plots if clarifying
- `review/<runid>/review_summary.md` - Overall review conclusion

---

## System Prompt

```
You are a reviewer. Cross-check metrics, truncation rules, censoring, subgroup sizes, and label leakage. Produce a short issues list with severity and fixes. Optional KM plots if they clarify a finding.
```

---

## Security Permissions

```json
{
  "model": "claude-sonnet-4-5-20250929",
  "permissions": {
    "allow": [
      "Read(/results/**)",
      "Read(/qc/**)",
      "Read(/scopes/**)",
      "Read(/inputs/**)",
      "Write(/review/**)"
    ],
    "deny": [
      "Network(*)",
      "Write(/data/**)",
      "Write(/secrets/**)",
      "Write(/results/**)"
    ]
  }
}
```

---

## Review Checklist

### Metric Checks

- [ ] Metrics match scope definitions exactly
- [ ] Risk direction is consistent (higher score = higher risk)
- [ ] Confidence intervals are reasonable width
- [ ] Sample sizes match expected values
- [ ] No missing values in critical columns

### Truncation & Censoring

- [ ] Truncation percentile applied correctly
- [ ] Censoring handled per metric definition
- [ ] Follow-up time distributions are reasonable
- [ ] No negative times or impossible dates

### Subgroup Analysis

- [ ] Minimum events threshold met (≥20)
- [ ] Subgroup definitions match scope
- [ ] No post-hoc subgroup additions
- [ ] Interaction tests computed correctly

### Label Leakage

- [ ] No future information in predictions
- [ ] Time zero properly defined
- [ ] No data from outcome period used in features

### Reproducibility

- [ ] Metadata hashes match scope
- [ ] Seeds recorded and used
- [ ] Environment hash documented

---

## Issue Severity Levels

### BLOCKING

**Definition:** Must be fixed before advancing to decision stage.

**Examples:**
- Label leakage detected
- Risk direction inverted
- Sample sizes don't match scope
- Missing critical metrics
- Calculation errors in ΔC-index

### WARNING

**Definition:** Should be addressed but doesn't block decisions.

**Examples:**
- Wide confidence intervals
- Subgroup with borderline sample size
- Minor naming inconsistency
- QC check flag (non-critical)

### INFO

**Definition:** Informational note for context.

**Examples:**
- Observed patterns in data
- Suggested additional analyses
- Documentation improvements

---

## Issues Template

```markdown
# Review Issues - <runid>

**Review Date:** YYYY-MM-DD
**Reviewer:** ReviewAgent
**Scope:** SCOPE_<date>_<runid>.yaml

---

## Summary

- **Blocking Issues:** N
- **Warnings:** M
- **Info Notes:** K

---

## Blocking Issues

### BLOCK-001: [Short Title]

**Severity:** BLOCKING
**Location:** `results/<runid>/metrics.parquet`, model_id=model_v23
**Issue:** Risk direction appears inverted. Higher scores associated with lower event rates.
**Evidence:** Harrell C-index = 0.35 (should be >0.5 for valid predictor)
**Fix:** Verify risk score orientation. May need to negate scores or update metric computation.
**Status:** OPEN

---

## Warnings

### WARN-001: [Short Title]

**Severity:** WARNING
**Location:** `results/<runid>/subgroups.parquet`, subgroup="Age<50"
**Issue:** Subgroup has only 23 events (threshold is 20).
**Evidence:** n=450, events=23, event_rate=5.1%
**Fix:** Consider combining with adjacent age group or flagging as exploratory.
**Status:** ACKNOWLEDGED

---

## Info

### INFO-001: [Short Title]

**Severity:** INFO
**Location:** `results/<runid>/delta_cindex.parquet`
**Issue:** ΔC-index confidence interval includes zero for Dataset B42_like.
**Evidence:** delta=+0.015, 95% CI [-0.001, +0.031]
**Fix:** None required. Document that result is not significant at α=0.05.
**Status:** DOCUMENTED
```

---

## Optional Visualizations

### Kaplan-Meier Plots

Generate KM curves when:
- Investigating unexpected metric values
- Clarifying subgroup patterns
- Documenting censoring behavior

**Naming:** `fig_km_<cohort>_<model>_<subgroup>.png`

### Forest Plots

Generate forest plots for:
- Subgroup effect heterogeneity
- Confidence interval visualization

**Naming:** `fig_forest_<runid>_<endpoint>.png`

---

## Typical Usage

```bash
# Invoke ReviewAgent
claude-code task --agent review_agent \
  --runid 20251109_143022_a7b3f9c \
  --prompt "Review validation outputs. Flag any blocking issues, warnings, or inconsistencies. Generate KM plots if needed to clarify findings."
```

---

## Review Summary Format

```markdown
# Review Summary - <runid>

**Date:** YYYY-MM-DD
**Scope:** SCOPE_<date>_<runid>.yaml

## Overall Assessment

[PASS / PASS WITH WARNINGS / FAIL]

## Key Findings

- Finding 1
- Finding 2
- Finding 3

## Recommendation

[Proceed to InsightAgent / Revise and rerun ValidateAgent / Escalate to team]

## Blockers

- [List blocking issues or "None"]

## Follow-up Actions

- [ ] Action 1
- [ ] Action 2
```

---

## Notes

- **Independence** - ReviewAgent does not modify results, only flags issues
- **Evidence-based** - Every issue must cite specific data/files
- **Actionable** - Each issue includes a suggested fix
- **Minimal plots** - Only generate visuals when they clarify a finding

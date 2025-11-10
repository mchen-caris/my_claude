# Cohort Profiler

**Category:** Function-Based Framework > Exploration

---

## Purpose

One-click cohort overview and readiness: counts, missingness, distributions, stratified summaries. Validate data quality before analysis begins.

---

## Inputs

- `data/cohort.parquet` (or `.csv`) - Cohort data table
- `schemas/cohort.json` (optional) - Schema for validation

---

## Outputs

**Artifacts:**
- `artifacts/cohort_profiler/run_<id>/baseline_table.csv` - Stratified baseline characteristics
- `artifacts/cohort_profiler/run_<id>/missingness.csv` - Missing data summary
- Figures:
  - `age_hist.png` - Age distribution
  - `stage_bar.png` - Stage distribution
  - `km_overall.png` - Kaplan-Meier for overall cohort
  - `km_by_arm.png` - KM stratified by treatment arm (if applicable)
- `reports/cohort_profiler/run_<id>/summary.md` - Human-readable summary

---

## System Prompt

```
Profile `data/cohort.parquet`: generate baseline table by arm and overall; compute event rate and median follow-up; run schema validation; produce KM for overall and by arm.
```

---

## Security Permissions

```json
{
  "model": "claude-sonnet-4-5-20250929",
  "permissions": {
    "allow": [
      "Read(/data/cohort.*)",
      "Read(/schemas/cohort.json)",
      "Write(/artifacts/cohort_profiler/**)",
      "Write(/reports/cohort_profiler/**)",
      "Write(/logs/**)"
    ],
    "deny": [
      "Network(*)",
      "Bash(*)",
      "Write(/data/**)"
    ]
  }
}
```

---

## Baseline Table Format

### baseline_table.csv

```csv
variable,overall,arm_control,arm_treatment,test_stat,p_value
n,1240,620,620,,
age_mean_sd,"56.2 (12.4)","55.8 (12.1)","56.6 (12.7)",t=0.98,0.33
age_median_iqr,"55 [48-64]","54 [47-63]","56 [49-65]",,
stage_I,520 (41.9%),265 (42.7%),255 (41.1%),,
stage_II,480 (38.7%),238 (38.4%),242 (39.0%),,
stage_III,240 (19.4%),117 (18.9%),123 (19.8%),χ²=0.52,0.77
node_negative,680 (54.8%),342 (55.2%),338 (54.5%),,
node_positive,560 (45.2%),278 (44.8%),282 (45.5%),χ²=0.04,0.84
grade_1,180 (14.5%),92 (14.8%),88 (14.2%),,
grade_2,620 (50.0%),308 (49.7%),312 (50.3%),,
grade_3,440 (35.5%),220 (35.5%),220 (35.5%),χ²=0.09,0.96
events,210 (16.9%),108 (17.4%),102 (16.5%),χ²=0.20,0.65
median_followup_years,"7.2 [5.8-9.1]","7.1 [5.7-9.0]","7.3 [5.9-9.2]",,
```

---

## Missingness Summary

### missingness.csv

```csv
variable,n_missing,pct_missing,action
age,0,0.0%,OK
stage,5,0.4%,OK
node_status,3,0.2%,OK
grade,12,1.0%,OK
event,0,0.0%,OK
time,0,0.0%,OK
biomarker_X,156,12.6%,WARN: High missingness
```

**Action Levels:**
- **OK:** <5% missing
- **WARN:** 5-15% missing
- **FAIL:** >15% missing (consider exclusion or imputation)

---

## Summary Report Template

```markdown
# Cohort Profile Summary

**Date:** 2025-11-09
**Cohort:** TAILORx-like
**Run ID:** 20251109_160000_cohpro

---

## Overview

- **Total N:** 1,240 patients
- **Arms:** Control (n=620), Treatment (n=620)
- **Events:** 210 (16.9%)
- **Median Follow-up:** 7.2 years [IQR: 5.8-9.1]

---

## Baseline Characteristics

**Age:**
- Mean: 56.2 years (SD 12.4)
- Range: 28-82 years
- Distribution: Near-normal, slight right skew

**Stage:**
- Stage I: 520 (41.9%)
- Stage II: 480 (38.7%)
- Stage III: 240 (19.4%)

**Node Status:**
- Negative: 680 (54.8%)
- Positive: 560 (45.2%)

**Grade:**
- Grade 1: 180 (14.5%)
- Grade 2: 620 (50.0%)
- Grade 3: 440 (35.5%)

**Balance:** No significant differences between arms (all p>0.05).

---

## Event Data

- **Total Events:** 210
- **Event Rate:** 16.9%
- **By Arm:**
  - Control: 108 (17.4%)
  - Treatment: 102 (16.5%)
  - Difference: -0.9% (NS, p=0.65)

**Event Time:**
- Median time to event: 4.8 years [IQR: 2.9-6.7]
- Earliest event: 6 months
- Latest event: 11.2 years

---

## Missingness

**Overall:** 1.2% missing across key variables (acceptable)

**By Variable:**
- Age, event, time: 0% missing ✓
- Stage: 0.4% missing (n=5) ✓
- Node status: 0.2% missing (n=3) ✓
- Grade: 1.0% missing (n=12) ✓
- Biomarker X: 12.6% missing (n=156) ⚠️

**Action:** Biomarker X has high missingness. Consider:
1. Exclude from primary analysis
2. Perform sensitivity analysis with complete cases
3. Use multiple imputation if needed

---

## Data Quality Checks

- [x] No duplicate patient IDs
- [x] All event times > 0
- [x] Censored times ≥ event times
- [x] No future dates
- [x] Stage/grade within valid ranges
- [x] Arm labels consistent

---

## Kaplan-Meier Summary

**Overall Survival:**
- 5-year: 87.3% [95% CI: 85.2-89.4%]
- 10-year: 83.1% [95% CI: 80.6-85.6%]

**By Arm:**
- Control 5-year: 86.8% [84.1-89.5%]
- Treatment 5-year: 87.8% [85.2-90.4%]
- Log-rank p=0.58 (NS)

---

## Readiness Assessment

**Status:** READY ✓

**Criteria Met:**
- [x] N ≥ 1,000 patients
- [x] Events ≥ 200
- [x] Median follow-up ≥ 5 years
- [x] Missing data <5% for key variables
- [x] No data quality issues
- [x] Arms well-balanced

**Recommendation:** Cohort is suitable for survival analysis. Proceed with metric computation.

---

## Figures

1. `age_hist.png` - Age distribution (near-normal)
2. `stage_bar.png` - Stage breakdown (balanced)
3. `km_overall.png` - Overall KM curve
4. `km_by_arm.png` - KM by treatment arm (no significant difference)

---

## Next Steps

1. Validate against schema (`schemas/cohort.json`)
2. Generate train/val/test splits
3. Proceed to Survival Metrics Engine
```

---

## Typical Usage

```bash
# Basic profiling
claude-code task --agent cohort_profiler \
  --prompt "Profile data/cohort.parquet: baseline table, missingness, KM curves, and readiness check."

# With schema validation
claude-code task --agent cohort_profiler \
  --prompt "Profile data/cohort.parquet with schema validation. Flag any schema violations or data quality issues."

# Stratified analysis
claude-code task --agent cohort_profiler \
  --prompt "Profile cohort stratified by stage (I/II/III). Generate separate KM curves and baseline tables for each stage."
```

---

## Figure Specifications

### Age Histogram
- Bins: 10-year intervals
- Overlay: Normal distribution curve
- Color: Neutral (gray/blue)

### Stage Bar Chart
- Horizontal bars
- Percentages labeled
- Sorted by stage (I, II, III)

### Kaplan-Meier Curves
- 95% confidence bands (shaded)
- At-risk table below plot
- Median survival line
- Log-rank test p-value displayed

---

## Schema Validation

If `schemas/cohort.json` provided, validate:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["patient_id", "age", "stage", "event", "time"],
  "properties": {
    "patient_id": {"type": "string", "pattern": "^SUBJ_[0-9]{6}$"},
    "age": {"type": "number", "minimum": 18, "maximum": 100},
    "stage": {"type": "integer", "enum": [1, 2, 3]},
    "node_status": {"type": "integer", "minimum": 0},
    "grade": {"type": "integer", "enum": [1, 2, 3]},
    "event": {"type": "integer", "enum": [0, 1]},
    "time": {"type": "number", "minimum": 0},
    "arm": {"type": "string", "enum": ["control", "treatment"]}
  }
}
```

Report violations in summary with severity (ERROR/WARN).

---

## Notes

- **Fast execution:** Should complete in <60 seconds for typical cohorts
- **Reusable:** Can be run on any cohort with standard survival columns
- **Audit-ready:** All figures and tables include timestamps and run IDs
- **PHI-safe:** Never include patient identifiers in outputs

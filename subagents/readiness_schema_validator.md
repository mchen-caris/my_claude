# Readiness & Schema Validator

**Category:** Function-Based Framework > Quality

---

## Purpose

Validate inputs against JSONSchema; check missingness, type drift, label drift, and leakage. Ensure data quality and analysis readiness before proceeding with metric computation.

---

## Inputs

- `data/**` - Cohort, predictions, and other data files
- `schemas/**` - JSONSchema definitions for validation

---

## Outputs

**Artifacts:**
- `reports/readiness_validator/run_<id>/readiness_report.md` - Comprehensive validation report
- `artifacts/readiness_validator/run_<id>/anomaly_flags.csv` - Specific data quality issues
- `artifacts/readiness_validator/run_<id>/schema_validation_results.json` - Detailed schema check results

---

## System Prompt

```
Validate cohort and prediction tables; flag any schema/label drift; ensure no patient overlaps across splits; output deterministic pass/fail and remediation steps.
```

---

## Security Permissions

```json
{
  "model": "claude-sonnet-4-5-20250929",
  "permissions": {
    "allow": [
      "Read(/data/**)",
      "Read(/schemas/**)",
      "Read(/splits/**)",
      "Read(/configs/**)",
      "Write(/artifacts/readiness_validator/**)",
      "Write(/reports/readiness_validator/**)",
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

## Anomaly Flags Table

### anomaly_flags.csv

```csv
category,severity,file,issue,location,count,remediation
SCHEMA,ERROR,data/cohort.parquet,Invalid stage value (4),row_328,1,Remove or recode row_328
SCHEMA,ERROR,predictions/model_v23/preds.parquet,Missing patient_id,rows_450-452,3,Add patient IDs
MISSING,WARN,data/cohort.parquet,High missingness for biomarker_X,column=biomarker_X,156 (12.6%),Consider exclusion or imputation
TYPE_DRIFT,WARN,data/cohort.parquet,Age changed from int to float,column=age,0,Document change
LABEL_DRIFT,ERROR,data/cohort.parquet,Event rate changed >5%,event column,+8.2%,Investigate cohort definition
LEAKAGE,CRITICAL,predictions/model_v99/preds.parquet,Future information detected,time_zero check,45 patients,BLOCK ANALYSIS
OVERLAP,ERROR,splits/train.csv + splits/test.csv,Patient overlap detected,SUBJ_000042 and 3 others,4 patients,Regenerate splits
DUPLICATE,WARN,data/cohort.parquet,Duplicate patient IDs,patient_id,2 duplicates,Deduplicate
RANGE,ERROR,predictions/model_v23/preds.parquet,Risk scores outside [0,1],risk_score column,12 rows,Clip or fix model output
TEMPORAL,CRITICAL,data/cohort.parquet,Negative event times,time column,3 patients,Fix time calculations
```

---

## Readiness Report

```markdown
# Data Readiness & Validation Report

**Date:** 2025-11-09
**Run ID:** 20251109_230000_readiness
**Cohort:** TAILORx_like
**Predictions:** 4 models validated

---

## Executive Summary

**Overall Status:** FAIL (3 CRITICAL issues, 2 ERRORS)

**Blocking Issues:** 3
- Label leakage detected in model_v99 predictions
- Event rate drift >5% (investigate cohort definition)
- Patient overlap in train/test splits

**Action Required:** Resolve blocking issues before proceeding to analysis.

---

## Schema Validation

### data/cohort.parquet

**Schema:** schemas/cohort.json (v1.2.0)

**Status:** FAIL

| Field | Expected Type | Actual Type | Required | Status | Issue |
|-------|---------------|-------------|----------|--------|-------|
| patient_id | string (pattern: ^SUBJ_[0-9]{6}$) | string | Yes | PASS | - |
| age | number (18-100) | number | Yes | PASS | - |
| stage | integer (1-3) | integer | Yes | ERROR | Invalid value '4' at row 328 |
| node_status | integer (≥0) | integer | Yes | PASS | - |
| grade | integer (1-3) | integer | Yes | PASS | - |
| event | integer (0 or 1) | integer | Yes | PASS | - |
| time | number (>0) | number | Yes | ERROR | Negative values at 3 rows |
| arm | string (enum) | string | No | PASS | - |

**Errors:** 2
1. **Invalid stage:** Row 328 has stage=4 (allowed: 1-3)
2. **Negative event times:** Rows 105, 672, 1108 have time <0

**Remediation:**
1. Investigate row 328: Likely data entry error. Recode or exclude.
2. Fix time calculations for negative values (likely date computation issue).

---

### predictions/model_v23/preds.parquet

**Schema:** schemas/predictions.json (v1.1.0)

**Status:** PASS

| Field | Expected Type | Status |
|-------|---------------|--------|
| patient_id | string | PASS |
| risk_score | number (0-1) | PASS |
| model_id | string | PASS |
| version | string | PASS |
| fold | integer | PASS |

**No schema violations.**

---

### predictions/model_v99/preds.parquet

**Schema:** schemas/predictions.json (v1.1.0)

**Status:** CRITICAL - BLOCK ANALYSIS

| Field | Status | Issue |
|-------|--------|-------|
| patient_id | PASS | - |
| risk_score | PASS | - |
| timestamp | CRITICAL | **Label leakage detected** |

**Issue:** Prediction timestamp is AFTER event date for 45 patients (3.6% of cohort). This indicates the model had access to future information.

**Examples:**
- SUBJ_000042: Event date 2020-03-15, Prediction timestamp 2020-06-22 (leak!)
- SUBJ_000128: Event date 2019-11-08, Prediction timestamp 2020-01-14 (leak!)

**Remediation:** DO NOT USE model_v99 predictions. Retrain with correct time-zero definition.

---

## Missingness Analysis

| Variable | N Total | N Missing | % Missing | Severity | Action |
|----------|---------|-----------|-----------|----------|--------|
| patient_id | 1240 | 0 | 0.0% | OK | None |
| age | 1240 | 0 | 0.0% | OK | None |
| stage | 1240 | 5 | 0.4% | OK | Acceptable |
| node_status | 1240 | 3 | 0.2% | OK | Acceptable |
| grade | 1240 | 12 | 1.0% | OK | Acceptable |
| event | 1240 | 0 | 0.0% | OK | None |
| time | 1240 | 0 | 0.0% | OK | None |
| biomarker_X | 1240 | 156 | 12.6% | WARN | High missingness |
| biomarker_Y | 1240 | 245 | 19.8% | FAIL | Exclude or impute |

**Summary:**
- **Core variables:** <2% missing (excellent)
- **Biomarker X:** 12.6% missing (acceptable with sensitivity analysis)
- **Biomarker Y:** 19.8% missing (too high; recommend exclusion)

**Recommendation:**
- Proceed with core variables
- Include Biomarker X with complete-case sensitivity analysis
- Exclude Biomarker Y from primary analysis

---

## Type Drift Detection

**Question:** Have data types changed from previous analyses?

| Variable | Previous Run | Current Run | Status | Impact |
|----------|--------------|-------------|--------|--------|
| age | int | float | WARN | Minimal (precision increased) |
| node_status | int | int | OK | No change |
| grade | int | int | OK | No change |

**Action:** Document age type change. Verify no loss of precision in downstream analyses.

---

## Label Drift Detection

**Question:** Has the event rate changed significantly from previous cohorts?

| Cohort | Event Rate | Change from Baseline | Status |
|--------|------------|---------------------|--------|
| Previous (2024-08) | 15.5% | Baseline | - |
| Current (2025-11) | 16.9% | +1.4% | OK |

**Threshold:** ±5% change triggers investigation

**Status:** OK (within acceptable range)

---

## Data Leakage Checks

### Time-Zero Validation

**Check:** Ensure predictions do not use information from after time-zero.

| Model | N Patients | Leakage Detected | Status |
|-------|-----------|------------------|--------|
| model_v23 | 1240 | 0 | PASS ✓ |
| model_v19_ref | 1240 | 0 | PASS ✓ |
| cox_baseline | 1240 | 0 | PASS ✓ |
| model_v99 | 1240 | 45 (3.6%) | **FAIL - CRITICAL** |

**Method:** Compare prediction timestamp vs (event date, time-zero).

**Remediation for model_v99:** Retrain with correct time-zero. Block from all analyses.

---

### Feature Leakage

**Check:** Ensure no features contain future information.

**Reviewed Features:**
- Clinical variables (age, stage, grade, nodes): Measured at baseline ✓
- Imaging features: Acquired before treatment start ✓
- Genomic scores: Derived from baseline samples ✓

**Status:** PASS (no feature leakage detected)

---

## Patient Overlap Across Splits

**Check:** Ensure train/val/test splits have no patient overlap.

| Split A | Split B | Overlap Count | Status |
|---------|---------|---------------|--------|
| train | val | 0 | PASS ✓ |
| train | test | 4 | **FAIL** |
| val | test | 0 | PASS ✓ |

**Overlapping Patients (train ∩ test):**
- SUBJ_000042
- SUBJ_000128
- SUBJ_000756
- SUBJ_001024

**Impact:** CRITICAL - Biased performance estimates

**Remediation:** Regenerate splits with fixed seed. Verify no overlap.

---

## Duplicate Patient IDs

| File | Duplicates Detected | Patient IDs | Action |
|------|---------------------|-------------|--------|
| data/cohort.parquet | 2 | SUBJ_000333 (2x), SUBJ_000891 (2x) | Deduplicate |
| predictions/model_v23/preds.parquet | 0 | - | OK |

**Remediation:** Investigate duplicates. Likely data entry error. Keep most recent record.

---

## Range & Validity Checks

### Risk Scores

| Model | Min | Max | Out of Range [0,1] | Status |
|-------|-----|-----|--------------------|--------|
| model_v23 | 0.02 | 0.89 | 0 | PASS ✓ |
| model_v19_ref | -0.03 | 0.95 | 12 (neg values) | ERROR |
| cox_baseline | 0.01 | 1.00 | 0 | PASS ✓ |

**Issue:** model_v19_ref has 12 negative risk scores (invalid).

**Remediation:** Clip to [0, 1] or fix model output transformation.

---

### Event Times

| Statistic | Value | Status |
|-----------|-------|--------|
| Min | -0.5 | ERROR (negative) |
| Max | 11.8 | OK |
| Mean | 7.2 | OK |
| Median | 7.0 | OK |

**Issue:** 3 patients with negative event times (rows 105, 672, 1108).

**Remediation:** Fix date calculations. Likely subtraction error (event before baseline).

---

## Readiness Checklist

| Criterion | Threshold | Actual | Status |
|-----------|-----------|--------|--------|
| N ≥ 1000 | 1000 | 1240 | ✓ PASS |
| Events ≥ 200 | 200 | 210 | ✓ PASS |
| Missing <5% (core vars) | 5% | 0.4% | ✓ PASS |
| No label leakage | 0 | 45 (model_v99) | ✗ FAIL |
| No split overlap | 0 | 4 | ✗ FAIL |
| Valid risk scores | 100% | 99.0% (model_v19) | ✗ FAIL |
| Valid event times | 100% | 99.8% | ✗ FAIL |

**Overall:** FAIL - 4 blocking issues

---

## Recommendations

### Immediate Actions (Block Analysis)

1. **Exclude model_v99** - Critical label leakage detected
2. **Regenerate splits** - Fix 4-patient overlap between train and test
3. **Fix model_v19_ref** - Clip or regenerate predictions with valid [0,1] range
4. **Fix event times** - Correct 3 negative values

### Optional Improvements

1. **Deduplicate cohort** - Remove 2 duplicate patient records
2. **Fix stage value** - Recode row 328 (stage=4 invalid)
3. **Document biomarker missingness** - Plan sensitivity analysis

### Validation on External Data

Before deployment, repeat this validation on external cohort to detect:
- Distribution shift
- Label prevalence change
- Feature availability

---

## Next Steps

1. Resolve all CRITICAL and ERROR issues
2. Re-run Readiness Validator to confirm PASS status
3. Only after PASS: Proceed to Survival Metrics Engine

**Do not proceed with analysis until all blocking issues are resolved.**
```

---

## Typical Usage

```bash
# Standard validation
claude-code task --agent readiness_schema_validator \
  --prompt "Validate data/cohort.parquet and all predictions in predictions/ against schemas. Flag leakage, overlap, and schema violations."

# Quick check before analysis
claude-code task --agent readiness_schema_validator \
  --quick \
  --prompt "Quick readiness check: schema validation, missingness, and leakage only."

# External cohort validation
claude-code task --agent readiness_schema_validator \
  --cohort external_cohort.parquet \
  --prompt "Validate external cohort against schemas. Check for distribution shift vs training cohort."
```

---

## Validation Categories

### 1. Schema Validation
- JSONSchema compliance
- Required fields present
- Data types correct
- Value ranges valid

### 2. Missingness
- Per-variable missing %
- Pattern of missingness (MAR, MCAR, MNAR)
- Impact on sample size

### 3. Type Drift
- Data type changes from previous runs
- Precision changes

### 4. Label Drift
- Event rate changes
- Class balance shifts

### 5. Leakage
- Temporal leakage (future information)
- Feature leakage (outcome in predictors)

### 6. Split Integrity
- No patient overlap across splits
- Stratification preserved

### 7. Range Checks
- Risk scores in [0, 1]
- Event times >0
- Clinical values in valid ranges

---

## Notes

- **Blocking issues:** CRITICAL and ERROR severity must be fixed before analysis
- **Automated:** Can be integrated into CI/CD pipeline
- **Deterministic:** Same inputs → same validation results
- **Reproducible:** All checks are scripted and versioned

# Metric Registry Curator

**Category:** Function-Based Framework > Design & Planning

---

## Purpose

Standardize metric definitions and confidence intervals so teams report the same fields. Lint existing metric outputs to ensure consistency across analyses.

---

## Inputs

- `configs/metric_registry.yaml` - Canonical metric definitions
- `artifacts/**/metrics_*.json` - Existing metric outputs to lint
- Optional: example metric files from prior runs

---

## Outputs

**Artifacts:**
- `reports/metric_registry/registry.md` - Human-readable registry
- `reports/metric_registry/lint_report.md` - Linting results
- `artifacts/metric_registry/canonical_examples/*.json` - Example templates

---

## System Prompt

```
Lint all metrics JSONs under `artifacts/**` to match `configs/metric_registry.yaml`. Normalize keys, ensure CI method is recorded (e.g., bootstrap B=2000 or Uno IPCW), add missing metadata.
```

---

## Security Permissions

```json
{
  "model": "claude-sonnet-4-5-20250929",
  "permissions": {
    "allow": [
      "Read(/configs/metric_registry.yaml)",
      "Read(/artifacts/**)",
      "Write(/reports/metric_registry/**)",
      "Write(/artifacts/metric_registry/**)",
      "Write(/logs/**)"
    ],
    "deny": [
      "Write(/data/**)",
      "Network(*)",
      "Bash(*)"
    ]
  }
}
```

---

## Metric Registry YAML

```yaml
# configs/metric_registry.yaml

version: "1.0.0"
last_updated: "2025-11-09"

metrics:
  harrell_cindex:
    full_name: "Harrell's Concordance Index"
    description: "Rank-based concordance for survival outcomes"
    version: "1.2.0"
    required_fields:
      - point
      - ci_lo
      - ci_hi
      - n
      - events
      - ci_method
      - ci_iterations
    optional_fields:
      - tied_method
      - fold_summary
    ci_method_allowed:
      - "bootstrap"
      - "jackknife"
    default_ci_iterations: 2000
    interpretation: "Higher is better; range [0, 1]; 0.5 = random"

  uno_cindex:
    full_name: "Uno's Time-Dependent Concordance Index"
    description: "IPCW-weighted concordance with censoring adjustment"
    version: "1.1.0"
    required_fields:
      - point
      - ci_lo
      - ci_hi
      - n
      - events
      - ci_method
      - truncation_percentile
    optional_fields:
      - fold_summary
    ci_method_allowed:
      - "bootstrap"
      - "perturbation"
    truncation_default: 90
    interpretation: "Higher is better; range [0, 1]"

  auc_t:
    full_name: "Time-Specific Area Under ROC Curve"
    description: "AUROC at specific time horizons with IPCW"
    version: "1.0.0"
    required_fields:
      - t
      - point
      - ci_lo
      - ci_hi
      - n
      - events_by_t
      - ci_method
    optional_fields:
      - sensitivity_at_spec
      - specificity_at_sens
    ci_method_allowed:
      - "bootstrap"
    interpretation: "Higher is better; range [0.5, 1.0]"

  calibration:
    full_name: "Calibration: Observed vs Expected"
    description: "Agreement between predicted and observed risk"
    version: "1.0.0"
    required_fields:
      - horizon
      - ece
      - calibration_slope
      - calibration_intercept
      - method
    optional_fields:
      - calibration_curve_csv
      - hosmer_lemeshow_p
    method_allowed:
      - "isotonic"
      - "platt"
      - "beta"
    interpretation: "Lower ECE is better; slope near 1.0 ideal"

  delta_cindex:
    full_name: "Delta C-index (Test vs Reference)"
    description: "Difference in C-index between models"
    version: "1.0.0"
    required_fields:
      - test_model_id
      - ref_model_id
      - delta
      - ci_lo
      - ci_hi
      - pvalue
      - ci_method
      - test_name
    optional_fields:
      - fold_wise_deltas
    ci_method_allowed:
      - "bootstrap_paired"
      - "permutation"
    interpretation: "Positive delta favors test model"

standard_metadata:
  required:
    - run_id
    - endpoint
    - cohort
    - model_id
    - version
    - provenance
  recommended:
    - analysis_date
    - analyst
    - notes
```

---

## Registry Document (Markdown)

```markdown
# Metric Registry

**Version:** 1.0.0
**Last Updated:** 2025-11-09

---

## Purpose

This registry defines standard metrics for survival analysis and model validation. All analyses should report metrics using these exact field names and methods.

---

## Metrics

### 1. Harrell's Concordance Index

**Name:** `harrell_cindex`
**Version:** 1.2.0
**Description:** Rank-based concordance for survival outcomes

**Required Fields:**
- `point` (float): Point estimate
- `ci_lo` (float): Lower 95% confidence bound
- `ci_hi` (float): Upper 95% confidence bound
- `n` (int): Total sample size
- `events` (int): Number of events
- `ci_method` (str): "bootstrap" or "jackknife"
- `ci_iterations` (int): Number of iterations (default: 2000)

**Optional Fields:**
- `tied_method` (str): How ties are handled
- `fold_summary` (dict): Cross-validation summary

**Interpretation:** Higher is better. Range [0, 1]. 0.5 indicates random prediction.

**Example:**
```json
{
  "metric": "harrell_cindex",
  "point": 0.712,
  "ci_lo": 0.685,
  "ci_hi": 0.739,
  "n": 1240,
  "events": 210,
  "ci_method": "bootstrap",
  "ci_iterations": 2000
}
```

---

### 2. Uno's Time-Dependent Concordance Index

**Name:** `uno_cindex`
**Version:** 1.1.0
**Description:** IPCW-weighted concordance with censoring adjustment

**Required Fields:**
- `point`, `ci_lo`, `ci_hi`, `n`, `events` (as above)
- `ci_method`: "bootstrap" or "perturbation"
- `truncation_percentile` (int): Event time truncation (default: 90)

**Interpretation:** Higher is better. Range [0, 1]. Accounts for censoring bias.

**Example:**
```json
{
  "metric": "uno_cindex",
  "point": 0.698,
  "ci_lo": 0.670,
  "ci_hi": 0.726,
  "n": 1240,
  "events": 210,
  "ci_method": "bootstrap",
  "ci_iterations": 2000,
  "truncation_percentile": 90
}
```

---

### 3. Time-Specific AUROC

**Name:** `auc_t`
**Version:** 1.0.0
**Description:** Area under ROC curve at specific time horizons

**Required Fields:**
- `t` (int): Time horizon in months
- `point`, `ci_lo`, `ci_hi`, `n` (as above)
- `events_by_t` (int): Events by time t
- `ci_method`: "bootstrap"

**Example:**
```json
{
  "metric": "auc_t",
  "t": 60,
  "point": 0.756,
  "ci_lo": 0.728,
  "ci_hi": 0.784,
  "n": 1240,
  "events_by_t": 142,
  "ci_method": "bootstrap",
  "ci_iterations": 2000
}
```

---

### 4. Calibration

**Name:** `calibration`
**Version:** 1.0.0
**Description:** Observed vs expected event rates

**Required Fields:**
- `horizon` (int): Time horizon in months
- `ece` (float): Expected Calibration Error
- `calibration_slope` (float): Calibration slope
- `calibration_intercept` (float): Calibration intercept
- `method` (str): "isotonic", "platt", or "beta"

**Interpretation:** Lower ECE is better. Slope near 1.0 and intercept near 0.0 indicate good calibration.

**Example:**
```json
{
  "metric": "calibration",
  "horizon": 60,
  "ece": 0.042,
  "calibration_slope": 0.96,
  "calibration_intercept": 0.08,
  "method": "isotonic"
}
```

---

### 5. Delta C-index

**Name:** `delta_cindex`
**Version:** 1.0.0
**Description:** Difference in C-index between test and reference models

**Required Fields:**
- `test_model_id` (str): Test model identifier
- `ref_model_id` (str): Reference model identifier
- `delta` (float): Difference (test - ref)
- `ci_lo`, `ci_hi` (float): 95% confidence interval
- `pvalue` (float): Two-sided p-value
- `ci_method`: "bootstrap_paired" or "permutation"
- `test_name` (str): Statistical test used

**Interpretation:** Positive delta favors test model. Significant if CI excludes 0 or p<0.05.

**Example:**
```json
{
  "metric": "delta_cindex",
  "test_model_id": "model_v23",
  "ref_model_id": "model_v19_ref",
  "delta": 0.024,
  "ci_lo": 0.008,
  "ci_hi": 0.040,
  "pvalue": 0.003,
  "ci_method": "bootstrap_paired",
  "test_name": "paired_bootstrap",
  "n": 1240
}
```

---

## Standard Metadata

Every metric file should include:

```json
{
  "run_id": "20251109_150000_eval",
  "endpoint": "distant_recurrence",
  "cohort": "TAILORx_like",
  "model_id": "model_v23",
  "version": "1.0.0",
  "analysis_date": "2025-11-09",
  "provenance": {
    "cohort_path": "data/cohort.parquet",
    "predictions_path": "predictions/model_v23/preds.parquet",
    "config": "configs/resolved_experiment.yaml"
  }
}
```

---

## Usage

### For Analysts

1. Reference this registry when computing metrics
2. Use exact field names
3. Include all required fields
4. Document CI method and iterations

### For Reviewers

1. Lint metric files against registry
2. Flag missing required fields
3. Verify CI methods are allowed
4. Check interpretation consistency
```

---

## Lint Report Template

```markdown
# Metric Registry Lint Report

**Date:** 2025-11-09
**Files Scanned:** 12
**Pass:** 9
**Fail:** 3

---

## Summary

| File | Status | Issues |
|------|--------|--------|
| artifacts/survival_metrics/model_v23/metrics_survival.json | PASS | 0 |
| artifacts/survival_metrics/model_v19/metrics_survival.json | FAIL | 2 |
| artifacts/model_compare/comparison_table.json | WARN | 1 |

---

## Failed Files

### artifacts/survival_metrics/model_v19/metrics_survival.json

**Status:** FAIL
**Issues:** 2

1. **Missing required field:** `ci_iterations`
   - **Metric:** harrell_cindex
   - **Fix:** Add `"ci_iterations": 2000`

2. **Invalid CI method:** `"ci_method": "unknown"`
   - **Metric:** uno_cindex
   - **Allowed:** ["bootstrap", "perturbation"]
   - **Fix:** Change to `"bootstrap"`

---

## Warnings

### artifacts/model_compare/comparison_table.json

**Status:** WARN
**Issues:** 1

1. **Missing recommended field:** `provenance`
   - **Fix:** Add provenance with cohort and config paths

---

## Recommendations

1. Update model_v19 metrics file with missing fields
2. Re-run with correct CI method for uno_cindex
3. Add provenance metadata to all comparison files
4. Consider standardizing across all team members

---

## Canonical Examples

Example files created in `artifacts/metric_registry/canonical_examples/`:
- `harrell_cindex_example.json`
- `uno_cindex_example.json`
- `auc_t_example.json`
- `calibration_example.json`
- `delta_cindex_example.json`
```

---

## Typical Usage

```bash
# Lint all metric files
claude-code task --agent metric_registry_curator \
  --prompt "Lint all metrics_*.json files in artifacts/** against metric_registry.yaml. Generate lint report and flag inconsistencies."

# Create canonical examples
claude-code task --agent metric_registry_curator \
  --prompt "Generate canonical example JSON files for all registered metrics. Include all required and optional fields with realistic values."

# Update registry
claude-code task --agent metric_registry_curator \
  --prompt "Add new metric 'rmst' (Restricted Mean Survival Time) to registry with required fields: point, ci_lo, ci_hi, horizon, n."
```

---

## Notes

- **Consistency:** Enforces cross-team standardization
- **Evolution:** Registry is versioned; breaking changes require major version bump
- **Validation:** Can integrate with CI/CD to auto-lint on commit
- **Documentation:** Registry serves as canonical reference for analysts

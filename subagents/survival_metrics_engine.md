# Survival Metrics Engine

**Category:** Function-Based Framework > Evaluation

---

## Purpose

Compute Harrell's C, Uno's C (IPCW), and AUC(t); aggregate across folds; emit standard JSON and CSVs. Provide rigorous, reproducible survival metrics with confidence intervals.

---

## Inputs

- `data/cohort.*` - Cohort with event/time columns
- `predictions/<model_id>/preds.parquet` - Model predictions
- `configs/resolved_experiment.yaml` - Experiment configuration with metric specs

---

## Outputs

**Artifacts:**
- `artifacts/survival_metrics/<model_id>/run_<id>/metrics_survival.json` - Standard metrics JSON
- `artifacts/survival_metrics/<model_id>/run_<id>/auc_t.csv` - Time-dependent AUC table
- `artifacts/survival_metrics/<model_id>/run_<id>/calibration_{h}.csv` - Calibration curves per horizon
- `reports/survival_metrics/<model_id>/summary.md` - Human-readable summary

---

## System Prompt

```
Compute Harrell C, Uno C with IPCW, and AUC(t) at 60 and 120 months using `preds.parquet` and cohort; provide bootstrap 95% CI (B=2000), and fold-level table with mean±sd.
```

---

## Security Permissions

```json
{
  "model": "claude-sonnet-4-5-20250929",
  "permissions": {
    "allow": [
      "Read(/data/cohort.*)",
      "Read(/predictions/**)",
      "Read(/configs/resolved_experiment.yaml)",
      "Write(/artifacts/survival_metrics/**)",
      "Write(/reports/survival_metrics/**)",
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

## Metrics Output (JSON)

### metrics_survival.json

```json
{
  "run_id": "20251109_180000_survmet",
  "model_id": "model_v23",
  "cohort": "TAILORx_like",
  "endpoint": "distant_recurrence",
  "n": 1240,
  "events": 210,
  "analysis_date": "2025-11-09",

  "harrell_cindex": {
    "point": 0.712,
    "ci_lo": 0.685,
    "ci_hi": 0.739,
    "ci_method": "bootstrap",
    "ci_iterations": 2000,
    "tied_method": "efron",
    "fold_summary": {
      "k": 5,
      "mean": 0.712,
      "std": 0.018,
      "range": [0.694, 0.735],
      "fold_values": [0.708, 0.694, 0.720, 0.735, 0.703]
    }
  },

  "uno_cindex": {
    "point": 0.698,
    "ci_lo": 0.670,
    "ci_hi": 0.726,
    "ci_method": "bootstrap",
    "ci_iterations": 2000,
    "truncation_percentile": 90,
    "fold_summary": {
      "k": 5,
      "mean": 0.698,
      "std": 0.021,
      "range": [0.680, 0.725]
    }
  },

  "auc_t": [
    {
      "t": 60,
      "point": 0.756,
      "ci_lo": 0.728,
      "ci_hi": 0.784,
      "n": 1240,
      "events_by_t": 142,
      "ci_method": "bootstrap",
      "ci_iterations": 2000
    },
    {
      "t": 120,
      "point": 0.721,
      "ci_lo": 0.691,
      "ci_hi": 0.751,
      "n": 1240,
      "events_by_t": 210,
      "ci_method": "bootstrap",
      "ci_iterations": 2000
    }
  ],

  "calibration": [
    {
      "horizon": 60,
      "ece": 0.042,
      "calibration_slope": 0.96,
      "calibration_intercept": 0.08,
      "method": "isotonic",
      "n_bins": 10
    },
    {
      "horizon": 120,
      "ece": 0.051,
      "calibration_slope": 0.93,
      "calibration_intercept": 0.11,
      "method": "isotonic",
      "n_bins": 10
    }
  ],

  "provenance": {
    "cohort_path": "data/cohort.parquet",
    "predictions_path": "predictions/model_v23/preds.parquet",
    "config_path": "configs/resolved_experiment.yaml",
    "config_hash": "abc123def456...",
    "metric_definitions": {
      "harrell_cindex": "1.2.0",
      "uno_cindex": "1.1.0"
    },
    "code_version": "1.3.0",
    "seed": 42
  }
}
```

---

## AUC(t) Table

### auc_t.csv

```csv
t,point,ci_lo,ci_hi,n,events_by_t,sensitivity_at_spec_90,specificity_at_sens_90
12,0.784,0.758,0.810,1240,38,0.65,0.68
24,0.772,0.745,0.799,1240,76,0.62,0.70
36,0.765,0.738,0.792,1240,104,0.61,0.71
48,0.760,0.732,0.788,1240,128,0.60,0.72
60,0.756,0.728,0.784,1240,142,0.59,0.73
72,0.748,0.719,0.777,1240,168,0.58,0.74
84,0.738,0.708,0.768,1240,184,0.56,0.75
96,0.730,0.699,0.761,1240,196,0.55,0.76
108,0.725,0.694,0.756,1240,204,0.54,0.76
120,0.721,0.691,0.751,1240,210,0.53,0.77
```

---

## Calibration Curve

### calibration_60.csv

```csv
bin,predicted_risk,observed_risk,n,events,ci_lo,ci_hi
1,0.05,0.04,124,5,0.01,0.07
2,0.10,0.09,124,11,0.04,0.14
3,0.12,0.11,124,14,0.06,0.16
4,0.14,0.15,124,19,0.09,0.21
5,0.16,0.16,124,20,0.10,0.22
6,0.18,0.17,124,21,0.11,0.23
7,0.21,0.22,124,27,0.15,0.29
8,0.25,0.24,124,30,0.17,0.31
9,0.30,0.32,124,40,0.24,0.40
10,0.42,0.44,124,55,0.35,0.53
```

---

## Summary Report Template

```markdown
# Survival Metrics Summary - model_v23

**Date:** 2025-11-09
**Run ID:** 20251109_180000_survmet
**Model:** model_v23
**Cohort:** TAILORx_like (n=1,240, events=210)

---

## Primary Metrics

### Harrell's C-index

**Point Estimate:** 0.712 [95% CI: 0.685-0.739]

**Interpretation:** Moderate discrimination. Model correctly orders 71.2% of patient pairs by risk.

**Cross-Validation:**
- Mean across 5 folds: 0.712 (SD 0.018)
- Range: [0.694, 0.735]
- Stability: Good (CV <3%)

---

### Uno's C-index (IPCW)

**Point Estimate:** 0.698 [95% CI: 0.670-0.726]

**Interpretation:** Time-dependent concordance accounting for censoring. Slightly lower than Harrell due to censoring adjustment.

**Truncation:** 90th percentile of event times (9.2 years)

---

## Time-Dependent Performance

### AUC(t) at Key Horizons

| Horizon | AUC | 95% CI | Events by t | Interpretation |
|---------|-----|--------|-------------|----------------|
| 5 years | 0.756 | [0.728, 0.784] | 142 | Good discrimination |
| 10 years | 0.721 | [0.691, 0.751] | 210 | Moderate discrimination |

**Trend:** AUC declines slightly over time (5y → 10y: -0.035), suggesting model is better at short-term prediction.

**Sensitivity/Specificity Trade-off:**
- At 90% specificity (5y): Sensitivity = 59%
- At 90% sensitivity (5y): Specificity = 73%

---

## Calibration

### 5-Year Horizon

**ECE:** 0.042 (excellent, <0.05 threshold)
**Slope:** 0.96 (near-ideal 1.0)
**Intercept:** 0.08 (slight over-prediction)

**Interpretation:** Model is well-calibrated. Predicted risks closely match observed event rates.

### 10-Year Horizon

**ECE:** 0.051 (good, marginally above threshold)
**Slope:** 0.93 (acceptable)
**Intercept:** 0.11 (moderate over-prediction)

**Interpretation:** Calibration degrades slightly at longer horizon. Consider recalibration for 10-year predictions.

---

## Bootstrap Confidence Intervals

**Method:** Stratified bootstrap with 2,000 iterations
**Seed:** 42 (fixed for reproducibility)
**Stratification:** By fold and event status

**Stability:**
- CI widths: Harrell ±0.027, Uno ±0.028 (acceptable)
- Bootstrap SE: Harrell 0.014, Uno 0.014

---

## Comparison to Literature

| Benchmark | Dataset | Harrell C | Reference |
|-----------|---------|-----------|-----------|
| Our model | TAILORx-like | 0.712 | This analysis |
| Genomic score | TAILORx | 0.68 | Sparano et al. NEJM 2018 |
| Clinical model | B42 | 0.71 | Cardoso et al. NEJM 2016 |

**Context:** Our model performance is competitive with published benchmarks.

---

## Data Quality

- [x] No missing predictions
- [x] Risk scores in valid range [0, 1]
- [x] Higher risk → higher event rate ✓
- [x] No leakage detected (time-zero validation passed)
- [x] Event times all positive

---

## Files

- **Metrics JSON:** `artifacts/survival_metrics/model_v23/run_<id>/metrics_survival.json`
- **AUC(t) table:** `artifacts/.../auc_t.csv`
- **Calibration (5y):** `artifacts/.../calibration_60.csv`
- **Calibration (10y):** `artifacts/.../calibration_120.csv`

---

## Next Steps

1. Run Model Comparison Planner to compare with baseline
2. Generate Subgroup Explorer for heterogeneity analysis
3. Use Visualization Studio for publication figures
```

---

## Typical Usage

```bash
# Compute metrics for single model
claude-code task --agent survival_metrics_engine \
  --model model_v23 \
  --prompt "Compute Harrell C, Uno C, AUC(t) at 60/120 months, and calibration. Use bootstrap with B=2000, seed=42."

# Batch processing for multiple models
claude-code task --agent survival_metrics_engine \
  --prompt "Compute survival metrics for all models in predictions/ directory. Use config from resolved_experiment.yaml."

# Custom horizons
claude-code task --agent survival_metrics_engine \
  --model model_v23 \
  --prompt "Compute AUC(t) at 12-month intervals from 12 to 120 months. Generate time-dependent performance curve."
```

---

## Key Algorithms

### Harrell's C-index
- Concordance for all comparable pairs
- Exact tie handling (Efron method)
- Bootstrap CI with 2,000 iterations

### Uno's C-index
- IPCW (Inverse Probability of Censoring Weighting)
- Truncation at 90th percentile of event times
- Handles informative censoring

### AUC(t)
- Time-specific binary classification at horizon t
- IPCW for censoring adjustment
- Bootstrap CI

### Calibration
- Isotonic regression (default) or Platt scaling
- 10 risk bins (deciles)
- ECE (Expected Calibration Error)

---

## Notes

- **Reproducibility:** Fixed seeds ensure identical results on re-run
- **Fold-level:** Metrics computed per fold, then aggregated
- **Metadata:** Full provenance in JSON for audit trail
- **Standards:** Follows Metric Registry conventions

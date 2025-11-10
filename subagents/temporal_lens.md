# Temporal Lens

**Category:** Function-Based Framework > Interpretation

---

## Purpose

Landmark and sliding-window performance; ΔAUC(t); early vs late risk insight. Understand how model performance changes over time and identify temporal patterns.

---

## Inputs

- `data/cohort.*` - Cohort with event/time data
- `predictions/<model_id>/preds.parquet` - Model predictions
- `configs/resolved_experiment.yaml` - Time horizons and windows

---

## Outputs

**Artifacts:**
- `artifacts/temporal_lens/run_<id>/auc_t_window.csv` - AUC by time window
- `artifacts/temporal_lens/run_<id>/landmark_analysis.csv` - Landmark performance at key timepoints
- `reports/temporal_lens/run_<id>/temporal_findings.md` - Interpretation and insights
- Figures:
  - `auc_t_curve.png` - AUC(t) over time with CI
  - `early_vs_late_delta.png` - Performance comparison by time window

---

## System Prompt

```
Compute AUC(t) curves with IPCW between 0–10y, report windows [0–5], [>5–10]. Provide interpretation bullets on where models diverge.
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
      "Write(/artifacts/temporal_lens/**)",
      "Write(/reports/temporal_lens/**)",
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

## AUC(t) Window Table

### auc_t_window.csv

```csv
window_start,window_end,window_label,auc,ci_lo,ci_hi,n_at_risk,events_in_window,interpretation
0,60,Early (0-5y),0.768,0.740,0.796,1240,142,Strong discrimination
60,120,Late (5-10y),0.698,0.665,0.731,1098,68,Moderate discrimination
0,36,Very Early (0-3y),0.782,0.752,0.812,1240,104,Strongest window
36,72,Mid (3-6y),0.745,0.715,0.775,1136,64,Good discrimination
72,120,Extended (6-10y),0.685,0.648,0.722,1072,46,Declining performance
```

**Insight:** Model performs best in early years (0-3y: AUC=0.782), with declining discrimination at extended follow-up (6-10y: AUC=0.685). Delta = -0.097.

---

## Landmark Analysis

### landmark_analysis.csv

```csv
landmark_time,horizon,n_at_risk,events_to_horizon,auc,ci_lo,ci_hi,interpretation
0,60,1240,142,0.756,0.728,0.784,Baseline prediction
12,60,1216,120,0.765,0.736,0.794,Slight improvement (survivors at 1y)
24,60,1188,96,0.772,0.742,0.802,Better discrimination (survivors at 2y)
36,60,1162,68,0.780,0.748,0.812,Strong (survivors at 3y)
60,120,1098,68,0.698,0.665,0.731,Late risk harder to predict
```

**Landmark Interpretation:** Conditional on survival to landmark time, model discrimination improves. Patients surviving 3 years have most accurate 5-year predictions (AUC=0.780).

---

## Temporal Findings Report

```markdown
# Temporal Analysis Findings - model_v23

**Date:** 2025-11-09
**Run ID:** 20251109_200000_templen
**Model:** model_v23
**Cohort:** TAILORx_like

---

## Executive Summary

Model v23 shows **time-dependent performance**:
- **Strongest:** Early prediction (0-3 years): AUC = 0.782
- **Moderate:** Mid-term (3-6 years): AUC = 0.745
- **Declining:** Extended (6-10 years): AUC = 0.685

**Key Finding:** Model is optimized for short-to-medium term risk (0-5 years). Performance degrades at extended follow-up.

---

## AUC(t) Curve Analysis

### Overall Trend

![AUC(t) Curve](auc_t_curve.png)

**Pattern:** Declining AUC(t) from 0.782 at 3 years to 0.685 at 10 years.

**Interpretation:**
- **Early events** (0-3y) are well-predicted (likely aggressive biology captured by imaging)
- **Late events** (>6y) are harder to predict (may involve different mechanisms)

**Clinical Implication:** Model is most useful for identifying patients at high short-term risk. Long-term predictions should be interpreted cautiously.

---

## Time Window Comparisons

| Window | Label | AUC | 95% CI | Events | Performance |
|--------|-------|-----|--------|--------|-------------|
| 0-3y | Very Early | 0.782 | [0.752, 0.812] | 104 | **Strong** |
| 3-6y | Mid | 0.745 | [0.715, 0.775] | 64 | Good |
| 6-10y | Extended | 0.685 | [0.648, 0.722] | 46 | Moderate |

**ΔAUC (Early vs Late):** -0.097 (p<0.001)

**Interpretation:** Statistically significant decline in discrimination over time. Model captures early risk factors but misses late-emerging patterns.

---

## Landmark Analysis

### Conditional Performance

**Question:** Among patients surviving to landmark time t, how well does the model predict events in the next 5 years?

| Landmark | N at Risk | AUC (next 5y) | Interpretation |
|----------|-----------|---------------|----------------|
| Baseline (t=0) | 1240 | 0.756 | Standard 5-year prediction |
| 1 year | 1216 | 0.765 | Slightly better (high-risk removed) |
| 2 years | 1188 | 0.772 | Improved (enriched lower-risk) |
| 3 years | 1162 | 0.780 | **Best** discrimination |

**Pattern:** Among survivors, model discrimination improves. This suggests:
1. Early events deplete the highest-risk patients
2. Remaining cohort is more homogeneous
3. Model's risk stratification is more effective in lower-risk population

---

## Early vs Late Risk Divergence

### Where Do Models Excel?

**Early Risk (0-5 years):**
- AUC = 0.768 [0.740-0.796]
- Events = 142
- **Biological Drivers:** Aggressive histology, high proliferation, unfavorable features visible on imaging

**Late Risk (5-10 years):**
- AUC = 0.698 [0.665-0.731]
- Events = 68
- **Biological Drivers:** Dormant micrometastases, late relapse biology (less captured by baseline imaging)

**Recommendation:**
- Use model for **treatment intensification decisions** in first 5 years
- Consider **alternative markers** (e.g., ctDNA, genomic) for late relapse prediction

---

## Comparison to Reference Model

| Window | model_v23 AUC | model_v19_ref AUC | ΔAUC | P-value |
|--------|---------------|-------------------|------|---------|
| 0-3y | 0.782 | 0.754 | +0.028 | 0.04 |
| 3-6y | 0.745 | 0.728 | +0.017 | 0.18 |
| 6-10y | 0.685 | 0.680 | +0.005 | 0.72 |

**Interpretation:** model_v23 improvement over reference is **concentrated in early years** (0-3y). Marginal benefit at extended follow-up.

---

## Clinical Implications

### Treatment Decision Timing

**High Confidence (0-5 years):**
- Escalation/de-escalation decisions well-supported
- AUC >0.75 in early window

**Lower Confidence (>5 years):**
- Predictions less reliable
- Consider serial reassessment with other biomarkers

### Risk Communication

**For Patients:**
- "This test is most accurate for predicting risk in the next 3-5 years."
- "For longer-term risk, we will continue monitoring with additional tests."

**For Clinicians:**
- Use model for short-term risk stratification
- Do not rely solely on baseline model for 10-year predictions

---

## Methodological Notes

**IPCW Adjustment:** All AUC(t) estimates use Inverse Probability of Censoring Weighting to account for censoring bias.

**Bootstrap CIs:** 2,000 iterations with stratified sampling by fold.

**Landmark Bias:** Landmark analysis conditions on survival to landmark time, preventing immortal time bias.

---

## Next Steps

1. **Subgroup Analysis:** Check if temporal pattern varies by age, stage, or biology
2. **Late Event Analysis:** Investigate features of late events (>5y) to inform model refinement
3. **Serial Predictions:** Explore updating predictions at 2-3 year landmarks with new clinical data
4. **External Validation:** Confirm temporal pattern in independent cohorts
```

---

## Typical Usage

```bash
# Standard temporal analysis
claude-code task --agent temporal_lens \
  --model model_v23 \
  --prompt "Compute AUC(t) from 0-10 years at monthly intervals. Generate early vs late window comparison and landmark analysis at 1y, 2y, 3y."

# Compare two models
claude-code task --agent temporal_lens \
  --models "model_v23,model_v19_ref" \
  --prompt "Compare temporal performance of both models. Identify where they diverge over time."

# Custom windows
claude-code task --agent temporal_lens \
  --windows "0-24,24-60,60-120" \
  --prompt "Analyze performance in custom time windows: 0-2y, 2-5y, 5-10y. Report ΔAUC between windows."
```

---

## Notes

- **IPCW:** Essential for unbiased time-dependent AUC estimation
- **Interpretation:** Declining AUC(t) is common; reflects biology, not model failure
- **Clinical value:** Identifies optimal time windows for model use
- **Landmark analysis:** Enables conditional risk assessment for survivors

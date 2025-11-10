# Effect Calibration & Explain

**Category:** Function-Based Framework > Interpretation

---

## Purpose

Calibration curves at horizons, decision thresholds, and simple explainers (e.g., partial dependence for clinical covariates). Assess model reliability and identify actionable risk thresholds.

---

## Inputs

- `data/cohort.*` - Cohort with clinical covariates
- `predictions/<model_id>/preds.parquet` - Model risk scores
- `configs/resolved_experiment.yaml` - Horizons and threshold specifications

---

## Outputs

**Artifacts:**
- `artifacts/effect_calibration/run_<id>/calibration_{h}.csv` - Calibration curve data per horizon
- `artifacts/effect_calibration/run_<id>/threshold_table.csv` - Risk thresholds with operating characteristics
- `artifacts/effect_calibration/run_<id>/partial_dependence.csv` - PD curves for key variables (optional)
- `reports/effect_calibration/run_<id>/calibration_note.md` - Interpretation
- Figures:
  - `calibration_5y.png`, `calibration_10y.png`
  - `threshold_tradeoff.png`

---

## System Prompt

```
Generate calibration at 60/120 months with smoothed curves and ECE; produce threshold table at target PPV/NPV for clinical review.
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
      "Write(/artifacts/effect_calibration/**)",
      "Write(/reports/effect_calibration/**)",
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

## Calibration Curve Data

### calibration_60.csv (5-year horizon)

```csv
bin,predicted_risk_mean,observed_risk,n,events,ci_lo,ci_hi,calibration_error
1,0.05,0.04,124,5,0.01,0.07,-0.01
2,0.09,0.09,124,11,0.04,0.14,0.00
3,0.12,0.13,124,16,0.07,0.19,+0.01
4,0.14,0.15,124,19,0.09,0.21,+0.01
5,0.16,0.16,124,20,0.10,0.22,0.00
6,0.18,0.17,124,21,0.11,0.23,-0.01
7,0.21,0.22,124,27,0.15,0.29,+0.01
8,0.25,0.26,124,32,0.18,0.34,+0.01
9,0.30,0.33,124,41,0.25,0.41,+0.03
10,0.42,0.44,124,55,0.35,0.53,+0.02
```

**Summary:**
- **ECE (Expected Calibration Error):** 0.042
- **Calibration Slope:** 0.96 (ideal=1.0)
- **Calibration Intercept:** 0.08 (ideal=0.0)
- **Max Error:** Bin 9 (+0.03)

**Interpretation:** Well-calibrated. Slight over-prediction in highest risk bin.

---

## Threshold Table

### threshold_table.csv

```csv
risk_threshold,sensitivity,specificity,ppv,npv,n_high_risk,events_high_risk,event_rate_high_risk,use_case
0.10,0.95,0.28,0.19,0.97,880,135,15.3%,High sensitivity screening
0.15,0.85,0.52,0.24,0.95,620,120,19.4%,Balanced
0.20,0.72,0.68,0.29,0.93,450,102,22.7%,Treatment escalation
0.25,0.58,0.78,0.34,0.91,320,82,25.6%,High specificity
0.30,0.45,0.85,0.40,0.89,220,64,29.1%,Very high specificity
0.35,0.32,0.90,0.45,0.88,150,45,30.0%,Extreme threshold
```

**Key Decision Points:**

- **Risk ≥0.20 (Treatment Escalation):** Sensitivity 72%, PPV 29%, identifies 450 high-risk patients
- **Risk ≥0.25 (High Specificity):** Sensitivity 58%, PPV 34%, identifies 320 high-risk patients
- **Risk <0.10 (Low Risk):** NPV 97%, confidently rules out high risk for 360 patients

---

## Calibration Note

```markdown
# Calibration & Threshold Analysis - model_v23

**Date:** 2025-11-09
**Run ID:** 20251109_220000_effcal
**Model:** model_v23
**Cohort:** TAILORx_like

---

## Executive Summary

**Calibration Status:** EXCELLENT (ECE <0.05 at both horizons)

**Key Findings:**
- Model predictions closely match observed risk (5y: ECE=0.042, 10y: ECE=0.051)
- Slight over-prediction in highest risk decile
- Proposed treatment threshold: Risk ≥0.20 (Sens=72%, PPV=29%)

---

## Calibration at 5 Years

![Calibration 5y](calibration_5y.png)

### Metrics

- **ECE:** 0.042 (excellent)
- **Calibration Slope:** 0.96 (near-ideal)
- **Calibration Intercept:** 0.08 (slight over-prediction)

### Interpretation

**Perfect calibration line (dashed gray):** Ideal agreement between predicted and observed

**Observed pattern:**
- Low-to-medium risk bins (0-0.25): Excellent agreement
- High risk bin (0.35-0.50): Slight over-prediction (+0.02 to +0.03)

**Clinical Implication:** Model is reliable for risk communication. Highest-risk predictions should be interpreted cautiously (may slightly overestimate).

---

## Calibration at 10 Years

![Calibration 10y](calibration_10y.png)

### Metrics

- **ECE:** 0.051 (good, marginally above 0.05 threshold)
- **Calibration Slope:** 0.93 (acceptable)
- **Calibration Intercept:** 0.11 (moderate over-prediction)

### Interpretation

Calibration degrades slightly at extended horizon (10y vs 5y):
- **Slope:** 0.93 (vs 0.96 at 5y) - slight under-confidence
- **Intercept:** 0.11 (vs 0.08 at 5y) - increased over-prediction

**Recommendation:** Consider **recalibration** for 10-year predictions using Platt scaling or isotonic regression if precision is critical.

---

## Decision Thresholds

### Clinical Use Cases

| Use Case | Threshold | Sensitivity | Specificity | PPV | NPV | N Flagged |
|----------|-----------|-------------|-------------|-----|-----|-----------|
| **Rule-out (low risk)** | <0.10 | 95% | 28% | 19% | 97% | 360 low-risk |
| **Balanced screening** | ≥0.15 | 85% | 52% | 24% | 95% | 620 screen+ |
| **Treatment escalation** | ≥0.20 | 72% | 68% | 29% | 93% | 450 escalate |
| **High specificity** | ≥0.25 | 58% | 78% | 34% | 91% | 320 escalate |

### Recommended Threshold

**Risk ≥0.20 for Treatment Escalation**

**Rationale:**
- Identifies 450 patients (36% of cohort)
- Event rate in high-risk group: 22.7% (vs 16.9% overall)
- Sensitivity: 72% (acceptable for escalation decision)
- PPV: 29% (1 in 3.4 high-risk patients will have event)

**Trade-off:** Misses 28% of events but avoids over-treatment of 640 lower-risk patients.

---

## Threshold Trade-off Curve

![Threshold Trade-off](threshold_tradeoff.png)

**Description:** Sensitivity vs Specificity at varying thresholds. Optimal balance at Risk=0.20.

---

## Operating Characteristics by Risk Group

### Risk Stratification

| Risk Group | Threshold | N | Events | Event Rate | 5y Survival | Clinical Action |
|------------|-----------|---|--------|------------|-------------|-----------------|
| **Very Low** | <0.10 | 360 | 18 | 5.0% | 95.0% | De-escalation candidate |
| **Low** | 0.10-0.15 | 260 | 30 | 11.5% | 88.5% | Standard care |
| **Intermediate** | 0.15-0.20 | 170 | 30 | 17.6% | 82.4% | Standard care |
| **High** | 0.20-0.30 | 320 | 88 | 27.5% | 72.5% | Escalation candidate |
| **Very High** | ≥0.30 | 130 | 44 | 33.8% | 66.2% | Escalation strongly consider |

**Gradient:** Clear dose-response relationship between model score and event rate.

---

## Calibration by Subgroup

**Question:** Is calibration consistent across subgroups?

| Subgroup | N | ECE @5y | Calibration Slope | Interpretation |
|----------|---|---------|-------------------|----------------|
| Age <50 | 450 | 0.038 | 0.98 | Excellent |
| Age ≥50 | 790 | 0.046 | 0.94 | Excellent |
| Node- | 680 | 0.051 | 0.92 | Good |
| Node+ | 560 | 0.040 | 0.97 | Excellent |
| Stage I | 520 | 0.062 | 0.88 | Acceptable |
| Stage II/III | 720 | 0.038 | 0.98 | Excellent |

**Pattern:** Calibration is strong across most subgroups. **Stage I** shows weaker calibration (ECE=0.062, slope=0.88), consistent with lower overall performance in this subgroup.

---

## Recalibration (If Needed)

### When to Recalibrate

- ECE >0.10 (poor calibration)
- Systematic over/under-prediction (slope <<0.9 or >>1.1)
- External validation on new population

### Methods

1. **Platt Scaling:** Logistic recalibration (fast, preserves rank)
2. **Isotonic Regression:** Non-parametric, flexible (used here)
3. **Beta Calibration:** Parametric alternative to Platt

### Example (10-year horizon)

**Before Recalibration:** ECE=0.051, Slope=0.93
**After Isotonic Regression:** ECE=0.028, Slope=0.99

**Recommendation:** Apply isotonic recalibration for 10-year predictions if precision is critical (e.g., shared decision-making).

---

## Partial Dependence (Explainability)

### Effect of Age on Risk

| Age | PD Risk (5y) | Interpretation |
|-----|--------------|----------------|
| 40 | 0.14 | Younger age → lower baseline risk (all else equal) |
| 50 | 0.16 | |
| 60 | 0.19 | |
| 70 | 0.22 | Older age → higher risk |

**Pattern:** Risk increases ~0.02 per decade (holding other features constant).

### Effect of Node Count

| Node Count | PD Risk (5y) | Interpretation |
|------------|--------------|----------------|
| 0 | 0.13 | Node-negative baseline |
| 1-3 | 0.19 | Moderate node involvement |
| >3 | 0.26 | Extensive nodal disease |

**Pattern:** Strong positive relationship between node count and risk.

---

## Clinical Risk Communication

### For Patients

**Low Risk (<0.10):**
- "Your risk of recurrence in the next 5 years is very low (less than 1 in 10)."
- "Standard treatment is likely sufficient."

**Intermediate Risk (0.15-0.20):**
- "Your risk is moderate (about 1 in 5 to 1 in 6)."
- "We will discuss treatment options and weigh benefits and side effects."

**High Risk (≥0.25):**
- "Your risk is higher (about 1 in 3 to 1 in 4)."
- "We recommend considering more intensive treatment."

### For Clinicians

**Model Confidence:**
- Calibration is strong (ECE <0.05)
- Predictions are reliable for decision-making
- High-risk predictions slightly overestimate (adjust by ~2-3%)

---

## Comparison to Reference Model

| Horizon | model_v23 ECE | model_v19_ref ECE | Improvement |
|---------|---------------|-------------------|-------------|
| 5 years | 0.042 | 0.053 | -0.011 (better) |
| 10 years | 0.051 | 0.068 | -0.017 (better) |

**Interpretation:** model_v23 is better calibrated than reference, especially at 10-year horizon.

---

## Recommendations

1. **Use Risk ≥0.20** as treatment escalation threshold (balances sensitivity and PPV)
2. **Communicate calibration quality** to clinicians (ECE <0.05 = reliable)
3. **Apply recalibration** for 10-year predictions if high precision needed
4. **Monitor Stage I** calibration in future analyses (weaker in current cohort)
5. **Validate thresholds** in external cohort before clinical deployment
```

---

## Typical Usage

```bash
# Standard calibration analysis
claude-code task --agent effect_calibration_explain \
  --model model_v23 \
  --horizons "60,120" \
  --prompt "Generate calibration curves at 5y and 10y. Compute ECE and produce threshold table with PPV/NPV at thresholds 0.15, 0.20, 0.25."

# Recalibration
claude-code task --agent effect_calibration_explain \
  --model model_v23 \
  --prompt "Apply isotonic recalibration to 10-year predictions. Report before/after ECE and slopes."

# Partial dependence
claude-code task --agent effect_calibration_explain \
  --model model_v23 \
  --variables "age,node_count,grade" \
  --prompt "Compute partial dependence curves for age, node count, and grade. Show how each affects predicted risk."
```

---

## Notes

- **Calibration:** Agreement between predicted probabilities and observed frequencies
- **ECE Benchmark:** <0.05 excellent, 0.05-0.10 good, >0.10 poor
- **Thresholds:** Choose based on clinical consequences of false positives/negatives
- **Recalibration:** Improves probability estimates without changing rank order

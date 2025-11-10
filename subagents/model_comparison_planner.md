# Model Comparison Planner

**Category:** Function-Based Framework > Evaluation

---

## Purpose

Normalize inputs from different teams and produce a single comparison book with ΔC, ranks, and tie-break rules. Enable fair, standardized model comparisons across teams.

---

## Inputs

- Multiple `predictions/<model_id>/preds.parquet` - Predictions from different models
- Corresponding `artifacts/survival_metrics/<model_id>/metrics_survival.json` - Metrics (if available)
- `configs/model_registry.yaml` - Model metadata (owner, contact, notes)
- `configs/resolved_experiment.yaml` - Experiment configuration

---

## Outputs

**Artifacts:**
- `artifacts/model_compare/run_<id>/comparison_table.csv` - Rows = model_id × endpoint × horizon
- `artifacts/model_compare/run_<id>/delta_table.csv` - Pairwise ΔC comparisons
- `artifacts/model_compare/run_<id>/league_table.csv` - Ranked models with tie-breaks
- `reports/model_compare/run_<id>/comparison_book.md` - Comprehensive comparison report
- Figures:
  - `delta_c_bar.png` - ΔC bar chart
  - `league_table.png` - Visual ranking

---

## System Prompt

```
Ingest predictions from listed `model_id`s; harmonize to a common schema; compute metrics if missing; generate a league table sorted by Uno C at 5y (primary), Harrell C as secondary. Apply tie-break rules and output a 'comparison_book.md' with consistent model naming.
```

---

## Security Permissions

```json
{
  "model": "claude-sonnet-4-5-20250929",
  "permissions": {
    "allow": [
      "Read(/predictions/**)",
      "Read(/artifacts/**)",
      "Read(/configs/model_registry.yaml)",
      "Read(/configs/resolved_experiment.yaml)",
      "Write(/artifacts/model_compare/**)",
      "Write(/reports/model_compare/**)",
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

## Model Registry

### configs/model_registry.yaml

```yaml
models:
  model_v23:
    full_name: "Deep Learning Model v23"
    owner: "Team Alpha"
    contact: "alpha@example.org"
    model_family: "ResNet-based survival"
    notes: "Multi-scale attention with clinical features"

  model_v19_ref:
    full_name: "Deep Learning Model v19 (Reference)"
    owner: "Team Alpha"
    contact: "alpha@example.org"
    model_family: "ResNet-based survival"
    notes: "Previous best model"

  cox_baseline:
    full_name: "Cox Proportional Hazards (Clinical)"
    owner: "Biostatistics Core"
    contact: "biostats@example.org"
    model_family: "Cox regression"
    notes: "Clinical variables only: age, stage, grade, nodes"

  genomic_score:
    full_name: "21-Gene Genomic Score"
    owner: "External Vendor"
    contact: "vendor@example.com"
    model_family: "Gene expression"
    notes: "Oncotype DX-like signature"

tie_break_rules:
  primary: "uno_c_5y"
  secondary: "harrell_c"
  tertiary: "auc_t_10y"
  quaternary: "calibration_ece_5y"
```

---

## Comparison Table

### comparison_table.csv

```csv
model_id,model_name,endpoint,harrell_c,harrell_ci_lo,harrell_ci_hi,uno_c_5y,uno_ci_lo,uno_ci_hi,auc_5y,auc_ci_lo,auc_ci_hi,auc_10y,ece_5y,ece_10y,n,events
model_v23,DL Model v23,distant_recurrence,0.712,0.685,0.739,0.698,0.670,0.726,0.756,0.728,0.784,0.721,0.042,0.051,1240,210
model_v19_ref,DL Model v19 (Ref),distant_recurrence,0.688,0.660,0.716,0.674,0.645,0.703,0.732,0.703,0.761,0.706,0.048,0.057,1240,210
cox_baseline,Cox Clinical,distant_recurrence,0.680,0.651,0.709,0.668,0.639,0.697,0.725,0.696,0.754,0.698,0.055,0.064,1240,210
genomic_score,21-Gene Score,distant_recurrence,0.682,0.654,0.710,0.670,0.641,0.699,0.728,0.699,0.757,0.700,0.052,0.060,1240,210
```

---

## Delta Table (Pairwise Comparisons)

### delta_table.csv

```csv
test_model_id,ref_model_id,metric,delta,ci_lo,ci_hi,pvalue,significant
model_v23,model_v19_ref,uno_c_5y,0.024,0.008,0.040,0.003,Yes
model_v23,cox_baseline,uno_c_5y,0.030,0.012,0.048,0.001,Yes
model_v23,genomic_score,uno_c_5y,0.028,0.010,0.046,0.002,Yes
model_v19_ref,cox_baseline,uno_c_5y,0.006,-0.012,0.024,0.52,No
genomic_score,cox_baseline,uno_c_5y,0.002,-0.016,0.020,0.82,No
```

---

## League Table (Ranked)

### league_table.csv

```csv
rank,model_id,model_name,uno_c_5y,uno_ci,harrell_c,auc_5y,ece_5y,owner,notes
1,model_v23,DL Model v23,0.698,[0.670-0.726],0.712,0.756,0.042,Team Alpha,Best overall
2,model_v19_ref,DL Model v19 (Ref),0.674,[0.645-0.703],0.688,0.732,0.048,Team Alpha,Previous best
3,genomic_score,21-Gene Score,0.670,[0.641-0.699],0.682,0.728,0.052,External,Commercial
4,cox_baseline,Cox Clinical,0.668,[0.639-0.697],0.680,0.725,0.055,Biostats,Baseline
```

---

## Comparison Book Template

```markdown
# Model Comparison Book - Run <id>

**Date:** 2025-11-09
**Run ID:** 20251109_190000_modcomp
**Cohort:** TAILORx_like (n=1,240, events=210)
**Endpoint:** Distant recurrence

---

## Executive Summary

**Models Compared:** 4
**Winner:** model_v23 (Deep Learning Model v23)
**Primary Metric:** Uno C-index @5y = 0.698 [0.670-0.726]
**Improvement vs Reference:** ΔC = +0.024 (p=0.003)

---

## League Table

| Rank | Model | Uno C @5y | Harrell C | AUC @5y | ECE @5y | Owner |
|------|-------|-----------|-----------|---------|---------|-------|
| 1 | DL Model v23 | 0.698 [0.670-0.726] | 0.712 | 0.756 | 0.042 | Team Alpha |
| 2 | DL Model v19 (Ref) | 0.674 [0.645-0.703] | 0.688 | 0.732 | 0.048 | Team Alpha |
| 3 | 21-Gene Score | 0.670 [0.641-0.699] | 0.682 | 0.728 | 0.052 | External |
| 4 | Cox Clinical | 0.668 [0.639-0.697] | 0.680 | 0.725 | 0.055 | Biostats |

**Tie-break Rules Applied:**
1. Primary: Uno C @5y (higher is better)
2. Secondary: Harrell C (higher is better)
3. Tertiary: AUC @10y (higher is better)
4. Quaternary: Calibration ECE @5y (lower is better)

---

## Pairwise Comparisons (vs Reference)

**Reference Model:** model_v19_ref

| Test Model | ΔC (Uno @5y) | 95% CI | P-value | Significant? |
|------------|--------------|--------|---------|--------------|
| model_v23 | +0.024 | [+0.008, +0.040] | 0.003 | Yes ✓ |
| genomic_score | -0.004 | [-0.022, +0.014] | 0.67 | No |
| cox_baseline | -0.006 | [-0.024, +0.012] | 0.52 | No |

**Interpretation:** Only model_v23 significantly outperforms the reference. Commercial and clinical baselines are not significantly different from the reference.

---

## Model Details

### 1. Deep Learning Model v23 (Rank 1)

**Owner:** Team Alpha (alpha@example.org)
**Family:** ResNet-based survival with multi-scale attention
**Features:** Histopathology images + clinical variables

**Performance:**
- Harrell C: 0.712 [0.685-0.739]
- Uno C @5y: 0.698 [0.670-0.726]
- AUC @5y: 0.756 [0.728-0.784]
- AUC @10y: 0.721 [0.691-0.751]
- Calibration ECE @5y: 0.042 (excellent)

**Strengths:**
- Best discrimination across all metrics
- Excellent calibration
- Significant improvement over reference (+0.024, p=0.003)

**Limitations:**
- Requires histopathology images (not always available)
- Higher computational cost

---

### 2. Deep Learning Model v19 (Rank 2, Reference)

**Owner:** Team Alpha
**Family:** ResNet-based survival

**Performance:**
- Harrell C: 0.688 [0.660-0.716]
- Uno C @5y: 0.674 [0.645-0.703]

**Notes:** Previous best model. Outperformed by v23 due to new attention mechanism.

---

### 3. 21-Gene Genomic Score (Rank 3)

**Owner:** External Vendor
**Family:** Gene expression signature

**Performance:**
- Harrell C: 0.682 [0.654-0.710]
- Uno C @5y: 0.670 [0.641-0.699]

**Notes:** Commercial benchmark. Competitive with clinical model but not with DL models.

---

### 4. Cox Clinical Baseline (Rank 4)

**Owner:** Biostatistics Core
**Family:** Cox proportional hazards

**Performance:**
- Harrell C: 0.680 [0.651-0.709]
- Uno C @5y: 0.668 [0.639-0.697]

**Notes:** Traditional clinical variables only. Serves as interpretable baseline.

---

## Harmonization Process

All models were harmonized to a common schema:

1. **Risk Score Direction:** Higher = higher risk (verified for all models)
2. **Time Origin:** Date of surgery (consistent)
3. **Endpoint:** Distant recurrence (same definition)
4. **Cohort:** Same 1,240 patients, same splits
5. **Metrics:** Computed with identical methods (bootstrap B=2000, seed=42)

**Recomputed Metrics:** 1 model (genomic_score) had missing Uno C; recomputed from predictions.

---

## Statistical Tests

**Method:** Paired bootstrap (B=2,000) with stratification by fold
**Seed:** 42 (reproducible)
**Significance Level:** α=0.05 (two-sided)

**Multiple Testing:** Not adjusted (pairwise comparisons are exploratory)

---

## Figures

### Delta C-index Bar Chart

![Delta C](delta_c_bar.png)

**Description:** ΔC (test vs reference) with 95% CI error bars. model_v23 is the only significant improvement.

### League Table Visualization

![League Table](league_table.png)

**Description:** Ranked models with primary metric (Uno C @5y) and confidence intervals.

---

## Folder and File Conventions

**Predictions:**
- Path: `predictions/<model_id>/preds.parquet`
- Required columns: `patient_id`, `risk_score`

**Metrics:**
- Path: `artifacts/survival_metrics/<model_id>/run_<id>/metrics_survival.json`
- Standard format per Metric Registry

**Registry:**
- Path: `configs/model_registry.yaml`
- Metadata: owner, contact, family, notes

---

## Recommendations

1. **Advance model_v23** to next validation stage (meets performance criteria)
2. **Retire genomic_score** (not cost-effective vs DL models)
3. **Maintain cox_baseline** (interpretable, low-cost reference)
4. **Archive model_v19_ref** (superseded by v23)

---

## Next Steps

1. External validation of model_v23 on independent cohort
2. Subgroup analysis to assess heterogeneity
3. Decision curve analysis for clinical utility
4. Prepare Decision Brief for leadership
```

---

## Typical Usage

```bash
# Compare all models in predictions/ directory
claude-code task --agent model_comparison_planner \
  --prompt "Compare all models in predictions/. Use Uno C @5y as primary metric. Generate league table and delta comparisons vs model_v19_ref."

# Custom reference model
claude-code task --agent model_comparison_planner \
  --ref cox_baseline \
  --prompt "Compare all DL models vs cox_baseline. Generate pairwise delta table and comparison book."

# Specific model subset
claude-code task --agent model_comparison_planner \
  --models "model_v23,model_v19_ref,cox_baseline" \
  --prompt "Compare only specified models. Apply tie-break rules from model_registry.yaml."
```

---

## Notes

- **Harmonization:** Ensures apples-to-apples comparisons
- **Recompute:** If metrics are missing or outdated, agent recomputes from predictions
- **Naming:** Uses model_registry.yaml for consistent, human-readable names
- **Tie-breaks:** Explicit rules prevent ambiguous rankings
- **Provenance:** All comparisons traceable to source predictions and configs

# Subgroup Explorer

**Category:** Function-Based Framework > Interpretation

---

## Purpose

Pre-specified subgroups (age, stage, nodal, RS strata, etc.), interaction screens, and forest plots. Identify effect heterogeneity and subgroup-specific performance.

---

## Inputs

- `data/cohort.*` - Cohort with subgroup columns (age, stage, grade, nodes, etc.)
- `predictions/<model_id>/preds.parquet` - Model predictions
- `configs/resolved_experiment.yaml` - Pre-specified subgroup definitions

---

## Outputs

**Artifacts:**
- `artifacts/subgroup_explorer/run_<id>/subgroup_metrics.csv` - Harrell/Uno C per subgroup
- `artifacts/subgroup_explorer/run_<id>/interaction_tests.csv` - Interaction test results
- `artifacts/subgroup_explorer/run_<id>/forest_plot.png` - Visual forest plot
- `reports/subgroup_explorer/run_<id>/insights.md` - Narrative interpretation

---

## System Prompt

```
For defined subgroups (age≤50, >50; stage I/II/III; RS low/int/high), compute Uno C and interaction tests. Output forest plot and short narrative on effect heterogeneity.
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
      "Write(/artifacts/subgroup_explorer/**)",
      "Write(/reports/subgroup_explorer/**)",
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

## Subgroup Metrics Table

### subgroup_metrics.csv

```csv
subgroup,n,events,event_rate,harrell_c,harrell_ci_lo,harrell_ci_hi,uno_c_5y,uno_ci_lo,uno_ci_hi,delta_vs_overall
Overall,1240,210,16.9%,0.712,0.685,0.739,0.698,0.670,0.726,0.000
Age<50,450,65,14.4%,0.742,0.698,0.786,0.728,0.682,0.774,+0.030
Age>=50,790,145,18.4%,0.695,0.662,0.728,0.681,0.647,0.715,-0.017
Node-,680,88,12.9%,0.688,0.648,0.728,0.675,0.634,0.716,-0.023
Node+,560,122,21.8%,0.726,0.691,0.761,0.712,0.675,0.749,+0.014
Grade1-2,800,120,15.0%,0.705,0.672,0.738,0.692,0.658,0.726,-0.006
Grade3,440,90,20.5%,0.724,0.682,0.766,0.710,0.666,0.754,+0.012
Stage_I,520,42,8.1%,0.670,0.615,0.725,0.658,0.601,0.715,-0.040
Stage_II,480,98,20.4%,0.718,0.679,0.757,0.704,0.663,0.745,+0.006
Stage_III,240,70,29.2%,0.732,0.679,0.785,0.718,0.663,0.773,+0.020
```

---

## Interaction Tests

### interaction_tests.csv

```csv
covariate,interaction_stat,df,pvalue,significant,interpretation
age_group,4.18,1,0.041,Yes,Significant heterogeneity by age
node_status,1.78,1,0.182,No,No significant interaction
grade,0.98,1,0.322,No,No significant interaction
stage,2.54,2,0.281,No,No significant interaction (trend)
```

---

## Forest Plot

**forest_plot.png** displays:
- Harrell C or Uno C per subgroup
- Point estimates with 95% CI error bars
- Overall estimate as reference line
- Subgroups sorted by effect size
- Sample sizes and event counts labeled

---

## Insights Report

```markdown
# Subgroup Analysis Insights - model_v23

**Date:** 2025-11-09
**Run ID:** 20251109_210000_subgrp
**Model:** model_v23
**Cohort:** TAILORx_like

---

## Executive Summary

**Heterogeneity Detected:** Significant age interaction (p=0.041)

**Key Findings:**
- **Younger patients (Age<50):** Best performance (Uno C=0.728 vs 0.698 overall)
- **Stage I:** Weakest performance (Uno C=0.658, Delta=-0.040)
- **Node-positive:** Good performance (Uno C=0.712)

**Recommendation:** Model is most effective in younger, node-positive, higher-stage patients.

---

## Performance by Subgroup

### Age

| Subgroup | N | Events | Uno C @5y | 95% CI | Delta vs Overall |
|----------|---|--------|-----------|--------|------------------|
| Age <50 | 450 | 65 (14.4%) | 0.728 | [0.682, 0.774] | +0.030 |
| Age ≥50 | 790 | 145 (18.4%) | 0.681 | [0.647, 0.715] | -0.017 |

**Interaction Test:** p=0.041 (significant)

**Interpretation:**
- Model performs **significantly better** in younger patients
- Possible reasons:
  1. Younger patients have more aggressive, imaging-visible tumors
  2. Older patients may have more indolent, late-relapsing disease
  3. Biology differences captured by histopathology

**Clinical Implication:** Consider age-stratified risk thresholds.

---

### Node Status

| Subgroup | N | Events | Uno C @5y | 95% CI | Delta vs Overall |
|----------|---|--------|-----------|--------|------------------|
| Node- | 680 | 88 (12.9%) | 0.675 | [0.634, 0.716] | -0.023 |
| Node+ | 560 | 122 (21.8%) | 0.712 | [0.675, 0.749] | +0.014 |

**Interaction Test:** p=0.182 (not significant)

**Interpretation:**
- No significant interaction, but trend toward better performance in node-positive
- Higher event rate in node+ provides more discriminative signal
- Both groups exceed clinically useful threshold (C>0.65)

---

### Grade

| Subgroup | N | Events | Uno C @5y | 95% CI | Delta vs Overall |
|----------|---|--------|-----------|--------|------------------|
| Grade 1-2 | 800 | 120 (15.0%) | 0.692 | [0.658, 0.726] | -0.006 |
| Grade 3 | 440 | 90 (20.5%) | 0.710 | [0.666, 0.754] | +0.012 |

**Interaction Test:** p=0.322 (not significant)

**Interpretation:** Consistent performance across grade. Model works well in both low and high grade tumors.

---

### Stage

| Subgroup | N | Events | Uno C @5y | 95% CI | Delta vs Overall |
|----------|---|--------|-----------|--------|------------------|
| Stage I | 520 | 42 (8.1%) | 0.658 | [0.601, 0.715] | -0.040 |
| Stage II | 480 | 98 (20.4%) | 0.704 | [0.663, 0.745] | +0.006 |
| Stage III | 240 | 70 (29.2%) | 0.718 | [0.663, 0.773] | +0.020 |

**Interaction Test:** p=0.281 (not significant, but trend)

**Interpretation:**
- **Stage I limitation:** Weakest performance (C=0.658)
  - Low event rate (8.1%) reduces discriminative power
  - May reflect floor effect (most Stage I patients do well)
  - Model adds limited value beyond clinical stage
- **Stage II/III:** Stronger performance where clinical need is greatest

**Clinical Implication:** Model is most useful in Stage II/III. Limited added value in Stage I.

---

## Forest Plot

![Forest Plot](forest_plot.png)

**Visual Summary:**
- Age <50: Strongest performance (rightmost)
- Stage I: Weakest performance (leftmost)
- Overall estimate: Reference line at 0.698

---

## Interaction Summary

| Covariate | P-value | Significant? | Clinical Relevance |
|-----------|---------|--------------|-------------------|
| Age | 0.041 | Yes | Consider age-stratified thresholds |
| Node status | 0.182 | No | Universal applicability |
| Grade | 0.322 | No | Universal applicability |
| Stage | 0.281 | No | Note Stage I limitation |

**Multiple Testing Note:** 4 tests performed. Bonferroni-adjusted α=0.0125. Only age interaction remains significant after adjustment.

---

## Positive Subgroups (Best Performance)

**Where model excels:**

1. **Age <50** (C=0.728): Young, likely aggressive biology
2. **Stage III** (C=0.718): High-risk, high event rate
3. **Node+** (C=0.712): Lymph node involvement
4. **Grade 3** (C=0.710): High grade tumors

**Pattern:** Model performs best in higher-risk subgroups with aggressive features.

---

## Watch-Out Subgroups (Weaker Performance)

**Where model is limited:**

1. **Stage I** (C=0.658): Low event rate, limited added value
2. **Node-** (C=0.675): Lower risk, harder to discriminate
3. **Age ≥50** (C=0.681): Older patients, possibly different biology

**Pattern:** Lower-risk subgroups show weaker discrimination.

---

## Clinical Recommendations

### Patient Selection for Model Use

**High Value:**
- Age <50
- Node-positive
- Stage II/III
- Grade 3

**Limited Value:**
- Stage I (clinical stage sufficient)
- Very low-risk profiles (most will do well anyway)

### Risk Communication

**Age-Stratified Interpretation:**
- Younger patients: Higher confidence in predictions
- Older patients: Interpret with caution, consider complementary markers

### Decision Thresholds

Consider age-specific or stage-specific thresholds rather than one-size-fits-all.

---

## Sensitivity Analyses

**Minimum Events Threshold:** All subgroups meet ≥20 events (range: 42-145)

**Sample Size Adequacy:**
- Smallest subgroup: Stage III (n=240) - adequate
- All CIs reasonably narrow (widths 0.08-0.12)

**Missing Data:** <2% missing for subgroup variables (excluded from subgroup-specific analyses)

---

## Comparison to Reference Model

| Subgroup | model_v23 Uno C | model_v19_ref Uno C | Δ | P-value |
|----------|-----------------|---------------------|---|---------|
| Age <50 | 0.728 | 0.690 | +0.038 | 0.02 |
| Age ≥50 | 0.681 | 0.665 | +0.016 | 0.24 |
| Node+ | 0.712 | 0.688 | +0.024 | 0.08 |
| Stage I | 0.658 | 0.655 | +0.003 | 0.86 |

**Insight:** model_v23 improvement is **concentrated in younger patients** (Age<50: Δ=+0.038, p=0.02).

---

## Next Steps

1. **Validate age interaction** in external cohort
2. **Investigate Stage I** performance (may need specialized model)
3. **Develop age-stratified thresholds** for clinical use
4. **Explore biological differences** between age groups (gene expression, immune features)
```

---

## Typical Usage

```bash
# Standard subgroup analysis
claude-code task --agent subgroup_explorer \
  --model model_v23 \
  --prompt "Compute subgroup metrics for age, stage, grade, node status as defined in resolved_experiment.yaml. Test for interactions and generate forest plot."

# Custom subgroups
claude-code task --agent subgroup_explorer \
  --subgroups "age<40,age_40-60,age>60" \
  --prompt "Analyze three age strata. Compute Uno C per group and test for trend."

# Comparison across models
claude-code task --agent subgroup_explorer \
  --models "model_v23,model_v19_ref" \
  --prompt "Compare subgroup performance of both models. Identify which subgroups show greatest improvement in v23."
```

---

## Notes

- **Pre-specification:** Subgroups must be declared in experiment plan (avoid post-hoc)
- **Interaction tests:** Formal tests for effect modification
- **Multiple testing:** Consider Bonferroni or FDR adjustment for multiple subgroups
- **Sample size:** Ensure adequate events per subgroup (≥20 recommended)

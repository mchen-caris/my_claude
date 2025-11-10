# Figure Pack Builder

**Category:** Function-Based Framework > Communication

---

## Purpose

Assemble a clean figure set with captions and sources for slide or manuscript use. Create publication-ready figure packages with consistent styling and documentation.

---

## Inputs

- Figure files from `artifacts/**/` (PNG, PDF)
- `metrics_*.json` - Metrics for captions
- Optional: `configs/figure_pack_spec.yaml` - Figure selection and ordering

---

## Outputs

**Artifacts:**
- `reports/figure_pack/run_<id>/figure_list.md` - Figure catalog with captions
- `reports/figure_pack/run_<id>/figures/` - Copied figures with stable names
- `reports/figure_pack/run_<id>/figure_pack.zip` - Zipped package for sharing
- `reports/figure_pack/run_<id>/powerpoint_template.pptx` (optional) - Pre-populated slides

---

## System Prompt

```
Assemble a figure pack: KM overall and by subgroup, calibration at 5y/10y, delta-C bar chart. Write short captions and source paths for each.
```

---

## Security Permissions

```json
{
  "model": "claude-sonnet-4-5-20250929",
  "permissions": {
    "allow": [
      "Read(/artifacts/**)",
      "Read(/reports/**)",
      "Read(/configs/figure_pack_spec.yaml)",
      "Write(/reports/figure_pack/**)",
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

## Figure List Template

```markdown
# Figure Pack - Model v23 Validation

**Date:** 2025-11-09
**Run ID:** 20251109_240000_figpack
**Purpose:** Publication and presentation figures for model v23 validation study

---

## Figure 1: Kaplan-Meier Curve - Overall Cohort

**File:** `figures/fig1_km_overall.png` (300 DPI, 8"×6")

**Caption:**
Kaplan-Meier curve for distant recurrence-free survival in the TAILORx-like validation cohort (n=1,240). Shaded area represents 95% confidence interval. Median survival: 8.4 years [95% CI: 7.2-9.6]. At-risk table shown below x-axis.

**Source:** `artifacts/visualization_studio/run_20251109_170000/km_overall.png`

**Use Case:** Manuscript Figure 1, Methods section

**Alternative Versions:**
- `fig1_km_overall_bw.png` - Black and white for print
- `fig1_km_overall_hi_res.pdf` - Vector format for publication

---

## Figure 2: Kaplan-Meier Stratified by Risk Group

**File:** `figures/fig2_km_risk_groups.png` (300 DPI, 8"×6")

**Caption:**
Kaplan-Meier curves stratified by model-predicted risk (median split). High-risk group (n=620, red) vs low-risk group (n=620, blue). Log-rank test: p<0.001. 5-year distant recurrence-free survival: Low risk 92.3% [89.8-94.8%] vs High risk 82.3% [79.2-85.4%].

**Source:** `artifacts/visualization_studio/run_20251109_170000/km_by_risk_group.png`

**Use Case:** Manuscript Figure 2, Results section

**Statistical Note:** Stratification performed at median risk score (0.17). Events: Low risk n=54 (8.7%), High risk n=156 (25.2%).

---

## Figure 3: Model Performance - League Table

**File:** `figures/fig3_league_table.png` (300 DPI, 10"×4")

**Caption:**
Comparison of model performance metrics. Four models evaluated: Deep learning v23, Deep learning v19 (reference), 21-gene genomic score, and Cox clinical baseline. Primary metric: Uno C-index @5y (time-dependent concordance with IPCW). Error bars represent 95% confidence intervals from 2,000 bootstrap iterations. Model v23 significantly outperforms all others (p<0.01 for all pairwise comparisons).

**Source:** `artifacts/model_compare/run_20251109_190000/league_table.png`

**Use Case:** Manuscript Figure 3, Results section or Presentation Slide 5

---

## Figure 4: Calibration Plot - 5 Year Horizon

**File:** `figures/fig4_calibration_5y.png` (300 DPI, 6"×6")

**Caption:**
Calibration plot comparing predicted vs observed 5-year distant recurrence risk for model v23. Dashed gray line indicates perfect calibration. Blue curve shows smoothed calibration (LOESS). Expected Calibration Error (ECE) = 0.042 (excellent; <0.05 threshold). Risk stratified into 10 deciles. Model predictions closely match observed event rates across full risk spectrum.

**Source:** `artifacts/effect_calibration/run_20251109_220000/calibration_5y.png`

**Use Case:** Manuscript Figure 4, Results section

**Interpretation Note:** Slight over-prediction in highest risk decile (predicted 0.42 vs observed 0.44).

---

## Figure 5: Calibration Plot - 10 Year Horizon

**File:** `figures/fig5_calibration_10y.png` (300 DPI, 6"×6")

**Caption:**
Calibration plot at 10-year horizon. ECE = 0.051 (good). Calibration degrades slightly at extended follow-up compared to 5-year (ECE 0.042 vs 0.051). Calibration slope 0.93 (ideal = 1.0).

**Source:** `artifacts/effect_calibration/run_20251109_220000/calibration_10y.png`

**Use Case:** Manuscript Supplementary Figure 1

---

## Figure 6: Forest Plot - Subgroup Analysis

**File:** `figures/fig6_forest_subgroups.png` (300 DPI, 8"×10")

**Caption:**
Forest plot showing Uno C-index @5y for model v23 across pre-specified subgroups. Point estimates with 95% confidence intervals. Vertical dashed line represents overall cohort performance (C=0.698). Significant age interaction detected (p=0.041). Model performs best in younger patients (Age<50: C=0.728) and is limited in Stage I disease (C=0.658).

**Source:** `artifacts/subgroup_explorer/run_20251109_210000/forest_plot.png`

**Use Case:** Manuscript Figure 5, Results section or Supplementary Figure 2

**Statistical Note:** Subgroups pre-specified in analysis plan. Minimum 20 events per subgroup enforced.

---

## Figure 7: Delta C-index Bar Chart

**File:** `figures/fig7_delta_cindex.png` (300 DPI, 8"×5")

**Caption:**
Improvement in Uno C-index @5y for model v23 compared to clinical baseline across validation cohorts. Error bars represent 95% confidence intervals from paired bootstrap (B=2,000). Asterisks denote statistical significance: * p<0.05, ** p<0.01. Model v23 shows significant improvement of ΔC=+0.030 [+0.012, +0.048] on TAILORx-like cohort.

**Source:** `artifacts/model_compare/run_20251109_190000/delta_c_bar.png`

**Use Case:** Manuscript Figure 6 or Presentation Slide 8

---

## Figure 8: Time-Dependent AUC Curve

**File:** `figures/fig8_auc_t_curve.png` (300 DPI, 8"×6")

**Caption:**
Time-dependent AUROC from 0 to 10 years with 95% confidence bands (shaded). AUC(t) declines from 0.782 at 3 years to 0.721 at 10 years, indicating stronger performance for short-to-medium term prediction. Vertical lines mark key horizons (5y, 10y).

**Source:** `artifacts/temporal_lens/run_20251109_200000/auc_t_curve.png`

**Use Case:** Manuscript Supplementary Figure 3

**Clinical Implication:** Model optimized for 0-5 year risk prediction. Long-term predictions (>5y) should be interpreted cautiously.

---

## Supplementary Figures (Additional)

### S1: Cohort Flow Diagram
**File:** `figures/figS1_cohort_flow.png`
**Caption:** CONSORT-style flow diagram showing cohort derivation, exclusions, and final analysis set.
**Source:** `reports/cohort_profiler/run_20251109_160000/cohort_flow.png`

### S2: Baseline Characteristics Table (Figure)
**File:** `figures/figS2_baseline_table.png`
**Caption:** Baseline characteristics stratified by treatment arm. No significant differences (all p>0.05).
**Source:** `artifacts/cohort_profiler/run_20251109_160000/baseline_table.png`

### S3: Early vs Late Risk Windows
**File:** `figures/figS3_early_vs_late.png`
**Caption:** Model performance comparison: Early risk window (0-5y) vs Late risk window (5-10y). AUC 0.768 vs 0.698, Delta=-0.070 (p<0.001).
**Source:** `artifacts/temporal_lens/run_20251109_200000/early_vs_late_delta.png`

---

## Figure Specifications

All figures meet publication standards:

| Specification | Value |
|---------------|-------|
| Resolution | 300 DPI minimum |
| Format | PNG (raster), PDF (vector) |
| Color Space | RGB for digital, CMYK for print |
| Font Size | ≥10pt for all text |
| Line Width | ≥1.5pt |
| Color Palette | Colorblind-safe (Okabe-Ito or similar) |

---

## File Organization

```
figures/
├── fig1_km_overall.png
├── fig1_km_overall_bw.png
├── fig1_km_overall_hi_res.pdf
├── fig2_km_risk_groups.png
├── fig3_league_table.png
├── fig4_calibration_5y.png
├── fig5_calibration_10y.png
├── fig6_forest_subgroups.png
├── fig7_delta_cindex.png
├── fig8_auc_t_curve.png
├── figS1_cohort_flow.png
├── figS2_baseline_table.png
└── figS3_early_vs_late.png
```

---

## Manuscript Integration

### Suggested Figure Placement

**Main Text Figures (8):**
1. KM overall → Methods/Results
2. KM by risk group → Results
3. League table → Results
4. Calibration 5y → Results
5. Forest plot subgroups → Results/Discussion
6. Delta C-index → Results
7-8. (Optional) Reserve for additional key findings

**Supplementary Figures (3+):**
- S1: Cohort flow
- S2: Baseline table
- S3: Temporal analysis
- S4+: Additional sensitivity analyses

---

## Presentation Slides (Optional)

### PowerPoint Template Included

**Slide 1:** Title
- Model v23 Validation Results
- TAILORx-like Cohort (n=1,240)

**Slide 2:** Study Design
- Cohort flow diagram (figS1)

**Slide 3:** Primary Results
- League table (fig3)
- Key metrics highlighted

**Slide 4:** Performance by Subgroup
- Forest plot (fig6)
- Age interaction emphasized

**Slide 5:** Calibration
- Calibration 5y (fig4)
- ECE = 0.042 (excellent)

**Slide 6:** Survival Curves
- KM by risk group (fig2)
- Clear separation (p<0.001)

**Slide 7:** Temporal Analysis
- AUC(t) curve (fig8)
- Best in 0-5y window

**Slide 8:** Delta C-index
- Improvement vs baseline (fig7)
- ΔC = +0.030 (p=0.001)

**Slide 9:** Summary
- Significant improvement
- Excellent calibration
- Recommend for clinical pilot

---

## Usage Notes

### For Manuscript Submission

1. Use high-res PNG (300 DPI) or PDF (vector)
2. Check journal's figure requirements (size, format, color)
3. Include source data if required
4. Cite figure pack run ID in methods for reproducibility

### For Presentations

1. Use PNG files (universally compatible)
2. Adjust size if needed (slides typically 10"×7.5")
3. Check color contrast for projector display
4. Test on presentation laptop before talk

### For Posters

1. Use PDF (vector) for best quality at large sizes
2. Increase font sizes (≥14pt minimum)
3. Simplify legends if needed for readability
4. Test print quality before final production

---

## Reproduction

All figures can be regenerated from source:

```bash
# Regenerate all figures
cd artifacts/
python regenerate_figures.py --run_id 20251109_170000

# Verify figure pack integrity
sha256sum -c figure_pack_checksums.txt
```

**Checksums:** `figure_pack_checksums.txt` included for verification.

---

## License & Attribution

**Data:** TAILORx-like cohort (de-identified)
**Code:** Open source (MIT License)
**Figures:** CC-BY 4.0 (attribution required)

**Attribution:**
```
Figures generated using Claude Code Agent Framework v1.3.0.
Source: artifacts/[agent]/run_[id]/
```

---

## Contact

**Questions or Requests:**
- Figure modifications: figurepack@example.org
- High-resolution exports: figurepack@example.org
- Colorblind-safe palettes: figurepack@example.org

---

*Figure pack generated by Figure Pack Builder Agent on 2025-11-09. All figures meet ICMJE and journal publication standards.*
```

---

## Typical Usage

```bash
# Standard figure pack
claude-code task --agent figure_pack_builder \
  --runid 20251109_143022 \
  --prompt "Assemble figure pack: KM overall, KM by risk, calibration 5y/10y, forest plot, delta C-index. Include captions and sources."

# Manuscript-specific
claude-code task --agent figure_pack_builder \
  --spec "configs/manuscript_figures.yaml" \
  --prompt "Build figure pack per manuscript spec: 6 main figures + 4 supplementary. Export as 300 DPI PNG and vector PDF."

# Presentation pack
claude-code task --agent figure_pack_builder \
  --format "presentation" \
  --prompt "Create presentation figure pack optimized for slides: larger fonts, high contrast, 10×7.5 inch size. Include PowerPoint template."
```

---

## Figure Pack Specification

### configs/figure_pack_spec.yaml

```yaml
figure_pack:
  name: "Model v23 Validation Figures"
  purpose: "Manuscript and presentation"
  date: "2025-11-09"

main_figures:
  - id: "fig1"
    source: "artifacts/visualization_studio/*/km_overall.png"
    title: "Kaplan-Meier Overall"
    format: ["png_300dpi", "pdf_vector"]

  - id: "fig2"
    source: "artifacts/visualization_studio/*/km_by_risk_group.png"
    title: "KM by Risk Group"
    format: ["png_300dpi", "pdf_vector"]

  - id: "fig3"
    source: "artifacts/model_compare/*/league_table.png"
    title: "Model Comparison"
    format: ["png_300dpi"]

supplementary_figures:
  - id: "figS1"
    source: "reports/cohort_profiler/*/cohort_flow.png"
    title: "Cohort Flow Diagram"

output:
  directory: "reports/figure_pack/run_<id>/figures/"
  zip_file: "figure_pack.zip"
  catalog: "figure_list.md"
  powerpoint: true
```

---

## Notes

- **Stable naming:** fig1, fig2, etc. (not tied to agent-specific names)
- **Multiple formats:** PNG for compatibility, PDF for quality
- **Captions:** Comprehensive, journal-ready
- **Sources:** Traceable to original artifacts
- **Reproducible:** Can regenerate entire pack from run IDs

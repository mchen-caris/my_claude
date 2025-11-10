# Decision Brief Writer

**Category:** Function-Based Framework > Communication

---

## Purpose

Convert artifacts into a two-page decision brief for leadership with key charts and a one-line recommendation. Synthesize complex analysis results into clear, actionable insights.

---

## Inputs

- Any `artifacts/**` - Metrics, comparisons, subgroup analyses
- Any `reports/**` - Agent-generated reports
- `metrics_*.json` - Structured metric outputs

---

## Outputs

**Artifacts:**
- `reports/decision_brief/run_<id>/decision_brief.md` - Concise leadership brief
- `reports/decision_brief/run_<id>/decision_brief.pdf` (optional) - PDF export if converter available

---

## System Prompt

```
Write a concise brief: context → methods → results (primary metric, CI, ΔC) → risks → decision. Include league table and top 3 charts. Keep it to two pages.
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
      "Write(/reports/decision_brief/**)",
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

## Decision Brief Template

```markdown
# Decision Brief: Model v23 Validation for Clinical Use

**Date:** 2025-11-09
**Prepared by:** Decision Brief Writer
**For:** Clinical Leadership Team
**Decision Required:** Advance model_v23 to clinical pilot?

---

## Recommendation

**ADVANCE model_v23 to limited clinical pilot**

Model v23 significantly outperforms current standard (ΔC=+0.024, p=0.003) with excellent calibration and no safety concerns. Recommend pilot deployment in Stage II/III patients ages <50.

---

## Context

**Question:** Should we deploy model_v23 for distant recurrence risk prediction in HR+/HER2- breast cancer?

**Current Standard:** Clinical model (Cox regression: age, stage, grade, nodes)

**Proposed:** Deep learning model with histopathology imaging + clinical features

**Decision Criteria:** ΔC-index ≥+0.02 with 95% CI lower bound >0, no subgroup harm

---

## Methods

**Cohort:** TAILORx-like patients (n=1,240, events=210)
**Validation:** 5-fold cross-validation, bootstrap CI (B=2,000)
**Primary Metric:** Uno C-index @5y (time-dependent with IPCW)
**Comparison:** vs clinical baseline and previous model (v19)

---

## Results

### Primary Outcome

| Model | Uno C @5y | 95% CI | ΔC vs Baseline | P-value |
|-------|-----------|--------|----------------|---------|
| **DL Model v23** | **0.698** | [0.670-0.726] | **+0.030** | **0.001** |
| DL Model v19 (ref) | 0.674 | [0.645-0.703] | +0.006 | 0.52 |
| Clinical Baseline | 0.668 | [0.639-0.697] | - | - |

**Interpretation:** v23 significantly outperforms both baseline (+0.030) and previous model (+0.024).

---

### League Table

| Rank | Model | Uno C | Harrell C | AUC @5y | Calibration ECE |
|------|-------|-------|-----------|---------|-----------------|
| 1 | DL v23 | 0.698 | 0.712 | 0.756 | 0.042 |
| 2 | DL v19 | 0.674 | 0.688 | 0.732 | 0.048 |
| 3 | Clinical | 0.668 | 0.680 | 0.725 | 0.055 |

---

### Calibration

**5-Year Predictions:**
- ECE: 0.042 (excellent, <0.05 threshold)
- Slope: 0.96 (near-perfect)
- Predicted risks closely match observed event rates

**Implication:** Model is reliable for patient communication.

---

### Subgroup Performance

**Strongest Performance:**
- Age <50: C=0.728 (best discrimination)
- Node+: C=0.712
- Stage II/III: C=0.704-0.718

**Limitation:**
- Stage I: C=0.658 (weaker; clinical stage likely sufficient)

**Interaction Test:** Significant age heterogeneity (p=0.041)

---

## Key Figures

### Figure 1: Delta C-index vs Baseline

![Delta C](../model_compare/run_20251109_190000/delta_c_bar.png)

v23 shows significant improvement; v19 does not.

---

### Figure 2: Calibration at 5 Years

![Calibration](../effect_calibration/run_20251109_220000/calibration_5y.png)

Excellent agreement between predicted and observed risk.

---

### Figure 3: Kaplan-Meier by Risk Group

![KM Risk](../visualization_studio/run_20251109_170000/km_by_risk_group.png)

Clear separation between model-predicted high and low risk groups (log-rank p<0.001).

---

## Risks & Limitations

### Technical

1. **Stage I Limited Value:** Model adds minimal benefit in early-stage disease (C=0.658)
2. **Long-term Prediction:** Performance declines at 10-year horizon (AUC=0.721 vs 0.756 at 5y)
3. **Requires Histopathology:** Need digitized whole-slide images (infrastructure requirement)

### Clinical

1. **Age Heterogeneity:** Stronger performance in younger patients (Age<50); may need age-stratified thresholds
2. **Validation Cohort:** TAILORx-like population; performance in other populations TBD
3. **Treatment Landscape:** Model trained on adjuvant-treated patients; may not generalize to neoadjuvant

### Operational

1. **Implementation Cost:** Requires slide scanning and compute infrastructure (~$50K setup + $20/case)
2. **Turnaround Time:** Current 24-48h; may delay clinical decisions
3. **Training Required:** Oncologists need guidance on interpreting AI risk scores

---

## Mitigation Strategies

| Risk | Mitigation |
|------|------------|
| Stage I limited value | Restrict initial pilot to Stage II/III |
| Age heterogeneity | Develop age-specific guidance, validate in external cohorts |
| Infrastructure cost | Start with limited pilot (50 cases), assess ROI before scaling |
| Turnaround time | Prioritize workflow optimization, target <24h for pilot |

---

## Financial Impact

**Cost per Case:**
- Slide scanning: $15
- Compute (inference): $3
- QC review: $2
- **Total:** $20/case

**Potential Benefit:**
- Avoid over-treatment: 10-15% of high-risk patients could de-escalate → $5,000 savings/patient
- Improve outcomes: Early intensification in true high-risk → ~2% improved 5y survival

**Break-even:** ~4 appropriate de-escalations per 100 cases

---

## Approval & Timeline

### Pilot Proposal

**Phase 1 (Months 1-3):** Limited pilot
- 50 cases (Stage II/III, Age<60)
- Parallel to standard care (shadow mode)
- Assess workflow, turnaround time, clinician feedback

**Phase 2 (Months 4-6):** Expanded pilot
- 200 cases
- Decision support mode (inform but don't dictate treatment)
- Collect outcomes data

**Phase 3 (Months 7-12):** Clinical integration
- Full deployment if pilot successful
- Prospective validation
- Reimbursement strategy

---

## Recommendation

**ADVANCE model_v23 to limited clinical pilot (Phase 1)**

**Rationale:**
- Significant performance improvement (ΔC=+0.030, p=0.001)
- Excellent calibration (reliable predictions)
- No subgroup harm detected
- Manageable risks with mitigation strategies

**Conditions:**
- Restrict to Stage II/III patients (initially)
- Target Age <60 (strongest performance)
- Shadow mode (parallel to standard care)
- Collect prospective outcomes and clinician feedback

**Expected Decision Date:** 2025-11-20 (Director review)

**Next Steps:**
1. Director approval (by 2025-11-15)
2. IRB submission for pilot (by 2025-11-22)
3. Workflow implementation (by 2025-12-01)
4. Pilot launch (by 2025-12-15)

---

## Approvals

| Role | Name | Date | Status |
|------|------|------|--------|
| Director, Computational Pathology | [Name] | YYYY-MM-DD | PENDING |
| Chief Medical Officer | [Name] | YYYY-MM-DD | PENDING |
| Chief Quality Officer | [Name] | YYYY-MM-DD | PENDING |

---

## Appendices

**A. Full Metrics Report:** `reports/survival_metrics/model_v23/summary.md`

**B. Comparison Book:** `reports/model_compare/run_20251109_190000/comparison_book.md`

**C. Subgroup Analysis:** `reports/subgroup_explorer/run_20251109_210000/insights.md`

**D. Calibration Details:** `reports/effect_calibration/run_20251109_220000/calibration_note.md`

**E. Governance Manifest:** `governance/20251109_143022_a7b3f9c/manifest.yaml`

---

**Contact:** decision-brief@example.org

---

*This brief synthesizes results from 5 specialized agents (Survival Metrics, Model Comparison, Subgroup Explorer, Calibration, Visualization) into a leadership-ready format.*
```

---

## Key Elements of Effective Brief

### 1. Executive Summary (Top)
- One-sentence recommendation
- Key metric with CI
- Risk/benefit balance

### 2. Context (1-2 paragraphs)
- What decision is being made?
- Why now?
- What are the alternatives?

### 3. Methods (1 paragraph)
- Just enough to establish credibility
- Sample size, validation approach
- Primary metric

### 4. Results (Tables + 3 figures max)
- Primary outcome table
- League table if comparing multiple models
- Key visualizations (calibration, KM, forest plot)

### 5. Risks (Bulleted)
- Technical limitations
- Clinical caveats
- Operational concerns

### 6. Recommendation (Clear call)
- ADVANCE / REFINE / NO CHANGE
- Rationale tied to evidence
- Conditions or next steps

### 7. Approval Block
- Stakeholder sign-off
- Timeline for decision

---

## Typical Usage

```bash
# Standard decision brief
claude-code task --agent decision_brief_writer \
  --runid 20251109_143022 \
  --prompt "Create decision brief for model_v23 validation. Include recommendation, key charts, risks, and approval workflow. Limit to 2 pages."

# Custom focus
claude-code task --agent decision_brief_writer \
  --focus "cost_benefit" \
  --prompt "Write decision brief emphasizing financial impact and ROI. Include cost per case and break-even analysis."

# Multi-model comparison
claude-code task --agent decision_brief_writer \
  --models "model_v23,model_v19_ref,cox_baseline" \
  --prompt "Create comparative brief for 3 models. Recommend which should advance to clinical use."
```

---

## Audience Considerations

### For Clinical Leadership
- Focus on clinical utility
- Emphasize safety and performance
- Simple language (minimize jargon)
- Clear recommendation

### For Executive Leadership
- Focus on strategic fit
- Emphasize ROI and timeline
- Risk management
- Competitive positioning

### For Regulatory
- Focus on validation rigor
- Emphasize reproducibility
- Safety profile
- Compliance (FDA, CLIA, etc.)

### For Technical Review
- More methodological detail
- Algorithm specifics
- Validation metrics
- Comparison to literature

---

## Notes

- **Two-page limit:** Forces clarity and prioritization
- **Visual:** 3 figures maximum (choose most impactful)
- **Actionable:** Must have clear recommendation
- **Self-contained:** Should be understandable without appendices
- **Traceable:** All claims link to upstream artifacts

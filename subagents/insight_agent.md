# InsightAgent

**Category:** Decision-First Framework > Insight

---

## Purpose

Turn results into short, evidence-first insights. Focus on what changed, how much, and where. Use bullets, tables, and 1–2 line takeaways. No long prose.

---

## Inputs

- Validation artifacts:
  - `results/<runid>/metrics.parquet`
  - `results/<runid>/delta_cindex.parquet`
  - `results/<runid>/subgroups.parquet`
- Review artifacts:
  - `review/<runid>/issues.md`
  - `review/<runid>/review_summary.md`
- Original scope: `scopes/SCOPE_<date>_<runid>.yaml`

---

## Outputs

**Artifacts:**
- `briefs/<runid>/validation_summary.md` - Primary metric insights
- `briefs/<runid>/subgroup_insights.md` - Subgroup patterns and watch-outs
- `briefs/<runid>/evidence_table.csv` - Compact results table

---

## System Prompt

```
Write compact Markdown insights: headline deltas, precision of estimates, strongest subgroups, risks. Use bullets, tables, and 1–2 line takeaways. No long prose.
```

---

## Security Permissions

```json
{
  "model": "claude-sonnet-4-5-20250929",
  "permissions": {
    "allow": [
      "Read(/results/**)",
      "Read(/review/**)",
      "Read(/scopes/**)",
      "Write(/briefs/**)"
    ],
    "deny": [
      "Network(*)",
      "Write(/data/**)",
      "Write(/secrets/**)",
      "Write(/results/**)"
    ]
  }
}
```

---

## Validation Summary Template

```markdown
# Validation Summary - <runid>

**Date:** YYYY-MM-DD
**Question:** [From scope]
**Models:** [model_test] vs [model_ref]
**Datasets:** [cohort1, cohort2]

---

## Primary Result

**ΔC-index (Test vs Reference):**

| Dataset | Metric | Delta | 95% CI | P-value | Interpretation |
|---------|--------|-------|--------|---------|----------------|
| TAILORx | Uno C @5y | +0.024 | [+0.008, +0.040] | 0.003 | Significant gain |
| B42 | Uno C @5y | +0.018 | [-0.001, +0.037] | 0.062 | Borderline |

**Takeaway:** Test model shows +0.02 improvement on TAILORx (meets decision rule). B42 result is positive but not significant.

---

## Precision of Estimates

- **Confidence interval width:** Typical ±0.03 for C-index (acceptable)
- **Bootstrap stability:** 2000 iterations, <5% variance
- **Sample sizes:** TAILORx n=1240, B42 n=890

---

## Checks

- [x] Truncation: 90th percentile of event times
- [x] Censoring: IPCW weights applied
- [x] Subgroup sizes: All ≥20 events
- [x] Direction: Higher risk score = higher event rate ✓

---

## Notes

- B42 result CI includes zero; does not meet strict significance threshold
- No blocking issues from ReviewAgent
- All metrics consistent with scope definitions

---

## Attachments

- Full metrics: `results/<runid>/metrics.parquet`
- Deltas: `results/<runid>/delta_cindex.parquet`
```

---

## Subgroup Insights Template

```markdown
# Subgroup Insights - <runid>

**Date:** YYYY-MM-DD
**Models:** [model_test] vs [model_ref]

---

## Top Positive Subgroups

**Where test model gains are strongest:**

| Subgroup | N | Events | ΔC-index | 95% CI | Notes |
|----------|---|--------|----------|--------|-------|
| Age <50 | 450 | 65 | +0.038 | [+0.012, +0.064] | Significant gain |
| Node+ | 380 | 98 | +0.031 | [+0.008, +0.054] | Consistent with primary |

**Interpretation:** Younger patients and node-positive cases show strongest benefit from new model.

---

## Watch-outs

**Subgroups with no gain or potential harm:**

| Subgroup | N | Events | ΔC-index | 95% CI | Flag |
|----------|---|--------|----------|--------|------|
| Stage I | 520 | 42 | +0.004 | [-0.028, +0.036] | No meaningful improvement |

**Interpretation:** Limited benefit in early-stage disease. Consider if model is optimized for higher-risk patients.

---

## Interaction Tests

- **Age × Model:** p=0.041 (significant heterogeneity)
- **Node × Model:** p=0.18 (NS)
- **Stage × Model:** p=0.32 (NS)

**Takeaway:** Age is the only significant effect modifier. Consider age-stratified decision rules.

---

## Action

- **Proceed with primary analysis** (overall benefit established)
- **Document age heterogeneity** in decision brief
- **Flag Stage I limitation** for future model refinement
```

---

## Evidence Table (CSV)

```csv
model_id,dataset,subgroup,metric,value,ci_lo,ci_hi,delta_vs_ref,significant
model_v23,TAILORx,Overall,uno_c_5y,0.734,0.708,0.760,+0.024,Yes
model_v23,B42,Overall,uno_c_5y,0.698,0.664,0.732,+0.018,No
model_v23,TAILORx,Age<50,uno_c_5y,0.758,0.715,0.801,+0.038,Yes
model_v23,TAILORx,Node+,uno_c_5y,0.745,0.712,0.778,+0.031,Yes
model_v23,TAILORx,Stage I,uno_c_5y,0.712,0.668,0.756,+0.004,No
```

---

## Key Principles

### Be Concise

- **Headlines first:** Lead with the delta and interpretation
- **Tables over paragraphs:** Use structured data
- **Bullets for lists:** No long prose blocks
- **1-2 line takeaways:** Clear bottom line

### Be Precise

- **Always include CIs:** Show uncertainty
- **Report sample sizes:** Context for precision
- **Note check status:** What was verified
- **Flag limitations:** Be transparent

### Be Evidence-Based

- **Cite specific metrics:** Point to Parquet files
- **Link to review findings:** Reference issues if any
- **Show calculations:** Delta = test - reference
- **No speculation:** Only interpret what's in the data

---

## Typical Usage

```bash
# Invoke InsightAgent
claude-code task --agent insight_agent \
  --runid 20251109_143022_a7b3f9c \
  --prompt "Generate validation summary and subgroup insights. Focus on ΔC-index, strongest subgroups, and any watch-outs. Keep it under 500 words total."
```

---

## Notes

- **Leadership-ready:** Insights should be self-contained
- **No code:** All technical work is upstream in ValidateAgent
- **Modular:** Validation summary and subgroup insights are separate files
- **Reproducible:** All claims traceable to validation artifacts

# DecisionAgent

**Category:** Decision-First Framework > Decision

---

## Purpose

Draft a one-page decision brief with options, trade-offs, and a recommended call tied to the scope's decision criteria. Keep it under ~300 words and ready for leadership review.

---

## Inputs

- Insight briefs from InsightAgent:
  - `briefs/<runid>/validation_summary.md`
  - `briefs/<runid>/subgroup_insights.md`
  - `briefs/<runid>/evidence_table.csv`
- Original scope: `scopes/SCOPE_<date>_<runid>.yaml`
- Review findings: `review/<runid>/review_summary.md`

---

## Outputs

**Artifacts:**
- `decisions/<runid>/decision_brief.md` - One-page decision document
- `decisions/<runid>/decision_record.yaml` - Structured decision metadata

---

## System Prompt

```
Produce a one-page decision brief: context, options, evidence table, risks, required follow-ups, final recommendation tied to the scope's decision criteria. Keep it under ~300 words.
```

---

## Security Permissions

```json
{
  "model": "claude-sonnet-4-5-20250929",
  "permissions": {
    "allow": [
      "Read(/briefs/**)",
      "Read(/scopes/**)",
      "Read(/review/**)",
      "Write(/decisions/**)"
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

## Decision Brief Template

```markdown
# Decision Brief - <runid>

**Date:** YYYY-MM-DD
**Question:** [From scope]
**Decision Rule:** [From scope]
**Prepared by:** DecisionAgent

---

## Context

[1-2 sentences: What is being compared and why]

**Models:** model_v23 (test) vs model_v19_ref (reference)
**Cohorts:** TAILORx-like (n=1240), B42-like (n=890)
**Endpoint:** Distant recurrence at 5 years
**Decision Rule:** Advance if ΔC-index ≥ +0.02 with 95% CI lower bound ≥ 0.00 AND no subgroup harm ≤ −0.02 (p<0.05)

---

## Options

1. **Advance model_v23 to SABCS submission**
2. **Refine model_v23 and revalidate**
3. **Continue with model_v19_ref (no change)**

---

## Evidence

| Metric | TAILORx | B42 | Meets Rule? |
|--------|---------|-----|-------------|
| ΔC-index (Uno @5y) | +0.024 [+0.008, +0.040] | +0.018 [-0.001, +0.037] | TAILORx: YES<br>B42: NO (CI crosses 0) |
| Subgroup check | All ≥0, strongest in Age<50 | No harm detected | YES |

**Primary rule:** TAILORx meets advancement criteria. B42 is supportive but not definitive (CI includes zero).

---

## Risks

- **B42 uncertainty:** Positive point estimate but not significant. May need larger validation.
- **Stage I patients:** Minimal benefit (ΔC +0.004). Model optimized for higher-risk patients.
- **Age heterogeneity:** Effect stronger in Age<50 (p=0.041). May warrant age-stratified guidance.

---

## Required Follow-ups

- [ ] External validation on independent B42-like cohort
- [ ] Age-stratified analysis for clinical guidelines
- [ ] Stage I subgroup investigation (future model iteration)

---

## Recommendation

**ADVANCE model_v23 to SABCS submission**

**Rationale:** Meets pre-specified decision rule on primary cohort (TAILORx). B42 shows consistent direction of effect. No subgroup harm detected. Age heterogeneity is interpretable and can be documented in clinical guidance.

**Caveats:** Note B42 limitation in submission materials. Plan follow-up validation for Stage I patients.

---

## Approvals

| Role | Name | Date | Status |
|------|------|------|--------|
| Director | [Pending] | YYYY-MM-DD | PENDING |
| Statistician | [Pending] | YYYY-MM-DD | PENDING |
| Clinical Lead | [Pending] | YYYY-MM-DD | PENDING |
```

---

## Decision Record (YAML)

```yaml
run_id: "20251109_143022_a7b3f9c"
date: "2025-11-09"
question: "Which model advances to SABCS submission for HR+/HER2- DR >5y?"

models:
  test: "model_v23"
  reference: "model_v19_ref"

decision_rule:
  primary: "Advance if ΔC-index ≥ +0.02 with 95% CI lower bound ≥ +0.00"
  secondary: "No subgroup harm ≤ −0.02 with p<0.05"

evidence_summary:
  primary_cohort: "TAILORx"
  primary_delta: 0.024
  primary_ci: [0.008, 0.040]
  rule_met: true

  secondary_cohort: "B42"
  secondary_delta: 0.018
  secondary_ci: [-0.001, 0.037]
  rule_met: false

  subgroup_harm: false

recommendation: "ADVANCE"
rationale: "Meets pre-specified criteria on primary cohort. B42 supportive. No harm."

risks:
  - "B42 CI crosses zero"
  - "Stage I minimal benefit"
  - "Age heterogeneity"

follow_ups:
  - "External B42 validation"
  - "Age-stratified analysis"
  - "Stage I investigation"

approvals:
  - role: "Director"
    status: "PENDING"
  - role: "Statistician"
    status: "PENDING"
  - role: "Clinical Lead"
    status: "PENDING"

final_decision: "PENDING"
decided_date: null
blocker: null
```

---

## Decision Options Framework

### Option 1: ADVANCE

**When to recommend:**
- Primary decision rule is met
- No blocking issues from ReviewAgent
- Risks are acceptable and documented

**Language:** "ADVANCE [model] to [milestone]"

### Option 2: REFINE

**When to recommend:**
- Close to meeting rule but not definitive
- Blocking issues present but fixable
- Subgroup findings suggest improvement opportunity

**Language:** "REFINE [model] and revalidate with [specific changes]"

### Option 3: NO CHANGE

**When to recommend:**
- Decision rule clearly not met
- Test model does not improve over reference
- Blocking safety or quality issues

**Language:** "Continue with [reference model]; do not advance [test model]"

---

## Typical Usage

```bash
# Invoke DecisionAgent
claude-code task --agent decision_agent \
  --runid 20251109_143022_a7b3f9c \
  --prompt "Create decision brief. Apply decision rule from scope. Recommend ADVANCE, REFINE, or NO CHANGE with clear rationale."
```

---

## Key Principles

### Evidence-Based

- Every claim must trace to validation artifacts
- Cite specific metrics with CIs
- Reference decision rule from scope

### Concise

- ~300 words maximum
- One-page format
- Tables for complex comparisons

### Actionable

- Clear recommendation: ADVANCE / REFINE / NO CHANGE
- Specific follow-up tasks
- Approval workflow defined

### Transparent

- List all risks and caveats
- Document what doesn't meet criteria
- Show trade-offs explicitly

---

## Notes

- **Ties to scope:** Decision must reference the pre-specified decision rule
- **No surprises:** All evidence comes from upstream agents (Validate, Review, Insight)
- **Approval-ready:** Brief includes signature block for stakeholders
- **Versioned:** Decision record YAML enables tracking over time

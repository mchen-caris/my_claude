# Experiment Planner

**Category:** Function-Based Framework > Design & Planning

---

## Purpose

Define the analysis question, endpoint(s), time horizons, splits, and metric plan with full provenance and seeds. Create a reproducible experiment plan that locks all parameters before analysis begins.

---

## Inputs

- `configs/experiment.yaml` - Experiment defaults and parameters
- `schemas/cohort.json` - Cohort schema definition
- `schemas/predictions.json` - Predictions schema definition

---

## Outputs

**Artifacts:**
- `reports/experiment_planner/run_<id>/experiment_plan.md` - Detailed experiment plan
- `configs/resolved_experiment.yaml` - Fully expanded, frozen configuration
- `splits/train.csv`, `splits/val.csv`, `splits/test.csv` - Generated splits (if requested)

---

## System Prompt

```
Create a reproducible experiment plan for endpoint={distant_recurrence}, horizons=[60,120], 5-fold CV with fixed seed=42. Validate inputs against schemas, emit `resolved_experiment.yaml`, and generate train/val/test CSVs. Include assumptions and risks.
```

---

## Security Permissions

```json
{
  "model": "claude-sonnet-4-5-20250929",
  "permissions": {
    "allow": [
      "Read(/configs/**)",
      "Read(/schemas/**)",
      "Write(/configs/resolved_experiment.yaml)",
      "Write(/splits/**)",
      "Write(/reports/experiment_planner/**)",
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

## Experiment Plan Template

```markdown
# Experiment Plan - <run_id>

**Created:** YYYY-MM-DD HH:MM:SS
**Author:** Experiment Planner Agent

---

## Objective

[Clear statement of research question]

**Example:** Evaluate whether model_v23 improves distant recurrence prediction compared to model_v19_ref in HR+/HER2- breast cancer patients.

---

## Dataset Scope

**Cohort:** TAILORx-like patients
**Inclusion Criteria:**
- HR+/HER2- breast cancer
- Stage I-III
- Adjuvant treatment received
- Minimum 5-year follow-up

**Exclusion Criteria:**
- Missing key clinical variables (grade, node status)
- Lost to follow-up <12 months
- Neoadjuvant therapy

**Expected N:** ~1,200 patients

---

## Endpoints

**Primary Endpoint:** Distant recurrence
- **Definition:** First distant metastasis or death from breast cancer
- **Time origin:** Date of surgery
- **Censoring:** Last follow-up date or non-BC death

**Secondary Endpoints:**
- Overall survival
- Local recurrence-free survival

---

## Time Horizons

- **60 months** (5 years)
- **120 months** (10 years)

---

## Metrics

| Metric | Version | Description | CI Method |
|--------|---------|-------------|-----------|
| Harrell C-index | 1.2.0 | Concordance with exact tie handling | Bootstrap (B=2000) |
| Uno C-index | 1.1.0 | Time-dependent C with IPCW | Bootstrap (B=2000) |
| AUC(t) | 1.0.0 | Time-specific AUROC | IPCW + Bootstrap |
| Calibration | 1.0.0 | Observed vs expected at horizons | Isotonic regression |

---

## Cross-Validation Strategy

**Method:** Stratified K-Fold
**K:** 5 folds
**Stratification:** Event status × Age group
**Seed:** 42 (fixed for reproducibility)

---

## Splits

| Split | N | Events | Event Rate | Purpose |
|-------|---|--------|------------|---------|
| Fold 0 | 240 | 42 | 17.5% | Validation |
| Fold 1 | 240 | 40 | 16.7% | Validation |
| Fold 2 | 240 | 43 | 17.9% | Validation |
| Fold 3 | 240 | 41 | 17.1% | Validation |
| Fold 4 | 240 | 44 | 18.3% | Validation |

**Split Files:**
- `splits/train.csv` (patient_id, fold, split, seed)
- `splits/val.csv`
- `splits/test.csv`

---

## Subgroups (Pre-specified)

| Subgroup | Definition | Min Events | Rationale |
|----------|------------|------------|-----------|
| Age <50 | age < 50 | 20 | Younger patients may have different biology |
| Age ≥50 | age >= 50 | 20 | Reference group |
| Node- | pN = 0 | 20 | Lower-risk group |
| Node+ | pN > 0 | 20 | Higher-risk group |
| Grade 1-2 | grade <= 2 | 20 | Lower grade tumors |
| Grade 3 | grade = 3 | 20 | Higher grade tumors |

---

## Assumptions

1. **Missing data:** <5% missing for key variables; complete-case analysis
2. **Follow-up:** Median follow-up ≥7 years
3. **Event rate:** ~15-20% distant recurrence at 10 years
4. **Risk score:** Higher score indicates higher risk (validated)

---

## Risks

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Insufficient events in subgroups | Medium | High | Monitor event counts; combine if needed |
| Follow-up censoring bias | Medium | Medium | Use IPCW methods |
| Missing data > 5% | Low | Medium | Perform sensitivity analysis |
| Label leakage | Low | Critical | Strict time-zero definition and review |

---

## Acceptance Checklist

Analysis is ready to proceed if:

- [ ] All schemas validated
- [ ] Splits generated with fixed seed
- [ ] Subgroups have ≥20 events each
- [ ] Metric definitions locked and versioned
- [ ] Time origin clearly defined
- [ ] Censoring rules explicit
- [ ] No label leakage risk identified

---

## Provenance

**Config File:** configs/experiment.yaml
**Config Hash:** abc123def456...
**Schemas:** cohort.json (v1.2.0), predictions.json (v1.1.0)
**Generated By:** Experiment Planner Agent v1.3.0
**Run ID:** 20251109_150000_explan
```

---

## Resolved Experiment YAML

```yaml
# Resolved Experiment Configuration
# Generated: 2025-11-09 15:00:00
# Run ID: 20251109_150000_explan

experiment:
  name: "Model v23 vs v19 Validation"
  objective: "Evaluate DR prediction improvement"
  date: "2025-11-09"
  run_id: "20251109_150000_explan"

cohort:
  name: "TAILORx_like"
  expected_n: 1200
  inclusion:
    - "HR+/HER2- breast cancer"
    - "Stage I-III"
    - "Adjuvant treatment"
    - "Min 5y follow-up"
  exclusion:
    - "Missing grade or node status"
    - "Follow-up <12 months"
    - "Neoadjuvant therapy"

endpoints:
  primary:
    name: "distant_recurrence"
    definition: "First distant metastasis or BC death"
    time_origin: "surgery_date"
    censoring: "last_followup or non_BC_death"
  secondary:
    - "overall_survival"
    - "local_recurrence_free"

horizons: [60, 120]  # months

metrics:
  - name: "harrell_cindex"
    version: "1.2.0"
    file: "metrics_def/harrell_cindex_v1.2.0.yaml"
    ci_method: "bootstrap"
    ci_iterations: 2000
  - name: "uno_cindex"
    version: "1.1.0"
    file: "metrics_def/uno_cindex_v1.1.0.yaml"
    ci_method: "bootstrap"
    ci_iterations: 2000
  - name: "auc_t"
    version: "1.0.0"
    horizons: [60, 120]
  - name: "calibration"
    version: "1.0.0"
    horizons: [60, 120]
    method: "isotonic"

cross_validation:
  method: "stratified_kfold"
  k: 5
  stratify_by: ["event", "age_group"]
  seed: 42

splits:
  train: "splits/train.csv"
  val: "splits/val.csv"
  test: "splits/test.csv"
  seed: 42

subgroups:
  - name: "Age<50"
    filter: "age < 50"
    min_events: 20
  - name: "Age>=50"
    filter: "age >= 50"
    min_events: 20
  - name: "Node-"
    filter: "pN == 0"
    min_events: 20
  - name: "Node+"
    filter: "pN > 0"
    min_events: 20
  - name: "Grade1-2"
    filter: "grade <= 2"
    min_events: 20
  - name: "Grade3"
    filter: "grade == 3"
    min_events: 20

provenance:
  config_source: "configs/experiment.yaml"
  config_hash: "abc123def456789..."
  schema_versions:
    cohort: "1.2.0"
    predictions: "1.1.0"
  generated_by: "experiment_planner"
  agent_version: "1.3.0"
```

---

## Split CSV Format

### splits/train.csv

```csv
patient_id,fold,split,seed,age_group,event
SUBJ_000001,0,train,42,<50,0
SUBJ_000002,0,train,42,>=50,1
SUBJ_000003,1,train,42,<50,0
...
```

---

## Typical Usage

```bash
# Generate experiment plan
claude-code task --agent experiment_planner \
  --prompt "Create experiment plan for endpoint=distant_recurrence, horizons=[60,120], 5-fold CV, seed=42. Validate schemas and generate splits."

# Update existing plan
claude-code task --agent experiment_planner \
  --prompt "Update experiment plan with new subgroups: ER high vs low. Regenerate splits if needed."
```

---

## Validation Steps

1. **Schema Validation:** Check cohort and predictions against JSONSchema
2. **Split Stratification:** Verify event rates balanced across folds
3. **Subgroup Sizes:** Check minimum events threshold (≥20)
4. **Metric Definitions:** Confirm all metric versions exist
5. **Time Origin:** Validate time-zero definition is explicit
6. **Censoring:** Ensure censoring rules are clear

---

## Notes

- **Immutability:** Once resolved_experiment.yaml is generated, it should not be modified
- **Version control:** Commit resolved config and splits to git
- **Seeds:** Fixed seeds ensure reproducible splits
- **Pre-specification:** All subgroups and metrics declared before seeing results

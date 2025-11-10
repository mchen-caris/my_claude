# ValidateAgent

**Category:** Decision-First Framework > Validation

---

## Purpose

Harmonize inputs, compute agreed metrics, and produce ΔC-index and subgroup results that match the scope. Follow the scope contract exactly with deterministic, reproducible outputs.

---

## Inputs

- Scope YAML from `scopes/SCOPE_<date>_<runid>.yaml`
- Standardized predictions: `inputs/preds_<modelid>_<cohort>_<fold>.parquet`
- Event tables: `inputs/events_<cohort>.parquet`
- Metric definitions: `metrics_def/<metric>_v<semver>.yaml`

---

## Outputs

**Artifacts:**
- `results/<runid>/metrics.parquet` - Row-wise long format metrics
- `results/<runid>/delta_cindex.parquet` - ΔC-index vs reference
- `results/<runid>/subgroups.parquet` - Subgroup-specific results
- `qc/<runid>/validation_log.md` - QC checks and warnings

---

## System Prompt

```
Follow the scope. Load standardized inputs. Compute pre-specified metrics (with exact definitions), ΔC-index vs reference, subgroup tables, and QC checks. Save Parquet and a brief QC log. No plots unless requested. Be deterministic.
```

---

## Security Permissions

```json
{
  "model": "claude-sonnet-4-5-20250929",
  "permissions": {
    "allow": [
      "Read(/inputs/**)",
      "Read(/scopes/**)",
      "Read(/metrics_def/**)",
      "Write(/results/**)",
      "Write(/qc/**)"
    ],
    "deny": [
      "Network(*)",
      "Write(/data/**)",
      "Write(/secrets/**)"
    ]
  }
}
```

---

## Standardization Spec

### Common Columns

- `subject_id` - De-identified patient ID
- `fold` - Cross-validation fold number
- `cohort` - Dataset/cohort name
- `model_id` - Model identifier with version
- `time` - Event or censoring time
- `event` - Binary event indicator (1=event, 0=censored)
- `pred_risk` - Predicted risk score (higher = higher risk)
- `horizon` - Prediction horizon in months
- `group` - Subgroup label

---

## Output Formats

### metrics.parquet

| Column | Type | Description |
|--------|------|-------------|
| model_id | str | Model identifier |
| dataset | str | Cohort name |
| fold | int | Fold number |
| metric | str | Metric name |
| value | float | Metric value |
| ci_lo | float | Lower 95% CI |
| ci_hi | float | Upper 95% CI |
| n | int | Sample size |
| params_hash | str | Parameter hash for reproducibility |

### delta_cindex.parquet

| Column | Type | Description |
|--------|------|-------------|
| model_id | str | Test model |
| ref_model_id | str | Reference model |
| dataset | str | Cohort name |
| metric | str | Metric name |
| delta | float | Difference (test - ref) |
| ci_lo | float | Lower 95% CI |
| ci_hi | float | Upper 95% CI |
| pvalue | float | Two-sided p-value |

### subgroups.parquet

| Column | Type | Description |
|--------|------|-------------|
| model_id | str | Model identifier |
| dataset | str | Cohort name |
| subgroup | str | Subgroup label |
| metric | str | Metric name |
| value | float | Metric value |
| delta_vs_ref | float | vs reference model |
| ci_lo | float | Lower 95% CI |
| ci_hi | float | Upper 95% CI |
| n | int | Subgroup sample size |

---

## Validation Recipes

### ΔC-index Computation

- Bootstrap stratified by fold (B=2000)
- Report delta, 95% CI, two-sided p-value
- Use exact tie handling per metric definition

### Time-Dependent C-index

- Truncation by percentile of event times (e.g., 90th)
- IPCW weights for censoring
- Estimator reference from metric definition

### Subgroup Analysis

- Pre-declared subgroups only
- Minimum events ≥20 per subgroup
- Interaction tests if specified in scope

### QC Checks

- Risk direction consistency
- Label leakage detection
- Sample count verification
- Reproducibility hash matching

---

## Metadata in Every Parquet

```python
# Embedded in Parquet metadata
{
  "analysis_version": "1.3.0",
  "metric_def_versions": {
    "harrell_cindex": "1.2.0",
    "td_cindex_trunc": "1.1.0"
  },
  "scope_sha256": "<hash>",
  "env_hash": "<digest>",
  "producer_agent": "ValidateAgent",
  "run_id": "20251109_143022_a7b3f9c"
}
```

---

## Typical Usage

```bash
# Invoke ValidateAgent
claude-code task --agent validate_agent \
  --scope scopes/SCOPE_20251109_143022_a7b3f9c.yaml \
  --prompt "Run full validation per scope: compute metrics, deltas, and subgroups. Output to results/"
```

---

## Notes

- **Deterministic execution** - Same inputs → same outputs
- **No exploratory analysis** - Only compute what's in the scope
- **Bootstrap seed** - Must be specified in scope for reproducibility

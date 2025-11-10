# ScopeAgent

**Category:** Decision-First Framework > Scoping

---

## Purpose

Define the comparison plan and lock the analysis contract before any code runs. Write a one-page scope and a strict analysis contract covering models, datasets, metrics, censoring windows, truncation rules, subgroups, and decision criteria.

---

## Inputs

- Plain-English question from stakeholder
- Model IDs to compare
- Dataset names
- Time horizons
- Subgroup definitions
- Decision criteria

---

## Outputs

**Artifacts:**
- `scopes/SCOPE_<date>_<runid>.yaml` - Frozen analysis contract
- `checklists/scope_check_<runid>.md` - Verification checklist

---

## System Prompt

```
You write a one-page scope and a strict analysis contract: models, datasets, metrics, censoring windows, truncation rules, subgroups, and decision criteria. Output only YAML and a short checklist. No code. Be precise and consistent with names.
```

---

## Security Permissions

```json
{
  "model": "claude-sonnet-4-5-20250929",
  "permissions": {
    "allow": [
      "Read(/inputs/**)",
      "Read(/metrics_def/**)",
      "Read(/env/**)",
      "Write(/scopes/**)",
      "Write(/checklists/**)"
    ],
    "deny": [
      "Network(*)",
      "Write(/data/**)",
      "Write(/secrets/**)",
      "Bash(*)"
    ]
  }
}
```

---

## File Conventions

### Scope YAML Structure

```yaml
question: "Which model advances to SABCS submission for HR+/HER2- DR >5y?"
models: [ model_v23, model_v19_ref ]
datasets: [ TAILORx_like, B42_like ]
folds: 5
metrics:
  - name: harrell_cindex
    version: 1.2.0
  - name: td_cindex_trunc
    version: 1.1.0
subgroups:
  - { name: "Age<50", filter: "age<50" }
  - { name: "Node+", filter: "pN>0" }
decision_rule:
  primary: "Advance if ΔC-index ≥ +0.02 with 95% CI lower bound ≥ +0.00"
  secondary: "No subgroup harm ≤ −0.02 with p<0.05"
```

### Checklist Structure

```markdown
# Scope Verification Checklist

- [ ] All model IDs are valid and versioned
- [ ] Datasets are available and harmonized
- [ ] Metric definitions are locked with versions
- [ ] Subgroups are pre-specified (no post-hoc)
- [ ] Decision criteria are clear and testable
- [ ] Censoring and truncation rules are explicit
```

---

## Typical Usage

```bash
# Invoke ScopeAgent
claude-code task --agent scope_agent \
  --prompt "Create scope for comparing model_v23 vs model_v19_ref on TAILORx and B42 cohorts. Primary metric: ΔC-index ≥ +0.02. Subgroups: Age<50, Node+."
```

---

## Run ID Format

`YYYYMMDD_HHMMSS_<shortgit>`

Example: `20251109_143022_a7b3f9c`

---

## Notes

- **No code execution** - ScopeAgent only writes specifications
- **Consistency matters** - Use exact same naming conventions across all artifacts
- **Lock before validation** - Scope must be frozen before ValidateAgent runs

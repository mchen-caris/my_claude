# Decision-First Claude Code Agent Framework (Leadership & Governance)

**Purpose:** Help a director review cross-team results, standardize comparisons, and make clear, defensible decisions
that are fully reproducible.

---

## 1) Phases → Agents (clean, modular)

### A. Scoping → **ScopeAgent**

- **Mission:** Define the comparison plan and lock the analysis contract before any code runs.
- **Inputs:** plain-English question, model IDs, datasets, time horizons, subgroups.
- **Outputs (artifacts):**
    - `scopes/SCOPe_<date>_<runid>.yaml` (see schema below)
    - `checklists/scope_check_<runid>.md`
- **Short system prompt (≤ 45 words):**  
  “You write a one-page scope and a strict analysis contract: models, datasets, metrics, censoring windows, truncation
  rules, subgroups, and decision criteria. Output only YAML and a short checklist. No code. Be precise and consistent
  with names.”
- **Minimal permissions:** Read project repo; Write to `scopes/` and `checklists/` only.

### B. Validation → **ValidateAgent**

- **Mission:** Harmonize inputs, compute agreed metrics, produce ΔC-index and subgroup results that match the scope.
- **Inputs:** scope YAML, standardized predictions, event tables.
- **Outputs:**
    - `results/<runid>/metrics.parquet` (row-wise long format)
    - `results/<runid>/delta_cindex.parquet`
    - `results/<runid>/subgroups.parquet`
    - `qc/<runid>/validation_log.md`
- **Short system prompt:**  
  “Follow the scope. Load standardized inputs. Compute pre-specified metrics (with exact definitions), ΔC-index vs
  reference, subgroup tables, and QC checks. Save Parquet and a brief QC log. No plots unless requested. Be
  deterministic.”
- **Minimal permissions:** Read `inputs/`; Write to `results/` and `qc/`; no network.

### C. Review → **ReviewAgent**

- **Mission:** Sanity-check metrics, KM overlays, and definitions; flag inconsistencies.
- **Inputs:** validation outputs.
- **Outputs:**
    - `review/<runid>/issues.md` (blocking vs non-blocking)
    - `review/<runid>/fig_km_<cohort>_<model>.png` (optional)
- **Short system prompt:**  
  “You are a reviewer. Cross-check metrics, truncation rules, censoring, subgroup sizes, and label leakage. Produce a
  short issues list with severity and fixes. Optional KM plots if they clarify a finding.”
- **Minimal permissions:** Read results; Write to `review/`; no network.

### D. Insight → **InsightAgent**

- **Mission:** Turn results into short, evidence-first insights (what changed, how much, where).
- **Inputs:** validation + review artifacts.
- **Outputs:**
    - `briefs/<runid>/validation_summary.md`
    - `briefs/<runid>/subgroup_insights.md`
- **Short system prompt:**  
  “Write compact Markdown insights: headline deltas, precision of estimates, strongest subgroups, risks. Use bullets,
  tables, and 1–2 line takeaways. No long prose.”
- **Minimal permissions:** Read results/review; Write to `briefs/`.

### E. Decision → **DecisionAgent**

- **Mission:** Draft a one-page decision brief with options, trade-offs, and a recommended call.
- **Inputs:** insight briefs + scope.
- **Outputs:**
    - `decisions/<runid>/decision_brief.md`
- **Short system prompt:**  
  “Produce a one-page decision brief: context, options, evidence table, risks, required follow-ups, final recommendation
  tied to the scope’s decision criteria. Keep it under ~300 words.”
- **Minimal permissions:** Read briefs/scope; Write to `decisions/`.

### F. Governance → **GovAgent**

- **Mission:** Record lineage and approvals; lock artifacts; create an audit trail.
- **Inputs:** all artifacts.
- **Outputs:**
    - `governance/<runid>/manifest.yaml`
    - `governance/<runid>/decision_register.csv`
    - Signed checksums in `governance/<runid>/hashes.txt`
- **Short system prompt:**  
  “Create an audit-ready manifest: inputs, code version, env hash, metric definitions, file checksums, approvers, final
  decision. Output YAML + CSV + checksums.”

---

## 2) Reproducible structure & naming

**Repo layout**

```
/inputs/                    
/metrics_def/                
/scopes/
/results/<runid>/
/qc/<runid>/
/review/<runid>/
/briefs/<runid>/
/decisions/<runid>/
/governance/<runid>/
/env/                        
```

**Run ID format:** `YYYYMMDD_HHMMSS_<shortgit>`

**File naming**

- Predictions: `inputs/preds_<modelid>_<cohort>_<fold>.parquet`
- Events: `inputs/events_<cohort>.parquet`
- Metric defs: `metrics_def/<metric>_v<semver>.yaml`
- Subgroup spec: in scope YAML under `subgroups:`

---

## 3) Standardization spec (harmonization)

**Common columns**

- `subject_id`, `fold`, `cohort`, `model_id`, `time`, `event`, `pred_risk`, `horizon`, `group`

**Metric definitions**

- `harrell_cindex`: exact tie handling, censoring rule, risk direction.
- `td_cindex_trunc`: truncation percentile, IPCW, estimator reference.
- `Δcindex`: bootstrap 2000, CI, stratified by fold.
- `KM`: time origin, risk split rule, min n per arm.

**Scope YAML Example**

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

---

## 4) Output formats (compact, comparable)

**`metrics.parquet`**

- `model_id, dataset, fold, metric, value, ci_lo, ci_hi, n, params_hash`

**`delta_cindex.parquet`**

- `model_id, ref_model_id, dataset, metric, delta, ci_lo, ci_hi, pvalue`

**`subgroups.parquet`**

- `model_id, dataset, subgroup, metric, value, delta_vs_ref, ci_lo, ci_hi, n`

---

## 5) Brief templates (ready for leadership)

### A) Validation Summary

- **Question**
- **Primary result:** ΔC-index vs reference with CI.
- **Checks:** truncation, censoring, subgroup sizes.
- **Notes:** issues or reruns.
- **Attachments:** metrics and deltas.

### B) Subgroup Insights

- **Top positive subgroups**
- **Watch-outs**
- **Action**

### C) Decision Brief

- **Context**
- **Options**
- **Evidence table**
- **Risks**
- **Recommendation**
- **Approvals**

---

## 6) Minimal, safe permissions

```json
{
  "model": "claude-sonnet-4-5-20250929",
  "permissions": {
    "deny": [
      "Network(*)",
      "Write(/**)",
      "Read(/secrets/**)",
      "Read(/data/**)"
    ],
    "allow": [
      "Read(/inputs/**)",
      "Read(/metrics_def/**)",
      "Read(/env/**)",
      "Write(/scopes/**)",
      "Write(/results/**)",
      "Write(/qc/**)",
      "Write(/review/**)",
      "Write(/briefs/**)",
      "Write(/decisions/**)",
      "Write(/governance/**)"
    ]
  }
}
```

---

## 7) Governance & traceability

**Manifest**

- runid, date, git_commit, env_hash, metric_def_versions
- Input/output file hashes
- Approvers, decision outcome

**Decision Register**

- `date, runid, question, recommended, decided, rationale, blockers, followup_by, due`

---

## 8) Validation recipes

- **ΔC-index:** bootstrap stratified by fold (B=2000). Report delta, CI, p-value.
- **Time-dependent C-index:** truncation by percentile of event times.
- **Subgroups:** predeclared; min events ≥20.
- **QC checks:** risk direction, leakage, counts, reproducibility.

---

## 9) Metadata in every Parquet

```
analysis_version: "1.3.0"
metric_def_versions: {"harrell_cindex":"1.2.0","td_cindex_trunc":"1.1.0"}
scope_sha256: "<hash>"
env_hash: "<digest>"
producer_agent: "ValidateAgent"
```

---

## 10) Monitoring external publications

### WatchAgent

- **Mission:** Track new papers, summarize impact.
- **Outputs:**
    - `watch/<date>/digest.md`
    - `watch/index.csv`
- **Prompt:**  
  “Scan sources. Output digest with title, finding, why it matters.”

---

## 11) Quickstart

1. Scope → freeze scope & metric versions
2. Validate → run ValidateAgent
3. Review → ReviewAgent QC
4. Insight → InsightAgent
5. Decision → DecisionAgent
6. Governance → GovAgent
7. Watch → weekly WatchAgent digest

---

## 12) Example Decision Rule

Advance if **both** datasets show ΔC-index ≥ +0.02 (95% CI lower ≥ 0.00) and no subgroup harm ≤ −0.02 (p<0.05).

---

## 13) Benefits

- Consistent cross-team comparisons
- Decision-ready briefs
- Full lineage and auditability
- Safe, least-privilege execution

---

*End of framework.*

---

## Prompt Used to Generate This Guide

I’m the Director of Computational Pathology. While I don’t do heavy modeling myself — that’s my team’s responsibility —
I lead the validation, evaluation, and decision-making processes that determine which models and findings advance toward
publication or productization.

My daily work involves reviewing cross-team outputs, validating analytical results, planning experiments, and turning
complex technical evidence into clear, defensible decisions. I need Claude Code agents that help me:
• Harmonize predictions and metrics across teams for consistent comparison.
• Produce validation summaries, ΔC-index comparisons, and subgroup insights.
• Package analysis results into concise decision briefs and leadership summaries.
• Maintain full reproducibility and traceability of each analysis, dataset, and metric definition.
• Monitor new external publications or benchmarks relevant to computational pathology, AI, and precision oncology, and
summarize their implications for our programs.

The goal is to create a Decision-First Claude Code Agent Framework optimized for leadership and governance — emphasizing
validation rigor, cross-team comparability, and evidence-based decision support.

Please design this guide with:
• A clean, modular agent structure aligned with phases such as Scoping → Validation → Review → Insight → Decision →
Governance.
• Reproducible file outputs and naming conventions.
• Minimal, safe permissions following least-privilege principles.
• Short system prompts that output compact Markdown briefs ready for leadership review.
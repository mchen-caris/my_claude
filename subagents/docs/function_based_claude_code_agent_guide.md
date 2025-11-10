# Function‑Based Claude Code Agent Framework

*For a hands‑on Director of Computational Pathology who leads analysis, validation, and decisions while the team handles
most modeling.*

This framework organizes Claude Code sub‑agents by daily functions. Every agent card includes: **Purpose**, **Inputs /
Outputs**, **Typical Prompt**, **File Conventions**, and **Security Permissions**. All prompts are concise and ready to
paste into your agent config or invoked as task descriptions.

> Guiding principles: least‑privilege permissions, reproducibility by default, consistent artifacts across teams, and
> outputs designed to flow into briefs and leadership decisions.

---

## Directory & Naming Conventions (Global)

- **Root**: `proj/<program>/<study>/<analysis_date_YYYYMMDD>/`
- **Inputs**: `data/` (read‑only), `schemas/` (JSON/YAML), `configs/` (YAML)
- **Working**: `work/<agent_name>/run_<timestamp>/`
- **Artifacts**: `artifacts/<agent_name>/run_<timestamp>/`
- **Reports**: `reports/<agent_name>/run_<timestamp>/`
- **Logs**: `logs/<agent_name>/run_<timestamp>.log`
- **Cache**: `cache/<agent_name>/`
- **Model Predictions (if present)**: `predictions/<model_id>/` with files `preds.parquet`, `metadata.json`
- **Standard metric file names**:
    - `metrics_survival.json` (Harrell C, Uno C, AUC(t), ΔC, CI, N, events)
    - `metrics_classification.json` (AUROC, AUPRC, F1, P/R at K, Calib)
    - `calibration_{horizon}.csv`
    - `km_{grouping}.png`, `forest_{endpoint}.png`
- **Dataset splits**: `splits/train.csv`, `splits/val.csv`, `splits/test.csv` (with `patient_id`, `fold`, `split`, seed)
- **Run ID**: `<YYYYMMDD>_<HHMMSS>_<agent_short>`; tag every artifact with `run_id` in metadata.

---

## Shared I/O Schemas (Minimal)

- **Cohort table**: `patient_id`, `arm`, `age`, `sex`, `stage`, `event`, `time`, additional clinical columns as needed.
- **Prediction table** (optional): `patient_id`, `risk_score` (higher = higher risk), `model_id`, `version`, `fold`,
  `timestamp`.
- **Schema spec** (`schemas/<name>.json`): JSONSchema draft‑07 compatible.
- **Metric JSON keys**:
  ```json
  {
    "endpoint": "distant_recurrence",
    "time_horizon_months": [60, 120],
    "harrell_c": {"point": 0.71, "ci": [0.68, 0.74], "n": 1240, "events": 210},
    "uno_c": {"point": 0.70, "ci": [0.67, 0.73]},
    "auc_t": [{"t": 60, "point": 0.76, "ci": [0.73, 0.79]}],
    "delta_c_vs_baseline": {"baseline_model_id": "cox_baseline", "point": 0.03},
    "fold_summary": {"k": 5, "mean": 0.712, "std": 0.018, "range": [0.694, 0.735]},
    "run_id": "20251109_223015_eval",
    "provenance": {"cohort_path": ".../data/cohort.parquet", "split_seed": 42}
  }
  ```

---

# Agents by Function

## 1) Design & Planning — Experiment and Metric Setup

### Agent: **Experiment Planner**

**Purpose**: Define the analysis question, endpoint(s), time horizons, splits, and metric plan with full provenance and
seeds.

**Inputs**:

- `configs/experiment.yaml` with defaults (endpoints, horizons, split strategy, seeds)
- `schemas/` for cohort and predictions

**Outputs**:

- `reports/experiment_planner/run_<id>/experiment_plan.md`
- `configs/resolved_experiment.yaml` (fully expanded, frozen)
- `splits/*.csv` (if generating splits)

**Typical Prompt**:

- *“Create a reproducible experiment plan for endpoint={distant_recurrence}, horizons=[60,120], 5-fold CV with fixed
  seed=42. Validate inputs against schemas, emit `resolved_experiment.yaml`, and generate train/val/test CSVs. Include
  assumptions and risks.”*

**File Conventions**:

- Plan file starts with: objective, dataset scope, endpoints, metrics, folds, seeds, exclusion criteria, risk log,
  acceptance checklist.

**Security Permissions (least‑privilege)**:

```json
{
  "allow": [
    "Read(configs/**)",
    "Read(schemas/**)",
    "Write(configs/resolved_experiment.yaml)",
    "Write(splits/**)",
    "Write(reports/experiment_planner/**)",
    "Write(logs/**)"
  ],
  "deny": [
    "Write(data/**)",
    "Network(*)",
    "Bash(*)"
  ]
}
```

---

### Agent: **Metric Registry Curator**

**Purpose**: Standardize metric definitions and confidence intervals so teams report the same fields.

**Inputs**:

- `configs/metric_registry.yaml` (canonical definitions)
- Optional examples from prior runs

**Outputs**:

- `reports/metric_registry/registry.md`
- Lint results applied to any `metrics_*.json` found in `artifacts/**`

**Typical Prompt**:

- *“Lint all metrics JSONs under `artifacts/**` to match `configs/metric_registry.yaml`. Normalize keys, ensure CI
  method is recorded (e.g., bootstrap B=2000 or Uno IPCW), add missing metadata.”*

**Security**:

```json
{
  "allow": [
    "Read(configs/metric_registry.yaml)",
    "Read(artifacts/**)",
    "Write(reports/metric_registry/registry.md)",
    "Write(logs/**)"
  ],
  "deny": [
    "Write(data/**)",
    "Network(*)",
    "Bash(*)"
  ]
}
```

---

## 2) Exploration — Cohort Summaries and Visualization

### Agent: **Cohort Profiler**

**Purpose**: One‑click cohort overview and readiness: counts, missingness, distributions, stratified summaries.

**Inputs**:

- `data/cohort.parquet` (or `.csv`), optional `schemas/cohort.json`

**Outputs**:

- `artifacts/cohort_profiler/run_<id>/baseline_table.csv`
- `artifacts/.../missingness.csv`
- Figures: `age_hist.png`, `stage_bar.png`, `km_overall.png`
- `reports/cohort_profiler/run_<id>/summary.md`

**Typical Prompt**:

- *“Profile `data/cohort.parquet`: generate baseline table by arm and overall; compute event rate and median follow‑up;
  run schema validation; produce KM for overall and by arm.”*

**Security**:

```json
{
  "allow": [
    "Read(data/cohort.*)",
    "Read(schemas/cohort.json)",
    "Write(artifacts/cohort_profiler/**)",
    "Write(reports/cohort_profiler/**)",
    "Write(logs/**)"
  ],
  "deny": [
    "Network(*)",
    "Bash(*)",
    "Write(data/**)"
  ]
}
```

---

### Agent: **Visualization Studio**

**Purpose**: Quick, consistent plots with controlled styles and saved code snippets for reuse.

**Inputs**:

- Any table from `artifacts/**` or `data/**`

**Outputs**:

- `artifacts/visualization_studio/run_<id>/*.png`
- `reports/visualization_studio/run_<id>/gallery.md`
- Snippet library: `artifacts/visualization_studio/snippets/*.py`

**Typical Prompt**:

- *“Generate ready‑to‑paste figures: KM curves for {endpoint}, forest plot for subgroups, calibration at 5y/10y, and
  save minimal Python that can recreate each figure.”*

**Security**:

```json
{
  "allow": [
    "Read(artifacts/**)",
    "Read(data/**)",
    "Write(artifacts/visualization_studio/**)",
    "Write(reports/visualization_studio/**)",
    "Write(logs/**)"
  ],
  "deny": [
    "Network(*)",
    "Bash(*)",
    "Write(data/**)"
  ]
}
```

---

## 3) Evaluation — Metric Computation and Model Comparison

### Agent: **Survival Metrics Engine**

**Purpose**: Compute Harrell’s C, Uno’s C (IPCW), and AUC(t); aggregate across folds; emit standard JSON and CSVs.

**Inputs**:

- `data/cohort.*` (event/time)
- `predictions/<model_id>/preds.parquet`
- `configs/resolved_experiment.yaml`

**Outputs**:

- `artifacts/survival_metrics/<model_id>/run_<id>/metrics_survival.json`
- `artifacts/.../auc_t.csv`, `calibration_{h}.csv`
- `reports/survival_metrics/<model_id>/summary.md`

**Typical Prompt**:

- *“Compute Harrell C, Uno C with IPCW, and AUC(t) at 60 and 120 months using `preds.parquet` and cohort; provide
  bootstrap 95% CI (B=2000), and fold‑level table with mean±sd.”*

**Security**:

```json
{
  "allow": [
    "Read(data/cohort.*)",
    "Read(predictions/**)",
    "Read(configs/resolved_experiment.yaml)",
    "Write(artifacts/survival_metrics/**)",
    "Write(reports/survival_metrics/**)",
    "Write(logs/**)"
  ],
  "deny": [
    "Network(*)",
    "Bash(*)",
    "Write(data/**)"
  ]
}
```

---

### Agent: **Model Comparison Planner**

**Purpose**: Normalize inputs from different teams and produce a single comparison book with ΔC, ranks, and tie‑break
rules.

**Inputs**:

- Multiple `predictions/<model_id>/preds.parquet`
- Corresponding `metrics_survival.json` if available
- Team mapping file: `configs/model_registry.yaml` (model_id → owner, contact, notes)

**Outputs**:

- `artifacts/model_compare/run_<id>/comparison_table.csv` (rows = model_id × endpoint × horizon)
- `reports/model_compare/run_<id>/comparison_book.md` with naming and folder conventions
- Figure set: `delta_c_bar.png`, `league_table.png`

**Typical Prompt**:

- *“Ingest predictions from listed `model_id`s; harmonize to a common schema; compute metrics if missing; generate a
  league table sorted by Uno C at 5y (primary), Harrell C as secondary. Apply tie‑break rules and output a
  ‘comparison_book.md’ with consistent model naming.”*

**Security**:

```json
{
  "allow": [
    "Read(predictions/**)",
    "Read(artifacts/**)",
    "Read(configs/model_registry.yaml)",
    "Write(artifacts/model_compare/**)",
    "Write(reports/model_compare/**)",
    "Write(logs/**)"
  ],
  "deny": [
    "Network(*)",
    "Bash(*)",
    "Write(data/**)"
  ]
}
```

---

## 4) Interpretation — Temporal, Subgroup, and Effect Analysis

### Agent: **Temporal Lens**

**Purpose**: Landmark and sliding‑window performance; ΔAUC(t); early vs late risk insight.

**Inputs**:

- Cohort and predictions
- Horizons and windows from `configs/resolved_experiment.yaml`

**Outputs**:

- `artifacts/temporal_lens/run_<id>/auc_t_window.csv`
- `reports/temporal_lens/run_<id>/temporal_findings.md`
- Figures: `auc_t_curve.png`, `early_vs_late_delta.png`

**Typical Prompt**:

- *“Compute AUC(t) curves with IPCW between 0–10y, report windows [0–5], [>5–10]. Provide interpretation bullets on
  where models diverge.”*

**Security**: same pattern as evaluation agents.

---

### Agent: **Subgroup Explorer**

**Purpose**: Pre‑specified subgroups (age, stage, nodal, RS strata, etc.), interaction screens, and forest plots.

**Inputs**:

- Cohort with subgroup columns
- Predictions

**Outputs**:

- `artifacts/subgroup_explorer/run_<id>/subgroup_metrics.csv` (Harrell/Uno per subgroup)
- `artifacts/.../interaction_tests.csv`
- `artifacts/.../forest_plot.png`
- `reports/subgroup_explorer/run_<id>/insights.md`

**Typical Prompt**:

- *“For defined subgroups (age≤50, >50; stage I/II/III; RS low/int/high), compute Uno C and interaction tests. Output
  forest plot and short narrative on effect heterogeneity.”*

**Security**: least‑privilege read/write; no network.

---

### Agent: **Effect Calibration & Explain**

**Purpose**: Calibration curves at horizons, decision thresholds, and simple explainers (e.g., partial dependence for
clinical covariates).

**Inputs**:

- Cohort, predictions, horizons

**Outputs**:

- `artifacts/effect_calibration/run_<id>/calibration_{h}.csv`
- `artifacts/.../threshold_table.csv` (risk cut‑points, sensitivity/specificity, PPV/NPV at horizons)
- `reports/effect_calibration/run_<id>/calibration_note.md`

**Typical Prompt**:

- *“Generate calibration at 60/120 months with smoothed curves and ECE; produce threshold table at target PPV/NPV for
  clinical review.”*

**Security**: same pattern.

---

## 5) Quality — Validation, Reproducibility, and Readiness Checks

### Agent: **Readiness & Schema Validator**

**Purpose**: Validate inputs against JSONSchema; check missingness, type drift, label drift, and leakage.

**Inputs**:

- `data/**`, `schemas/**`

**Outputs**:

- `reports/readiness_validator/run_<id>/readiness_report.md`
- `artifacts/.../anomaly_flags.csv`

**Typical Prompt**:

- *“Validate cohort and prediction tables; flag any schema/label drift; ensure no patient overlaps across splits; output
  deterministic pass/fail and remediation steps.”*

**Security**:

```json
{
  "allow": [
    "Read(data/**)",
    "Read(schemas/**)",
    "Write(artifacts/readiness_validator/**)",
    "Write(reports/readiness_validator/**)",
    "Write(logs/**)"
  ],
  "deny": [
    "Network(*)",
    "Bash(*)",
    "Write(data/**)"
  ]
}
```

---

### Agent: **Reproducibility Runner**

**Purpose**: Freeze versions, seeds, and dependencies; re‑run selected analyses; compare bit‑for‑bit artifacts.

**Inputs**:

- `configs/resolved_experiment.yaml`
- Prior `artifacts/**`

**Outputs**:

- `reports/repro_runner/run_<id>/repro_summary.md`
- `artifacts/repro_runner/run_<id>/diff_index.csv`

**Typical Prompt**:

- *“Re‑execute Survival Metrics Engine with identical seeds; compare outputs to prior run; produce a diff index and a
  conclusion (match vs drift).”*

**Security**: read configs and artifacts; write new artifacts and reports; no network.

---

## 6) Communication — Insight Narration and Decision Briefs

### Agent: **Decision Brief Writer**

**Purpose**: Convert artifacts into a two‑page decision brief for leadership with key charts and a one‑line
recommendation.

**Inputs**:

- Any `artifacts/**`, `reports/**`, `metrics_*` JSONs

**Outputs**:

- `reports/decision_brief/run_<id>/decision_brief.md`
- Optional `decision_brief.pdf` if a converter is allowed in your environment

**Typical Prompt**:

- *“Write a concise brief: context → methods → results (primary metric, CI, ΔC) → risks → decision. Include league table
  and top 3 charts. Keep it to two pages.”*

**Security**:

```json
{
  "allow": [
    "Read(artifacts/**)",
    "Read(reports/**)",
    "Write(reports/decision_brief/**)",
    "Write(logs/**)"
  ],
  "deny": [
    "Network(*)",
    "Bash(*)",
    "Write(data/**)"
  ]
}
```

---

### Agent: **Figure Pack Builder**

**Purpose**: Assemble a clean figure set with captions and sources for slide or manuscript use.

**Inputs**:

- Figure files and metric JSONs

**Outputs**:

- `reports/figure_pack/run_<id>/figure_list.md`
- `reports/.../figures/` copied with stable names

**Typical Prompt**:

- *“Assemble a figure pack: KM overall and by subgroup, calibration at 5y/10y, delta‑C bar chart. Write short captions
  and source paths for each.”*

**Security**: read artifacts; write reports; no network.

---

## 7) External Awareness — Literature and Benchmark Monitoring

### Agent: **Literature Radar**

**Purpose**: Track new papers in computational pathology, AI, and precision oncology; summarize impact and action items.

**Inputs**:

- `configs/lit_watch.yaml` (topics, journals, alert terms)

**Outputs**:

- `reports/literature_radar/run_<id>/digest.md`
- `artifacts/literature_radar/run_<id>/references.json`

**Typical Prompt**:

- *“Monitor latest papers for topics in `configs/lit_watch.yaml`. Summarize key findings, benchmarks, and any methods to
  trial. End with action items and links.”*

**Security**:

```json
{
  "allow": [
    "Read(configs/lit_watch.yaml)",
    "Write(artifacts/literature_radar/**)",
    "Write(reports/literature_radar/**)",
    "Write(logs/**)",
    "Network(https://*)"
  ],
  "deny": [
    "Write(data/**)",
    "Bash(*)"
  ]
}
```

---

# Minimal Prompt Templates (Copy/Paste)

- **Experiment Planner**
    - “Plan endpoint={endpoint}, horizons={H}, K‑fold={K}, seed={S}. Validate schemas and emit resolved config and
      splits.”

- **Cohort Profiler**
    - “Profile `data/cohort.parquet` for readiness. Baseline table by arm, missingness, KM overall/by arm.”

- **Survival Metrics Engine**
    - “Given cohort and `predictions/{model_id}/preds.parquet`, compute Harrell C, Uno C (IPCW), AUC(t) at {H}.
      Bootstrap 95% CI (B=2000). Emit standard JSON.”

- **Model Comparison Planner**
    - “Normalize inputs from {model_ids}, recompute metrics if needed, build league table (primary Uno C @5y). Apply
      naming rules and produce `comparison_book.md`.”

- **Temporal Lens**
    - “AUC(t) curve 0–10y; landmarks [0–5], [>5–10]; identify windows where models diverge and quantify ΔAUC(t).”

- **Subgroup Explorer**
    - “Compute subgroup metrics and interaction screens on predefined groups; output forest plot and narrative.”

- **Effect Calibration & Explain**
    - “Calibration at {H}; ECE; threshold table for PPV/NPV targets; document cut‑points.”

- **Readiness & Schema Validator**
    - “Validate schemas, type drift, leakage, and split integrity; produce pass/fail with remediation.”

- **Reproducibility Runner**
    - “Re‑run selected analyses with frozen seeds; compare artifacts and report diff index.”

- **Decision Brief Writer**
    - “Create a two‑page brief: context → methods → results → risks → decision; include league table and top charts.”

- **Figure Pack Builder**
    - “Assemble figure pack with stable names, captions, and sources for slides/manuscript.”

- **Literature Radar**
    - “Weekly digest for topics in `configs/lit_watch.yaml`; summarize implications and actions.”

---

## Consistent Model & File Naming (for Cross‑Team Work)

- **Model ID**: `<org>_<program>_<endpoint>_<algo>_<ver>` e.g., `caris_dr_cox_1_3_0`
- **Prediction File**: `predictions/<model_id>/preds.parquet`
- **Registry**: `configs/model_registry.yaml` with fields `owner`, `contact`, `model_family`, `notes`
- **Tie‑break rules**: primary `uno_c@5y`, then `harrell_c`, then `auc_t@10y`, then calibration ECE@5y.

---

## Security Defaults (Apply to All Agents)

- Deny write access to `data/**`. Agents read from `data/**` but write only to `artifacts/**` and `reports/**`.
- No network except **Literature Radar**. If network is needed elsewhere, add explicit domain allowlist.
- No shell commands by default. If formatting tools are needed, allow a limited `Bash(python -m black ...)` as a
  separate formatting step in CI, not inside agents.
- Always write a `provenance.json` with `run_id`, datetime, code version, config hash, and input paths.

---

## Quick Start Checklist

1. Create `schemas/` for cohort and predictions; validate with **Readiness & Schema Validator**.
2. Use **Experiment Planner** to freeze splits and metrics.
3. Run **Cohort Profiler** for baseline and readiness.
4. For each `model_id`, run **Survival Metrics Engine** and store artifacts.
5. Run **Model Comparison Planner** to build the league table and naming.
6. Use **Temporal Lens** and **Subgroup Explorer** to deepen interpretation.
7. Produce **Decision Brief** and **Figure Pack** for leadership.
8. Schedule **Literature Radar** weekly and triage action items.

---

*End of framework.*

---

## Prompt Used to Generate This Guide

I’m the Director of Computational Pathology. Although my team handles most of the modeling, I remain deeply hands-on
with data visualization, exploratory analysis, validation planning, and experiment design.

I want Claude Code agents that help me efficiently complete non-modeling analytical work, organize results across
projects, and generate clear insights for decision-making. These agents should be structured by functional categories to
match how I work every day.

My key priorities include:
• Cohort profiling, data readiness checks, and schema validation.
• Computing survival metrics (Harrell’s C, Uno’s C, AUC(t)) and standardizing outputs across teams.
• Conducting temporal and subgroup analyses for deeper interpretation.
• Designing and documenting validation studies with reproducibility and version control.
• Generating well-formatted briefs and figures for leadership communication.
• Staying current with new developments in computational pathology, AI, and precision oncology through automated
literature awareness.

Please create a Function-Based Claude Code Agent Framework organized by categories:

1. Design & Planning – experiment and metric setup.
2. Exploration – cohort summaries and visualization.
3. Evaluation – metric computation and model comparison.
4. Interpretation – temporal, subgroup, and effect analysis.
5. Quality – validation, reproducibility, and readiness checks.
6. Communication – insight narration and decision briefs.
7. External Awareness – literature and benchmark monitoring.

Each agent should include purpose, inputs/outputs, a typical prompt, file conventions, and security permissions —
practical enough for direct use in my coding environment.

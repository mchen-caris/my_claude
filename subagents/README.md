# Sub-Agent Guide for a Director of Computational Pathology (Decision & Insight Focus)

This guide lists the sub-agents you need to structure analytical, validation, and decision-making workflows. It includes the Model Comparison Planner for harmonizing and comparing results across teams, and it follows Claude Code best practices: tight scope, least privilege, stable I/O contracts, and reproducible decision-grade outputs.

## Table of Contents
- [Guiding Principles](#guiding-principles)
- [Agent Catalog](#agent-catalog)
  - [Cohort & Baseline (Exploration)](#cohort--baseline-exploration)
  - [Survival Metrics & Comparisons (Evaluation)](#survival-metrics--comparisons-evaluation)
  - [Temporal Patterns & Visualization (Interpretation)](#temporal-patterns--visualization-interpretation)
  - [Effects, Signals, and Interactions (Clinical Insight)](#effects-signals-and-interactions-clinical-insight)
  - [Validation, Reproducibility, and Readiness (Quality)](#validation-reproducibility-and-readiness-quality)
  - [Communication & Decision Support (Leadership Output)](#communication--decision-support-leadership-output)
  - [External Awareness (Focused Intake)](#external-awareness-focused-intake)
- [Model Comparison Planner Deep Dive](#model-comparison-planner-deep-dive)
- [Orchestration Map](#orchestration-map)
- [Folder & File Conventions](#folder--file-conventions)
- [Security & Permissions Defaults](#security--permissions-defaults)
- [System Prompt Template](#system-prompt-template)
- [Recommended Initial Deployment](#recommended-initial-deployment)

## Guiding Principles
- One agent -> one clear job so outputs stay consistent, composable, and testable.
- Enforce least privilege; deny `Read(data/**)`, `Write(data/**)`, `Network(**)`, and `Process(**)` by default.
- Keep stable I/O conventions; agents write to fixed folders such as `outputs/insights/` and `outputs/figures/`.
- Chain work through files instead of conversation memory for auditability.
- Produce readable deliverables: compact Markdown briefs with figures or tables.
- End every summary with Insights, Limitations, and Next Steps to reinforce decision framing.
- Log input files, hashes, run IDs, and code version for every report to guarantee reproducibility.

## Agent Catalog

### Cohort & Baseline (Exploration)

#### Cohort & Baseline Profiler
- **Purpose**: Snapshot cohorts, check balance, and highlight risks before modeling.
- **Inputs**: `inputs/clinical.csv`, optional `inputs/molecular.parquet`.
- **Outputs**:
  - `outputs/insights/baseline_table.md`
  - `outputs/figures/baseline_*`
- **Permissions**: Read `inputs/**`; write `outputs/insights/**` and `outputs/figures/**`.
- **Handoff**: Provides a clean cohort picture for downstream analyses.
- **Prompt**
  ```text
  Make a baseline table by HR/HER2 and mark SMD > 0.1.
  ```

#### Multimodal Correlation Mapper *(optional)*
- **Purpose**: Identify cross-modality relationships (e.g., image vs molecular).
- **Outputs**:
  - `outputs/figures/corr_*`
  - `outputs/insights/corr_summary.md`
- **Use when**: Exploring feature relationships prior to modeling.

### Survival Metrics & Comparisons (Evaluation)

#### Survival Core Metrics Hub
- **Purpose**: Compute Harrell's C, Uno's C, AUC(t), and classification metrics from a single interface.
- **Outputs**:
  - `outputs/metrics/core_metrics.csv`
  - `outputs/insights/core_metrics_brief.md`
- **Handoff**: Feeds Comparator and Decision Brief agents.
- **Prompt**
  ```text
  Compute Harrell's C, Uno's C @ 5y/10y, and AUC(t) 0–10y.
  ```

#### Survival Model Comparator
- **Purpose**: Compare multiple models using paired bootstrap deltas.
- **Outputs**:
  - `outputs/metrics/leaderboard.csv`
  - `outputs/metrics/delta_table.csv`
  - `outputs/insights/comparator_summary.md`
- **Prompt**
  ```text
  Compare Models A–F; report ΔC (Harrell/Uno) with 1,000 bootstraps.
  ```

#### Model Comparison Planner *(new; see deep dive below)*
- **Purpose**: Harmonize predictions across teams, enforce schemas, run shared evaluation protocols, and maintain a run registry.
- **Outputs**: Standardized preds (`outputs/preds/**`), metrics, deltas, figures, and `outputs/insights/comparison_brief.md`.

### Temporal Patterns & Visualization (Interpretation)

#### KM & Risk Visualization Studio
- **Purpose**: Generate KM curves, risk boxplots, and risk-time plots.
- **Outputs**:
  - `outputs/figures/km_*`
  - `outputs/figures/risk_*`
  - `outputs/insights/viz_captions.md`
- **Prompt**
  ```text
  KM by risk quartiles; add numbers at risk and caption.
  ```

#### Landmark & Temporal Pattern Analyst
- **Purpose**: Perform landmark analyses (e.g., 0–5y vs > 5y) and RMST windows.
- **Outputs**:
  - `outputs/metrics/landmark_metrics.csv`
  - `outputs/figures/landmark_*`
  - `outputs/insights/temporal_patterns.md`
- **Prompt**
  ```text
  Run landmark 5y and 10y; where does separation peak?
  ```

### Effects, Signals, and Interactions (Clinical Insight)

#### Univariate & Multivariate Effect Scanner
- **Purpose**: Produce univariable hazard screens and multivariable Cox summaries.
- **Outputs**:
  - `outputs/metrics/univariate_hr.csv`
  - `outputs/metrics/multivariable_hr.csv`
  - `outputs/figures/forest_*`
  - `outputs/insights/effect_scanner.md`

#### Predictive Utility & Treatment Interaction Agent
- **Purpose**: Quantify treatment x risk interactions and evaluate early vs late benefits.
- **Outputs**:
  - `outputs/metrics/interaction_tests.csv`
  - `outputs/figures/interaction_*`
  - `outputs/insights/predictive_signal.md`

### Validation, Reproducibility, and Readiness (Quality)

#### Validation Evidence Summarizer
- **Purpose**: Aggregate LoB/LoD, precision, ICC, and PPA/NPA/OPA into an executive dashboard.
- **Outputs**:
  - `outputs/insights/validation_dashboard.md`
  - `outputs/figures/icc_*`
  - `outputs/figures/agreement_*`

#### Reproducibility & Run Audit Companion
- **Purpose**: Check versions, seeds, and metric drift across runs.
- **Outputs**:
  - `outputs/insights/repro_audit.md`
  - `outputs/metrics/run_diff.csv`

### Communication & Decision Support (Leadership Output)

#### Results Narrator
- **Purpose**: Translate technical outputs into a narrative brief or slide deck.
- **Outputs**:
  - `outputs/insights/results_brief.md`
  - `docs/slides/results_brief.md`

#### Decision Brief Assembler
- **Purpose**: Gather all agents' deliverables into an action-focused memo.
- **Outputs**: `outputs/insights/decision_brief.md`

#### Documentation & Confluence Curator *(optional)*
- **Purpose**: Maintain internal documentation consistency.
- **Outputs**: `docs/confluence/*.md`

### External Awareness (Focused Intake)

#### Literature & Benchmark Scout
- **Purpose**: Scan new multimodal pathology papers and compare metrics with internal benchmarks.
- **Outputs**:
  - `docs/lit/monthly_digest.md`
  - `outputs/insights/benchmark_gap_table.md`

## Model Comparison Planner Deep Dive

**Mission**: Unify predictions from different groups into a consistent schema, enforce naming and folder standards, compute comparable metrics, and maintain a registry of all runs.

**Core capabilities**
- Schema Adapter: Map arbitrary team outputs to a standard schema via YAML adapters.
- Validator: Check required columns, data types, NA rates, and logical consistency.
- Protocol Runner: Apply a standard evaluation protocol (Harrell's C, Uno's C, AUC(t), calibration, paired ΔC).
- Aggregator: Produce leaderboards and delta tables.
- Reporter: Write executive summaries and update a run registry.

**Standard schema**
```text
patient_id | time | event | risk | model_id | cohort_id | subset | fold_id | seed | version
```

**Naming convention**
```text
<prefix>_<project>__<model_id>__<cohort_id>__<subset>__fold-<id>__seed-<num>__v-<tag>.<ext>
```

**Example**
```text
preds_bcr__deep_multi__EXT_A__test__fold-all__seed-0__v-2025-11-09.csv
```

**Folder layout**
```text
inputs/
  adapters/
  raw_preds/
outputs/
  preds/
  metrics/
  insights/
  figures/
registry/
  runs.csv
  models.yaml
  cohorts.yaml
docs/
  templates/
```

**Adapter YAML example**
```yaml
map:
  patient_id: "case_id"
  time: "os_time_months"
  event: "os_event"
  risk: "risk_score"
derive:
  cohort_id: "INT"
  subset: "test"
  model_id: "deep_multi"
```

**Outputs**
- Standardized prediction files -> `outputs/preds/`
- Metrics & deltas -> `outputs/metrics/`
- Figures -> `outputs/figures/`
- Registry updates -> `registry/runs.csv`
- Summary brief -> `outputs/insights/comparison_brief.md`

**Typical prompts**
```text
Adapt and validate all raw predictions under inputs/raw_preds/ using adapter YAMLs.
Compute leaderboard and ΔC versus baseline model.
Write a comparison brief with 3 key insights and next steps.
```

## Orchestration Map
- Cohort Profiler -> establish data readiness.
- Metrics Hub -> compute baseline metrics.
- Model Comparison Planner -> harmonize and organize multi-team results.
- Comparator -> deliver fine-grained statistical comparisons.
- Visualization Studio + Landmark Analyst -> show temporal trends.
- Effect Scanner + Interaction Agent -> extract clinical meaning.
- Validation Summarizer + Repro Audit -> ensure reliability.
- Results Narrator + Decision Brief Assembler -> communicate conclusions.
- Benchmark Scout -> keep external context current.

## Folder & File Conventions
```text
inputs/            # lightweight, shareable data extracts
outputs/
  figures/
  metrics/
  insights/
docs/
  slides/
  lit/
  confluence/
registry/
  runs.csv
  models.yaml
  cohorts.yaml
.claude/
  agents/*.md
```

## Security & Permissions Defaults
- Deny by default: `Read(data/**)`, `Write(data/**)`, `Network(**)`, `Process(**)`.
- Allow only the minimal folders an agent needs (typically `inputs/**`, `outputs/**`, `docs/**`, `registry/**`).
- No side effects outside approved folders.
- Each agent logs input paths, row counts, run ID, code version, and file hashes.

## System Prompt Template
- Style: concise, decision-oriented, plain English.
- Goals: list 3–5 concrete bullets.
- Inputs/Outputs: cite explicit file patterns.
- Constraints: enumerate all denies/allows.
- Quality checks: validate columns, NA rates, and sample counts.
- Finish: always include Insights / Limitations / Decisions.

## Recommended Initial Deployment
1. Cohort & Baseline Profiler
2. Survival Core Metrics Hub
3. Model Comparison Planner *(anchor for multi-team organization)*
4. Survival Model Comparator
5. KM & Risk Visualization Studio
6. Landmark & Temporal Pattern Analyst
7. Effect Scanner + Predictive Interaction Agent
8. Validation Evidence Summarizer
9. Repro Audit Companion
10. Results Narrator + Decision Brief Assembler
11. Literature & Benchmark Scout (add once routines stabilize)

This integrated structure delivers standardized, auditable model comparisons across reports, clean decision-ready summaries for leadership and regulatory discussions, and disciplined outputs through strict naming, folder organization, and run registry control.

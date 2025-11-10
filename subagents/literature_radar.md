# Literature Radar

**Category:** Function-Based Framework > External Awareness

---

## Purpose

Track new papers in computational pathology, AI, and precision oncology; summarize impact and action items. Stay current with external developments and competitive landscape.

---

## Inputs

- `configs/lit_watch.yaml` - Topics, journals, search terms
- `watch/index.csv` (optional) - Previous papers tracked

---

## Outputs

**Artifacts:**
- `reports/literature_radar/run_<id>/digest.md` - Weekly literature summary
- `artifacts/literature_radar/run_<id>/references.json` - Structured reference data
- `watch/index.csv` - Updated index with all tracked papers

---

## System Prompt

```
Monitor latest papers for topics in `configs/lit_watch.yaml`. Summarize key findings, benchmarks, and any methods to trial. End with action items and links.
```

---

## Security Permissions

```json
{
  "model": "claude-sonnet-4-5-20250929",
  "permissions": {
    "allow": [
      "Read(/configs/lit_watch.yaml)",
      "Read(/watch/**)",
      "Write(/artifacts/literature_radar/**)",
      "Write(/reports/literature_radar/**)",
      "Write(/watch/**)",
      "Write(/logs/**)",
      "Network(https://pubmed.ncbi.nlm.nih.gov/*)",
      "Network(https://arxiv.org/*)",
      "Network(https://www.biorxiv.org/*)",
      "Network(https://www.medrxiv.org/*)"
    ],
    "deny": [
      "Write(/data/**)",
      "Bash(*)"
    ]
  }
}
```

---

## Configuration File

### configs/lit_watch.yaml

```yaml
# Literature Monitoring Configuration

version: "1.0.0"
schedule: "weekly"  # or "daily", "monthly"
max_papers_per_digest: 25

topics:
  computational_pathology:
    keywords:
      - "deep learning histopathology"
      - "computational pathology AI"
      - "whole slide image analysis"
      - "digital pathology machine learning"
    journals:
      - "Nature Medicine"
      - "The Lancet Digital Health"
      - "npj Digital Medicine"
      - "Journal of Pathology Informatics"
      - "Modern Pathology"

  survival_models:
    keywords:
      - "survival prediction deep learning"
      - "prognostic model cancer AI"
      - "recurrence risk prediction"
      - "time-to-event neural network"
    journals:
      - "Journal of Clinical Oncology"
      - "Clinical Cancer Research"
      - "JAMA Oncology"
      - "Annals of Oncology"

  model_validation:
    keywords:
      - "external validation AI cancer"
      - "benchmark dataset digital pathology"
      - "C-index comparison survival"
      - "clinical utility AI"
    journals:
      - "Nature Medicine"
      - "The Lancet Oncology"
      - "NEJM AI"

  breast_cancer:
    keywords:
      - "breast cancer AI prediction"
      - "HR+ HER2- prognosis"
      - "distant recurrence breast"
      - "oncotype DX"
    journals:
      - "Journal of Clinical Oncology"
      - "Breast Cancer Research"
      - "npj Breast Cancer"

alert_terms:
  high_priority:
    - "randomized controlled trial"
    - "FDA approval"
    - "external validation"
    - "multi-center study"
    - "prospective validation"

  benchmark_datasets:
    - "TCGA"
    - "TAILORx"
    - "MINDACT"
    - "I-SPY"
    - "B42"

  competitors:
    - "PathAI"
    - "Paige.AI"
    - "Proscia"
    - "Indica Labs"

sources:
  - name: "PubMed"
    url: "https://pubmed.ncbi.nlm.nih.gov/"
    enabled: true
  - name: "arXiv"
    categories: ["cs.CV", "cs.LG", "stat.ML", "q-bio"]
    enabled: true
  - name: "bioRxiv"
    enabled: true
  - name: "medRxiv"
    enabled: true
```

---

## Literature Digest

```markdown
# Literature Digest - Week of November 9, 2025

**Generated:** 2025-11-09
**Period:** November 2-9, 2025
**Papers Reviewed:** 23
**High Priority:** 5

---

## Executive Summary

**Key Trends:**
- 3 new external validation studies for AI pathology models
- 2 FDA clearances announced (PathAI colon cancer, Paige.AI prostate)
- 1 major benchmark dataset released (TCGA-BRCA with 10-year follow-up)
- Calibration methods gaining attention (4 papers)

**Action Items:**
- Review new TCGA-BRCA benchmark for comparison
- Evaluate calibration-aware training methods
- Monitor PathAI FDA clearance process (similar to our pipeline)

---

## High Priority Papers

### 1. Multi-Center External Validation of AI for Breast Cancer Prognosis

**Authors:** Garcia et al.
**Journal:** Journal of Clinical Oncology
**Published:** 2025-11-05
**DOI:** 10.1200/JCO.2025.XX.XXXX

**Key Findings:**
- External validation of DL model across 5 sites (n=3,240)
- Harrell C-index: 0.71 [0.68-0.74]
- Significant site heterogeneity (C range: 0.66-0.75)
- **Recalibration required** at each site (ECE improved from 0.12 to 0.05)

**Why It Matters:**
- Demonstrates need for site-specific recalibration (relevant for our multi-site plans)
- Performance (C=0.71) comparable to our model (C=0.71)
- Provides roadmap for external validation protocol

**Comparison to Our Work:**
- **Their C-index:** 0.71 [0.68-0.74]
- **Our C-index:** 0.71 [0.69-0.74]
- Performance similar, but they validated across 5 sites (we have only 1 so far)

**Action Items:**
- [ ] Review their recalibration methodology (Section 4.2)
- [ ] Plan multi-site validation with similar protocol
- [ ] Add site-specific calibration to our pipeline
- [ ] Invite Dr. Garcia for seminar (contact: garcia@uni.edu)

**Link:** https://doi.org/10.1200/JCO.2025.XX.XXXX

---

### 2. Calibration-Aware Training Improves Clinical Utility of AI Risk Models

**Authors:** Wang et al.
**Journal:** The Lancet Digital Health
**Published:** 2025-11-03
**DOI:** 10.1016/S2589-7500(25)00XXX-X

**Key Findings:**
- Novel training objective: combine discrimination + calibration loss
- ECE reduced from 0.08 to 0.03 on held-out data
- C-index maintained (0.74 vs 0.74 standard training)
- **Better clinical decision curves** (net benefit +5% at 20% risk threshold)

**Method:**
```python
# Simplified loss function
loss = ce_loss + lambda * calibration_loss
# where calibration_loss = ECE or Brier score
```

**Why It Matters:**
- Calibration is critical for clinical deployment (risk communication)
- Our current model: ECE=0.042 (already good, but could improve to 0.03)
- Simple to implement (just change loss function)

**Action Items:**
- [ ] Implement calibration-aware training in next model iteration
- [ ] Compare to current approach on validation set
- [ ] Evaluate impact on decision curve analysis

**Link:** https://doi.org/10.1016/S2589-7500(25)00XXX-X

---

### 3. TCGA-BRCA 10-Year Survival Benchmark Dataset Released

**Authors:** TCGA Research Network
**Source:** Nature Cancer
**Published:** 2025-11-08
**DOI:** 10.1038/s43018-025-XXXX-X

**Dataset:**
- n=1,845 breast cancer patients
- 10-year follow-up (median 8.2 years)
- WSI + clinical + genomic + 10y outcomes
- **Public release:** November 15, 2025

**Benchmark Results (from paper):**

| Model | Harrell C | Uno C @5y |
|-------|-----------|-----------|
| Clinical only | 0.68 | 0.66 |
| Genomic (PAM50) | 0.72 | 0.70 |
| DL (imaging) | 0.74 | 0.72 |
| **DL + clinical + genomic** | **0.78** | **0.76** |

**Why It Matters:**
- New public benchmark for comparison
- Our model: C=0.71 (below DL benchmark of 0.74)
- **Multimodal integration** (DL + clinical + genomic) achieves C=0.78 (best)

**Action Items:**
- [ ] Download dataset on Nov 15 (requires dbGaP access)
- [ ] Run our model on TCGA-BRCA cohort for comparison
- [ ] Explore multimodal integration (imaging + genomic)
- [ ] Update benchmark tracking table

**Link:** https://doi.org/10.1038/s43018-025-XXXX-X

---

### 4. PathAI Receives FDA Clearance for AI Colon Cancer Risk Model

**Source:** PathAI Press Release + FDA 510(k) Database
**Date:** 2025-11-02
**510(k) Number:** K253XXX

**Product:**
- AIM Colon: AI risk model for colorectal cancer recurrence
- Harrell C-index: 0.73 (per FDA submission)
- Validated on 2,500 patients across 8 sites

**Regulatory Pathway:**
- 510(k) clearance (predicate: Oncotype DX Colon)
- CAP/CLIA lab deployment
- Reimbursement: CPT code 81XXX (pending)

**Why It Matters:**
- First AI histopathology risk model to receive FDA clearance for **solid tumor prognosis**
- Establishes precedent for our regulatory strategy
- Demonstrates feasibility of multi-site validation for clearance

**Comparison to Our Model:**
- **Their organ:** Colon
- **Our organ:** Breast
- **Similar approach:** DL on WSI → risk score
- **Similar performance:** C~0.73 (ours: 0.71)

**Action Items:**
- [ ] Review 510(k) submission (public after 30 days)
- [ ] Analyze validation protocol for regulatory best practices
- [ ] Discuss with regulatory team (target: Q1 2026 submission)

**Link:** https://www.pathai.com/news/fda-clearance

---

### 5. Continuous-Time Survival Models Outperform Discrete-Time Approaches

**Authors:** Lee et al.
**Journal:** arXiv preprint (cs.LG)
**Published:** 2025-11-07
**arXiv:** 2025.XXXXX

**Key Findings:**
- Continuous-time attention network (no discrete time bins)
- **C-index improvement:** +0.03 vs discrete-time baseline
- Reduced bias in long-tail event times
- Computational cost: +15% (acceptable)

**Method:**
- Models hazard as continuous function h(t) using attention mechanism
- No need to choose time bins (avoids discretization bias)

**Why It Matters:**
- Our current model uses discrete-time bins (12-month intervals)
- May introduce bias, especially for late events (>5 years)
- Relatively easy to implement (modify output layer)

**Action Items:**
- [ ] Review arxiv preprint in detail (Methods section)
- [ ] Prototype continuous-time model on our data
- [ ] Compare to current discrete-time approach
- [ ] Schedule journal club discussion

**Link:** https://arxiv.org/abs/2025.XXXXX

---

## Computational Pathology Updates

### Notable Methods

#### 1. Multi-Scale Attention for WSI Analysis (Chen et al., Nature Methods)
- Combines patch-level and slide-level attention
- **Performance:** C=0.76 on lung cancer survival
- **Our relevance:** Could improve our current single-scale approach

#### 2. Self-Supervised Pretraining on 100K WSIs (Smith et al., ICCV)
- Foundation model for pathology (similar to ImageNet pretraining)
- Publicly available weights
- **Action:** Test on our breast cancer cohort

#### 3. Interpretable Risk Scores via Attention Heatmaps (Johnson et al., Medical Image Analysis)
- Visualize which regions drive risk prediction
- **Clinical benefit:** Pathologist review and trust
- **Action:** Implement for our model

---

## Benchmark Updates

| Dataset | Best Reported C | Our Model C | Gap | Reference |
|---------|-----------------|-------------|-----|-----------|
| TCGA-BRCA | **0.78** (multimodal) | 0.71 | **-0.07** | TCGA Network 2025 |
| TCGA-BRCA | 0.74 (DL only) | 0.71 | -0.03 | TCGA Network 2025 |
| TAILORx-like | 0.71 | 0.71 | 0.00 | Garcia et al. 2025 |
| MINDACT | 0.69 | TBD | TBD | Literature benchmark |

**Insight:** Our model is competitive on TAILORx-like data but lags on TCGA (multimodal integration needed).

---

## Clinical Validation Studies

### Multi-Center Validation Reports

**Study 1:** Garcia et al. (JCO) - 5 sites, n=3,240
- **Result:** C=0.71, significant site heterogeneity
- **Lesson:** Recalibration essential

**Study 2:** Brown et al. (Lancet Oncology) - Prospective validation
- **Result:** C=0.68 [0.64-0.72] (slightly lower than retrospective)
- **Lesson:** Prospective validation often yields lower performance

**Study 3:** Miller et al. (JAMA Oncology) - Decision impact study
- **Result:** AI changed treatment in 18% of cases
- **Lesson:** Clinical utility ≠ statistical significance

---

## Regulatory & Ethics

### FDA Guidance Update (November 1, 2025)

**Key Points:**
- New guidance on AI/ML-enabled medical devices
- Emphasis on **continuous monitoring** post-deployment
- Requirement for **bias assessment** across demographics
- **Reimbursement pathway:** CPT codes for AI diagnostics

**Action Items:**
- [ ] Review full guidance document (62 pages)
- [ ] Update validation framework to include monitoring plan
- [ ] Plan bias assessment (age, race, ethnicity)
- [ ] Engage with payers for reimbursement strategy

**Link:** https://www.fda.gov/medical-devices/software-medical-device-samd/artificial-intelligence-and-machine-learning-aiml-enabled-medical-devices

---

## Competitor Activity

### PathAI
- FDA clearance for colon cancer (Nov 2)
- Expanding to breast cancer (rumored Q1 2026)
- **Competitive threat:** High

### Paige.AI
- FDA clearance for prostate cancer (Oct 15)
- $100M Series D funding
- **Focus:** Prostate and breast

### Proscia
- Partnership with Mayo Clinic announced
- Focus on workflow integration
- **Competitive threat:** Medium (different niche)

---

## Emerging Topics

### 1. Multimodal Integration
- Trend: Combining imaging + genomic + clinical
- Performance: +0.04 to +0.06 C-index improvement
- **Our gap:** Currently imaging + clinical only

### 2. Calibration Methods
- Increased focus on calibration (not just discrimination)
- Methods: Calibration-aware training, Platt scaling, isotonic regression
- **Our status:** Good calibration (ECE=0.042), but room to improve

### 3. Foundation Models
- Large pretrained models (100K+ WSIs)
- Transfer learning for specific tasks
- **Opportunity:** Leverage public foundation models

### 4. Explainability
- Attention heatmaps, SHAP values
- **Clinical need:** Pathologist interpretability and trust

---

## Action Summary

### High Priority (This Month)

- [ ] Download TCGA-BRCA dataset (Nov 15)
- [ ] Run our model on TCGA for benchmark comparison
- [ ] Review PathAI 510(k) submission (available ~Dec 1)
- [ ] Implement calibration-aware training
- [ ] Schedule journal club: Garcia et al. (JCO)

### Medium Priority (This Quarter)

- [ ] Prototype continuous-time survival model
- [ ] Explore multimodal integration (add genomic data)
- [ ] Implement attention heatmaps for interpretability
- [ ] Plan multi-site external validation study

### Low Priority (Monitoring)

- [ ] Monitor PathAI breast cancer product launch
- [ ] Track FDA guidance updates
- [ ] Update literature review document quarterly

---

## Papers Added to Index

- **25 new papers** added to `watch/index.csv`
- **5 flagged as high priority**
- **3 relevant datasets identified** (TCGA-BRCA, MINDACT update, I-SPY3)

---

## Next Digest

**Scheduled:** 2025-11-16
**Focus Topics:**
- Any follow-up on TCGA-BRCA benchmark
- Continued monitoring of FDA regulatory updates
- New multimodal methods

---

**Prepared by:** Literature Radar Agent
**Contact:** litreview@example.org
```

---

## Typical Usage

```bash
# Weekly digest
claude-code task --agent literature_radar \
  --prompt "Generate weekly literature digest for computational pathology, survival models, and validation studies. Flag high-priority papers and create action items."

# Focused search
claude-code task --agent literature_radar \
  --topic "calibration methods survival models" \
  --prompt "Search for recent papers (last 3 months) on calibration methods for survival prediction. Summarize top 5 papers."

# Competitor monitoring
claude-code task --agent literature_radar \
  --focus "competitors" \
  --prompt "Monitor recent activity from PathAI, Paige.AI, Proscia. Include FDA clearances, funding, and product launches."

# Benchmark tracking
claude-code task --agent literature_radar \
  --focus "benchmarks" \
  --prompt "Find new public datasets and benchmark results for breast cancer prognosis. Compare to our model performance."
```

---

## Search Strategy

### 1. Keyword Search
- Combine topic keywords with Boolean operators
- Use MeSH terms for PubMed
- Filter by publication date

### 2. Journal Scanning
- Monitor high-impact journals (Nature Med, Lancet, JCO)
- RSS feeds for new issues
- Table of contents alerts

### 3. Preprint Servers
- arXiv (cs.CV, cs.LG, stat.ML)
- bioRxiv / medRxiv (new biomedical preprints)

### 4. Competitor Monitoring
- Company press releases
- FDA clearance database (510(k), PMA)
- Patent filings (USPTO, Google Patents)

---

## Notes

- **Cadence:** Weekly recommended (configurable)
- **Automation:** Can schedule via cron
- **Collaboration:** Share digest with team
- **Long-term:** Maintain `watch/index.csv` for historical tracking
- **Network access:** Required for literature search

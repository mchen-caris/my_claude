# WatchAgent

**Category:** Decision-First Framework > External Awareness

---

## Purpose

Track new papers, benchmarks, and methods in computational pathology, AI, and precision oncology. Summarize impact and generate action items for the team.

---

## Inputs

- Topic and journal watchlist: `configs/watch_topics.yaml`
- Previous digest index: `watch/index.csv`
- Search queries and alert terms from config

---

## Outputs

**Artifacts:**
- `watch/<date>/digest.md` - Weekly summary of new papers
- `watch/<date>/references.json` - Structured reference data
- `watch/index.csv` - Updated index with all tracked papers

---

## System Prompt

```
Scan sources. Output digest with title, finding, why it matters. Track new papers in computational pathology, AI, precision oncology. Summarize implications and actions.
```

---

## Security Permissions

```json
{
  "model": "claude-sonnet-4-5-20250929",
  "permissions": {
    "allow": [
      "Read(/configs/watch_topics.yaml)",
      "Read(/watch/**)",
      "Write(/watch/**)",
      "Network(https://pubmed.ncbi.nlm.nih.gov/*)",
      "Network(https://arxiv.org/*)",
      "Network(https://www.biorxiv.org/*)",
      "Network(https://www.medrxiv.org/*)",
      "Network(https://scholar.google.com/*)"
    ],
    "deny": [
      "Write(/data/**)",
      "Write(/secrets/**)",
      "Bash(*)"
    ]
  }
}
```

---

## Watch Topics Configuration

```yaml
# configs/watch_topics.yaml

topics:
  - name: "Computational Pathology AI"
    keywords:
      - "deep learning histopathology"
      - "computational pathology"
      - "digital pathology AI"
      - "whole slide image analysis"
    journals:
      - "Nature Medicine"
      - "The Lancet Digital Health"
      - "npj Digital Medicine"
      - "Journal of Pathology Informatics"

  - name: "Survival Prediction Models"
    keywords:
      - "survival prediction deep learning"
      - "prognostic model cancer"
      - "recurrence risk AI"
      - "time-to-event neural network"
    journals:
      - "Journal of Clinical Oncology"
      - "Clinical Cancer Research"
      - "JAMA Oncology"

  - name: "Model Validation & Benchmarks"
    keywords:
      - "external validation AI"
      - "benchmark dataset pathology"
      - "C-index comparison"
      - "clinical validation AI cancer"
    journals:
      - "Nature Medicine"
      - "The Lancet Oncology"
      - "NEJM AI"

alert_terms:
  high_priority:
    - "randomized controlled trial"
    - "FDA approval"
    - "external validation"
    - "multi-center study"

  relevant_datasets:
    - "TCGA"
    - "TAILORx"
    - "MINDACT"
    - "I-SPY"

schedule: "weekly"
max_papers_per_digest: 20
```

---

## Digest Template

```markdown
# Literature Digest - Week of <date>

**Generated:** YYYY-MM-DD
**Period:** YYYY-MM-DD to YYYY-MM-DD
**Papers Reviewed:** N
**High Priority:** M

---

## High Priority Papers

### 1. [Title of Paper]

**Authors:** First Author et al.
**Journal:** Nature Medicine
**Published:** 2025-11-05
**DOI:** 10.1038/xxxxx

**Key Finding:**
- [1-2 sentence summary of main result]
- [Key metric or performance number]

**Why It Matters:**
- [Impact on our work]
- [Relevance to current models/approaches]
- [Benchmark comparison if applicable]

**Action Items:**
- [ ] Review methods for potential adoption
- [ ] Compare to our current C-index performance
- [ ] Add to benchmark tracking spreadsheet

**Link:** https://doi.org/10.1038/xxxxx

---

### 2. [Next Paper...]

---

## Computational Pathology Updates

### Notable Methods

- **Paper:** [Title]
- **Method:** [e.g., "Multi-scale attention for survival prediction"]
- **Performance:** [e.g., "C-index 0.78 on TCGA"]
- **Relevance:** [How it relates to our work]

### Benchmark Updates

| Dataset | Best Reported C-index | Our Model | Gap | Reference |
|---------|----------------------|-----------|-----|-----------|
| TCGA-BRCA | 0.76 | 0.74 | -0.02 | Smith et al. 2025 |
| TAILORx-like | 0.73 | 0.73 | 0.00 | Jones et al. 2025 |

---

## Survival Modeling Advances

### New Methods

1. **Continuous-time attention networks** (Lee et al., arxiv:2025.xxxxx)
   - Replaces discrete-time bins with continuous hazard
   - +0.03 C-index improvement on benchmark
   - **Action:** Review for next model iteration

2. **Calibration-aware training** (Wang et al., Lancet Digital Health)
   - Optimizes for both discrimination and calibration
   - ECE improvement from 0.08 to 0.03
   - **Action:** Consider for model refinement

---

## Clinical Validation Studies

### External Validation Reports

- **Study:** Multi-center validation of AI prognostic model (Garcia et al., JCO)
- **Cohorts:** 5 sites, n=3,240
- **Result:** C-index 0.71 (95% CI: 0.68-0.74)
- **Lessons:** Site heterogeneity requires recalibration
- **Action:** Review protocol for our multi-site validation

---

## Regulatory & Ethics

- **FDA guidance update** on AI/ML medical devices (2025-11-01)
- **Key point:** Emphasis on continuous monitoring post-deployment
- **Action:** Update validation framework to include monitoring plan

---

## Action Summary

**High Priority:**
- [ ] Review continuous-time attention method (Lee et al.)
- [ ] Update benchmark tracking with 3 new papers
- [ ] Compare calibration-aware training to current approach

**Medium Priority:**
- [ ] Add new TCGA benchmark results to comparison table
- [ ] Review FDA guidance document
- [ ] Schedule journal club for Garcia et al. validation study

**Low Priority:**
- [ ] Archive references in Zotero
- [ ] Update literature review document

---

## Papers Added to Index

- 8 new papers added to `watch/index.csv`
- 3 flagged as high priority
- 2 relevant datasets identified

---

## Next Digest

**Scheduled:** YYYY-MM-DD
**Focus Topics:** [Any special focus areas for next week]
```

---

## References JSON Structure

```json
{
  "digest_date": "2025-11-09",
  "period_start": "2025-11-02",
  "period_end": "2025-11-09",
  "papers": [
    {
      "id": "smith2025_natmed",
      "title": "Deep learning for distant recurrence prediction in breast cancer",
      "authors": ["Smith J", "Doe A", "Johnson B"],
      "journal": "Nature Medicine",
      "published_date": "2025-11-05",
      "doi": "10.1038/s41591-025-xxxxx",
      "url": "https://doi.org/10.1038/s41591-025-xxxxx",
      "keywords": ["deep learning", "breast cancer", "survival prediction"],
      "priority": "HIGH",
      "key_findings": [
        "C-index 0.76 on TCGA-BRCA",
        "Multi-scale attention improves long-term prediction"
      ],
      "relevance": "Direct comparison to our models; better performance on TCGA",
      "action_items": [
        "Review multi-scale attention architecture",
        "Compare to our TCGA results",
        "Add to benchmark table"
      ],
      "topics": ["Computational Pathology AI", "Survival Prediction Models"]
    }
  ]
}
```

---

## Index CSV Format

```csv
id,date_added,title,first_author,journal,year,doi,priority,topics,reviewed,notes
smith2025_natmed,2025-11-09,Deep learning for distant recurrence...,Smith J,Nature Medicine,2025,10.1038/s41591-025-xxxxx,HIGH,"Computational Pathology AI;Survival Prediction",Yes,Added to benchmark table
lee2025_arxiv,2025-11-09,Continuous-time attention for survival...,Lee K,arXiv,2025,arxiv:2025.xxxxx,HIGH,"Survival Prediction Models",No,Preprint - monitor for journal publication
```

---

## Typical Usage

```bash
# Run weekly digest
claude-code task --agent watch_agent \
  --prompt "Generate weekly literature digest for computational pathology, survival models, and validation studies. Flag high-priority papers and create action items."

# Ad-hoc search
claude-code task --agent watch_agent \
  --prompt "Search for recent papers on calibration methods for survival models. Add to index and summarize key findings."
```

---

## Search Sources

### Primary Sources

- **PubMed** - Clinical and biomedical literature
- **arXiv** - Preprints (cs.CV, cs.LG, stat.ML, q-bio)
- **bioRxiv / medRxiv** - Life sciences preprints
- **Google Scholar** - Broad academic search

### Journal-Specific

- Nature Medicine, The Lancet Digital Health, npj Digital Medicine
- Journal of Clinical Oncology, Clinical Cancer Research
- Journal of Pathology Informatics

---

## Priority Levels

### HIGH

- RCTs or large validation studies
- FDA approvals or regulatory updates
- Performance exceeding our current benchmarks
- Novel methods with clear improvement
- Multi-center external validation

### MEDIUM

- Single-center validation studies
- Incremental method improvements
- Review papers summarizing field
- Dataset announcements

### LOW

- Purely methodological papers (no application)
- Small sample studies
- Conference abstracts (not full papers)

---

## Action Item Categories

### Review

- Read paper in detail
- Discuss in journal club
- Evaluate for adoption

### Benchmark

- Add to comparison table
- Compare metrics to our models
- Update competitive landscape

### Implement

- Prototype new method
- Test on our data
- Integrate into pipeline

### Monitor

- Track for follow-up publications
- Watch for external validation
- Check for dataset release

---

## Notes

- **Cadence:** Weekly digest recommended (configurable)
- **Automation:** Can be scheduled via cron or workflow automation
- **Collaboration:** Digest shared with team for discussion
- **Archive:** All papers indexed in `watch/index.csv` for long-term tracking
- **Network required:** WatchAgent needs internet access for literature search

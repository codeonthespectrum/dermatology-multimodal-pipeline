# 🏥 Multimodal Dermatology Database & Intelligent Algorithms

> Research project at the **Federal University of Piauí (UFPI)** — building a multimodal dermatology database and intelligent algorithms for clinical analysis.
>
> This repository hosts the full pipeline across multiple phases:
> - **Phase 1 — RPA:** Automated construction of the multimodal database (rule-based ETL)
> - **Phase 2 — NLP Transformer:** Neural extraction over the same sources (BERTimbau + zero-shot + sentence embeddings)
> - **Phase 3 — Domain Knowlodge Algorithm:** Approch using a ground truth in dermatology to extract and construct a new dataset from the same sources
---
## All pipelines avaliable at `/src`
## Overview

This repository implements a research pipeline for constructing and exploiting a comprehensive dermatology dataset that integrates both **textual** and **visual** clinical data. The ultimate goal is to develop models capable of understanding dermatological content across both modalities.

The work is organized in sequential phases. **Phase 1 (RPA)** is responsible for data acquisition, structuring, and unification using deterministic rules (regex, curated vocabularies, MD5 deduplication). **Phase 2 (NLP Transformer)** revisits the same three sources with a neural pipeline — replacing fixed term lists with semantic discovery, zero-shot classification, and dense embeddings — producing a richer database that includes named entities, entity relations, section classifications, and per-section vector representations ready for multimodal training.

### Research Pipeline

```
Phase 1: RPA              →  Rule-based extraction & structuring
Phase 2: NLP Transformer  →  Neural extraction, NER, relations & embeddings
Phase 3: DSINLP           →  Domain Knowlodge model
```

---

## Data Sources

| Source | Type | Description |
|--------|------|-------------|
| **Azulay's Dermatology** | PDF (1,157 pages) | A comprehensive Brazilian dermatology textbook used as the primary reference across medical schools. Contains structured clinical text, disease descriptions, diagnostic criteria, and clinical images with captions. |
| **HAM10000** | Images + CSV | 10,015 dermoscopic images across 7 diagnostic categories. A well-established benchmark dataset for skin lesion classification. |
| **derm7pt** | Images + Metadata | 2,013 dermoscopic and clinical images annotated with the 7-point checklist — a standardized scoring system used in clinical dermatoscopy. |

---

## License

This project is developed for academic research purposes. The datasets used (HAM10000, derm7pt) are subject to their respective licenses. The Azulay textbook content is used strictly for educational and research purposes within the scope of this university project.

---

## Acknowledgments

- **Federal University of Piauí (UFPI)** — Institutional support
- [HAM10000 Dataset](https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/DBW86T) — Tschandl et al., 2018
- [derm7pt Dataset](https://derm.cs.sfu.ca/Welcome.html) — Kawahara et al., 2018
- Azulay's Dermatology — Primary clinical reference

## Final database
- Available soon for use

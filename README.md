# 🏥 RPA Pipeline for Dermatology Database Construction

> **Phase 1** of an ongoing research project at the **Federal University of Piauí (UFPI)** — Building a multimodal database and intelligent algorithms for dermatology.

---

## Overview

This repository contains the **Robotic Process Automation (RPA)** pipeline developed as the first stage of a research initiative focused on constructing a comprehensive dermatology database that integrates both textual and visual clinical data. The ultimate goal of this research is to build models capable of understanding dermatological content across both modalities — text and images.

The RPA automates the extraction, transformation, and loading (ETL) of data from three heterogeneous sources into a unified, structured SQLite database ready for downstream machine learning tasks.

### Research Pipeline

```
Phase 1: RPA (this repository)     →  Data acquisition & structuring
Phase 2: Transformer               →  Multimodal model training
Phase 3: Custom Algorithm           →  Novel approach for dermatology AI
```

---

## Data Sources

| Source | Type | Description |
|--------|------|-------------|
| **Azulay's Dermatology** | PDF (1,157 pages) | A comprehensive Brazilian dermatology textbook used as the primary reference across medical schools. Contains structured clinical text, disease descriptions, diagnostic criteria, and clinical images with captions. |
| **HAM10000** | Images + CSV | 10,015 dermoscopic images across 7 diagnostic categories. A well-established benchmark dataset for skin lesion classification. |
| **derm7pt** | Images + Metadata | 2,013 dermoscopic and clinical images annotated with the 7-point checklist — a standardized scoring system used in clinical dermatoscopy. |

---

## Architecture

The pipeline is composed of five specialized robots, each responsible for a distinct stage of the automation process:

```
┌─────────────────────────────────────────────────────────────────┐
│                        RPA PIPELINE                             │
├─────────────┬─────────────┬─────────────┬──────────┬────────────┤
│   Robot 1   │   Robot 2   │   Robot 3   │ Robot 4  │  Robot 5   │
│   Azulay    │  HAM10000   │   derm7pt   │ Database │ Validation │
│  Extraction │  Processing │  Processing │ Builder  │  & Report  │
└──────┬──────┴──────┬──────┴──────┬──────┴─────┬────┴─────┬──────┘
       │             │             │            │          │
       ▼             ▼             ▼            ▼          ▼
   Text, Images  Images +     Images +     Unified     Quality
   & Structure   Metadata     7-pt Data    SQLite DB   Report
```

### Robot 1 — Azulay Extraction

Processes the 1,157-page PDF using PyMuPDF with the following sub-tasks:

- **Text extraction** — Page-by-page raw text extraction
- **Hierarchical structure detection** — Identifies chapters, sections, and subsections through regex pattern matching on heading styles (uppercase titles, numbered chapters, capitalized subtitles)
- **Image extraction** — Extracts embedded images, filters artifacts (icons, backgrounds), deduplicates via MD5 hashing, and associates images with their captions using proximity-based matching
- **Dermatological enrichment** — Scans each section against a curated vocabulary of 80+ clinical terms (diseases, lesion morphology, anatomical structures, therapeutic procedures) and indexes them for search

### Robot 2 — HAM10000 Processing

- Reads the metadata CSV and maps each record to its corresponding image file through recursive directory traversal
- Enriches diagnostic codes with full Portuguese names, clinical descriptions, and ICD-10 codes
- Extracts image dimensions and computes integrity hashes
- Handles the 7 diagnostic categories: melanoma (mel), basal cell carcinoma (bcc), melanocytic nevi (nv), actinic keratoses (akiec), benign keratosis-like lesions (bkl), dermatofibroma (df), and vascular lesions (vasc)

### Robot 3 — derm7pt Processing

- Dynamically maps the dataset directory structure, handling 35+ image subdirectories
- Parses the metadata CSV with automatic column detection for diagnosis and image ID fields
- Normalizes file path references (e.g., `NEL/Nel026.jpg` → `Nel026`) for correct metadata-to-image association
- Extracts the 7-point checklist criteria: pigment network, blue-whitish veil, vascular structures, pigmentation, streaks, dots and globules, and regression structures

### Robot 4 — Unified Database Builder

Constructs a relational SQLite database with the following schema:

```
capitulos ─────────── secoes_texto ─────── relacao_secao_termo ─── termos_dermatologicos
                           │
                      imagens_azulay
                      
imagens_ham10000 ──── diagnosticos ──── imagens_derm7pt

rpa_metadata
```

**8 tables** with foreign keys, indexes on diagnostic codes, anatomical locations, chapter numbers, and term occurrence counts for optimized querying.

### Robot 5 — Validation & Quality Report

- Verifies record counts and referential integrity across all tables
- Computes a completeness score (0–100) based on source coverage
- Generates a detailed JSON report with per-source statistics, error logs, and actionable recommendations
- Identifies potential data quality issues (e.g., sections with fewer than 10 words)

---

## Results
<img width="2680" height="1034" alt="06_cross_dataset" src="https://github.com/user-attachments/assets/e0f2affb-538b-4e4f-88a4-2dca697a0159" />
<img width="2680" height="887" alt="07_qualidade_dados" src="https://github.com/user-attachments/assets/1019cf87-a29f-41a2-89d5-1e1f23514f2e" />



### Database Summary

| Metric | Value |
|--------|-------|
| **Total records** | 24,805 |
| **Text sections (Azulay)** | 10,299 |
| **Words extracted** | 683,023 |
| **Images from Azulay** | 2,353 |
| **Images from HAM10000** | 10,015 |
| **Images from derm7pt** | 2,013 |
| **Chapters identified** | 20 |
| **Dermatological terms indexed** | 99 |
| **Unique diagnoses** | 8+ |
| **Pipeline errors** | 0 |
| **Completeness score** | 100/100 |

### Diagnostic Distribution — HAM10000

| Code | Diagnosis | Count |
|------|-----------|-------|
| nv | Melanocytic Nevi | 6,705 |
| mel | Melanoma | 1,113 |
| bkl | Benign Keratosis | 1,099 |
| bcc | Basal Cell Carcinoma | 514 |
| akiec | Actinic Keratoses | 327 |
| vasc | Vascular Lesions | 142 |
| df | Dermatofibroma | 115 |

### Top 10 Dermatological Terms (Azulay)

| Term | Occurrences |
|------|-------------|
| Erythema | 699 |
| Lesion | 525 |
| Dermis | 441 |
| Ulcer | 376 |
| Tumor | 369 |
| Vascular | 320 |
| Pruritus | 311 |
| Dermatitis | 286 |
| Plaque | 279 |
| Scar | 276 |

---

## Output Files

The pipeline generates the following artifacts, ready for downstream model training:

```
RPA_Output/
├── dermatologia.db                    # Unified SQLite database
├── relatorio_rpa.json                 # Detailed quality report
├── textos_azulay.jsonl                # Structured text for Transformer training
├── imagens_metadata_unificado.csv     # Unified image metadata across datasets
├── pares_texto_imagem.jsonl           # Text-image pairs for multimodal training
└── imagens_extraidas/
    ├── azulay/                        # Images extracted from the textbook
    ├── ham10000/                      # (references to original paths)
    └── derm7pt/                       # (references to original paths)
```

## Tech Stack

- **Python 3.10+**
- **PyMuPDF (fitz)** — PDF text and image extraction
- **Pillow** — Image processing and validation
- **Pandas** — Metadata handling and data transformation
- **SQLite3** — Relational database storage
- **kagglehub** — Dataset acquisition
- **Matplotlib / Seaborn** — Visualization and analysis

---

## Project Context

This work is part of an undergraduate research project (Iniciação Científica) at the **Federal University of Piauí (UFPI)**, focused on building databases and algorithms for healthcare with an emphasis on dermatology.

The research follows a three-phase approach:

1. **RPA** (this repo) — Automated construction of a multimodal dermatology database from heterogeneous sources
2. **Transformer** — Training a model capable of understanding both textual clinical descriptions and dermoscopic images
3. **Custom Algorithm** — Development of a novel approach for dermatological analysis

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

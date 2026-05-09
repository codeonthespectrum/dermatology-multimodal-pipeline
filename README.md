# 🏥 Multimodal Dermatology Database & Intelligent Algorithms

> Ongoing research project at the **Federal University of Piauí (UFPI)** — building a multimodal dermatology database and intelligent algorithms for clinical analysis.
>
> This repository hosts the full pipeline across multiple phases:
> - **Phase 1 — RPA:** Automated construction of the multimodal database (rule-based ETL)
> - **Phase 2 — NLP Transformer:** Neural extraction over the same sources (BERTimbau + zero-shot + sentence embeddings)
> - **Phase 3 — Custom Algorithm:** Novel multimodal approach for dermatology AI *(on going)*

---

## Overview

This repository implements a research pipeline for constructing and exploiting a comprehensive dermatology dataset that integrates both **textual** and **visual** clinical data. The ultimate goal is to develop models capable of understanding dermatological content across both modalities.

The work is organized in sequential phases. **Phase 1 (RPA)** is responsible for data acquisition, structuring, and unification using deterministic rules (regex, curated vocabularies, MD5 deduplication). **Phase 2 (NLP Transformer)** revisits the same three sources with a neural pipeline — replacing fixed term lists with semantic discovery, zero-shot classification, and dense embeddings — producing a richer database that includes named entities, entity relations, section classifications, and per-section vector representations ready for multimodal training.

### Research Pipeline

```
Phase 1: RPA              →  Rule-based extraction & structuring
Phase 2: NLP Transformer  →  Neural extraction, NER, relations & embeddings
Phase 3: Algorithm        →  Multimodal model & novel approach for dermatology AI
```

### RPA vs. NLP Transformer at a glance

| Aspect | Phase 1 (RPA) | Phase 2 (NLP Transformer) |
|---|---|---|
| Term detection | Fixed list of 99 terms via regex | Semantic discovery via sentence embeddings + zero-shot |
| Section structuring | Heading regex on the PDF | Same structuring **+** zero-shot classification into 20 clinical categories |
| Output per section | Text + matched terms | Text + NER entities + relations + classification + embedding |
| Clinical variables (HAM10000 / derm7pt) | Diagnosis only | Diagnosis **+ age (with Z-score), sex, anatomical localization** |
| Database tables | 8 | 13 (8 RPA-equivalent + 5 neural) |

---

## Data Sources

| Source | Type | Description |
|--------|------|-------------|
| **Azulay's Dermatology** | PDF (1,157 pages) | A comprehensive Brazilian dermatology textbook used as the primary reference across medical schools. Contains structured clinical text, disease descriptions, diagnostic criteria, and clinical images with captions. |
| **HAM10000** | Images + CSV | 10,015 dermoscopic images across 7 diagnostic categories. A well-established benchmark dataset for skin lesion classification. |
| **derm7pt** | Images + Metadata | 2,013 dermoscopic and clinical images annotated with the 7-point checklist — a standardized scoring system used in clinical dermatoscopy. |

---

# Phase 1 — RPA (Data Acquisition & Structuring)

The **Robotic Process Automation (RPA)** pipeline automates the extraction, transformation, and loading (ETL) of data from three heterogeneous sources into a unified, structured SQLite database ready for downstream tasks.

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

## RPA Output Files

```
RPA_Output/
├── dermatologia.db                    # Unified SQLite database
├── relatorio_rpa.json                 # Detailed quality report
├── textos_azulay.jsonl                # Structured text for downstream training
├── imagens_metadata_unificado.csv     # Unified image metadata across datasets
├── pares_texto_imagem.jsonl           # Text-image pairs for multimodal training
└── imagens_extraidas/
    ├── azulay/                        # Images extracted from the textbook
    ├── ham10000/                      # (references to original paths)
    └── derm7pt/                       # (references to original paths)
```

---

# Phase 2 — NLP Transformer (Neural Extraction)

This phase reprocesses the same three sources with a neural pipeline built around **BERTimbau**, **multilingual sentence-transformers**, and **zero-shot classification with mDeBERTa**. It replaces the rule-based components of the RPA with semantic methods, enriches the records with clinical variables, and adds entirely new tables for named entities, entity relations, section classifications, and dense embeddings.

## Models

| Role | Model |
|---|---|
| Token-level Portuguese encoder | `neuralmind/bert-base-portuguese-cased` (BERTimbau) |
| Sentence embeddings | `paraphrase-multilingual-MiniLM-L12-v2` |
| Zero-shot classification | `MoritzLaurer/mDeBERTa-v3-base-mnli-xnli` |
| Linguistic backbone | spaCy `pt_core_news_sm` (tokenization, POS, dependencies, NER, noun chunks) |

## Architecture

The pipeline mirrors the RPA's five-robot structure to allow direct comparison, but each robot has neural responsibilities:

```
┌─────────────────────────────────────────────────────────────────┐
│                    NLP TRANSFORMER PIPELINE                     │
├─────────────┬─────────────┬─────────────┬──────────┬────────────┤
│   Robot 1   │   Robot 2   │   Robot 3   │ Robot 4  │  Robot 5   │
│  Azulay +   │  HAM10000   │   derm7pt   │ Database │ Validation │
│  NER + Cls  │  + clinical │  + clinical │ Builder  │ + NLP mtrx │
│  + Embeds   │   vars      │    vars     │ (13 tbl) │            │
└──────┬──────┴──────┬──────┴──────┬──────┴─────┬────┴─────┬──────┘
       │             │             │            │          │
       ▼             ▼             ▼            ▼          ▼
  Sections +     Images +      Images +     Unified     Quality +
  NER + rels +   age/sex/loc   age/sex/loc  SQLite DB   NER metrics
  classif +
  embeddings
```

### Robot 1 — Azulay Neural Extraction

Same PDF parsing as the RPA (text, hierarchy, images), plus four neural layers per section:

1. **Named Entity Recognition (3-layer hybrid).**
   - **Layer 1 — spaCy NER:** general entities + noun-chunk candidates from the Portuguese spaCy model.
   - **Layer 2 — Semantic discovery:** ~180 medical seed terms (the 99 RPA terms + dermatology-specific anchors) are encoded once with sentence-transformers; candidate phrases are batch-encoded and matched by cosine similarity.
   - **Layer 3 — Categorization:** each entity is assigned to one of **16 fine-grained categories** (DOENÇA, NEOPLASIA, INFECÇÃO, AUTOIMUNE, INFLAMATÓRIA, SINTOMA, LESÃO_ELEMENTAR, MORFOLOGIA, ESTRUTURA, LOCALIZAÇÃO, PROCEDIMENTO, TRATAMENTO, MEDICAMENTO, EXAME, HISTOPATOLOGIA, PATÓGENO) via the closest seed (argmax over the similarity matrix).
   - Optimization note: an earlier version ran zero-shot per entity (~168 s/section). The current similarity-based approach drops it to ~1–3 s/section while preserving fine-grained categories.
2. **Relation extraction** between co-occurring entities in the same sentence, inferred from category pairs (e.g., TRATAMENTO–DOENÇA → `TRATA`, SINTOMA–DOENÇA → `MANIFESTA`, LOCALIZAÇÃO–DOENÇA → `LOCALIZA_EM`, etc.) with dependency-aware heuristics.
3. **Section classification (zero-shot):** each section is assigned to one of **20 clinical categories** (inflammatory dermatosis, benign/malignant neoplasms, bacterial/fungal/viral/parasitic infections, autoimmune, bullous, allergic, pigmentation disorders, vascular, adnexal, genodermatoses, occupational, drug eruptions, anatomy & histology, semiology, therapy, etc.) using `multi_label=True` over the first 1000 chars.
4. **Sentence embeddings** for every section, persisted as float vectors in the database for retrieval and downstream multimodal use.

### Robot 2 — HAM10000 (clinically enriched)

Everything from the RPA, **plus** clinical variables extracted from the metadata CSV:

- `idade` (age) with `idade_zscore` computed against the cohort median/std
- `sexo` (male / female / unknown) with a numeric `sexo_codigo`
- `localizacao` (back, face, trunk, lower extremity, etc.) with a numeric `localizacao_codigo`

These are stored alongside the existing diagnosis fields and indexed for query.

### Robot 3 — derm7pt (clinically enriched)

Same structure mapping and 7-point parsing as the RPA, **plus** the same `age / sex / localization` triple (with Z-score on age) wherever the metadata exposes them. Diagnostic labels are normalized to a Portuguese display form.

### Robot 4 — Unified Neural Database

Builds the SQLite database, keeping the 8 RPA-equivalent tables and adding 5 neural tables:

```
─── RPA-equivalent (kept) ─────────────────────────────────────────
capitulos · secoes_texto · imagens_azulay · imagens_ham10000
imagens_derm7pt · diagnosticos · termos_dermatologicos
relacao_secao_termo

─── NLP Transformer (new) ─────────────────────────────────────────
entidades_ner          (texto, categoria, score_confianca, secao_id,
                        posicao_inicio, posicao_fim, metodo_deteccao)
relacoes_entidades     (entidade_origem, entidade_destino, tipo_relacao,
                        score_confianca, secao_id, sentenca)
classificacoes_secao   (secao_id, categoria_principal, score_principal,
                        todas_categorias)
embeddings_secao       (secao_id, vetor, modelo, dimensao)
pipeline_metadata      (run info + model versions)
```

Indexes are added on `entidades_ner(secao_id, categoria, texto)`, `relacoes_entidades(secao_id)`, `classificacoes_secao(categoria_principal)`, and on the new clinical fields `imagens_ham10000(localizacao, sexo)` / `imagens_derm7pt(sexo)`.

### Robot 5 — Validation & NLP Metrics

All RPA-style integrity checks **plus** NLP-specific reporting:

- Entity counts and mean confidence per category (16 categories)
- Distribution of section classifications and mean confidence
- Embedding coverage across sections
- Detection-method breakdown (`spacy_ner+seed` vs. `transformer_discovery`) with mean confidence per method
- Clinical-variable summaries (age mean/min/max, sex split, distinct localizations)
- Top 30 discovered terms with their assigned category and confidence

The output is a single `relatorio_nlp_transformer.json` plus 8 PNG charts (NER per category, top-30 terms, term cloud, section classifications, relation types, dataset diagnostics, clinical variables, and detection methods).

## NLP Transformer Output Files

```
NLP_Transformer_Output/
├── dermatologia_transformer.db        # Unified SQLite (13 tables)
├── relatorio_nlp_transformer.json     # Quality + NLP metrics report
├── embeddings/                        # Per-section sentence embeddings
├── imagens_extraidas/
│   ├── azulay/
│   ├── ham10000/
│   └── derm7pt/
├── 01_entidades_por_categoria.png     # NER per category
├── 02_top30_termos.png                # Top 30 discovered terms
├── 03_nuvem_palavras_transformer.png  # Term cloud
├── 04_classificacao_secoes.png        # Sections by clinical category
├── 05_relacoes_entidades.png          # Relation types (TRATA, MANIFESTA, …)
├── 06_diagnosticos_datasets.png       # HAM10000 + derm7pt distribution
├── 07_variaveis_clinicas.png          # Age, sex, localization
└── 08_metodos_deteccao.png            # spaCy vs. Transformer discovery
```

---

## Tech Stack

- **Python 3.10+**
- **PyMuPDF (fitz)** — PDF text and image extraction
- **Pillow** — Image processing and validation
- **Pandas / NumPy** — Metadata handling and numerical processing
- **SQLite3** — Relational database storage
- **kagglehub** — Dataset acquisition
- **Matplotlib / Seaborn** — Visualization and analysis
- **PyTorch + Hugging Face Transformers** — BERTimbau and mDeBERTa zero-shot
- **sentence-transformers** — Multilingual sentence embeddings
- **spaCy** (`pt_core_news_sm`) — Tokenization, POS, dependencies, NER
- **scikit-learn** — Cosine similarity, label encoding, evaluation metrics
- **unidecode / tqdm** — Text normalization and progress reporting

---

## Project Context

This work is part of an undergraduate research project (Iniciação Científica) at the **Federal University of Piauí (UFPI)**, focused on building databases and algorithms for healthcare with an emphasis on dermatology.

The research follows a three-phase approach:

1. **RPA** — Rule-based construction of a multimodal dermatology database from heterogeneous sources
2. **NLP Transformer** — Neural extraction over the same sources, adding NER, relations, classification, embeddings, and clinical variables
3. **Custom Algorithm** — Development of a novel multimodal approach for dermatological analysis

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

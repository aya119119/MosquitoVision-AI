# 🦟 MosquitoVision AI

**A CNN built from scratch to identify mosquito genera (Aedes · Anopheles · Culex) from photographs, to support public-health surveillance.**

![Python](https://img.shields.io/badge/Python-3.10+-blue) ![PyTorch](https://img.shields.io/badge/PyTorch-CNN-red) ![Status](https://img.shields.io/badge/status-in%20progress-orange)

> ⚠️ **Research prototype.** It identifies mosquito *genus* from a photo. It does **not** detect pathogens, identify exact species, or replace expert entomologists.

---

## 📌 Table of contents
1. [Motivation](#1-motivation)
2. [Project goal](#2-project-goal)
3. [Dataset](#3-dataset)
4. [Progress so far](#4-progress-so-far)
5. [Roadmap](#5-roadmap)
6. [Repository structure](#6-repository-structure)
7. [How to run](#7-how-to-run)
8. [Scientific limitations](#8-scientific-limitations)
9. [References](#9-references)

---

## 1. Motivation
Some mosquitoes are disease vectors: *Aedes* (dengue, Zika, chikungunya, yellow fever), *Anopheles* (malaria), *Culex* (West Nile virus, Japanese encephalitis). According to the WHO, vector-borne diseases cause more than 700,000 deaths per year.
Identifying mosquitoes traditionally needs trained specialists examining physical features under a microscope, which is slow and hard to scale. An automatic image classifier could help surveillance teams triage specimens faster.

## 2. Project goal
Train a **custom CNN (no pretrained classifier)** that takes a mosquito photo and returns:
- the predicted genus,
- a confidence score,
- a short public-health context,
- a warning when the prediction is uncertain ("expert verification recommended").

## 3. Dataset
**MosqVision-3K** (Masud, 2024) – about 3,000 JPEG images, 3 classes, from MosquitoAlert, Mendeley Data, IEEE DataPort and Dryad.
- Source: https://zenodo.org/records/14238701 (DOI `10.5281/zenodo.14238701`)
- ⚠️ The dataset is **not included** in this repo. The notebook downloads it automatically. Check Zenodo's reuse terms before redistributing images.

## 4. Progress so far

### ✅ Phase 0 – Environment & data download
- Google Colab + Google Drive project structure
- Dataset downloaded from Zenodo by code (Zenodo API with fallback URL), archive inspected *before* extraction
- Fixed random seed (42) for reproducibility

### ✅ Phase 1 – Dataset understanding & quality audit
Principle: *a model is only as good as its data, so the data was audited before any modelling.*

| Check | Why |
|---|---|
| Images per class | Is the dataset balanced? |
| Sample gallery | Look at the data with human eyes |
| Corrupted / tiny files | Prevent crashes and useless inputs |
| Size, format, brightness per class | Detect *shortcuts* (e.g. one class from a different camera) |
| Exact duplicates (MD5) + near-duplicates (perceptual hash) | Prevent train/test leakage |
| Same image under two labels | Detect labelling errors |

**Class distribution**

![Class distribution](docs/images/class_distribution_raw.png)

**Sample images**

![Sample gallery](docs/images/sample_gallery.png)

**Image size & brightness per class (shortcut check)**

![Size and brightness](docs/images/size_brightness_by_class.png)

**Key numbers** *(fill in from your own run – `results/dataset_summary.txt`)*

| Item | Value |
|---|---|
| Total images | `...` |
| Unreadable images | `...` |
| Exact duplicates | `...` |
| Redundant near-duplicates | `...` |
| Images with conflicting labels (removed) | `...` |

**What I observed:** `...` *(write 2–3 sentences about the gallery, class balance and any shortcut you noticed)*

### ✅ Phase 2 – Preprocessing & leak-free split

| Step | What | Why |
|---|---|---|
| Resize | 128 × 128 × 3 | CNN needs fixed input size; keeps training light |
| Normalise | pixel / 255 → [0, 1] | stabler training |
| Label encoding | Aedes 0 · Anopheles 1 · Culex 2 | networks use numbers |
| Split | 70 % train / 15 % val / 15 % test | train, tune, final honest evaluation |
| Split method | **Stratified + group-aware** | classes stay balanced, near-duplicates never cross splits |
| Verification | assertions on cluster IDs and MD5 hashes | code fails if any leak exists |
| Augmentation | flips, ±15° rotation, small crops, brightness/contrast | reduces memorisation – **train only, after the split** |

**Split sizes** *(fill in from `results/split_table.csv`)*

| Split | Aedes | Anopheles | Culex | Total |
|---|---|---|---|---|
| Train | | | | |
| Validation | | | | |
| Test | | | | |

**Original vs augmented images**

![Augmentation](docs/images/augmentation_examples.png)

## 5. Roadmap

| Phase | Description | Status |
|---|---|---|
| 0 | Setup & data download | ✅ Done |
| 1 | Data exploration & quality audit | ✅ Done |
| 2 | Preprocessing & leak-free split | ✅ Done |
| 3 | Custom CNN in PyTorch | 🔄 In progress |
| 4 | Training, overfitting analysis, improvements | ⏳ Planned |
| 5 | Scientific evaluation (precision, recall, F1, confusion matrix, error analysis) | ⏳ Planned |
| 6 | Streamlit application with uncertainty warning | ⏳ Planned |
| Ext. | Grad-CAM · pretrained benchmark · robustness test | ⏳ Optional |

**Planned CNN architecture:** 3 × [Conv 3×3 → ReLU → MaxPool] (32 → 64 → 128 filters) → Global Average Pooling → Dropout → Linear(3).

## 6. Repository structure
```
MosquitoVision/
├── notebooks/MosquitoVision_AI.ipynb   # full pipeline
├── docs/images/                        # figures used in this README
├── results/                            # plots, reports (added as phases finish)
├── models/                             # trained weights (Phase 4+)
├── app/                                # Streamlit app (Phase 6)
├── requirements.txt
└── README.md
```

## 7. How to run
1. Open `notebooks/MosquitoVision_AI.ipynb` in **Google Colab** (Runtime → T4 GPU).
2. Run the cells from top to bottom. Phase 0 mounts Google Drive and downloads the dataset automatically.

## 8. Scientific limitations
1. Mosquito identification is **not** pathogen detection.
2. Labels are **genus-level**, not species-level.
3. A prediction is **not** a confirmed identification: experts must verify uncertain cases.
4. The dataset is multi-source and **not representative of Moroccan mosquito populations**.
5. Good test accuracy would **not** prove real-world effectiveness (different cameras, backgrounds and lighting must be tested).

## 9. References
1. WHO – *Vector-borne diseases* fact sheet. https://www.who.int/en/news-room/fact-sheets/detail/vector-borne-diseases
2. Pise et al. (2022) – *Dataset of Vector Mosquito Images*, Data in Brief. https://doi.org/10.1016/j.dib.2022.108573
3. Masud (2024) – *MosqVision-3K*, Zenodo. https://zenodo.org/records/14238701
4. Nolte et al. (2024) – *Robust mosquito species identification from diverse body and wing images using deep learning*, Parasites & Vectors. https://doi.org/10.1186/s13071-024-06459-3

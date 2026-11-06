# 🦟 MosquitoVision AI

**A CNN built from scratch (PyTorch) that identifies mosquito genus — *Aedes*, *Anopheles* or *Culex* — from a photograph, to support public-health surveillance.**

> ⚠️ **Research prototype.** It identifies mosquito *genus* from an image. It does **not** detect pathogens, identify exact species, or replace expert entomologists.

---

## 1. Why this project?
Mosquitoes are disease vectors. *Aedes* is associated with dengue, Zika, chikungunya and yellow fever; *Anopheles* with malaria; *Culex* with West Nile virus and Japanese encephalitis. The WHO reports more than 700,000 deaths per year from vector-borne diseases.
Identifying mosquitoes traditionally requires trained specialists examining physical features, which is slow and hard to scale. A reliable image classifier could help surveillance teams triage specimens faster, and flag the uncertain ones for experts.

## 2. Goal
Train a **custom CNN (no pretrained classifier)** that takes a photo and returns the predicted genus, a confidence score, a short public-health context, and an *"uncertain — expert verification recommended"* warning when the model is not sure.

## 3. Dataset
**MosqVision-3K** (Masud, 2024) — JPEG photos of three genera, merged from MosquitoAlert, Mendeley Data, IEEE DataPort and Dryad.
Source: https://zenodo.org/records/14238701 · DOI `10.5281/zenodo.14238701`.
The dataset is **not included** in this repository (the notebook downloads it automatically; check Zenodo's reuse terms before redistributing images).

---

## 4. Phase 1 — Understanding the data (quality audit)

**Principle:** a model is only as good as its data, so the data was audited *before* any modelling.

The notebook scans every image once and records its label, format, colour mode, width, height, brightness, an MD5 fingerprint (finds exact copies) and a 64-bit **perceptual hash** (finds near-identical images).

### 4.1 Class balance
| label | images | percent |
|---|---|---|
| Aedes | 1000 | 33.3 |
| Anopheles | 1000 | 33.3 |
| Culex | 1000 | 33.3 |

Total images found: **3000** · unreadable/corrupted: **0**

![Number of images per class](docs/images/class_distribution_raw.png)

*Number of images per class*


Balanced classes make plain accuracy meaningful; with imbalance, per-class recall and F1 would matter more.

### 4.2 Looking at the data
![Random samples from each class](docs/images/sample_gallery.png)

*Random samples from each class*


### 4.3 Shortcut check (size & brightness per class)
The dataset merges several sources. If one class had been photographed with a different camera or resolution, the CNN could learn *"this looks like source X"* instead of *"this is an Aedes"*. The plots compare image size and brightness across classes.

![Image size and brightness by class](docs/images/size_brightness_by_class.png)

*Image size and brightness by class*


### 4.4 Duplicates and label conflicts
If the same photo appears in both the training and test sets, the model is graded on something it has already memorised and the accuracy is fake. Exact copies are found with MD5; near-copies with a perceptual hash (Hamming distance ≤ 4 of 64 bits). Duplicates are merged into *clusters*.

| Check | Result |
|---|---|
| Exact duplicate files | **1919** |
| Redundant exact + near-duplicate images | **1923** |
| Images in clusters with conflicting labels (removed) | **0** |

---

## 5. Phase 2 — Preprocessing and leak-free split

| Step | What | Why |
|---|---|---|
| Resize | 128×128×3 (49,152 numbers per image) | the CNN needs a fixed input size; keeps training light |
| Normalise | pixel / 255 → [0, 1] | smaller, consistent numbers make training stabler |
| Label encoding | Aedes 0 · Anopheles 1 · Culex 2 | networks work with numbers |
| Split | 70 % train / 15 % val / 15 % test | train, tune, and a final honest exam |
| Split method | **stratified + duplicate-aware** | class proportions preserved; near-duplicates never cross splits |
| Verification | assertions on cluster IDs and MD5 hashes | the code stops if any leak exists |
| Augmentation | flips, ±15° rotation, small crops, brightness/contrast | reduces memorisation — **train only, after the split** |

### Split sizes
| split | Aedes | Anopheles | Culex | total | share_% |
|---|---|---|---|---|---|
| train | 715 | 685 | 651 | 2051 | 68.4 |
| val | 132 | 155 | 122 | 409 | 13.6 |
| test | 153 | 160 | 227 | 540 | 18.0 |

### Original vs augmented images
![Original image (left) and random augmentations](docs/images/augmentation_examples.png)

*Original image (left) and random augmentations*


Validation and test images are never augmented: they must represent real, unmodified photos.

---

## 6. Phase 3 — The CNN (built from scratch)

Conv blocks learn visual patterns (edges → textures → shapes), ReLU adds non-linearity, max-pooling shrinks the maps, global average pooling reduces parameters, dropout fights overfitting, and a final linear layer outputs 3 scores.

![Baseline architecture](docs/images/architecture_diagram.png)

*Baseline architecture*


---

## 7. Phase 4 — Training and improvement

Loss: CrossEntropy · Optimiser: Adam (lr 0.001) · Batch size 32 · Early stopping + best-validation checkpoint. Several experiments were compared on **validation** data only (the test set stayed untouched).

| name | params | best_epoch | train_acc_at_best | val_acc_at_best | train_val_gap | best_val_loss | notes |
|---|---|---|---|---|---|---|---|
| exp0_no_augmentation | 93635 | 25 | 0.9473 | 0.9878 | -0.0404 | 0.0748 | baseline CNN, NO augmentation (ablation) |
| exp1_baseline | 93635 | 30 | 0.8986 | 0.9413 | -0.0427 | 0.1597 | roadmap baseline + augmentation |

![Training and validation loss/accuracy of the selected model](docs/images/training_history.png)

*Training and validation loss/accuracy of the selected model*


A large gap between training and validation accuracy indicates overfitting.

---

## 9. Extensions

![Grad-CAM: regions that drove the prediction](docs/images/gradcam_examples.png)

*Grad-CAM: regions that drove the prediction*

![Accuracy under simulated image degradations](docs/images/robustness.png)

*Accuracy under simulated image degradations*


---

## Scientific limitations
1. Mosquito identification is **not** pathogen detection.
2. Labels are **genus-level**, not species-level.
3. A prediction is **not** a confirmed identification; experts must verify uncertain cases.
4. The dataset is multi-source and **not representative of Moroccan mosquito populations**.
5. Good test accuracy does **not** prove real-world effectiveness (other cameras, backgrounds and lighting must be tested).

## How to run
Open `MosquitoVision_AI.ipynb` in Google Colab (Runtime → T4 GPU) and run the cells top to bottom. The dataset downloads automatically.

## References
1. WHO — *Vector-borne diseases* fact sheet. https://www.who.int/en/news-room/fact-sheets/detail/vector-borne-diseases
2. Pise et al. (2022) — *Dataset of Vector Mosquito Images*, Data in Brief. https://doi.org/10.1016/j.dib.2022.108573
3. Masud (2024) — *MosqVision-3K*, Zenodo. https://zenodo.org/records/14238701
4. Nolte et al. (2024) — *Robust mosquito species identification from diverse body and wing images using deep learning*, Parasites & Vectors. https://doi.org/10.1186/s13071-024-06459-3

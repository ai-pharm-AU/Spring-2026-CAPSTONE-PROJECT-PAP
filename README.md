# Spring 2026 Capstone Project - Group 1

# Plant Promoter Activity Prediction Using Machine Learning

![Python](https://img.shields.io/badge/Python-3.12+-blue) ![PyTorch](https://img.shields.io/badge/PyTorch-2.x-orange) ![HuggingFace](https://img.shields.io/badge/HuggingFace-Transformers-yellow) ![Status](https://img.shields.io/badge/Status-Capstone%20Project-blue) ![Platform](https://img.shields.io/badge/Platform-Google%20Colab-yellow)

---

## Overview

Predicting how strongly a plant promoter drives gene expression — straight from its DNA sequence — is one of the core problems in plant synthetic biology and crop improvement. This capstone trains and compares **six machine-learning models** on the same large pan-plant STARR-seq dataset spanning *Arabidopsis thaliana*, *Zea mays*, and *Sorghum bicolor*, so the only thing that varies between experiments is the model architecture and the feature representation.

Starting from **79,842** raw promoter sequences (170 bp) paired with activity measurements across six experimental conditions (tobacco vs maize, ± enhancer, light vs dark), preprocessing and an inner-join on `gene` give a clean **67,860-promoter** working dataset that every model is trained and evaluated on.

The six models fall into three families:

* **Deep learning** — three iterations of a CNN + BiLSTM hybrid that fuses one-hot DNA with frozen embeddings from the GENERator-v2-eukaryote-1.2b genomic language model (v1 baseline → v2 improved → Final )
* **Linear** — ElasticNet regression with L1+L2 regularisation, trained per condition with in-silico mutagenesis for interpretability
* **Gradient boosting** — XGBoost baseline plus a CatBoost vs improved-XGBoost head-to-head with enriched 1,034-dim features

All models are scored on the same held-out test split using **R²**, **RMSE**, and **Pearson r** for regression and **Accuracy** / **F1** / **AUROC** for the Final classifier.

---

## Method

### Models

* **CNN-BiLSTM v1** (initial hybrid; expression input — leaky baseline)
* **CNN-BiLSTM v2** (dropout, validation monitoring, pos-weighted BCE, threshold tuning, LR scheduling)
* **CNN-BiLSTM Final** (no expression input, AUC-based selection, max\|expr\| > 3.0 label rule)
* **ElasticNet** regression (L1+L2, per-condition, in-silico mutagenesis)
* **XGBoost** regression (baseline)
* **XGBoost vs CatBoost** (improved features + log-transformed targets)

### Sequence Embeddings

* **GENERator-v2-eukaryote-1.2b-base** — frozen 1.2 B-parameter genomic language model
* Mean-pooled last-hidden-state vectors of length **2048**
* Cached as `X_gen.npy` (67,860 × 2048) to avoid re-running the forward pass

### Training Strategy

* Transfer learning from frozen genomic foundation model
* Stratified **70 / 15 / 15** train / validation / test split (`random_state=42`)
* Adam optimiser (`lr=1e-3`, `weight_decay=1e-4`)
* `ReduceLROnPlateau` scheduler with best-AUC checkpointing for the Final model
* Per-condition models trained for ElasticNet, XGBoost, and CatBoost (six per family)
* Evaluation metrics: **R²**, **RMSE**, **Pearson r** (regression); **Accuracy**, **F1**, **AUROC** (classification)

### Datasets

* **Supplementary Table 1** — 79,842 plant promoter sequences (≤ 200 bp) with species, GC content, strand
* **Supplementary Table 2** — 78,677 log2-normalised activity rows across 6 conditions
* Inner-joined on `gene` → **67,860 promoters** (final working dataset)
* Source: AlphaGenome pan-plant STARR-seq dataset (Avsec et al., *Nature*, 2026)

### Tools

* Python 3.12
* PyTorch (CNN, BiLSTM, training loops)
* Hugging Face Transformers (GENERator embedding extraction)
* XGBoost / CatBoost / scikit-learn
* pandas / numpy / matplotlib / seaborn
* Google Colab Pro (T4 GPU) for deep models; CPU for tree-based models

---

## Results

### CNN-BiLSTM Final

* True sequence-only classifier (no expression input)
* Per-species confusion matrices and ROC curves consistent across Arabidopsis, Maize, Sorghum
* Predictions saved for all **67,860** promoters: 30,537 functional / 37,323 non-functional
* Best valid classifier in the study

### ElasticNet (regression)

* **Avg test R²: 0.33 – 0.40**
* Best condition R²: **0.45** (with enhancer + light, tobacco)
* Strength: interpretable, sparse, fast; in-silico mutagenesis identifies positional importance

### XGBoost baseline (regression)

* **Avg test R²: 0.534**  |  Avg test RMSE: 0.960
* Best condition R²: **0.584** (with enhancer + light, tobacco)
* Train–test gap: ~0.243 (moderate overfitting)

### CatBoost (regression — overall winner)

* **Avg test R²: 0.58 – 0.60**
* Best condition R²: **~0.59** (with enhancer + light, tobacco)
* Smallest train–test gap of the three tree-based variants

### Key Findings

* Every regression model posts its highest R² on the **"with enhancer + light, tobacco"** condition
* **CatBoost** wins quantitative regression; the **Final no-leakage CNN+BiLSTM+GENERator** wins functional classification
* Data leakage in v1 produced near-perfect (~99.997% AUC) but **invalid** metrics; the Final redesign demonstrates that sequence-only prediction is genuinely possible

---

## Contributors

* **Abhipsha Sahoo** — Auburn University
* **Yeshaswee Sai Ganesh Volety** — Auburn University


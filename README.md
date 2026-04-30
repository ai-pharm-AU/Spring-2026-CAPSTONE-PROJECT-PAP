Spring 2026 Capstone Project

Plant Promoter Activity Prediction Using Machine Learning

Overview
This project explores how machine-learning model architecture and feature representation impact the prediction of plant promoter activity from DNA sequence. Most prior work either uses simple classical models on hand-engineered features or large deep models without rigorously controlling for label leakage.
We evaluate whether combining a CNN+BiLSTM hybrid with frozen GENERator-v2 foundation-model embeddings, alongside ElasticNet and gradient-boosted-tree baselines, improves prediction accuracy and generalisation across three plant species (Arabidopsis thaliana, Zea mays, Sorghum bicolor) and six STARR-seq experimental conditions.

Method
Models

* CNN-BiLSTM v1 (initial hybrid; expression input — leaky baseline)
* CNN-BiLSTM v2 (dropout, val monitoring, pos-weighted BCE, threshold tuning, LR scheduling)
* CNN-BiLSTM Final (no expression input, AUC-based selection, max|expr| > 3.0 label rule)
* ElasticNet regression (L1+L2, per-condition, in-silico mutagenesis)
* XGBoost regression (baseline)
* XGBoost vs CatBoost (improved features + log-transformed targets)

Sequence Embeddings

* GENERator-v2-eukaryote-1.2b-base (frozen, 1.2 B parameters)
* Mean-pooled last-hidden-state vectors of length 2048
* Cached as X_gen.npy (67,860 × 2048) to avoid re-running the forward pass

Training Strategy

* Transfer learning from frozen genomic foundation model
* Stratified 70/15/15 train/val/test split (random_state = 42)
* Adam optimiser (lr = 1e-3, weight_decay = 1e-4)
* ReduceLROnPlateau scheduler, best-AUC checkpointing for the Final model
* Per-condition models trained for ElasticNet, XGBoost, CatBoost (six per family)
* Evaluation metrics: R², RMSE, Pearson r (regression); Accuracy, F1, AUROC (classification)

Datasets

* Supplementary Table 1: 79,842 plant promoter sequences (≤ 200 bp) with species, GC content, strand
* Supplementary Table 2: 78,677 log2-normalised activity rows across 6 conditions
* Inner-joined on gene → 67,860 promoters (final working dataset)
* Source: AlphaGenome pan-plant STARR-seq dataset (Avsec et al., Nature, 2026)

Tools

* Python 3.12
* PyTorch (CNN, BiLSTM, training loops)
* Hugging Face Transformers (GENERator embedding extraction)
* XGBoost, CatBoost, scikit-learn
* pandas / numpy / matplotlib / seaborn
* Google Colab Pro (T4 GPU) for deep learning; CPU for tree-based models

Results

CNN-BiLSTM Final (no leakage)

* True sequence-only classifier (no expression input)
* Per-species confusion matrices and ROC curves are consistent across Arabidopsis, Maize, and Sorghum
* Predictions saved for all 67,860 promoters: 30,537 functional, 37,323 non-functional

ElasticNet (regression)

* Avg test R²: 0.33 – 0.40
* Best condition R²: 0.45 (with enhancer + light, tobacco)
* Strength: interpretable, sparse, fast; in-silico mutagenesis identifies positional importance

XGBoost baseline (regression)

* Avg test R²: 0.534 | Avg test RMSE: 0.960
* Best condition R²: 0.584 (with enhancer + light, tobacco)
* Train–test gap: ~0.243 (moderate overfitting)

CatBoost (regression — best)

* Avg test R²: 0.58 – 0.60
* Best condition R²: ~0.62 (with enhancer + light, tobacco)
* Smallest train–test gap of the three tree-based variants

Key findings

* Every regression model posts its highest R² on the "with enhancer + light, tobacco" condition
* CatBoost wins quantitative regression; the Final no-leakage CNN+BiLSTM+GENERator wins functional classification
* Sequence-composition features hit a ceiling around R² ≈ 0.60 — the remainder requires motif spacing, chromatin context, or measurement-noise modelling
* Data leakage in v1 produced near-perfect (~99.997% AUC) but invalid metrics; the Final redesign demonstrates that sequence-only prediction is genuinely possible

Contributors

* Yeshaswee Sai Ganesh Volety — Auburn University
* Abhipsha Sahoo — Auburn University

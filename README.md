# Hybrid Network Intrusion Detection System

**Course:** CSC 5800 - Intelligent Systems: Algorithms and Tools  
**University:** Wayne State University  
**Semester:** Winter 2026  
**Student:** Fahad Qaseem Khawar  
**Instructor:** Dr. Suzan Arslanturk

---

## Overview

A two-stage hybrid NIDS pipeline built on the UNSW-NB15 dataset. Stage one uses Isolation Forest for unsupervised anomaly detection to flag suspicious traffic. Stage two applies supervised classifiers (Random Forest, XGBoost, LightGBM, SVM) to categorize flagged traffic into 9 attack types. K-Means clustering and Apriori association rule mining provide additional unsupervised analysis and interpretable forensic rules.

---

## Project Structure

```
NIDS_Project/
├── notebooks/
│   ├── 01_Setup.ipynb              # Environment setup, data download, merge
│   ├── 02_Preprocessing_EDA.ipynb  # Cleaning, EDA, feature engineering, SMOTE
│   ├── 03_Modeling.ipynb           # Anomaly detection, classification, clustering, Apriori
│   └── 04_Demo.ipynb               # End-to-end demo with sample predictions
├── report/
│   ├── CSC5800_NIDS_Report_Final.docx
│   └── CSC5800_Project_Proposal.pdf
├── data/
│   └── cluster_summary.csv         # K-Means cluster composition statistics
├── figures/                        # All generated plots (18 figures)
├── visualizations/
│   └── nids_kmeans_viz.html        # Interactive K-Means cluster plot (Plotly)
├── requirements.txt
└── README.md
```

---

## Dataset

**UNSW-NB15** -- University of New South Wales, Australia  
- 257,673 records used (from ~2.5M total), 49 original features, 30 selected via Mutual Information  
- 93,000 normal (36.1%) / 164,673 attack (63.9%)  
- Official train split: 82,332 records | test split: 175,341 records  
- Attack categories: Generic, Exploits, Fuzzers, DoS, Reconnaissance, Backdoor, Analysis, Shellcode, Worms  
- Download: https://research.unsw.edu.au/projects/unsw-nb15-dataset  
- Kaggle mirror: https://www.kaggle.com/datasets/dhoogla/unswnb15

The raw dataset is not included in this repo due to size. Notebook 01 handles downloading it automatically.

---

## Pipeline

| Stage | Method | Details |
|---|---|---|
| Preprocessing | Cleaning, Min-Max scaling, label encoding, SMOTE | SMOTE balances training set to 131,738 per class |
| EDA | Correlation heatmap, mutual information, distribution plots | Top features: sttl, dttl, ct_state_ttl |
| Anomaly Detection | Isolation Forest | n_estimators=200, contamination=0.15 |
| Binary Classification | Random Forest, XGBoost, LightGBM, SVM | Evaluated on 51,535 test records |
| Multi-class Classification | XGBoost (softprob) | 9 attack categories |
| Clustering | K-Means + PCA | K=4, Silhouette=0.4808 |
| Rule Mining | Apriori | 115 attack-consequent rules extracted |

---

## Results

### Binary Classification (51,535 test records)

| Model | Accuracy | Precision | Recall | F1 | ROC-AUC |
|---|---|---|---|---|---|
| Isolation Forest | 0.6223 | 0.8471 | 0.4990 | 0.6281 | 0.7926 |
| Random Forest | 0.9797 | 0.9912 | 0.9768 | 0.9840 | 0.9981 |
| **XGBoost (Best)** | **0.9830** | **0.9948** | **0.9785** | **0.9866** | **0.9989** |
| LightGBM | 0.9803 | 0.9945 | 0.9747 | 0.9845 | 0.9984 |
| SVM | 0.9157 | 0.9582 | 0.9078 | 0.9323 | 0.9768 |

Cross-validation (5-fold, 50k stratified subsample): XGBoost CV F1 = 0.9803 ± 0.0013, RF CV F1 = 0.9716 ± 0.0015

### Multi-class Attack Classification (XGBoost)

| Attack Category | F1 |
|---|---|
| Generic | 0.98 |
| Fuzzers | 0.90 |
| Reconnaissance | 0.82 |
| Exploits | 0.69 |
| Shellcode | 0.63 |
| DoS | 0.45 |
| Analysis / Worms | 0.00 (insufficient training samples) |

### K-Means Cluster Composition (K=4, Silhouette=0.4808)

| Cluster | Total | Attack Rate |
|---|---|---|
| 0 | 23,505 | 92.2% attack |
| 1 | 5,606 | 77.4% attack |
| 2 | 10,294 | 66.5% attack |
| 3 | 12,130 | 0.7% attack (near-pure normal) |

### Top Association Rules (by Lift)

| Rule | Support | Confidence | Lift |
|---|---|---|---|
| {state_enc_L, sttl_L} => {dttl_H, ATK_Exploits} | 0.116 | 0.888 | 5.151 |
| {dttl_H, sttl_L} => {state_enc_L, ATK_Exploits} | 0.116 | 0.888 | 5.040 |
| {sttl_L, proto_enc_H} => {dttl_H, ATK_Exploits} | 0.116 | 0.868 | 5.035 |
| {ct_state_ttl_L, sttl_L} => {dttl_H, ATK_Exploits} | 0.114 | 0.865 | 5.017 |
| {sttl_L} => {dttl_H, ATK_Exploits} | 0.116 | 0.863 | 5.005 |

---

## Key Findings

- TTL features (sttl, dttl, ct_state_ttl) are the most discriminative across all methods -- mutual information ranking, both tree model importances, and the highest-lift association rules.
- K-Means with no label access produced a 99.3% pure normal cluster and a 92.2% attack cluster, validating the selected feature set's natural separability.
- XGBoost is the recommended model: best accuracy, lowest false positive rate (0.009), GPU-accelerated, stable CV performance.
- LightGBM is the preferred alternative in CPU-only environments -- nearly identical accuracy with faster training.
- Isolation Forest alone achieves only F1=0.628 but flags a subset containing 84.8% of all true attacks, making it effective as a coarse first-stage filter.

---

## Setup

```bash
pip install -r requirements.txt
```

Run notebooks in order: `01 -> 02 -> 03 -> 04`

All notebooks are designed for Google Colab. Intermediate outputs are saved to Google Drive under `MyDrive/CSC5800_NIDS_Project/`.

---

## References

- Moustafa & Slay (2015). UNSW-NB15: A comprehensive data set for network intrusion detection systems. MilCIS.
- Liu et al. (2008). Isolation Forest. IEEE ICDM.
- Chen & Guestrin (2016). XGBoost: A scalable tree boosting system. KDD.
- Ke et al. (2017). LightGBM. NeurIPS.
- Chawla et al. (2002). SMOTE: Synthetic Minority Over-sampling Technique. JAIR.

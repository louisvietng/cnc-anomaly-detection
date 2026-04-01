# CNC Anomaly Detection

Fault detection on CNC machining data using multi-domain feature extraction and classical machine learning.

---

## Dataset

**Tnani, M. A., Feil, M., & Diepold, K.** (2022). *A new benchmark dataset for machine learning in machining*. Procedia CIRP.  
DOI: [10.1016/j.procir.2022.04.022](https://doi.org/10.1016/j.procir.2022.04.022) · License: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)  
*Data loader adapted from the authors' reference implementation — [github.com/boschresearch/CNC_Machining](https://github.com/boschresearch/CNC_Machining)*

| Property | Detail |
|---|---|
| Machines | M01, M02, M03 |
| Processes | OP00 – OP14 (15 total; OP14 excluded for M03) |
| File format | HDF5 (`.h5`), one file per machine–process–run |
| Signals | X / Y / Z-axis acceleration |
| Sampling rate | 2 kHz |
| H5 structure | `(n_samples, 3)` — each row is one timestep: `[acc_x, acc_y, acc_z]` |
| Labels | `good` / `bad` |
| Class balance | ~4% bad (70 bad, 1,632 good) |

**Notes on dataset structure:**

- Multiple `.h5` files per process represent repeated runs of the **same operation** recorded across different time periods (2019–2021)

- Process labels and one process slot (M03 OP14) are partially withheld for confidentiality (Biegel et al., 2024) — labels are anonymized and shuffled per machine, so `machine_process` (e.g. `M01_OP00`) must be used as a combined identifier rather than `process` alone

---

## Project Structure

```
cnc-anomaly-detection/
├── data/
│   └── raw/                         # HDF5 files — not tracked in git
├── notebooks/
│   ├── 00_EDA.ipynb                 # Exploratory data analysis
│   ├── 01_preprocessing.ipynb       # Signal preprocessing pipeline
│   ├── 02_feature_extraction.ipynb  # Multi-domain feature extraction
│   └── 03_modeling.ipynb            # Model training and evaluation
├── src/
│   └── utils/
│       └── data_loader_utils.py     # Adapted from Bosch reference implementation
├── results/                         # Saved metrics and figures
└── README.md
```

---

## Planned Approach

### Feature Engineering

Four complementary feature domains will be extracted per axis (X, Y, Z):

| Domain | Full Name | Features |
|---|---|---|
| Time statistics | — | Duration, RMS, Kurtosis, Std |
| FFT | Fast Fourier Transform | Magnitude Spectrum |
| STFT | Short-Time Fourier Transform | Time-Resolved Frequency Content |
| WPT | Wavelet Packet Transform | Sub-Band Energy |

### Class Imbalance

- ~4% bad samples — severe imbalance
- Handle with class weights + SMOTE applying to training data only

### Evaluation

Two splits for every model:

| Split | Description |
|---|---|
| Random 80/20 stratified | Standard benchmark |
| Machine-out (M01+M02 → M03) | Cross-machine generalisation |

Primary metric: **F1 score** — Accuracy is misleading at 4% class imbalance.

### Models

Algorithms evaluated sequentially — each informing the next:

| # | Algorithm | Rationale |
|---|---|---|
| 1 | Logistic Regression | Linear baseline |
| 2 | Lasso (LassoCV) | Baseline with automatic feature selection |
| 3 | Decision Tree | Interpretable non-linear baseline |
| 4 | Random Forest | Ensemble; feature importance for domain ranking |
| 5 | XGBoost | Gradient boosting; handles imbalance well |
| 6 | LightGBM | Faster alternative to XGBoost at scale |
| 7 | Isolation Forest | Unsupervised anomaly detection baseline |
| 8 | One-Class SVM | Unsupervised; trained on good samples only |
| 9 | LSTM | Temporal patterns in raw signal sequence |

---

## Setup

Download the dataset from [github.com/boschresearch/CNC_Machining](https://github.com/boschresearch/CNC_Machining) and place it under `data/raw/`.

*Environment file will be added once dependencies are finalized.*

---

## Key Findings

*To be populated after EDA is complete.*
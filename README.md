# Microbial Community Shifts Enable Early Detection of Refrigerated Chicken Breast Spoilage via AI-Based Analysis

This repository contains the full analysis pipeline for the paper:

> **"Microbial community shifts enable early detection of refrigerated chicken breast spoilage via AI based analysis"**  
> Eun Seo Lee, Tae Ho Kim, Hansol Doh, Young Suk Kim, Woo Ju Kim, and Sun Ae Kim  
> *Nature Communications* (under review, 2026)  
> DOI: TBD upon acceptance

We characterise microbial succession in refrigerated chicken breast (0–18 days storage, n = 270 samples) using 16S rRNA genus-level relative abundance data. CLR-transformed compositional data are passed through automated spoilage-stage labelling (Optuna or K-means), two feature-selection strategies (Fold-change Top-10 or PLS-DA VIP ≥ 1.0), machine-learning classification of spoilage stage, TPC regression for early spoilage detection, and SHAP explainability analysis. The best-performing pipeline (K-means labelling + VIP ≥ 1.0 features + SVR) achieves MAE = 0.325 and R² = 0.850 on held-out test data.

---

## Repository Structure

```
folder/
├── data/
│   input: microbial abundance + TPC measurements
│   ├── K_means_pkl/             # Preprocessed/intermediate data for K-means labelling branch
│   │   ├── all_preprocessed_data.pkl
│   │   ├── all_top10_data.pkl
│   │   ├── anova_results.pkl
│   │   ├── change_analysis_results.pkl
│   │   └── genus_final_data.pkl
│   └── optuna_pkl/              # Preprocessed/intermediate data for Optuna labelling branch
│       ├── all_preprocessed_data.pkl
│       ├── all_top10_data.pkl
│       ├── anova_results.pkl
│       └── genus_final_data.pkl
│
├── preprocessing.ipynb          # Step 1 – CLR transformation, imputation, ANOVA/BH filtering
│
├── optuna_labeling/             # Step 2–6, Optuna labelling branch
│   ├── optuna_Labeling.ipynb    # Step 2 – Optuna threshold optimisation (500 trials)
│   ├── rate_of_change/          # Feature selection: Fold-change Top-10
│   │   ├── Optuna_ROC_Saniation_TPC_analysis.ipynb        # EDA / TPC trend analysis
│   │   ├── Optuna_ROC_TPC_classification_model_train_test.ipynb  # Step 4 classification
│   │   └── Optuna_ROC_TPC_Regression_model_train_test.ipynb     # Step 5 regression
│   └── vip_top10/               # Feature selection: PLS-DA VIP ≥ 1.0
│       ├── optuna_VIP_TPC_analysis_classification.ipynb   # Step 4 classification
│       └── optuna_VIP_TPC_analysis_regressor_model.ipynb  # Step 5 regression + SHAP
│
└── k_means_labeling/            # Step 2–6, K-means labelling branch
    ├── K_means_labeling.ipynb   # Step 2 – K-means (k=3) TPC clustering
    ├── rate_of_change/          # Feature selection: Fold-change Top-10
    │   ├── K_means_ROC_Saniation_TPC_analysis.ipynb
    │   ├── K_means_ROC_TPC_classification_model_train_test.ipynb
    │   └── K_means_ROC_TPC_Regression_model_train_test.ipynb
    └── vip_top10/               # Feature selection: PLS-DA VIP ≥ 1.0  ← best condition
        ├── K_means_VIP_TPC_analysis_classification.ipynb
        └── K_means_VIP_TPC_analysis_regressor.ipynb
```

The four experimental conditions reported in the paper map to:

| Condition | Labelling | Feature selection | Notebook folder |
|-----------|-----------|-------------------|-----------------|
| 1 | Optuna | Fold-change Top-10 | `optuna_labeling/rate_of_change/` |
| 2 | Optuna | VIP ≥ 1.0 | `optuna_labeling/vip_top10/` |
| 3 | K-means | Fold-change Top-10 | `k_means_labeling/rate_of_change/` |
| **4 (best)** | **K-means** | **VIP ≥ 1.0** | **`k_means_labeling/vip_top10/`** |

---

## Requirements

| Software | Version |
|----------|---------|
| Python | 3.12.2 |
| scikit-learn | 1.8.0 |
| optuna | 4.3.0 |
| shap | 0.48.0 |
| lightgbm | 4.6.0 |
| xgboost | 3.0.2 |
| tensorflow | 2.16.2 |
| pandas | 2.3.3 |
| numpy | 1.26.4 |
| scipy | 1.16.3 |
| matplotlib | 3.10.8 |
| seaborn | 0.13.2 |
| plotly | 5.24.1 |
| statsmodels | 0.14.2 |
| openpyxl | 3.1.5 |

---

## Installation

### Option A — conda (recommended)

```bash
conda env create -f environment.yml
conda activate chicken_spoilage_ml
```

### Option B — pip

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

Then launch Jupyter:

```bash
jupyter lab
# or
jupyter notebook
```

---

## Input Data

**File:** `data/Sanitation.xlsx`

The workbook contains multiple sheets (one per taxonomic level):

| Sheet | Taxonomic level | Column range |
|-------|----------------|--------------|
| 0 | Microbial metadata (TPC, TBARS) | B:D |
| 1 | Phylum | B:AI |
| 2 | Class | B:BM |
| 3 | Order | B:ET |
| 4 | Family | B:JV |
| 5 | Genus | B:... |

- Each row is one sample (`Description` column, e.g. `A7-4` = day 7, replicate 4).
- Abundance values are **relative abundances (%)** that sum to 100 per sample.
- `TPC (log CFU/g)` column contains total plate count measurements used for labelling and regression.
- Four samples that failed sequencing (`A5-22`, `A7-4`, `A7-16`, `A7-28`) are imputed using same-day averages in `preprocessing.ipynb`.

---

## How to Reproduce Results

Run the notebooks **in order**. Each notebook saves intermediate data as `.pkl` files under `data/optuna_pkl/` or `data/K_means_pkl/` so that subsequent notebooks can load them without re-running earlier steps.

### Step 1 — Preprocessing (`preprocessing.ipynb`)

1. Open `preprocessing.ipynb`.
2. Update `data_dir` in cell 1 to point to your local `data/` folder:
   ```python
   data_dir = "data"          # relative path from repo root
   sanitation_path = os.path.join(data_dir, 'Sanitation.xlsx')
   ```
3. Run all cells.

**What it does:**
- Imputes sequencing-failed samples using same-day mean values; normalises rows to sum = 100.
- Replaces zeros with pseudo-count `1e-6` before log transformation.
- Applies CLR (Centered Log-Ratio) transformation to move data to Euclidean space.
- Caps outliers per storage day using IQR (×1.5 fence).
- Runs one-way ANOVA + Benjamini-Hochberg FDR correction (q < 0.05) to retain only significantly changing genera.
- Saves preprocessed data to `data/optuna_pkl/all_preprocessed_data.pkl` and `data/K_means_pkl/all_preprocessed_data.pkl`.

---

### Step 2 — Spoilage-Stage Labelling

Choose the labelling method you want to evaluate.

#### 2A — Optuna labelling (`optuna_labeling/optuna_Labeling.ipynb`)

- Runs 500 Optuna trials to search for TPC thresholds (θ₁, θ₂) that maximise a custom scoring function rewarding clean fresh → middle → spoilage progression.
- Saves labelled data to `data/optuna_pkl/`.

#### 2B — K-means labelling (`k_means_labeling/K_means_labeling.ipynb`)

- Clusters TPC values into k = 3 groups; sets stage boundaries at midpoints between cluster centroids.
- Saves labelled data to `data/K_means_pkl/`.

---

### Step 3 + 4 — Feature Selection and Classification

For each labelling/feature-selection combination open the corresponding classification notebook:

| Condition | Notebook |
|-----------|----------|
| Optuna + Fold-change Top-10 | `optuna_labeling/rate_of_change/Optuna_ROC_TPC_classification_model_train_test.ipynb` |
| Optuna + VIP ≥ 1.0 | `optuna_labeling/vip_top10/optuna_VIP_TPC_analysis_classification.ipynb` |
| K-means + Fold-change Top-10 | `k_means_labeling/rate_of_change/K_means_ROC_TPC_classification_model_train_test.ipynb` |
| K-means + VIP ≥ 1.0 **(best)** | `k_means_labeling/vip_top10/K_means_VIP_TPC_analysis_classification.ipynb` |

Each notebook:
- Applies the relevant feature selection strategy internally (fold-change ranking or PLS-DA VIP).
- Trains five classifiers: Logistic Regression, SVC, k-NN, Random Forest, XGBoost.
- Uses 80/20 train–test split with 5-fold cross-validation and GridSearchCV tuning.
- Reports Accuracy, Macro-F1, and ROC-AUC per model and condition.

---

### Step 5 — TPC Regression

For each labelling/feature-selection combination open the corresponding regression notebook:

| Condition | Notebook |
|-----------|----------|
| Optuna + Fold-change Top-10 | `optuna_labeling/rate_of_change/Optuna_ROC_TPC_Regression_model_train_test.ipynb` |
| Optuna + VIP ≥ 1.0 | `optuna_labeling/vip_top10/optuna_VIP_TPC_analysis_regressor_model.ipynb` |
| K-means + Fold-change Top-10 | `k_means_labeling/rate_of_change/K_means_ROC_TPC_Regression_model_train_test.ipynb` |
| K-means + VIP ≥ 1.0 **(best)** | `k_means_labeling/vip_top10/K_means_VIP_TPC_analysis_regressor.ipynb` |

Each notebook:
- Filters samples to **fresh + middle** stages only (early-spoilage detection scenario).
- Trains four regressors: SVR, Random Forest, XGBoost, LightGBM.
- Reports MAE, RMSE, R² on the held-out test set.
- The best result (Condition 4, SVR) achieves **MAE = 0.325, R² = 0.850**.

---

### Step 6 — SHAP Explainability

SHAP analysis is embedded in the regression notebooks for Conditions 2 and 4 (VIP ≥ 1.0 feature sets). The classification notebooks for Condition 4 additionally include SHAP for the Logistic Regression model.

- Key features identified: **Iodobacter**, **Vagococcus** (classification); **Iodobacter**, **Streptococcus** (regression).
- Bar plots are generated inline within each notebook.

---

### TPC Trend Analysis (optional EDA)

Exploratory data analysis notebooks are also provided:

- `optuna_labeling/rate_of_change/Optuna_ROC_Saniation_TPC_analysis.ipynb`
- `k_means_labeling/rate_of_change/K_means_ROC_Saniation_TPC_analysis.ipynb`

These visualise TPC trends over storage days and the resulting stage boundaries under each labelling method.

---

## Expected Outputs

All figures are generated inline in the notebooks. Save them via the notebook interface or add `plt.savefig(...)` / `fig.write_image(...)` calls as needed.

| Output | Location |
|--------|----------|
| CLR-transformed data tables | Displayed in `preprocessing.ipynb` |
| ANOVA / BH-FDR results | Displayed in `preprocessing.ipynb` |
| Labelling boundary plots | Labelling notebooks |
| PLS-DA score plots | Classification notebooks |
| Model performance tables (Accuracy / F1 / AUC) | Classification notebooks |
| Model performance tables (MAE / RMSE / R²) | Regression notebooks |
| SHAP bar plots | VIP regression/classification notebooks |
| Microbial abundance trend plots | TPC analysis notebooks |

---

## Citation

If you use this code or data in your own work, please cite:

```bibtex
@article{lee2026chicken,
  title   = {Microbial community shifts enable early detection of refrigerated
             chicken breast spoilage via AI based analysis},
  author  = {Lee, Eun Seo and Kim, Tae Ho and Doh, Hansol and 
             Kim, Young Suk and Kim, Woo Ju and Kim, Sun Ae},
  journal = {Nature Communications},
  year    = {2026},
  doi     = {TBD}
}
```

---

## Installation
**Typical install time:** approximately 5–10 minutes on a standard 
Desktop computer.


**Expected run time (full pipeline):**

| Notebook | Estimated time |
|----------|---------------|
| preprocessing.ipynb | ~2–3 min |
| Optuna labelling | ~10 min (500 trials) |
| K-means labelling | ~1 min |
| Each classification notebook | ~3–5 min |
| Each regression notebook | ~3–5 min |
| **Total (all 4 conditions)** | **~30–40 min** |

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.

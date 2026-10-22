# What drives a country's CO₂ emissions? Explainable ML on World Bank ESG data

**Module:** 6FTC2177 Next Generation Data Science, Assignment 1: *Explainable Machine Learning Modelling on a Real-World Dataset*
**Theme:** Societal challenges, climate change
**Author:** *[Keith Phang, 25034227]*

---

## 1. Purpose

CO₂ emissions per person differ more than 100-fold between countries. This project builds an end-to-end, leakage-safe machine-learning pipeline that **predicts a country's CO₂ emissions per person** from World Bank environmental, social and governance indicators. It then uses **Explainable AI (XAI)** to show *which* national characteristics the model links to high emissions and *why* it makes individual predictions.

- **Task:** regression on `log1p(CO₂ tonnes per person)`
- **Unit:** one country in one year (191 countries, 2000–2020, 4,011 rows)
- **Validation:** grouped by country, so the model is always tested on countries it has never seen. All learned preprocessing (sparse-column removal, log-transform choice, imputation, scaling) is refitted inside every fold
- **Scope:** retrospective estimation for observed years (benchmarking and gap-filling), **not** forecasting future emissions

## 2. Key results

| | Result |
|---|---|
| Final model | LightGBM, tuned with randomised search (grouped 5-fold CV), chosen by a pre-set CV rule |
| Test performance (39 unseen countries) | **R² = 0.93** (log scale), typical error factor **×1.24** (≈ +24% / −19%), correct emissions band **88%** (95% bootstrap interval 82–93%) |
| Baseline (predict the median) | R² ≈ 0, error factor ×2.3, correct band 35% |
| Robustness | Over 25 repeated grouped splits, tuning beat the default model in 19/25. On the single test set the default model scored better, which is reported, not used to change the choice. The gap-filling method made almost no difference (±0.002 RMSE) |
| AutoML (FLAML) | Gradient-boosted trees lead its leaderboard; re-scored like for like, its best configuration ties with the manual model (better in about half of 25 splits) |
| PCA | PC1 (30% of variance) summarises development and correlates 0.84 with emissions; PCA was **not** used in the final model because it made LightGBM worse at every *k* |
| Most influential predictors (SHAP, grouped SHAP, permutation, ALE, LIME agree) | **Energy use per person**, then energy access and income, and the **energy mix** (renewable vs fossil share). These are predictive associations, not causal effects |

## 3. Data

- **Source:** World Bank Data360, *Environment, Social & Governance (ESG)* database (`WB_ESG`): https://data360.worldbank.org/en/dataset/WB_ESG
- **Licence:** CC BY 4.0
- **Size:** raw file about 185 MB in long format (461,719 rows, 71 indicators); processed modelling table 4,011 × 76

See [`data/README.md`](data/README.md) for full provenance and the processing steps. The raw file is not stored in the repo; the notebook downloads it automatically.

## 4. Repository structure

```
ngds-esg-co2-xai/
├── NGDS_Assignment1_ESG_CO2.ipynb   # main notebook: all code, outputs and commentary
├── requirements.txt                 # Python libraries needed
├── README.md                        # this file
├── .gitignore                       # excludes the 185 MB raw file and temporary files
├── data/
│   ├── README.md                    # data source, licence, processing steps
│   ├── esg_country_year_2000_2020.csv   # processed modelling table
│   └── esg_data_dictionary.csv          # meaning and unit of every column
└── figures/                         # every chart from the notebook (PNG), used in the slides
```

## 5. Setup and how to run

### Option A: Google Colab (recommended)

1. Open the notebook in Colab: **File → Open notebook → GitHub**, then paste this repository's URL.
2. **Runtime → Run all.** The first cell installs SHAP, LIME, FLAML and LightGBM automatically.
3. The first run downloads the raw World Bank file (~185 MB). Total runtime is about **15–20 minutes** on Colab's free tier (about 12 minutes on a faster machine), of which AutoML is a fixed 2 minutes and the repeated cross-validation a few minutes.

### Option B: Local Jupyter

```bash
git clone <this-repo-url>
cd ngds-esg-co2-xai
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter notebook NGDS_Assignment1_ESG_CO2.ipynb
```

If `data/esg_country_year_2000_2020.csv` is present, the notebook loads it directly. Otherwise it rebuilds it from the raw download.

### Reproducibility

- One random seed (`SEED = 42`) controls all splits, models and searches.
- All preprocessing is inside scikit-learn `Pipeline` / `ColumnTransformer` objects, fitted on training data only.
- **Exception:** FLAML AutoML searches for a fixed time (120 s), so its leaderboard can differ slightly between machines. All other results are deterministic.

## 6. Notebook outline

| Section | Content |
|---|---|
| 1–2 | Setup; load and reshape the Data360 file (long → wide, remove aggregates) |
| 3 | Problem definition, metrics (RMSE on log scale, R², MAE in tonnes) and success criteria |
| 4 | Data quality: missing values, duplicates, **leakage audit** |
| 5 | Within-country gap-filling (plus a past-only variant), grouped train/test split (80/20 by country), fold-safe `NumericPrep` transformer |
| 6 | EDA: target distribution, univariate, bivariate, trends over time, multicollinearity |
| 7 | PCA: scree plot, loadings, projection, and an experiment on whether to use PCA in the model |
| 8 | Preprocessing pipelines (linear vs tree) with `ColumnTransformer`; evidence that decisions are re-learned per fold |
| 9 | Baseline, Ridge, Random Forest and LightGBM compared with grouped CV; gap-filling sensitivity check |
| 10 | Hyperparameter tuning (`RandomizedSearchCV`, grouped CV); repeated grouped CV (25 splits) to test whether tuning reliably helps |
| 11 | AutoML with FLAML on raw features (grouped CV), leaderboard, like-for-like re-scoring, limitations |
| 12 | Pre-set selection rule, test-set evaluation (log and tonne metrics, band accuracy), country-level bootstrap, error analysis |
| 13 | XAI: SHAP (beeswarm, grouped, dependence, waterfall), PDP/ICE, ALE, permutation importance, LIME with stability check, cross-check, limitations |
| 14 | Conclusions, limitations and next steps |

## 7. Limitations

- **Associations, not causes:** XAI explains the model, not the real world; agreement between XAI methods is not proof of a mechanism.
- **Estimation, not forecasting:** results apply to observed years, not future emissions.
- **Few independent units:** 4,011 rows come from only 191 countries, so test scores depend on which 39 countries were drawn.
- **Energy use is close to a mechanical driver of CO₂**, which makes the prediction task easier.
- About 16% of input values were imputed after within-country gap-filling.

## 8. References

- World Bank (2026). *Environment, Social & Governance (ESG) database*, Data360. CC BY 4.0.
- Lundberg, S. & Lee, S.-I. (2017). A unified approach to interpreting model predictions (SHAP). *NeurIPS*.
- Ribeiro, M. T., Singh, S. & Guestrin, C. (2016). "Why should I trust you?" Explaining the predictions of any classifier (LIME). *KDD*.
- Apley, D. & Zhu, J. (2020). Visualizing the effects of predictor variables in black box supervised learning models (ALE). *JRSS B*.
- Wang, C. et al. (2021). FLAML: A fast and lightweight AutoML library. *MLSys*.
- Bergstra, J. & Bengio, Y. (2012). Random search for hyper-parameter optimization. *JMLR*.
- Molnar, C. *Interpretable Machine Learning*. https://christophm.github.io/interpretable-ml-book/

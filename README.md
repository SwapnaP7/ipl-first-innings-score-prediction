# IPL First-Innings Score Prediction

## Overview

This project uses machine learning regression to predict the final score of an IPL first innings from the current match situation: the batting and bowling teams, runs and wickets so far, balls bowled, and runs scored and wickets lost in the last 5 overs. It predicts the first-innings total only, not the match winner.

The dataset has one row per ball, so many rows belong to the same match. To avoid leakage between training and evaluation, the data is split by match, and models are compared with cross-validation that also keeps each match's rows together. The whole workflow is in one Jupyter notebook.

## Objectives

- Predict the final first-innings score from the current match state
- Compare several regression models
- Prevent match-level data leakage
- Evaluate the selected model on unseen matches

## Dataset

The notebook expects `data/ipl_dataset.csv`, ball-by-ball records of IPL first innings. The version used for the reported results has 76,014 rows and 15 columns covering 617 matches played between 2008-04-18 and 2017-05-21, with 0 missing values and 0 duplicate rows. Columns: `mid`, `date`, `venue`, `batting_team`, `bowling_team`, `batsman`, `bowler`, `runs`, `wickets`, `overs`, `runs_last_5`, `wickets_last_5`, `striker`, `non-striker`, `total`.

- **Input features:** batting team, bowling team, runs, wickets, balls elapsed (converted from `overs`), runs in the last 5 overs, wickets in the last 5 overs
- **Target:** `total` (final first-innings score)
- **Filtering:** only eight teams from the historical dataset are retained to maintain a consistent team set for modeling (53,811 rows), and only situations after at least 5 completed overs (40,545 rows from 437 matches remain)

**Availability, source and licence:** the dataset is **not included** in this repository because its original source and licence could not be verified, so redistribution rights are unknown. See [`data/README.md`](data/README.md) for the expected columns and where to place the file.

## Methodology

```
Data cleaning
  -> Team filtering
  -> First-five-over filtering
  -> Cricket-over conversion
  -> Match-level train/test split
  -> Group-aware cross-validation
  -> Model selection
  -> Final training
  -> Held-out test evaluation
```

1. **Cricket-over conversion:** `overs` is in cricket notation (completed overs plus the legal balls bowled in the current over, so `5.6` is followed by `6.1`), which is not an ordinary decimal number. It is converted to `balls_elapsed = 6 x completed overs + balls`, for example `5.1 -> 31` and `5.6 -> 36`. The first-five-over filter keeps `balls_elapsed >= 30`.
2. **Match-level split:** `GroupShuffleSplit` on the match ID, 80% / 20% of matches with `random_state=42`: 349 training matches (32,376 rows) and 88 test matches (8,169 rows). The notebook checks that no match is in both sets.
3. **Preprocessing:** team names are one-hot encoded and numerical features are standardised. Both steps are part of each model's scikit-learn `Pipeline`, so they are fitted only on the data that model is trained on.
4. **Model selection:** 5-fold `GroupKFold` cross-validation on the training matches only. The regularisation strength of Lasso Regression is chosen the same way. The model with the highest mean cross-validation R² is selected.
5. **Final evaluation:** the selected model is fitted on all training matches and evaluated once on the held-out test matches. The test set is not used to choose the model or any hyperparameter.

## Models

- Decision Tree Regressor
- Linear Regression
- Random Forest Regressor
- Lasso Regression
- Support Vector Regression
- Neural Network (MLP Regressor)

Only the Lasso `alpha` is tuned; the other models use their default settings (the MLP uses a logistic activation and a higher iteration limit so that it converges).

## Evaluation

**Cross-validation on the training matches** (5 group folds, mean ± standard deviation):

| Model | R² | MAE | RMSE |
|---|---:|---:|---:|
| Lasso Regression | 0.6392 ± 0.0330 | 13.45 ± 0.66 | 17.83 ± 0.92 |
| Linear Regression | 0.6358 ± 0.0343 | 13.54 ± 0.61 | 17.90 ± 0.86 |
| Support Vector Regression | 0.6259 ± 0.0428 | 13.73 ± 0.81 | 18.14 ± 1.10 |
| Random Forest | 0.5178 ± 0.0596 | 15.11 ± 0.62 | 20.56 ± 0.89 |
| Neural Network | 0.3713 ± 0.0928 | 17.79 ± 0.61 | 23.44 ± 0.72 |
| Decision Tree | 0.2276 ± 0.1017 | 19.34 ± 0.37 | 25.99 ± 0.48 |

**Selected model: Lasso Regression** (regularisation strength `alpha` = 0.1, also chosen by group-aware cross-validation). It has the highest mean cross-validation R² (0.6392).

**Held-out test result** (single evaluation on 88 unseen matches):

| R² | MAE | MSE | RMSE |
|---:|---:|---:|---:|
| 0.5962 | 13.43 | 337.93 | 18.38 |

The notebook also shows six illustrative predictions for hand-entered match situations; they are not a substitute for the test-set evaluation above. The trained pipeline (preprocessing and model) is saved to `models/lasso_regression_model.joblib`.

## Limitations

- The data covers only 2008-2017; teams, rules and playing styles have changed since.
- There is no live match information, and no player form, injury, venue, pitch or weather features.
- Results come from one random match-level train/test split, and the test R² (0.5962) is lower than the cross-validation R² (0.6392). The cross-validation gap between the best models is small compared with the fold-to-fold variation, so the ranking among the top models is not clear-cut.
- Only the Lasso `alpha` was tuned, and its cross-validation score is slightly optimistic because `alpha` was chosen on the same folds.
- Predictions are estimates for historical match situations after 5 overs and are not guaranteed scores.

## Technologies

- Python
- Pandas
- NumPy
- Scikit-learn
- Matplotlib
- Seaborn
- Joblib
- Jupyter Notebook

## Repository Structure

```
ipl-first-innings-score-prediction/
├── README.md
├── requirements.txt
├── .gitignore
├── data/
│   └── README.md        (dataset instructions; ipl_dataset.csv is not included)
├── models/
│   └── lasso_regression_model.joblib
└── notebooks/
    └── ipl_first_innings_score_prediction.ipynb
```

## How to Run

Requires Python 3.11 or newer (needed by the pinned scikit-learn version).

```
git clone <repository-url>
cd ipl-first-innings-score-prediction
python -m venv .venv
```

Activate the environment (`.venv\Scripts\activate` on Windows, `source .venv/bin/activate` on macOS/Linux), then:

```
pip install -r requirements.txt
jupyter notebook
```

Place the dataset at `data/ipl_dataset.csv` (see `data/README.md`), then open `notebooks/ipl_first_innings_score_prediction.ipynb` and run all cells from top to bottom. Jupyter can be started from the repository root (as above) or from the `notebooks` folder. The notebook reads `data/ipl_dataset.csv` and saves the final model to `models/`. On a single CPU core the full run took about 25 minutes, most of it the neural-network cross-validation.

# Is Claim Denial a Property of the Insurer or the Plan?

ANLY 530 (Principles and Applications of Machine Learning), KA1 conference-paper project. Author: Ankit Bisht.

## Research question

How much of the variation in claim denial rates across ACA Marketplace plans is attributable to the insurer rather than to the plan's design, and can publicly reported plan attributes predict a plan's denial rate for insurers not seen during training?

The primary result is the variance decomposition. The prediction experiments test what it implies: a model should predict well only when it already knows the insurer.

## Repository contents

| File | Purpose |
|---|---|
| `claim_denial_exploratory.ipynb` | Milestone 1. Data audit, target description, insurer-versus-plan variance decomposition, and prediction under random versus insurer-grouped cross-validation |
| `milestone2_analysis.ipynb` | Milestone 2. Selection, insurer-concentration and claim-volume checks on the insurer share of variance; repeated insurer-grouped cross-validation with paired model-baseline comparisons; separate tests for insurers new to the Marketplace. Reproduces every number, table and figure in the Milestone 2 report |
| `paper_analysis.ipynb` | Reproduces every table and figure in the draft paper: fitness-for-use audit, variance-components models, plan-attribute effects, grouped cross-validation and temporal validation (claims years 2022 to 2024) |
| `figures/` | Figures written by `paper_analysis.ipynb` (`fig*.png`) and `milestone2_analysis.ipynb` (`m2_fig*.png`) |
| `requirements.txt` | Python packages and versions |

## Data

CMS Transparency in Coverage Public Use Files, individual-market medical plans (sheet "Ind QHP"):

- Plan year 2026 file: claims for calendar year 2024 (main analysis)
- Plan year 2025 file: claims for calendar year 2023 (prior-year insurer denial rate, year-ahead tests)
- Plan year 2024 file: claims for calendar year 2022 (year-ahead tests and temporal calibration)

Source: https://www.cms.gov/marketplace/resources/data/public-use-files

One row is one plan. The target is the in-network claim denial rate (claims denied / claims received). The data are public, aggregated and contain no patient information. The notebooks download the files from CMS on first run and cache them in `data/`, which is excluded from the repository.

Two things to know about the files:

- The plan year 2024 file has no `Exchange_Type` column and names enrollment `Enrollment_Data`. Exchange type is fixed within a state, so `milestone2_analysis.ipynb` fills it from the state.
- The enrollment field is not on the same scale in every file (for the same plans, the 2024 claims-year values are a median 12% of the 2023 values), so enrollment is used only within a year.

## How to run

```
pip install -r requirements.txt
jupyter notebook claim_denial_exploratory.ipynb
jupyter notebook milestone2_analysis.ipynb
jupyter notebook paper_analysis.ipynb
```

Run all cells from top to bottom. The first run needs an internet connection to download the CMS files (about 3 MB). `milestone2_analysis.ipynb` takes about 10 minutes on a 16-core machine because it fits every model on 150 insurer-grouped folds; it uses all CPU cores and needs scikit-learn 1.6 or later. `paper_analysis.ipynb` takes a few minutes because of the cluster bootstrap. Seed: 42.

## Evaluation design

- Metric: mean absolute error of the denial rate (0.01 = one percentage point).
- Baseline: the training-fold mean, scored on the same folds as every model.
- Folds grouped by insurer (`GroupKFold`), so no insurer appears in both training and test data. Random 5-fold is reported only to show how much it overstates performance.
- Milestone 2: the grouped 5-fold split is repeated 10 times (50 test folds). Each model is compared with the baseline fold by fold, with a 95% interval from the corrected resampled t-test (Nadeau & Bengio, 2003). The comparison is repeated with plans weighted by claims received and with the largest insurer removed.
- Every model (ridge, random forest, gradient boosting) is tuned inside each training fold, with inner folds also grouped by insurer.
- Insurers new to the Marketplace are tested separately: train on one year, predict the next, for insurers that CMS flagged as new the year before.
- Denial reasons, resubmissions, appeals and same-year insurer totals are excluded because they are recorded alongside or after the denials.

## Status

Milestone 1 (Week 10): research protocol and exploratory notebook.
Milestone 2 (Week 12): progress report and `milestone2_analysis.ipynb` (selection, concentration and claim-volume checks, paired model comparisons, new-insurer tests).
Final paper and presentation: Week 13.

# Is Claim Denial a Property of the Insurer or the Plan?

ANLY 530 (Principles and Applications of Machine Learning), KA1 conference-paper project. Author: Ankit Bisht.

## Research question

How much of the variation in claim denial rates across ACA Marketplace plans is attributable to the insurer rather than to the plan's design, and can publicly reported plan attributes predict a plan's denial rate for insurers not seen during training?

## Repository contents

| File | Purpose |
|---|---|
| `claim_denial_exploratory.ipynb` | Data audit, target description, insurer-versus-plan variance decomposition, and prediction under random versus insurer-grouped cross-validation |
| `requirements.txt` | Python packages and versions |

## Data

CMS Transparency in Coverage Public Use Files, individual-market medical plans (sheet "Ind QHP"):

- Plan year 2026 file: claims for calendar year 2024 (main analysis)
- Plan year 2025 file: claims for calendar year 2023 (prior-year insurer denial rate)

Source: https://www.cms.gov/marketplace/resources/data/public-use-files

One row is one plan. The target is the in-network claim denial rate (claims denied / claims received). The data are public, aggregated and contain no patient information. The notebook downloads both files from CMS on first run and caches them in `data/`, which is excluded from the repository.

## How to run

```
pip install -r requirements.txt
jupyter notebook claim_denial_exploratory.ipynb
```

Run all cells from top to bottom. The first run needs an internet connection to download the CMS files (about 2 MB). Seed: 42.

## Evaluation design

- Metric: mean absolute error of the denial rate (0.01 = one percentage point).
- Baseline: the training-fold mean, scored on the same folds as every model.
- Folds grouped by insurer (`GroupKFold`), so no insurer appears in both training and test data; random 5-fold is reported only to show how much it overstates performance.
- Ridge penalty tuned inside each training fold with inner folds also grouped by insurer.
- Denial reasons, resubmissions, appeals and same-year insurer totals are excluded because they are recorded alongside or after the denials.

## Status

Milestone 1 (Week 10): research protocol and exploratory notebook.
Milestone 2 (Week 12): temporal validation (2023 to 2024), better use of insurer history, bias check on excluded plans.
Final paper and presentation: Week 13.

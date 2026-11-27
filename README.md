# Is Claim Denial a Property of the Insurer or the Plan?

ANLY 530 (Principles and Applications of Machine Learning), Harrisburg University of Science and Technology. Final research project (KA2). Author: Ankit Bisht.

## Research question

Insurers selling ACA Marketplace plans must report how many claims they receive and deny, and CMS publishes the counts plan by plan. This project asks:

1. **RQ1 (primary).** How much of the variation in plan denial rates lies between insurers rather than among plans of the same insurer, and does that share hold when excluded plans, insurer concentration, claim volume and other years are taken into account?
2. **RQ2.** Can publicly reported plan attributes, alone or with the insurer's prior-year denial rate, predict denial rates for established insurers left out of model training?
3. **RQ3.** How well can denial rates be predicted for insurers in their first year on the Marketplace?

## Main findings

- About 61% of the variance in 2024 plan denial rates lies between insurers (95% cluster-bootstrap interval 45% to 74%); plan type, metal level and exchange type explain 3.7% of that insurer variance. Every robustness check keeps the share above one half.
- With whole insurers held out (10 x 5-fold cross-validation grouped by insurer), no model predicts 2024 plan denial rates measurably better than the training mean, although random splits suggest about 40% lower error. Plan attributes alone never help; the only gains (claims year 2023, and without the largest insurer) come from gradient boosting with the insurer's prior-year denial rate.
- Insurers in their first claims year have prediction errors 1.65 times larger than other insurers, because their denial rates vary about twice as widely.

## Repository contents

| Path | Purpose |
|---|---|
| `final_analysis.ipynb` | **Final paper (KA2).** Reproduces every table and figure in the manuscript and writes them to `results/` |
| `results/tables/` | Tables 1 to 7 of the analysis as CSV files (written by `final_analysis.ipynb`) |
| `results/figures/` | Figures 1 to 4 of the manuscript (written by `final_analysis.ipynb`) |
| `milestone2_analysis.ipynb` | Milestone 2 progress report analysis (kept unchanged; writes `figures/m2_fig*.png`) |
| `claim_denial_exploratory.ipynb` | Milestone 1 exploratory notebook (kept unchanged) |
| `paper_analysis.ipynb` | Analysis for an earlier draft paper (writes `figures/fig*.png`) |
| `manuscript/` | The final research paper (PDF and Word) |
| `data/README.md` | Where the CMS files come from and how the notebooks obtain them |
| `requirements.txt` | Python packages with the versions used |
| `LICENSE` | MIT license for the code (the CMS data are public and are not stored here) |

## Data

Public CMS Transparency in Coverage files for plan years 2024, 2025 and 2026, which report claims for calendar years 2022, 2023 and 2024. One row is one plan; the target is the in-network denial rate (claims denied / claims received). The files are public and contain no patient information. The notebooks download them from CMS on first run and cache them in `data/`, which is not committed. See [data/README.md](data/README.md) for sources, codes and the differences between files.

## How to reproduce the results

1. Install Python 3.12 or later (results were produced with Python 3.14.2) and the packages:

   ```
   pip install -r requirements.txt
   ```

2. Run the final notebook from top to bottom, either in Jupyter or from the command line:

   ```
   jupyter nbconvert --to notebook --execute --inplace --ExecutePreprocessor.timeout=5400 final_analysis.ipynb
   ```

   The first run needs an internet connection to download the CMS files (about 3 MB). The full run takes about 10 minutes on a 16-core computer and uses all CPU cores, because every model is fitted on more than 200 cross-validation folds with tuning inside each fold.

3. The tables and figures appear in `results/`:

   | Manuscript item | Output file |
   |---|---|
   | Table 1 (coverage) | `results/tables/table1_coverage.csv` |
   | Table 2, Figure 2 (insurer share under each check) | `results/tables/table2_insurer_share.csv`, `results/figures/fig2_insurer_share.png` |
   | Table 3 (plan-attribute effects) | `results/tables/table3_plan_attribute_effects.csv` |
   | Table 4 (random vs. grouped validation) | `results/tables/table4_model_comparison_2024.csv` |
   | Table 5, Figure 3 (robustness of the model comparison) | `results/tables/table5_model_comparison_sensitivity.csv`, `results/figures/fig3_paired_differences.png` |
   | Table 6, Figure 4 (first-year insurers) | `results/tables/table6_year_ahead_new_insurers.csv`, `results/figures/fig4_new_insurers.png` |
   | Prior-year history (Section 4.6) | `results/tables/table7_prior_year_history.csv` |
   | Figure 1 (plan and insurer rates) | `results/figures/fig1_rates.png` |

   The CSV files store proportions (0.01 = one percentage point); the manuscript reports the same values in percentage points.

The Milestone 1 and Milestone 2 notebooks run the same way and are kept as submitted.

## Evaluation design

- **Metric:** mean absolute error of the denial rate.
- **Baseline:** the training-fold mean, scored on the same folds as every model.
- **Main validation:** five-fold cross-validation grouped by insurer (`GroupKFold`), repeated 10 times with different insurer assignments (50 test folds). No insurer appears in both training and test data. Random five-fold results are reported only to show how much they overstate performance.
- **Model comparison:** fold-by-fold difference from the baseline with a 95% corrected resampled t interval (Nadeau & Bengio, 2003). Repeated with claim-volume weights, without the largest insurer and on claims year 2023.
- **Models:** ridge regression, random forest and gradient boosting, each tuned inside every training fold on inner folds grouped by insurer.
- **Leakage controls:** all preprocessing (one-hot encoding, imputation, scaling) is fitted inside the training folds; denial reasons, resubmissions, appeals and same-year insurer totals are excluded.
- **New insurers:** the CMS new-insurer flag identifies insurers in their first claims year; models trained on one year predict the next.

## Reproducibility notes and limitations

- Seeds: 42 for models and bootstraps; repeat r of the grouped split uses seed r. Folds run in parallel, but each is seeded, so results do not depend on the order in which folds finish.
- `statsmodels` prints a "MLE may be on the boundary" notice for every mixed-model fit, including fits whose variance components are clearly positive; the notebook silences it and prints the variance components so they can be checked.
- `GroupKFold(shuffle=True)` requires scikit-learn 1.6 or later. Other package versions may change results in the last decimal place.
- The data are self-reported plan-level aggregates for states on the federal platform. Results describe patterns, not individual claim decisions or causal effects.

## Status

- Milestone 1 (Week 10): research protocol and exploratory notebook.
- Milestone 2 (Week 12): progress report and `milestone2_analysis.ipynb`.
- Final paper and recorded presentation (Week 13): `final_analysis.ipynb` and `results/`.

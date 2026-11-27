# Data

The analysis uses the **CMS Transparency in Coverage Public Use Files** (individual-market medical plans, sheet "Transparency {year} - Ind QHP"). They are public files published by the Centers for Medicare & Medicaid Services (CMS); no registration or data-use agreement is needed.

| Plan-year file | Claims year it reports | Local file name |
|---|---|---|
| 2024 | 2022 | `transparency_in_coverage_puf_2024.xlsx` |
| 2025 | 2023 | `transparency_in_coverage_puf_2025.xlsx` |
| 2026 | 2024 | `transparency_in_coverage_puf_2026.xlsx` |

- Landing page: https://www.cms.gov/marketplace/resources/data/public-use-files
- Direct downloads used by the notebooks: `https://download.cms.gov/marketplace-puf/{year}/transparency-in-coverage-puf.zip` for year 2024, 2025 and 2026.

## How the files get here

You do not need to download anything by hand. Each notebook checks this folder and, if a file is missing, downloads the zip from CMS, extracts the Excel workbook and saves it here under the name above (about 3 MB in total). To use a copy you downloaded yourself, save the workbook from inside the zip under the matching name.

The Excel files are not stored in the repository (`.gitignore` excludes everything in this folder except this README).

## What is in the files

- One row is one plan offered on the Marketplace in the plan year, identified by `Plan_ID` and a state-specific `Issuer_ID`.
- Claims counts refer to the calendar year two years before the plan year, so only plans that existed in the claims year can report them.
- The data are aggregated counts reported by insurers. They contain no patient information.
- Codes used instead of numbers: `N/A` (plan or insurer new to the Exchange), `*` (not available), `**` (small count suppressed), `***` (not required for this plan type). The notebooks read these as missing, never as zero.

## Known differences between files

- The plan-year 2024 file has no `Exchange_Type` column and names enrollment `Enrollment_Data` (later files: `Average Monthly Enrollment`). Exchange type is fixed within a state, so `final_analysis.ipynb` fills it from the state.
- The enrollment field is not on the same scale in every file (for the same plans, the claims-year 2024 values are a median 12% of the 2023 values), so enrollment is used only within a year.
- Denial counts by reason add up to more than total in-network denials, so reason counts are not used.

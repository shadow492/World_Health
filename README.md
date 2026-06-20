# WHO World Health Statistics 2020 — Data Analytics Project

A multi-phase data analytics project applying the WHO World Health Statistics 2020
dataset to seven research questions spanning life expectancy, communicable disease
burden, water/sanitation/hygiene (WASH) access, healthcare workforce composition, and
financial protection against health expenditure.

## Project Overview

This project follows the six-phase data analytics lifecycle — Ask, Prepare, Process,
Analyze, Share, Act — from raw CSV files through to statistically tested findings.
Seven research questions were defined in Phase 1 and carried through data cleaning
(Phase 3) and analysis (Phase 4):

| # | Question | Focus |
|---|---|---|
| Q1 | Global HALE vs. life expectancy trend, 2000–2019 | Healthy life expectancy trend and sex-disaggregated gap |
| Q2 | NTD intervention burden trend, 2010–2018 | Global trend, country rankings, largest movers |
| Q3 | Basic vs. safe sanitation quality gap | Country-level gap and urban/rural comparison |
| Q4 | Catastrophic health expenditure trend, 1985–2018 | Long-run trend, country rankings, convergence/divergence |
| Q5 | Pharmacist-to-doctor ratio as a predictor of financial hardship | Correlation with catastrophic health expenditure |
| Q6 | NTD burden quartile progress analysis | Whether high-burden countries progress more slowly |
| Q7 | WASH infrastructure floor for disease control | Threshold effect on malaria and TB incidence |

The project has completed Phase 4 (Analyze). Phase 5 (Share) and Phase 6 (Act) are
the next steps.

## Repository Structure

```
.
├── README.md
├── METHODOLOGY.md
├── requirements.txt
├── metadata_report.md
├── Phase1_FinalQuestions.md
├── Phase1_Questions_WHO_WorldHealth.md
├── Phase3_DataCleaning_Report.md
├── Phase4_Analysis_Report.md
├── data/
│   ├── CommunicableDisease/
│   ├── DrinkingWater/
│   ├── HealthWorkForce/
│   ├── LifeExpectencyAndHealthyLifeExpectency/
│   ├── UHCincludingFinancialRisk/
│   └── phase3_outputs/
└── notebooks/
    ├── notebook_1.ipynb
    ├── data_cleaning.ipynb
    ├── Q1.ipynb
    ├── Q2.ipynb
    ├── Q3.ipynb
    ├── Q4.ipynb
    ├── Q5.ipynb
    ├── Q6.ipynb
    └── Q7.ipynb
```

### Data

`data/` contains the raw CSV files from the WHO World Health Statistics 2020 dataset,
organized into the five indicator categories used in this project: communicable
disease, drinking water and sanitation, health workforce, life expectancy, and
universal health coverage / financial risk.

`data/phase3_outputs/` contains the cleaned, question-specific CSVs produced during
Phase 3 (e.g. `q1_hale_vs_le.csv`, `q6_ntd_quartile.csv`). These are the files the
Phase 4 notebooks load directly; they are not regenerated from raw data unless
`data_cleaning.ipynb` is re-run.

### Notebooks

- `notebook_1.ipynb` — initial data exploration and metadata extraction; produced `metadata_report.md`
- `data_cleaning.ipynb` — Phase 3 cleaning pipeline; produced the `phase3_outputs/` CSVs and `Phase3_DataCleaning_Report.md`
- `Q1.ipynb`, `Q2.ipynb`, `Q3.ipynb`, `Q4.ipynb`, `Q5.ipynb`, `Q6.ipynb`, `Q7.ipynb` — Phase 4 analysis notebooks, one per research question

### Documentation

- `metadata_report.md` — column-level metadata (types, fill rates, value ranges) for every raw file
- `Phase1_Questions_WHO_WorldHealth.md` — dataset scoping notes and known constraints established before analysis began
- `Phase1_FinalQuestions.md` — the seven research questions, with datasets used, what each answers, and scope notes
- `Phase3_DataCleaning_Report.md` — full record of every cleaning and transformation operation
- `Phase4_Analysis_Report.md` — full record of every analysis, statistical test, and finding, by question
- `METHODOLOGY.md` — consolidated methodology summary across all phases

## How to Reproduce

1. Install dependencies:
   ```
   pip install -r requirements.txt
   ```
2. Run `notebooks/data_cleaning.ipynb` to regenerate `data/phase3_outputs/` from the raw files in `data/`.
3. Run any of the `Q2.ipynb` – `Q30.ipynb` notebooks independently. Each loads its corresponding file(s) from `phase3_outputs/` and does not depend on the others.

## Data Source

WHO World Health Statistics 2020, accessed via Kaggle. The data is included in this
repository as downloaded for the purposes of this analysis; refer to the original
Kaggle dataset listing for usage terms.

## Key Limitations

A full, per-question account of limitations is in `Phase4_Analysis_Report.md`
(Section 3) and `Phase3_DataCleaning_Report.md` (Section 5). The recurring constraints
across the project are:

- No file in the dataset contains population, income, or GDP figures, so every
  "global" figure is an unweighted average across countries, and absolute-count
  rankings (e.g. NTD burden) are confounded by population size rather than reflecting
  pure burden severity.
- Country coverage varies by question — from roughly 100 countries (Q13) to 195
  (Q10, Q28) — so no result should be read as globally representative without
  checking the specific N for that question.
- Snapshot-based analyses (Q20, Q25) compare variables drawn from each source file's
  own latest reported year independently, which can introduce a year mismatch between
  the variables being compared.
- All relationships identified are correlational. No file contains policy timing,
  intervention coverage, or other causal-mechanism data.

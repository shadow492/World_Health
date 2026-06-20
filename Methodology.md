# Methodology

This document consolidates the methodology applied across the project's four
completed phases. Full operation-by-operation detail lives in
`Phase3_DataCleaning_Report.md` and `Phase4_Analysis_Report.md`; this document is
the connective summary of what was decided, why, and how it was applied
consistently across questions.

## 1. Analytical Framework

The project follows the six-phase data analytics lifecycle: Ask, Prepare, Process,
Analyze, Share, Act. Phases 1 through 4 are complete. Each phase produced a written
artifact before the next phase began, so that every transformation and statistical
choice in Phase 4 traces back to a decision recorded earlier rather than being made
ad hoc during analysis.

## 2. Phase 1 — Ask

Seven research questions were defined against the actual structure of the dataset —
column types, country coverage, time ranges, and disaggregation availability — as
documented in `metadata_report.md`. Each question's source files, what it would
specifically answer, and its scope limitations were written down in
`Phase1_FinalQuestions.md` before any code was written.

Two structural constraints shaped how the questions were framed from the start.
First, no file in the dataset contains population, income, or GDP data, which rules
out per-capita normalization or wealth-controlled analysis anywhere in the project.
Second, country coverage varies sharply by file (from under 100 to 195 countries),
which determines which questions can support a global claim and which cannot. Q6
was framed as a quartile-progress question rather than a population-threshold
question specifically because of the first constraint: testing a true threshold
effect against absolute NTD burden would have required external population data to
convert the NTD count into a rate.

## 3. Phase 2 / Phase 3 — Prepare & Process

Eleven raw CSV files across five subfolders were loaded and cleaned in a single
pipeline (`data_cleaning.ipynb`), producing ten analysis-ready CSVs in
`data/phase3_outputs/`. Two categories of operation were applied:

**Global operations**, applied to every file before question-specific work began:
column name standardization (`Location → country`, `Period → year`, `Dim1 → dim1`,
`First Tooltip → value`) and an inspection pass (shape, dtypes, null percentage) to
establish a factual baseline before any transformation.

**File-level and question-specific operations**, applied only where needed:

- The TB incidence file stores values as uncertainty-range strings
  (e.g. `"183 [150–220]"`); the central estimate was extracted by splitting on `[`
  and converting to float, since the point estimate — not the confidence interval —
  is what trend and threshold analysis requires.
- The life expectancy file is the only one extending to 1920; it was scoped to
  `year >= 2000` for Q2 so it would join cleanly against the HALE file, which begins
  at 2000.
- Files carrying a `dim1` column with Total/Urban/Rural rows per country-year
  (sanitation, safe sanitation, catastrophic expenditure) were split into named
  Total/Urban/Rural slices. Aggregating across all three values without splitting
  first would triple-count every country-year observation.
- The two workforce files used in Q5 (pharmacists, doctors) were reduced to one
  row per country by taking each country's latest reported year, since reporting
  years are irregular and not every country shares a common latest year.
- The NTD file was pivoted to one row per country with `count_2010` and `count_2018`
  columns for Q6, with `abs_change`, `pct_change`, and a 2010-baseline quartile
  label (`burden_q`) derived directly in the cleaning step.

## 4. Phase 4 — Analyze: Cross-Cutting Conventions

Several methodological conventions were established once and then applied
consistently across every question that encountered the relevant situation, rather
than being decided independently each time:

**Unweighted country means for "global" figures.** With no population data anywhere
in the project, every global trend (Q2, Q10, Q13, Q20) is a simple average across
countries, with each country weighted equally regardless of population. A statement
like "the global rate fell from X to Y" describes the average country, not the
average person.

**Period averages over single-year snapshots, where coverage is consistent.** For
country-level rankings, the full-period average (e.g. 2000–2019 for Q2's top-10 gap
ranking, or the full window for Q3's quadrant analysis) was preferred over a single
year, since it is less sensitive to one anomalous year. Where reporting years are
irregular across countries instead (Q3, Q5), a latest-year-per-country snapshot
was used, since forcing a single common year would drop most non-reporting
countries.

**Explicit `np.isfinite()` filtering before any statistical test on a percentage
change.** A percentage change computed from a zero or near-zero baseline produces
`NaN` (0/0) or `inf` (nonzero/0). This was confirmed concretely in Q28, where 13
countries had a 2010 NTD baseline of zero; without filtering, `spearmanr` returned
`NaN` for the entire result rather than for just those rows.

**Kruskal-Wallis in place of one-way ANOVA for skewed, small-baseline-driven
distributions.** Percentage-change distributions built from small baseline counts
are heavily right-skewed, since a handful of countries can show very large changes
from a tiny base. ANOVA assumes roughly normal, equal-variance groups; Kruskal-Wallis
is the rank-based, distribution-free alternative and was used for the Q28 quartile
comparison and as the basis for the equivalent two-group choice (Mann-Whitney) in
Q7's threshold search.

**Spearman alongside Pearson wherever normality or linearity is uncertain.** Ratio
and count-based metrics (the pharmacist:doctor ratio in Q5, NTD burden in Q6) are
typically right-skewed rather than normal, so Spearman (monotonic, rank-based) was
computed alongside Pearson (linear) rather than relying on either alone.

**Explicit string `labels=` in `pd.qcut()`.** Calling `qcut()` without `labels=`
returns `pandas.Interval` objects rather than strings as the category values. Code
that later filters on a string label against an Interval-typed column will silently
return an empty result. This was diagnosed during Q6 and fixed by always passing
explicit string labels, with a defensive check kept in the code in case the dataset
is regenerated.

## 5. Question-by-Question Methods Summary

| Question | Datasets | Primary methods |
|---|---|---|
| Q1 | HALE + life expectancy | Global trend, gap (LE − HALE), CAGR, sex-disaggregated gap, top-10 country gap ranking |
| Q2 | NTD interventions | Global sum trend with YoY change, latest-year snapshot ranking, 2010→2018 endpoint pivot |
| Q3 | Basic + safe sanitation | Global trend, scatter vs. basic=safe diagonal, median-split quadrant ranking, urban/rural comparison |
| Q4 | Catastrophic expenditure (10%) | Coverage check, global trend, latest-year ranking, annualized first-vs-last change, cross-country standard deviation, urban/rural comparison |
| Q5 | Pharmacists + doctors + catastrophic expenditure | Pearson and Spearman correlation, median-split group comparison, top-10 ratio chart |
| Q6 | NTD interventions (2010, 2018 pivot) | `isfinite` filtering, median/IQR by quartile, Kruskal-Wallis, Spearman, per-quartile outlier extraction |
| Q7 | Drinking water + sanitation + malaria + TB | Period-average collapse, Spearman baseline check, decile binning, Mann-Whitney threshold grid search, water×sanitation quadrant analysis |

Full detail for each row — every transformation, the justification for each test
choice, and how the outputs map back to the original question — is in
`Phase4_Analysis_Report.md`.

## 6. Limitations Carried Across the Project

- **No population, income, or GDP data anywhere in the dataset.** Every global
  figure is an unweighted country average, and every absolute-count ranking (Q2,
  Q6) reflects population size as a confound rather than pure burden severity.
- **Country coverage varies by question** — roughly 100 for Q3, 108 for Q7's
  malaria side, 130–140 for Q25, 153 for Q20, and 195 for the rest. No result should
  be described as globally representative without restating the actual N for that
  analysis.
- **Snapshot-based comparisons (Q4, Q5) carry a year-mismatch risk**, since each
  variable was taken from its own source file's latest available year
  independently rather than a forced common year.
- **All relationships identified are correlational.** No file in the dataset
  contains policy timing, intervention coverage, or other causal-mechanism data, so
  none of the findings in Phase 4 support a causal claim on their own.

## 7. Tools and Environment

- **Environment:** Jupyter Notebook, run locally
- **Language:** Python
- **Libraries:** pandas, numpy, scipy, matplotlib
- **Data source:** WHO World Health Statistics 2020, accessed via Kaggle

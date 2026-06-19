# Phase 3 — Data Cleaning & Preparation Report
## WHO World Health Statistics 2020

> **Lifecycle Stage:** Phase 3 (Process)
> This document records every cleaning and transformation operation applied to the raw
> CSV files, the objective behind each operation, and how each operation contributed
> to producing the final analysis-ready dataset for each question.

---

## Overview

11 raw CSV files were loaded across 5 subfolders. The cleaning pipeline was executed in
a single Jupyter Notebook and produced 10 analysis-ready dataframes, exported as CSVs
to a `phase3_outputs` folder. The pipeline is divided into global operations (applied
to all files) and question-specific operations (applied only to the files relevant to
each question).

---

## Section 1 — Global Operations

These operations were applied to all 11 files before any question-specific work began.

---

### 1.1 Column Name Standardisation

**Applied to:** All 11 files

**Operation:** Renamed four columns to consistent lowercase names across all files.

| Original Name   | Standardised Name |
|-----------------|-------------------|
| `Location`      | `country`         |
| `Period`        | `year`            |
| `Dim1`          | `dim1`            |
| `First Tooltip` | `value`           |

**Objective:** The WHO CSV files use title-case column names with spaces, which are
error-prone to type repeatedly and inconsistent across files. Standardising to
lowercase single-word names before any downstream work means every subsequent
operation uses the same column references regardless of which file it is operating on.
The rename was written defensively — only columns that actually exist in a given file
were renamed, so files missing `dim1` (e.g. the NTD file) were not affected.

---

### 1.2 Inspection Pass

**Applied to:** All 11 files

**Operation:** For each file, printed shape, column names, data types, and null
percentage per column.

**Objective:** Establish a factual baseline of what the data looks like before any
transformation. This step surfaces dtype mismatches, unexpected nulls, and column
name variations that must be resolved before analysis. It also confirms that files
described as fully numeric in Phase 1 are indeed numeric, and that the one known
string-encoded column (TB incidence) is identifiable by dtype.

---

## Section 2 — File-Level Cleaning Operations

---

### 2.1 TB Incidence — String Column Parsing

**File:** `incedenceOfTuberculosis.csv`
**Used in:** Q30

**Operation:** The `value` column in this file stores incidence figures as
uncertainty-range strings in the format `"183 [150–220]"`. A parsing function was
applied to every row that splits the string on the `[` character, takes the left
portion (the central estimate), strips whitespace, and converts to `float`. Rows
where conversion failed were assigned `NaN`.

**Objective:** The column was of type `object` (string), making it unusable for any
numeric computation. The central estimate — the number to the left of the bracket —
is the WHO's reported point estimate for incidence and is the correct value to use for
trend and threshold analysis. The uncertainty range itself is not needed for this
project. After parsing, the column was verified to have the correct `float64` dtype
and an acceptable null count before proceeding.

**Why the central estimate:** The values inside the brackets are lower and upper bounds
of a confidence interval. For trend analysis and correlation, the point estimate is
the standard choice. Using bounds would require two separate numeric columns and a
more complex analysis design that is out of scope for this project.

---

### 2.2 Life Expectancy — Year Scope Restriction

**File:** `lifeExpectancyAtBirth.csv`
**Used in:** Q2

**Operation:** Filtered the dataframe to rows where `year >= 2000`, discarding all
observations from 1920 to 1999.

**Objective:** The life expectancy file is the only file in the entire dataset that
extends to 1920. All other files begin at 2000 at the earliest, and the HALE file
(which must be joined to life expectancy for Q2) begins at exactly 2000. Retaining
the pre-2000 rows would serve no purpose for Q2 and would add noise to any global
aggregation. Restricting to 2000 onwards ensures that the join with HALE produces
a clean inner match without any unmatched rows on the life expectancy side.

---

### 2.3 Dim1 Files — Urban / Rural / Total Split

**Files:** `atLeastBasicSanitizationServices.csv`, `safelySanitization.csv`,
`population10SDG3.8.2.csv`
**Used in:** Q13, Q20, Q25, Q30

**Operation:** Each file contains three rows per country per year — one for `Total`,
one for `Urban`, and one for `Rural` — stored in the `dim1` column. Each file was
split into three separate named dataframes by filtering on the `dim1` value:

- `san_total`, `san_urban`, `san_rural`
- `safe_total`, `safe_urban`, `safe_rural`
- `pop10_total`, `pop10_urban`, `pop10_rural`

**Objective:** Aggregating across all three `dim1` values in a single operation would
triple-count every country-year observation, producing meaningless global totals and
averages. By splitting into named slices at the cleaning stage, every downstream cell
is forced to make an explicit choice about which population group it is operating on.
The `Total` slices are used for national-level trend analysis. The `Urban` and `Rural`
slices are used for equity gap analysis where the question specifically requires
comparing residence types.

---

### 2.4 Workforce Files — Latest Year Per Country Snapshot

**Files:** `medicalDoctors.csv`, `pharmacists.csv`
**Used in:** Q25

**Operation:** Both workforce files contain multiple years of observations per country,
but reporting is irregular — not every country has the same most recent year. A
function was applied to each file that sorts by `year` descending, drops duplicate
country rows keeping only the first (most recent) observation, and resets the index.
This produced two single-row-per-country dataframes: `doctors_latest` and
`pharmacists_latest`.

**Objective:** Q25 asks for the pharmacist-to-doctor ratio as a structural
characteristic of each country's healthcare workforce — a current snapshot, not a
time series. Using a time series for a ratio calculation would require aligning both
files to a common year, which is complicated by the irregular reporting schedule.
Taking the latest available year per country provides the most current picture of
workforce composition while avoiding the complexity of multi-year alignment. The year
column was retained in the output so that the snapshot year is visible and can be
acknowledged in Phase 4.

---

## Section 3 — Question-Specific Dataset Construction

---

### Q2 — Global HALE vs. Life Expectancy Trend (2000–2019)

**Raw files used:** `HALElifeExpectancyAtBirth.csv`, `lifeExpectancyAtBirth.csv`
**Output file:** `q2_hale_vs_le.csv`

**Operations:**

1. Renamed `value` column in each file to `hale` and `le` respectively before merging,
   to distinguish the two metrics after the join.
2. Performed an inner join on `country + year + dim1`.
3. Added a derived column `hale_yield = hale / le`.

**How the final dataset was formed:**

Both files share 184 countries, the same `dim1` categories (Male, Female, Both sexes),
and the same 2000–2019 year range after the life expectancy scope restriction in
Section 2.2. The inner join on all three keys produces one row per country per year
per sex category with both the HALE and life expectancy values side by side. The
`hale_yield` column (healthy years as a proportion of total years) is a derived metric
needed for Phase 4 to assess whether healthy years are growing faster or slower than
total years. The join uses `how="inner"` to retain only country-year-sex combinations
present in both files, ensuring no rows carry a null in either metric.

---

### Q10 — NTD Intervention Burden Trend (2010–2018)

**Raw files used:** `interventionAgianstNTDs.csv`
**Output file:** `q10_ntd_trend.csv`

**Operations:**

1. Selected `country`, `year`, and `value` columns only, discarding any metadata
   columns not needed for analysis.

**How the final dataset was formed:**

The NTD file required no cleaning beyond column standardisation. The `value` column
was already stored as `int64` with 100% fill across 195 countries and 9 years
(2010–2018). The full time series is retained for Q10 so that Phase 4 can compute
year-on-year global totals and country-level rankings across the entire period.

---

### Q28 — NTD Burden Quartile Progress Analysis (2010–2018)

**Raw files used:** `interventionAgianstNTDs.csv`
**Output file:** `q28_ntd_quartile.csv`

**Operations:**

1. Filtered the NTD time series to rows where `year` is 2010 or 2018 only.
2. Pivoted the result so each country occupies one row with columns `count_2010` and
   `count_2018`.
3. Dropped countries where either endpoint year was missing.
4. Added `abs_change = count_2018 − count_2010`.
5. Added `pct_change = abs_change / count_2010 × 100`.
6. Added `burden_q` by applying `pd.qcut()` to `count_2010` with 4 bins, labelled
   Q1 (lowest) through Q4 (highest).

**How the final dataset was formed:**

Q28 asks whether the highest-burden countries make proportionally less progress than
lower-burden countries. To answer this, the analysis needs each country's starting
burden (2010), ending burden (2018), and a grouping variable that places them into
burden tiers. The pivot collapses the time series into a single analytical row per
country. The `pct_change` column is the progress metric. The `burden_q` column is
the grouping variable — assigning quartiles based on the 2010 baseline means the
groups reflect burden at the start of the observation period, not the end, which is
the correct reference point for measuring progress. Countries missing either endpoint
were dropped because progress cannot be computed without both values.

---

### Q13 — Basic vs. Safe Sanitation Quality Gap

**Raw files used:** `atLeastBasicSanitizationServices.csv`, `safelySanitization.csv`
**Output files:** `q13_sanitation_gap.csv`, `q13_sanitation_gap_ur.csv`

**Operations:**

1. Used the `san_total` and `safe_total` slices from the Dim1 split (Section 2.3).
2. Renamed `value` to `basic_pct` and `safe_pct` respectively before merging.
3. Performed an inner join on `country + year`.
4. Added `quality_gap = basic_pct − safe_pct`.
5. Repeated steps 2–4 using the Urban and Rural slices together (retaining `dim1`)
   to produce a second dataset for equity analysis.

**How the final dataset was formed:**

The quality gap metric requires both percentages to be present for the same country
and year. The inner join enforces this — only country-years present in both files are
retained. The safe sanitation file covers approximately 100 countries versus 195 in
the basic file, so the join naturally restricts the dataset to the ~100 country
intersection. This scope limitation is a data constraint, not a cleaning choice, and
is documented here so it can be acknowledged explicitly in Phase 4. The urban/rural
version retains the `dim1` column so that Phase 4 can compare the quality gap between
residence types within the same country.

---

### Q20 — Catastrophic Health Expenditure Trend (1985–2018)

**Raw files used:** `population10SDG3.8.2.csv`
**Output files:** `q20_catastrophic_exp.csv`, `q20_catastrophic_exp_ur.csv`

**Operations:**

1. Used the `pop10_total` slice for the national trend dataset, selecting `country`,
   `year`, and `value` and renaming `value` to `catex_pct`.
2. Concatenated the `pop10_urban` and `pop10_rural` slices (retaining `dim1`) for
   the urban/rural gap dataset.

**How the final dataset was formed:**

The Dim1 split in Section 2.3 was the critical operation here. Without it, aggregating
the file directly would sum Urban, Rural, and Total rows for the same country-year,
tripling all values. The `Total` slice gives the national-level figure used for trend
analysis over the full 1985–2018 window. The concatenated Urban + Rural dataset allows
Phase 4 to compare financial hardship between residence types. No year filtering was
applied — the full 1985–2018 range is retained because Q20 specifically asks about the
long-run trend, which is the longest time window available in any file in this project.

---

### Q25 — Pharmacist-to-Doctor Ratio as Predictor of Financial Hardship

**Raw files used:** `pharmacists.csv`, `medicalDoctors.csv`, `population10SDG3.8.2.csv`
**Output file:** `q25_pharma_doctor_ratio.csv`

**Operations:**

1. Applied the latest-year-per-country snapshot to both workforce files (Section 2.4),
   producing `doctors_latest` and `pharmacists_latest`.
2. Renamed `value` to `doc_per_10k` and `pharm_per_10k` respectively.
3. Merged the two workforce snapshots on `country` with an inner join.
4. Added `pharm_doc_ratio = pharm_per_10k / doc_per_10k`, with zero doctor values
   replaced by `NaN` before division to prevent division-by-zero errors.
5. Applied the latest-year-per-country function to `pop10_total` to get the most
   recent expenditure figure per country.
6. Joined the ratio dataframe to the expenditure snapshot on `country` with an inner
   join.

**How the final dataset was formed:**

This is the only three-file join in the project. The workforce snapshots are joined
first because both measure the same thing (workforce density) at the same point in
time (latest available year). The ratio is derived from this merged workforce frame.
The expenditure snapshot is then joined to add the outcome variable (catastrophic
expenditure rate). Using latest-year snapshots for both workforce and expenditure —
rather than aligning on a specific common year — maximises country coverage, since
not all countries reported data in the same year. The inner join across all three
files reduces the country count to approximately 130–140, which is the natural
intersection of the three files' coverage.

---

### Q30 — WASH Infrastructure Floor for Communicable Disease Control

**Raw files used:** `basicDrinkingWaterServices.csv`,
`atLeastBasicSanitizationServices.csv`, `incedenceOfMalaria.csv`,
`incedenceOfTuberculosis.csv`
**Output files:** `q30_wash_malaria.csv`, `q30_wash_tb.csv`

**Operations:**

1. Used the `san_total` slice (Section 2.3) for the sanitation side of the WASH join.
2. Renamed `value` to `water_pct` and `sanitation_pct` respectively.
3. Merged `water` and `san_total` on `country + year` (inner join) to create a
   combined `wash` dataframe with both WASH coverage metrics per country-year.
4. Joined `wash` to `malaria` on `country + year` (inner join), renaming the malaria
   `value` column to `malaria_inc`.
5. Joined `wash` to `tb` (post string-parse) on `country + year` (inner join),
   renaming the TB `value` column to `tb_inc`.

**How the final dataset was formed:**

The WASH-disease threshold analysis requires all four metrics — water coverage,
sanitation coverage, and disease incidence — to be present for the same country and
year. The join is performed in two steps: first combining the two WASH files into a
single frame, then joining to each disease file separately. This produces two output
datasets rather than one because the malaria file covers only 108 countries (versus
195 for TB), and forcing a four-way join would unnecessarily discard TB data for the
87 countries not in the malaria file. Keeping them separate allows Phase 4 to run the
threshold analysis independently for each disease and then compare results. The TB
`value` column was already parsed to `float64` by the string-parsing step in
Section 2.1 before this join was executed.

---

## Section 4 — Output Summary

| Output File                    | Source Files                                      | Rows | Countries | Year Range  |
|--------------------------------|---------------------------------------------------|------|-----------|-------------|
| `q2_hale_vs_le.csv`            | HALElifeExpectancy, lifeExpectancyAtBirth         | —    | 184       | 2000–2019   |
| `q10_ntd_trend.csv`            | interventionAgianstNTDs                           | —    | 195       | 2010–2018   |
| `q13_sanitation_gap.csv`       | atLeastBasicSanitation, safelySanitization        | —    | ~100      | 2000–2017   |
| `q13_sanitation_gap_ur.csv`    | atLeastBasicSanitation, safelySanitization        | —    | ~100      | 2000–2017   |
| `q20_catastrophic_exp.csv`     | population10SDG3.8.2                              | —    | 153       | 1985–2018   |
| `q20_catastrophic_exp_ur.csv`  | population10SDG3.8.2                              | —    | 153       | 1985–2018   |
| `q25_pharma_doctor_ratio.csv`  | pharmacists, medicalDoctors, population10SDG3.8.2 | —    | ~130–140  | Latest year |
| `q28_ntd_quartile.csv`         | interventionAgianstNTDs                           | —    | 195       | 2010 & 2018 |
| `q30_wash_malaria.csv`         | basicDrinkingWater, atLeastBasicSanitation, incedenceOfMalaria | — | ~108 | 2000–2017 |
| `q30_wash_tb.csv`              | basicDrinkingWater, atLeastBasicSanitation, incedenceOfTuberculosis | — | 195 | 2000–2017 |

> Row counts left blank — fill in from the Cell 16 verification output in your notebook.

---

## Section 5 — Known Limitations Carried Into Phase 4

1. **Q13 country scope.** The quality gap analysis is limited to the ~100 countries
   present in `safelySanitization.csv`. The 95 countries absent from that file cannot
   be assessed and should be acknowledged in the findings.

2. **Q25 snapshot year mismatch.** The latest available year per country differs across
   the pharmacist, doctor, and expenditure files. The ratio and the expenditure figure
   for a given country may not refer to the same year. This is a data constraint, not
   a cleaning error, but it should be noted when interpreting correlations.

3. **Q30 malaria scope.** The WASH-malaria join is limited to 108 countries. The
   WASH-TB join covers 195 countries. Cross-disease threshold comparisons must
   acknowledge this asymmetry.

4. **TB central estimate only.** The uncertainty bounds from the TB string column
   were discarded. All TB incidence values in `q30_wash_tb.csv` are point estimates.
   The true incidence lies within a confidence interval that is not represented in
   the output.

5. **All joins are correlational.** No file in this dataset contains information about
   policy changes, intervention timing, or causal mechanisms. All associations found
   in Phase 4 are observational.

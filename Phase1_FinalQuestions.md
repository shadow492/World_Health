# Phase 1 — Finalized Questions
## WHO World Health Statistics 2020

> 7 questions selected for analysis from the full Phase 1 question bank.
> Each entry documents the question, the datasets it draws from, what it
> specifically answers, and the justification for its answerability.

---

## Q2 — Global HALE vs. Life Expectancy Trend (2000–2019)

**Question:**
What is the global trend in healthy life expectancy (HALE) from 2000 to 2019?
Has HALE improved faster or slower than total life expectancy over this period?

**Datasets Used:**

| File | Metric | Period | Countries |
|---|---|---|---|
| `HALElifeExpectancyAtBirth.csv` | HALE at birth (years) | 2000–2019 | 184 |
| `lifeExpectancyAtBirth.csv` | Life expectancy at birth (years) | 1920–2019 | 184 |

**What It Answers:**
- The global year-over-year trend in HALE from 2000 to 2019
- The global year-over-year trend in total life expectancy across the same window
- Whether the gap between life expectancy and HALE has widened or narrowed — i.e., are people gaining more years of life but spending those extra years in poor health, or are healthy years keeping pace with total years?
- Which sex (male/female) is driving the trend via the `Dim1` column available in both files

**Answerability Justification:**
Both files are fully numeric (`First Tooltip` is `float64` in both), cover identical countries (184), and span an overlapping period (2000–2019). The join is straightforward on `Location + Period + Dim1`. No string parsing, no external data, and no disaggregation gap. This is the cleanest two-file join in the entire dataset.

**Scope Note:** Scope the join to 2000–2019 only. `lifeExpectancyAtBirth.csv` extends to 1920 but HALE data begins at 2000; the historical range is irrelevant for this question.

---

## Q10 — NTD Intervention Burden Trend (2010–2018)

**Question:**
How has the number of people requiring interventions for neglected tropical diseases
(NTDs) changed from 2010 to 2018? Which countries account for the largest absolute
NTD burden?

**Datasets Used:**

| File | Metric | Period | Countries |
|---|---|---|---|
| `interventionAgianstNTDs.csv` | People requiring NTD interventions (count) | 2010–2018 | 195 |

**What It Answers:**
- The global total count of people requiring NTD interventions each year from 2010 to 2018
- The year-on-year direction of change (is the burden shrinking, flat, or growing?)
- Country-level ranking by absolute burden — which 10–15 countries account for the majority of the global count
- Which countries showed the largest absolute reduction over the 8-year period, and which showed the largest increase or stagnation

**Answerability Justification:**
`First Tooltip` is stored as `int64` (range: 0 to 846,000,000) with 100% fill across 195 countries. No string parsing required. The file is single-indicator, single-disaggregation, and self-contained. All sub-questions above are answerable within this one file.

**Scope Note:** The metric is an absolute count of people, not a rate. Country comparisons should be framed as "largest burden by count," not "highest-burden relative to population." Statements like "India accounts for X% of the global NTD burden" are valid; statements like "India has higher NTD burden per capita than Country Y" are not, as population data is absent.

---

## Q13 — Basic vs. Safe Sanitation: The Quality Gap

**Question:**
How do access to basic sanitation and access to safe (managed) sanitation differ at the
country level? Which countries have high basic access but low safe access — indicating a
"quality gap" in sanitation?

**Datasets Used:**

| File | Metric | Period | Countries |
|---|---|---|---|
| `atLeastBasicSanitizationServices.csv` | Population using basic sanitation (%) | 2000–2017 | 195 |
| `safelySanitization.csv` | Population using safe sanitation services (%) | 2000–2017 | 100 |

**What It Answers:**
- For the ~100 countries present in both files: the gap between basic sanitation access % and safe sanitation access % at the country level
- Which countries sit in the "high basic, low safe" quadrant — meaning they have widespread sanitation infrastructure that does not meet safety/management standards
- Whether this quality gap has closed or widened over the 2000–2017 period
- Urban vs. rural dimension of the quality gap via the `Dim1` column available in both files

**Answerability Justification:**
Both files are `float64`, 100% fill, with a `Dim1` column (Urban/Rural/Total) enabling equity-disaggregated analysis. The join is on `Location + Period + Dim1`. The quality gap metric (basic % − safe %) is a derived column computable directly from the two files.

**Scope Note:** `safelySanitization.csv` covers 100 countries versus 195 in the basic file. The analysis is valid only for the ~100-country intersection. The 95 countries absent from the safe sanitation file cannot be assessed for a quality gap — this should be acknowledged explicitly in the findings.

---

## Q20 — Catastrophic Health Expenditure Trend (1985–2018)

**Question:**
What percentage of the population faces catastrophic health expenditure (spending more
than 10% of household income on health), and how has this changed from 1985 to 2018?

**Datasets Used:**

| File | Metric | Period | Countries |
|---|---|---|---|
| `population10SDG3.8.2.csv` | Population with health spending >10% of household income (%) | 1985–2018 | 153 |

**What It Answers:**
- The global trend in catastrophic health expenditure at the 10% threshold from 1985 to 2018
- Country-level identification of where catastrophic expenditure is highest, lowest, and changing fastest
- The urban vs. rural dimension: which residence type faces greater financial hardship, via the `Dim1` column (Urban/Rural/Total)
- Whether the burden of catastrophic expenditure is converging across countries over time or diverging

**Answerability Justification:**
`First Tooltip` is `float64` (range: 0.0 to 54.53%) with 100% fill. `Dim1` provides Urban/Rural/Total disaggregation directly. The file is self-contained for all sub-questions listed. The 1985–2018 window is the longest time range in the entire dataset, enabling genuine long-run trend analysis that no other file in this project can support.

**Scope Note:** Filter on `Dim1 = Total` for national-level trend analysis to avoid double-counting urban and rural rows in the same aggregation. Use `Dim1 = Urban` and `Dim1 = Rural` separately for the residence-gap analysis.

---

## Q25 — Pharmacist-to-Doctor Ratio as a Predictor of Financial Hardship

**Question:**
What is the pharmacist-to-doctor ratio per country, and do countries with a high ratio
also show higher rates of catastrophic health expenditure?

**Datasets Used:**

| File | Metric | Period | Countries |
|---|---|---|---|
| `pharmacists.csv` | Pharmacists per 10,000 population | 1990–2018 | 185 |
| `medicalDoctors.csv` | Medical doctors per 10,000 population | 1990–2018 | 194 |
| `population10SDG3.8.2.csv` | Population with health spending >10% of income (%) | 1985–2018 | 153 |

**What It Answers:**
- The pharmacist-to-doctor ratio for each country at the latest available year
- Whether countries with a high pharmacist:doctor ratio — indicating potential dependence on self-medication via pharmacies rather than formal clinical care — also report higher rates of catastrophic health expenditure
- Whether this relationship holds across time (using the overlapping 1990–2018 window) or is only observable at a single point

**Answerability Justification:**
All three files have `float64` numeric indicators with 100% fill. The ratio is a derived metric (pharmacists per 10,000 ÷ doctors per 10,000) computable directly. The three-file join is on `Location + Period`. The country intersection across all three files is approximately 130–140 countries, which is sufficient to detect a pattern or its absence.

**Scope Note:** Use the most recent year available per country for the ratio, since workforce data is irregularly reported and not every country has the same latest year. For the correlation with expenditure, align on a shared year range (1990–2018 intersection). A high pharmacist:doctor ratio alone does not confirm self-medication dependency — it is a proxy that this question tests, not a direct measure.

---

## Q28 — NTD Burden Quartile Progress Analysis

**Question (reframed from original):**
Do countries in the highest NTD burden quartile by absolute count make proportionally
less progress in reducing that count from 2010 to 2018 than countries in lower quartiles?

**Datasets Used:**

| File | Metric | Period | Countries |
|---|---|---|---|
| `interventionAgianstNTDs.csv` | People requiring NTD interventions (count) | 2010–2018 | 195 |

**What It Answers:**
- The absolute change in NTD intervention count per country from 2010 to 2018
- Whether countries grouped into burden quartiles (by their 2010 baseline count) show systematically different rates of progress — specifically whether the highest-burden quartile reduces its count at a slower proportional rate than lower-burden quartiles
- Which individual countries are outliers: high-burden but fast-improving, or low-burden but stagnating

**Answerability Justification:**
The file is fully self-contained for this reframed version of the question. Quartile assignment uses the 2010 baseline count; progress is measured as (2018 count − 2010 count) ÷ 2010 count, expressed as a percentage change. Both values are available in the file with no joins required.

**Reframe Justification:** The original Q28 asked whether a "threshold effect" exists above a certain population size, which requires per-capita normalization and therefore external population data. The reframed version replaces per-capita normalization with quartile segmentation using the absolute count itself as the grouping variable — this preserves the analytical intent (do the most-burdened countries progress more slowly?) while remaining fully answerable from the single file available.

---

## Q30 — WASH Infrastructure Floor for Communicable Disease Control

**Question:**
Is there a threshold level of basic sanitation and drinking water access below which
malaria and TB incidence remain persistently high, and above which they reliably
decline — and if so, where is that threshold?

**Datasets Used:**

| File | Metric | Period | Countries |
|---|---|---|---|
| `basicDrinkingWaterServices.csv` | Population using basic drinking water (%) | 2000–2017 | 195 |
| `atLeastBasicSanitizationServices.csv` | Population using basic sanitation (%) | 2000–2017 | 195 |
| `incedenceOfMalaria.csv` | Malaria incidence per 1,000 at-risk | 2000–2018 | 108 |
| `incedenceOfTuberculosis.csv` | TB incidence per 100,000/year | 2000–2019 | 195 |

**What It Answers:**
- Whether there is a non-linear relationship between WASH coverage and disease incidence — specifically a step-change pattern where disease burden only reliably declines above a critical access threshold
- The approximate threshold level (e.g., "below 60% basic sanitation, TB incidence shows no consistent decline")
- Whether the threshold, if it exists, differs between malaria and TB — which would indicate that the two diseases respond differently to WASH infrastructure improvements
- Whether drinking water access and sanitation access have independent effects or whether the relationship only holds when both are present above threshold simultaneously

**Answerability Justification:**
Drinking water and sanitation files are `float64`, 100% fill, 195 countries. Malaria is `float64`, 108 countries. TB requires string parsing of `First Tooltip` to extract the central estimate before use — this is a Phase 3 prerequisite for the TB portion of this question.

**Phase 3 Prerequisite:** The TB file's `First Tooltip` column stores values as uncertainty-range strings (e.g., `"183 [150–220]"`). The central estimate must be extracted by splitting on `[` and converting to float before this file can be used numerically. The malaria file has no such issue and can proceed directly to analysis.

**Scope Note:** The malaria portion of this analysis is limited to the 108 countries present in `incedenceOfMalaria.csv`. The TB portion covers 195 countries. Cross-disease comparison should acknowledge this asymmetry. The threshold analysis is correlational — a WASH access level below which disease burden persistently clusters does not establish that improving WASH causes disease reduction, only that the two are associated.

---

## Consolidated Dataset Reference

| File | Used In | Phase 3 Action Required |
|---|---|---|
| `HALElifeExpectancyAtBirth.csv` | Q2 | None |
| `lifeExpectancyAtBirth.csv` | Q2 | Scope to 2000–2019 only |
| `interventionAgianstNTDs.csv` | Q10, Q28 | None |
| `atLeastBasicSanitizationServices.csv` | Q13, Q30 | Filter `Dim1` deliberately |
| `safelySanitization.csv` | Q13 | Note 100-country scope |
| `population10SDG3.8.2.csv` | Q20, Q25 | Filter `Dim1 = Total` for national aggregates |
| `pharmacists.csv` | Q25 | Use latest year per country |
| `medicalDoctors.csv` | Q25 | Use latest year per country |
| `basicDrinkingWaterServices.csv` | Q30 | None |
| `incedenceOfMalaria.csv` | Q30 | None |
| `incedenceOfTuberculosis.csv` | Q30 | **Parse string column before use** |

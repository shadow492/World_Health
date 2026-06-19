# Phase 1: Ask & Define the Problem
## WHO World Health Statistics 2020 — Analytical Question Bank

> **Lifecycle Stage:** Phase 1 (Ask) — conducted after Phase 2 (Prepare/Collect).
> All questions below are scoped strictly to the 22 CSV files available in this project.

---

## Dataset Inventory

| Folder | File | Indicator | Time Range | Countries | Disaggregation |
|---|---|---|---|---|---|
| CommunicableDisease | hepatitusBsurfaceAntigen.csv | HBsAg prevalence in children under 5 (%) | 2015 only | 194 | None |
| CommunicableDisease | incedenceOfMalaria.csv | Malaria incidence per 1,000 at-risk | 2000–2018 | 108 | None |
| CommunicableDisease | incedenceOfTuberculosis.csv | TB incidence per 100,000/year | 2000–2019 | 195 | None |
| CommunicableDisease | interventionAgianstNTDs.csv | People requiring NTD interventions (count) | 2010–2018 | 195 | None |
| CommunicableDisease | newHivInfections.csv | New HIV infections per 1,000 uninfected | 2000–2019 | 171 | Sex (Male/Female/Both) |
| DrinkingWater | atLeastBasicSanitizationServices.csv | Population using basic sanitation (%) | 2000–2017 | 195 | Urban/Rural/Total |
| DrinkingWater | basicDrinkingWaterServices.csv | Population using basic drinking water (%) | 2000–2017 | 195 | None |
| DrinkingWater | basicHandWashing.csv | Population with handwashing facilities at home (%) | 2000–2017 | 97 | Urban/Rural/Total |
| DrinkingWater | safelySanitization.csv | Population using safe sanitation services (%) | 2000–2017 | 100 | Urban/Rural/Total |
| HealthWorkForce | dentists.csv | Dentists per 10,000 population | 1990–2019 | 191 | None |
| HealthWorkForce | medicalDoctors.csv | Medical doctors per 10,000 population | 1990–2018 | 194 | None |
| HealthWorkForce | nursingAndMidwife.csv | Nurses & midwives per 10,000 population | 1990–2018 | 194 | None |
| HealthWorkForce | pharmacists.csv | Pharmacists per 10,000 population | 1990–2018 | 185 | None |
| LifeExpectancy | HALElifeExpectancyAtBirth.csv | Healthy life expectancy (HALE) at birth, years | 2000–2019 | 184 | Sex (Male/Female/Both) |
| LifeExpectancy | HALeWHOregionLifeExpectancyAtBirth.csv | HALE & life expectancy at birth, by WHO region | 2000–2019 | 6 WHO regions | Sex (Male/Female/Both) |
| LifeExpectancy | WHOregionLifeExpectancyAtBirth.csv | Life expectancy at birth, by WHO region | 2000–2019 | 6 WHO regions | Sex (Male/Female/Both) |
| LifeExpectancy | lifeExpectancyAtBirth.csv | Life expectancy at birth, years | 1920–2019 | 184 | Sex (Male/Female/Both) |
| LifeExpectancy | ofHaleInLifeExpectancy.csv | HALE as % of life expectancy, by WHO region | 2000–2019 | 6 WHO regions | Sex (Male/Female/Both) |
| UHCincludingFinancialRisk | dataAvailibilityForUhc.csv | UHC data availability index | Single period | 183 | None |
| UHCincludingFinancialRisk | population10SDG3.8.2.csv | Population spending >10% of income on health (%) | 1985–2018 | 153 | Urban/Rural/Total |
| UHCincludingFinancialRisk | population25SDG3.8.2.csv | Population spending >25% of income on health (%) | 1985–2018 | 153 | Urban/Rural/Total |
| UHCincludingFinancialRisk | uhcCoverage.csv | UHC Service Coverage Index (0–100) | 2015 & 2017 | 183 | None |

---

## Known Data Constraints

These must be acknowledged before analysis begins; they affect how questions are framed and how conclusions can be drawn.

1. **No income or GDP data.** Wealth effects cannot be controlled for without joining to an external source (e.g., World Bank income classification). Many observed correlations may be confounded by income.
2. **Sparse temporal overlap.** Most cross-dataset joins are limited to the window 2010–2017. Hepatitis B is a single year (2015). UHC coverage is 2015/2017 only.
3. **Handwashing and safe sanitation cover only ~100 countries.** Analysis using these files is not globally representative and may skew toward countries that have data collection capacity.
4. **NTD file is a count (people needing interventions), not a rate.** Without population data, you cannot compute a per-capita burden. Absolute counts favour large-population countries.
5. **HIV infections include a `Dim1` column (sex disaggregation).** Queries must filter or group by `Dim1` deliberately; mixing all three values (Male, Female, Both) in the same aggregation will triple-count.
6. **TB incidence is stored as a string** (`First Tooltip` column, 1993 unique values). It likely encodes uncertainty ranges (e.g., `"183 [150-220]"`). This column must be parsed before numeric analysis.
7. **Life expectancy data extends to 1920**, but all other datasets begin at 2000 at the earliest. The historical stretch of `lifeExpectancyAtBirth.csv` can only be used for within-dataset longitudinal analysis.
8. **UHC data availability index** (`dataAvailibilityForUhc.csv`) is a metadata indicator, not a health outcome. It measures how much data a country has reported, not how good its healthcare is.

---

## Part A: Conventional Questions

These are the standard, well-established questions this class of dataset is known to support in academic and policy research.

### A1 — Life Expectancy & Healthy Life Expectancy

**Q1.** Which countries had the highest and lowest life expectancy at birth in 2019, and how does that ranking compare to 2000? Which countries made the largest gains in 19 years?

**Q2.** What is the global trend in healthy life expectancy (HALE) from 2000 to 2019? Has HALE improved faster or slower than total life expectancy over this period?

**Q3.** How large is the gender gap in life expectancy and HALE across WHO regions? Has the gap widened or narrowed between 2000 and 2019?

**Q4.** Using the `ofHaleInLifeExpectancy` file: what proportion of life is lived in full health per WHO region? Which region has the worst ratio of healthy years to total years lived?

**Q5.** Which WHO region showed the most improvement in life expectancy from 2000 to 2019, and which stagnated?

---

### A2 — Communicable Disease Burden

**Q6.** Which countries carry the highest malaria burden (incidence per 1,000 at-risk), and has that burden decreased consistently from 2000 to 2018, or are there countries where incidence reversed after an initial decline?

**Q7.** What is the global trend in TB incidence per 100,000 from 2000 to 2019? Are there countries where incidence is still rising rather than falling?

**Q8.** How do new HIV infection rates differ between males and females globally and by region? Which sex is bearing a disproportionately increasing share of new infections in recent years?

**Q9.** What is the distribution of Hepatitis B surface antigen prevalence in children under 5 across the 194 reported countries? Which WHO region has the highest burden?

**Q10.** How has the number of people requiring interventions for neglected tropical diseases (NTDs) changed from 2010 to 2018? Which countries account for the largest absolute NTD burden?

---

### A3 — WASH (Water, Sanitation & Hygiene)

**Q11.** What is the global trend in access to basic drinking water services from 2000 to 2017? Which regions are progressing fastest and slowest?

**Q12.** What is the urban-rural gap in access to basic sanitation services, and how has that gap changed from 2000 to 2017 across countries?

**Q13.** How do access to basic sanitation and access to safe (managed) sanitation differ at the country level? Which countries have high basic access but low safe access — indicating a "quality gap" in sanitation?

**Q14.** What percentage of the population has handwashing facilities at home, and how does this vary between urban and rural populations across the 97 covered countries?

---

### A4 — Healthcare Workforce

**Q15.** Which countries meet the WHO minimum threshold of 44.5 health workers (doctors + nurses/midwives) per 10,000 population, and which fall critically short?

**Q16.** How has medical doctor density evolved from 1990 to 2018 globally? Are low-density countries catching up, or is the gap with high-density countries widening?

**Q17.** What is the distribution of nursing and midwifery personnel across countries, and where are the largest shortfalls relative to WHO recommendations?

**Q18.** How does pharmacist density vary globally? Which regions are most underserved in pharmaceutical human resources?

---

### A5 — Universal Health Coverage & Financial Risk

**Q19.** What is the current distribution of UHC Service Coverage Index scores globally? How many countries are above versus below the SDG target threshold of 80?

**Q20.** What percentage of the population faces catastrophic health expenditure (>10% threshold) across countries, and how has this changed from 1985 to 2018?

**Q21.** How does the severity of financial hardship differ between the 10% threshold and the 25% threshold populations? Which countries have a large gap between the two, indicating a significant "deep poverty" health spending group?

**Q22.** Are urban or rural populations more likely to face catastrophic health expenditure? Does this urban-rural gap vary by region?

---

## Part B: Unconventional & Out-of-the-Box Questions

These questions move beyond standard descriptive and trend analysis. Each one requires either a non-obvious join, a derived metric, or a framing that is not found in the existing literature on this dataset.

---

**Q23. The Healthy Life Efficiency Question**

*"Which countries are most and least efficient at converting total life years into healthy life years?"*

By computing HALE ÷ Life Expectancy for every country-year in the period 2000–2019, you get a "healthy yield ratio" — a measure of healthcare quality that is independent of raw longevity. A country with LE = 75 and HALE = 70 (93.3% yield) is performing better in quality terms than a country with LE = 80 and HALE = 68 (85% yield). This surfaces countries that are keeping people alive but not well — a different and arguably more important failure mode than low life expectancy.

*Source files:* `HALElifeExpectancyAtBirth.csv` + `lifeExpectancyAtBirth.csv`

---

**Q24. The Stagnation Trap**

*"Which countries achieved fast gains in malaria or TB reduction between 2000–2010, but then stalled or reversed between 2010–2019?"*

Progress in communicable disease control is not always linear. A country can achieve large early gains (low-cost interventions, donor-funded campaigns) and then plateau once easy-to-reach populations are covered. Identifying these "stagnating improvers" is actionable: they signal where existing control strategies have hit a ceiling and a new approach is needed. Split the time series into two equal windows, compute average annual rate of change for each, and compare.

*Source files:* `incedenceOfMalaria.csv`, `incedenceOfTuberculosis.csv`

---

**Q25. Workforce Composition Imbalance as a Proxy for Healthcare Access Type**

*"What is the pharmacist-to-doctor ratio per country, and do countries with a high ratio also show higher catastrophic health expenditure?"*

A country with many pharmacists but few doctors may be one where the population primarily self-medicates via over-the-counter pharmacy purchases rather than accessing formal clinical care. This is not inherently bad, but it may drive up out-of-pocket health spending (pharmacies bill individuals directly; formal public healthcare systems can subsidize costs). Testing the pharmacist:doctor ratio against SDG 3.8.2 expenditure data would surface whether this informal-care-dependency pattern is associated with financial hardship.

*Source files:* `pharmacists.csv`, `medicalDoctors.csv`, `population10SDG3.8.2.csv`, `population25SDG3.8.2.csv`

---

**Q26. The Sanitation Equity Gap as a Disease Predictor**

*"Is the urban-rural gap in sanitation access a stronger predictor of communicable disease burden than the absolute level of national sanitation access?"*

The conventional analysis looks at the national average sanitation coverage and correlates it with disease outcomes. The unconventional question is whether inequality in sanitation access (urban % − rural %) is a stronger signal than the average itself. A country with 70% average coverage but a 60-point urban-rural gap is distributing that coverage very unequally — and the rural poor bear the full disease burden. This question requires deriving an equity gap metric and correlating it with malaria and TB incidence.

*Source files:* `atLeastBasicSanitizationServices.csv` (with urban/rural Dim1), `incedenceOfMalaria.csv`, `incedenceOfTuberculosis.csv`

---

**Q27. The UHC Coverage-Financial Protection Paradox**

*"Do countries with higher UHC Service Coverage Index scores consistently show lower rates of catastrophic health expenditure — or are there countries with high coverage but high financial hardship, suggesting that coverage alone is not sufficient for financial protection?"*

High service coverage (UHC SCI) and financial protection are supposed to move together, but they can decouple. A country may have broadly available formal healthcare that is, nonetheless, not free at the point of use. Testing this relationship will reveal whether UHC coverage is a reliable proxy for financial protection or whether they must be tracked independently. Look specifically for the "high SCI, high catastrophic expenditure" outlier group.

*Source files:* `uhcCoverage.csv`, `population10SDG3.8.2.csv`, `population25SDG3.8.2.csv`

---

**Q28. The NTD Burden Threshold Effect**

*"Is there a population size above which countries with a large NTD burden make proportionally less progress in reducing the number of people requiring interventions?"*

Very large burden countries (hundreds of millions of people needing treatment) face logistical and systemic challenges that smaller-burden countries do not. If a threshold effect exists — where progress slows dramatically beyond a certain absolute burden size — it would support the argument for prioritizing systemic healthcare reform over targeted disease-specific campaigns in the most burdened nations.

*Note: requires external population data to normalize the NTD count into a rate. Frame this as an analysis of absolute change in count over 2010–2018, segmented by burden quartile.*

*Source file:* `interventionAgianstNTDs.csv`

---

**Q29. Historical Life Expectancy Divergence — The Twin Country Analysis**

*"Which pairs of countries had near-identical life expectancy in the 1960s or 1970s but have since diverged dramatically by 2019? What does the post-divergence period reveal about their trajectories?"*

The `lifeExpectancyAtBirth.csv` file uniquely extends to 1920, allowing a long-run historical analysis impossible with most global health datasets. Countries that started at similar levels but diverged are natural quasi-experiments. Identifying these pairs (using both-sex combined figures) and examining the decade in which divergence began can surface hypotheses about what drove the difference — hypotheses that, while not testable within this dataset alone, can define the scope for future analysis.

*Source file:* `lifeExpectancyAtBirth.csv`

---

**Q30. The WASH Infrastructure Floor — Minimum Threshold for Communicable Disease Control**

*"Is there a threshold level of basic sanitation and drinking water access below which malaria and TB incidence remain persistently high, and above which they reliably decline — and if so, where is that threshold?"*

Rather than treating the sanitation-disease relationship as linear, this question asks whether there is a step-change threshold — a floor of WASH infrastructure below which gains in disease control are negligible regardless of medical interventions. If such a threshold exists (e.g., below 50% basic sanitation coverage, TB incidence shows no consistent decline), it would reframe the policy question from "how much sanitation helps" to "does sanitation need to reach a critical mass before it helps."

*Source files:* `basicDrinkingWaterServices.csv`, `atLeastBasicSanitizationServices.csv`, `incedenceOfMalaria.csv`, `incedenceOfTuberculosis.csv`

---

**Q31. Catastrophic Expenditure and Communicable Disease Burden — Are They Coupled?**

*"Do countries with higher NTD burden and higher HIV/malaria/TB incidence also have higher rates of catastrophic health expenditure, and has this relationship changed over time?"*

The hypothesis is that communicable disease burden and financial hardship co-occur — not just because both are products of poverty, but because treating and living with communicable diseases directly drives out-of-pocket health spending. Testing this with temporal data (overlapping window 2010–2017) allows you to ask whether improvements in disease burden are followed by reductions in financial hardship, or whether the two move independently — suggesting that disease-specific campaigns do not automatically produce financial protection.

*Source files:* `incedenceOfMalaria.csv`, `incedenceOfTuberculosis.csv`, `newHivInfections.csv`, `interventionAgianstNTDs.csv`, `population10SDG3.8.2.csv`

---

## Part C: Cross-Dataset Analysis Opportunities

The following pairs/groups of datasets have meaningful overlap in country coverage and time period, enabling joined analyses. These are the only valid joins available within this project's data alone.

| Join Type | Left Dataset | Right Dataset | Overlap Period | Joint Question |
|---|---|---|---|---|
| Workforce → UHC | medicalDoctors + nursingAndMidwife | uhcCoverage | 2015/2017 | Does workforce density predict UHC SCI? (Q15/Q19) |
| Workforce → Financial Risk | medicalDoctors + pharmacists | population10SDG3.8.2 | 1990–2018 | Does workforce composition predict financial hardship? (Q25) |
| WASH → Disease | basicDrinkingWaterServices + atLeastBasicSanitizationServices | incedenceOfMalaria + incedenceOfTuberculosis | 2000–2017 | Sanitation floor threshold (Q30) |
| HALE → Life Expectancy | HALElifeExpectancyAtBirth | lifeExpectancyAtBirth | 2000–2019 | Healthy yield ratio (Q23) |
| UHC → Financial Risk | uhcCoverage | population10/25SDG3.8.2 | 2015/2017 | UHC-financial protection paradox (Q27) |
| Disease → Financial Risk | incedenceOfMalaria + newHivInfections + interventionAgianstNTDs | population10SDG3.8.2 | 2010–2017 | Disease burden and catastrophic expenditure (Q31) |
| Sanitation Urban/Rural → Disease | atLeastBasicSanitizationServices (Dim1) | incedenceOfMalaria | 2000–2017 | Equity gap as disease predictor (Q26) |

---

## Part D: What This Dataset Cannot Answer

Understanding the boundaries of the data is as important as the questions it can answer. The following questions are out of scope and should not be attempted without supplementing with external data sources:

- **Cause-specific mortality.** No mortality data by cause of death is present. You cannot determine what diseases are killing people, only their incidence rates.
- **Income-stratified analysis.** There is no GDP, income class, or poverty rate column in any file. All observed correlations across country groups may be confounded by national wealth.
- **Policy or intervention attribution.** There is no data on when specific health policies were enacted, making causal attribution of any trend impossible within this dataset.
- **Vaccination coverage.** Hepatitis B surface antigen prevalence can be influenced by vaccination campaigns, but no vaccination rate data is present.
- **NTD treatment coverage.** The NTD file counts people *needing* interventions, not people who *received* them. Coverage rate (a critical metric) cannot be computed without external population denominators.
- **Urban vs. rural disease incidence.** Disease files (malaria, TB, HIV) have no urban/rural disaggregation, even though the WASH files do. The equity-gap analysis in Q26 is a correlation only, not a direct exposure-outcome link.
- **Healthcare quality or outcomes.** Workforce density (inputs) is available, but patient outcomes (outputs) are not. A country with 50 doctors per 10,000 may have poor health outcomes if those doctors are in one city.

---

## Summary: Priority Question List

The following 10 questions represent the core analytical agenda — a mix of foundational descriptive work, necessary for grounding, and the most analytically significant unconventional questions.

| Priority | Question | Type | Difficulty | Primary Files |
|---|---|---|---|---|
| 1 | Q2 — Global HALE vs. LE trend 2000–2019 | Conventional | Low | HALElifeExpectancyAtBirth, lifeExpectancyAtBirth |
| 2 | Q23 — Healthy life efficiency ratio | Unconventional | Low | HALElifeExpectancyAtBirth, lifeExpectancyAtBirth |
| 3 | Q6 — Malaria incidence trend & reversals | Conventional | Medium | incedenceOfMalaria |
| 4 | Q8 — HIV new infections by sex | Conventional | Low | newHivInfections |
| 5 | Q24 — Stagnation trap (malaria + TB) | Unconventional | Medium | incedenceOfMalaria, incedenceOfTuberculosis |
| 6 | Q12 — Urban-rural sanitation gap trends | Conventional | Low | atLeastBasicSanitizationServices |
| 7 | Q27 — UHC vs. financial protection paradox | Unconventional | Medium | uhcCoverage, population10/25SDG3.8.2 |
| 8 | Q15 — Workforce threshold vs. WHO minimum | Conventional | Medium | medicalDoctors, nursingAndMidwife |
| 9 | Q25 — Pharmacist:doctor ratio → expenditure | Unconventional | Medium | pharmacists, medicalDoctors, population10SDG3.8.2 |
| 10 | Q29 — Historical divergence twin-country analysis | Unconventional | High | lifeExpectancyAtBirth |

---

*This document defines the scope of Phase 1. All questions listed under Part A and Part B are answerable from the 22 files in this project. Questions in Part D require external data and are out of scope for this project.*

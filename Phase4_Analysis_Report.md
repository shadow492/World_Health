# Phase 4 — Analysis Report
## WHO World Health Statistics 2020

> **Lifecycle Stage:** Phase 4 (Analyze)
> This document records, question by question, what each analysis aimed to answer, every
> mathematical operation and statistical test applied, the datasets and any in-code
> transformations used, why the chosen approach is appropriate, how the resulting outputs
> answer the original objective, and the limitations specific to each question. It covers
> all 7 questions finalized in Phase 1 and built on the cleaned data from Phase 3.

---

## Overview

Phase 4 analysis was executed in Jupyter Notebook, loading the question-specific CSVs
exported from `phase3_outputs/` during Phase 3. Each question was treated as a
self-contained analytical unit: load → inspect → transform/derive → test → visualize →
extract country-level detail. Code was generated and run iteratively, with the person
pasting exact outputs/errors back for targeted fixes rather than full rewrites.

---

## Section 1 — Cross-Cutting Analytical Conventions

These conventions were established early and applied consistently across questions,
rather than being decided fresh each time.

**Unweighted country means for "global" figures.** No file in this dataset contains
population data, so every "global trend" is a simple average across countries, with
each country counted equally regardless of population size. This was applied in Q2,
Q10's global trend, Q13, and Q20. It means statements like "the global rate fell from
X to Y" describe the average country, not the average person.

**Period averages over single-year snapshots, where coverage is consistent.**
For country-level rankings, using the full-period average (e.g., 2000–2019 for Q2)
rather than one year reduces sensitivity to a single anomalous year and reflects a
sustained pattern. This was used for Q2's top-10 gap ranking and Q13's quadrant
analysis. Where reporting years are irregular across countries (Q20, Q25), a
latest-year-per-country snapshot was used instead, since forcing a single global year
would drop most non-reporting countries.

**`np.isfinite()` filtering before any statistical test.** Percentage-change metrics
computed from a zero or near-zero baseline produce `NaN` (0/0) or `inf` (nonzero/0).
This surfaced concretely in Q28, where 13 countries had a 2010 NTD baseline of zero.
Any correlation, group test, or aggregate statistic run without filtering these out
first will silently return `NaN` for the whole result (confirmed with `spearmanr`
during Q28 debugging) or distort means/medians with infinite values.

**Kruskal-Wallis over one-way ANOVA for skewed, small-baseline-driven distributions.**
`pct_change` distributions built from small baseline counts are heavily right-skewed
(a handful of countries can show >1000% increases off tiny bases). ANOVA assumes
roughly normal, equal-variance groups; Kruskal-Wallis is the rank-based, distribution-
free equivalent and was used instead for the Q28 quartile comparison.

**Spearman alongside Pearson wherever normality or linearity is uncertain.** Ratio
metrics (pharmacist:doctor ratio, NTD burden) are typically right-skewed rather than
normal, so Spearman (monotonic, rank-based) was computed alongside Pearson (linear)
in Q25 and Q28, rather than relying on Pearson alone.

**Explicit string `labels=` in `pd.qcut()`.** Calling `qcut()` without `labels=`
returns `pandas.Interval` objects as the category values, not strings. Any later code
that filters on a string label (e.g., `df["burden_q"] == "Q4_highest"`) will silently
return an empty DataFrame against Interval-typed categories. This was diagnosed and
fixed in Q28 by always passing explicit string labels.

---

## Section 2 — Question-by-Question Analysis

---

### Q2 — Global HALE vs. Life Expectancy Trend (2000–2019)

**Objective:** Determine the global trend in HALE and life expectancy from 2000–2019,
whether HALE has kept pace with total life expectancy, and which sex is driving any
divergence.

**Why this question was selected:** Identified in Phase 1 as the lowest-difficulty,
highest-priority question — both source files are fully numeric, cover identical
countries (184) and an overlapping period (2000–2019), and join cleanly on
`country + year + dim1` with no string parsing or external data required.

**Datasets used:** `HALElifeExpectancyAtBirth.csv` and `lifeExpectancyAtBirth.csv`
(scoped to 2000–2019 in Phase 3), inner-joined and exported as `q2_hale_vs_le.csv`
with a derived `hale_yield = hale / le` column. No further structural changes were
made in Phase 4; the only runtime check was confirming `dim1` actually contains
`"Both sexes"`, `"Male"`, `"Female"` before filtering on it.

**Operations, transformations, and tests:**
1. Filtered to `dim1 == "Both sexes"`, grouped by `year`, averaged `hale` and `le` →
   `global_trend`. Necessary to avoid triple-counting across sex categories and to
   produce a single global series per metric per year. Output: a 20-row year-indexed
   table of mean global HALE and LE, plotted as two overlaid line series.
2. `gap = le − hale` computed on `global_trend`. This operationalizes the core
   question directly — a widening gap means people are gaining years of life faster
   than years of *healthy* life. Output: a single time series plotted on its own axis.
3. Total change and CAGR (`(end/start)^(1/n) − 1`) computed for both HALE and LE at
   the start and end of the series. Total change alone doesn't account for the
   different absolute bases of HALE (~58 yrs) vs. LE (~66 yrs), so CAGR was added to
   give a normalized, directly comparable growth rate. Output: two printed growth-rate
   figures plus the gap value at the first and last year, giving a single quantitative
   answer to "faster or slower."
4. Sex-disaggregated version of the same gap calculation, grouped by `year` and
   `dim1` for Male/Female only. Necessary to isolate which sex is driving the global
   pattern, as required by the Phase 1 sub-question. Output: a two-line gap-over-time
   plot (Male vs. Female) plus printed per-sex total change in HALE, LE, and gap.
5. *(Added mid-session, beyond original Phase 1 scope)* Top-10 countries by average
   LE−HALE gap across 2000–2019 (average per country, not a single-year snapshot, per
   explicit decision to favor a sustained-pattern view), followed by a grouped bar
   chart of average HALE vs. average LE for those same 10 countries.

**How the outputs answer the objective:** The global trend and gap charts show
directly whether HALE is keeping pace with LE at the world level; the CAGR comparison
converts that visual impression into an unambiguous number rather than leaving "faster
or slower" to eyeballing two line slopes. The sex-disaggregated gap chart directly
answers the "which sex" sub-question by showing whether the male or female gap line
sits higher or is widening faster. The top-10 country extension moves the finding from
an abstract global statement to a concrete, actionable list of which specific
countries show the largest unhealthy-years burden — necessary for Phase 5/6 framing.

**Side notes:** The global trend is an unweighted country average, not a
population-weighted one (no population data exists in this project to weight by).
The `dim1` label format was a runtime assumption (verified, not guaranteed) since WHO
sometimes uses shorthand codes in other extracts.

---

### Q10 — NTD Intervention Burden Trend (2010–2018)

**Objective:** Determine how the global count of people requiring NTD interventions
changed from 2010–2018, and which countries account for the largest absolute burden
and the largest absolute movement (up or down).

**Why this question was selected:** The single source file is `int64`, 100% filled,
single-indicator, and self-contained — no joins or string parsing required, making it
the second-lowest-difficulty question and a natural pairing with Q28 (which reuses the
same 2010/2018 endpoint structure).

**Datasets used:** `interventionAgianstNTDs.csv`, reduced to `country`, `year`,
`value` in Phase 3 and exported as `q10_ntd_trend.csv`. No further changes in Phase 4
beyond column selection already done upstream.

**Operations, transformations, and tests:**
1. `groupby("year")["value"].sum()` → global yearly total, with `.diff()` and
   `.pct_change()` added for year-on-year absolute and percentage change. Sum (not
   mean) is correct here because the metric is an absolute count of people, not a
   rate — summing preserves the "total people needing intervention" interpretation.
   Output: a 9-row year-indexed table plus a line plot of the global burden trend.
2. Latest-year (2018) snapshot, with `pct_of_global = value / total_global * 100`
   computed per country, sorted to the top 15 by absolute count. A snapshot (not a
   period average) is correct for "current burden ranking," since the question asks
   which countries account for the burden now, not on average over the window.
   Output: a horizontal bar chart of the top 15 countries plus the cumulative % of
   global burden they represent.
3. Pivot of `year ∈ {2010, 2018}` to `count_2010`/`count_2018` per country (dropping
   countries missing either endpoint), with `abs_change` derived. This is the same
   endpoint-pivot structure later reused directly in Q28. Output: top-10 tables of
   largest absolute reductions and largest absolute increases, plotted as two
   side-by-side horizontal bar charts.

**How the outputs answer the objective:** The global yearly total with YoY change
directly answers "how has the burden changed" at the world level. The top-15 ranking
with % of global total answers "which countries account for the largest burden" in
the precise framing Phase 1 specified — absolute share, not per-capita comparison
(population data doesn't exist to support the latter). The largest-movers chart
answers the "biggest swings" sub-question by surfacing specific countries rather than
leaving it implicit in the global trend line.

**Side notes:** All figures are absolute counts, not rates — a country with a large
reduction in raw people-count is not necessarily improving faster than a smaller
country, since population size confounds every comparison here. This must be stated
explicitly alongside any ranking in the write-up.

---

### Q13 — Basic vs. Safe Sanitation Quality Gap (2000–2017)

**Objective:** Quantify the gap between basic and safe sanitation access at the
country level, identify the "high basic access, low safe access" quadrant, determine
whether the gap has closed or widened over time, and compare the gap between urban
and rural populations.

**Why this question was selected:** Both files are `float64`, fully filled, and carry
a `dim1` (Urban/Rural/Total) column, enabling the equity dimension directly. Flagged
as low-difficulty in Phase 1 despite the country-coverage mismatch between the two
source files.

**Datasets used:** `atLeastBasicSanitizationServices.csv` and `safelySanitization.csv`,
split by `dim1` in Phase 3 (`san_total`/`safe_total` for the national version,
Urban/Rural slices retained for the equity version), inner-joined on `country + year`
and exported as `q13_sanitation_gap.csv` and `q13_sanitation_gap_ur.csv` with a
derived `quality_gap = basic_pct − safe_pct`. No further structural changes in Phase 4.

**Operations, transformations, and tests:**
1. `groupby("year")` mean of `basic_pct`, `safe_pct`, `quality_gap` → global trend,
   plotted as two overlaid lines plus the gap as its own series. Necessary to see
   whether the average country's gap is closing or widening over the 2000–2017 window,
   read directly off the printed first-year vs. last-year gap values.
2. Per-country average (`groupby("country")` mean across all years) of basic, safe,
   and gap, plotted as a scatter of basic % vs. safe % with a basic=safe reference
   line. The scatter is the most direct visual test of the "quality gap" concept: any
   point far below the diagonal has a large quality gap, regardless of its absolute
   sanitation level.
3. Quadrant filter: countries with `basic_pct` at or above the median, sorted by
   `quality_gap` descending, top 10. Median was used as the "high basic access"
   threshold rather than a fixed cutoff like 80%, since it adapts to the actual
   distribution of the ~100-country sample rather than an arbitrary external number.
   Output: a horizontal bar chart of the top 10 "high basic, low safe" countries.
4. Urban vs. rural comparison: `groupby("dim1")` mean of basic/safe/gap on the
   urban/rural-retained dataset, plotted as a two-bar comparison. Directly answers the
   urban/rural sub-question from Phase 1.

**How the outputs answer the objective:** The global trend chart and printed gap
values answer "closing or widening" without ambiguity. The scatter plot is the
clearest visual representation of the core "quality gap" concept the question is
built around, since it shows the full distribution rather than just an average. The
quadrant ranking converts that visual concept into a concrete, ranked list of
countries — directly satisfying the question's explicit ask for which countries sit in
that quadrant. The urban/rural bar chart gives a direct, single-comparison answer to
whether residence type matters for this gap.

**Side notes:** This entire analysis covers only the ~100 countries present in
`safelySanitization.csv`; the other 95 countries in the basic-sanitation file cannot
be assessed for a quality gap at all and any "global" framing in the write-up must
state this explicitly. The median-based quadrant threshold is a methodological choice,
not a WHO-defined standard — a fixed cutoff (e.g., 80%) would produce a different (and
defensible) set of countries.

---

### Q20 — Catastrophic Health Expenditure Trend (1985–2018)

**Objective:** Determine the trend in catastrophic health expenditure (>10% of
household income) from 1985–2018, identify which countries currently face the highest
and lowest rates and which are changing fastest, compare urban vs. rural burden, and
determine whether countries are converging or diverging over time.

**Why this question was selected:** The single source file spans the longest time
range in the entire project (1985–2018) and is the only file capable of supporting a
genuine long-run trend question; `dim1` provides the urban/rural dimension directly.

**Datasets used:** `population10SDG3.8.2.csv`, split by `dim1` in Phase 3
(`pop10_total` for the national series, Urban/Rural concatenated for the equity
version), exported as `q20_catastrophic_exp.csv` and `q20_catastrophic_exp_ur.csv`
with `value` renamed to `catex_pct`. No further structural changes in Phase 4.

**Operations, transformations, and tests:**
1. `groupby("year")["country"].nunique()` → reporting coverage per year, checked
   *before* trusting any global trend statistic. Necessary because the 1985–2018
   window has known sparse early coverage (per Phase 1/3 documentation); any
   statement about the 1980s trend must be qualified by how few countries actually
   reported that year.
2. `groupby("year")["catex_pct"].mean()` → global trend, plotted as a single line.
   Same unweighted-average convention as Q2/Q10/Q13.
3. Latest-year-per-country snapshot (sort descending by year, drop duplicates keeping
   first) → used for "currently highest/lowest," consistent with how the irregular
   workforce files were handled in Phase 3. A single fixed global year would have
   dropped most non-reporting countries, so a snapshot is the only viable approach
   here. Output: top-10 highest and lowest tables, plus a bar chart of the highest 10.
4. Per-country first-vs-last available year change, rather than fixed 1985/2018
   endpoints, since most countries lack data at both literal endpoints. `annual_change
   = total_change / years_elapsed` normalizes for countries with different reporting
   windows before ranking "fastest changing." Output: top-10 fastest-increasing and
   fastest-decreasing tables.
5. `groupby("year")["catex_pct"].std()` → cross-country spread over time, plotted as
   a single line. This is the direct test for convergence vs. divergence: a falling
   standard deviation means countries are becoming more similar to each other; a
   rising one means they're pulling apart.
6. Urban vs. rural: `groupby(["year", "dim1"])` mean, plotted as two overlaid lines.

**How the outputs answer the objective:** The coverage check protects every later
claim from being undermined by a few-country early-period artifact. The global trend
line and snapshot rankings give the descriptive baseline the question asks for
directly. The first-vs-last annualized change isolates "changing fastest" in a way
that's fair across countries with different reporting histories — a raw total-change
ranking would unfairly favor countries with longer windows. The standard-deviation
plot is the most direct possible test of convergence/divergence: it answers that
specific sub-question with one number per year rather than requiring inference from
the overlaid country trends.

**Side notes:** Annualized change can still be noisy for countries with very few
reported years (a 2-year window produces a less reliable rate than a 20-year one);
this should be flagged for any country highlighted as a "fastest mover" with a short
`years_elapsed`. The convergence/divergence read is a spread metric only — it doesn't
indicate which direction (toward better or worse outcomes) the convergence is heading.

---

### Q25 — Pharmacist-to-Doctor Ratio as a Predictor of Financial Hardship

**Objective:** Compute the pharmacist-to-doctor ratio per country and test whether
countries with a high ratio also show higher catastrophic health expenditure.

**Why this question was selected:** The only three-file join in the project; despite
the added complexity, all three source files are fully numeric with a derivable ratio
metric, and the ~130–140 country intersection was judged sufficient in Phase 1 to
detect a pattern or its absence.

**Datasets used:** `pharmacists.csv` and `medicalDoctors.csv` (reduced to
latest-year-per-country snapshots in Phase 3), merged and joined to a latest-year
snapshot of `pop10_total`, exported as `q25_pharma_doctor_ratio.csv` with
`pharm_doc_ratio = pharm_per_10k / doc_per_10k` (zero-doctor denominators set to `NaN`
before division). In Phase 4, the `value` column was defensively re-renamed to
`catex_pct` if it hadn't already been renamed upstream, and rows with a missing ratio
or expenditure value were dropped before any correlation work.

**Operations, transformations, and tests:**
1. Scatter plot of `pharm_doc_ratio` vs. `catex_pct` across the cleaned country set —
   the direct visual test of the hypothesized relationship before computing any
   summary statistic.
2. Pearson **and** Spearman correlation between the two variables. Pearson alone
   would assume a linear relationship and normal-ish data; since the ratio is a
   right-skewed quantity (a few countries can have very high pharmacist:doctor
   ratios), Spearman was computed alongside it to capture any monotonic-but-nonlinear
   relationship Pearson might understate.
3. Median split into "High ratio" / "Low ratio" groups, with `groupby` mean/median/
   count of `catex_pct` per group, plotted as a two-bar comparison. This converts a
   continuous correlation into a directly interpretable group contrast — useful as a
   secondary check that the correlation isn't being driven entirely by a handful of
   extreme points.
4. Top-10 countries by ratio, plotted as a dual-axis bar+line chart (ratio as bars,
   expenditure as an overlaid line) to show both variables side-by-side at the
   country level rather than only as an aggregate correlation.

**How the outputs answer the objective:** The scatter and correlation coefficients
together give the core quantitative answer — whether a high ratio associates with
higher catastrophic expenditure, and how strong/linear that association is. Running
both Pearson and Spearman protects the conclusion from being an artifact of a
relationship that's monotonic but not linear (or vice versa). The median-split group
comparison is a robustness check in plain terms ("do high-ratio countries actually
look different on average") that a single correlation number can obscure. The top-10
dual-axis chart grounds the statistical finding in specific, nameable countries.

**Side notes:** This is the only question in the project without a time-series
version — all three source files were reduced to latest-year snapshots in Phase 3,
so the Phase 1 sub-question "does this relationship hold across time" is **not**
answerable from the data as constructed; this is a carried-forward scope limitation,
not an oversight. The workforce-ratio year and expenditure year may not match for a
given country, since each was taken from that file's own latest available year
independently. A high ratio alone does not confirm a self-medication-dependency
mechanism — the correlation is a proxy test, not a causal demonstration.

---

### Q28 — NTD Burden Quartile Progress Analysis (2010–2018)

**Objective:** Test whether countries in the highest 2010 NTD-burden quartile make
proportionally less progress reducing that count by 2018 than countries in lower
quartiles.

**Why this question was selected:** A reframed version of the original Phase 1 Q28
(which required external population data for a true threshold-effect test). The
reframe substitutes quartile segmentation on the absolute 2010 baseline for per-capita
normalization, preserving the analytical intent — "do the most-burdened countries
progress more slowly?" — while remaining fully answerable from the single NTD file.

**Datasets used:** `interventionAgianstNTDs.csv`, pivoted to 2010/2018 endpoints in
Phase 3 with `abs_change`, `pct_change`, and `burden_q` (quartile of `count_2010`,
labelled `Q1_lowest`–`Q4_highest`) derived, exported as `q28_ntd_quartile.csv`. In
Phase 4, two issues surfaced and were fixed in-code: (1) `pct_change` was `inf` or
`NaN` for 13 countries with a zero 2010 baseline, handled with an explicit
`np.isfinite()` flag column rather than dropping rows silently; (2) a defensive check
was added to force `burden_q` back to string labels via `pd.qcut(..., labels=[...])`
if it were ever loaded as raw `Interval` objects (confirmed via dtype/value inspection
not to be the case in the final run, but kept as a guard).

**Operations, transformations, and tests:**
1. `np.isfinite()` flag on `pct_change`, confirmed against the actual data: 10
   countries with `NaN` (0/0, e.g. Andorra, Belarus, Canada, Denmark, Switzerland) and
   3 with `inf` (a nonzero 2018 count appearing from a zero 2010 baseline — Croatia,
   Monaco, Turkmenistan). All 13 are genuine zero/near-zero-burden countries, not a
   data-quality artifact, and were excluded from quartile statistics rather than
   imputed, since there is no meaningful "% change" from a true zero.
2. Median/IQR summary table per quartile (replacing an initially-tried boxplot,
   which was dominated by the small number of extreme `pct_change` outliers from
   near-zero baselines and obscured the typical pattern within each quartile).
   `groupby("burden_q")["pct_change"].agg(median, q25, q75, n)`.
3. Error-bar plot of median `pct_change` ± IQR per quartile (Q1_lowest → Q4_highest).
   This is the direct visual test of the core hypothesis: if Q4_highest's median sits
   closer to zero (less negative) than Q1_lowest's, that's visual evidence of slower
   proportional progress among the most-burdened countries.
4. Kruskal-Wallis test across the four quartile groups on `pct_change` (finite values
   only). Chosen over one-way ANOVA because the `pct_change` distributions are
   heavily skewed by the small-baseline-count effect even after removing
   non-finite rows.
5. Spearman correlation between `count_2010` (continuous baseline) and `pct_change`,
   on the finite subset. This tests the same hypothesis without the information loss
   of binning into quartiles — a negative rho would mean larger 2010 burden associates
   with less negative (i.e., slower) proportional progress.
6. Per-quartile outlier extraction: within each `burden_q` group, the 3 fastest
   improvers (most negative `pct_change`) and 3 worst performers (highest
   `pct_change`), surfaced as specific country tables.

**How the outputs answer the objective:** The median/IQR table and error-bar plot
give a direct, quartile-by-quartile read of whether progress slows as 2010 baseline
burden rises — exactly what the question asks. The Kruskal-Wallis test adds a formal
significance check on top of the visual pattern, appropriate given the confirmed
skew in the underlying distributions. The Spearman correlation provides a second,
non-binned test of the same relationship, guarding against an artifact of quartile
boundary placement. The per-quartile outlier tables surface the specific countries
that drive or contradict the overall pattern (e.g., a high-burden country making fast
progress despite its quartile, or a low-burden country stagnating) — directly
satisfying the Phase 1 ask for outlier identification.

**Side notes:** 13 of 193 pivoted countries (≈6.7%) were excluded from all quartile
statistics due to a zero 2010 baseline; this should be stated as a deliberate
exclusion-by-construction in the write-up, not glossed over as missing data. The
`burden_q` interval/label issue did not actually manifest in this run (`burden_q`
loaded correctly as string labels from `phase3_outputs`), but the defensive fix
remains in the code in case the dataset is regenerated from scratch with a fresh
`qcut()` call that omits explicit labels.

---

### Q30 — WASH Infrastructure Floor for Communicable Disease Control (2000–2017)

**Objective:** Determine whether there is a threshold level of basic sanitation and
drinking water access below which malaria and TB incidence remain persistently high,
and above which they reliably decline.

**Why this question was selected:** Tests whether the sanitation–disease relationship
is a step-change/threshold effect rather than a simple linear association — a
materially different policy implication ("reach critical mass" vs. "more is always
better") that the conventional correlation-only framing in Part A of the question bank
doesn't address.

**Datasets used:** `basicDrinkingWaterServices.csv` and
`atLeastBasicSanitizationServices.csv` (Total slice) joined to `incedenceOfMalaria.csv`
and `incedenceOfTuberculosis.csv` (post string-parse) separately in Phase 3, producing
`q30_wash_malaria.csv` (~108 countries) and `q30_wash_tb.csv` (195 countries). Verified
in Phase 4: both files load with zero nulls across `water_pct`, `sanitation_pct`, and
the respective disease-incidence column, with country/year counts exactly matching
the Phase 3 documentation (108 countries / 2000–2017 for malaria; 195 countries /
2000–2017 for TB).

**Operations, transformations, and tests:**
1. Collapsed each file from a country-year panel to one row per country via
   `groupby("country").mean()` across the full window — a period average, consistent
   with the project's convention of preferring averages over snapshots where coverage
   is consistent across the window (confirmed: zero nulls, full 2000–2017 range for
   both files).
2. Spearman correlation between each WASH variable (water, sanitation) and each
   disease variable (malaria, TB) — four combinations total. This is the baseline
   monotonicity check run *before* any threshold search: if there's no monotonic
   relationship at all, a threshold-effect search is unlikely to be meaningful.
3. Decile binning (`pd.cut` into 10 equal-width bins from 0–100%) of each WASH
   variable, with median ± IQR of the corresponding disease incidence computed per
   bin and plotted as four error-bar charts. This is the direct visual test for a
   step-change: a linear relationship produces a smoothly declining median across
   bins, while a true threshold effect produces a flat (high) median across the lower
   bins followed by a sharp drop at some point.
4. Threshold grid-search: for each candidate split point (10% increments), a
   Mann-Whitney U test compares the disease-incidence distribution below the
   threshold against above it. Mann-Whitney (rather than a t-test) was used as the
   two-group, distribution-free analog to the Kruskal-Wallis convention already
   established for Q28, appropriate given disease-incidence data is not expected to
   be normally distributed. The threshold with the lowest p-value across the grid is
   the candidate "floor."
5. Quadrant analysis: countries split into High/Low water and High/Low sanitation via
   median split (independently for each), with median disease incidence computed for
   each of the four water×sanitation combinations. This tests whether water and
   sanitation have independent effects or only matter jointly — directly answering
   the Phase 1 sub-question about whether the two WASH metrics interact.

**How the outputs answer the objective:** The baseline Spearman check establishes
whether a relationship exists at all before any threshold claim is made. The decile
bin plots are the primary visual evidence for or against a step-change pattern, since
they show the full shape of the relationship rather than a single summary
correlation. The Mann-Whitney grid-search converts that visual impression into a
specific, defensible threshold value with a formal significance test behind it,
directly answering "where is that threshold." The quadrant analysis addresses whether
the threshold (if found) applies to water and sanitation independently or only when
both are satisfied together — a distinction the question explicitly asks about.

**Side notes:** The malaria analysis runs on ~108 countries vs. 195 for TB — the
known scope asymmetry documented since Phase 1/3 — so any cross-disease comparison of
where the threshold falls must state this difference in coverage explicitly. The
`min_group=10` floor in the grid-search may produce unreliable p-values at extreme
thresholds (e.g., 10% or 90%) for the smaller malaria sample if either side of the
split falls below that minimum; group sizes (`n_below`/`n_above`) should be checked
alongside the p-value before reporting a "best" threshold. As with Q26 in the
original Phase 1 question bank, this entire analysis is correlational — it identifies
where disease incidence and WASH coverage co-occur, not that improving WASH access
causes the decline.

---

## Section 3 — Limitations Carried Forward to Phase 5

These are consolidated from the per-question side notes above, since several recur
across multiple questions and should be stated once, prominently, in any written
findings document:

- **No population data anywhere in the project.** Every "global" figure (Q2, Q10,
  Q13, Q20) is an unweighted average across countries, not a population-weighted
  measure of the average person's experience. Every absolute-count ranking (Q10, Q28)
  reflects population size as a confound, not pure burden severity.
- **Country coverage varies by question** — ~100 for Q13, ~108 for Q30's malaria
  side, ~130–140 for Q25, 153 for Q20, 195 for the rest — and no claim of global
  coverage should be made without restating the actual N for that specific analysis.
- **All snapshot-based analyses (Q20, Q25) carry a year-mismatch risk** between the
  variables being compared, since each was taken from its own file's latest available
  year independently rather than a forced common year.
- **All relationships found (Q13, Q25, Q26-style WASH analysis in Q30, Q28) are
  correlational.** No file in the project contains policy timing, intervention
  coverage, or causal-mechanism data, so none of these findings support a causal claim
  on their own.

---

## Section 4 — Phase 4 Output Summary

| Question | Primary Test/Method | Key Output Files (input) |
|---|---|---|
| Q2 | Global trend, CAGR, sex-disaggregated gap | `q2_hale_vs_le.csv` |
| Q10 | Global sum trend, snapshot ranking, endpoint pivot | `q10_ntd_trend.csv` |
| Q13 | Scatter vs. diagonal, median-split quadrant, urban/rural | `q13_sanitation_gap.csv`, `q13_sanitation_gap_ur.csv` |
| Q20 | Coverage check, snapshot ranking, annualized change, std-dev spread | `q20_catastrophic_exp.csv`, `q20_catastrophic_exp_ur.csv` |
| Q25 | Pearson + Spearman, median-split groups | `q25_pharma_doctor_ratio.csv` |
| Q28 | isfinite filter, median/IQR, Kruskal-Wallis, Spearman | `q28_ntd_quartile.csv` |
| Q30 | Spearman, decile bins, Mann-Whitney threshold grid, quadrants | `q30_wash_malaria.csv`, `q30_wash_tb.csv` |

All 7 finalized Phase 1 questions have now been carried through Phase 4. The next
step is Phase 5 (Share): consolidating these findings, charts, and stated limitations
into a written report.

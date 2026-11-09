---
name: longitudinal-microbiome-differential-abundance-analysis
description: Use when analyzing longitudinal (repeated-measures) microbiome or other
  high-dimensional count/continuous/binary/ordinal feature data where abundance change
  over time (treatment proxy) must be disentangled from multiple covariates such as
  medications.
when_to_use_negative:
- When the data are cross-sectional (no repeated time points or before/after structure),
  since LongDat tests change over time (proxy for treatments)
- When only one or zero covariates exist and a simple paired model suffices — LongDat's
  key advantage appears 'especially when there were multiple covariates'
- When the input is raw sequencing reads rather than a feature/abundance table with
  sample metadata (use a profiler like MetaPhlAn/HUMAnN first)
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: LongDat
  role: 'Primary R package: covariate-sensitive longitudinal analysis of high-dimensional
    (e.g. microbiome) data; install via install.packages(''LongDat'')'
  repo: https://github.com/CCY-dev/LongDat
- name: longdat_cont()
  role: Longitudinal analysis pipeline function for continuous time variables; run
    on the master table with data_type = 'count' and test_var = 'Day'
  repo: https://github.com/CCY-dev/LongDat
- name: longdat_disc()
  role: Longitudinal analysis pipeline function for discrete time variables (e.g.
    Time_point, before/after treatment)
  repo: https://github.com/CCY-dev/LongDat
- name: cuneiform_plot()
  role: Visualization of the longdat result table
  repo: https://github.com/CCY-dev/LongDat
- name: MaAsLin2
  role: Comparator tool for reanalysis with and without medication covariates; dropped
    from 27 to 2 significant species with covariates in the fasting MetS cohort due
    to model errors with multiple fixed effects
  repo: https://github.com/biobakery/biobakery
- name: ANCOM
  role: Comparator tool benchmarked against LongDat on simulated and real data
  repo: https://github.com/FrederickHuangLin/ANCOM-Code-Archive
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad063
    title: 'LongDat: an R package for covariate-sensitive longitudinal analysis of
      high-dimensional data'
  source_content_hash: sha256:7d90712e3650fec8c13ca3c68a41a3565be19240fbda3fdb71f2074150ff694c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCY-dev/LongDat
  tools:
  - ANCOM
  - LongDat
  - MaAsLin2
  - R
  - cuneiform_plot()
  - longdat_cont()
  - longdat_disc()
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# longitudinal-microbiome-differential-abundance-analysis

## Summary

A covariate-sensitive longitudinal differential abundance workflow built on the LongDat R package (longdat_cont()/longdat_disc()), which tests whether high-dimensional features (e.g. species-level gut microbiome abundances) change significantly over time while detecting and controlling for covariates, and benchmarks results against MaAsLin2, ANCOM, lgpr and ZIBR.

## When to use

Use when analyzing longitudinal (repeated-measures) microbiome or other high-dimensional count/continuous/binary/ordinal feature data where abundance change over time (treatment proxy) must be disentangled from multiple covariates such as medications.

## When NOT to use

- When the data are cross-sectional (no repeated time points or before/after structure), since LongDat tests change over time (proxy for treatments)
- When only one or zero covariates exist and a simple paired model suffices — LongDat's key advantage appears 'especially when there were multiple covariates'
- When the input is raw sequencing reads rather than a feature/abundance table with sample metadata (use a profiler like MetaPhlAn/HUMAnN first)

## Inputs

- Species-level (feature) abundance table (count data) from a longitudinal cohort, e.g. Maifeld et al. 2021 fasting MetS cohort
- Sample metadata table including time variable (continuous, e.g. Day) and covariates (e.g. medication use)
- Master table built from metadata + feature tables via make_master_table()

## Outputs

- LongDat result table with p values, Signal column, and EffectSize per feature
- Covariate table listing features whose time effect is reducible to covariates
- Randomized control table (when data_type = 'count')
- Cuneiform plot of the result table via cuneiform_plot()
- Comparison table of significant species counts per tool/condition, e.g. mets_significant_species_comparison.csv

## How to apply

Assemble metadata and feature tables into a master table with make_master_table(), then run longdat_cont() with data_type = 'count' and a continuous test_var (e.g. 'Day'), setting a seed first because data_type = 'count' triggers a randomized control test that must be reproducible. For discrete time points (e.g. before/after treatment), use longdat_disc() with data_type = 'count' and a discrete test_var. Inspect the Signal column of the result table (e.g. 'OK_nc' = significant change through time with no potential covariate; 'OK_nrc' = OK and not reducible to covariate; 'NS' = no time effect) and the effect-size column for increasing/decreasing patterns, then check the covariate table to see which time effects are reducible to covariates. Because LongDat loops over one covariate per model, it handles many fixed effects robustly, unlike MaAsLin2, whose models for most species returned errors with multiple fixed effects in the fasting MetS cohort reanalysis (dropping from 27 to only 2 significant species). Visualize the result table with cuneiform_plot() and export comparison counts (e.g. mets_significant_species_comparison.csv) for tool-by-condition tabulation.

## Related tools

- **LongDat** (Primary R package: covariate-sensitive longitudinal analysis of high-dimensional (e.g. microbiome) data; install via install.packages('LongDat')) — https://github.com/CCY-dev/LongDat
- **longdat_cont()** (Longitudinal analysis pipeline function for continuous time variables; run on the master table with data_type = 'count' and test_var = 'Day') — https://github.com/CCY-dev/LongDat
- **longdat_disc()** (Longitudinal analysis pipeline function for discrete time variables (e.g. Time_point, before/after treatment)) — https://github.com/CCY-dev/LongDat
- **cuneiform_plot()** (Visualization of the longdat result table) — https://github.com/CCY-dev/LongDat
- **MaAsLin2** (Comparator tool for reanalysis with and without medication covariates; dropped from 27 to 2 significant species with covariates in the fasting MetS cohort due to model errors with multiple fixed effects) — https://github.com/biobakery/biobakery
- **ANCOM** (Comparator tool benchmarked against LongDat on simulated and real data) — https://github.com/FrederickHuangLin/ANCOM-Code-Archive

## Examples

```
test_cont <- longdat_cont(input = LongDat_cont_master_table, data_type = "count", test_var = "Day")
```

## Evaluation signals

- Result table contains a Signal column for every feature with expected values (e.g. 'OK_nc', 'OK_nrc', 'NS'); 'OK_nc' indicates a significant time effect with no potential covariate
- For data_type = 'count', a randomized control table is produced and is reproducible only if a seed was set before running
- EffectSize column shows plausible increasing/decreasing patterns (e.g. BacteriumA -0.78 decreasing, BacteriumC 0.71 increasing in the dummy example)
- Features whose Signal shows covariate-driven time effects must appear in the covariate table; blank covariate table implies no covariate effect found
- Benchmark check: significant-species counts per tool per condition should target the reported 27 species per tool without covariates (19 overlapping) and MaAsLin2's drop to only 2 with covariates (Fig. 5, Supplementary Tables S2–S5)

## Limitations

- When data_type equals 'count', forgetting to set a seed makes the randomized control test non-reproducible
- Comparator MaAsLin2 returned errors for most species when multiple fixed effects (e.g. medication covariates) were included, so covariate-adjusted comparisons undercount significant species
- LongDat is designed for longitudinal datasets testing change over time as a proxy for treatments; it is not a general cross-sectional differential abundance tool
- The archived standalone ANCOM functions are no longer maintained; users are directed to ANCOMBC

## Evidence

- [abstract] LongDat analyzes longitudinal multivariable cohort data while accounting for covariates: "We introduce LongDat, an R package that analyzes longitudinal multivariable (cohort) data while simultaneously accounting for a potentially large number of covariates."
- [intro] Set seed for reproducible randomized control test with count data: "When data_type equals to "count", please remember to set seed (as shown below) so that you'll get reproducible randomized control test."
- [results] Check covariate table for covariate effects in results: "If there is covariate effect in the result, please see the covariate table to find out what the covariates are."
- [readme] Output table contains p values, effect sizes and covariates per feature: "The output table contains p values, effect sizes and covariates of each feature, making the downstream analysis easy."

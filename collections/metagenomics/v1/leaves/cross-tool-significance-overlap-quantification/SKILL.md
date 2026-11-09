---
name: cross-tool-significance-overlap-quantification
description: Decision support for choosing between longitudinal differential-abundance
  tools when multiple covariates (e.g. medications) may confound a time/treatment
  effect in high-dimensional microbiome data.
when_to_use_negative:
- Data are cross-sectional with no time or repeated-measure structure — LongDat's
  longitudinal pipelines (longdat_cont/longdat_disc) and this comparison assume longitudinal
  design
- You only need a single tool's differential-abundance output rather than a tool-vs-tool
  concordance assessment
- The feature table is not count data and you cannot supply the matching data_type;
  LongDat requires the correct data_type (count, proportion, binary, ordinal, continuous)
  for valid signal calls
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: LongDat
  role: Covariate-sensitive longitudinal differential-abundance tool; provides the
    Signal column and covariate table used as one side of the comparison and disentangles
    time effects from medication covariates
  repo: https://github.com/CCY-dev/LongDat
- name: MaAsLin2
  role: Comparison tool; run on the same abundance and metadata tables with and without
    covariates to record significant species and expose model failures with multiple
    fixed effects
  repo: https://github.com/biobakery/biobakery
- name: longdat_cont()
  role: LongDat pipeline function used with data_type = "count" and a continuous time
    test variable for both covariate-free and covariate-adjusted runs
  repo: https://github.com/CCY-dev/LongDat
- name: R
  role: Language environment in which make_master_table(), longdat_cont() and MaAsLin2
    are executed and results are tabulated to CSV
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad063
    title: 'LongDat: an R package for covariate-sensitive longitudinal analysis of
      high-dimensional data'
  source_content_hash: sha256:d93914ed99b1737497d16c847ad24a10dfb08a14204947fa3deb97bef4e6a221
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCY-dev/LongDat
  tools:
  - LongDat
  - MaAsLin2
  - R
  - longdat_cont()
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# cross-tool-significance-overlap-quantification

## Summary

A skill for benchmarking differential-abundance calls by running two longitudinal tools (LongDat and MaAsLin2) on the same species-level abundance table and metadata, with and without medication covariates, then tabulating significant species counts per tool per condition and quantifying their overlap.

## When to use

Decision support for choosing between longitudinal differential-abundance tools when multiple covariates (e.g. medications) may confound a time/treatment effect in high-dimensional microbiome data.

## When NOT to use

- Data are cross-sectional with no time or repeated-measure structure — LongDat's longitudinal pipelines (longdat_cont/longdat_disc) and this comparison assume longitudinal design
- You only need a single tool's differential-abundance output rather than a tool-vs-tool concordance assessment
- The feature table is not count data and you cannot supply the matching data_type; LongDat requires the correct data_type (count, proportion, binary, ordinal, continuous) for valid signal calls

## Inputs

- Species-level gut microbial abundance table from a longitudinal cohort (e.g. Maifeld et al. 2021 fasting MetS cohort)
- Sample metadata including time variable and medication covariates
- Master table assembled from metadata and feature tables via make_master_table()

## Outputs

- Significant species lists per tool (LongDat, MaAsLin2) per condition (with/without covariates)
- Tabulated significant species counts per tool per condition with overlap quantification
- mets_significant_species_comparison.csv containing summary counts and species lists
- LongDat result table with Signal/EffectSize columns plus covariate table

## How to apply

Assemble the species-level abundance table and sample metadata (here from the Maifeld et al. 2021 fasting MetS cohort) into a master table with make_master_table(). Run longdat_cont() with data_type = "count" and the continuous time test variable, setting a seed first so the randomized control test is reproducible; run it once without medication covariates and once with them, reading the Signal column ('OK_nc', 'OK_nrc', 'NS') and the covariate table to see which time-associated species are reducible to covariates. Run MaAsLin2 on the same tables, once without and once with medication covariates, recording significant species at the study's significance threshold. Tabulate significant species counts per tool per condition and quantify overlap — the reference result is 27 significant species per tool without covariates (19 overlapping) and MaAsLin2's drop to only 2 species with covariates because its models error with multiple fixed effects, whereas LongDat loops over one covariate per model and handles multiple covariates robustly (Fig. 5, Supplementary Tables S2-S5). Write the summary counts and species lists to mets_significant_species_comparison.csv.

## Related tools

- **LongDat** (Covariate-sensitive longitudinal differential-abundance tool; provides the Signal column and covariate table used as one side of the comparison and disentangles time effects from medication covariates) — https://github.com/CCY-dev/LongDat
- **MaAsLin2** (Comparison tool; run on the same abundance and metadata tables with and without covariates to record significant species and expose model failures with multiple fixed effects) — https://github.com/biobakery/biobakery
- **longdat_cont()** (LongDat pipeline function used with data_type = "count" and a continuous time test variable for both covariate-free and covariate-adjusted runs) — https://github.com/CCY-dev/LongDat
- **R** (Language environment in which make_master_table(), longdat_cont() and MaAsLin2 are executed and results are tabulated to CSV)

## Examples

```
set.seed(1); test_cont <- longdat_cont(input = mets_master_table, data_type = "count", test_var = "Day")
```

## Evaluation signals

- Without covariates, each tool reports 27 significant species with 19 overlapping — a large deviation in this baseline suggests data assembly or threshold errors
- With medication covariates included, MaAsLin2 should drop to only 2 significant species because most per-species models error with multiple fixed effects; LongDat should instead return a covariate-disentangled result, so equal counts for both tools here indicate the covariate runs were misconfigured
- LongDat result table Signal values are 'OK_nc'/'OK_nrc'/'NS' for each species, and time-associated species reducible to covariates must appear in the covariate table
- Set a seed before each longdat_cont() run with data_type = "count"; re-running with the same seed must reproduce the randomized control test and identical significance calls
- mets_significant_species_comparison.csv must contain per-tool, per-condition counts and species lists matching Supplementary Tables S2-S5

## Limitations

- MaAsLin2 models for most species return errors when given multiple fixed effects (medication covariates), causing under-reporting of significant species rather than a true biological result
- LongDat with data_type = "count" involves a randomized control test, so results are only reproducible if a seed is set before running
- Benchmark findings are grounded in the fasting MetS cohort reanalysis of Maifeld et al. 2021; counts (27/19/2) and Supplementary Tables S2-S5 are cohort-specific and may not transfer to other datasets

## Evidence

- [abstract] LongDat outperforms competitors with multiple covariates: "We showed that LongDat outperformed these tools in accuracy, runtime and memory cost, especially when there were multiple covariates."
- [intro] Set seed for reproducible randomized control test with count data: "When data_type equals to "count", please remember to set seed (as shown below) so that you'll get reproducible randomized control test."
- [results] Interpret Signal column and covariate table: "If there is covariate effect in the result, please see the covariate table to find out what the covariates are."
- [readme] Output table contents enable downstream cross-tool comparison: "The output table contains p values, effect sizes and covariates of each feature, making the downstream analysis easy."
- [abstract] MaAsLin2 comparison context from the article's benchmarking statement: "We tested and compared LongDat with other tools (i.e. MaAsLin2, ANCOM, lgpr and ZIBR) on both simulated and real data."

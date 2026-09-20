---
name: medication-covariate-adjustment
description: Use when reanalyzing longitudinal gut-microbiome data (e.g. the Maifeld
  et al. 2021 fasting MetS cohort) where medication use is a confounder and the goal
  is to identify which time-associated species are reducible to medication covariates.
when_to_use_negative:
- The dataset is cross-sectional with no longitudinal (time/treatment) structure —
  LongDat takes longitudinal datasets as input and tests for change of features over
  time.
- The design has no covariates (e.g. medications) to disentangle, so a single time-effect
  model without covariate adjustment suffices.
- The abundance data is not count data and a discrete test variable applies — longdat_disc()
  with the appropriate data_type is the correct pipeline instead.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3318
tools:
- name: LongDat
  role: 'Primary pipeline: longdat_cont() with data_type = ''count'' loops over one
    covariate per model, robustly handling multiple medication covariates and flagging
    whether time effects are reducible to covariates'
  repo: https://github.com/CCY-dev/LongDat
- name: MaAsLin2
  role: Comparison tool run with and without medication covariates; with multiple
    fixed effects its models for most species return errors, dropping reported significant
    species to only 2
- name: R
  role: Execution environment for make_master_table(), longdat_cont(), set.seed for
    reproducibility, and tabulating significant species per tool per condition
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad063
    title: 'LongDat: an R package for covariate-sensitive longitudinal analysis of
      high-dimensional data'
  source_content_hash: sha256:76dcb367c8aa989b769b9cd4f4b9cf34462b892567ce2fa47deb595765ac99aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCY-dev/LongDat
  tools:
  - LongDat
  - MaAsLin2
  - R
  - longdat_cont()
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# medication-covariate-adjustment

## Summary

A skill for disentangling medication-covariate effects from intervention/time effects in longitudinal microbiome cohorts, using LongDat's longdat_cont() covariate-looping approach (which handles multiple covariates robustly) rather than fitting MaAsLin2 models with multiple fixed effects, which fail with errors for most species.

## When to use

Use when reanalyzing longitudinal gut-microbiome data (e.g. the Maifeld et al. 2021 fasting MetS cohort) where medication use is a confounder and the goal is to identify which time-associated species are reducible to medication covariates.

## When NOT to use

- The dataset is cross-sectional with no longitudinal (time/treatment) structure — LongDat takes longitudinal datasets as input and tests for change of features over time.
- The design has no covariates (e.g. medications) to disentangle, so a single time-effect model without covariate adjustment suffices.
- The abundance data is not count data and a discrete test variable applies — longdat_disc() with the appropriate data_type is the correct pipeline instead.

## Inputs

- Species-level gut microbial abundance table (count data) from the fasting MetS cohort (Maifeld et al., 2021)
- Sample metadata including time variable (continuous, e.g. days) and medication covariates
- Master table created from metadata and feature tables via make_master_table()

## Outputs

- LongDat result table with Signal, p values, and EffectSize per feature
- LongDat covariate table listing covariate effects per feature (blank when none found)
- Randomized control table (produced when data_type = 'count')
- Comparison summary, e.g. mets_significant_species_comparison.csv with significant species counts per tool per condition and overlap

## How to apply

Assemble the species-level abundance table and sample metadata into a master table with make_master_table(), then run longdat_cont() with data_type = 'count' and a continuous time test variable (e.g. Day), setting a seed first since count data triggers a randomized control test. Run longdat_cont() a second time including medication covariates; LongDat loops over one covariate per model, avoiding the model-fitting errors MaAsLin2 encounters with multiple fixed effects. Inspect the Signal column of the result table (e.g. 'OK_nrc' = time effect not reducible to covariate; 'OK_nc' = significant change over time with no potential covariate) and check the covariate table to identify which time-associated species are explained by medications. Run MaAsLin2 on the same abundance and metadata tables with and without medication covariates and tabulate significant species per tool per condition to quantify the contrast (reference: 27 significant species per tool without covariates, 19 overlapping; MaAsLin2 drops to only 2 species with covariates while LongDat retains a robust covariate-disentangled result, Fig. 5, Supplementary Tables S2-S5). Write summary counts and species lists to a comparison CSV such as mets_significant_species_comparison.csv.

## Related tools

- **LongDat** (Primary pipeline: longdat_cont() with data_type = 'count' loops over one covariate per model, robustly handling multiple medication covariates and flagging whether time effects are reducible to covariates) — https://github.com/CCY-dev/LongDat
- **MaAsLin2** (Comparison tool run with and without medication covariates; with multiple fixed effects its models for most species return errors, dropping reported significant species to only 2)
- **R** (Execution environment for make_master_table(), longdat_cont(), set.seed for reproducibility, and tabulating significant species per tool per condition)

## Examples

```
set.seed(1); test_cont <- longdat_cont(input = mets_master_table, data_type = "count", test_var = "Day")
```

## Evaluation signals

- Signal column of the longdat_cont result table distinguishes time effects: 'OK_nrc' (significant and not reducible to covariate) versus 'OK_nc' (significant change through time, no potential covariate), with NS for no effect of time.
- The covariate table is non-blank exactly when covariate effects are detected; it should be blank when no covariate effect is found.
- A randomized control table is produced for data_type = 'count' and must be reproducible because a seed was set before running.
- Benchmark counts reproduce the article's results: 27 significant species per tool without covariates (19 overlapping) and MaAsLin2 dropping to 2 species with covariates versus LongDat's robust covariate-disentangled result (Fig. 5, Supplementary Tables S2-S5).
- mets_significant_species_comparison.csv is written with summary counts and species lists per tool per condition.

## Limitations

- When models include multiple fixed effects, MaAsLin2 returns errors for most species in the fasting MetS cohort, reporting only 2 significant species with covariates — a fragile contrast to LongDat's one-covariate-per-model looping.
- When data_type equals 'count', a seed must be set or the randomized control test is not reproducible.
- If dependency/namespace errors occur during installation (e.g. object not exported by a dependency namespace), install the dependency package first before loading LongDat.

## Evidence

- [abstract] LongDat accounts for a potentially large number of covariates in longitudinal cohort data: "We introduce LongDat, an R package that analyzes longitudinal multivariable (cohort) data while simultaneously accounting for a potentially large number of covariates."
- [intro] LongDat detects and controls for covariates while testing change over time: "analyzes if there is significant change of the features over time (proxy for treatments), while detects and controls for covariates at the same time"
- [results] Covariate table reveals which covariates explain a feature's signal: "If there is covariate effect in the result, please see the covariate table to find out what the covariates are."
- [intro] Seed must be set for count data so the randomized control test is reproducible: "When data_type equals to "count", please remember to set seed (as shown below) so that you'll get reproducible randomized control test."
- [readme] Covariate-sensitive output table supports downstream medication-adjusted analysis: "The output table contains p values, effect sizes and covariates of each feature, making the downstream analysis easy."

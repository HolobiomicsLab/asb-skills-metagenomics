---
name: continuous-time-treatment-effect-detection
description: Use when the input is a longitudinal master table (count data) with a
  continuous time variable (e.g. Day) and the goal is to identify features whose abundance
  changes significantly over time while detecting covariate effects.
when_to_use_negative:
- The time variable is discrete (e.g. before/after treatment time points) — use longdat_disc()
  instead, which reports signals like 'OK_nrc'
- The data are cross-sectional with no time dimension — a longitudinal pipeline is
  not applicable
- Input is still separate metadata and feature tables that have not been combined
  — first create the master table with make_master_table()
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: LongDat
  role: R package providing longdat_cont(), make_master_table(), and cuneiform_plot()
    for the continuous-time longitudinal pipeline
  repo: https://github.com/CCY-dev/LongDat
- name: R
  role: Runtime environment; install with install.packages('LongDat')
- name: MaAsLin2
  role: Comparison tool run in negative binomial (NEGBIN) mode in the benchmark of
    covariate-free simulations
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad063
    title: 'LongDat: an R package for covariate-sensitive longitudinal analysis of
      high-dimensional data'
  source_content_hash: sha256:d8c3aa30d92a72fc778bcfc354ab798370010ac7d8f76ccadc2a2d975a900563
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCY-dev/LongDat
  tools:
  - LongDat
  - MaAsLin2
  - R
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# continuous-time-treatment-effect-detection

## Summary

Detect whether features change significantly over time (time as a continuous treatment proxy) in longitudinal high-dimensional count data using longdat_cont() from the LongDat R package, while simultaneously controlling for covariates. It outputs p-values, effect sizes, and a covariate table per feature.

## When to use

Use when the input is a longitudinal master table (count data) with a continuous time variable (e.g. Day) and the goal is to identify features whose abundance changes significantly over time while detecting covariate effects.

## When NOT to use

- The time variable is discrete (e.g. before/after treatment time points) — use longdat_disc() instead, which reports signals like 'OK_nrc'
- The data are cross-sectional with no time dimension — a longitudinal pipeline is not applicable
- Input is still separate metadata and feature tables that have not been combined — first create the master table with make_master_table()

## Inputs

- LongDat_cont_master_table (data frame combining metadata and feature tables, e.g. 2 time points day 0 and 7)
- LongDat_cont_metadata_table (metadata with the continuous time variable)
- LongDat_cont_feature_table (high-dimensional feature/abundance table)
- data_type = 'count' (also supports proportion, binary, ordinal, continuous)

## Outputs

- Result table with per-feature p values, Signal (e.g. OK_nc, OK_nrc, NS), and EffectSize columns
- Covariate table (blank when no covariate effect is found)
- Randomized control table (for count data_type)
- Optional cuneiform_plot() visualization of the result table

## How to apply

Build a master table from metadata and feature tables with make_master_table(), then call longdat_cont(input = master_table, data_type = 'count', test_var = '<continuous time column>'). Because data_type equals 'count', set a seed before running so the randomized control test is reproducible. The function returns a result table with a Signal column (e.g. 'OK_nc' for significant time effect with no covariate, 'NS' for no effect), an EffectSize column (e.g. -0.78 decreasing, 0.71 increasing), and a covariate table that is blank when no covariate effect is found. Inspect the Signal column to classify features and check the covariate table for covariate effects; optionally visualize with cuneiform_plot().

## Related tools

- **LongDat** (R package providing longdat_cont(), make_master_table(), and cuneiform_plot() for the continuous-time longitudinal pipeline) — https://github.com/CCY-dev/LongDat
- **R** (Runtime environment; install with install.packages('LongDat'))
- **MaAsLin2** (Comparison tool run in negative binomial (NEGBIN) mode in the benchmark of covariate-free simulations)

## Examples

```
test_cont <- longdat_cont(input = LongDat_cont_master_table, data_type = "count", test_var = "Day", seed = 123)
```

## Evaluation signals

- Result table contains a Signal column where significant features show 'OK_nc' and non-significant ones show 'NS'
- Covariate table is blank when no covariate effect is present (expected in covariate-free simulations)
- When data_type = 'count', re-running with the same seed reproduces the same randomized control table
- EffectSize column carries interpretable signed values (e.g. -0.78 decreasing, 0.71 increasing patterns)
- Benchmark invariants: accuracy in the 0.9–1 range and LongDat's FDR median of zero across all tested effect sizes and sample sizes (10–300)

## Limitations

- When data_type equals 'count', a seed must be set because the randomized control test is randomized; forgetting it makes results non-reproducible
- If a result is reducible to a covariate, the time effect may be flagged as covariate-driven and must be interpreted via the covariate table
- If package installation fails with namespace export errors (e.g. object 'A' not exported by namespace:B_package), install the dependency B_package first
- LongDat's advantage over MaAsLin2/ANCOM/lgpr/ZIBR is largest when there are multiple covariates; on covariate-free simulated data MaAsLin2 attains higher TPR at small sample sizes at the cost of higher FDR

## Evidence

- [intro] LongDat R package takes longitudinal dataset as input data and analyzes if there is significant change of the features over time: "LongDat R package takes longitudinal dataset as input data and analyzes if there is significant change of the features over time"
- [intro] When data_type equals to "count", please remember to set seed (as shown below) so that you'll get reproducible randomized control test.: "When data_type equals to "count", please remember to set seed (as shown below) so that you'll get reproducible randomized control test."
- [results] BacteriumA and BacteriumC have decreasing and increasing patterns, respectively. From the next column "EffectSize", we know that the effect sizes are -0.78 and 0.71, respectively.: "BacteriumA and BacteriumC have decreasing and increasing patterns, respectively. From the next column "EffectSize", we know that the effect sizes are -0.78 and 0.71, respectively."
- [results] Since there is no covariate effect found in this example (according to the result table), the covariate table is blank.: "Since there is no covariate effect found in this example (according to the result table), the covariate table is blank."
- [intro] The function make_master_table() helps you to create master table from metadata and feature tables.: "The function make_master_table() helps you to create master table from metadata and feature tables."
- [readme] Install LongDat by typing install.packages("LongDat") in R.: [evidence span withheld — anchor status: not_found]

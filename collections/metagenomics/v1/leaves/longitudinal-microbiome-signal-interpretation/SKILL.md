---
name: longitudinal-microbiome-signal-interpretation
description: Use when you have longitudinal multivariable data (count, proportion,
  binary, ordinal or continuous) with a time/treatment variable and potentially many
  covariates, and need covariate-sensitive biomarker detection.
when_to_use_negative:
- When the input is separate metadata and feature tables rather than a combined master
  table — use make_master_table() first
- When the goal is cross-sectional differential abundance with no longitudinal/time
  structure — tools like ANCOM or MaAsLin2 address that setting instead
- When the analysis must reproduce a static result and data_type = 'count' without
  setting a seed, since the randomized control test is stochastic
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: LongDat
  role: Core R package providing make_master_table(), longdat_cont(), longdat_disc()
    and cuneiform_plot() for covariate-sensitive longitudinal analysis
  repo: https://github.com/CCY-dev/LongDat
- name: R
  role: Execution environment; LongDat is installed via install.packages('LongDat')
    and run as an R library
- name: MaAsLin2
  role: Benchmarked comparison tool for longitudinal multivariable analysis
- name: ANCOM
  role: Benchmarked comparison tool for differential abundance with covariates
  repo: https://github.com/FrederickHuangLin/ANCOM-Code-Archive
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad063
    title: 'LongDat: an R package for covariate-sensitive longitudinal analysis of
      high-dimensional data'
  source_content_hash: sha256:be175b61e42f76d56ddd5108e01a2c183381862ffb3eb106cd21dea5520deb79
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCY-dev/LongDat
  tools:
  - ANCOM
  - LongDat
  - MaAsLin2
  - R
  - longdat_cont()
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# longitudinal-microbiome-signal-interpretation

## Summary

Run the LongDat three-stage longitudinal pipeline (longdat_cont() or longdat_disc()) to test whether features change significantly over time while classifying each feature's time signal as reducible, entangled with, or independent of covariates, and interpret the Signal and EffectSize columns.

## When to use

Use when you have longitudinal multivariable data (count, proportion, binary, ordinal or continuous) with a time/treatment variable and potentially many covariates, and need covariate-sensitive biomarker detection.

## When NOT to use

- When the input is separate metadata and feature tables rather than a combined master table — use make_master_table() first
- When the goal is cross-sectional differential abundance with no longitudinal/time structure — tools like ANCOM or MaAsLin2 address that setting instead
- When the analysis must reproduce a static result and data_type = 'count' without setting a seed, since the randomized control test is stochastic

## Inputs

- Master table (e.g. LongDat_cont_master_table / LongDat_disc_master_table combining metadata and feature tables)
- data_type parameter ('count', 'proportion', 'binary', 'ordinal' or 'continuous')
- test_var parameter naming the time variable ('Day' for continuous, 'Time_point' for discrete)
- Random seed for reproducibility when data_type = 'count'

## Outputs

- Result table with p values, Signal column ('OK_nc', 'OK_nrc', 'NS') and EffectSize (Spearman's rho) per feature
- Covariate table listing covariate effects (blank when none found)
- Randomized control table (for count data) or Normalize_method table (for measurement/other data types)
- Optional CSV exports (longdat_cont_result_table.csv, longdat_cont_covariate_table.csv) and cuneiform_plot() visualization

## How to apply

Build a master table with make_master_table() from the metadata and feature tables, then call longdat_cont() for a continuous time variable (e.g. test_var = 'Day') or longdat_disc() for a discrete one (e.g. test_var = 'Time_point'), setting a seed when data_type = 'count' so the randomized control test is reproducible. The pipeline first fits a GLMM null time model, then loops over selected covariates one at a time, each added alone as a fixed effect with the time variable to classify the time association as reducible to a covariate, entangled, or not reducible. For continuous time, a non-parametric effect-size step computes Spearman's rho per feature. Read the Signal column of the result table (e.g. 'OK_nc' = significant time effect with no covariate, 'OK_nrc' = significant and not reducible to covariate, 'NS' = no significant effect) and check the covariate table for covariate effects; optionally visualize with cuneiform_plot().

## Related tools

- **LongDat** (Core R package providing make_master_table(), longdat_cont(), longdat_disc() and cuneiform_plot() for covariate-sensitive longitudinal analysis) — https://github.com/CCY-dev/LongDat
- **R** (Execution environment; LongDat is installed via install.packages('LongDat') and run as an R library)
- **MaAsLin2** (Benchmarked comparison tool for longitudinal multivariable analysis)
- **ANCOM** (Benchmarked comparison tool for differential abundance with covariates) — https://github.com/FrederickHuangLin/ANCOM-Code-Archive

## Examples

```
set.seed(123); test_cont <- longdat_cont(input = LongDat_cont_master_table, data_type = "count", test_var = "Day")
```

## Evaluation signals

- Signal column values are interpretable: BacteriumA and BacteriumC show 'OK_nc' in the worked longdat_cont example while BacteriumB shows 'NS'
- EffectSize column shows the expected direction: BacteriumA decreases over time with effect size -0.78 and BacteriumC increases with 0.71 (Spearman's rho)
- In the example, no covariate effect was found, so the covariate table should be blank
- With data_type = 'count', the randomized control test output is reproducible across runs only when a seed is set
- In the longdat_disc example, BacteriumA shows 'OK_nrc', meaning its time effect is independent of potential covariates

## Limitations

- When data_type = 'count', the randomized control test requires a seed for reproducibility; without it results vary between runs
- Package installation can fail with namespace export errors ('object A is not exported by namespace:B_package'), fixed by installing the dependency package first
- LongDat focuses on longitudinal microbiome data but is extended to binary, categorical and continuous data; cross-sectional-only designs are out of scope

## Evidence

- [readme] LongDat R package takes longitudinal dataset as input data and analyzes if there is significant change of the features over time (proxy for treatments), while detects and controls for covariates at the same time.: "LongDat R package takes longitudinal dataset as input data and analyzes if there is significant change of the features over time (proxy for treatments), while detects and controls for covariates"
- [intro] When data_type equals to "count", please remember to set seed (as shown below) so that you'll get reproducible randomized control test.: "When data_type equals to "count", please remember to set seed (as shown below) so that you'll get reproducible randomized control test."
- [intro] The function make_master_table() helps you to create master table from metadata and feature tables.: "The function make_master_table() helps you to create master table from metadata and feature tables."
- [results] treatment is a significant predictor for BacteriumA as it shows "OK_nrc" (which represents "OK and not reducible to covariate"): "treatment is a significant predictor for BacteriumA as it shows "OK_nrc" (which represents "OK and not reducible to covariate")"
- [results] BacteriumA and BacteriumC have decreasing and increasing patterns, respectively. From the next column "EffectSize", we know that the effect sizes are -0.78 and 0.71, respectively.: "BacteriumA and BacteriumC have decreasing and increasing patterns, respectively. From the next column "EffectSize", we know that the effect sizes are -0.78 and 0.71"
- [results] Since there is no covariate effect found in this example (according to the result table), the covariate table is blank.: "Since there is no covariate effect found in this example (according to the result table), the covariate table is blank."

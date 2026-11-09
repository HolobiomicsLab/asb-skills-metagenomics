---
name: longitudinal-glmm-null-model-testing
description: Use when a longitudinal dataset (count, proportion, binary, ordinal or
  continuous features over time points) must be tested for significant change of features
  over time (proxy for treatments) while simultaneously detecting and controlling
  for potentially many covariates.
when_to_use_negative:
- The analysis is cross-sectional or has no repeated measures over time — LongDat's
  GLMM null time model requires longitudinal data with time points (e.g. day 0 and
  7)
- A competing tool is preferred for non-longitudinal differential abundance; the article
  benchmarked LongDat against MaAsLin2, ANCOM, lgpr and ZIBR and found LongDat advantageous
  mainly for longitudinal data with multiple covariates
- The feature table needs compositional zero preprocessing/structural-zero detection
  as in ANCOM's feature_table_pre_process() — that is a different tool's workflow,
  not LongDat's
edam_operation: null
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3174
tools:
- name: LongDat
  role: R package providing longdat_cont()/longdat_disc(), make_master_table() and
    cuneiform_plot() for the null-time-model, covariate model test and effect-size
    pipeline
  repo: https://github.com/CCY-dev/LongDat
- name: R
  role: Runtime environment; install via install.packages("LongDat") and set seed
    before count-data runs
- name: ANCOM
  role: Benchmarked alternative; its archived README documents longitudinal lme-based
    analysis with covariates via ANCOM(feature_table, meta_data, struc_zero, main_var,
    adj_formula, rand_formula)
  repo: https://github.com/FrederickHuangLin/ANCOM-Code-Archive
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad063
    title: 'LongDat: an R package for covariate-sensitive longitudinal analysis of
      high-dimensional data'
  source_content_hash: sha256:1fefca8a6f2cfa88713efcac9ca63bb168c60abb0c78158883cd0563aa12d8a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCY-dev/LongDat
  tools:
  - ANCOM
  - LongDat
  - R
  - longdat_cont()
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# longitudinal-glmm-null-model-testing

## Summary

Three-stage LongDat longitudinal pipeline (longdat_cont/longdat_disc) that first fits a GLMM null time model per feature, then loops covariates one at a time to classify the time effect as reducible/entangled/not-reducible, and finally computes non-parametric effect sizes (Spearman's rho for continuous time) with covariate-sensitive signals like 'OK_nc' and 'NS'.

## When to use

Use when a longitudinal dataset (count, proportion, binary, ordinal or continuous features over time points) must be tested for significant change of features over time (proxy for treatments) while simultaneously detecting and controlling for potentially many covariates.

## When NOT to use

- The analysis is cross-sectional or has no repeated measures over time — LongDat's GLMM null time model requires longitudinal data with time points (e.g. day 0 and 7)
- A competing tool is preferred for non-longitudinal differential abundance; the article benchmarked LongDat against MaAsLin2, ANCOM, lgpr and ZIBR and found LongDat advantageous mainly for longitudinal data with multiple covariates
- The feature table needs compositional zero preprocessing/structural-zero detection as in ANCOM's feature_table_pre_process() — that is a different tool's workflow, not LongDat's

## Inputs

- LongDat master table data frame (e.g. LongDat_cont_master_table / LongDat_disc_master_table) combining metadata and feature tables
- data_type string ('count', 'proportion', 'binary', 'ordinal' or 'continuous')
- test_var column name of the time variable (e.g. 'Day' for continuous, 'Time_point' for discrete)

## Outputs

- Result table (longdat_cont_result_table.csv) with p values, Signal ('OK_nc', 'OK_nrc', 'NS'), and EffectSize (e.g. Spearman's rho) per feature
- Covariate table (longdat_cont_covariate_table.csv) listing covariate effects, blank when none found
- Randomized control table (when data_type = 'count') or a Normalize_method table (for measurement/others data)
- Optional cuneiform_plot() visualization of the result table

## How to apply

First build a master table from metadata and feature tables with make_master_table(), then set a random seed because the randomized control test is not reproducible otherwise when data_type equals 'count'. Call longdat_cont() (continuous time variable such as 'Day') or longdat_disc() (discrete time variable such as 'Time_point') with data_type = 'count'; the pipeline fits GLMM null time models per feature, then runs the covariate model test in which each selected covariate is added alone as a fixed effect together with the time variable to examine whether the time association can be reduced to that covariate's influence, and finally computes effect sizes (Spearman's rho for continuous time). Read the Signal column of the result table: 'OK_nc' marks a significant time effect with no covariate, 'OK_nrc' marks OK and not reducible to covariate, and 'NS' marks no significant effect; effect sizes (e.g. -0.78 for BacteriumA decreasing, 0.71 for BacteriumC increasing) give direction. Check the covariate table for covariate effects (blank when none found), optionally visualize with cuneiform_plot(), and export longdat_cont_result_table.csv and longdat_cont_covariate_table.csv.

## Related tools

- **LongDat** (R package providing longdat_cont()/longdat_disc(), make_master_table() and cuneiform_plot() for the null-time-model, covariate model test and effect-size pipeline) — https://github.com/CCY-dev/LongDat
- **R** (Runtime environment; install via install.packages("LongDat") and set seed before count-data runs)
- **ANCOM** (Benchmarked alternative; its archived README documents longitudinal lme-based analysis with covariates via ANCOM(feature_table, meta_data, struc_zero, main_var, adj_formula, rand_formula)) — https://github.com/FrederickHuangLin/ANCOM-Code-Archive

## Examples

```
set.seed(123); test_cont <- longdat_cont(input = LongDat_cont_master_table, data_type = "count", test_var = "Day")
```

## Evaluation signals

- BacteriumA and BacteriumC show 'OK_nc' in the Signal column with EffectSize -0.78 and 0.71 respectively, while BacteriumB shows 'NS' on the example LongDat_cont_master_table
- The covariate table is blank when no covariate effect was found in the result table
- The randomized control test is reproducible across runs only when a seed is set before calling with data_type = 'count'
- Output tables are written as longdat_cont_result_table.csv and longdat_cont_covariate_table.csv and can be plotted with cuneiform_plot()
- Signal values conform to the documented vocabulary: 'OK_nc' (significant, no covariate), 'OK_nrc' (OK and not reducible to covariate), 'NS' (no significant effect)

## Limitations

- When data_type equals 'count', a seed must be set or the randomized control test is not reproducible
- Dependency installation can fail with namespace export errors (e.g. object not exported by a B_package namespace); the dependency must be installed before LongDat
- LongDat focuses on longitudinal microbiome data; extension to binary, categorical and continuous data types is supported but the pipeline's strength is multiple-covariate longitudinal settings

## Evidence

- [methods] covariate model test loops over covariates one at a time: each selected covariate is added alone as a fixed effect together with the time variable in GLMMs to examine whether the time association can be reduced to that covariate's influence.: [evidence span withheld — anchor status: not_found]
- [results] signal OK_nc and NS interpretation with effect sizes: "BacteriumA and BacteriumC have decreasing and increasing patterns, respectively. From the next column "EffectSize", we know that the effect sizes are -0.78 and 0.71, respectively."
- [intro] seed requirement for count data: "When data_type equals to "count", please remember to set seed (as shown below) so that you'll get reproducible randomized control test."
- [results] Signal vocabulary OK_nrc: "treatment is a significant predictor for BacteriumA as it shows "OK_nrc" (which represents "OK and not reducible to covariate")"
- [readme] README introduction of package scope: "LongDat R package takes longitudinal dataset as input data and analyzes if there is significant change of the features over time (proxy for treatments), while detects and controls for covariates at"

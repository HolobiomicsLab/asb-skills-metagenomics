---
name: nonparametric-effect-size-estimation
description: Use when analyzing longitudinal high-dimensional data with a continuous
  time variable (e.g. 'Day') and you need per-feature effect sizes and direction (increase/decrease
  over time) in addition to significance testing.
when_to_use_negative:
- The time variable is discrete/categorical (e.g. before/after treatment Time_point)
  — use the longdat_disc() pipeline instead
- The goal is cross-sectional differential abundance rather than longitudinal change
  over time — tools like ANCOM or MaAsLin2 fit that setting
- Input is separate metadata and feature tables that have not yet been combined —
  first build a master table with make_master_table()
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3174
tools:
- name: LongDat
  role: R package providing longdat_cont() whose pipeline computes Spearman's rho
    effect sizes for continuous time along with GLMM significance testing
  repo: https://github.com/CCY-dev/LongDat
- name: R
  role: runtime environment in which LongDat is installed and executed
- name: cuneiform_plot()
  role: visualization of the result table produced by the pipeline
  repo: https://github.com/CCY-dev/LongDat
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad063
    title: 'LongDat: an R package for covariate-sensitive longitudinal analysis of
      high-dimensional data'
  source_content_hash: sha256:a56c16ffeb2d9b04723fc7b9897b6582753275bb242f501475ad7954b3e85b5f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCY-dev/LongDat
  tools:
  - LongDat
  - R
  - cuneiform_plot()
  - longdat_cont()
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# nonparametric-effect-size-estimation

## Summary

Estimate direction and magnitude of longitudinal feature changes using non-parametric effect sizes (Spearman's rho for continuous time) as implemented in the LongDat R package's longdat_cont() pipeline, alongside p values and covariate annotations.

## When to use

Use when analyzing longitudinal high-dimensional data with a continuous time variable (e.g. 'Day') and you need per-feature effect sizes and direction (increase/decrease over time) in addition to significance testing.

## When NOT to use

- The time variable is discrete/categorical (e.g. before/after treatment Time_point) — use the longdat_disc() pipeline instead
- The goal is cross-sectional differential abundance rather than longitudinal change over time — tools like ANCOM or MaAsLin2 fit that setting
- Input is separate metadata and feature tables that have not yet been combined — first build a master table with make_master_table()

## Inputs

- LongDat master table data frame (e.g. LongDat_cont_master_table) combining metadata and feature tables
- data_type specification ('count', 'proportion', 'binary', 'ordinal' or 'continuous')
- test_var: name of the continuous time variable (e.g. 'Day')

## Outputs

- Result table with Signal, p values, and EffectSize columns per feature (written to longdat_cont_result_table.csv)
- Covariate table listing covariate effects, blank when none found (written to longdat_cont_covariate_table.csv)
- Randomized control table when data_type = 'count'

## How to apply

Run the LongDat GLMM pipeline on a master table (built with make_master_table() from metadata and feature tables) by calling longdat_cont(input = master_table, data_type = 'count', test_var = 'Day'); set a random seed first because data_type = 'count' triggers a randomized control test requiring reproducibility. After the null time model and the per-covariate model loop (which classifies time effects as reducible, entangled, or not reducible to covariates), the pipeline computes non-parametric effect sizes — Spearman's rho for continuous time — for each feature. Read the result table's EffectSize column: in the worked example BacteriumA has effect size -0.78 (decreasing over time) and BacteriumC has 0.71 (increasing), while BacteriumB shows 'NS' (no significant time effect). Cross-check the Signal column ('OK_nc' = significant time effect with no covariate, 'NS' = no effect) and optionally visualize with cuneiform_plot().

## Related tools

- **LongDat** (R package providing longdat_cont() whose pipeline computes Spearman's rho effect sizes for continuous time along with GLMM significance testing) — https://github.com/CCY-dev/LongDat
- **R** (runtime environment in which LongDat is installed and executed)
- **cuneiform_plot()** (visualization of the result table produced by the pipeline) — https://github.com/CCY-dev/LongDat

## Examples

```
set.seed(123); test_cont <- longdat_cont(input = LongDat_cont_master_table, data_type = "count", test_var = "Day")
```

## Evaluation signals

- Result table Signal column: significant features carry 'OK_nc' (significant time effect, no covariate) and non-significant ones 'NS'
- EffectSize column populated with non-parametric effect sizes (Spearman's rho for continuous time); sign indicates direction, e.g. -0.78 for BacteriumA (decreasing) and 0.71 for BacteriumC (increasing) in the example
- Randomized control test is reproducible across runs because a seed was set when data_type = 'count'
- Covariate table is blank when no covariate effect is detected, matching the Signal values in the result table
- Output CSVs (longdat_cont_result_table.csv, longdat_cont_covariate_table.csv) are written with the expected per-feature schema

## Limitations

- When data_type = 'count', results depend on a randomized control test, so a seed must be set for reproducibility
- The Signal semantics differ between longdat_cont and longdat_disc (e.g. 'OK_nrc' appears for discrete pipelines), so interpretations are not interchangeable
- Effect sizes are interpreted relative to covariate-test outcomes: a significant time effect may be reducible to a covariate rather than independent

## Evidence

- [results] From the next column "EffectSize", we know that the effect sizes are -0.78 and 0.71, respectively.: "BacteriumA and BacteriumC have decreasing and increasing patterns, respectively. From the next column "EffectSize", we know that the effect sizes are -0.78 and 0.71, respectively."
- [methods] each selected covariate is added alone as a fixed effect together with the time variable in GLMMs: [evidence span withheld — anchor status: not_found]
- [readme] When data_type equals to "count", please remember to set seed (as shown below) so that you'll get reproducible randomized control test.: "When data_type equals to "count", please remember to set seed (as shown below) so that you'll get reproducible randomized control test."
- [results] treatment is a significant predictor for BacteriumA and BacteriumC as they show "OK_nc": "treatment is a significant predictor for BacteriumA and BacteriumC as they show "OK_nc""
- [results] Since there is no covariate effect found in this example (according to the result table), the covariate table is blank.: "Since there is no covariate effect found in this example (according to the result table), the covariate table is blank."

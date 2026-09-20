---
name: covariate-reducibility-classification
description: Use when analyzing longitudinal high-dimensional data (e.g. gut microbial
  abundance over Day or Time_point) where a significant time effect on a feature must
  be checked against potentially confounding covariates before being declared a treatment
  effect.
when_to_use_negative:
- The analysis goal is cross-sectional differential abundance without a time structure
  — tools like ANCOM or MaAsLin2 handle that setting instead.
- The input is separate metadata and feature tables that have not yet been combined
  — build the master table with make_master_table() first.
- Data are meta'omic sequence profiles needing taxonomic/pathway profiling rather
  than longitudinal statistical testing.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0769
tools:
- name: LongDat
  role: R package implementing the GLMM null time model, per-covariate reducibility
    loop, and non-parametric effect-size (Spearman's rho) step
  repo: https://github.com/CCY-dev/LongDat
- name: longdat_cont()
  role: Longitudinal analysis pipeline function executing the three-stage test for
    a continuous time variable
  repo: https://github.com/CCY-dev/LongDat
- name: R
  role: Runtime environment in which the LongDat package and its GLMM pipelines are
    executed
  repo: https://cran.r-project.org/web/packages/LongDat/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad063
    title: 'LongDat: an R package for covariate-sensitive longitudinal analysis of
      high-dimensional data'
  source_content_hash: sha256:fd4ea73ed23be5d8075137d16cbf8f3f6f1fbcbf00ed5b7f4deca1607b8f16d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCY-dev/LongDat
  tools:
  - LongDat
  - R
  - longdat_cont()
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# covariate-reducibility-classification

## Summary

Classifies whether a significant time/treatment effect on each feature in a longitudinal GLMM analysis is reducible to a covariate's influence, using LongDat's per-covariate model loop in longdat_cont() or longdat_disc(). Produces Signal labels (OK_nc, OK_nrc, NS) that separate genuine time effects from covariate-driven ones.

## When to use

Use when analyzing longitudinal high-dimensional data (e.g. gut microbial abundance over Day or Time_point) where a significant time effect on a feature must be checked against potentially confounding covariates before being declared a treatment effect.

## When NOT to use

- The analysis goal is cross-sectional differential abundance without a time structure — tools like ANCOM or MaAsLin2 handle that setting instead.
- The input is separate metadata and feature tables that have not yet been combined — build the master table with make_master_table() first.
- Data are meta'omic sequence profiles needing taxonomic/pathway profiling rather than longitudinal statistical testing.

## Inputs

- LongDat master table (e.g. LongDat_cont_master_table or LongDat_disc_master_table) combining metadata and feature tables
- data_type setting ('count', 'proportion', 'binary', 'ordinal' or 'continuous')
- test_var name of the time variable (e.g. 'Day' for continuous time, 'Time_point' for discrete time)

## Outputs

- Result table with per-feature Signal column (OK_nc / OK_nrc / NS), p values and effect sizes (e.g. longdat_cont_result_table.csv)
- Covariate table listing covariate effects; blank when no covariate effect is found (e.g. longdat_cont_covariate_table.csv)
- Randomized control table (when data_type = 'count')

## How to apply

First run the GLMM null time model on the LongDat master table (e.g. LongDat_cont_master_table with test_var = 'Day') to detect which features change significantly over time; set a random seed beforehand when data_type = 'count' so the randomized control test is reproducible. Then, for each feature with a significant time effect, loop over covariates one at a time: each selected covariate is added alone as a fixed effect together with the time variable in GLMMs, to examine whether the time association can be reduced to that covariate's influence. Classify the feature as 'reducible' (time effect explained by a covariate), 'entangled' or 'not-reducible', and report the label in the Signal column (OK_nc = significant time effect with no covariate; OK_nrc = OK and not reducible to covariate; NS = no significant effect). For continuous time, quantify direction and magnitude with a non-parametric effect size (Spearman's rho), as in the worked example where BacteriumA and BacteriumC are 'OK_nc' with effect sizes -0.78 and 0.71 while BacteriumB is 'NS'. Finally, inspect the covariate table for identified covariate effects — it will be blank (as in the article's example) when no covariate effect was found.

## Related tools

- **LongDat** (R package implementing the GLMM null time model, per-covariate reducibility loop, and non-parametric effect-size (Spearman's rho) step) — https://github.com/CCY-dev/LongDat
- **longdat_cont()** (Longitudinal analysis pipeline function executing the three-stage test for a continuous time variable) — https://github.com/CCY-dev/LongDat
- **R** (Runtime environment in which the LongDat package and its GLMM pipelines are executed) — https://cran.r-project.org/web/packages/LongDat/

## Examples

```
set.seed(123); test_cont <- longdat_cont(input = LongDat_cont_master_table, data_type = "count", test_var = "Day")
```

## Evaluation signals

- Signal column values are from the expected vocabulary: 'OK_nc', 'OK_nrc', 'NS' (and related reducible/entangled/not-reducible labels).
- In the article's dummy two-timepoint (day 0 and 7) example, BacteriumA and BacteriumC show 'OK_nc' with effect sizes -0.78 and 0.71, and BacteriumB shows 'NS' — deviations suggest a pipeline error.
- The covariate table is blank when the result table reports no covariate effect; a populated covariate table without corresponding result-table entries indicates inconsistency.
- With data_type = 'count', the run must also emit a randomized control table, and results must be reproducible given the same seed.
- Output tables match the expected schemas: longdat_cont_result_table.csv and longdat_cont_covariate_table.csv.

## Limitations

- When data_type equals 'count', results depend on a randomized control test, so a seed must be set for reproducibility.
- A blank covariate table means no covariate effect was found — absence of covariate entries should not be read as evidence covariates were not tested.
- LongDat is designed for longitudinal data; it should not be assumed to outperform alternatives (MaAsLin2, ANCOM, lgpr, ZIBR) outside the multiple-covariate longitudinal setting where it was benchmarked.
- The covariate model test adds each covariate alone with time; interactions or joint multi-covariate reduction are not part of the described loop.

## Evidence

- [other] Per-covariate model loop design: [evidence span withheld — anchor status: not_found]
- [intro] Purpose of LongDat: "analyzes if there is significant change of the features over time (proxy for treatments), while detects and controls for covariates at the same time"
- [intro] Seed requirement for count data: "When data_type equals to "count", please remember to set seed (as shown below) so that you'll get reproducible randomized control test."
- [results] Signal interpretation in worked example: "treatment is a significant predictor for BacteriumA and BacteriumC as they show "OK_nc""
- [results] Effect sizes for example features: "BacteriumA and BacteriumC have decreasing and increasing patterns, respectively. From the next column "EffectSize", we know that the effect sizes are -0.78 and 0.71, respectively."
- [results] Covariate table blank when no covariate effect: "Since there is no covariate effect found in this example (according to the result table), the covariate table is blank."
- [results] OK_nrc meaning in discrete pipeline: "treatment is a significant predictor for BacteriumA as it shows "OK_nrc" (which represents "OK and not reducible to covariate")"
- [readme] Seed note from README: "LongDat R package takes longitudinal dataset as input data and analyzes if there is significant change of the features over time (proxy for treatments), while detects and controls for covariates at"

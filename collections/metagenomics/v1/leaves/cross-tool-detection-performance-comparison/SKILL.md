---
name: cross-tool-detection-performance-comparison
description: Use when comparing detection performance (FDR, TPR) of longitudinal high-dimensional
  tools like LongDat and MaAsLin2 across varying numbers of covariates on simulated
  microbiome count data.
when_to_use_negative:
- The data are cross-sectional with no time dimension — LongDat and MaAsLin2 are being
  applied here to longitudinal multivariable cohort data
- Only one tool is of interest and no identically simulated replicate set exists for
  a fair head-to-head comparison
- The analysis goal is single-dataset inference rather than method benchmarking; use
  the tool's own pipeline and inspect its Signal column instead
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: LongDat
  role: Tool under benchmark; longdat_cont() pipeline is run on every simulation replicate
    and its FDR/TPR stability across covariate counts is evaluated
  repo: https://github.com/CCY-dev/LongDat
- name: MaAsLin2
  role: Comparison tool run on the identical simulation replicates; expected to show
    FDR escalation and TPR diminishment above four covariates
- name: R
  role: Implementation environment for simulation, longdat_cont(), MaAsLin2 runs,
    and FDR/TPR computation
- name: microbiomeDASim
  role: Simulates longitudinal count datasets with varying numbers of covariates correlated
    with time
- name: SparseDOSSA2
  role: Simulates sparse microbiome count datasets used as benchmark replicates
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad063
    title: 'LongDat: an R package for covariate-sensitive longitudinal analysis of
      high-dimensional data'
  source_content_hash: sha256:43d6a4df179701c48f5bbf4820f105d664c19f1d463510380ba6c4292b4add9b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCY-dev/LongDat
  tools:
  - LongDat
  - MaAsLin2
  - R
  - SparseDOSSA2
  - microbiomeDASim
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cross-tool-detection-performance-comparison

## Summary

Benchmark longitudinal feature-detection tools head-to-head on identically simulated replicates, computing false discovery rate (FDR) and true positive rate (TPR) per covariate setting. This skill operationalizes the LongDat paper's finding that LongDat stays stable while MaAsLin2's FDR escalates and TPR diminishes as covariates increase.

## When to use

Use when comparing detection performance (FDR, TPR) of longitudinal high-dimensional tools like LongDat and MaAsLin2 across varying numbers of covariates on simulated microbiome count data.

## When NOT to use

- The data are cross-sectional with no time dimension — LongDat and MaAsLin2 are being applied here to longitudinal multivariable cohort data
- Only one tool is of interest and no identically simulated replicate set exists for a fair head-to-head comparison
- The analysis goal is single-dataset inference rather than method benchmarking; use the tool's own pipeline and inspect its Signal column instead

## Inputs

- Simulated longitudinal microbiome count datasets from microbiomeDASim and SparseDOSSA2 with 1, 2, 4, 8 and 16 dummy covariates correlated with time (rho 0.25-0.99)
- Master table in LongDat format built from metadata and feature tables (e.g. via make_master_table())

## Outputs

- benchmark_results.csv with FDR and TPR per tool per covariate setting
- Figures reproducing Fig. 4 and Supplementary Figs S3-S8 showing FDR and TPR trajectories across covariate counts
- Per-tool result tables from longdat_cont() and MaAsLin2 runs on each replicate

## How to apply

Simulate longitudinal count datasets with microbiomeDASim and SparseDOSSA2, varying the number of dummy covariates correlated with time over 1, 2, 4, 8 and 16 with correlation rho between 0.25 and 0.99. Run longdat_cont() from the LongDat R package on every simulation replicate and run MaAsLin2 on the identical replicates for a head-to-head comparison. Compute FDR and TPR per tool per covariate setting, and additionally compute the covariate-effect success rate, checking that LongDat's median rate reaches ~0.95. Summarize FDR and TPR trajectories across covariate counts into figures reproducing Fig. 4 and Supplementary Figs S3-S8, and write benchmark_results.csv. The rationale is that tools handling multiple covariates poorly lose control of false discoveries and power as confounders accumulate.

## Related tools

- **LongDat** (Tool under benchmark; longdat_cont() pipeline is run on every simulation replicate and its FDR/TPR stability across covariate counts is evaluated) — https://github.com/CCY-dev/LongDat
- **MaAsLin2** (Comparison tool run on the identical simulation replicates; expected to show FDR escalation and TPR diminishment above four covariates)
- **R** (Implementation environment for simulation, longdat_cont(), MaAsLin2 runs, and FDR/TPR computation)
- **microbiomeDASim** (Simulates longitudinal count datasets with varying numbers of covariates correlated with time)
- **SparseDOSSA2** (Simulates sparse microbiome count datasets used as benchmark replicates)

## Examples

```
test_cont <- longdat_cont(input = LongDat_cont_master_table, data_type = "count", test_var = "Day")
```

## Evaluation signals

- LongDat's FDR stays stable and its TPR does not diminish above four covariates
- MaAsLin2's FDR escalates and TPR diminishes as the number of covariates increases, especially with more than four covariates
- LongDat's median covariate-effect success rate reaches ~0.95
- benchmark_results.csv is produced with FDR and TPR per tool per covariate setting (1, 2, 4, 8, 16 covariates)
- Both tools were run on identical simulation replicates for a valid head-to-head comparison

## Limitations

- Findings are grounded in simulated data from microbiomeDASim and SparseDOSSA2 with covariates correlated with time; real-data behavior may differ
- The benchmark tests covariate counts only up to 16 and correlation rho between 0.25 and 0.99; settings outside this grid are untested
- The paper's conclusions emphasize multiple covariates; with few covariates the advantage of LongDat over MaAsLin2, ANCOM, lgpr and ZIBR is less pronounced

## Evidence

- [other] Simulate longitudinal count datasets with microbiomeDASim and SparseDOSSA2, varying the number of dummy covariates correlated with time over 1, 2, 4, 8 and 16 with correlation rho between 0.25 and 0.99.: [evidence span withheld — anchor status: not_found]
- [other] LongDat's performance remains stable across all numbers of covariates, while MaAsLin2's FDR escalates and TPR diminishes as covariates increase, especially with more than four covariates.: [evidence span withheld — anchor status: not_found]
- [abstract] We tested and compared LongDat with other tools (i.e. MaAsLin2, ANCOM, lgpr and ZIBR) on both simulated and real data.: "We tested and compared LongDat with other tools (i.e. MaAsLin2, ANCOM, lgpr and ZIBR) on both simulated and real data."
- [abstract] LongDat outperformed these tools in accuracy, runtime and memory cost, especially when there were multiple covariates.: "We showed that LongDat outperformed these tools in accuracy, runtime and memory cost, especially when there were multiple covariates."
- [intro] Run longdat_cont() on master table with count data type, testing a continuous time variable.: "test_cont <- longdat_cont(input = LongDat_cont_master_table,
data_type = "count",
test_var = "Day","

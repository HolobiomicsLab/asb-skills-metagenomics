---
name: covariate-effect-success-rate-evaluation
description: Use when comparing longitudinal differential-abundance tools (e.g. LongDat
  vs MaAsLin2) on simulated multi-covariate data and you need to quantify whether
  covariate control keeps FDR stable and covariate-effect success rate near 0.95.
when_to_use_negative:
- Data are cross-sectional or single-time-point, since LongDat and this benchmark
  target longitudinal (multi-time-point) designs
- Covariates of interest are independent of time (rho near 0), so the FDR-escalation
  regime that distinguishes LongDat from MaAsLin2 will not be triggered
- The goal is a single-tool analysis rather than a head-to-head benchmark — inspect
  LongDat's Signal column and covariate table directly instead
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3053
tools:
- name: LongDat
  role: 'Tested pipeline: longdat_cont() run on every simulation replicate; its stable
    FDR/TPR and ~0.95 covariate-effect success rate are the expected outcome'
  repo: github.com/CCY-dev/LongDat
- name: MaAsLin2
  role: Head-to-head comparator run on identical replicates; expected FDR escalation
    and TPR diminishment as covariates increase beyond four
  repo: https://github.com/biobakery/biobakery
- name: R
  role: Implementation environment for simulation, both pipelines, and metric computation
  repo: https://cran.r-project.org/web/packages/LongDat/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad063
    title: 'LongDat: an R package for covariate-sensitive longitudinal analysis of
      high-dimensional data'
  source_content_hash: sha256:602ed9c29dfb2b16492b5689b137e15bcdf5a3562294d7b593cfa65694e7f2be
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LongDat
  - MaAsLin2
  - R
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# covariate-effect-success-rate-evaluation

## Summary

Benchmark skill that evaluates how well longitudinal methods control confounding as dummy covariates correlated with time are added, by computing FDR, TPR, and a covariate-effect success rate (~0.95 median for LongDat) on simulated longitudinal microbiome count data.

## When to use

Use when comparing longitudinal differential-abundance tools (e.g. LongDat vs MaAsLin2) on simulated multi-covariate data and you need to quantify whether covariate control keeps FDR stable and covariate-effect success rate near 0.95.

## When NOT to use

- Data are cross-sectional or single-time-point, since LongDat and this benchmark target longitudinal (multi-time-point) designs
- Covariates of interest are independent of time (rho near 0), so the FDR-escalation regime that distinguishes LongDat from MaAsLin2 will not be triggered
- The goal is a single-tool analysis rather than a head-to-head benchmark — inspect LongDat's Signal column and covariate table directly instead

## Inputs

- Simulated longitudinal microbiome count datasets from microbiomeDASim and SparseDOSSA2 (1, 2, 4, 8, 16 dummy covariates; rho 0.25-0.99)
- LongDat master table (e.g. LongDat_cont_master_table format: metadata + feature table)
- Known ground-truth positive/negative features per simulation replicate

## Outputs

- benchmark_results.csv with FDR, TPR, and covariate-effect success rate per tool per covariate setting
- FDR and TPR trajectory figures across covariate counts (reproducing Fig. 4 and Supplementary Figs S3-S8)

## How to apply

Simulate longitudinal count datasets with microbiomeDASim and SparseDOSSA2, varying the number of dummy covariates correlated with time over 1, 2, 4, 8 and 16 with correlation rho between 0.25 and 0.99. Run longdat_cont() from the LongDat R package and MaAsLin2 on identical simulation replicates, then compute false discovery rate (FDR) and true positive rate (TPR) per tool per covariate setting. Compute the covariate-effect success rate and check that LongDat's median rate reaches ~0.95; summarize FDR/TPR trajectories across covariate counts into figures reproducing Fig. 4 and Supplementary Figs S3-S8 and write benchmark_results.csv. The rationale is that LongDat accounts for a potentially large number of covariates simultaneously, whereas MaAsLin2's FDR escalates and TPR diminishes above four covariates.

## Related tools

- **LongDat** (Tested pipeline: longdat_cont() run on every simulation replicate; its stable FDR/TPR and ~0.95 covariate-effect success rate are the expected outcome) — github.com/CCY-dev/LongDat
- **MaAsLin2** (Head-to-head comparator run on identical replicates; expected FDR escalation and TPR diminishment as covariates increase beyond four) — https://github.com/biobakery/biobakery
- **R** (Implementation environment for simulation, both pipelines, and metric computation) — https://cran.r-project.org/web/packages/LongDat/

## Examples

```
set.seed(1); test_cont <- longdat_cont(input = LongDat_cont_master_table, data_type = "count", test_var = "Day")
```

## Evaluation signals

- LongDat's FDR remains stable across all covariate counts (1, 2, 4, 8, 16) rather than rising
- LongDat's TPR does not diminish above four covariates, while MaAsLin2's FDR escalates and TPR diminishes
- Median covariate-effect success rate for LongDat reaches ~0.95
- benchmark_results.csv contains per-tool, per-covariate-setting FDR/TPR rows for every replicate setting
- Result tables show expected Signal values (e.g. 'OK_nc') for simulated true-positive features

## Limitations

- The MaAsLin2 performance degradation is demonstrated mainly in the correlated-covariate regime (rho 0.25-0.99, >4 covariates); results may differ with uncorrelated covariates
- Simulations use longitudinal count data; behavior for proportion, binary, ordinal or continuous data types requires separate benchmarking
- For count data_type, LongDat's randomized control test requires setting a seed for reproducibility

## Evidence

- [abstract] LongDat outperformed MaAsLin2, ANCOM, lgpr and ZIBR in accuracy, runtime and memory cost, especially when there were multiple covariates.: "We showed that LongDat outperformed these tools in accuracy, runtime and memory cost, especially when there were multiple covariates."
- [other] Simulated datasets with correlated dummy covariates drive the benchmark: [evidence span withheld — anchor status: not_found]
- [methods] Validation criterion for the benchmark run: [evidence span withheld — anchor status: not_found]
- [results] Result table Signal inspection grounds covariate-effect success computation: "treatment is a significant predictor for BacteriumA and BacteriumC as they show "OK_nc""
- [readme] LongDat installation path used in the benchmark environment: [evidence span withheld — anchor status: not_found]

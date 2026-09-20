---
name: covariate-confounding-benchmark-design
description: Use when benchmarking longitudinal microbiome differential-abundance
  tools under covariate-time confounding, or when deciding whether a tool's FDR/TPR
  is robust to increasing numbers of dummy covariates.
when_to_use_negative:
- The input is a single real cohort dataset with no simulated ground truth, so FDR/TPR
  cannot be computed against known positives
- The question is a routine LongDat/longdat_cont() analysis run rather than a methodological
  benchmark against competing tools
- Data are cross-sectional with no time dimension, since LongDat targets longitudinal
  data with significant change of features over time
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: LongDat
  role: Benchmarked longitudinal pipeline; longdat_cont() is run on every simulation
    replicate to measure FDR/TPR stability across covariate counts
  repo: https://github.com/CCY-dev/LongDat
- name: MaAsLin2
  role: Comparison baseline run on identical simulation replicates; expected to show
    FDR escalation and TPR diminution above four covariates
  repo: https://github.com/biobakery/biobakery
- name: microbiomeDASim
  role: Generator of simulated longitudinal count datasets with controlled covariate-time
    correlation
- name: SparseDOSSA2
  role: Generator of simulated microbiome count datasets used in the covariate-scaling
    benchmark
- name: R
  role: Execution environment for the simulation, longdat_cont() pipeline, MaAsLin2
    runs, and FDR/TPR computation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad063
    title: 'LongDat: an R package for covariate-sensitive longitudinal analysis of
      high-dimensional data'
  source_content_hash: sha256:20930c6d8abd6ba08364e759a34ed525077c94bd2e3b6be912bd64f64a8d150c
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# covariate-confounding-benchmark-design

## Summary

Benchmark design for evaluating how longitudinal high-dimensional analysis methods degrade as covariates correlated with time accumulate, following the LongDat vs MaAsLin2 head-to-head simulation study (Chen et al. 2023, Bioinformatics Advances, DOI 10.1093/bioadv/vbad063).

## When to use

Use when benchmarking longitudinal microbiome differential-abundance tools under covariate-time confounding, or when deciding whether a tool's FDR/TPR is robust to increasing numbers of dummy covariates.

## When NOT to use

- The input is a single real cohort dataset with no simulated ground truth, so FDR/TPR cannot be computed against known positives
- The question is a routine LongDat/longdat_cont() analysis run rather than a methodological benchmark against competing tools
- Data are cross-sectional with no time dimension, since LongDat targets longitudinal data with significant change of features over time

## Inputs

- Simulated longitudinal count datasets with 1, 2, 4, 8, or 16 dummy covariates correlated with time (rho 0.25-0.99) from microbiomeDASim and SparseDOSSA2
- LongDat-formatted master table built from metadata and feature tables
- MaAsLin2 input files for the identical simulation replicates

## Outputs

- FDR and TPR trajectories per tool per covariate setting
- benchmark_results.csv
- Figures reproducing Fig. 4 and Supplementary Figs S3-S8
- Covariate-effect success rate summary (median ~0.95 for LongDat)

## How to apply

Simulate longitudinal count datasets with microbiomeDASim and SparseDOSSA2, varying the number of dummy covariates correlated with time over 1, 2, 4, 8 and 16 with correlation rho between 0.25 and 0.99. Run longdat_cont() in R on every simulation replicate (setting a seed for reproducibility with count data) and run MaAsLin2 on identical replicates for the head-to-head comparison. Compute false discovery rate (FDR) and true positive rate (TPR) per tool per covariate setting, and compute the covariate-effect success rate, checking that LongDat's median rate reaches ~0.95. Summarize FDR and TPR trajectories across covariate counts into figures reproducing Fig. 4 and Supplementary Figs S3-S8, and write benchmark_results.csv.

## Related tools

- **LongDat** (Benchmarked longitudinal pipeline; longdat_cont() is run on every simulation replicate to measure FDR/TPR stability across covariate counts) — https://github.com/CCY-dev/LongDat
- **MaAsLin2** (Comparison baseline run on identical simulation replicates; expected to show FDR escalation and TPR diminution above four covariates) — https://github.com/biobakery/biobakery
- **microbiomeDASim** (Generator of simulated longitudinal count datasets with controlled covariate-time correlation)
- **SparseDOSSA2** (Generator of simulated microbiome count datasets used in the covariate-scaling benchmark)
- **R** (Execution environment for the simulation, longdat_cont() pipeline, MaAsLin2 runs, and FDR/TPR computation)

## Examples

```
test_cont <- longdat_cont(input = LongDat_cont_master_table, data_type = "count", test_var = "Day")
```

## Evaluation signals

- LongDat's FDR remains stable while MaAsLin2's FDR escalates and TPR diminishes as covariates increase, especially above four covariates
- Median covariate-effect success rate for LongDat reaches approximately 0.95
- FDR and TPR trajectories reproduce Fig. 4 and Supplementary Figs S3-S8
- benchmark_results.csv contains per-tool, per-covariate-setting FDR and TPR values
- Randomized control tests for count data are reproducible because the seed was set before running longdat_cont()

## Limitations

- The divergence between tools appears mainly with more than four covariates; at low covariate counts the tools may perform similarly
- Findings were established on simulated longitudinal microbiome count data plus real data; LongDat's scope extends to binary, categorical and continuous data, but this benchmark design specifically targets longitudinal count simulations
- When data_type equals 'count', forgetting to set a seed breaks reproducibility of the randomized control test

## Evidence

- [abstract] Benchmark rationale from abstract: "We tested and compared LongDat with other tools (i.e. MaAsLin2, ANCOM, lgpr and ZIBR) on both simulated and real data."
- [abstract] Benchmark outcome motivating the covariate-scaling design: "We showed that LongDat outperformed these tools in accuracy, runtime and memory cost, especially when there were multiple covariates."
- [intro] Reproducibility requirement for the count-data benchmark runs: "When data_type equals to "count", please remember to set seed (as shown below) so that you'll get reproducible randomized control test."
- [results] Output tables produced by the benchmarked pipeline: "If you have count data (data_type equals to "count"), then there are chances that you get a third table "randomized control table"."
- [methods] Core pipeline under benchmark: "Longitudinal analysis pipeline with longdat_cont()"

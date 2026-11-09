---
name: false-discovery-rate-and-true-positive-rate-estimation
description: Use when head-to-head benchmarking of differential-abundance or longitudinal
  methods on simulated data with known ground truth is needed, and FDR/TPR trajectories
  must be quantified as covariate count or confounding strength varies.
when_to_use_negative:
- Real datasets without ground truth — FDR and TPR cannot be computed without simulated
  known-positive and known-negative features
- Single-covariate or cross-sectional designs where the covariate-count stress gradient
  (1 to 16 correlated dummy covariates) that drives the LongDat vs MaAsLin2 divergence
  is absent
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: LongDat
  role: Tool under test; its longdat_cont() pipeline is run on every simulation replicate
    and evaluated for stable FDR/TPR across covariate counts
  repo: https://github.com/CCY-dev/LongDat
- name: MaAsLin2
  role: Comparator tool run on identical simulation replicates; its escalating FDR
    and diminishing TPR above four covariates serve as the benchmark contrast
  repo: https://github.com/biobakery/biobakery
- name: R
  role: Language in which the longdat_cont() pipeline and benchmark computation are
    executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad063
    title: 'LongDat: an R package for covariate-sensitive longitudinal analysis of
      high-dimensional data'
  source_content_hash: sha256:b011cd9048ffe98dcc4382d5988db4927b0dfe9603b5aca876f3b3398880f151
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCY-dev/LongDat
  tools:
  - LongDat
  - MaAsLin2
  - R
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# false-discovery-rate-and-true-positive-rate-estimation

## Summary

Benchmarking method for computing false discovery rate (FDR) and true positive rate (TPR) when comparing longitudinal high-dimensional tools such as LongDat and MaAsLin2 across increasing numbers of covariates, as used to reproduce Fig. 4 and Supplementary Figs S3-S8 of the LongDat paper.

## When to use

Use when head-to-head benchmarking of differential-abundance or longitudinal methods on simulated data with known ground truth is needed, and FDR/TPR trajectories must be quantified as covariate count or confounding strength varies.

## When NOT to use

- Real datasets without ground truth — FDR and TPR cannot be computed without simulated known-positive and known-negative features
- Single-covariate or cross-sectional designs where the covariate-count stress gradient (1 to 16 correlated dummy covariates) that drives the LongDat vs MaAsLin2 divergence is absent

## Inputs

- Simulated longitudinal count datasets with known ground-truth signal features (microbiomeDASim / SparseDOSSA2 replicates)
- Covariate settings grid: 1, 2, 4, 8, 16 dummy covariates correlated with time (rho 0.25-0.99)
- Per-tool result tables (LongDat result table with Signal column; MaAsLin2 q-value/association output)

## Outputs

- FDR and TPR per tool per covariate setting
- Covariate-effect success rate (median ~0.95 expected for LongDat)
- Figures reproducing Fig. 4 and Supplementary Figs S3-S8 (FDR/TPR trajectories across covariate counts)
- benchmark_results.csv

## How to apply

Simulate longitudinal microbiome count datasets (e.g. with microbiomeDASim and SparseDOSSA2) where dummy covariates correlated with time vary over 1, 2, 4, 8 and 16, with correlation rho between 0.25 and 0.99, so that ground-truth positive and negative features are known. Run each tool (longdat_cont() in R for LongDat; MaAsLin2 for the comparator) on identical simulation replicates, then compute FDR and TPR per tool per covariate setting from the called vs. true features. Additionally compute a covariate-effect success rate, checking that LongDat's median rate reaches ~0.95. Summarize FDR and TPR trajectories across covariate counts into figures reproducing Fig. 4 and Supplementary Figs S3-S8 and write benchmark_results.csv. Accept the run if LongDat's FDR stays stable and TPR does not diminish above four covariates while MaAsLin2's FDR escalates and TPR diminishes.

## Related tools

- **LongDat** (Tool under test; its longdat_cont() pipeline is run on every simulation replicate and evaluated for stable FDR/TPR across covariate counts) — https://github.com/CCY-dev/LongDat
- **MaAsLin2** (Comparator tool run on identical simulation replicates; its escalating FDR and diminishing TPR above four covariates serve as the benchmark contrast) — https://github.com/biobakery/biobakery
- **R** (Language in which the longdat_cont() pipeline and benchmark computation are executed)

## Examples

```
Rscript -e 'set.seed(1); res <- longdat_cont(input = LongDat_cont_master_table, data_type = "count", test_var = "Day"); fdr <- sum(res$Signal!="OK" & true_positive==FALSE)/sum(res$Signal!="OK")'
```

## Evaluation signals

- LongDat's FDR remains stable across all tested covariate counts (1, 2, 4, 8, 16)
- LongDat's TPR does not diminish when more than four covariates are present, while MaAsLin2's FDR escalates and TPR diminishes above four covariates
- Median covariate-effect success rate for LongDat reaches approximately 0.95
- FDR/TPR trajectories reproduce Fig. 4 and Supplementary Figs S3-S8 and are persisted to benchmark_results.csv
- Both tools were run on identical simulation replicates for each rho (0.25-0.99) and covariate-count setting

## Limitations

- Conclusions rest on simulated longitudinal microbiome count data (microbiomeDASim / SparseDOSSA2); real-data behavior may differ
- The LongDat randomized control test with count data requires setting a seed for reproducibility, so benchmark replicates must control seeding to be comparable
- The benchmark gradient depends on dummy covariates correlated with time; other confounding structures are not covered by this validation

## Evidence

- [abstract] We showed that LongDat outperformed these tools in accuracy, runtime and memory cost, especially when there were multiple covariates.: "We showed that LongDat outperformed these tools in accuracy, runtime and memory cost, especially when there were multiple covariates."
- [results] LongDat's performance remains stable across all numbers of covariates, while MaAsLin2's FDR escalates and TPR diminishes: [evidence span withheld — anchor status: not_found]
- [methods] Simulate longitudinal count datasets with microbiomeDASim and SparseDOSSA2, varying the number of dummy covariates: [evidence span withheld — anchor status: not_found]
- [methods] Compute false discovery rate (FDR) and true positive rate (TPR) per tool per covariate setting.: [evidence span withheld — anchor status: not_found]
- [intro] When data_type equals to "count", please remember to set seed (as shown below) so that you'll get reproducible randomized control test.: "When data_type equals to "count", please remember to set seed (as shown below) so that you'll get reproducible randomized control test."

---
name: fdr-estimation-across-simulation-replicates
description: Use when comparing longitudinal differential-abundance tools on simulated
  count data and you need per-setting FDR, TPR, and accuracy aggregated over replicate
  simulations with known ground truth.
when_to_use_negative:
- Data are real experimental data without known ground truth — FDR and TPR cannot
  be computed against simulated truth
- Datasets contain multiple covariates — the article's core result is that LongDat's
  advantage (accuracy, runtime, memory) is greatest with multiple covariates, so a
  covariate-free benchmark does not represent that regime
- Single non-replicated comparison — FDR medians across 100 replicates per setting
  are the basis of the conclusion; a one-off run cannot support distributional claims
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: LongDat
  role: Method under test; longdat_cont() run on simulated master tables with time
    as the continuous test variable and data_type 'count'
  repo: https://github.com/CCY-dev/LongDat
- name: MaAsLin2
  role: Comparator method run in negative binomial (NEGBIN) mode on the same simulated
    datasets
  repo: https://github.com/biobakery/biobakery
- name: R
  role: Language for simulation, running both tools, computing FDR/TPR/accuracy, and
    aggregating metrics
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad063
    title: 'LongDat: an R package for covariate-sensitive longitudinal analysis of
      high-dimensional data'
  source_content_hash: sha256:55462859638307e5151000ba01b132c71f875af00ce717c0c5e5405384d655fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCY-dev/LongDat
  tools:
  - LongDat
  - MaAsLin2
  - R
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# fdr-estimation-across-simulation-replicates

## Summary

Benchmark-method skill for estimating the false discovery rate (and TPR/accuracy) of longitudinal feature-detection tools by aggregating results across replicate microbiomeDASim simulations, as used to compare LongDat against MaAsLin2 (NEGBIN mode).

## When to use

Use when comparing longitudinal differential-abundance tools on simulated count data and you need per-setting FDR, TPR, and accuracy aggregated over replicate simulations with known ground truth.

## When NOT to use

- Data are real experimental data without known ground truth — FDR and TPR cannot be computed against simulated truth
- Datasets contain multiple covariates — the article's core result is that LongDat's advantage (accuracy, runtime, memory) is greatest with multiple covariates, so a covariate-free benchmark does not represent that regime
- Single non-replicated comparison — FDR medians across 100 replicates per setting are the basis of the conclusion; a one-off run cannot support distributional claims

## Inputs

- Longitudinal microbiome count datasets simulated with microbiomeDASim (sample sizes 10–300, two effect sizes, no covariates, 100 replicates per setting)
- Master tables in LongDat format (metadata + feature table) for each simulated dataset
- Ground-truth labels of truly affected vs. unaffected features from the simulation

## Outputs

- Per-replicate, per-method accuracy, true positive rate, and false discovery rate
- Per-method runtime and peak memory consumption for each sample-size and effect-size setting
- Aggregated benchmark table (e.g. simulation_benchmark_results.csv) across all 100 simulations per setting

## How to apply

Simulate longitudinal microbiome count datasets with microbiomeDASim across sample sizes (e.g. 10–300) and effect sizes, generating ~100 replicate simulations per parameter setting so FDR and TPR distributions are stable. Run each candidate tool on the same simulated master tables — here longdat_cont() from the LongDat R package with time as the continuous test variable (setting the seed when data_type is 'count', since LongDat performs a randomized control test), and MaAsLin2 in negative binomial (NEGBIN) mode as comparator. For each replicate and method, classify features as significant vs. non-significant and compute accuracy, true positive rate, and false discovery rate against the simulation ground truth. Record per-method runtime and peak memory consumption per sample-size/effect-size setting, then aggregate across the 100 replicates and compile all metrics (e.g. into a simulation_benchmark_results.csv). Judge correctness by checking the reported accuracy range and FDR behavior: in the source study, LongDat and MaAsLin2 both fell in 0.9–1 accuracy, but LongDat's FDR median stayed at zero across all settings while MaAsLin2 showed higher TPR at small sample sizes at the cost of higher FDR.

## Related tools

- **LongDat** (Method under test; longdat_cont() run on simulated master tables with time as the continuous test variable and data_type 'count') — https://github.com/CCY-dev/LongDat
- **MaAsLin2** (Comparator method run in negative binomial (NEGBIN) mode on the same simulated datasets) — https://github.com/biobakery/biobakery
- **R** (Language for simulation, running both tools, computing FDR/TPR/accuracy, and aggregating metrics)

## Examples

```
install.packages("LongDat"); set.seed(1); res <- longdat_cont(input = master_table, data_type = "count", test_var = "Day")
```

## Evaluation signals

- Accuracy of significant-feature detection falls within the expected 0.9–1 range for both LongDat and MaAsLin2
- LongDat's FDR median equals zero across all tested effect sizes and sample sizes
- LongDat's TPR increases monotonically with sample size, while MaAsLin2 shows higher TPR at small sample sizes accompanied by higher FDR
- Seed-dependent randomized control test for data_type 'count' is reproducible when the seed is set before longdat_cont()
- Aggregated metrics table contains per-method runtime and peak memory for every sample-size/effect-size combination across 100 replicates

## Limitations

- Benchmark setting is covariate-free; the article's stronger claims (LongDat outperforming MaAsLin2, ANCOM, lgpr and ZIBR in accuracy, runtime and memory) apply especially when there are multiple covariates
- LongDat requires setting a seed when data_type equals 'count' because of its randomized control test, otherwise results are not reproducible
- Installing LongDat can fail with namespace export errors from dependency packages, requiring the dependency (e.g. B_package) to be installed first

## Evidence

- [abstract] LongDat outperformed MaAsLin2, ANCOM, lgpr and ZIBR in accuracy, runtime and memory cost, especially when there were multiple covariates.: "We showed that LongDat outperformed these tools in accuracy, runtime and memory cost, especially when there were multiple covariates."
- [intro] Set seed before longdat_cont() with count data for reproducible randomized control test.: "When data_type equals to "count", please remember to set seed (as shown below) so that you'll get reproducible randomized control test."
- [readme] LongDat output table contains p values, effect sizes and covariates per feature, enabling downstream metric computation.: "The output table contains p values, effect sizes and covariates of each feature, making the downstream analysis easy."
- [intro] LongDat analyzes longitudinal datasets for significant change of features over time while detecting and controlling covariates.: "LongDat R package takes longitudinal dataset as input data and analyzes if there is significant change of the features over time"

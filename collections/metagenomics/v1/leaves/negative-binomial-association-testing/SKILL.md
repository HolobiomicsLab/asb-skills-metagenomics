---
name: negative-binomial-association-testing
description: Use when the input is a longitudinal microbiome count dataset (master
  table with samples x features over time) and the goal is to detect features changing
  significantly over time while comparing error control (TPR/FDR) across negative-binomial
  methods.
when_to_use_negative:
- The data are not longitudinal or the test variable is not time/treatment over repeated
  measurements — LongDat is designed for longitudinal analysis of feature change over
  time.
- The data are already a preprocessed proportion/binary/continuous feature table where
  data_type='count' (and thus the randomized control test) does not apply.
- A single method's absolute sensitivity matters more than calibrated error control
  — MaAsLin2 NEGBIN can attain higher TPR at small sample sizes but with higher FDR,
  whereas this skill's benchmark target is LongDat's zero-FDR behavior.
edam_operation: http://edamontology.org/operation_3659
tools:
- name: LongDat
  role: 'Primary method: longdat_cont() runs the longitudinal analysis pipeline on
    the count-data master table with a continuous time test variable, producing result,
    covariate and randomized-control tables'
  repo: https://github.com/CCY-dev/LongDat
- name: MaAsLin2
  role: Comparison method run in negative binomial (NEGBIN) mode on the same simulated
    datasets to benchmark TPR/FDR trade-offs
  repo: https://github.com/biobakery/biobakery
- name: R
  role: Implementation environment for installing LongDat, setting seeds, simulating
    data and computing benchmark metrics
  repo: https://github.com/CCY-dev/LongDat
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad063
    title: 'LongDat: an R package for covariate-sensitive longitudinal analysis of
      high-dimensional data'
  source_content_hash: sha256:564866f5bd6332358630d0b2d2067542e08f66d9b8e96266bb68fb421d86ae78
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCY-dev/LongDat
  tools:
  - LongDat
  - MaAsLin2
  - R
  edam_operation: http://edamontology.org/operation_3659
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# negative-binomial-association-testing

## Summary

Benchmark and apply negative-binomial-based longitudinal feature association testing by running LongDat's longdat_cont() (count data type) against MaAsLin2 in NEGBIN mode, on simulated or real longitudinal microbiome count data, scoring accuracy, true positive rate and false discovery rate.

## When to use

Use when the input is a longitudinal microbiome count dataset (master table with samples x features over time) and the goal is to detect features changing significantly over time while comparing error control (TPR/FDR) across negative-binomial methods.

## When NOT to use

- The data are not longitudinal or the test variable is not time/treatment over repeated measurements — LongDat is designed for longitudinal analysis of feature change over time.
- The data are already a preprocessed proportion/binary/continuous feature table where data_type='count' (and thus the randomized control test) does not apply.
- A single method's absolute sensitivity matters more than calibrated error control — MaAsLin2 NEGBIN can attain higher TPR at small sample sizes but with higher FDR, whereas this skill's benchmark target is LongDat's zero-FDR behavior.

## Inputs

- Longitudinal master table (metadata + feature table, e.g. LongDat_cont_master_table format) with count-valued features and a continuous time variable
- microbiomeDASim-simulated longitudinal count datasets (e.g. sample sizes 10-300, two effect sizes, no covariates, 100 replicates per setting)
- Metadata table defining sample IDs and the time/test variable (e.g. 'Day')

## Outputs

- LongDat result table with p values, effect sizes and Signal column per feature (e.g. 'OK_nc', 'NS') plus a randomized control table for count data
- MaAsLin2 NEGBIN-mode significance calls per feature
- Per-method accuracy, TPR and FDR aggregated across simulation replicates (e.g. simulation_benchmark_results.csv), plus runtime and peak memory metrics

## How to apply

First generate or load a longitudinal master table (metadata plus feature table, e.g. via make_master_table()) with a continuous time test variable such as 'Day'. Run longdat_cont() from LongDat with data_type = 'count' and the time column as test_var, remembering to set.seed() beforehand because count data triggers a randomized control test. Run MaAsLin2 on the same data in negative binomial (NEGBIN) mode as the comparator. For each replicate, compute accuracy of significant-feature detection, true positive rate and false discovery rate, and optionally record runtime and peak memory per setting. In the paper's benchmark on microbiomeDASim-simulated covariate-free data (100 replicates per setting, sample sizes 10-300, two effect sizes), both methods reached accuracy 0.9-1, but LongDat kept an FDR median of zero across all settings while MaAsLin2 traded higher FDR for higher TPR at small sample sizes.

## Related tools

- **LongDat** (Primary method: longdat_cont() runs the longitudinal analysis pipeline on the count-data master table with a continuous time test variable, producing result, covariate and randomized-control tables) — https://github.com/CCY-dev/LongDat
- **MaAsLin2** (Comparison method run in negative binomial (NEGBIN) mode on the same simulated datasets to benchmark TPR/FDR trade-offs) — https://github.com/biobakery/biobakery
- **R** (Implementation environment for installing LongDat, setting seeds, simulating data and computing benchmark metrics) — https://github.com/CCY-dev/LongDat

## Examples

```
set.seed(1); test_cont <- longdat_cont(input = LongDat_cont_master_table, data_type = "count", test_var = "Day")
```

## Evaluation signals

- Accuracy of significant-feature detection falls within 0.9-1 across tested effect sizes and sample sizes
- LongDat's FDR median is zero across all tested settings, while TPR increases with sample size
- For count data, a randomized control table is produced and results are reproducible across reruns because the seed was set
- The result table contains p values, effect sizes and Signal values (e.g. 'OK_nc' for significant time effects, 'NS' for no effect) for every feature
- Runtime and peak memory are recorded per sample-size/effect-size setting and are consistent with LongDat's claimed low-memory, efficient profile

## Limitations

- When data_type equals 'count', forgetting to set.seed() makes the randomized control test non-reproducible.
- Package installation can fail with namespace-export errors from dependency packages (e.g. "object 'A' is not exported by 'namespace:B_package'"); install the dependency first.
- The reported zero-FDR behavior of LongDat was observed on covariate-free simulations; MaAsLin2 NEGBIN may achieve higher TPR at small sample sizes, so the choice involves a TPR-FDR trade-off.

## Evidence

- [abstract] Benchmark context from the abstract: "We tested and compared LongDat with other tools (i.e. MaAsLin2, ANCOM, lgpr and ZIBR) on both simulated and real data."
- [readme] Count-mode randomized control test: "When data_type equals to "count", please remember to set seed (as shown below) so that you'll get reproducible randomized control test."
- [results] Count data pipeline produces a third table: "If you have count data (data_type equals to "count"), then there are chances that you get a third table "randomized control table"."
- [methods] Continuous-time pipeline entry point: "Longitudinal analysis pipeline with longdat_cont()"

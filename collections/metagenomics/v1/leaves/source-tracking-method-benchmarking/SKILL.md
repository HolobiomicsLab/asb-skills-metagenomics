---
name: source-tracking-method-benchmarking
description: Use when you need to quantitatively compare microbial source tracking
  methods under controlled synthetic sinks with known ground-truth mixing proportions,
  especially with many nuisance candidate sources and unknown-source contributions
  up to 90%.
when_to_use_negative:
- When the candidate source set is small and fully covers all potential environments
  — the benchmark targets source exploration with many nuisance sources; existing
  methods are suboptimal mainly when the number of contributing sources is much smaller
  than candidates considered
- When you need accurate source tracking on real (non-simulated) sinks without ground-truth
  mixing proportions — the metrics here require known convex-mixture ground truth
- When the analysis goal is a single method's application rather than head-to-head
  method comparison against FEAST, SourceTracker2, and RAD
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: STENSL
  role: Method under test; expectation-maximization algorithm with sparsity (L1 Lasso
    source selection, lambda = 1e-6) for identifying contributing sources among many
    candidate environments
  repo: https://github.com/cozygene/FEAST/tree/STENSL
- name: FEAST
  role: Competitor method; Fast Expectation-mAximization microbial Source Tracking
    run on identical simulated sink-source sets
  repo: https://github.com/cozygene/FEAST
- name: SourceTracker2
  role: Competitor baseline source tracking method scored on the same simulation metrics
- name: RAD
  role: Competitor baseline source tracking method scored on the same simulation metrics
- name: R
  role: Implementation environment for FEAST/STENSL (R >= 3.4.4) with dependencies
    Rcpp, RcppArmadillo, vegan, dplyr, reshape2, gridExtra, ggplot2, ggthemes
  repo: https://github.com/cozygene/FEAST
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/msystems.00995-21
    title: 'STENSL: Microbial Source Tracking with ENvironment SeLection'
  source_content_hash: sha256:16b3abc56a81fb42e175710ff0432aff06f5199aba4274041ad48c440571b716
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FEAST
  - R
  - RAD
  - STENSL
  - SourceTracker2
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# source-tracking-method-benchmarking

## Summary

A simulation-based benchmarking protocol for microbial source tracking methods that compares STENSL against FEAST, SourceTracker2, and RAD on Earth Microbiome Project-derived synthetic sinks, scoring each method by mixing-proportion MSE, false positive rate of identified sources, and absolute error of the estimated unknown contribution.

## When to use

Use when you need to quantitatively compare microbial source tracking methods under controlled synthetic sinks with known ground-truth mixing proportions, especially with many nuisance candidate sources and unknown-source contributions up to 90%.

## When NOT to use

- When the candidate source set is small and fully covers all potential environments — the benchmark targets source exploration with many nuisance sources; existing methods are suboptimal mainly when the number of contributing sources is much smaller than candidates considered
- When you need accurate source tracking on real (non-simulated) sinks without ground-truth mixing proportions — the metrics here require known convex-mixture ground truth
- When the analysis goal is a single method's application rather than head-to-head method comparison against FEAST, SourceTracker2, and RAD

## Inputs

- Earth Microbiome Project (EMP) source environment samples (count matrices)
- Simulated sink communities as convex mixtures of K=10 true sources from M=50 candidates, with unknown-source contribution 0-90%
- FEAST/STENSL input files: m-by-n count matrix and m-by-3 metadata table with columns 'Env', 'SourceSink', 'id'

## Outputs

- simulation_benchmark_results.csv reporting MSE of mixing proportions, false positive rate of identified sources, and absolute error of estimated unknown contribution for STENSL, SourceTracker2, FEAST, and RAD
- Significance test results for pairwise MSE comparisons (STENSL vs each competitor) with stated P-value thresholds
- Per-sink source contribution matrices (S1 by S2, rows summing to 1, including an unknown-source column)

## How to apply

Draw source environments from real EMP samples and simulate synthetic sinks as convex mixtures of K=10 contributing sources selected from M=50 candidate sources, varying the unknown-source contribution from 0% to 90% so each method faces the same ground truth. Run STENSL on each sink using its expectation-maximization algorithm with sparsity (L1-regularized Lasso source selection, lambda = 1e-6, chosen by cross-validation and grid-search over likelihood), and run SourceTracker2, FEAST, and RAD on the identical sink-source sets. For every method compute three metrics: the mean-squared error of estimated mixing proportions, the false positive rate of identified contributing sources, and the absolute error of the estimated unknown contribution. Assess significance of MSE differences between STENSL and each competitor (STENSL vs SourceTracker2 P < 9.13e-7, vs FEAST P < 1.01e-6, vs RAD P < 9.13e-7) and compile all metrics into a per-method results table validated against Table S1 and Fig 2. The rationale is that with hundreds of potential source environments, existing methods add considerable error, so the benchmark must stress-test both source selection accuracy and unknown-source estimation.

## Related tools

- **STENSL** (Method under test; expectation-maximization algorithm with sparsity (L1 Lasso source selection, lambda = 1e-6) for identifying contributing sources among many candidate environments) — https://github.com/cozygene/FEAST/tree/STENSL
- **FEAST** (Competitor method; Fast Expectation-mAximization microbial Source Tracking run on identical simulated sink-source sets) — https://github.com/cozygene/FEAST
- **SourceTracker2** (Competitor baseline source tracking method scored on the same simulation metrics)
- **RAD** (Competitor baseline source tracking method scored on the same simulation metrics)
- **R** (Implementation environment for FEAST/STENSL (R >= 3.4.4) with dependencies Rcpp, RcppArmadillo, vegan, dplyr, reshape2, gridExtra, ggplot2, ggthemes) — https://github.com/cozygene/FEAST

## Examples

```
devtools::install_github("cozygene/FEAST", ref = "STENSL"); FEAST_output <- FEAST(C = otus, metadata = metadata, different_sources_flag = 1, dir_path = "~/FEAST/Data_files/", outfile = "demo")
```

## Evaluation signals

- simulation_benchmark_results.csv reports all three metrics (MSE, FPR, unknown absolute error) for all four methods
- STENSL achieves significantly lower MSE than each competitor: P < 9.13e-7 vs SourceTracker2, P < 1.01e-6 vs FEAST, P < 9.13e-7 vs RAD
- STENSL is the only method that consistently estimates real source contributions even at unknown-source contributions up to 90%
- FEAST output matrix P has S1 rows (sinks) by S2 columns (sources including an unknown source), with each row summing to 1
- Results reproduce the qualitative pattern of Table S1 and Fig 2 of the STENSL paper

## Limitations

- Detailed simulation protocol is delegated to supplementary Text S1, not the main text, so exact simulation parameters may require consulting the supplementary material
- The Lasso hyperparameter lambda was fixed to 1e-6 in the authors' experiments without a reported sensitivity analysis
- Earth Microbiome Project experiment details are only described in supplementary Text S3; no software version is specified for STENSL/FEAST
- STENSL's code is a branch of the FEAST codebase, so results are tied to that implementation lineage rather than an independently versioned release

## Evidence

- [results] In EMP-based simulations with positive unknown contributions up to 90%, STENSL was the only method to consistently estimate real source contributions, achieving significantly lower mean-squared error: [evidence span withheld — anchor status: not_found]
- [abstract] an expectation-maximization algorithm with sparsity that enables the identification of contributing sources among a large set of potential microbial environments: "an expectation-maximization algorithm with sparsity that enables the identification of contributing sources among a large set of potential microbial environments"
- [discussion] we determine an optimal value for hyperparameter λ through cross-validation: "we determine an optimal value for hyperparameter λ through cross-validation"
- [readme] FEAST returns an S1 by S2 matrix P, where S1 is the number sinks and S2 is the number of sources (including an unknown source). Each row in matrix P sums to 1.: "FEAST returns an S1 by S2 matrix P, where S1 is the number sinks and S2 is the number of sources (including an unknown source)."
- [methods] install FEAST using devtools: [evidence span withheld — anchor status: not_found]

---
name: mixing-proportion-accuracy-evaluation
description: Use when simulated sinks with known ground-truth source mixing proportions
  and unknown-source contributions (0–90%) are available and you must compare source
  tracking methods on estimation accuracy.
when_to_use_negative:
- When sinks have no known ground-truth composition (real-world sinks), since MSE/FPR/unknown-error
  metrics require simulated truth
- When the goal is only to estimate contributions for a small pre-defined source set
  without nuisance candidates — the skill's value comes from many candidate sources
  (M=50) with K=10 true contributors
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: STENSL
  role: EM-based sparse source tracking method under evaluation; Lasso source selection
    with lambda = 1e-6, cross-validated
  repo: https://github.com/cozygene/FEAST/tree/STENSL
- name: FEAST
  role: Competitor method run on the identical simulated sink-source sets; also the
    codebase STENSL branches from
  repo: https://github.com/cozygene/FEAST
- name: SourceTracker2
  role: Competitor source tracking method in the benchmark
- name: RAD
  role: Competitor source tracking method in the benchmark
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/msystems.00995-21
    title: 'STENSL: Microbial Source Tracking with ENvironment SeLection'
  source_content_hash: sha256:2b383313e5bcbc5f0056e5e5196c25572ddd4f07095495499ace7fde2bb9ce03
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
yes_rate: 0.7692307692307693
n_claims: 13
---

# mixing-proportion-accuracy-evaluation

## Summary

A benchmarking skill that quantifies how accurately microbial source tracking methods (STENSL, FEAST, SourceTracker2, RAD) estimate source mixing proportions, using MSE of estimated proportions, false positive rate of identified contributing sources, and absolute error of the estimated unknown contribution on simulated sinks built from EMP samples.

## When to use

Use when simulated sinks with known ground-truth source mixing proportions and unknown-source contributions (0–90%) are available and you must compare source tracking methods on estimation accuracy.

## When NOT to use

- When sinks have no known ground-truth composition (real-world sinks), since MSE/FPR/unknown-error metrics require simulated truth
- When the goal is only to estimate contributions for a small pre-defined source set without nuisance candidates — the skill's value comes from many candidate sources (M=50) with K=10 true contributors

## Inputs

- Count matrix (m samples x n taxa) from Earth Microbiome Project source environments
- Metadata table with 'Env', 'SourceSink', 'id' columns marking sources vs simulated sinks
- Ground-truth mixing proportions for K=10 true sources among M=50 candidates, plus the known unknown-source fraction (0–90%)

## Outputs

- simulation_benchmark_results.csv with MSE, FPR, and unknown-contribution absolute error per method (STENSL, FEAST, SourceTracker2, RAD)
- S1 x S2 mixing-proportion matrix P (rows sum to 1, including an unknown source column) per method
- P-value summary of MSE significance comparisons (e.g. STENSL vs FEAST P < 1.01e-6)

## How to apply

Draw source environments from Earth Microbiome Project samples and simulate synthetic sinks as convex mixtures of K=10 true contributing sources selected from M=50 candidates, varying the unknown-source contribution from 0% to 90%. Run each method (STENSL via its expectation-maximization algorithm with L1-regularized Lasso source selection at lambda = 1e-6, cross-validated; competitors FEAST, SourceTracker2, and RAD) on identical simulated sink-source sets. For each method compute the MSE of estimated mixing proportions, the false positive rate of identified contributing sources, and the absolute error of the estimated unknown contribution, then assess significance of MSE differences (STENSL vs SourceTracker2 P < 9.13e-7, vs FEAST P < 1.01e-6, vs RAD P < 9.13e-7) and compile per-method metrics into a results table (e.g. simulation_benchmark_results.csv). Validation succeeds if the table reports all three metrics for all four methods and STENSL's MSE is significantly lower than each competitor at the stated P-value thresholds (cf. Table S1, Fig 2).

## Related tools

- **STENSL** (EM-based sparse source tracking method under evaluation; Lasso source selection with lambda = 1e-6, cross-validated) — https://github.com/cozygene/FEAST/tree/STENSL
- **FEAST** (Competitor method run on the identical simulated sink-source sets; also the codebase STENSL branches from) — https://github.com/cozygene/FEAST
- **SourceTracker2** (Competitor source tracking method in the benchmark)
- **RAD** (Competitor source tracking method in the benchmark)

## Examples

```
FEAST_output <- FEAST(C = otus, metadata = metadata, different_sources_flag = 1, dir_path = "~/FEAST/Data_files/", outfile="demo")
```

## Evaluation signals

- simulation_benchmark_results.csv contains all three metrics (MSE, FPR, unknown AE) for all four methods
- STENSL MSE is significantly lower than SourceTracker2 (P < 9.13e-7), FEAST (P < 1.01e-6), and RAD (P < 9.13e-7)
- STENSL remains the only method that consistently estimates real source contributions even at 90% unknown-source contribution
- Per-sink FEAST/STENSL output matrix P has rows summing to 1 and includes an unknown source column
- FPR of identified contributing sources is reported alongside MSE, showing reduced noise from noncontributing sources

## Limitations

- STENSL/FEAST software versions are unspecified; simulation protocol details are delegated to supplementary Text S1 and are not in the main text
- The Lasso hyperparameter was fixed at lambda = 1e-6 without a reported sensitivity analysis in the main text
- The STENSL code is a branch of the FEAST codebase, so installation dependencies (R >= 3.4.4, Rcpp, RcppArmadillo, vegan, dplyr, reshape2, gridExtra, ggplot2, ggthemes) apply and can complicate setup, especially on macOS

## Evidence

- [abstract] STENSL is significantly more accurate in identifying the contributing sources as well as the unknown source, even when considering hundreds of potential source environments: "STENSL is significantly more accurate in identifying the contributing sources as well as the unknown source, even when considering hundreds of potential source environments"
- [discussion] Cross-validation for lambda: "we determine an optimal value for hyperparameter λ through cross-validation"
- [discussion] Sparse Lasso source selection: "formulating a least-squares problem between x and YTα while ensuring that α is sparsely approximated by L1 regularization"
- [readme] FEAST output matrix schema: "*FEAST* returns an S1 by S2 matrix P, where S1 is the number sinks and S2 is the number of sources (including an unknown source). Each row in matrix P sums to 1."

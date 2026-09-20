---
name: sparse-source-selection-em-estimation
description: Use when a sink microbial community must be decomposed into mixing proportions
  over many candidate source environments (e.g., M=50 EMP sources) where only a few
  truly contribute and an unobserved unknown source may account for up to 90% of the
  sink.
when_to_use_negative:
- The full set of potential sources is small and already curated for a single study,
  so nuisance-source noise is not a concern and standard FEAST suffices
- The sink is known a priori to be fully explained by observed sources with essentially
  zero unknown contribution and no source selection is needed
- Input is raw sequencing reads rather than a compositional count table; STENSL expects
  count profiles, not FASTQ
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: STENSL
  role: Primary EM-with-sparsity algorithm for source selection and mixing-proportion
    estimation; implemented as a branch of the FEAST codebase
  repo: https://github.com/cozygene/FEAST/tree/STENSL
- name: FEAST
  role: Baseline competitor (Fast Expectation-mAximization microbial Source Tracking)
    run on identical simulated sink-source sets for comparison, and the codebase STENSL
    branches from
  repo: https://github.com/cozygene/FEAST
- name: SourceTracker2
  role: Baseline microbial source tracking competitor in the simulation benchmark
- name: R
  role: Implementation environment; FEAST/STENSL installed via devtools with Rcpp,
    RcppArmadillo, vegan, dplyr, reshape2, gridExtra, ggplot2, ggthemes dependencies
  repo: https://github.com/cozygene/FEAST
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/msystems.00995-21
    title: 'STENSL: Microbial Source Tracking with ENvironment SeLection'
  source_content_hash: sha256:10f788ba8daf1abe5e41b3a2a697fe6994d8dbb3ac98519880f0b0acd605a1d6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FEAST
  - R
  - STENSL
  - SourceTracker2
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# sparse-source-selection-em-estimation

## Summary

Applies STENSL, an expectation-maximization algorithm with sparsity (L1-regularized Lasso source selection), to estimate microbial source mixing proportions and identify contributing sources among a large pool of nuisance candidate sources while explicitly modeling an unknown source.

## When to use

Use when a sink microbial community must be decomposed into mixing proportions over many candidate source environments (e.g., M=50 EMP sources) where only a few truly contribute and an unobserved unknown source may account for up to 90% of the sink.

## When NOT to use

- The full set of potential sources is small and already curated for a single study, so nuisance-source noise is not a concern and standard FEAST suffices
- The sink is known a priori to be fully explained by observed sources with essentially zero unknown contribution and no source selection is needed
- Input is raw sequencing reads rather than a compositional count table; STENSL expects count profiles, not FASTQ

## Inputs

- Sink count vector x (taxon counts) from e.g. a simulated EMP-based sink
- Source count matrix Y (taxa by M candidate source environments, e.g. M=50 EMP environments)
- Lasso hyperparameter lambda (e.g. 1e-6, chosen by cross-validation)
- Number of truly contributing sources K (e.g. K=10 in the simulation benchmark)

## Outputs

- Vector of estimated source mixing proportions alpha over the M candidate sources plus the unknown source
- Sparse set of identified contributing sources (via L1/EM source selection)
- Estimated unknown-source contribution proportion
- Benchmark metrics per method: MSE of mixing proportions, false positive rate of identified sources, absolute error of estimated unknown contribution (simulation_benchmark_results.csv)

## How to apply

Model the sink as a convex combination of contributing sources, nuisance (noncontributing) sources, and an unobserved unknown source. For a fixed L1 hyperparameter lambda, run expectation-maximization to infer the mixing proportions (alpha) and source-selection indicators (gamma); determine lambda via cross-validation (the paper used lambda = 1e-6). Initialize the unknown source by subtracting the scaled counts of the top contributing source ranked by the L1 regularization: Y^M+1 = max(0, x - alpha^Y_argmax). Optionally grid-search lambda across a range and return the parameter set with the highest likelihood. Compute mixing-proportion MSE, false positive rate of identified contributing sources, and unknown-contribution absolute error against known simulated ground truth to validate the run.

## Related tools

- **STENSL** (Primary EM-with-sparsity algorithm for source selection and mixing-proportion estimation; implemented as a branch of the FEAST codebase) — https://github.com/cozygene/FEAST/tree/STENSL
- **FEAST** (Baseline competitor (Fast Expectation-mAximization microbial Source Tracking) run on identical simulated sink-source sets for comparison, and the codebase STENSL branches from) — https://github.com/cozygene/FEAST
- **SourceTracker2** (Baseline microbial source tracking competitor in the simulation benchmark)
- **R** (Implementation environment; FEAST/STENSL installed via devtools with Rcpp, RcppArmadillo, vegan, dplyr, reshape2, gridExtra, ggplot2, ggthemes dependencies) — https://github.com/cozygene/FEAST

## Examples

```
devtools::install_github("cozygene/FEAST", ref = "STENSL"); FEAST_output <- FEAST(C = otus, metadata = metadata, different_sources_flag = 1, dir_path = "~/FEAST/Data_files/", outfile="demo")
```

## Evaluation signals

- STENSL mixing-proportion MSE is significantly lower than SourceTracker2 (P < 9.13e-7), FEAST (P < 1.01e-6), and RAD (P < 9.13e-7) in EMP-based simulations
- STENSL remains the only method to consistently estimate real source contributions even with positive unknown contributions up to 90%
- Estimated proportions sum to 1 over the M candidate sources plus the unknown source (each row of the output matrix sums to 1)
- False positive rate of identified contributing sources stays low across the M=50 candidate pool with K=10 true contributors
- Cross-validated lambda yields the highest-likelihood parameter set over the grid-search range

## Limitations

- Lasso hyperparameter was fixed to a single value (lambda = 1e-6) in the paper's experiments without a reported sensitivity analysis
- Detailed simulation protocol and EMP experiment details are only in supplementary Texts S1 and S3, not the main text
- No software version is specified for STENSL/FEAST, and the code is a branch of the FEAST repo with license listed as NOASSERTION

## Evidence

- [abstract] we developed STENSL (microbial Source Tracking with ENvironment SeLection), an expectation-maximization algorithm with sparsity that enables the identification of contributing sources: "we developed STENSL (microbial Source Tracking with ENvironment SeLection), an expectation-maximization algorithm with sparsity"
- [discussion] For a fixed lambda, we use expectation-maximization to infer parameters alpha and gamma.: "For a fixed λ, we use expectation-maximization to infer parameters α and γ."
- [discussion] we determine an optimal value for hyperparameter lambda through cross-validation: "we determine an optimal value for hyperparameter λ through cross-validation"
- [discussion] In our experiments, we chose λ = 1 × 10−6: "In our experiments, we chose λ = 1 × 10−6"
- [abstract] STENSL is significantly more accurate in identifying the contributing sources as well as the unknown source, even when considering hundreds of potential source environments: "STENSL is significantly more accurate in identifying the contributing sources as well as the unknown source"
- [discussion] The code for STENSL is a branch of the FEAST codebase, which can be found on https://github.com/cozygene/FEAST/tree/STENSL: "The code for STENSL is a branch of the FEAST codebase"
- [readme] FEAST will save the file "demo_FEAST.txt" (a file containing matrix P).: "FEAST will save the file "demo_FEAST.txt" (a file containing matrix P)"
- [readme] Each row in matrix P sums to 1. Pij is the contribution of source j to sink i.: "Each row in matrix P sums to 1. Pij is the contribution of source j to sink i"

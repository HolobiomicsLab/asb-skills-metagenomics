---
name: microbiome-source-tracking-simulation
description: Use when evaluating or comparing source-tracking algorithms on compositional
  data with many candidate sources (e.g., M=50 environments, K=10 true contributors)
  and substantial unknown-source mixing (0–90%), or when validating that a new MST
  method actually reduces noise from noncontributing sources.
when_to_use_negative:
- The candidate-source pool is small and fully enumerated with no unknown source —
  the sparsity/nuisance-source advantage of STENSL is irrelevant and standard FEAST
  suffices.
- The goal is running source tracking on real (not simulated) sinks; this skill is
  a benchmarking protocol, not a production estimation pipeline.
edam_topics:
- http://edamontology.org/topic_0621
tools:
- name: STENSL
  role: Method under test; EM algorithm with sparsity (L1/Lasso source selection)
    that identifies contributing sources and estimates the unknown contribution
  repo: https://github.com/cozygene/FEAST/tree/STENSL
- name: FEAST
  role: Competitor method; also the codebase STENSL branches from, invoked on identical
    simulated sink-source sets
  repo: https://github.com/cozygene/FEAST
- name: SourceTracker2
  role: Competitor method run on the same simulated sink-source sets
- name: R
  role: Implementation environment (R >= 3.4.4) with Rcpp/RcppArmadillo, vegan, dplyr,
    reshape2, gridExtra, ggplot2, ggthemes dependencies
  repo: https://github.com/cozygene/FEAST
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/msystems.00995-21
    title: 'STENSL: Microbial Source Tracking with ENvironment SeLection'
  source_content_hash: sha256:482ec2c26e9966171e61b5f5e165006b955f0defdd26756c8f09028114efddf4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FEAST
  - R
  - STENSL
  - SourceTracker2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# microbiome-source-tracking-simulation

## Summary

Benchmark microbial source tracking methods (STENSL, SourceTracker2, FEAST, RAD) on synthetic sinks built from real Earth Microbiome Project samples, scoring mean-squared error of mixing proportions, false-positive source rate, and unknown-contribution absolute error. Rationale: existing MST methods accumulate error as the pool of candidate sources grows, so controlled simulations with nuisance sources and large unknown fractions are needed to expose that failure mode.

## When to use

Use when evaluating or comparing source-tracking algorithms on compositional data with many candidate sources (e.g., M=50 environments, K=10 true contributors) and substantial unknown-source mixing (0–90%), or when validating that a new MST method actually reduces noise from noncontributing sources.

## When NOT to use

- The candidate-source pool is small and fully enumerated with no unknown source — the sparsity/nuisance-source advantage of STENSL is irrelevant and standard FEAST suffices.
- The goal is running source tracking on real (not simulated) sinks; this skill is a benchmarking protocol, not a production estimation pipeline.

## Inputs

- Earth Microbiome Project source environment samples (compositional count data)
- Synthetic sinks simulated as convex mixtures of K=10 sources from M=50 candidates with 0–90% unknown contribution
- Sink-source metadata in FEAST format: three-column table (Env, SourceSink, id) plus an m-by-n count matrix
- Installed methods: STENSL (FEAST STENSL branch), SourceTracker2, FEAST, RAD

## Outputs

- simulation_benchmark_results.csv with per-method MSE, false-positive rate, and unknown-contribution absolute error
- P-values for STENSL vs each competitor (thresholds 9.13e-7, 1.01e-6, 9.13e-7)
- Estimated S1-by-S2 contribution matrix P (rows sum to 1, including an unknown-source column)

## How to apply

Draw source environments from Earth Microbiome Project samples and simulate synthetic sinks as convex mixtures of K=10 contributing sources selected from M=50 candidate sources, sweeping the unknown-source contribution from 0% to 90%. Run STENSL on each sink via its expectation-maximization algorithm with L1-regularized Lasso source selection (lambda = 1e-6, chosen by cross-validation over a grid that returns the parameters with the highest likelihood). Run SourceTracker2, FEAST, and RAD on identical sink-source sets as competitors. For each method compute MSE of estimated mixing proportions, false-positive rate of identified contributing sources, and absolute error of the estimated unknown contribution; assess significance of MSE differences (STENSL vs SourceTracker2 P < 9.13e-7, vs FEAST P < 1.01e-6, vs RAD P < 9.13e-7) and compile metrics into simulation_benchmark_results.csv.

## Related tools

- **STENSL** (Method under test; EM algorithm with sparsity (L1/Lasso source selection) that identifies contributing sources and estimates the unknown contribution) — https://github.com/cozygene/FEAST/tree/STENSL
- **FEAST** (Competitor method; also the codebase STENSL branches from, invoked on identical simulated sink-source sets) — https://github.com/cozygene/FEAST
- **SourceTracker2** (Competitor method run on the same simulated sink-source sets)
- **R** (Implementation environment (R >= 3.4.4) with Rcpp/RcppArmadillo, vegan, dplyr, reshape2, gridExtra, ggplot2, ggthemes dependencies) — https://github.com/cozygene/FEAST

## Examples

```
FEAST_output <- FEAST(C = otus, metadata = metadata, different_sources_flag = 1, dir_path = "~/FEAST/Data_files/", outfile="demo")
```

## Evaluation signals

- Output table must report all three metrics (MSE, FPR, unknown absolute error) for all four methods
- STENSL MSE must be significantly lower than each competitor at the stated P-value thresholds (Table S1, Fig 2)
- STENSL remains the only method that consistently estimates real source contributions even at 90% unknown contribution
- Every row of the estimated contribution matrix P sums to 1 and includes an unknown-source column
- STENSL's identified contributing sources show low false-positive rate against the known K=10 ground truth

## Limitations

- Detailed simulation protocol is delegated to supplementary Text S1, not the main text; exact generative parameters must be taken from SI.
- The Lasso hyperparameter is fixed to lambda = 1e-6 without a reported sensitivity analysis.
- No software version is specified for STENSL/FEAST; STENSL code is a branch of the FEAST codebase and installation requires manual devtools install from GitHub.
- Existing (non-STENSL) methods add considerable error as the number of considered sources increases, so competitor metrics will degrade with large M by design.

## Evidence

- [abstract] STENSL accuracy claim: "STENSL is significantly more accurate in identifying the contributing sources as well as the unknown source, even when considering hundreds of potential source environments"
- [abstract] Sparsity rationale: "By incorporating sparsity into the estimation of potential source environments, STENSL improves the accuracy of true source contribution"
- [discussion] Lambda selection procedure: "we determine an optimal value for hyperparameter λ through cross-validation"
- [readme] Output contract for contribution matrix: "FEAST returns an S1 by S2 matrix P, where S1 is the number sinks and S2 is the number of sources (including an unknown source). Each row in matrix P sums to 1."

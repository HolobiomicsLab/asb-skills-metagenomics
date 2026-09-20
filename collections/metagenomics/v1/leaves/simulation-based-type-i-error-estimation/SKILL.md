---
name: simulation-based-type-i-error-estimation
description: 'Use when validating a new or competing multivariate association test
  on simulated count data: you need empirical type I error under β = 0 and empirical
  power under alternatives varying signal density (10%, 20%, 30%) and effect size.'
when_to_use_negative:
- The input is real (not simulated) association data with a tested hypothesis — use
  permutation-based significance testing with the full 10,000 permutations rather
  than simulation-based error calibration.
- The goal is taxonomic profiling or OTU clustering (e.g. PAM clustering by cophenetic
  distance) rather than evaluating statistical test calibration.
- Only one method's raw P-value is needed per replicate with no need for empirical
  calibration checks across scenarios.
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: AMAT
  role: The multivariate association test under evaluation; applied with distance-correlation-based
    feature selection and a data-adaptive GLM test to each simulated replicate, with
    B = 500 permutations and Γ = {2,3,4,8}.
  repo: https://github.com/kzb193/AMAT
- name: R
  role: Computing environment in which the simulations, replicate generation, and
    AMAT applications are implemented.
  repo: https://github.com/kzb193/AMAT
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqab120
    title: Adaptive and powerful microbiome multivariate association analysis via
      feature selection
  source_content_hash: sha256:e4889a3fb42c27f8f5d42d3da4c994d9b3a73798e2d633b5a2203a92c087885a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/kzb193/AMAT
  repo_url: https://github.com/kzb193/AMAT
  tools:
  - AMAT
  - R
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# simulation-based-type-i-error-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Empirically estimates the type I error rate of a multivariate microbiome association test (here AMAT) by generating replicate null datasets (β = 0) from Dirichlet-multinomial OTU simulations modeled on a real 856-OTU throat microbiome, then computing the proportion of rejections at the nominal 0.05 level.

## When to use

Use when validating a new or competing multivariate association test on simulated count data: you need empirical type I error under β = 0 and empirical power under alternatives varying signal density (10%, 20%, 30%) and effect size.

## When NOT to use

- The input is real (not simulated) association data with a tested hypothesis — use permutation-based significance testing with the full 10,000 permutations rather than simulation-based error calibration.
- The goal is taxonomic profiling or OTU clustering (e.g. PAM clustering by cophenetic distance) rather than evaluating statistical test calibration.
- Only one method's raw P-value is needed per replicate with no need for empirical calibration checks across scenarios.

## Inputs

- OTU count table from the real reference dataset (throat microbiome, 856 OTUs × 60 samples) used to estimate multinomial relative abundances
- Simulation design parameters: outcome type (continuous via linear model, binary via logistic model), β (0 for null; alternatives vary), signal density (10%, 20%, 30%), effect size
- Test settings: number of permutations (B = 500 for simulation replicates; the article used 10,000 permutations for the real-data analyses), tuning parameter candidates Γ = {2,3,4,8}

## Outputs

- Empirical type I error rate per scenario (proportion of null replicates rejected at nominal 0.05)
- Empirical power per alternative scenario
- Aggregated CSV table (simulation_results.csv) with rows per scenario and columns for outcome type, scenario parameters, empirical type I error, and empirical power

## How to apply

Simulate OTU count tables mimicking the real throat microbiome (856 OTUs, 60 samples) by drawing counts from a multinomial distribution with the observed relative abundances, under both a linear (continuous outcome) and a logistic (binary outcome) model. Generate replicate datasets under the null (β = 0) and under alternatives varying signal density (10%, 20%, 30%) and effect size. Apply AMAT to each replicate — distance-correlation-based feature selection followed by the data-adaptive GLM-based test — with B = 500 permutations and tuning candidates Γ = {2,3,4,8} in R. Compute empirical type I error as the proportion of null replicates rejected at nominal 0.05, and empirical power for alternative replicates, then aggregate rows per scenario and outcome type into a CSV (e.g. simulation_results.csv). The rationale is that well-calibrated tests should reject at approximately the nominal rate under the null; in the article all tests were well calibrated except MiHC, which was conservative especially with binary outcomes.

## Related tools

- **AMAT** (The multivariate association test under evaluation; applied with distance-correlation-based feature selection and a data-adaptive GLM test to each simulated replicate, with B = 500 permutations and Γ = {2,3,4,8}.) — https://github.com/kzb193/AMAT
- **R** (Computing environment in which the simulations, replicate generation, and AMAT applications are implemented.) — https://github.com/kzb193/AMAT

## Evaluation signals

- Each empirical type I error rate falls within the binomial 95% interval around the nominal 0.05 level.
- All reported power and type I error values lie in [0, 1].
- Null-scenario rejection proportions for AMAT approximate 0.05 across both continuous and binary outcome configurations.
- MiHC, if included as a comparator, is expected to show conservative behavior (type I error below nominal), especially with binary outcomes — deviations elsewhere flag implementation errors.
- The output CSV has one row per scenario with non-missing outcome type, scenario parameters, type I error, and power columns.

## Limitations

- Type I error estimates are Monte Carlo quantities: with limited replicates the binomial uncertainty around 0.05 is non-trivial, so tests may appear miscalibrated by chance.
- Simulated counts are drawn from a multinomial distribution fitted to a single reference community (throat microbiome, 856 OTUs); calibration results may not transfer to other communities or other count-generating models.
- Even well-calibrated tests can be misjudged if the number of permutations per replicate (B = 500 here) is too small for fine-grained P-value resolution.

## Evidence

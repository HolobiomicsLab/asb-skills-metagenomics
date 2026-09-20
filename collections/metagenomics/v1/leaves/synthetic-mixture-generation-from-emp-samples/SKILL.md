---
name: synthetic-mixture-generation-from-emp-samples
description: Use when you need simulated source-sink microbiome data with known ground-truth
  mixing proportions — e.g., testing whether a source tracking method can recover
  contributions when many nuisance candidate sources (M=50) and unknown fractions
  (0–90%) are present.
when_to_use_negative:
- You have real, naturally formed sink communities whose true source contributions
  cannot be validated — this skill only provides synthetic mixtures with known ground
  truth
- You only have a single or few candidate sources without nuisance candidates — the
  benchmark targets scenarios with many candidate sources (M=50) where state-of-the-art
  methods add considerable error
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: STENSL
  role: Source tracking method under benchmark; expectation-maximization with sparsity
    used on simulated sinks
  repo: https://github.com/cozygene/FEAST/tree/STENSL
- name: FEAST
  role: Competitor method run on identical simulated sink-source sets; STENSL code
    is a branch of the FEAST codebase
  repo: https://github.com/cozygene/FEAST
- name: SourceTracker2
  role: Competitor source tracking method run on the same simulated mixtures
- name: RAD
  role: Competitor source tracking method run on the same simulated mixtures
- name: R
  role: Implementation environment for FEAST/STENSL and benchmark scripts
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/msystems.00995-21
    title: 'STENSL: Microbial Source Tracking with ENvironment SeLection'
  source_content_hash: sha256:605636b0ae56680fae5e58658f375b89ba52452ad4fc4d8ac8e1c85d0c1821b2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FEAST
  - R
  - RAD
  - STENSL
  - SourceTracker2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# synthetic-mixture-generation-from-emp-samples

## Summary

Generate synthetic microbial sinks as convex mixtures of Earth Microbiome Project samples to benchmark microbial source tracking methods (STENSL vs FEAST, SourceTracker2, RAD) with controlled unknown-source contributions.

## When to use

Use when you need simulated source-sink microbiome data with known ground-truth mixing proportions — e.g., testing whether a source tracking method can recover contributions when many nuisance candidate sources (M=50) and unknown fractions (0–90%) are present.

## When NOT to use

- You have real, naturally formed sink communities whose true source contributions cannot be validated — this skill only provides synthetic mixtures with known ground truth
- You only have a single or few candidate sources without nuisance candidates — the benchmark targets scenarios with many candidate sources (M=50) where state-of-the-art methods add considerable error

## Inputs

- Earth Microbiome Project source environment samples (count matrices)
- Candidate source pool of M=50 environments
- Simulated sink specification: K=10 contributing sources and target unknown-source contribution fraction (0% to 90%)
- Count matrix (m samples by n taxa) and 3-column metadata table ('Env', 'SourceSink', 'id') in FEAST/STENSL format

## Outputs

- Synthetic sink-source count matrices with known ground-truth mixing proportions
- Per-method metrics: MSE of estimated mixing proportions, false positive rate of contributing-source identification, absolute error of estimated unknown contribution
- Compiled benchmark table (simulation_benchmark_results.csv) covering all metrics for STENSL, FEAST, SourceTracker2, and RAD

## How to apply

Draw source environments from Earth Microbiome Project samples and construct synthetic sinks as convex mixtures of K=10 contributing sources selected from M=50 candidate sources, varying the unknown-source contribution from 0% to 90%. Provide the identical simulated sink-source count matrix and metadata (Env, SourceSink, id columns) to each method being compared — STENSL (expectation-maximization with L1-regularized Lasso source selection, cross-validated lambda, e.g. 1e-6), FEAST, SourceTracker2, and RAD. For each method, compute the MSE of estimated mixing proportions, the false positive rate of identified contributing sources, and the absolute error of the estimated unknown contribution. Compile per-method metrics into a results table (e.g., simulation_benchmark_results.csv) and assess significance of MSE differences against stated thresholds.

## Related tools

- **STENSL** (Source tracking method under benchmark; expectation-maximization with sparsity used on simulated sinks) — https://github.com/cozygene/FEAST/tree/STENSL
- **FEAST** (Competitor method run on identical simulated sink-source sets; STENSL code is a branch of the FEAST codebase) — https://github.com/cozygene/FEAST
- **SourceTracker2** (Competitor source tracking method run on the same simulated mixtures)
- **RAD** (Competitor source tracking method run on the same simulated mixtures)
- **R** (Implementation environment for FEAST/STENSL and benchmark scripts)

## Examples

```
metadata <- Load_metadata(metadata_path = "~/FEAST/Data_files/metadata_example_multi.txt"); otus <- Load_CountMatrix(CountMatrix_path = "~/FEAST/Data_files/otu_example_multi.txt"); FEAST_output <- FEAST(C = otus, metadata = metadata, different_sources_flag = 1, dir_path = "~/FEAST/Data_files/", outfile="demo")
```

## Evaluation signals

- All three metrics (mixing-proportion MSE, contributing-source false positive rate, unknown-contribution absolute error) are reported for all four methods
- STENSL MSE is significantly lower than each competitor at stated thresholds (vs SourceTracker2 P < 9.13e-7, vs FEAST P < 1.01e-6, vs RAD P < 9.13e-7)
- Estimated mixing proportion vectors sum to 1, matching the convex-mixture construction of the synthetic sinks
- Performance is evaluated across the full gradient of unknown-source contributions from 0% to 90%
- Methods with hundreds of candidate sources should not show increasing error for STENSL, unlike state-of-the-art baselines

## Limitations

- The detailed simulation protocol is delegated to Supplementary Text S1 and is not included in the main document
- The Lasso hyperparameter was fixed to a single value (lambda = 1e-6) without a reported sensitivity analysis
- Earth Microbiome Project experiment details are delegated to supplementary Text S3

## Evidence

- [methods] Draw source environments from Earth Microbiome Project samples and simulate synthetic sinks as convex mixtures of K=10 contributing sources selected from M=50 candidate sources, varying the unknown-source contribution from 0% to 90%.: [evidence span withheld — anchor status: not_found]
- [abstract] settings in which state-of-the-art microbial source tracking methods add considerable error: "settings in which state-of-the-art microbial source tracking methods add considerable error"
- [abstract] STENSL is significantly more accurate in identifying the contributing sources as well as the unknown source, even when considering hundreds of potential source environments: "STENSL is significantly more accurate in identifying the contributing sources as well as the unknown source, even when considering hundreds of potential source environments"
- [supplementary] For a detailed description of the simulation, see Supplementary Material.: "For a detailed description of the simulation, see Supplementary Material."
- [readme] _C_ - An _m_ by _n_ count matrix, where _m_ is the number samples and _n_ is the number of taxa.: "_C_ - An _m_ by _n_ count matrix, where _m_ is the number samples and _n_ is the number of taxa."

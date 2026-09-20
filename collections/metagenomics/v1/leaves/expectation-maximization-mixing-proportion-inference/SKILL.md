---
name: expectation-maximization-mixing-proportion-inference
description: Use when an m-by-n microbiome count matrix (rows = SampleID, taxa as
  columns) plus 3-column metadata (Env, SourceSink, id) must be decomposed into per-sink
  source contributions, especially with many noncontributing sources or an unknown
  source.
when_to_use_negative:
- When the true number of candidate sources is small and all are known contributors
  — plain FEAST suffices and STENSL's sparsity/unknown-source machinery is unnecessary
  overhead
- When the goal is source exploration with an unknown source using Aitchison-distance
  preselection only — the article shows this strategy has very low accuracy in the
  presence of an unknown source
- When inputs are not count tables in the required SampleID-by-taxa, 3-column metadata
  format — column order and 'Env'/'SourceSink'/'id' names must be respected
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: FEAST
  role: Core EM engine that takes C, metadata, EM_iterations, COVERAGE, different_sources_flag
    and returns the mixing-proportion matrix P
  repo: github.com/cozygene/FEAST
- name: STENSL
  role: Sparse branch of the FEAST codebase that adds L1-regularized source selection,
    cross-validated lambda, and unknown-source initialization
  repo: github.com/cozygene/FEAST/tree/STENSL
- name: devtools
  role: Used to install FEAST from GitHub (ref = "FEAST")
  repo: github.com/cozygene/FEAST
- name: Rcpp
  role: Dependency required before installing FEAST
- name: RcppArmadillo
  role: Dependency required before installing FEAST
- name: PlotSourceContribution
  role: Barplot visualization of the returned mixing proportions per sink
  repo: github.com/cozygene/FEAST
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1128/msystems.00995-21
    title: 'STENSL: Microbial Source Tracking with ENvironment SeLection'
  source_content_hash: sha256:87fb823735ea454b3b6436f943c8af84184a6709eb216ee65c0039b93c9bbbd9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FEAST
  - PlotSourceContribution
  - R
  - Rcpp
  - RcppArmadillo
  - STENSL
  - SourceTracker2
  - devtools
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6842105263157895
n_claims: 19
---

# expectation-maximization-mixing-proportion-inference

## Summary

Estimate microbial source-to-sink mixing proportions with FEAST/STENSL, expectation-maximization algorithms that model a sink community as a convex combination of contributing, nuisance, and unknown sources. STENSL adds L1-regularized sparsity so accuracy holds even with hundreds of candidate sources.

## When to use

Use when an m-by-n microbiome count matrix (rows = SampleID, taxa as columns) plus 3-column metadata (Env, SourceSink, id) must be decomposed into per-sink source contributions, especially with many noncontributing sources or an unknown source.

## When NOT to use

- When the true number of candidate sources is small and all are known contributors — plain FEAST suffices and STENSL's sparsity/unknown-source machinery is unnecessary overhead
- When the goal is source exploration with an unknown source using Aitchison-distance preselection only — the article shows this strategy has very low accuracy in the presence of an unknown source
- When inputs are not count tables in the required SampleID-by-taxa, 3-column metadata format — column order and 'Env'/'SourceSink'/'id' names must be respected

## Inputs

- m-by-n tab-delimited count matrix with SampleID row names and taxa-id columns
- m-by-3 metadata table with columns 'Env', 'SourceSink' ('Source'/'Sink'), 'id'
- numeric EM_iterations (default 1000), COVERAGE (default: minimal sequencing depth per sink-source group), different_sources_flag (0/1)

## Outputs

- S1-by-S2 matrix P of per-sink source contributions including an unknown source; each row sums to 1; Pij = NA indicates source j not used for sink i
- output file '<outfile>_FEAST.txt' containing matrix P

## How to apply

Install FEAST from GitHub via devtools::install_github("cozygene/FEAST", ref = "FEAST") after loading Rcpp, RcppArmadillo, and vegan. Load the count table with Load_CountMatrix and the metadata with Load_metadata, ensuring the metadata columns are exactly 'Env', 'SourceSink', 'id', where sources carry blank ids and sinks get ids 1..number of sinks. Call FEAST(C = otus, metadata = metadata, EM_iterations = 1000, different_sources_flag = 1, outfile = ...) to obtain an S1-by-S2 matrix P (rows sum to 1, includes an unknown source column); Pij = NA means source j was unused for sink i. STENSL, the sparse branch of the same codebase, additionally performs Lasso-based source selection via a quadratic program, cross-validation to pick lambda, a grid search over lambda choosing highest-likelihood EM parameters, and initializes the unknown source by subtracting scaled counts of the top contributing source. Rarefy uneven depths with FEAST_rarefy(x, maxdepth) to the minimal depth within each sink-source group before running.

## Related tools

- **FEAST** (Core EM engine that takes C, metadata, EM_iterations, COVERAGE, different_sources_flag and returns the mixing-proportion matrix P) — github.com/cozygene/FEAST
- **STENSL** (Sparse branch of the FEAST codebase that adds L1-regularized source selection, cross-validated lambda, and unknown-source initialization) — github.com/cozygene/FEAST/tree/STENSL
- **devtools** (Used to install FEAST from GitHub (ref = "FEAST")) — github.com/cozygene/FEAST
- **Rcpp** (Dependency required before installing FEAST)
- **RcppArmadillo** (Dependency required before installing FEAST)
- **PlotSourceContribution** (Barplot visualization of the returned mixing proportions per sink) — github.com/cozygene/FEAST

## Examples

```
FEAST_output <- FEAST(C = otus, metadata = metadata, EM_iterations = 1000, different_sources_flag = 1, dir_path = "~/FEAST/Data_files/", outfile="demo")
```

## Evaluation signals

- Each row of the output matrix P sums to exactly 1 and contains S2 columns including an 'unknown' source
- NA entries in P must correspond to sources excluded from a given sink's analysis, not zero contributions
- On in vitro human/mouse gut mixtures with 50 noncontributing sources (unknown proportions 0%–~80%), STENSL MSE should rank best against lab ground truth, with Wilcoxon p-values below reported thresholds (SourceTracker2 P < 2.2e-16, FEAST P < 2.1e-16, RAD P < 2.8e-16)
- Reproducing the README demo should yield proportions matching the documented output row (e.g., infant gut 2: ~0.51 for Adult gut 1, ~0.43 unknown)
- Cross-validation should return a lambda grid value whose EM parameters attain the highest likelihood

## Limitations

- Estimation error increases as the number of considered sources grows for existing methods (FEAST, SourceTracker2); STENSL addresses this but requires its sparse branch, not default FEAST
- No software version specified for STENSL/FEAST; Lasso hyperparameter was fixed to lambda = 1e-6 without reported sensitivity analysis
- Requires rarefaction via FEAST_rarefy to a common depth (default minimal sequencing depth within each sink-source group) when depths differ
- Mac users may hit Rcpp/RcppArmadillo build issues; README advises installing homebrew or xcode first

## Evidence

- [abstract] we developed STENSL (microbial Source Tracking with ENvironment SeLection), an expectation-maximization algorithm with sparsity that enables the identification of contributing sources: "we developed STENSL (microbial Source Tracking with ENvironment SeLection), an expectation-maximization algorithm with sparsity that enables the identification of contributing sources"
- [discussion] The code for STENSL is a branch of the FEAST codebase: "The code for STENSL is a branch of the FEAST codebase, which can be found on https://github.com/cozygene/FEAST/tree/STENSL"
- [discussion] formulating a least-squares problem between x and YTα while ensuring that α is sparsely approximated by L1 regularization: "formulating a least-squares problem between x and YTα while ensuring that α is sparsely approximated by L1 regularization"
- [discussion] we determine an optimal value for hyperparameter λ through cross-validation: "we determine an optimal value for hyperparameter λ through cross-validation"
- [readme] Value: FEAST returns an S1 by S2 matrix P, where S1 is the number sinks and S2 is the number of sources (including an unknown source). Each row in matrix P sums to 1.: "FEAST returns an S1 by S2 matrix P, where S1 is the number sinks and S2 is the number of sources (including an unknown source). Each row in matrix P sums to 1. Pij is the contribution of source j to"
- [methods] install FEAST using devtools: "devtools::install_github("cozygene/FEAST", ref = "FEAST")"

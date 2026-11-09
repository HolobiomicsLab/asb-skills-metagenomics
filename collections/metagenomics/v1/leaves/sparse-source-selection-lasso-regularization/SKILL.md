---
name: sparse-source-selection-lasso-regularization
description: Use when estimating microbial source contributions to a sink community
  from count data with many candidate sources (e.g., 50+ nuisance environments from
  public repositories like EMP), and existing methods (FEAST, SourceTracker2, RAD)
  inflate error as source count grows.
when_to_use_negative:
- When the candidate source set is small and all sources are likely contributing —
  plain FEAST already estimates proportions without needing sparse selection
- When the goal is only exploratory source exploration without an unknown-source component
  and the prior Aitchison-distance selection strategy suffices (though the paper shows
  its accuracy is very low with an unknown source)
- When input is not compositional count data with a proper Env/SourceSink/id metadata
  table (e.g., already-normalized abundance tables without count semantics)
edam_operation: http://edamontology.org/operation_3731
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: STENSL
  role: EM-with-sparsity algorithm performing Lasso-based source selection; code is
    a branch of the FEAST codebase
  repo: https://github.com/cozygene/FEAST/tree/STENSL
- name: FEAST
  role: Host codebase installed via devtools; provides Load_CountMatrix/Load_metadata
    input loaders and the FEAST baseline estimator
  repo: https://github.com/cozygene/FEAST
- name: devtools
  role: Used to install the FEAST/STENSL package from GitHub
- name: Rcpp
  role: Dependency required prior to installing FEAST
- name: RcppArmadillo
  role: Dependency required prior to installing FEAST
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1128/msystems.00995-21
    title: 'STENSL: Microbial Source Tracking with ENvironment SeLection'
  source_content_hash: sha256:3873018ada3cd6a4e00f564ce1bf5006ad148ae5ab3eacc57a89b92623e830c1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FEAST
  - R
  - Rcpp
  - RcppArmadillo
  - STENSL
  - SourceTracker2
  - devtools
  edam_operation: http://edamontology.org/operation_3731
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# Sparse source selection with Lasso regularization (STENSL)

## Summary

Applies STENSL, an expectation-maximization algorithm with L1 (Lasso) sparsity via a quadratic program, to identify contributing microbial sources among a large set of candidate environments while modeling an unknown source. It reduces noise from noncontributing sources so mixing-proportion accuracy does not degrade as candidate sources scale into the hundreds.

## When to use

Use when estimating microbial source contributions to a sink community from count data with many candidate sources (e.g., 50+ nuisance environments from public repositories like EMP), and existing methods (FEAST, SourceTracker2, RAD) inflate error as source count grows.

## When NOT to use

- When the candidate source set is small and all sources are likely contributing — plain FEAST already estimates proportions without needing sparse selection
- When the goal is only exploratory source exploration without an unknown-source component and the prior Aitchison-distance selection strategy suffices (though the paper shows its accuracy is very low with an unknown source)
- When input is not compositional count data with a proper Env/SourceSink/id metadata table (e.g., already-normalized abundance tables without count semantics)

## Inputs

- m-by-n tab-delimited count matrix (rows = SampleID, columns = taxa) loadable via Load_CountMatrix
- m-by-3 metadata table with columns Env, SourceSink, id (values 'Source'/'Sink') loadable via Load_metadata
- Sink-source configuration (same groups of sources per sink across all methods being compared)

## Outputs

- Per-sink sparse source contribution estimates (mixing proportions, including an unknown source) with noncontributing sources driven toward zero
- Selected contributing source set identified via the Lasso-sparse α
- Method-comparison statistics (e.g., MSE against known mixing proportions, Wilcoxon signed-rank p-values) when benchmarked

## How to apply

Install FEAST from GitHub (devtools::install_github("cozygene/FEAST", ref = "FEAST"); STENSL is the STENSL branch) with dependencies Rcpp, RcppArmadillo, and vegan, then load the m-by-n count matrix with Load_CountMatrix and the 3-column metadata (Env, SourceSink, id) with Load_metadata. STENSL formulates a least-squares problem between the sink x and YTα with L1 regularization, solving a quadratic program for a sparsely approximated α; the Lasso hyperparameter λ is chosen by cross-validation (in the authors' experiments λ = 1e-6), and for each fixed λ expectation-maximization infers α and γ. A grid-search over λ returns the parameter set with highest likelihood, and the unknown source is initialized by subtracting the scaled counts of the top contributing source (ŶM+1 = max(0, x − α̂ᵀYargmaxα̂)). Run STENSL on the same sink-source configurations used for competing methods and compare estimated proportions against ground truth, e.g., by MSE and Wilcoxon signed-rank tests as in the in vitro gut-mixture benchmark with 50 noncontributing sources.

## Related tools

- **STENSL** (EM-with-sparsity algorithm performing Lasso-based source selection; code is a branch of the FEAST codebase) — https://github.com/cozygene/FEAST/tree/STENSL
- **FEAST** (Host codebase installed via devtools; provides Load_CountMatrix/Load_metadata input loaders and the FEAST baseline estimator) — https://github.com/cozygene/FEAST
- **devtools** (Used to install the FEAST/STENSL package from GitHub)
- **Rcpp** (Dependency required prior to installing FEAST)
- **RcppArmadillo** (Dependency required prior to installing FEAST)

## Examples

```
devtools::install_github("cozygene/FEAST", ref = "FEAST"); FEAST_output <- FEAST(C = otus, metadata = metadata, EM_iterations = 1000, different_sources_flag = 1, outfile = "demo")
```

## Evaluation signals

- MSE of estimated proportions against known true mixing proportions must show STENSL most accurate, with Wilcoxon signed-rank p-values below the reported thresholds (SourceTracker2 P < 2.2e-16, FEAST P < 2.1e-16, RAD P < 2.8e-16 on the in vitro data)
- Estimated contribution rows sum to 1 (rows of the S1-by-S2 matrix P), with noncontributing sources receiving near-zero sparse weights
- Error in mixing-proportion estimation should remain stable (not increase) as the number of candidate sources grows into the hundreds
- Cross-validation-selected λ (grid-searched for highest likelihood; experiments used λ = 1e-6) should be reported/checked rather than arbitrarily fixed
- Unknown source component should be present in the output and initialized as max(0, x − α̂ᵀYargmaxα̂) from the top contributing source

## Limitations

- Lasso hyperparameter λ in the authors' experiments was fixed to 1 × 10−6 without a reported sensitivity analysis
- Detailed simulation protocol and in vitro assembly protocols are only in supplementary Texts S1 and S2; the in vitro data have no accession/deposit
- No software version is specified for STENSL/FEAST, and installation issues with Rcpp/RcppArmadillo on Mac may require homebrew or xcode

## Evidence

- [abstract] we developed STENSL (microbial Source Tracking with ENvironment SeLection), an expectation-maximization algorithm with sparsity: "we developed STENSL (microbial Source Tracking with ENvironment SeLection), an expectation-maximization algorithm with sparsity that enables the identification of contributing sources"
- [discussion] Lasso quadratic program: "formulating a least-squares problem between x and YTα while ensuring that α is sparsely approximated by L1 regularization"
- [discussion] lambda grid search: "a grid-search algorithm to perform EM across a range of values, returning the set of parameters which obtains the highest likelihood"
- [discussion] lambda fixed at 1e-6: "In our experiments, we chose λ = 1 × 10−6"
- [methods] installation command: "devtools::install_github("cozygene/FEAST", ref = "FEAST")"
- [readme] FEAST input/output contract: "FEAST returns an S1 by S2 matrix P, where S1 is the number sinks and S2 is the number of sources (including an unknown source). Each row in matrix P sums to 1."

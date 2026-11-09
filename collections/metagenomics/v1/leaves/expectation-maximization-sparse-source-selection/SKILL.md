---
name: expectation-maximization-sparse-source-selection
description: Use when a microbial source tracking sink must be decomposed against
  many candidate sources (e.g., hundreds of environments from public repositories
  like HMP or EMP), where prior methods add considerable error and only a small subset
  of sources truly contribute.
when_to_use_negative:
- When the full set of contributing sources is already known and small (simple source
  tracking without source exploration), where FEAST or SourceTracker2 suffice without
  sparsity-based selection
- When the input is raw sequence data or an unprocessed feature table requiring upstream
  taxonomy assignment and count-matrix construction
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0610
tools:
- name: STENSL
  role: Core expectation-maximization-with-sparsity algorithm that identifies contributing
    sources among many candidates; used via its branch of the FEAST codebase
  repo: https://github.com/cozygene/FEAST/tree/STENSL
- name: FEAST
  role: Host framework installed via devtools; FEAST() produces the S1-by-S2 mixing
    proportion matrix P and Load_metadata/Load_CountMatrix load inputs
  repo: https://github.com/cozygene/FEAST
- name: SourceTracker2
  role: Baseline microbial source tracking method run on the identical sink-source
    configuration for comparison against STENSL attributions
- name: devtools
  role: Used to install FEAST from GitHub
- name: Rcpp
  role: Dependency attached before installing FEAST/STENSL
- name: RcppArmadillo
  role: Dependency attached before installing FEAST/STENSL
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1128/msystems.00995-21
    title: 'STENSL: Microbial Source Tracking with ENvironment SeLection'
  source_content_hash: sha256:a854f32efd4065339cd2ba5fe8a4bb4f7f0bc5a5e2124e7fd2198e3fedb5bf27
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0610
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# expectation-maximization-sparse-source-selection

## Summary

Apply STENSL, an expectation-maximization algorithm with sparsity (L1-regularized Lasso source selection inside the FEAST codebase), to estimate mixing proportions of contributing sources and an unknown source in a microbial sink even when hundreds of candidate source environments are considered.

## When to use

Use when a microbial source tracking sink must be decomposed against many candidate sources (e.g., hundreds of environments from public repositories like HMP or EMP), where prior methods add considerable error and only a small subset of sources truly contribute.

## When NOT to use

- When the full set of contributing sources is already known and small (simple source tracking without source exploration), where FEAST or SourceTracker2 suffice without sparsity-based selection
- When the input is raw sequence data or an unprocessed feature table requiring upstream taxonomy assignment and count-matrix construction

## Inputs

- m-by-n count matrix (samples x taxa) of read counts, e.g., tab-delimited otu_example_multi.txt
- m-by-3 metadata table with columns 'Env', 'SourceSink' (Source/Sink), and 'id'
- Candidate source environments (e.g., focal subject plus 88 HMP individuals across 15 body sites)

## Outputs

- S1-by-S2 mixing proportion matrix P where each row sums to 1, including an unknown-source column
- Saved output file demo_FEAST.txt containing matrix P
- Per-source attribution estimates, e.g., 43.1% oral (17.2% buccal mucosa, 15.3% tongue dorsum, 10.6% throat) and 26% unknown for the HMP saliva sink

## How to apply

Install the FEAST codebase hosting the STENSL branch with devtools::install_github("cozygene/FEAST", ref = "FEAST") after attaching Rcpp and RcppArmadillo. Load an m-by-n count table and an m-by-3 metadata table ('Env', 'SourceSink', 'id') with Load_metadata and Load_CountMatrix, then run STENSL's EM with sparsity: for a fixed lambda it uses expectation-maximization to infer alpha and gamma, formulates a least-squares problem with L1 regularization to sparsely approximate alpha, determines lambda by cross-validation (lambda = 1e-6 was used in the paper's experiments), and initializes the unknown source as Y^M+1 = max(0, x - alpha_hat * Y_argmax). Run FEAST(C = otus, metadata = metadata, different_sources_flag = 1, dir_path = ..., outfile = "demo") to obtain an S1-by-S2 mixing proportion matrix P (rows summing to 1, including an unknown-source column), and optionally compare against SourceTracker2 on the identical sink-source configuration.

## Related tools

- **STENSL** (Core expectation-maximization-with-sparsity algorithm that identifies contributing sources among many candidates; used via its branch of the FEAST codebase) — https://github.com/cozygene/FEAST/tree/STENSL
- **FEAST** (Host framework installed via devtools; FEAST() produces the S1-by-S2 mixing proportion matrix P and Load_metadata/Load_CountMatrix load inputs) — https://github.com/cozygene/FEAST
- **SourceTracker2** (Baseline microbial source tracking method run on the identical sink-source configuration for comparison against STENSL attributions)
- **devtools** (Used to install FEAST from GitHub)
- **Rcpp** (Dependency attached before installing FEAST/STENSL)
- **RcppArmadillo** (Dependency attached before installing FEAST/STENSL)

## Examples

```
devtools::install_github("cozygene/FEAST", ref = "FEAST"); FEAST_output <- FEAST(C = otus, metadata = metadata, different_sources_flag = 1, dir_path = "~/FEAST/Data_files/", outfile = "demo")
```

## Evaluation signals

- Output matrix demo_FEAST.txt has rows that sum to 1 (S1-by-S2 matrix P including the unknown-source column)
- Reproduced HMP saliva-sink analysis approximates 43.1% oral / 26% unknown for STENSL (17.2% buccal mucosa, 15.3% tongue dorsum, 10.6% throat) and approximately 4.9% oral / 0.1% unknown for SourceTracker2
- Cross-validation selects the Lasso hyperparameter lambda and the grid-search returns the parameter set with the highest likelihood
- Noncontributing sources receive near-zero mixing proportions consistent with the L1-regularized sparsity of alpha

## Limitations

- No software version is specified for STENSL/FEAST; installation issues with Rcpp/RcppArmadillo can occur on macs (homebrew or xcode may be required)
- The Lasso hyperparameter was fixed to a single value (lambda = 1e-6) in the paper's experiments without a reported sensitivity analysis
- Existing state-of-the-art methods add considerable error when hundreds of potential source environments are considered, which is the scenario STENSL targets; the prior Aitchison-distance strategy has very low accuracy in the presence of an unknown source

## Evidence

- [abstract] STENSL (microbial Source Tracking with ENvironment SeLection), an expectation-maximization algorithm with sparsity that enables the identification of contributing sources: "we developed STENSL (microbial Source Tracking with ENvironment SeLection), an expectation-maximization algorithm with sparsity that enables the identification of contributing sources"
- [discussion] EM inference and grid-search over lambda: "For a fixed λ, we use expectation-maximization to infer parameters α and γ."
- [discussion] Lasso sparse source selection rationale: "formulating a least-squares problem between x and YTα while ensuring that α is sparsely approximated by L1 regularization"
- [methods] FEAST call and output specification: "FEAST_output <- FEAST(C = otus, metadata = metadata, different_sources_flag = 1, dir_path = "~/FEAST/Data_files/", outfile="demo")"
- [readme] README description of the output matrix P: "Each row in matrix P sums to 1. Pij is the contribution of source j to sink i."

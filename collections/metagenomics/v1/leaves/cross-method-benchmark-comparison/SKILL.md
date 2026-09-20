---
name: cross-method-benchmark-comparison
description: Use when a sink community's origins must be quantified and validated
  by benchmarking a new source tracking method (STENSL) against a baseline (SourceTracker2)
  on the exact same candidate source set, e.g., 88 HMP individuals across 15 body
  sites.
when_to_use_negative:
- The goal is method development or simulation benchmarking rather than reproducing
  a published real-data analysis on a fixed sink-source configuration
- Candidate source environments are few and known a priori (no 'source exploration'
  problem) — plain FEAST estimation without cross-method comparison is sufficient
- The input already contains per-source mixing proportions from one method only, with
  no second method run on the identical configuration
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: STENSL
  role: 'Primary method under test: EM with sparsity (L1-regularized Lasso source
    selection, cross-validated lambda) identifying contributing sources among the
    88-individual HMP candidate pool'
  repo: https://github.com/cozygene/FEAST/tree/STENSL
- name: SourceTracker2
  role: Baseline method run on the identical sink-source configuration for comparison
    of oral/unknown attribution estimates
- name: FEAST
  role: Codebase hosting the STENSL branch; FEAST() also produces the S1-by-S2 mixing
    proportion matrix P with an unknown-source column
  repo: https://github.com/cozygene/FEAST
- name: devtools
  role: Installs FEAST from GitHub (ref = 'FEAST')
- name: Rcpp / RcppArmadillo
  role: Required dependencies installed and loaded prior to FEAST installation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1128/msystems.00995-21
    title: 'STENSL: Microbial Source Tracking with ENvironment SeLection'
  source_content_hash: sha256:c08e5f63a3dc6fb7929ed25a7163b1329c3577e4fd4a72ac4c5e820a1c00c538
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FEAST
  - R
  - Rcpp
  - Rcpp / RcppArmadillo
  - RcppArmadillo
  - STENSL
  - SourceTracker2
  - devtools
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# cross-method-benchmark-comparison

## Summary

Runs two or more microbial source tracking methods (here STENSL and SourceTracker2) on an identical sink-source configuration and compares estimated mixing proportions against published reference values, to quantify how much of a sink (e.g., HMP saliva) is attributed to focal-source environments.

## When to use

Use when a sink community's origins must be quantified and validated by benchmarking a new source tracking method (STENSL) against a baseline (SourceTracker2) on the exact same candidate source set, e.g., 88 HMP individuals across 15 body sites.

## When NOT to use

- The goal is method development or simulation benchmarking rather than reproducing a published real-data analysis on a fixed sink-source configuration
- Candidate source environments are few and known a priori (no 'source exploration' problem) — plain FEAST estimation without cross-method comparison is sufficient
- The input already contains per-source mixing proportions from one method only, with no second method run on the identical configuration

## Inputs

- m-by-n count matrix (samples x taxa) for the sink and all candidate sources, tab-delimited with 'SampleID' row names
- m-by-3 metadata table with columns 'Env', 'SourceSink' ('Source'/'Sink'), and 'id' (1..number of sinks; sources get ids only for distinct source sets)
- Same sink-source configuration passed identically to both STENSL and SourceTracker2
- Published reference values for comparison (e.g., Fig 4A: 43.1% oral / 26% unknown for STENSL; 4.9% / 0.1% for SourceTracker2)

## Outputs

- S1-by-S2 mixing proportion matrix P (rows sum to 1, includes an unknown-source column), saved as demo_FEAST.txt
- Per-source contribution estimates for the saliva sink (buccal mucosa, tongue dorsum, throat fractions) from each method
- Side-by-side STENSL vs SourceTracker2 oral/unknown proportion comparison

## How to apply

Install FEAST from GitHub with devtools::install_github('cozygene/FEAST', ref = 'FEAST'), attaching Rcpp and RcppArmadillo first since macOS install issues are common. Load the m-by-n count table and m-by-3 metadata ('Env', 'SourceSink', 'id') with Load_metadata and Load_CountMatrix, defining the saliva sink and candidate sources (focal subject's oral sites plus 88 individuals across 15 body sites). Run STENSL (EM with sparsity: L1-regularized Lasso source selection, cross-validated lambda; lambda = 1e-6 as used in the paper's experiments) on the saliva sink, then run SourceTracker2 on the identical sink-source configuration so the comparison is apples-to-apples. Also call FEAST(C = otus, metadata = metadata, different_sources_flag = 1, dir_path = ..., outfile = 'demo') to produce the S1-by-S2 mixing proportion matrix P (rows sum to 1, including an unknown-source column). Compare the estimated oral and unknown proportions against the published Fig 4A values: STENSL ~43.1% oral (17.2% buccal mucosa, 15.3% tongue dorsum, 10.6% throat) with ~26% unknown, versus SourceTracker2 ~4.9% oral with ~0.1% unknown — the divergence itself is the finding that STENSL identifies contributing sources that sparse-prior methods miss.

## Related tools

- **STENSL** (Primary method under test: EM with sparsity (L1-regularized Lasso source selection, cross-validated lambda) identifying contributing sources among the 88-individual HMP candidate pool) — https://github.com/cozygene/FEAST/tree/STENSL
- **SourceTracker2** (Baseline method run on the identical sink-source configuration for comparison of oral/unknown attribution estimates)
- **FEAST** (Codebase hosting the STENSL branch; FEAST() also produces the S1-by-S2 mixing proportion matrix P with an unknown-source column) — https://github.com/cozygene/FEAST
- **devtools** (Installs FEAST from GitHub (ref = 'FEAST'))
- **Rcpp / RcppArmadillo** (Required dependencies installed and loaded prior to FEAST installation)

## Examples

```
Packages <- c("Rcpp","RcppArmadillo"); lapply(Packages, library, character.only=TRUE); devtools::install_github("cozygene/FEAST", ref="FEAST"); FEAST_output <- FEAST(C = otus, metadata = metadata, different_sources_flag = 1, dir_path = "~/FEAST/Data_files/", outfile="demo")
```

## Evaluation signals

- demo_FEAST.txt output matrix must have rows that sum to 1
- Reproduced STENSL estimates approximate 43.1% oral / 26% unknown for the saliva sink
- Reproduced SourceTracker2 estimates approximate 4.9% oral / 0.1% unknown
- Per-site STENSL breakdown approximates 17.2% buccal mucosa, 15.3% tongue dorsum, 10.6% throat
- Both methods are verified to have consumed the identical sink-source configuration (same m-by-3 metadata and count matrix)

## Limitations

- The Lasso hyperparameter was fixed to a single value (lambda = 1e-6) in the paper's experiments without a reported sensitivity analysis
- Detailed simulation protocols and HMP/EMP experiment details are in supplementary Texts S1 and S3, not in the main text
- No software version is specified for STENSL/FEAST, complicating exact reproduction
- FEAST is implemented in R (>= 3.4.4); macOS users may hit Rcpp/RcppArmadillo install issues requiring homebrew or xcode

## Evidence

- [abstract] STENSL is significantly more accurate in identifying the contributing sources as well as the unknown source, even when considering hundreds of potential source environments: "STENSL is significantly more accurate in identifying the contributing sources as well as the unknown source, even when considering hundreds of potential source environments"
- [methods] install FEAST using devtools: "install *FEAST* using devtools"
- [methods] FEAST call with different_sources_flag: "FEAST(C = otus, metadata = metadata, different_sources_flag = 1, dir_path = "~/FEAST/Data_files/", outfile="demo")"
- [intro] methods such as FEAST and SourceTracker2 quantify source fractions in a sink: "methods such as FEAST (1) and SourceTracker2 (2) quantify the fraction, or proportion, of different microbial samples (sources) in a target microbial community (sink)"
- [readme] FEAST output matrix rows sum to 1 including unknown source: "*FEAST* returns an S1 by S2 matrix P, where S1 is the number sinks and S2 is the number of sources (including an unknown source). Each row in matrix P sums to 1."

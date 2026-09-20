---
name: high-dimensional-feature-screening
description: Use when the feature matrix is high-dimensional relative to the sample
  size (e.g. hundreds of standardized OTUs vs tens of samples) and most taxa are expected
  to be noise, so the multivariate association test must operate on a selected subset
  S of taxa.
when_to_use_negative:
- When the number of taxa in the tested clade is small and there is no noise-accumulation
  concern — the article notes accumulation of small noises deteriorates performance
  'especially when the number of taxa being tested is moderate or relatively large'
- When the input is raw unnormalized count data — counts must first be converted to
  compositional form and OTU-wise standardized before screening
- When the goal is per-taxon univariate association testing rather than multivariate
  clade-level association with FWER control via Bonferroni correction
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_3697
tools:
- name: AMAT
  role: R package implementing the distance correlation learning screening (Algorithm
    1) and the subsequent adaptive aSPU test on the selected subset
  repo: https://github.com/kzb193/AMAT
- name: R
  role: Computing environment in which the screening script and AMAT test are executed
- name: QIIME
  role: Upstream pipeline used to construct the OTUs (denoised 16S rRNA gene sequences,
    region V1–V2) that form the screening input matrix
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqab120
    title: Adaptive and powerful microbiome multivariate association analysis via
      feature selection
  source_content_hash: sha256:f6c08b51c0fbda1aaa945451d90652271922c97cf13c748c340228c7a9294d8e
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
  - QIIME
  - R
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# High-dimensional feature screening

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Pre-filters high-dimensional compositional microbiome feature matrices (e.g. hundreds of OTUs) down to an associated-feature subset S using distance correlation learning, so that the subsequent adaptive multivariate association test (AMAT's T_AMAT / aSPU with Γ = {2, 3, 4, 8}) is computed on selected taxa rather than all p features, alleviating noise accumulation.

## When to use

Use when the feature matrix is high-dimensional relative to the sample size (e.g. hundreds of standardized OTUs vs tens of samples) and most taxa are expected to be noise, so the multivariate association test must operate on a selected subset S of taxa.

## When NOT to use

- When the number of taxa in the tested clade is small and there is no noise-accumulation concern — the article notes accumulation of small noises deteriorates performance 'especially when the number of taxa being tested is moderate or relatively large'
- When the input is raw unnormalized count data — counts must first be converted to compositional form and OTU-wise standardized before screening
- When the goal is per-taxon univariate association testing rather than multivariate clade-level association with FWER control via Bonferroni correction

## Inputs

- OTU abundance count matrix (e.g. 856 OTUs x 60 samples, throat microbiome) normalized to compositional form and OTU-wise standardized (Z*)
- Adjusted response vector (continuous or binary outcome, e.g. smoking status)
- Optional: clustering/taxa-set assignments (e.g. PAM clusters by cophenetic distance) if testing at genus/family/order/class/phylum levels with taxa-sets of at least five taxa

## Outputs

- Selected feature set S (indices/names of taxa retained after distance correlation learning screening)
- Screened test statistic T_AMAT computed on Z*_S with power set Γ = {2, 3, 4, 8}
- Permutation-based P-value (e.g. written to pamat_pvalue.csv) plus reusable null permutation statistics

## How to apply

First normalize the OTU count matrix to compositional form and standardize each OTU (yielding Z*), and prepare the adjusted response vector. Then apply the Algorithm 1 screening step (distance correlation learning, as implemented in the AMAT R package at https://github.com/kzb193/AMAT) to Z* to obtain the selected feature set S; the rationale is that score-based weighting leaves small non-zero weights on noise taxa whose accumulation deteriorates the multivariate test when the number of taxa is moderate or large. Within AMAT, the same screening step is re-run on each of the B = 500 permuted responses so the null statistics can be reused for both the PSPU(γ) combination and the final P-value, keeping permutation cost low. Decision point: features not in S receive exactly zero weight — this is the intended behavior distinguishing AMAT from weighting-based competitors like aMiSPU, OMiAT, and MiHC.

## Related tools

- **AMAT** (R package implementing the distance correlation learning screening (Algorithm 1) and the subsequent adaptive aSPU test on the selected subset) — https://github.com/kzb193/AMAT
- **R** (Computing environment in which the screening script and AMAT test are executed)
- **QIIME** (Upstream pipeline used to construct the OTUs (denoised 16S rRNA gene sequences, region V1–V2) that form the screening input matrix)

## Examples

```
Rscript -e "remotes::install_github('kzb193/AMAT'); library(AMAT)"  # then run Algorithm 1 screening on standardized OTU matrix Z* to obtain feature set S
```

## Evaluation signals

- Screening runs on Z* without error and returns a feature subset S of size < p (features excluded get exactly zero weight, not small non-zero weights)
- Final P-value P_AMAT is a single numeric value in [0, 1] computed as (1/B) Σ I[T(b) ≤ T_AMAT] over B = 500 permutations
- Null permutation statistics are shared between the PSPU(γ) combination step and the final P-value (verifiable by checking the same null statistic vector is reused)
- The selected set S is re-derived for each permuted response, i.e. screening is re-run inside the permutation loop, not fixed once
- On simulated throat-microbiome-like data (n = 100, 10–30% signal densities), AMAT power should reach roughly 44.7–52.5%, outperforming runner-up tests at ~33.6–38.3%

## Limitations

- Most microbial taxa within a clade are likely not associated with the outcome, so screening quality directly limits power; a screening step that misses signal taxa will hurt the final test
- Low-rank taxa with opposite effect directions can cancel out during aggregation, a failure mode the feature-selection approach is designed to mitigate but not eliminate
- Permutation cost scales with B (500 permutations used in the AMAT testing stage; 10,000 permutations were used in the article's real-data comparisons), so screening plus testing must be efficient enough to rerun per permutation

## Evidence

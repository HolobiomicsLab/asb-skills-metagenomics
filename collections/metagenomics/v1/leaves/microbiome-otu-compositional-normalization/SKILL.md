---
name: microbiome-otu-compositional-normalization
description: Use when the input is a raw OTU abundance count matrix (e.g. 16S V1-V2
  OTUs from QIIME) that must be compositional-normalized and OTU-wise standardized
  before feature screening or multivariate association testing in the AMAT framework.
when_to_use_negative:
- Input is already a compositional or standardized feature matrix (e.g. previously
  relabeled relative abundances on the same scale) — re-normalization would distort
  the intended scale
- Downstream analysis is not multivariate association testing in the AMAT/aSPU framework
  and requires untransformed counts (e.g. some differential-abundance or phylogenetic
  kernel methods that operate on counts or other transforms)
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0610
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Implementation environment in which the compositional normalization, OTU-wise
    standardization, and downstream AMAT screening/testing are scripted
  repo: https://github.com/kzb193/AMAT
- name: QIIME
  role: Upstream pipeline that produces the denoised 16S rRNA OTU table that this
    normalization step consumes
- name: AMAT
  role: Downstream R package whose Algorithm 1 screening and Algorithm 2 testing require
    the standardized compositional matrix Z*
  repo: https://github.com/kzb193/AMAT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqab120
    title: Adaptive and powerful microbiome multivariate association analysis via
      feature selection
  source_content_hash: sha256:02a65db995a5aafd73044fbccfe909f813443a635184563a206eb8d3daab0f59
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
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0610
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# microbiome-otu-compositional-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prepares a raw OTU count table for AMAT-style multivariate association testing by converting counts to compositional (relative-abundance) form and then standardizing each OTU column to produce the transformed matrix Z* used downstream for feature screening and the aSPU/AMAT test.

## When to use

Use when the input is a raw OTU abundance count matrix (e.g. 16S V1-V2 OTUs from QIIME) that must be compositional-normalized and OTU-wise standardized before feature screening or multivariate association testing in the AMAT framework.

## When NOT to use

- Input is already a compositional or standardized feature matrix (e.g. previously relabeled relative abundances on the same scale) — re-normalization would distort the intended scale
- Downstream analysis is not multivariate association testing in the AMAT/aSPU framework and requires untransformed counts (e.g. some differential-abundance or phylogenetic kernel methods that operate on counts or other transforms)

## Inputs

- Raw OTU abundance count matrix (OTUs × samples), e.g. from a QIIME OTU table (16S rRNA gene region V1-V2)
- Adjusted response vector (e.g. smoking status or other outcome) aligned with samples
- Optional filtering thresholds: OTU counts > 3 in > 3 samples

## Outputs

- Compositional-normalized, OTU-wise standardized matrix Z* used as input to AMAT screening (Algorithm 1) and testing (Algorithm 2)

## How to apply

Load the OTU abundance matrix (OTUs × samples, e.g. 856 OTUs × 60 samples from the throat microbiome data) together with the adjusted response vector. First apply quality control and filtering consistent with the study design (e.g. keep OTUs with counts of more than three in more than three samples, as in the gut data yielding 557 OTUs), then normalize counts to compositional form so each sample sums to one. Next standardize each OTU (column) to zero mean and unit variance, producing the matrix Z* that feeds the Algorithm 1 distance-correlation screening and the aSPU/AMAT test computed on the selected subset Z*_S. Standardization is required because AMAT computes sum-of-powered-score statistics on individual taxa, so features must be on a comparable scale before sparse weights are assigned.

## Related tools

- **R** (Implementation environment in which the compositional normalization, OTU-wise standardization, and downstream AMAT screening/testing are scripted) — https://github.com/kzb193/AMAT
- **QIIME** (Upstream pipeline that produces the denoised 16S rRNA OTU table that this normalization step consumes)
- **AMAT** (Downstream R package whose Algorithm 1 screening and Algorithm 2 testing require the standardized compositional matrix Z*) — https://github.com/kzb193/AMAT

## Examples

```
Z <- sweep(otu, 1, rowSums(otu), "/"); Zstar <- scale(Z)  # compositional normalization + OTU-wise standardization before AMAT
```

## Evaluation signals

- After compositional normalization, each sample's relative abundances sum to 1 (within floating-point tolerance)
- After OTU-wise standardization, each OTU column of Z* has mean approximately 0 and standard deviation approximately 1
- Row/column dimensions of Z* match the filtered OTU table (e.g. 856 OTUs × 60 samples for the throat data; 557 OTUs for the gut data) and sample order matches the adjusted response vector
- No negative or missing values remain in Z*; zero-count OTUs removed by the filter step (counts > 3 in > 3 samples) do not appear

## Limitations

- Compositional transformation discards absolute sequencing-depth information, which is intrinsic to compositional microbiome data and assumed acceptable by the AMAT framework
- Standardizing low-prevalence OTUs amplifies noise; the counts > 3 in > 3 samples filter should precede normalization, and the method targets sparse association signals where 'most microbial taxa within a clade are likely not associated with the outcome'

## Evidence

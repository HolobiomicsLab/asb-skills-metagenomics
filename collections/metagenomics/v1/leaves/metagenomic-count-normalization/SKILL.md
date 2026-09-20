---
name: metagenomic-count-normalization
description: Use when the input is a sparse, high-throughput marker-gene (OTU/species/taxon)
  count matrix with two or more sample groups, and per-feature differential abundance
  statistics must account for both normalization and under-sampling of microbial communities.
when_to_use_negative:
- The data are not sparse high-throughput sequencing counts (e.g. continuous abundance
  measurements or already-tested per-feature statistics) — metagenomeSeq is designed
  for statistical analysis of sparse high-throughput sequencing
- The goal is not to determine features (OTUs or other taxa) differentially abundant
  between two or more groups of multiple samples
- Input is only a diversity or composition summary with no feature-level count matrix,
  so no per-feature differential abundance testing is possible
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: metagenomeSeq
  role: Bioconductor package implementing the novel normalization technique and the
    under-sampling-aware statistical model; its zig EM routine fits the zero-inflated
    mixture model and yields per-feature differential-abundance statistics
  repo: github.com/HCBravoLab/metagenomeSeq
- name: BiocManager
  role: Used to install the latest release version of metagenomeSeq from Bioconductor
- name: devtools
  role: Used to install the latest development version of metagenomeSeq from the Bioconductor-mirror
    GitHub repository via install_github
  repo: Bioconductor-mirror/metagenomeSeq
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/nmeth.2658
    title: Differential abundance analysis for microbial marker-gene surveys
  source_content_hash: sha256:14a64e59a104b88a38dcd524313e8433f246d7ca1f3b17c2c660b313cd2d656f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BiocManager
  - devtools
  - metagenomeSeq
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# metagenomic-count-normalization

## Summary

Applies metagenomeSeq's novel normalization technique together with a statistical model that accounts for under-sampling, so that differential abundance of OTUs/taxa can be assessed in sparse high-throughput marker gene count data.

## When to use

Use when the input is a sparse, high-throughput marker-gene (OTU/species/taxon) count matrix with two or more sample groups, and per-feature differential abundance statistics must account for both normalization and under-sampling of microbial communities.

## When NOT to use

- The data are not sparse high-throughput sequencing counts (e.g. continuous abundance measurements or already-tested per-feature statistics) — metagenomeSeq is designed for statistical analysis of sparse high-throughput sequencing
- The goal is not to determine features (OTUs or other taxa) differentially abundant between two or more groups of multiple samples
- Input is only a diversity or composition summary with no feature-level count matrix, so no per-feature differential abundance testing is possible

## Inputs

- Raw or normalized count matrix of marker-gene survey sequencing reads (rows = OTU/species/taxon features, columns = samples)
- Feature labels (OTU/species/taxon identifiers)
- Sample annotation table assigning each sample to one of two or more groups

## Outputs

- Fitted zero-inflated mixture model (zig) per feature
- Per-feature differential-abundance statistics table (effect sizes and significance), e.g. differential_abundance_statistics.csv

## How to apply

Install metagenomeSeq from Bioconductor via BiocManager (or the development version from the Bioconductor-mirror GitHub repo via devtools install_github). Load the count matrix, feature labels (OTU/species/taxon identifiers), and the two-or-more-group sample annotations into a metagenomeSeq count-data object with a design matrix encoding the group comparison. Fit the zero-inflated Gaussian model using the zig EM fitting routine, which jointly addresses the effects of normalization and under-sampling of microbial communities on disease association detection. Extract per-feature fitted effect sizes and significance values for features differentially abundant between the groups and export the statistics table. Correct application means the under-sampling component, not just depth normalization, is part of the fit.

## Related tools

- **metagenomeSeq** (Bioconductor package implementing the novel normalization technique and the under-sampling-aware statistical model; its zig EM routine fits the zero-inflated mixture model and yields per-feature differential-abundance statistics) — github.com/HCBravoLab/metagenomeSeq
- **BiocManager** (Used to install the latest release version of metagenomeSeq from Bioconductor)
- **devtools** (Used to install the latest development version of metagenomeSeq from the Bioconductor-mirror GitHub repository via install_github) — Bioconductor-mirror/metagenomeSeq

## Examples

```
if (!requireNamespace("BiocManager", quietly=TRUE)); install.packages("BiocManager"); BiocManager::install("metagenomeSeq")
```

## Evaluation signals

- The zig EM fit completes on the metagenomeSeq count-data object without failure for the features in the count matrix
- The exported statistics table (differential_abundance_statistics.csv) contains per-feature effect sizes and significance values matching the number of features in the input matrix
- Feature identifiers in the output table correspond 1:1 with the OTU/species/taxon labels supplied as input
- Design matrix correctly encodes the two-or-more-group comparison specified in the sample annotations
- In validation, results should be consistent with reported behavior that metagenomeSeq outperforms previously used tools on simulated data and published microbiota datasets

## Limitations

- The article/README provides no changelog for the package, so version-specific behavior changes are not traceable from this source
- The development version must be installed from a GitHub mirror (Bioconductor-mirror/metagenomeSeq) via devtools, which may differ from the Bioconductor release
- The model is targeted at sparse high-throughput sequencing from marker gene surveys; its under-sampling assumption may not suit dense or non-sequencing abundance data

## Evidence

- [abstract] Our approach, implemented in the metagenomeSeq Bioconductor package, relies on a novel normalization technique and a statistical model that accounts for under-sampling: "Our approach, implemented in the metagenomeSeq Bioconductor package, relies on a novel normalization technique and a statistical model that accounts for under-sampling"
- [readme] metagenomeSeq is designed to address the effects of both normalization and undersampling of microbial communities on disease association detection and the testing of feature correlations.: "metagenomeSeq is designed to address the effects of both normalization and undersampling of microbial communities on disease association detection and the testing of feature correlations."
- [methods] if (!requireNamespace("BiocManager", quietly=TRUE)) install.packages("BiocManager") BiocManager::install("metagenomeSeq"): "if (!requireNamespace("BiocManager", quietly=TRUE))
    install.packages("BiocManager")
BiocManager::install("metagenomeSeq")"
- [intro] metagenomeSeq is designed to determine features (be it Operational Taxonomic Units (OTU) or other taxa) that are differentially abundant across populations: [evidence span withheld — anchor status: not_found]

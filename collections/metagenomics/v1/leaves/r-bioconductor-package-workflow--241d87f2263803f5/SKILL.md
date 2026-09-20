---
name: r-bioconductor-package-workflow
description: Use when analyzing sparse OTU/marker-gene count matrices from two or
  more sample groups to detect differentially abundant features while accounting for
  normalization and under-sampling of microbial communities.
when_to_use_negative:
- When the goal is standard count-based differential expression on well-covered RNA-seq
  data without under-sampling concerns — the paper benchmarks and outperforms tools
  like DESeq, edgeR, Metastats, and LEfSe specifically on sparse, under-sampled marker-gene
  surveys
- When the input is not a marker-gene/OTU count matrix (e.g. continuous abundance
  or compositional data without counts), since metagenomeSeq's zero-inflated model
  is built for sparse high-throughput sequencing counts
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: metagenomeSeq
  role: Core Bioconductor package implementing the novel normalization and the zero-inflated
    statistical model (zig EM fitting) that accounts for under-sampling to call differentially
    abundant features
  repo: github.com/HCBravoLab/metagenomeSeq
- name: BiocManager
  role: Installer used to install the latest release version of metagenomeSeq from
    Bioconductor
- name: devtools
  role: Used to install the latest development version of metagenomeSeq from GitHub
provenance:
  source_task_ids:
  - task_004
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/nmeth.2658
    title: Differential abundance analysis for microbial marker-gene surveys
  source_content_hash: sha256:e8d1d329faed176bff90ce9cd2118aa487acaf1bbcd55549b153e5dc38db42ce
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BiocManager
  - devtools
  - metagenomeSeq
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# r-bioconductor-package-workflow

## Summary

An R/Bioconductor package workflow pattern for installing and running metagenomeSeq, which combines a novel normalization technique with a statistical model accounting for under-sampling to detect differentially abundant features (OTUs or other taxa) in sparse high-throughput marker-gene survey data.

## When to use

Use when analyzing sparse OTU/marker-gene count matrices from two or more sample groups to detect differentially abundant features while accounting for normalization and under-sampling of microbial communities.

## When NOT to use

- When the goal is standard count-based differential expression on well-covered RNA-seq data without under-sampling concerns — the paper benchmarks and outperforms tools like DESeq, edgeR, Metastats, and LEfSe specifically on sparse, under-sampled marker-gene surveys
- When the input is not a marker-gene/OTU count matrix (e.g. continuous abundance or compositional data without counts), since metagenomeSeq's zero-inflated model is built for sparse high-throughput sequencing counts

## Inputs

- Sparse OTU/marker-gene count matrix (features x samples)
- Feature labels (OTU/species/taxon identifiers)
- Sample group annotations (two or more groups)
- Design matrix encoding the group comparison

## Outputs

- Per-feature differential-abundance statistics table (effect sizes and significance, e.g. differential_abundance_statistics.csv)
- Ranked list of differentially abundant features between groups

## How to apply

Install the latest release of metagenomeSeq from Bioconductor via BiocManager::install('metagenomeSeq'), or the development version with devtools::install_github('Bioconductor-mirror/metagenomeSeq'). Load the sparse OTU count matrix together with feature labels (OTU/species/taxon identifiers) and the two-or-more-group sample annotations. Set up the metagenomeSeq count-data object with a design matrix encoding the group comparison. Fit the zero-inflated Gaussian mixture model via the zig EM fitting routine so that the effects of both normalization and under-sampling of the microbial communities are accounted for in the fit. Extract per-feature fitted differential-abundance statistics (effect sizes and significance) and rank or export the results table (e.g. differential_abundance_statistics.csv). The rationale, per the paper, is that metagenomeSeq is designed to address the effects of both normalization and undersampling on disease association detection.

## Related tools

- **metagenomeSeq** (Core Bioconductor package implementing the novel normalization and the zero-inflated statistical model (zig EM fitting) that accounts for under-sampling to call differentially abundant features) — github.com/HCBravoLab/metagenomeSeq
- **BiocManager** (Installer used to install the latest release version of metagenomeSeq from Bioconductor)
- **devtools** (Used to install the latest development version of metagenomeSeq from GitHub)

## Examples

```
if (!requireNamespace("BiocManager", quietly=TRUE)) install.packages("BiocManager"); BiocManager::install("metagenomeSeq")
```

## Evaluation signals

- The metagenomeSeq fit runs via the zig EM routine without errors and produces fitted statistics for each feature in the count matrix
- The output table contains per-feature effect sizes and significance values for every OTU/taxon tested between the annotated groups
- On simulated data with known ground truth, metagenomeSeq's detection metrics exceed those of baseline tools (DESeq, edgeR, Metastats, LEfSe), reproducing the paper's outperformance claim
- Results table is written to the expected CSV (e.g. differential_abundance_statistics.csv) with one row per feature and correct feature identifiers

## Limitations

- The model targets sparse, under-sampled high-throughput marker-gene survey data; performance claims were demonstrated on simulated data and several published microbiota datasets
- The paper reports no changelog in the README, so development-version installs from the mirror may vary in behavior
- Requires installation of a Bioconductor package (release via BiocManager or development via devtools), adding dependency overhead

## Evidence

- [readme] metagenomeSeq is designed to determine features (be it Operational Taxanomic Unit (OTU), species, etc.) that are differentially abundant between two or more groups of multiple samples.: [evidence span withheld — anchor status: external_doc]

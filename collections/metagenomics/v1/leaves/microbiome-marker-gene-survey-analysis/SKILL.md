---
name: microbiome-marker-gene-survey-analysis
description: Use when the input is a sparse high-throughput microbiome OTU/taxa count
  matrix with two or more sample groups and the goal is to identify features that
  are differentially abundant across populations.
when_to_use_negative:
- The data are not sparse marker-gene count data — metagenomeSeq is designed for sparse
  high-throughput sequencing from marker-gene surveys, not for continuous or non-count
  measurements.
- The analysis goal is not differential abundance between two or more groups of samples
  (e.g., unsupervised ordination or clustering without group labels).
- A dense, well-sampled count matrix with no under-sampling concern is already available
  — other DA tools may suffice since metagenomeSeq's advantage comes from addressing
  normalization and undersampling.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3174
tools:
- name: metagenomeSeq
  role: Core Bioconductor package implementing the normalization technique and statistical
    model that account for under-sampling of microbial communities when testing OTUs/taxa
    for differential abundance across populations
  repo: github.com/HCBravoLab/metagenomeSeq
- name: BiocManager
  role: Used to install the latest release version of the metagenomeSeq Bioconductor
    package
- name: devtools
  role: Used to install the latest development version of metagenomeSeq from GitHub
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/nmeth.2658
    title: Differential abundance analysis for microbial marker-gene surveys
  source_content_hash: sha256:a1c5d2790a404b73731a47d33ec7b0b021f88dcb8657257ef01c6ada0ca406b5
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
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# microbiome-marker-gene-survey-analysis

## Summary

Differential abundance analysis of OTU/taxa count matrices from microbial marker-gene surveys using metagenomeSeq, which combines a novel normalization technique with a statistical model (zero-inflated Gaussian fit by expectation-maximization) that accounts for under-sampling of microbial communities.

## When to use

Use when the input is a sparse high-throughput microbiome OTU/taxa count matrix with two or more sample groups and the goal is to identify features that are differentially abundant across populations.

## When NOT to use

- The data are not sparse marker-gene count data — metagenomeSeq is designed for sparse high-throughput sequencing from marker-gene surveys, not for continuous or non-count measurements.
- The analysis goal is not differential abundance between two or more groups of samples (e.g., unsupervised ordination or clustering without group labels).
- A dense, well-sampled count matrix with no under-sampling concern is already available — other DA tools may suffice since metagenomeSeq's advantage comes from addressing normalization and undersampling.

## Inputs

- OTU count matrix from a microbial marker-gene survey (sparse high-throughput sequencing counts)
- Sample group labels (two or more groups of multiple samples) for differential abundance testing

## Outputs

- Per-feature differential-abundance statistics table (effect sizes and significance values) with ranked features, e.g., differential_abundance_results.csv

## How to apply

Load the published microbiota OTU count matrix and corresponding sample group labels into R, then install metagenomeSeq from Bioconductor via BiocManager. Apply metagenomeSeq's normalization technique, which is specifically designed to address under-sampling of microbial communities in sparse high-throughput sequencing data. Fit metagenomeSeq's statistical model to test each OTU or taxon for differential abundance between the two or more groups of samples. Extract per-feature statistics (differential-abundance effect sizes and significance values), rank the features, and export the ranked table (e.g., differential_abundance_results.csv). Validate by confirming the method's relative performance: the paper shows, using simulated data and several published microbiota datasets, that metagenomeSeq outperforms the tools currently used in this field.

## Related tools

- **metagenomeSeq** (Core Bioconductor package implementing the normalization technique and statistical model that account for under-sampling of microbial communities when testing OTUs/taxa for differential abundance across populations) — github.com/HCBravoLab/metagenomeSeq
- **BiocManager** (Used to install the latest release version of the metagenomeSeq Bioconductor package)
- **devtools** (Used to install the latest development version of metagenomeSeq from GitHub)

## Examples

```
if (!requireNamespace("BiocManager", quietly=TRUE)) install.packages("BiocManager"); BiocManager::install("metagenomeSeq")
```

## Evaluation signals

- The output is a ranked differential-abundance table containing per-feature effect sizes and significance values for each OTU/taxon.
- Normalization has been applied with metagenomeSeq's under-sampling-aware technique rather than a naive library-size scaling.
- The fitted model tests each feature for differential abundance across the defined sample groups (two or more groups).
- On simulated data and published microbiota datasets, results should be consistent with the paper's finding that metagenomeSeq outperforms existing differential abundance tools in this field.
- The saved CSV (e.g., differential_abundance_results.csv) contains one row per feature with coherent statistics (no missing per-feature effect size/significance columns).

## Limitations

- The skill depends on metagenomeSeq's assumptions about under-sampling of microbial communities; the package is designed specifically to address normalization and undersampling effects on disease association detection and feature-correlation testing.
- The development version is installed from a GitHub mirror (Bioconductor-mirror/metagenomeSeq) and may be less stable than the Bioconductor release version.
- No changelog is available, per the article's extracted entities.

## Evidence

- [readme] README purpose statement: [evidence span withheld — anchor status: external_doc]

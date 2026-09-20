---
name: multiple-sample-group-statistical-testing
description: Use when you have a sparse high-throughput OTU/taxon count matrix with
  sample group labels and need to identify features differentially abundant between
  two or more groups.
when_to_use_negative:
- Input is not sparse marker-gene count data (e.g. already normalized composition
  data or non-sequencing feature tables)
- The goal is feature correlation testing rather than differential abundance between
  two or more groups
edam_operation: http://edamontology.org/operation_0314
tools:
- name: metagenomeSeq
  role: Normalization of undersampled sparse counts and statistical model to test
    each OTU/taxon for differential abundance between two or more sample groups
  repo: https://github.com/HCBravoLab/metagenomeSeq
- name: BiocManager
  role: Installer used to obtain the latest release version of metagenomeSeq from
    Bioconductor
- name: devtools
  role: Installer used to obtain the latest development version of metagenomeSeq from
    GitHub
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/nmeth.2658
    title: Differential abundance analysis for microbial marker-gene surveys
  source_content_hash: sha256:988d500025fa41a498e80c8222e0a83e6bd7c5992dcd389f5da3a2cde2a237ec
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HCBravoLab/metagenomeSeq
  tools:
  - BiocManager
  - devtools
  - metagenomeSeq
  edam_operation: http://edamontology.org/operation_0314
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# multiple-sample-group-statistical-testing

## Summary

Differential abundance testing of sparse microbial marker-gene (OTU/taxon) count features across two or more sample groups using metagenomeSeq, whose normalization and statistical model explicitly address undersampling of microbial communities.

## When to use

Use when you have a sparse high-throughput OTU/taxon count matrix with sample group labels and need to identify features differentially abundant between two or more groups.

## When NOT to use

- Input is not sparse marker-gene count data (e.g. already normalized composition data or non-sequencing feature tables)
- The goal is feature correlation testing rather than differential abundance between two or more groups

## Inputs

- OTU/taxon count matrix (sparse high-throughput sequencing counts) from a microbial marker-gene survey
- Sample group labels assigning each sample to one of two or more populations

## Outputs

- Ranked per-feature differential-abundance table (effect sizes and significance values per OTU/taxon, e.g. differential_abundance_results.csv)

## How to apply

Load the published microbiota OTU count matrix and the corresponding sample group labels into R, then install the metagenomeSeq release from Bioconductor via BiocManager (or the development version via devtools from the Bioconductor-mirror GitHub mirror). Normalize the sparse count data with metagenomeSeq's novel normalization technique, which is specifically designed to address undersampling of microbial communities rather than generic compositional scaling. Run metagenomeSeq's statistical model to test each OTU or taxon for differential abundance across the two or more sample groups, then extract the per-feature effect sizes and significance values and rank features accordingly. Save the ranked differential-abundance table (e.g. differential_abundance_results.csv) as the deliverable for downstream comparison against existing DA tools.

## Related tools

- **metagenomeSeq** (Normalization of undersampled sparse counts and statistical model to test each OTU/taxon for differential abundance between two or more sample groups) — https://github.com/HCBravoLab/metagenomeSeq
- **BiocManager** (Installer used to obtain the latest release version of metagenomeSeq from Bioconductor)
- **devtools** (Installer used to obtain the latest development version of metagenomeSeq from GitHub)

## Examples

```
BiocManager::install("metagenomeSeq")
```

## Evaluation signals

- Output CSV contains one row per OTU/taxon with both effect size and significance value columns
- Features are ranked by the extracted differential-abundance statistics, not in original matrix order
- Normalization step applied metagenomeSeq's own technique (designed for undersampling), not generic library-size scaling
- Testing was performed across two or more groups simultaneously, not just pairwise tests
- On simulated data and published microbiota datasets, metagenomeSeq should outperform existing differential abundance analysis tools per the paper's validation

## Limitations

- Advantage is demonstrated on simulated data and published microbiota datasets; performance on other data types is not established here
- Designed for sparse high-throughput sequencing data; both normalization and under-sampling effects must be relevant to the dataset for the model's assumptions to hold
- No changelog is available for the repository, limiting version tracking

## Evidence

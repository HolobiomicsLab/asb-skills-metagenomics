---
name: sparse-metagenomic-count-data-processing
description: Use when input is a sparse high-throughput marker-gene survey count matrix
  (samples as columns, OTU features as rows) and the goal is normalization prior to
  differential abundance analysis that must account for under-sampling.
when_to_use_negative:
- The count data are not from a sparse marker-gene survey, so under-sampling is not
  the dominant bias the normalization is designed to counteract
- Differential abundance testing is already the goal and metagenomeSeq's cumNorm +
  statistical model can be applied directly instead of reimplementing the normalization
  step
- The input is already a normalized feature table rather than raw OTU counts
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3307
tools:
- name: metagenomeSeq
  role: Provides the novel cumulative-sum-style normalization (cumNorm) and the under-sampling-aware
    statistical model that the standalone normalization function must replicate
  repo: github.com/HCBravoLab/metagenomeSeq
- name: BiocManager
  role: Installs the latest release version of metagenomeSeq from Bioconductor
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/nmeth.2658
    title: Differential abundance analysis for microbial marker-gene surveys
  source_content_hash: sha256:5b28d0bb94468b273cb41e999c1eb814fcdc4ea18601743bf734cf5bb70e0c57
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BiocManager
  - metagenomeSeq
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# sparse-metagenomic-count-data-processing

## Summary

Normalizes sparse marker-gene OTU count matrices using the cumulative-sum-style normalization underlying metagenomeSeq, so that undersampling of microbial communities is countered before differential abundance modeling.

## When to use

Use when input is a sparse high-throughput marker-gene survey count matrix (samples as columns, OTU features as rows) and the goal is normalization prior to differential abundance analysis that must account for under-sampling.

## When NOT to use

- The count data are not from a sparse marker-gene survey, so under-sampling is not the dominant bias the normalization is designed to counteract
- Differential abundance testing is already the goal and metagenomeSeq's cumNorm + statistical model can be applied directly instead of reimplementing the normalization step
- The input is already a normalized feature table rather than raw OTU counts

## Inputs

- Marker-gene OTU count matrix as CSV (samples as columns, features/OTUs as rows)

## Outputs

- Normalized count matrix written as normalized_counts.csv
- Standalone R normalization function css_normalize.R

## How to apply

Install metagenomeSeq from Bioconductor via BiocManager::install("metagenomeSeq"). Load the marker-gene OTU count matrix from CSV with samples as columns and features as rows. Implement a standalone R function (e.g. css_normalize.R) that, for each sample, computes a cumulative-sum-style scaling factor from the count distribution and divides all counts by that factor — this is the novel normalization technique metagenomeSeq relies on to counteract undersampling of microbial communities. Apply the function to the count matrix and write the result to normalized_counts.csv. Validate by comparing against metagenomeSeq's own cumNorm routine applied to the same input matrix.

## Related tools

- **metagenomeSeq** (Provides the novel cumulative-sum-style normalization (cumNorm) and the under-sampling-aware statistical model that the standalone normalization function must replicate) — github.com/HCBravoLab/metagenomeSeq
- **BiocManager** (Installs the latest release version of metagenomeSeq from Bioconductor)

## Examples

```
if (!requireNamespace("BiocManager", quietly=TRUE)) install.packages("BiocManager"); BiocManager::install("metagenomeSeq")
```

## Evaluation signals

- css_normalize.R runs without error on the loaded count matrix (script_runs)
- normalized_counts.csv exists and is in CSV format (file_exists, file_format_is)
- Normalized values agree with the output of metagenomeSeq's own cumNorm routine on the same input matrix (output_matches_reference)
- Each sample's counts are divided by a per-sample cumulative-sum-style scaling factor, preserving feature count proportions within samples

## Limitations

- The normalization is specific to sparse marker-gene survey data where under-sampling of microbial communities is a common feature; it is not a generic read-depth normalization
- Cumulative-sum scaling must operate on the marker-gene count data prior to any under-sampling-aware statistical model, so order of operations matters

## Evidence

---
name: cumulative-sum-scaling-implementation
description: Use when a sparse high-throughput marker-gene survey count matrix must
  be normalized for under-sampling before differential abundance analysis with metagenomeSeq,
  and the scaling factors must be computed and applied outside (but consistent with)
  the package's own routine.
when_to_use_negative:
- The input is already normalized or is not a raw marker-gene/OTU count matrix (e.g.
  relative abundances or already CSS-scaled values)
- The goal is differential abundance inference itself rather than the normalization
  step preceding metagenomeSeq's under-sampling-aware statistical model
- The data are not sparse high-throughput sequencing counts, so under-sampling correction
  is not the relevant concern
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: metagenomeSeq
  role: Source of the cumulative-sum scaling normalization strategy; its cumNorm routine
    serves as the reference implementation against which the standalone function's
    output is validated
  repo: github.com/HCBravoLab/metagenomeSeq
- name: BiocManager
  role: Used to install the latest release version of metagenomeSeq from Bioconductor
    before applying the normalization
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/nmeth.2658
    title: Differential abundance analysis for microbial marker-gene surveys
  source_content_hash: sha256:91a699042438a9c7c0a221afe1ba0daec764277aa63b49a6f9ad1ea055c134e1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BiocManager
  - metagenomeSeq
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# cumulative-sum-scaling-implementation

## Summary

Implements the cumulative-sum-style scaling normalization that metagenomeSeq relies on to counteract under-sampling of sparse marker-gene count data, as a standalone function that can be applied to an OTU count matrix prior to under-sampling-aware differential abundance modeling.

## When to use

Use when a sparse high-throughput marker-gene survey count matrix must be normalized for under-sampling before differential abundance analysis with metagenomeSeq, and the scaling factors must be computed and applied outside (but consistent with) the package's own routine.

## When NOT to use

- The input is already normalized or is not a raw marker-gene/OTU count matrix (e.g. relative abundances or already CSS-scaled values)
- The goal is differential abundance inference itself rather than the normalization step preceding metagenomeSeq's under-sampling-aware statistical model
- The data are not sparse high-throughput sequencing counts, so under-sampling correction is not the relevant concern

## Inputs

- Marker-gene OTU count matrix loaded from a CSV file (samples as columns, features as rows)
- R environment with metagenomeSeq installed via BiocManager

## Outputs

- normalized_counts.csv containing the normalized count matrix in CSV format
- css_normalize.R, a standalone R function computing per-sample cumulative-sum scaling factors

## How to apply

Load the marker-gene OTU count matrix from CSV with samples as columns and features as rows. For each sample, compute a cumulative-sum-style scaling factor from the sample's count distribution and divide all of that sample's counts by the factor, mirroring the normalization strategy metagenomeSeq applies before its under-sampling-aware statistical model. Write the standalone function as css_normalize.R in R, apply it to the count matrix, and save the result as normalized_counts.csv. Rationale: the article states metagenomeSeq 'relies on a novel normalization technique and a statistical model that accounts for under-sampling', so the normalization must operate on the marker-gene count data prior to that model. Verify correctness by comparing the output against metagenomeSeq's own cumNorm routine applied to the same input matrix.

## Related tools

- **metagenomeSeq** (Source of the cumulative-sum scaling normalization strategy; its cumNorm routine serves as the reference implementation against which the standalone function's output is validated) — github.com/HCBravoLab/metagenomeSeq
- **BiocManager** (Used to install the latest release version of metagenomeSeq from Bioconductor before applying the normalization)

## Examples

```
BiocManager::install("metagenomeSeq"); counts <- read.csv("counts.csv"); write.csv(css_normalize(counts), "normalized_counts.csv")
```

## Evaluation signals

- css_normalize.R runs without error (script_runs)
- normalized_counts.csv exists and is in CSV format (file_exists, file_format_is)
- Normalized values agree with the normalization applied by metagenomeSeq's own cumNorm routine on the same input matrix (output_matches_reference)
- Every sample in the output matrix has been divided by its own scaling factor (per-sample, not global, scaling)

## Limitations

- The technique is specific to sparse high-throughput marker-gene survey data where under-sampling of microbial communities is common; its rationale does not transfer to non-sparse or non-count data
- Correctness is only meaningful relative to metagenomeSeq's cumNorm behavior on the same input matrix; divergence from that reference indicates an implementation error

## Evidence

- [abstract] Our approach, implemented in the metagenomeSeq Bioconductor package, relies on a novel normalization technique and a statistical model that accounts for under-sampling: "Our approach, implemented in the metagenomeSeq Bioconductor package, relies on a novel normalization technique and a statistical model that accounts for under-sampling"
- [readme] metagenomeSeq is designed to address the effects of both normalization and undersampling of microbial communities on disease association detection: "metagenomeSeq is designed to address the effects of both normalization and undersampling of microbial communities on disease association detection"
- [other] Install the latest release of metagenomeSeq from Bioconductor using BiocManager::install("metagenomeSeq") in R.: [evidence span withheld — anchor status: not_found]
- [intro] Statistical analysis for sparse high-throughput sequencing: "Statistical analysis for sparse high-throughput sequencing"

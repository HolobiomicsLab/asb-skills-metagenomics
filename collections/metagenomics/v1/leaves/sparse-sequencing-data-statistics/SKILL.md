---
name: sparse-sequencing-data-statistics
description: Use when comparing feature (OTU/species/taxon) abundance across two or
  more sample groups in sparse, high-throughput marker-gene survey count data where
  under-sampling and normalization effects must be modeled to detect differential
  abundance.
when_to_use_negative:
- The analysis does not involve two or more sample groups — metagenomeSeq is designed
  to determine features (be it OTU, species, etc.) differentially abundant between
  two or more groups of multiple samples, so single-group summaries are out of scope.
- Sample annotations do not define groups, so a design matrix encoding the group comparison
  cannot be set up for the zig EM fit.
- The data are not sparse high-throughput sequencing counts, since the model accounts
  for under-sampling, a common feature of large-scale marker gene studies.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
tools:
- name: metagenomeSeq
  role: Core Bioconductor package providing the novel normalization technique and
    the zero-inflated (zig) statistical model, fit by an EM routine, that accounts
    for under-sampling of microbial communities when computing per-feature differential-abundance
    statistics.
  repo: github.com/HCBravoLab/metagenomeSeq
- name: BiocManager
  role: Installs the latest release version of the metagenomeSeq Bioconductor package
    (BiocManager::install("metagenomeSeq")).
- name: devtools
  role: Installs the latest development version of metagenomeSeq from the Bioconductor-mirror
    GitHub repository via install_github("Bioconductor-mirror/metagenomeSeq").
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/nmeth.2658
    title: Differential abundance analysis for microbial marker-gene surveys
  source_content_hash: sha256:6990ba3d22d4f938d5064ab0189dcac2e4f5f6d17c4da8d0635d9fcc8f003ca4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BiocManager
  - devtools
  - metagenomeSeq
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# sparse-sequencing-data-statistics

## Summary

Statistical analysis for sparse high-throughput marker-gene sequencing data: metagenomeSeq combines a novel normalization technique with a zero-inflated statistical model that accounts for under-sampling of microbial communities to identify features (OTU/species/taxa) differentially abundant between two or more groups.

## When to use

Use when comparing feature (OTU/species/taxon) abundance across two or more sample groups in sparse, high-throughput marker-gene survey count data where under-sampling and normalization effects must be modeled to detect differential abundance.

## When NOT to use

- The analysis does not involve two or more sample groups — metagenomeSeq is designed to determine features (be it OTU, species, etc.) differentially abundant between two or more groups of multiple samples, so single-group summaries are out of scope.
- Sample annotations do not define groups, so a design matrix encoding the group comparison cannot be set up for the zig EM fit.
- The data are not sparse high-throughput sequencing counts, since the model accounts for under-sampling, a common feature of large-scale marker gene studies.

## Inputs

- Normalized count matrix of marker-gene survey features (OTU/species/taxa) by samples
- Feature labels (OTU/species/taxon identifiers)
- Sample annotations assigning each sample to two or more groups for the comparison

## Outputs

- Per-feature differential-abundance statistics (effect sizes and significance) for features differentially abundant between groups
- differential_abundance_statistics.csv

## How to apply

Install metagenomeSeq: the latest Bioconductor release via BiocManager (BiocManager::install("metagenomeSeq")) or the development version via devtools install_github from the Bioconductor-mirror repository. Load the normalized count matrix together with feature labels (OTU/species/taxon identifiers) and the two-or-more-group sample annotations. Set up the metagenomeSeq count-data object and a design matrix encoding the group comparison for the samples. Fit metagenomeSeq's zero-inflated mixture model with the zig EM fitting routine so that the effects of both normalization and under-sampling of the microbial communities are accounted for in the fit. Extract per-feature fitted differential-abundance statistics (effect sizes and significance) for features differentially abundant between the groups, and write the per-feature statistics table to differential_abundance_statistics.csv.

## Related tools

- **metagenomeSeq** (Core Bioconductor package providing the novel normalization technique and the zero-inflated (zig) statistical model, fit by an EM routine, that accounts for under-sampling of microbial communities when computing per-feature differential-abundance statistics.) — github.com/HCBravoLab/metagenomeSeq
- **BiocManager** (Installs the latest release version of the metagenomeSeq Bioconductor package (BiocManager::install("metagenomeSeq")).)
- **devtools** (Installs the latest development version of metagenomeSeq from the Bioconductor-mirror GitHub repository via install_github("Bioconductor-mirror/metagenomeSeq").)

## Examples

```
if (!requireNamespace("BiocManager", quietly=TRUE)) install.packages("BiocManager"); BiocManager::install("metagenomeSeq")
```

## Evaluation signals

- The zig EM fitting routine completes and yields fitted per-feature differential-abundance statistics (effect sizes and significance) for the OTU/species/taxon features in the count matrix.
- differential_abundance_statistics.csv is produced with one row per feature and both effect-size and significance columns reflecting the two-or-more-group design matrix.
- Fitted statistics come from a zero-inflated mixture model in which the effects of both normalization and under-sampling of the microbial communities are accounted for, not from raw-count comparisons.
- Results are sanity-checked against the paper's benchmark claim that metagenomeSeq outperforms the tools currently used in this field, as shown using simulated data and several published microbiota datasets.

## Limitations

- No changelog is found in the repository README, so version-specific changes to metagenomeSeq behavior are not documented there.
- The under-sampling statistical model targets a common feature of large-scale marker gene studies, so its rationale is specific to sparse high-throughput sequencing data and does not transfer to data without under-sampled structure.
- Development installs via devtools from the Bioconductor-mirror repository may differ from the latest Bioconductor release installed via BiocManager.

## Evidence

- [readme] Purpose: differential abundance between two or more groups: [evidence span withheld — anchor status: external_doc]

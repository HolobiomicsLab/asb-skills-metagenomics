---
name: sparse-count-data-normalization
description: Use when a sparse high-throughput OTU/taxa count matrix from a microbiota
  survey must be normalized for undersampling of microbial communities before testing
  differential abundance between two or more sample groups.
when_to_use_negative:
- Input is already normalized or is not a sparse high-throughput sequencing count
  matrix — metagenomeSeq's normalization is built for sparse count data affected by
  undersampling of microbial communities.
- The analysis goal is not finding features (OTUs, species, etc.) differentially abundant
  between two or more groups of multiple samples — e.g., purely unsupervised ordination
  or sample-level classification.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0196
tools:
- name: metagenomeSeq
  role: Core Bioconductor package implementing the novel normalization technique for
    under-sampled sparse counts and the statistical model that tests each OTU for
    differential abundance between groups.
  repo: https://github.com/HCBravoLab/metagenomeSeq
- name: BiocManager
  role: Installs the latest release of metagenomeSeq from Bioconductor inside R before
    normalization and testing are run.
- name: devtools
  role: Installs the latest development version of metagenomeSeq via install_github('Bioconductor-mirror/metagenomeSeq')
    when the dev build is required.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/nmeth.2658
    title: Differential abundance analysis for microbial marker-gene surveys
  source_content_hash: sha256:32bd042464df2dd7a00085cb4c52e2f3fb71407dd81ed6d7a3dbcb55df03b562
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HCBravoLab/metagenomeSeq
  tools:
  - BiocManager
  - devtools
  - metagenomeSeq
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# sparse-count-data-normalization

## Summary

Normalizes sparse high-throughput sequencing count data (OTU/taxa count matrices from microbiota marker-gene surveys) with metagenomeSeq's novel normalization technique, which is designed to address undersampling of microbial communities. The normalized data then feed metagenomeSeq's statistical model to identify differentially abundant features between two or more groups of samples.

## When to use

Use when a sparse high-throughput OTU/taxa count matrix from a microbiota survey must be normalized for undersampling of microbial communities before testing differential abundance between two or more sample groups.

## When NOT to use

- Input is already normalized or is not a sparse high-throughput sequencing count matrix — metagenomeSeq's normalization is built for sparse count data affected by undersampling of microbial communities.
- The analysis goal is not finding features (OTUs, species, etc.) differentially abundant between two or more groups of multiple samples — e.g., purely unsupervised ordination or sample-level classification.

## Inputs

- Sparse high-throughput OTU/taxa count matrix (features x samples) from a published microbiota dataset
- Sample group labels assigning each sample to one of two or more populations

## Outputs

- Normalized version of the sparse count matrix (metagenomeSeq normalization applied)
- Ranked per-feature differential-abundance table with effect sizes and significance values (e.g., differential_abundance_results.csv)

## How to apply

Load the published microbiota OTU count matrix and the corresponding sample group labels into R, then install the metagenomeSeq release from Bioconductor with BiocManager::install('metagenomeSeq') (or the development version via devtools install_github from Bioconductor-mirror/metagenomeSeq). Run metagenomeSeq's normalization technique on the sparse high-throughput count matrix; the method is explicitly designed to jointly address normalization and the under-sampling of microbial communities that biases disease-association detection. With the normalized counts, fit metagenomeSeq's statistical model to test each OTU (or other taxa) for differential abundance across the two or more groups of multiple samples. Extract the per-feature differential-abundance effect sizes and significance values, rank the features, and save the ranked table as differential_abundance_results.csv. Judge success by reproducing the paper's reported advantage: on simulated data and several published microbiota datasets, the metagenomeSeq pipeline outperforms the tools currently used for differential abundance analysis in this field.

## Related tools

- **metagenomeSeq** (Core Bioconductor package implementing the novel normalization technique for under-sampled sparse counts and the statistical model that tests each OTU for differential abundance between groups.) — https://github.com/HCBravoLab/metagenomeSeq
- **BiocManager** (Installs the latest release of metagenomeSeq from Bioconductor inside R before normalization and testing are run.)
- **devtools** (Installs the latest development version of metagenomeSeq via install_github('Bioconductor-mirror/metagenomeSeq') when the dev build is required.)

## Examples

```
if (!requireNamespace('BiocManager', quietly=TRUE)) install.packages('BiocManager'); BiocManager::install('metagenomeSeq')
```

## Evaluation signals

- R session loads metagenomeSeq installed from the Bioconductor release via BiocManager (or, if reproducing dev behavior, the Bioconductor-mirror GitHub install).
- Normalization runs on the sparse OTU count matrix without altering the feature x sample structure of the input.
- Per-feature output includes differential-abundance effect sizes and significance values for every OTU tested between the groups.
- A ranked table is produced at differential_abundance_results.csv with one ranked row per feature.
- On the paper's benchmarks (simulated data and several published microbiota datasets), the pipeline reproduces metagenomeSeq's advantage over existing differential-abundance tools.

## Limitations

- The normalization and model are designed for sparse high-throughput sequencing data and the effects of undersampling of microbial communities; suitability for other data modalities is not established by this paper.
- The outperformance claim is demonstrated using simulated data and several published microbiota datasets; behavior on other datasets may vary.
- Two install paths exist (BiocManager release vs devtools install from Bioconductor-mirror/metagenomeSeq); the development version may differ from the published release.

## Evidence

- [readme] metagenomeSeq is designed to determine features (be it Operational Taxanomic Unit (OTU), species, etc.): [evidence span withheld — anchor status: external_doc]

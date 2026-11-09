---
name: sparse-marker-gene-count-simulation
description: Use when you need simulated sparse high-throughput marker-gene survey
  count data with known differentially abundant features to validate or benchmark
  differential abundance methods on undersampled microbial communities.
when_to_use_negative:
- Input is already real (non-simulated) marker-gene survey data without known differential-abundance
  ground truth — simulation-based scoring against ground truth cannot be applied
- Data are not sparse high-throughput sequencing count matrices (e.g. non-count or
  low-throughput data), since the skill targets undersampling in sparse microbial
  community counts
- The goal is a single-method differential abundance call rather than comparative
  benchmarking against tools like DESeq, edgeR, Metastats, or LEfSe
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3053
- http://edamontology.org/topic_0610
tools:
- name: metagenomeSeq
  role: Primary method applied to the simulated sparse count matrices to call differentially
    abundant features via its novel normalization technique and under-sampling statistical
    model
  repo: github.com/HCBravoLab/metagenomeSeq
- name: BiocManager
  role: Used to install the latest release version of the metagenomeSeq package in
    R for the benchmarking run
- name: devtools
  role: Used to install the latest development version of metagenomeSeq from GitHub
    when the release version is insufficient
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/nmeth.2658
    title: Differential abundance analysis for microbial marker-gene surveys
  source_content_hash: sha256:b06402354994d4742638e11e52e2ea5c8ee9266095274488ae4c18d7484d5bde
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BiocManager
  - devtools
  - metagenomeSeq
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3053
  - http://edamontology.org/topic_0610
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# sparse-marker-gene-count-simulation

## Summary

Generate simulated sparse OTU/marker-gene count matrices with a known ground-truth set of differentially abundant features so that metagenomeSeq's normalization and under-sampling statistical model can be benchmarked against tools such as DESeq, edgeR, Metastats, and LEfSe.

## When to use

Use when you need simulated sparse high-throughput marker-gene survey count data with known differentially abundant features to validate or benchmark differential abundance methods on undersampled microbial communities.

## When NOT to use

- Input is already real (non-simulated) marker-gene survey data without known differential-abundance ground truth — simulation-based scoring against ground truth cannot be applied
- Data are not sparse high-throughput sequencing count matrices (e.g. non-count or low-throughput data), since the skill targets undersampling in sparse microbial community counts
- The goal is a single-method differential abundance call rather than comparative benchmarking against tools like DESeq, edgeR, Metastats, or LEfSe

## Inputs

- Simulated sparse OTU/marker-gene count matrix (features x samples) spanning two or more sample groups
- Ground-truth label set of differentially abundant features embedded in the simulation
- Simulated group/sample metadata assigning samples to populations

## Outputs

- Simulated count matrices for each method under test
- Per-method detection performance metrics computed against the simulated ground truth
- Comparative benchmark table (e.g. benchmark_results.csv)

## How to apply

Simulate sparse OTU or other taxa count matrices for features across two or more sample groups, embedding a known ground-truth set of differentially abundant features and mimicking the undersampling characteristic of real microbial marker-gene surveys (sparse, high-throughput sequencing data). Run the method under evaluation — e.g. metagenomeSeq, installed via BiocManager for the release or devtools::install_github('Bioconductor-mirror/metagenomeSeq') for development — plus baseline tools such as DESeq, edgeR, Metastats, and LEfSe on the identical simulated count matrices. Score each method's differential-abundance calls against the simulated ground truth and record detection performance metrics per tool. Judge success by writing the comparative table (e.g. to benchmark_results.csv) and confirming the evaluated method's metrics exceed those of the baselines, reproducing the paper's claim that metagenomeSeq outperforms tools currently used in this field.

## Related tools

- **metagenomeSeq** (Primary method applied to the simulated sparse count matrices to call differentially abundant features via its novel normalization technique and under-sampling statistical model) — github.com/HCBravoLab/metagenomeSeq
- **BiocManager** (Used to install the latest release version of the metagenomeSeq package in R for the benchmarking run)
- **devtools** (Used to install the latest development version of metagenomeSeq from GitHub when the release version is insufficient)

## Examples

```
if (!requireNamespace("BiocManager", quietly=TRUE)) install.packages("BiocManager"); BiocManager::install("metagenomeSeq")
```

## Evaluation signals

- Comparative results table (benchmark_results.csv) is produced with detection performance metrics for metagenomeSeq and every baseline tool run on the identical simulated count matrices
- metagenomeSeq's detection metrics against the simulated ground truth exceed those of DESeq, edgeR, Metastats, and LEfSe, reproducing the paper's outperformance finding
- Simulated matrices exhibit the sparsity/undersampling the method targets, and the embedded ground-truth differentially abundant feature set is respected when scoring calls
- metagenomeSeq installs cleanly via BiocManager (release) or devtools::install_github (development) before the benchmark run

## Limitations

- The outperformance claim was demonstrated using simulated data plus several published microbiota datasets; the simulation must faithfully mimic sparse, undersampled microbial communities or results may not generalize
- The package's installation path differs by version channel: latest release via BiocManager vs. development via the Bioconductor-mirror GitHub install, which can produce different behavior
- The article's enriched index notes 'No changelog found' for the package, so version-to-version behavioral changes are not traceable from this context

## Evidence

- [intro] metagenomeSeq targets differential abundance across populations: [evidence span withheld — anchor status: not_found]

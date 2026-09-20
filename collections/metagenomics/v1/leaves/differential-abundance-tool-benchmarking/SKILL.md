---
name: differential-abundance-tool-benchmarking
description: Use when you must reproduce or verify that metagenomeSeq improves differential
  abundance calling on sparse, undersampled marker-gene survey data relative to default
  DESeq, edgeR, Metastats, and LEfSe workflows.
when_to_use_negative:
- The input is already normalized, non-sparse, or otherwise unaffected by undersampling
  of microbial communities — the paper's model targets sparse, undersampled marker-gene
  survey data
- The task is disease-association prediction rather than differential feature calling
  across populations, which metagenomeSeq is specifically designed for
- No ground truth (simulated or otherwise) is available to score methods, making benchmark
  comparison of detection performance meaningless
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3308
tools:
- name: metagenomeSeq
  role: 'Method under test: applies its novel normalization technique and statistical
    model accounting for under-sampling to call differentially abundant OTUs/taxa
    across populations'
  repo: https://github.com/HCBravoLab/metagenomeSeq
- name: BiocManager
  role: Installs the latest release of the metagenomeSeq Bioconductor package for
    the benchmark
- name: devtools
  role: Installs the latest development version of metagenomeSeq from GitHub if the
    release is insufficient
- name: DESeq
  role: Baseline tool run on the identical simulated count matrices with its default
    differential-abundance workflow
- name: edgeR
  role: Baseline tool run on the identical simulated count matrices with its default
    differential-abundance workflow
- name: Metastats
  role: Baseline tool run on the identical simulated count matrices with its default
    differential-abundance workflow
- name: LEfSe
  role: Baseline tool run on the identical simulated count matrices with its default
    differential-abundance workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/nmeth.2658
    title: Differential abundance analysis for microbial marker-gene surveys
  source_content_hash: sha256:ddff54a6efb434d938184a40c1234dd23df40be8d24d7058171fb3e3dab78f69
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HCBravoLab/metagenomeSeq
  tools:
  - BiocManager
  - DESeq
  - LEfSe
  - Metastats
  - devtools
  - edgeR
  - metagenomeSeq
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# differential-abundance-tool-benchmarking

## Summary

Benchmark metagenomeSeq's normalization plus under-sampling (zero-inflated) statistical model against standard differential-abundance tools (DESeq, edgeR, Metastats, LEfSe) on sparse OTU/marker-gene count data with simulated ground truth.

## When to use

Use when you must reproduce or verify that metagenomeSeq improves differential abundance calling on sparse, undersampled marker-gene survey data relative to default DESeq, edgeR, Metastats, and LEfSe workflows.

## When NOT to use

- The input is already normalized, non-sparse, or otherwise unaffected by undersampling of microbial communities — the paper's model targets sparse, undersampled marker-gene survey data
- The task is disease-association prediction rather than differential feature calling across populations, which metagenomeSeq is specifically designed for
- No ground truth (simulated or otherwise) is available to score methods, making benchmark comparison of detection performance meaningless

## Inputs

- Simulated sparse OTU/marker-gene count matrices for features across two or more sample groups with known ground-truth differentially abundant features
- Optionally, published microbiota datasets for confirmation of the simulation-based result

## Outputs

- Per-tool detection performance metrics computed against the simulated ground truth
- Comparative benchmark results table (benchmark_results.csv)

## How to apply

Install metagenomeSeq from Bioconductor via BiocManager (or the development version from GitHub via devtools::install_github). Simulate sparse OTU/marker-gene count matrices across two or more sample groups with a known ground-truth set of differentially abundant features, mimicking undersampled microbial communities. Apply metagenomeSeq's novel normalization and zero-inflated under-sampling model to call differentially abundant features, then run DESeq, edgeR, Metastats, and LEfSe on the identical simulated count matrices with their default workflows. Score every method against the simulated ground truth (e.g. detection performance metrics) and write a comparative table (benchmark_results.csv). The skill is validated when metagenomeSeq's metrics exceed all baseline tools, matching the paper's claim that it outperforms currently used tools.

## Related tools

- **metagenomeSeq** (Method under test: applies its novel normalization technique and statistical model accounting for under-sampling to call differentially abundant OTUs/taxa across populations) — https://github.com/HCBravoLab/metagenomeSeq
- **BiocManager** (Installs the latest release of the metagenomeSeq Bioconductor package for the benchmark)
- **devtools** (Installs the latest development version of metagenomeSeq from GitHub if the release is insufficient)
- **DESeq** (Baseline tool run on the identical simulated count matrices with its default differential-abundance workflow)
- **edgeR** (Baseline tool run on the identical simulated count matrices with its default differential-abundance workflow)
- **Metastats** (Baseline tool run on the identical simulated count matrices with its default differential-abundance workflow)
- **LEfSe** (Baseline tool run on the identical simulated count matrices with its default differential-abundance workflow)

## Examples

```
BiocManager::install("metagenomeSeq")
```

## Evaluation signals

- metagenomeSeq's detection performance metrics (against the simulated ground-truth differentially abundant features) exceed those of DESeq, edgeR, Metastats, and LEfSe run on the identical count matrices
- Each tool is evaluated on identical simulated input count matrices, so differences in metrics are attributable to the statistical method, not data preprocessing
- benchmark_results.csv contains a comparative table with a performance-metric row/column for every tool, populated without missing entries
- The simulated data are demonstrably sparse/undersampled, matching the data regime the zero-inflated model is designed for

## Limitations

- The outperformance claim is demonstrated on simulated data and several published microbiota datasets; results on other data types are not established by this paper
- The paper's supporting index notes 'No changelog found', so development-version behavior (installed via install_github) may vary
- Baseline tools are run with default workflows, so suboptimal tuning of baselines may partly drive performance gaps

## Evidence

- [abstract] metagenomeSeq outperforms the tools currently used in this field, as shown using simulated data and several published microbiota datasets: "We show, using simulated data and several published microbiota datasets, that metagenomeSeq outperforms the tools currently used in this field."
- [methods] metagenomeSeq is designed to address the effects of both normalization and undersampling of microbial communities on disease association detection: "metagenomeSeq is designed to address the effects of both normalization and undersampling of microbial communities on disease association detection"
- [intro] metagenomeSeq determines differentially abundant features across populations: [evidence span withheld — anchor status: not_found]
- [readme] BiocManager installation of the release version: "To install the latest release version of metagenomeSeq:"

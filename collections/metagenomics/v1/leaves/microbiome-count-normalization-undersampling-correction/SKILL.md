---
name: microbiome-count-normalization-undersampling-correction
description: Use when calling differentially abundant features (OTUs, species, or
  other taxa) across two or more sample groups from sparse, undersampled microbial
  count matrices where library-size normalization biases dominate.
when_to_use_negative:
- Data are dense, well-sampled count matrices where under-sampling effects are negligible
  and standard RNA-seq tools (DESeq, edgeR) suffice
- The analysis goal is abundance profiling or taxonomic assignment rather than differential
  abundance testing across populations
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: metagenomeSeq
  role: Core Bioconductor package providing the novel normalization technique and
    zero-inflated under-sampling statistical model for differential abundance detection
  repo: https://github.com/HCBravoLab/metagenomeSeq
- name: BiocManager
  role: Installer for the latest release version of metagenomeSeq from Bioconductor
- name: devtools
  role: Installer for the latest development version of metagenomeSeq from GitHub
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/nmeth.2658
    title: Differential abundance analysis for microbial marker-gene surveys
  source_content_hash: sha256:c3fa7ebcb5719fd07ec307d2caefb454531e19acc862b169fa60b35b42c793c4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HCBravoLab/metagenomeSeq
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

# microbiome-count-normalization-undersampling-correction

## Summary

Applies metagenomeSeq's novel normalization technique and zero-inflated under-sampling statistical model to identify differentially abundant OTUs/taxa in sparse high-throughput marker-gene survey data, where standard RNA-seq tools (DESeq, edgeR) and Metastats/LEfSe underperform.

## When to use

Use when calling differentially abundant features (OTUs, species, or other taxa) across two or more sample groups from sparse, undersampled microbial count matrices where library-size normalization biases dominate.

## When NOT to use

- Data are dense, well-sampled count matrices where under-sampling effects are negligible and standard RNA-seq tools (DESeq, edgeR) suffice
- The analysis goal is abundance profiling or taxonomic assignment rather than differential abundance testing across populations

## Inputs

- Sparse OTU / marker-gene count matrix (features x samples) from high-throughput sequencing surveys
- Sample group labels defining two or more populations (e.g., case/control)
- Optional: simulated count matrices with known ground-truth differentially abundant features for benchmarking

## Outputs

- List of differentially abundant features (OTUs/species/taxa) across populations
- Feature correlation test results
- Benchmark comparison table (e.g., benchmark_results.csv) of detection performance metrics per method (metagenomeSeq vs DESeq, edgeR, Metastats, LEfSe)

## How to apply

Install metagenomeSeq from Bioconductor via BiocManager (or the development version via devtools install_github). Input sparse OTU/marker-gene count matrices with samples grouped into two or more populations, then run metagenomeSeq's normalization and its statistical model that accounts for under-sampling of microbial communities to call differentially abundant features and test feature correlations. For benchmarking, run DESeq, edgeR, Metastats, and LEfSe on the identical count matrices as baselines and compute detection performance metrics against a known ground-truth set of differentially abundant features (e.g., from simulated data). Rationale: under-sampling effects in marker-gene surveys cause spurious disease associations in naive normalization approaches, which metagenomeSeq's model explicitly addresses.

## Related tools

- **metagenomeSeq** (Core Bioconductor package providing the novel normalization technique and zero-inflated under-sampling statistical model for differential abundance detection) — https://github.com/HCBravoLab/metagenomeSeq
- **BiocManager** (Installer for the latest release version of metagenomeSeq from Bioconductor)
- **devtools** (Installer for the latest development version of metagenomeSeq from GitHub)

## Examples

```
BiocManager::install("metagenomeSeq")
```

## Evaluation signals

- On simulated sparse count matrices with known ground truth, metagenomeSeq's detection performance metrics must exceed those of DESeq, edgeR, Metastats, and LEfSe run on the identical matrices
- Benchmark results written to benchmark_results.csv include comparative metrics for every baseline method
- Called differentially abundant features are OTUs/species that differ between two or more defined sample groups
- The installed package is a released Bioconductor version (BiocManager) or tracks the HCBravoLab/Bioconductor-mirror GitHub source when using the development version

## Limitations

- Designed for sparse, undersampled marker-gene survey data; performance advantage was demonstrated on simulated data and published microbiota datasets, so dense or deeply sequenced data may not benefit
- Development-version installation via GitHub (Bioconductor-mirror/metagenomeSeq) may lag or diverge from the Bioconductor release channel
- No changelog available in the repository documentation to track model or behavior changes between versions

## Evidence

- [abstract] metagenomeSeq outperforms the tools currently used in this field, as shown using simulated data and several published microbiota datasets: "We show, using simulated data and several published microbiota datasets, that metagenomeSeq outperforms the tools currently used in this field."

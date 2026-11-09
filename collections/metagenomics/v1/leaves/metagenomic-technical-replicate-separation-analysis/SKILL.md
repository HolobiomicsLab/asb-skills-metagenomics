---
name: metagenomic-technical-replicate-separation-analysis
description: Decision support for choosing Mash sketch and k-mer sizes when k-mer
  distances must distinguish technical resequencings of one sample from other metagenomes
  in a large-scale pipeline.
when_to_use_negative:
- The datasets contain no technical replicates, so replicate-separation cannot be
  used as a validation signal
- You need strain-level or assembly-based comparison rather than approximate MinHash
  distances; Mash sketches trade resolution for speed
- Input is long-read sequencing data — ARGprofiler and this benchmark are designed
  for short-read data only
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: Mash
  role: Creates MinHash sketches and computes pairwise distances for each sketch-size/k-mer-size
    combination
  repo: https://github.com/marbl/Mash
- name: CCPhylo
  role: Dynamic Neighbor-Joining clustering of Mash distance matrices to verify sub-clustering
    of technical replicates
- name: fastq-dl
  role: Downloads the sewage metagenome FASTQ data from ENA
  repo: https://github.com/rpetit3/fastq-dl
- name: ARGprofiler
  role: Pipeline whose Mash sketch defaults are being tuned by this benchmark
  repo: https://github.com/genomicepidemiology/ARGprofiler
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae086
    title: ARGprofiler—a pipeline for large-scale analysis of antimicrobial resistance
      genes and their flanking regions in metagenomic datasets
  source_content_hash: sha256:4896eca5b05b85f7daa18102b670fd01e8df9e1af8af0fbffb00562d4236e8f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marbl/Mash
  tools:
  - ARGprofiler
  - CCPhylo
  - Mash
  - fastq-dl
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# metagenomic-technical-replicate-separation-analysis

## Summary

Benchmark Mash MinHash sketch-size and k-mer-size parameter combinations against technical replicate resequencings to select k-mer-based distance parameters that reliably cluster same-sample replicates apart from other metagenomes.

## When to use

Decision support for choosing Mash sketch and k-mer sizes when k-mer distances must distinguish technical resequencings of one sample from other metagenomes in a large-scale pipeline.

## When NOT to use

- The datasets contain no technical replicates, so replicate-separation cannot be used as a validation signal
- You need strain-level or assembly-based comparison rather than approximate MinHash distances; Mash sketches trade resolution for speed
- Input is long-read sequencing data — ARGprofiler and this benchmark are designed for short-read data only

## Inputs

- Short-read FASTQ files for ~72 sewage metagenomes downloaded from ENA (fastq-dl), including technical replicate resequencings of a single sample
- Known replicate grouping (which runs are resequencings of the same sample)

## Outputs

- Mash distance matrices for each sketch-size/k-mer-size combination
- mash_sketch_benchmark_results.tsv tabulating replicate-separation outcome per parameter combination
- Chosen default sketch size (10^4) and k-mer size (31) for the pipeline

## How to apply

Retrieve the short-read FASTQ data from ENA with fastq-dl (here: 36 Copenhagen sewage samples from PRJEB34633, 18 urban sewage samples from Munk et al. 2022, and 18 technical replicate resequencings of one Copenhagen sample from PRJEB63576). Build Mash 2.3 sketches across a parameter grid — sketch sizes 10^3, 10^4, 10^5, 10^6 crossed with k-mer sizes 16, 21, 27, and 31 — and compute all pairwise Mash distances per combination. For each combination, test whether within-sample distances among replicate resequencings are lower than between-sample distances among distinct sewage samples, and verify replicate sub-clustering with Dynamic Neighbor-Joining in CCPhylo 0.8.3. Select the smallest sketch size that achieves clear replicate separation; k-mer size 31 with sketch size >=10^4 efficiently separated replicates, so sketch size 10^4 was set as the ARGprofiler default.

## Related tools

- **Mash** (Creates MinHash sketches and computes pairwise distances for each sketch-size/k-mer-size combination) — https://github.com/marbl/Mash
- **CCPhylo** (Dynamic Neighbor-Joining clustering of Mash distance matrices to verify sub-clustering of technical replicates)
- **fastq-dl** (Downloads the sewage metagenome FASTQ data from ENA) — https://github.com/rpetit3/fastq-dl
- **ARGprofiler** (Pipeline whose Mash sketch defaults are being tuned by this benchmark) — https://github.com/genomicepidemiology/ARGprofiler

## Examples

```
fastq-dl -a PRJEB34633 -t 16; mash sketch -k 31 -s 10000 -o cph_replicates *.fastq.gz; mash dist cph_replicates.msh cph_replicates.msh > pairwise_distances.tsv
```

## Evaluation signals

- In mash_sketch_benchmark_results.tsv, k=31 with sketch size >=10^4 rows show technical replicates efficiently separated from the remaining sewage samples
- Rows with small sketch sizes (10^3) or short k-mer sizes (16, 21) fail to clearly distinguish replicates from other samples
- Dynamic Neighbor-Joining trees (CCPhylo 0.8.3) show the replicate runs forming a distinct sub-cluster only under passing parameter combinations
- Within-sample replicate pairwise Mash distances are consistently lower than between-sample distances for the selected parameters
- The selected sketch size (10^4) is adopted as the ARGprofiler default, matching the paper's reported choice

## Limitations

- Very small sketch sizes and short k-mers produce noisy distances that do not distinguish same-sample replicates from distinct sewage samples
- ARGprofiler is designed to only work with short-read sequencing data, so this parameter tuning does not transfer to long-read datasets
- Validation relies on the availability of technical replicate resequencings, which may not exist for other sample types

## Evidence

- [abstract] Mash sketches for fast searching and comparisons: "the pipeline also creates Mash sketches for fast searching and comparisons of sequencing runs"
- [discussion] k-mer-based comparison: "k-mer-based comparison of sequence runs"
- [discussion] ARGprofiler repo: "publicly available under the Apache-2.0 license at https://github.com/genomicepidemiology/ARGprofiler"
- [readme] Mash README: [evidence span withheld — anchor status: external_doc]

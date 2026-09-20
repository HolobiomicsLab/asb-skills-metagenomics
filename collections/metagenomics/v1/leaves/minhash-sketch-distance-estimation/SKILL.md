---
name: minhash-sketch-distance-estimation
description: Use when you need sample-level similarity/distance estimates from raw
  short-read FASTQ files — e.g. clustering metagenomes, identifying re-sequencings,
  or selecting Mash sketch/k-mer defaults.
when_to_use_negative:
- Input data are long reads — ARGprofiler (and this Mash-based comparison) is designed
  to only work with short-read sequencing data
- You need genome-level reference comparison or alignment-based distances rather than
  fast sample-level similarity of sequencing runs
- Sketch memory/space constraints force sketch sizes below ~10^4 — results will not
  reliably distinguish near-identical samples
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3697
tools:
- name: Mash
  role: Creates MinHash sketches of sequencing runs and computes all pairwise Mash
    distances across sketch-size/k-mer-size combinations
  repo: https://github.com/marbl/Mash
- name: fastq-dl
  role: Downloads the short-read FASTQ data for the sewage metagenomes from ENA using
    project/sample accessions
  repo: https://github.com/rpetit3/fastq-dl
- name: CCPhylo
  role: Dynamic Neighbor-Joining clustering of Mash distance matrices to verify replicate
    sub-clustering
- name: ARGprofiler
  role: Pipeline whose Mash sketch step applies the selected default (k=31, sketch
    size 10^4) for fast searching and comparison of sequencing runs
  repo: https://github.com/genomicepidemiology/ARGprofiler
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae086
    title: ARGprofiler—a pipeline for large-scale analysis of antimicrobial resistance
      genes and their flanking regions in metagenomic datasets
  source_content_hash: sha256:6063326484dea1c9dfe1d50c06db87ba983c8b70533689823afa8f5b702532e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marbl/Mash
  tools:
  - ARGprofiler
  - CCPhylo
  - Mash
  - fastq-dl
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# minhash-sketch-distance-estimation

## Summary

Benchmark and apply MinHash sketching (Mash 2.3) to estimate sequence distances between metagenomic sequencing runs, tuning sketch size and k-mer size so that distances reliably separate technical replicates from distinct samples. In ARGprofiler this established k=31 and sketch size 10^4 as defaults for k-mer-based comparison of sequencing runs.

## When to use

Use when you need sample-level similarity/distance estimates from raw short-read FASTQ files — e.g. clustering metagenomes, identifying re-sequencings, or selecting Mash sketch/k-mer defaults.

## When NOT to use

- Input data are long reads — ARGprofiler (and this Mash-based comparison) is designed to only work with short-read sequencing data
- You need genome-level reference comparison or alignment-based distances rather than fast sample-level similarity of sequencing runs
- Sketch memory/space constraints force sketch sizes below ~10^4 — results will not reliably distinguish near-identical samples

## Inputs

- Short-read FASTQ files for metagenomic samples (e.g. 72 sewage metagenomes downloaded from ENA with fastq-dl)
- Sample metadata identifying technical replicates vs. distinct samples
- Mash 2.3 parameter grid (sketch sizes 10^3–10^6; k-mer sizes 16, 21, 27, 31)

## Outputs

- Mash MinHash sketch files per sample and parameter combination
- Pairwise Mash distance matrices for each sketch-size/k-mer-size combination
- Summary table of replicate-separation outcomes per parameter combination (mash_sketch_benchmark_results.tsv)
- Dynamic Neighbor-Joining clusterings (CCPhylo) verifying replicate sub-clustering

## How to apply

Retrieve the short-read data (e.g. via fastq-dl from ENA accessions such as PRJEB34633, PRJEB63576), then create Mash 2.3 MinHash sketches for all samples over a parameter grid of sketch sizes 10^3, 10^4, 10^5, 10^6 and k-mer sizes 16, 21, 27, 31. Compute all pairwise Mash distances per parameter combination and evaluate whether within-sample distances among technical replicate resequencings are lower than between-sample distances among distinct sewage samples. Verify replicate sub-clustering with Dynamic Neighbor-Joining (CCPhylo 0.8.3). The article's benchmark showed very small sketch sizes and short k-mers fail to distinguish replicates, whereas k=31 with sketch size >=10^4 efficiently separates technical replicates, so sketch size 10^4 was set as the ARGprofiler default.

## Related tools

- **Mash** (Creates MinHash sketches of sequencing runs and computes all pairwise Mash distances across sketch-size/k-mer-size combinations) — https://github.com/marbl/Mash
- **fastq-dl** (Downloads the short-read FASTQ data for the sewage metagenomes from ENA using project/sample accessions) — https://github.com/rpetit3/fastq-dl
- **CCPhylo** (Dynamic Neighbor-Joining clustering of Mash distance matrices to verify replicate sub-clustering)
- **ARGprofiler** (Pipeline whose Mash sketch step applies the selected default (k=31, sketch size 10^4) for fast searching and comparison of sequencing runs) — https://github.com/genomicepidemiology/ARGprofiler

## Examples

```
mash sketch -k 31 -s 10000 -o sample.msh sample.fastq.gz; mash dist sample.msh all.msh > distances.tsv
```

## Evaluation signals

- For k=31 and sketch size >=10^4, within-sample distances among technical replicate resequencings are lower than between-sample distances among distinct sewage samples
- Very small sketch sizes (e.g. 10^3) and short k-mers (16, 21, 27) fail to clearly distinguish replicates from other samples in the results table
- Dynamic Neighbor-Joining (CCPhylo) trees show technical replicate resequencings sub-clustering together for the accepted parameter combinations
- mash_sketch_benchmark_results.tsv tabulates the replicate-separation outcome for all 16 parameter combinations and supports the choice of sketch size 10^4 as default

## Limitations

- Very small sketch sizes and short k-mer sizes produce distances that cannot reliably separate technical replicates of the same sample from other sewage metagenomes
- ARGprofiler's Mash comparison is designed for short-read sequencing data only; long-read support is not available
- Distances depend on the sketching stochastic sampling; a too-low sketch size increases distance estimation error for near-identical runs

## Evidence

- [abstract] the pipeline also creates Mash sketches for fast searching and comparisons of sequencing runs: "the pipeline also creates Mash sketches for fast searching and comparisons of sequencing runs"
- [discussion] k-mer-based comparison of sequence runs: "k-mer-based comparison of sequence runs"
- [readme] Mash is normally distributed as a dependency-free binary for Linux or OSX: [evidence span withheld — anchor status: external_doc]

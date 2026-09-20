---
name: sketch-parameter-grid-optimization
description: Decision support for choosing Mash sketch-size and k-mer-size defaults
  when pairwise distances must discriminate technical replicates of the same metagenomic
  sample from other samples.
when_to_use_negative:
- Input reads are long-read sequencing data — ARGprofiler is designed to only work
  with short-read sequencing data
- The question concerns ARG abundance or flanking-region assembly rather than read-level
  sequence similarity between sequencing runs
- Distances are already computed at the selected parameters and only need interpretation,
  not a parameter search
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: Mash
  role: Creates MinHash sketches and computes pairwise distances across the sketch-size/k-mer-size
    grid
  repo: https://github.com/marbl/Mash
- name: fastq-dl
  role: Downloads the sewage metagenome FASTQ files from ENA using study accessions
  repo: https://github.com/rpetit3/fastq-dl
- name: CCPhylo
  role: Dynamic Neighbor-Joining clustering of distance matrices to verify replicate
    sub-clustering
- name: ARGprofiler
  role: Pipeline whose Mash sketch default (10^4) is being calibrated by this benchmark
  repo: https://github.com/genomicepidemiology/ARGprofiler
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae086
    title: ARGprofiler—a pipeline for large-scale analysis of antimicrobial resistance
      genes and their flanking regions in metagenomic datasets
  source_content_hash: sha256:b39fb09207078a908dfe53107732b80d6c5cc57648abf929635d56deeb2a50c4
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# sketch-parameter-grid-optimization

## Summary

Benchmarks MinHash sketch sizes and k-mer sizes in a parameter grid to find the combination that reliably separates technical replicates from distinct samples, as done to set ARGprofiler's Mash sketch default to 10^4.

## When to use

Decision support for choosing Mash sketch-size and k-mer-size defaults when pairwise distances must discriminate technical replicates of the same metagenomic sample from other samples.

## When NOT to use

- Input reads are long-read sequencing data — ARGprofiler is designed to only work with short-read sequencing data
- The question concerns ARG abundance or flanking-region assembly rather than read-level sequence similarity between sequencing runs
- Distances are already computed at the selected parameters and only need interpretation, not a parameter search

## Inputs

- FASTQ short-read sequencing files for the metagenomic samples and technical replicate resequencings (e.g. from ENA accessions PRJEB34633 and PRJEB63576)
- Parameter grid specification: sketch sizes 10^3, 10^4, 10^5, 10^6 and k-mer sizes 16, 21, 27, 31

## Outputs

- Pairwise Mash distance matrices for each of the 16 sketch-size/k-mer-size combinations
- Neighbor-joining clusterings (CCPhylo) verifying replicate sub-clustering
- Summary table mash_sketch_benchmark_results.tsv recording replicate-separation outcome per parameter combination

## How to apply

Retrieve the sequencing data (e.g. 72 sewage metagenomes: 36 Copenhagen samples from PRJEB34633, 18 urban sewage samples from Munk et al. 2022, and 18 technical replicate resequencings from PRJEB63576) with fastq-dl. Build Mash 2.3 MinHash sketches across a grid of sketch sizes (10^3, 10^4, 10^5, 10^6) and k-mer sizes (16, 21, 27, 31) and compute all pairwise Mash distances for each of the 16 combinations. For each combination, test whether within-sample replicate distances are lower than between-sample distances, and verify replicate sub-clustering with Dynamic Neighbor-Joining in CCPhylo 0.8.3. In this benchmark, k-mer size 31 with sketch size >=10^4 efficiently separated technical replicates, so sketch size 10^4 was adopted as the ARGprofiler default.

## Related tools

- **Mash** (Creates MinHash sketches and computes pairwise distances across the sketch-size/k-mer-size grid) — https://github.com/marbl/Mash
- **fastq-dl** (Downloads the sewage metagenome FASTQ files from ENA using study accessions) — https://github.com/rpetit3/fastq-dl
- **CCPhylo** (Dynamic Neighbor-Joining clustering of distance matrices to verify replicate sub-clustering)
- **ARGprofiler** (Pipeline whose Mash sketch default (10^4) is being calibrated by this benchmark) — https://github.com/genomicepidemiology/ARGprofiler

## Examples

```
mash sketch -k 31 -s 10000 -o rep_sketch *.fastq.gz; mash dist rep_sketch.msh rep_sketch.msh > pairwise_distances.tsv
```

## Evaluation signals

- In mash_sketch_benchmark_results.tsv, k=31 with sketch size >=10^4 combinations show technical replicates efficiently separated from the remaining samples
- Small sketch sizes (10^3) and short k-mer sizes (16, 21) fail to clearly distinguish replicates from other sewage samples
- Within-sample distances among the 18 technical replicate resequencings are lower than between-sample distances among distinct samples for passing combinations
- Dynamic Neighbor-Joining (CCPhylo 0.8.3) clusterings show the replicate samples sub-clustering together at the selected parameters

## Limitations

- Small sketch sizes give unstable distance estimates that fail to separate technical replicates from distinct sewage samples
- The benchmark is specific to short-read sewage metagenomes; other sample types or sequencing technologies may require re-tuning
- ARGprofiler only supports short-read sequencing data, so the tuned parameters cannot be transferred to long-read workflows

## Evidence

- [abstract] the pipeline also creates Mash sketches for fast searching and comparisons of sequencing runs: "the pipeline also creates Mash sketches for fast searching and comparisons of sequencing runs"
- [other] technical replicates efficiently separated with k-mer size 31 and sketch size >=10^4: [evidence span withheld — anchor status: not_found]
- [discussion] k-mer-based comparison of sequence runs: "k-mer-based comparison of sequence runs"
- [readme] Mash is normally distributed as a dependency-free binary for Linux or OSX: [evidence span withheld — anchor status: external_doc]
- [readme] fastq-dl downloads FASTQ files from ENA or SRA: [evidence span withheld — anchor status: external_doc]

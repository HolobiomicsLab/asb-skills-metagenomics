---
name: dynamic-neighbor-joining-distance-clustering
description: Use when tuning pairwise sequence-distance parameters (e.g. Mash sketch
  size, k-mer size) so that replicate or same-source samples cluster apart from distinct
  samples in a distance-matrix analysis.
when_to_use_negative:
- Input is long-read sequencing data — ARGprofiler and this benchmark are designed
  only for short-read data
- You need exact phylogenetic reconstruction rather than replicate-separation diagnostics
  of sketch-based distances
- No replicate or same-source samples are available to validate whether parameter
  settings distinguish within-sample from between-sample distances
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: Mash
  role: Creates MinHash sketches and computes pairwise distances for each sketch-size/k-mer-size
    parameter combination
  repo: https://github.com/marbl/Mash
- name: CCPhylo
  role: Implements Dynamic Neighbor-Joining used to verify technical replicates sub-cluster
    in the distance matrices
- name: fastq-dl
  role: Downloads the 72 sewage metagenome short-read datasets from ENA
  repo: https://github.com/rpetit3/fastq-dl
- name: ARGprofiler
  role: Pipeline whose Mash sketch defaults are being tuned via this benchmark
  repo: https://github.com/genomicepidemiology/ARGprofiler
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae086
    title: ARGprofiler—a pipeline for large-scale analysis of antimicrobial resistance
      genes and their flanking regions in metagenomic datasets
  source_content_hash: sha256:28c1d9a827ad4770ef3e493c5a4774aeea49ca972e383d70fd0e419ddf3a2d21
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marbl/Mash
  tools:
  - ARGprofiler
  - CCPhylo
  - Mash
  - fastq-dl
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Dynamic Neighbor-Joining Distance Clustering

## Summary

Benchmarking and tuning of Mash sketch and k-mer parameters on 72 sewage metagenomes so that MinHash distances reliably separate technical replicates from distinct samples, validated by clustering distance matrices with Dynamic Neighbor-Joining (CCPhylo 0.8.3).

## When to use

Use when tuning pairwise sequence-distance parameters (e.g. Mash sketch size, k-mer size) so that replicate or same-source samples cluster apart from distinct samples in a distance-matrix analysis.

## When NOT to use

- Input is long-read sequencing data — ARGprofiler and this benchmark are designed only for short-read data
- You need exact phylogenetic reconstruction rather than replicate-separation diagnostics of sketch-based distances
- No replicate or same-source samples are available to validate whether parameter settings distinguish within-sample from between-sample distances

## Inputs

- 72 sewage metagenomic short-read FASTQ files (36 Copenhagen sewage samples from PRJEB34633, 18 Munk et al. 2022 urban sewage samples, 18 technical replicate resequencings of one Copenhagen sample from PRJEB63576)
- Mash 2.3 MinHash sketches at sketch sizes 10^3–10^6 and k-mer sizes 16, 21, 27, 31
- All pairwise Mash distance matrices for each of the 16 parameter combinations

## Outputs

- mash_sketch_benchmark_results.tsv tabulating replicate-separation outcome per sketch-size/k-mer-size combination
- Dynamic Neighbor-Joining clusterings (CCPhylo 0.8.3) of each distance matrix showing whether technical replicates sub-cluster
- Selected default parameters for ARGprofiler (k-mer size 31, sketch size 10^4)

## How to apply

Retrieve the sequencing runs (here 72 sewage metagenomes from PRJEB34633, Munk et al. 2022 urban sewage, and 18 technical replicate resequencings from PRJEB63576) with fastq-dl, then build Mash 2.3 MinHash sketches for a grid of sketch sizes (10^3, 10^4, 10^5, 10^6) and k-mer sizes (16, 21, 27, 31). Compute all pairwise Mash distances for each of the 16 parameter combinations. For each combination, check whether within-sample distances among technical replicate resequencings are lower than between-sample distances among distinct sewage samples. Confirm sub-clustering of replicates by clustering each distance matrix with Dynamic Neighbor-Joining in CCPhylo 0.8.3, then tabulate replicate-separation outcomes per combination and adopt the smallest parameters that separate replicates cleanly — the paper settled on k-mer size 31 with sketch size 10^4 as the ARGprofiler default.

## Related tools

- **Mash** (Creates MinHash sketches and computes pairwise distances for each sketch-size/k-mer-size parameter combination) — https://github.com/marbl/Mash
- **CCPhylo** (Implements Dynamic Neighbor-Joining used to verify technical replicates sub-cluster in the distance matrices)
- **fastq-dl** (Downloads the 72 sewage metagenome short-read datasets from ENA) — https://github.com/rpetit3/fastq-dl
- **ARGprofiler** (Pipeline whose Mash sketch defaults are being tuned via this benchmark) — https://github.com/genomicepidemiology/ARGprofiler

## Examples

```
mash sketch -k 31 -s 10000 *.fastq.gz && mash dist *.msh > mash_distances.tsv
```

## Evaluation signals

- k=31 with sketch size >=10^4 combinations show technical replicates efficiently separated from all remaining sewage samples
- Sketch size 10^3 and short k-mers (16, 21, 27) fail to clearly distinguish replicates from distinct samples
- Dynamic Neighbor-Joining (CCPhylo 0.8.3) trees show the 18 replicate resequencings sub-clustering together in each validated parameter combination
- Results table covers all 16 sketch-size/k-mer-size combinations and is written to mash_sketch_benchmark_results.tsv

## Limitations

- Very small sketch sizes and short k-mer sizes do not reliably separate technical replicates from other sewage metagenomes
- Mash sketch quality depends on available sequencing depth; sketch size bounds the sensitivity of the MinHash estimate

## Evidence

- [results] Small sketch sizes and short k-mers failed to distinguish technical replicates from other sewage samples, whereas technical replicates were efficiently separated with k-mer size 31 and sketch size >=10^4: [evidence span withheld — anchor status: not_found]
- [abstract] Mash sketches for fast searching and comparison: "the pipeline also creates Mash sketches for fast searching and comparisons of sequencing runs"
- [discussion] k-mer-based comparison of sequence runs: "k-mer-based comparison of sequence runs"
- [readme] fastq-dl README: [evidence span withheld — anchor status: external_doc]

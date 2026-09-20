---
name: alignment-free-genome-distance-computation
description: Use when you need pairwise dissimilarities among whole genomes or unassembled
  shotgun reads without alignment (e.g. phylogeny recovery, metagenome comparison),
  especially where background-adjusted k-mer measures are required.
when_to_use_negative:
- The input is already a distance/feature table or the sequences require detailed
  alignment information (mutation positions, gene boundaries) rather than a genome-level
  dissimilarity
- Only a handful of very short sequences where an alignment-based tool such as BLAST
  would be both feasible and more informative
- You need a strictly normalized phylogenetic tree with substitution models rather
  than an alignment-free clustering tree
edam_operation: http://edamontology.org/operation_0289
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3293
tools:
- name: CAFE
  role: Standalone executable/GUI that computes all 28 alignment-free dissimilarity
    measures and writes PHYLIP-format output
  repo: https://github.com/younglululu/CAFE
- name: jellyfish
  role: Accelerates k-mer counting inside CAFE via the -J option (e.g. jellyfish-2.2.4)
- name: Phylip
  role: Consumes CAFE's PHYLIP dissimilarity matrices for hierarchical clustering
    (neighbour-joining / UPGMA dendrograms)
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkx351
    title: 'CAFE: aCcelerated Alignment-FrEe sequence analysis'
  source_content_hash: sha256:014aa766f98605dfdc6899da4be9a9828d8db1a8f230ca73242f464f390ef105
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/younglululu/CAFE
  repo_url: https://github.com/younglululu/CAFE
  tools:
  - CAFE
  - JELLYFISH
  - Phylip
  - jellyfish
  edam_operation: http://edamontology.org/operation_0289
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3293
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# alignment-free-genome-distance-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute pairwise alignment-free dissimilarity matrices among genomic sequences or NGS shotgun reads using CAFE, which integrates 28 measures (conventional k-mer counts, background-adjusted D2star/D2shepp/CVtree, and presence/absence measures) accelerated via JELLYFISH k-mer counting, with PHYLIP-format output for downstream clustering or visualization.

## When to use

Use when you need pairwise dissimilarities among whole genomes or unassembled shotgun reads without alignment (e.g. phylogeny recovery, metagenome comparison), especially where background-adjusted k-mer measures are required.

## When NOT to use

- The input is already a distance/feature table or the sequences require detailed alignment information (mutation positions, gene boundaries) rather than a genome-level dissimilarity
- Only a handful of very short sequences where an alignment-based tool such as BLAST would be both feasible and more informative
- You need a strictly normalized phylogenetic tree with substitution models rather than an alignment-free clustering tree

## Inputs

- Assembled genome FASTA files or unassembled NGS shotgun reads with extension '.fasta', '.fa', or '.fna' (comma-separated list via -I or a folder via -F)
- JELLYFISH executable path for accelerated k-mer counting (e.g. jellyfish-2.2.4/bin/./jellyfish)

## Outputs

- Pairwise dissimilarity matrix in standard PHYLIP format (e.g. result.Ma.phylip), one per selected measure via -O
- Plain-text distance output (default -T plain) or cytoscape/mds formats for network and 2D projection analyses
- Downstream dendrogram, heatmap, PCoA, or network visualizations via the GUI

## How to apply

Run the stand-alone executable ./cafe with -I (comma-separated FASTA files) or -F (folder of .fasta/.fa/.fna files), select measures with -D (e.g. Ma, D2star, D2shepp, CVtree, Jaccard), set k-mer length with -K (e.g. 10 or 14), and optionally fix the Markov order with -M (a single value, a per-file list, or -1 to auto-infer via BIC). Pass -J with the jellyfish executable path (e.g. jellyfish-2.2.4/bin/./jellyfish) to accelerate k-mer counting and -S to cache k-mer count binary files for reuse. Use -O to name the output file and -T to choose the downstream format (plain, phylip for hierarchical clustering, cytoscape, or mds); add -R to count reverse-complement k-mers when working with NGS reads. Background-adjusted measures (CVtree, D2star, D2shepp) markedly outperform non-adjusted measures on genome-scale comparisons, so prefer them for phylogenetic benchmarking; build UPGMA or neighbour-joining trees from the PHYLIP matrices to evaluate recovery of known groups.

## Related tools

- **CAFE** (Standalone executable/GUI that computes all 28 alignment-free dissimilarity measures and writes PHYLIP-format output) — https://github.com/younglululu/CAFE
- **jellyfish** (Accelerates k-mer counting inside CAFE via the -J option (e.g. jellyfish-2.2.4))
- **Phylip** (Consumes CAFE's PHYLIP dissimilarity matrices for hierarchical clustering (neighbour-joining / UPGMA dendrograms))

## Examples

```
./cafe -M 0 -O output_path -S model_dir -T plain -I speciesA.fa,speciesB.fa -J /panfs/cmb-panasas2/ylu465/jellyfish-2.2.4/bin/./jellyfish -K 10 -D D2star,Ma
```

## Evaluation signals

- Output file named via -O exists and is in standard PHYLIP lower-triangular format with one row/column per input FASTA
- On the 21-primate benchmark (K=14, M=12), background-adjusted measures (D2star, D2shepp, CVtree) yield Spearman correlations against evolutionary distances at least as high as non-adjusted k-mer count measures
- On the 27 E. coli/Shigella genomes (D2S/D2shepp, K=14, M=1, UPGMA tree), every ECOR group except A and B2 forms a monophyletic cluster
- With -S, cached k-mer count binary files appear in the model folder (one per input FASTA and Markov model)
- Rerunning the same -I, -K, -M, -D settings reproduces identical pairwise dissimilarities (deterministic given fixed parameters)

## Limitations

- The original implementations of D2star and D2S are slow due to calculating expected k-mer counts; CAFE mitigates but computational cost still grows with genome size and k
- The potential Markov Order is usually unclear to the user; the -1 auto-inference via BIC is described as 'simple yet time-consuming'
- On Windows, the default cafe_win.exe may not run due to User Account Control permissions and may need renaming to cafe_win_2.exe/cafe_win_3.exe variants
- No software version number or license is stated for CAFE in the article; the repository has no changelog

## Evidence

- [abstract] a standalone software, aCcelerated Alignment-FrEe sequence analysis (CAFE), for efficient calculation of 28 alignment-free dissimilarity measures: "we report a standalone software, aCcelerated Alignment-FrEe sequence analysis (CAFE), for efficient calculation of 28 alignment-free dissimilarity measures"
- [abstract] CAFE output format and jellyfish acceleration: "CAFE allows for both assembled genome sequences and unassembled NGS shotgun reads as input, and wraps the output in a standard PHYLIP format"
- [methods] -J jellyfish option: "-J < jfexe_path >: Use jellyfish to accelerate kmer counting"
- [readme] Markov order auto-inference caveat: "The simple yet time-consuming way is to choose '-1' as inferring the optimal Markov Order automatically by using the Bayesian Information Criterion (BIC)."
- [results] d2S recovers ECOR groups: [evidence span withheld — anchor status: not_found]

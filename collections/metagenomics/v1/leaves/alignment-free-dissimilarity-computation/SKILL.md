---
name: alignment-free-dissimilarity-computation
description: Use when input is assembled genomic FASTA files or unassembled NGS shotgun
  reads (.fasta/.fa/.fna) and the goal is a pairwise alignment-free dissimilarity
  matrix for clustering, PCoA, heatmap or network analysis.
when_to_use_negative:
- The question requires detailed alignment information (e.g. mutation positions, homologous
  region annotation) — the article notes alignment-based methods are inappropriate
  for some tasks, but alignment-free distances discard per-site alignment detail.
- Input sequences are not in FASTA format with extensions .fasta/.fa/.fna — the CAFE
  input loaders expect these extensions.
- 'JS divergence with -M: the README states automatic Markov order inference (''-1'')
  is ''not suitable for JS'', so BIC-based order selection cannot be used for that
  measure.'
edam_operation: http://edamontology.org/operation_0289
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3293
tools:
- name: CAFE
  role: Standalone executable/GUI that computes the 28 alignment-free dissimilarity
    measures and wraps output in PHYLIP format
  repo: https://github.com/younglululu/CAFE
- name: JELLYFISH
  role: Accelerates the k-mer counting step via the -J option (jellyfish 2.2.4 referenced
    in the README)
- name: Phylip
  role: Downstream hierarchical clustering of the emitted PHYLIP dissimilarity matrix
  repo: http://evolution.genetics.washington.edu/phylip.html
provenance:
  source_task_ids:
  - task_005
  - task_001
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkx351
    title: 'CAFE: aCcelerated Alignment-FrEe sequence analysis'
  source_content_hash: sha256:33cb69a8f6e193d8c1d569498c5161f0ca97ab457b15eb61af16d5442e681fdc
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
  edam_operation: http://edamontology.org/operation_0289
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3293
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# alignment-free-dissimilarity-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute pairwise dissimilarity matrices among genomes or metagenomes without alignment, using CAFE's 28 k-mer-based measures (conventional count measures like Manhattan, background-adjusted measures like D2star/D2shepp/CVtree, and presence/absence measures like Jaccard), accelerated by JELLYFISH k-mer counting.

## When to use

Use when input is assembled genomic FASTA files or unassembled NGS shotgun reads (.fasta/.fa/.fna) and the goal is a pairwise alignment-free dissimilarity matrix for clustering, PCoA, heatmap or network analysis.

## When NOT to use

- The question requires detailed alignment information (e.g. mutation positions, homologous region annotation) — the article notes alignment-based methods are inappropriate for some tasks, but alignment-free distances discard per-site alignment detail.
- Input sequences are not in FASTA format with extensions .fasta/.fa/.fna — the CAFE input loaders expect these extensions.
- JS divergence with -M: the README states automatic Markov order inference ('-1') is 'not suitable for JS', so BIC-based order selection cannot be used for that measure.

## Inputs

- FASTA sequence files (.fasta, .fa, .fna) — assembled genomes or NGS shotgun reads
- CAFE command-line options: -D measure list, -K k-mer length, -M Markov order, -J jellyfish path, -R, -L, -S model dir

## Outputs

- Pairwise dissimilarity matrix in standard PHYLIP format (e.g. result.Ma.phylip)
- Plain / mds / cytoscape output formats via -T
- Caches of per-file k-mer count binary model files via -S

## How to apply

Load the FASTA inputs into CAFE (GUI or the stand-alone ./cafe executable) and select one or more comma-separated distance measures with -D (e.g. -D D2star,Ma,CVtree from the 28 available: 10 conventional k-mer count measures, 3 background-adjusted measures, 15 presence/absence measures). Set the k-mer length with -K (default 8) and, for the background-adjusted measures D2star, D2shepp and CVtree, choose the Markov order with -M; use -M -1 (the default) to infer the optimal order automatically via the Bayesian Information Criterion. Optionally pass -J with the path to the jellyfish executable (e.g. jellyfish 2.2.4) to accelerate k-mer counting, -R to include reverse-complement k-mers for shotgun reads, and -L for a minimum k-mer occurrence threshold. Export the symmetric dissimilarity matrix in standard PHYLIP format using -T phylip with -O result.phylip, or use -T plain/mds/cytoscape for other downstream formats; the matrix then feeds dendrogram (neighbour-joining), PCoA, heatmap, or network visualization.

## Related tools

- **CAFE** (Standalone executable/GUI that computes the 28 alignment-free dissimilarity measures and wraps output in PHYLIP format) — https://github.com/younglululu/CAFE
- **JELLYFISH** (Accelerates the k-mer counting step via the -J option (jellyfish 2.2.4 referenced in the README))
- **Phylip** (Downstream hierarchical clustering of the emitted PHYLIP dissimilarity matrix) — http://evolution.genetics.washington.edu/phylip.html

## Examples

```
./cafe -M 0 -O output_path -S model_dir -T plain -I speciesA.fa,speciesB.fa -J jellyfish-2.2.4/bin/./jellyfish -K 10 -D D2star,Ma
```

## Evaluation signals

- The output file parses as a standard PHYLIP lower-triangular/square distance matrix with n sequence labels and a symmetric off-diagonal structure.
- On random genome pairs with -D D2star, CAFE's runtime and peak memory are in the vicinity of the reported 24.0x speedup and ~55.3% peak memory consumption versus the original Reinert et al. implementation.
- For the mammalian gut metagenome benchmark (28 samples, -D D2shepp -K 5 -M 0), UPGMA clustering of the matrix separates the three dietary groups (hindgut fermenters, foregut fermenters, carnivores) into distinct, non-interleaved clusters.
- Recomputing with a chosen fixed measure (e.g. -D Ma -K 8) reproduces identical off-diagonal distances to a reference run.
- Running progress is traceable in the console/log; each input FASTA corresponds to a saved k-mer count model file when -S is used.

## Limitations

- Original implementations of the background-adjusted statistics (D2star, D2S) were slow due to calculating expected k-mer counts; CAFE mitigates but the BIC-based Markov order inference with -M -1 is described as 'the simple yet time-consuming way'.
- Automatic Markov order inference (-M -1) is not suitable for the JS measure.
- On Windows, cafe_win.exe may fail due to User Account Control permission settings; a variant executable (cafe_win_2.exe / cafe_win_3.exe) may need to be renamed.
- No software version number or license is stated in the article/repository for CAFE; no changelog is available.

## Evidence

- [readme] CAFE works with sequence data, both long genomic sequences and shotgun sequence reads from NGS technologies, and subsequently generates pairwise dissimilarities among the sequences as output.: "CAFE works with sequence data, both long genomic sequences and shotgun sequence reads from NGS technologies, and subsequently generates pairwise dissimilarities among the sequences as output."
- [abstract] CAFE ... for efficient calculation of 28 alignment-free dissimilarity measures: "a standalone software, aCcelerated Alignment-FrEe sequence analysis (CAFE), for efficient calculation of 28 alignment-free dissimilarity measures"
- [readme] The default Markov Order is -1 as inferring the optimal Markov Order automatically by using the Bayesian Information Criterion (BIC).: "The default Markov Order is -1 as inferring the optimal Markov Order automatically by using the Bayesian Information Criterion (BIC)."
- [readme] The order value could be non-negative integer but less than Kmer length or "-1" ... (not suitable for JS).: "The order value could be non-negative integer but less than Kmer length or "-1" with the special intention to automatically infer the suitable order (not suitable for JS)."
- [intro] CAFE ... counts k-mers by JELLYFISH: "CAFE works with sequence data, both assembled genomic sequences and unassembled shotgun sequence reads from NGS technologies and counts k-mers by JELLYFISH (20)"

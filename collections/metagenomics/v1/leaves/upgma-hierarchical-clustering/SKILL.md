---
name: upgma-hierarchical-clustering
description: Use when a lower-triangular pairwise distance matrix in PHYLIP format
  (e.g. d2S from CAFE with -K 14, -M 1) must be converted into a hierarchical clustering
  topology to test whether predefined groups form monophyletic clades.
when_to_use_negative:
- The input is already a phylogeny or an alignment — use alignment-based tree building
  rather than clustering an alignment-free distance matrix
- Only k-mer count vectors are available and no pairwise distance matrix has been
  computed — CAFE must first produce the PHYLIP distances
- The downstream visualization expected by the user is CAFE's built-in dendrogram,
  which uses neighbour-joining rather than UPGMA
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3293
tools:
- name: CAFE
  role: Computes the d2S (D2shepp) pairwise dissimilarity matrix from genome FASTA
    files and writes it in PHYLIP format via -T phylip
  repo: https://github.com/younglululu/CAFE
- name: JELLYFISH
  role: Accelerates CAFE's k-mer counting when invoked with -J <jellyfish executable
    path>
- name: Phylip
  role: Hierarchical clustering (UPGMA) of the parsed lower-triangular distance matrix
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkx351
    title: 'CAFE: aCcelerated Alignment-FrEe sequence analysis'
  source_content_hash: sha256:78b44261b84c83c2b7ae432821716ef77c0f82fdb149d9ef872df642ebb43145
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
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3293
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# upgma-hierarchical-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds a UPGMA hierarchical clustering tree from a CAFE-computed alignment-free d2S distance matrix (exported in PHYLIP format) so that sequence relationships among genomes — e.g. the 27 E. coli/Shigella genomes and their ECOR groups — can be assessed via clade monophyly.

## When to use

Use when a lower-triangular pairwise distance matrix in PHYLIP format (e.g. d2S from CAFE with -K 14, -M 1) must be converted into a hierarchical clustering topology to test whether predefined groups form monophyletic clades.

## When NOT to use

- The input is already a phylogeny or an alignment — use alignment-based tree building rather than clustering an alignment-free distance matrix
- Only k-mer count vectors are available and no pairwise distance matrix has been computed — CAFE must first produce the PHYLIP distances
- The downstream visualization expected by the user is CAFE's built-in dendrogram, which uses neighbour-joining rather than UPGMA

## Inputs

- 27 E. coli/Shigella genome FASTA files (.fasta/.fa/.fna)
- CAFE output distance matrix in PHYLIP format (e.g. result.D2shepp.phylip)
- Tip-to-group label map assigning each genome to an ECOR group (A, B1, B2, D, E, S)

## Outputs

- Newick-format UPGMA clustering tree over the 27 genome tips
- Tab-separated per-ECOR-group monophyly report

## How to apply

Run CAFE in command-line mode with -D D2shepp, -K 14, -M 1, optionally -J <jellyfish path> to accelerate k-mer counting, and -T phylip so the d2S pairwise distances are written in standard PHYLIP format (e.g. result.D2shepp.phylip). Parse the lower-triangular matrix and build a UPGMA tree (Phylip's UPGMA option or scipy.cluster.hierarchy linkage with method='average'), exporting the topology as a Newick string. Note that CAFE's built-in GUI dendrogram uses neighbour-joining; for UPGMA specifically, feed the saved PHYLIP result into an external clustering step. Map each tip (the 27 E. coli/Shigella FASTA filenames) to its ECOR group label (A, B1, B2, D, E, or S) and test clade monophyly per group on the tree. Rationale: d2S is a background-adjusted k-mer measure (Markov order -M; default -1 infers the optimal order via BIC), so its pairwise distances are suitable input for agglomerative clustering of complete genomes without alignment.

## Related tools

- **CAFE** (Computes the d2S (D2shepp) pairwise dissimilarity matrix from genome FASTA files and writes it in PHYLIP format via -T phylip) — https://github.com/younglululu/CAFE
- **JELLYFISH** (Accelerates CAFE's k-mer counting when invoked with -J <jellyfish executable path>)
- **Phylip** (Hierarchical clustering (UPGMA) of the parsed lower-triangular distance matrix)

## Examples

```
./cafe -D D2shepp -I seq1.fa,...,seq27.fa -K 14 -M 1 -J jellyfish-2.2.4/bin/./jellyfish -T phylip -O result.D2shepp.phylip
```

## Evaluation signals

- The PHYLIP output file parses as a valid 27x27 lower-triangular matrix whose row/column names match the input FASTA filenames
- The exported Newick tree contains exactly 27 tips corresponding one-to-one with the input genomes
- Every ECOR group forms a monophyletic cluster except groups A and B2, reproducing the Figure 5 finding
- Rerunning CAFE with the same -K 14, -M 1 parameters yields a consistent (same topology) distance matrix and tree

## Limitations

- CAFE's built-in dendrogram visualization uses the neighbour-joining algorithm, not UPGMA; UPGMA must be performed externally on the saved PHYLIP file
- The Markov order for background adjustment is unclear to users; choosing -1 triggers automatic BIC-based inference, which is described as time-consuming
- Windows users may need to rename cafe_win_2.exe/cafe_win_3.exe to cafe_win.exe due to User Account Control permission settings before the executable works
- No software version number or license is stated for CAFE in the article or README

## Evidence

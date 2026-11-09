---
name: spearman-rank-correlation-analysis
description: Use when you have pairwise dissimilarity matrices from multiple distance
  measures and an independent reference phylogenetic/evolutionary distance set, and
  need to rank which measures recover known evolutionary relationships.
when_to_use_negative:
- Inputs are unassembled NGS reads or metagenomes where no reliable reference evolutionary
  distances exist to benchmark against
- The goal is distance-based clustering/visualization (dendrogram, heatmap, PCoA,
  network) rather than benchmarking measure performance against a reference
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_0622
tools:
- name: CAFE
  role: Computes the 28 alignment-free dissimilarity measures (with -K 14, -M 12)
    and outputs PHYLIP-format pairwise matrices via -T phylip
  repo: https://github.com/younglululu/CAFE
- name: jellyfish 2.2.4
  role: Accelerates k-mer counting during CAFE distance calculations via the -J option
- name: Phylip
  role: Output format (-T phylip) for the dissimilarity matrices consumed downstream
    by the Ape R package
  repo: http://evolution.genetics.washington.edu/phylip.html
- name: Ape (R package)
  role: Converts PHYLIP dissimilarity matrices and the reference tree into evolutionary
    (patristic) distances before correlation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkx351
    title: 'CAFE: aCcelerated Alignment-FrEe sequence analysis'
  source_content_hash: sha256:e6cbc3eee528f846bd2808c2caa378281f1ce569cacb1837652930f722c34845
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
  - Ape (R package)
  - CAFE
  - Phylip
  - jellyfish 2.2.4
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# spearman-rank-correlation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark alignment-free genome dissimilarity measures by computing the Spearman rank correlation between each measure's pairwise distances and reference evolutionary (patristic) distances, as done for the 21-primate-genome comparison in the CAFE paper.

## When to use

Use when you have pairwise dissimilarity matrices from multiple distance measures and an independent reference phylogenetic/evolutionary distance set, and need to rank which measures best recover known evolutionary relationships.

## When NOT to use

- Inputs are unassembled NGS reads or metagenomes where no reliable reference evolutionary distances exist to benchmark against
- The goal is distance-based clustering/visualization (dendrogram, heatmap, PCoA, network) rather than benchmarking measure performance against a reference

## Inputs

- 21 primate genome sequence FASTA files (.fasta/.fa/.fna)
- Reference primate phylogenetic tree with branch lengths (for Ape patristic distances)
- CAFE stand-alone executable (./cafe) and jellyfish 2.2.4 executable path

## Outputs

- 28 PHYLIP-format pairwise dissimilarity matrices (one per distance measure, e.g. result.Ma.phylip)
- spearman_correlations.csv with one row per dissimilarity measure and its Spearman coefficient

## How to apply

Run CAFE's stand-alone executable on a comma-separated list of 21 primate genome FASTA files with -K 14 (k-mer length), -M 12 (Markov order for background-adjusted measures D2star, D2shepp, JS), and -D covering all 28 measures (11 conventional k-mer count measures such as Ch, Canberra, Chisq, Cosine, Co-phylog, D2, Eu, FFP, JS, Ma, Pearson; 3 background-adjusted measures CVtree, D2shepp, D2star; and presence/absence measures such as Jaccard and Dice). Pass the jellyfish 2.2.4 executable path via -J to accelerate k-mer counting, and write each measure's pairwise matrix in PHYLIP format with -O and -T phylip. Using the Ape R package, convert the reference tree's branch lengths into patristic distances, then compute the Spearman rank correlation between each CAFE matrix and the reference distances across all genome pairs. Assemble the 28 per-measure coefficients into spearman_correlations.csv and compare background-adjusted versus non-adjusted measures.

## Related tools

- **CAFE** (Computes the 28 alignment-free dissimilarity measures (with -K 14, -M 12) and outputs PHYLIP-format pairwise matrices via -T phylip) — https://github.com/younglululu/CAFE
- **jellyfish 2.2.4** (Accelerates k-mer counting during CAFE distance calculations via the -J option)
- **Phylip** (Output format (-T phylip) for the dissimilarity matrices consumed downstream by the Ape R package) — http://evolution.genetics.washington.edu/phylip.html
- **Ape (R package)** (Converts PHYLIP dissimilarity matrices and the reference tree into evolutionary (patristic) distances before correlation)

## Examples

```
./cafe -K 14 -M 12 -T phylip -O out_dir/ -I p1.fa,p2.fa,...,p21.fa -J jellyfish-2.2.4/bin/./jellyfish -D D2star,D2shepp,CVtree,Ma,Eu,Jaccard,Dice; in R: cor(cafe_dist, patristic_dist, method="spearman")
```

## Evaluation signals

- spearman_correlations.csv exists and contains one row per dissimilarity measure (28 rows for the full CAFE -D list)
- Background-adjusted measures (D2star, D2shepp, CVtree) yield Spearman correlations at least as high as non-background-adjusted k-mer count measures, consistent with Figure 3 of the paper
- Each PHYLIP matrix is a valid 21x21 lower-triangular dissimilarity matrix matching the input FASTA list
- Correlations are computed across all primate genome pairs against the Ape-derived patristic distances, not raw branch lengths

## Limitations

- The Markov order (-M) is required for D2star, D2shepp and JS; if unclear, -1 triggers automatic BIC-based order inference, which is described as 'simple yet time-consuming'
- On Windows, the default cafe_win.exe may not work due to User Account Control permission settings; alternate executables (cafe_win_2.exe, cafe_win_3.exe) may need renaming
- No software version number or license is stated for CAFE in the article or repository

## Evidence

- [other] Workflow step from source card on patristic conversion and correlation: [evidence span withheld — anchor status: not_found]

---
name: kmer-count-dissimilarity-measure-selection
description: Use when comparing whole genomes or metagenomes alignment-free with k-mer
  counts — e.g. 21 primate genomes at k=14, Markov order 12 — and you must choose
  among 28 dissimilarity measures.
when_to_use_negative:
- When sequences require full base-level alignment detail (e.g. gene regulatory region
  homology or exact variant calling), since alignment-free measures only give genome-level
  dissimilarities
- When input is not raw sequence data (e.g. already-computed distance matrices or
  feature tables) — use downstream PHYLIP-loading/visualization instead of recomputing
  distances
- When k-mer Markov order is unknown and sequences are too short for BIC-based order
  inference via -M -1, background-adjusted measures may be unreliable
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0092
tools:
- name: CAFE
  role: Stand-alone executable (./cafe) that computes the 28 alignment-free dissimilarity
    measures and outputs PHYLIP-format matrices
  repo: https://github.com/younglululu/CAFE
- name: jellyfish 2.2.4
  role: Accelerates k-mer counting during distance calculation, invoked via the -J
    flag
- name: Phylip
  role: Output format (-T phylip) for the dissimilarity matrices so they can be consumed
    by downstream hierarchical clustering and R analysis
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkx351
    title: 'CAFE: aCcelerated Alignment-FrEe sequence analysis'
  source_content_hash: sha256:7adaedd230b2d0feeef28d2e2a5e5a3cf3195f5ee6b87d8040ca66ff7aa0b144
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
  - Phylip
  - jellyfish 2.2.4
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# kmer-count-dissimilarity-measure-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Selects and runs alignment-free k-mer dissimilarity measures (conventional k-mer count, background-adjusted D2star/D2shepp/CVtree, and presence/absence measures) via CAFE, and benchmarks them against evolutionary distances using Spearman rank correlation.

## When to use

Use when comparing whole genomes or metagenomes alignment-free with k-mer counts — e.g. 21 primate genomes at k=14, Markov order 12 — and you must choose among 28 dissimilarity measures.

## When NOT to use

- When sequences require full base-level alignment detail (e.g. gene regulatory region homology or exact variant calling), since alignment-free measures only give genome-level dissimilarities
- When input is not raw sequence data (e.g. already-computed distance matrices or feature tables) — use downstream PHYLIP-loading/visualization instead of recomputing distances
- When k-mer Markov order is unknown and sequences are too short for BIC-based order inference via -M -1, background-adjusted measures may be unreliable

## Inputs

- Comma-separated list of primate genome FASTA files (.fasta/.fa/.fna)
- Reference evolutionary (patristic) distances from a phylogeny
- jellyfish executable path (e.g. jellyfish-2.2.4/bin/./jellyfish)

## Outputs

- One PHYLIP-format pairwise dissimilarity matrix per measure (e.g. result.Ma.phylip)
- Table of 28 per-measure Spearman rank correlation coefficients saved as spearman_correlations.csv

## How to apply

Run the CAFE stand-alone executable on a comma-separated FASTA file list with -K 14 and -M 12, invoking all 28 measures via -D (11 conventional k-mer count measures such as Ma, Eu, Ch, Cosine, JS, FFP; 3 background-adjusted measures D2star, D2shepp, CVtree; presence/absence measures such as Jaccard, Dice, Kulczynski). Pass the jellyfish executable with -J (e.g. jellyfish-2.2.4) to accelerate k-mer counting. Write each measure's pairwise dissimilarity matrix with -O in PHYLIP format (-T phylip) for downstream analysis. Convert PHYLIP matrices to evolutionary (patristic) distances with the Ape R package, then compute Spearman rank correlation between each measure's distances and reference evolutionary distances across all genome pairs. Rationale: on primate and vertebrate genomes, background-adjusted measures (CVTree, d2*, d2S) markedly outperform non-background-adjusted ones because they correct k-mer counts with a Markov sequence model (set via -M; -1 invokes BIC-based automatic order inference).

## Related tools

- **CAFE** (Stand-alone executable (./cafe) that computes the 28 alignment-free dissimilarity measures and outputs PHYLIP-format matrices) — https://github.com/younglululu/CAFE
- **jellyfish 2.2.4** (Accelerates k-mer counting during distance calculation, invoked via the -J flag)
- **Phylip** (Output format (-T phylip) for the dissimilarity matrices so they can be consumed by downstream hierarchical clustering and R analysis)

## Examples

```
./cafe -M 12 -O output_path -T phylip -I speciesA.fa,speciesB.fa,...,speciesU.fa -J jellyfish-2.2.4/bin/./jellyfish -K 14 -D D2star,D2shepp,CVtree,Ma,Eu,Ch
```

## Evaluation signals

- spearman_correlations.csv exists and contains exactly one row per dissimilarity measure (28 rows)
- Background-adjusted measures (D2star, D2shepp, CVtree) yield Spearman correlations at least as high as non-background-adjusted k-mer count measures, consistent with Figure 3 of the paper
- Each PHYLIP matrix is a square 21x21 lower-triangle dissimilarity matrix matching the input FASTA list order
- PHYLIP output files load correctly in the Ape R package for patristic distance conversion
- All measures complete without error when jellyfish is supplied via -J

## Limitations

- Background-adjusted measures (D2star, D2shepp) require specifying a Markov order (-M); the default -1 infers the optimal order via BIC but is time-consuming and not suitable for JS
- The original implementations of d2* and d2S are slow due to calculating expected k-mer counts; CAFE mitigates but does not eliminate computational cost
- No software version number or license information is stated for CAFE, and no changelog is available for the repository

## Evidence

- [results] For the primate (and vertebrate) genome comparisons, the background-adjusted dissimilarity measures (CVTree, d2*, d2S) markedly outperform the non-background-adjusted measures.: [evidence span withheld — anchor status: not_found]

---
name: kmer-frequency-counting
description: Use when input is FASTA genomic sequences (.fasta/.fa/.fna) or unassembled
  NGS shotgun reads, and alignment-free pairwise distances (e.g. Ma, D2star, CVtree,
  Jaccard) must be computed from k-mer count profiles.
when_to_use_negative:
- Sequences are not in FASTA format (.fasta/.fa/.fna) — e.g. raw FASTQ or other containers
  not accepted by the CAFE data selector
- Full alignment-based comparison is required (e.g. gene regulatory regions where
  homologous positions must be resolved) rather than alignment-free k-mer statistics
- Only a few very short sequences where k-mer frequency profiles are too sparse to
  support stable dissimilarity estimation at the chosen K
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: CAFE
  role: Standalone tool that orchestrates k-mer counting and per-measure transformations
    of k-mer counts/frequencies into the 28 alignment-free dissimilarity measures
  repo: https://github.com/younglululu/CAFE
- name: JELLYFISH
  role: Accelerates the k-mer counting step; invoked from CAFE via the -J option with
    the path to the jellyfish executable
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkx351
    title: 'CAFE: aCcelerated Alignment-FrEe sequence analysis'
  source_content_hash: sha256:1d4e94def78feaaad20941cf880cddf759844a7221734c66456f41c747d3e4b0
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# kmer-frequency-counting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Counting k-mer occurrences in genomic sequences or NGS shotgun reads with CAFE's JELLYFISH-accelerated step, optionally including reverse complements and occurrence thresholds, to produce k-mer count/frequency profiles that feed CAFE's 28 alignment-free dissimilarity measures.

## When to use

Use when input is FASTA genomic sequences (.fasta/.fa/.fna) or unassembled NGS shotgun reads, and alignment-free pairwise distances (e.g. Ma, D2star, CVtree, Jaccard) must be computed from k-mer count profiles.

## When NOT to use

- Sequences are not in FASTA format (.fasta/.fa/.fna) — e.g. raw FASTQ or other containers not accepted by the CAFE data selector
- Full alignment-based comparison is required (e.g. gene regulatory regions where homologous positions must be resolved) rather than alignment-free k-mer statistics
- Only a few very short sequences where k-mer frequency profiles are too sparse to support stable dissimilarity estimation at the chosen K

## Inputs

- Genomic sequence files with extension .fasta, .fa or .fna (assembled genomes or NGS shotgun reads)
- Directory of FASTA files (via -F) or comma-separated FASTA file list (via -I)
- Path to JELLYFISH executable (e.g. jellyfish-2.2.4) for accelerated k-mer counting
- K-mer length K (integer; default 8 in GUI example)
- Optional lower occurrence threshold (-L, default 0), reverse-complement flag (-R), and Markov order (-M; -1 triggers BIC-based automatic inference)

## Outputs

- Per-sequence k-mer count profiles (optionally cached as binary model files via -S)
- Measure-specific transformed data: binary presence/absence indicators, normalized k-mer frequencies, or background-adjusted counts
- Pairwise dissimilarity matrix for selected measures (e.g. result.Ma.phylip in standard PHYLIP format when -T phylip)

## How to apply

Load FASTA sequences into CAFE (via -I file list or -F directory) and count k-mers for every sequence using the JELLYFISH-accelerated step, pointing -J at the jellyfish executable (e.g. jellyfish-2.2.4). Set the k-mer length with -K (default K=8 in the GUI example); optionally pass -R to count reverse complements, a common practice for shotgun reads, and -L to restrict to k-mers with occurrence >= lower (default 0). Save reusable count models with -S so each input FASTA corresponds to a particular binary model file. The resulting k-mer counts are then transformed per selected measure — binarized for presence/absence measures, normalized to frequencies for conventional count measures, or compared against Markov-order expected background counts for D2star, D2shepp and CVtree — before pairwise dissimilarity computation. The choice of measure is passed via the comma-separated -D option (e.g. -D D2star,Ma).

## Related tools

- **CAFE** (Standalone tool that orchestrates k-mer counting and per-measure transformations of k-mer counts/frequencies into the 28 alignment-free dissimilarity measures) — https://github.com/younglululu/CAFE
- **JELLYFISH** (Accelerates the k-mer counting step; invoked from CAFE via the -J option with the path to the jellyfish executable)

## Examples

```
./cafe -M 0 -O output_path -S model_dir -T plain -I speciesA.fa,speciesB.fa -J /panfs/cmb-panasas2/ylu465/jellyfish-2.2.4/bin/./jellyfish -K 10 -D D2star,Ma
```

## Evaluation signals

- The output file parses as a standard PHYLIP distance matrix with n sequence labels and a symmetric off-diagonal structure matching the number of inputs
- Running the GUI example with Manhattan distance produces a file named result.Ma.phylip in standard phylip format
- Re-running with the same -K, -L, -R and -M settings (reusing -S cached models) reproduces identical distances
- With -M -1, the inferred Markov order is a non-negative integer less than K (or a per-file list matching the number of FASTA inputs)
- Counts cached in the -S model directory correspond one-to-one with each input FASTA file

## Limitations

- Markov order auto-inference via BIC with -M -1 is 'not suitable for JS' (Jensen-Shannon divergence)
- The -M list form requires the number of comma-separated orders to match the number of FASTA files, and orders must be non-negative integers less than K
- On Windows, the default cafe_win.exe may fail due to User Account Control permission settings, requiring renaming to cafe_win_2.exe/cafe_win_3.exe
- No software version number or license is stated for CAFE in the article/README

## Evidence

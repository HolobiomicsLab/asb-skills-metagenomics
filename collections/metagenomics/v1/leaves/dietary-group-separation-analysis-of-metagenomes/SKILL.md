---
name: dietary-group-separation-analysis-of-metagenomes
description: Use when you have unassembled NGS shotgun-read FASTA metagenomes (extensions
  '.fasta', '.fa' or '.fna') with host diet or phenotype metadata and want to test
  whether alignment-free d2S distances recapitulate those groups.
when_to_use_negative:
- Input is already a k-mer count matrix, OTU/ASV table, or precomputed dissimilarity
  matrix — the d2S computation step is redundant; proceed directly to clustering.
- The goal is alignment-based reconstruction of host phylogeny rather than testing
  phenotype (diet) concordance; the paper's premise is that alignment-based methods
  are not appropriate for NGS short reads, so mixing goals confuses the acceptance
  criterion.
- Sequences require substitution-model-aware evolutionary distances with assembled
  references rather than background-adjusted k-mer count statistics on raw reads.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0621
tools:
- name: CAFE
  role: Stand-alone executable computing the d2S (D2shepp) background-adjusted k-mer
    dissimilarity matrix with -D D2shepp -K 5 -M 0 and exporting it in PHYLIP format
    via -T phylip
  repo: https://github.com/younglululu/CAFE
- name: JELLYFISH
  role: Accelerates the k-mer counting step of CAFE on shotgun reads via the -J option
    (jellyfish 2.2.4 executable path)
- name: Phylip
  role: Hierarchical clustering of the exported PHYLIP distance matrix (UPGMA) to
    build the dietary-group dendrogram
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkx351
    title: 'CAFE: aCcelerated Alignment-FrEe sequence analysis'
  source_content_hash: sha256:a99d998b59b67ca2e4c0885efc545f44443eafe7c2bdcdcce5331c177546ce08
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# dietary-group-separation-analysis-of-metagenomes

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce separation of mammalian dietary groups (hindgut fermenters, foregut fermenters, carnivores) from 28 NGS gut metagenomic shotgun-read samples by computing alignment-free d2S dissimilarities (k = 5, Markov order 0) with CAFE/JELLYFISH and clustering them into a UPGMA dendrogram.

## When to use

Use when you have unassembled NGS shotgun-read FASTA metagenomes (extensions '.fasta', '.fa' or '.fna') with host diet or phenotype metadata and want to test whether alignment-free d2S distances recapitulate those groups.

## When NOT to use

- Input is already a k-mer count matrix, OTU/ASV table, or precomputed dissimilarity matrix — the d2S computation step is redundant; proceed directly to clustering.
- The goal is alignment-based reconstruction of host phylogeny rather than testing phenotype (diet) concordance; the paper's premise is that alignment-based methods are not appropriate for NGS short reads, so mixing goals confuses the acceptance criterion.
- Sequences require substitution-model-aware evolutionary distances with assembled references rather than background-adjusted k-mer count statistics on raw reads.

## Inputs

- 28 mammalian gut metagenomic NGS shotgun-read files in FASTA format ('.fasta', '.fa' or '.fna')
- JELLYFISH executable path (e.g. jellyfish-2.2.4/bin/./jellyfish)
- Sample-to-dietary-group metadata (8 hindgut fermenters, 13 foregut fermenters, 7 carnivores) for coloring the dendrogram

## Outputs

- Pairwise d2S dissimilarity matrix in PHYLIP format (result.D2shepp.phylip)
- Hierarchically clustered dendrogram of the 28 samples, colored by dietary group (e.g. dendrogram.png)

## How to apply

Load the 28 mammalian gut metagenome FASTA files into CAFE and run the stand-alone executable with the background-adjusted d2S measure: -D D2shepp, -K 5 (k-mer length), -M 0 (fix Markov order 0 for the background-adjusted k-mer counts), optionally -R to count reverse-complement k-mers as is common for NGS reads, and -J pointing at the jellyfish 2.2.4 executable to accelerate k-mer counting. Export the pairwise dissimilarity matrix in standard PHYLIP format with -T phylip (e.g. result.D2shepp.phylip). Build a hierarchical clustering of the 28 samples using UPGMA on the d2S distances and render the dendrogram with samples colored by dietary group (8 hindgut fermenters, 13 foregut fermenters, 7 simple-gut carnivores). Note the README documents neighbor-joining for CAFE's built-in dendrogram, so use an external clustering routine (e.g. Phylip) if strict UPGMA is required.

## Related tools

- **CAFE** (Stand-alone executable computing the d2S (D2shepp) background-adjusted k-mer dissimilarity matrix with -D D2shepp -K 5 -M 0 and exporting it in PHYLIP format via -T phylip) — https://github.com/younglululu/CAFE
- **JELLYFISH** (Accelerates the k-mer counting step of CAFE on shotgun reads via the -J option (jellyfish 2.2.4 executable path))
- **Phylip** (Hierarchical clustering of the exported PHYLIP distance matrix (UPGMA) to build the dietary-group dendrogram)

## Examples

```
./cafe -D D2shepp -K 5 -M 0 -R -T phylip -F data_dir -S model_dir -O result.D2shepp.phylip -J jellyfish-2.2.4/bin/./jellyfish
```

## Evaluation signals

- Output file result.D2shepp.phylip exists, is a symmetric 28x28 matrix, and parses as standard PHYLIP format.
- The UPGMA dendrogram reproduces the qualitative acceptance criterion: the three dietary groups form distinct, non-interleaved clusters rather than grouping by host phylogeny.
- Cluster sizes match the metadata counts: 8 hindgut-fermenting herbivores, 13 foregut-fermenting herbivores, 7 simple-gut carnivores.
- Configuration check: the run used -D D2shepp, -K 5, and -M 0 (Markov order fixed at 0), not the default -1 BIC-based automatic order inference.
- If -R was used, reverse-complement k-mer counting was in effect, which the README recommends as a common practice for NGS shotgun reads.

## Limitations

- The default Markov order is -1 (optimal order inferred by BIC), which the README describes as time-consuming; fixing -M 0 as done here trades model fit for speed and reproducibility.
- CAFE's built-in dendrogram uses the neighbour-joining algorithm per the README, not UPGMA, so the required UPGMA clustering must be done externally (e.g. with Phylip).
- No software version number or license is stated for CAFE in the article or README, complicating exact reproducibility; on Windows the default cafe_win.exe may fail due to permission settings and must be replaced by cafe_win_2.exe/cafe_win_3.exe.

## Evidence

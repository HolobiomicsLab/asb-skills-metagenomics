---
name: markov-order-background-adjusted-kmer-statistics
description: Use when computing background-adjusted k-mer dissimilarities (D2star,
  D2shepp, CVtree) in CAFE and the Markov background order must be chosen or auto-inferred
  per sequence set.
when_to_use_negative:
- Input analysis uses only conventional k-mer count measures (Ma, Eu, Ch, JS, etc.)
  that do not require a background Markov model
- 'Using the JS divergence measure with -M -1: BIC-based automatic order inference
  is explicitly not suitable for JS'
- Sequences are being compared via alignment-based methods where full alignments are
  appropriate and available
edam_operation: http://edamontology.org/operation_0289
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: CAFE
  role: Standalone command-line/GUI tool implementing 28 alignment-free dissimilarity
    measures including the background-adjusted CVtree, D2star and D2shepp, with the
    -M flag controlling the Markov order
  repo: https://github.com/younglululu/CAFE
- name: JELLYFISH
  role: Optional accelerator for the k-mer counting step invoked by CAFE via -J <jfexe_path>
    (e.g. jellyfish 2.2.4)
- name: Phylip
  role: Downstream hierarchical clustering of the CAFE PHYLIP-format distance matrix
    (e.g. UPGMA on the 27 ECOR genomes)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkx351
    title: 'CAFE: aCcelerated Alignment-FrEe sequence analysis'
  source_content_hash: sha256:8b08bb88c31ba7b5b6bac11a432d5454f8d4f3460785ddffc781344f7f26262d
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
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# markov-order-background-adjusted-kmer-statistics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configure the Markov order (-M) that defines the background sequence model for CAFE's background-adjusted alignment-free dissimilarity measures (CVtree, D2star, D2shepp), either with a fixed order or automatic inference via BIC. This order corrects raw k-mer counts for expected occurrences under a Markov model, which is essential for these statistics.

## When to use

Use when computing background-adjusted k-mer dissimilarities (D2star, D2shepp, CVtree) in CAFE and the Markov background order must be chosen or auto-inferred per sequence set.

## When NOT to use

- Input analysis uses only conventional k-mer count measures (Ma, Eu, Ch, JS, etc.) that do not require a background Markov model
- Using the JS divergence measure with -M -1: BIC-based automatic order inference is explicitly not suitable for JS
- Sequences are being compared via alignment-based methods where full alignments are appropriate and available

## Inputs

- Genome FASTA files with extension '.fasta', '.fa', or '.fna' (assembled genomes or NGS shotgun reads)
- K-mer length choice (e.g. -K 14 for the ECOR E. coli/Shigella workflow)
- Chosen Markov order value(s), or -1 for automatic BIC-based inference

## Outputs

- Pairwise dissimilarity matrix in standard PHYLIP format (e.g. 'result.phylip') when -T phylip is used
- Plain-text or other format distance matrix depending on -T (plain, phylip, cytoscape, mds)
- Optionally saved k-mer count/model binary files in the -S directory for reuse

## How to apply

Run CAFE on FASTA inputs ('.fasta', '.fa', '.fna') with -D set to one of the background-adjusted measures (D2shepp, D2star, CVtree) and pass -M <order>. The Markov order encodes the sequence model used to compute expected k-mer counts; it must be a non-negative integer strictly less than the k-mer length set via -K. Choose a single order to apply to all sequences, or a comma-separated list matching the number of input FASTA files for per-sequence orders. When the appropriate order is unclear, set -M -1 so CAFE infers the optimal order automatically using the Bayesian Information Criterion (BIC); note this option is not suitable for JS and can be time-consuming. For the ECOR 27 E. coli/Shigella workflow, D2shepp with K=14 and M=1 recovers ECOR groups via UPGMA, providing a reference parameterization.

## Related tools

- **CAFE** (Standalone command-line/GUI tool implementing 28 alignment-free dissimilarity measures including the background-adjusted CVtree, D2star and D2shepp, with the -M flag controlling the Markov order) — https://github.com/younglululu/CAFE
- **JELLYFISH** (Optional accelerator for the k-mer counting step invoked by CAFE via -J <jfexe_path> (e.g. jellyfish 2.2.4))
- **Phylip** (Downstream hierarchical clustering of the CAFE PHYLIP-format distance matrix (e.g. UPGMA on the 27 ECOR genomes))

## Examples

```
./cafe -M 1 -O out.d2S.phylip -S model_dir -T phylip -I ecoli27/*.fasta -J jellyfish-2.2.4/bin/./jellyfish -K 14 -D D2shepp
```

## Evaluation signals

- The distance matrix output is written in valid standard PHYLIP lower-triangular format with one row per input FASTA file
- Fixed -M values are non-negative integers strictly less than the k-mer length K; per-sequence order lists match the number of input FASTA files exactly
- With -M -1, the log shows CAFE inferring the optimal Markov order via BIC and completes without errors (verify executable permissions on cafe_win.exe first, as documented for Windows)
- For the ECOR benchmark, d2S with K=14 and M=1 yields a UPGMA tree in which every ECOR group (A, B1, B2, D, E, S) forms a monophyletic cluster except groups A and B2
- Reusing cached k-mer count binaries via -S reproduces identical distances across runs

## Limitations

- Markov order must be less than the k-mer length; invalid combinations will not run
- Automatic order inference with -M -1 uses BIC and is described as 'simple yet time-consuming', and is not suitable for the JS measure
- On Windows, the default cafe_win.exe may fail due to User Account Control permission settings and may need to be replaced with cafe_win_2.exe or cafe_win_3.exe
- The original implementations of d2* and d2S were relatively slow due to the requirement of calculating expected k-mer counts; CAFE mitigates but memory/compute demands remain for large genomes
- No software version number or license is stated for CAFE in the article or README

## Evidence

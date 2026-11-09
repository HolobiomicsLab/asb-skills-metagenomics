---
name: genomic-sequence-preprocessing
description: Use when computing background-adjusted alignment-free measures (CVtree,
  D2star, D2shepp) on genomic FASTA files or NGS shotgun reads and the Markov model
  order encoding the sequence is unknown to the user.
when_to_use_negative:
- When using the JS (Jensen-Shannon) measure — the README states automatic order inference
  is 'not suitable for JS'
- When the Markov order is already known or dictated by prior analysis — supply a
  single value or comma-separated list directly instead of -1
- When the input is not sequence data (e.g. a distance matrix or feature table) —
  this skill operates on FASTA sequences prior to dissimilarity computation
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_3174
tools:
- name: CAFE
  role: Standalone executable that evaluates candidate Markov orders via BIC when
    -M is set to -1, and computes the k-mer counts feeding CVtree, D2star, and D2shepp
  repo: https://github.com/younglululu/CAFE
- name: JELLYFISH
  role: Accelerates k-mer counting during preprocessing, invoked through CAFE's -J
    option with the executable path (e.g. jellyfish-2.2.4)
  repo: https://github.com/younglululu/CAFE
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkx351
    title: 'CAFE: aCcelerated Alignment-FrEe sequence analysis'
  source_content_hash: sha256:401d5660ebfa2fc535258bf77f972099e00e870807c17b545e954351bfd79f1a
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# genomic-sequence-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocess genomic FASTA sequences for alignment-free analysis with CAFE by inferring the appropriate Markov model order via the Bayesian Information Criterion (BIC) — required for background-adjusted measures (CVtree, D2star, D2shepp) when the proper order is unclear — and optionally accelerating k-mer counting with JELLYFISH.

## When to use

Use when computing background-adjusted alignment-free measures (CVtree, D2star, D2shepp) on genomic FASTA files or NGS shotgun reads and the Markov model order encoding the sequence is unknown to the user.

## When NOT to use

- When using the JS (Jensen-Shannon) measure — the README states automatic order inference is 'not suitable for JS'
- When the Markov order is already known or dictated by prior analysis — supply a single value or comma-separated list directly instead of -1
- When the input is not sequence data (e.g. a distance matrix or feature table) — this skill operates on FASTA sequences prior to dissimilarity computation

## Inputs

- Genomic FASTA sequence files (.fasta, .fa, .fna) for assembled genomes (e.g. primate genomes, E. coli/Shigella genomes)
- Unassembled NGS shotgun sequence reads (e.g. gut metagenome reads)
- Path to the JELLYFISH executable (e.g. jellyfish-2.2.4/bin/./jellyfish) for accelerated k-mer counting

## Outputs

- Inferred optimal Markov order per sequence/dataset (e.g. markov_orders.csv with columns dataset and inferred_order)
- JELLYFISH-accelerated k-mer count binary models (via -S model_dir) reusable across runs

## How to apply

Load the input sequences (assembled genomes or unassembled NGS shotgun reads, files with extension '.fasta', '.fa' or '.fna') into CAFE. For each sequence, set the Markov Order option (-M) to '-1', which triggers automatic inference of the suitable order by evaluating the BIC over candidate orders, keeping candidates below the k-mer length K; alternatively provide a comma-separated order list matching the number of fasta files. Note that automatic BIC inference is not suitable for the JS (Jensen-Shannon) measure. Optionally pass the JELLYFISH executable path via -J (e.g. jellyfish-2.2.4) to accelerate k-mer counting, and set the lower occurrence cutoff with -L (default 0) and reverse-complement handling with -R for shotgun reads. Record the inferred optimal order per dataset (e.g. in markov_orders.csv) so downstream runs of CVtree/D2star/D2shepp use a reproducible order.

## Related tools

- **CAFE** (Standalone executable that evaluates candidate Markov orders via BIC when -M is set to -1, and computes the k-mer counts feeding CVtree, D2star, and D2shepp) — https://github.com/younglululu/CAFE
- **JELLYFISH** (Accelerates k-mer counting during preprocessing, invoked through CAFE's -J option with the executable path (e.g. jellyfish-2.2.4)) — https://github.com/younglululu/CAFE

## Examples

```
./cafe -M -1 -I speciesA.fa,speciesB.fa -K 10 -D D2star,CVtree -J jellyfish-2.2.4/bin/./jellyfish -S model_dir -T plain
```

## Evaluation signals

- The inferred_order value recorded for each dataset is a non-negative integer strictly less than the k-mer length K
- Validation on reference datasets: inferred order equals 12 for primate genomes, 1 for E. coli/Shigella genomes, and 0 for gut metagenome reads, checked by direct field comparison
- The chosen BIC-minimizing order is consistent across repeated runs on the same input sequences
- Downstream runs of CVtree/D2star/D2shepp reuse the recorded order (via -M or -S cached models) and produce a pairwise distance matrix in PHYLIP format

## Limitations

- Automatic BIC-based order inference is not suitable for the JS measure (only D2star, D2shepp, CVtree use the Markov order)
- The order candidate must remain below the k-mer length K, so very small K values constrain inference
- On Windows, the default cafe_win.exe may not work due to User Account Control permission settings and may need renaming to cafe_win_2.exe/cafe_win_3.exe or running as administrator
- BIC-based order inference is described as the time-consuming option compared to specifying a fixed order

## Evidence

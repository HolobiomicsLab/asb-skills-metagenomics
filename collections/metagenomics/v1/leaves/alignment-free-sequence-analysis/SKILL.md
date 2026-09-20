---
name: alignment-free-sequence-analysis
description: Use when comparing assembled genomic sequences or unassembled NGS shotgun
  reads without alignments, and the Markov order required by background-adjusted k-mer
  measures (CVTree, D2star, D2shepp) is unclear to the user.
when_to_use_negative:
- When an alignment-based approach is appropriate — CAFE targets cases where alignment-based
  methods are not appropriate, such as gene regulatory regions, NGS short reads, and
  virus-host associations.
- When the user already knows the Markov order — a single explicit order (or a per-sequence
  comma-separated list matching the number of FASTA files) can be given with -M, avoiding
  the time cost of BIC inference.
- For JS (Jensen-Shannon divergence) — the -M -1 automatic BIC order inference is
  explicitly documented as 'not suitable for JS'.
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3168
tools:
- name: CAFE
  role: Computes the 28 alignment-free dissimilarity measures (conventional k-mer
    count, background-adjusted, and presence/absence) and, when -M is set to -1, infers
    the optimal Markov order for D2star/D2shepp/CVtree via the BIC
  repo: https://github.com/younglululu/CAFE
- name: JELLYFISH
  role: Optional external k-mer counter used with CAFE's -J option to accelerate k-mer
    counting on large inputs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkx351
    title: 'CAFE: aCcelerated Alignment-FrEe sequence analysis'
  source_content_hash: sha256:3ff76adbcd46fe50fd78e07d5f414ed57c02de35757a5b27714ac87ca0562a5e
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
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# alignment-free-sequence-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Method for computing alignment-free dissimilarity measures between genomes or metagenomic reads with CAFE, including automatic selection of the Markov model order required by background-adjusted measures (CVTree, d2*, D2shepp) via the Bayesian Information Criterion (BIC).

## When to use

Use when comparing assembled genomic sequences or unassembled NGS shotgun reads without alignments, and the Markov order required by background-adjusted k-mer measures (CVTree, D2star, D2shepp) is unclear to the user.

## When NOT to use

- When an alignment-based approach is appropriate — CAFE targets cases where alignment-based methods are not appropriate, such as gene regulatory regions, NGS short reads, and virus-host associations.
- When the user already knows the Markov order — a single explicit order (or a per-sequence comma-separated list matching the number of FASTA files) can be given with -M, avoiding the time cost of BIC inference.
- For JS (Jensen-Shannon divergence) — the -M -1 automatic BIC order inference is explicitly documented as 'not suitable for JS'.

## Inputs

- FASTA sequence files (.fasta, .fa, .fna) — assembled genomes or unassembled NGS shotgun reads
- Comma-separated list of sequence files (-I) or a folder of FASTA files (-F)
- K-mer length K and optionally a Jellyfish executable path for accelerated counting

## Outputs

- Pairwise dissimilarity matrix in PHYLIP format (e.g. result.Ma.phylip)
- Inferred optimal Markov order per dataset (e.g. markov_orders.csv with columns dataset, inferred_order)
- Optional visual outputs: dendrogram, heatmap, PCoA 2D projection, network display

## How to apply

Load FASTA inputs (.fasta, .fa, .fna — genomic sequences or shotgun reads) into CAFE via -I or -F. For background-adjusted measures (CVTree, D2star, D2shepp), set the Markov order to -1 with the -M option so CAFE evaluates candidate orders (each kept below the k-mer length K) and picks the one minimizing the Bayesian Information Criterion, instead of guessing the order manually. Optionally pass the -J flag pointing at the Jellyfish 2.2.4 executable to accelerate k-mer counting. Run e.g. ./cafe -D D2star,Ma,CVtree -K <intK> -M -1 to compute the pairwise dissimilarity matrix; output goes to a phylip-format distance file when -T phylip is chosen (or plain/mds/cytoscape). Record the inferred optimal order per dataset (e.g. in markov_orders.csv with columns dataset and inferred_order) and then feed the distance matrix to downstream dendrogram, heatmap, PCoA, or network analysis. Rationale: the original d2*/d2S implementations are slow because they compute expected k-mer counts, and CAFE speeds this up while removing the need for the user to hand-pick the Markov order.

## Related tools

- **CAFE** (Computes the 28 alignment-free dissimilarity measures (conventional k-mer count, background-adjusted, and presence/absence) and, when -M is set to -1, infers the optimal Markov order for D2star/D2shepp/CVtree via the BIC) — https://github.com/younglululu/CAFE
- **JELLYFISH** (Optional external k-mer counter used with CAFE's -J option to accelerate k-mer counting on large inputs)

## Examples

```
./cafe -M -1 -O output_path -T phylip -I speciesA.fa,speciesB.fa -J jellyfish-2.2.4/bin/./jellyfish -K 10 -D D2star,Ma,CVtree
```

## Evaluation signals

- Inferred Markov order is a non-negative integer strictly less than the k-mer length K (per -M constraints: order 'could be non-negative integer but less than Kmer length').
- Direct field checks against reported values: inferred_order = 12 for primate genomes, 1 for E. coli/Shigella genomes, 0 for gut metagenome reads.
- Output file parses as valid PHYLIP lower-triangular distance matrix with an entry for every input sequence pair.
- markov_orders.csv contains exactly the columns 'dataset' and 'inferred_order' with one row per input dataset.
- Repeated runs with -M -1 on the same input reproduce the same inferred order (BIC is deterministic given the data).

## Limitations

- Automatic BIC order inference is time-consuming relative to specifying a fixed order, and is not suitable for the JS measure.
- On Windows, the default cafe_win.exe may not work due to User Account Control permission settings; the README instructs renaming cafe_win_2.exe/cafe_win_3.exe and possibly running CAFEGUI as administrator.
- CAFE provides no stated software version number or license information in the article context.

## Evidence

- [readme] BIC inference of Markov order: "Usually the potential Markov Order remains unclear to the user. The simple yet time-consuming way is to choose '-1' as inferring the optimal Markov Order automatically by using the Bayesian"

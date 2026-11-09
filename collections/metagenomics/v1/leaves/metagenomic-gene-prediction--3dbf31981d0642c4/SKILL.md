---
name: metagenomic-gene-prediction
description: Use when input is unassembled metagenomic DNA reads (e.g. Sanger, 454,
  or Illumina shotgun reads) and the goal is nucleotide-level gene prediction with
  sensitivity/precision scoring against known gene coordinates.
when_to_use_negative:
- Input is a single complete prokaryotic genome — vanilla Glimmer routinely identifies
  >99% of genes there and Glimmer-MG's extra classification/clustering machinery is
  unnecessary and more computationally expensive
- Reads are already assembled into long contigs or a finished genome where metagenomic
  fragmentation handling (gene-length model, truncated-gene modeling) adds no benefit
- The task requires exact performance parity with prior methods under tight compute
  budgets — the paper notes Glimmer-MG is more computationally expensive than existing
  software
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Glimmer-MG
  role: 'Primary gene predictor: IMM-based coding-region identification with classification-,
    clustering-, and error-model-augmented retraining'
  repo: github.com/davek44/Glimmer-MG
- name: Glimmer
  role: Underlying IMM gene-prediction framework used to seed initial predictions
    before per-cluster retraining
- name: Phymm
  role: Phylogenetic classifier used to parameterize gene prediction models per sequence
    (replacing GC-content-based model selection)
- name: Scimm
  role: Clustering method (PhyScimm with Phymm augmentation) grouping sequences likely
    from the same organism to enable per-cluster retraining
- name: FragGeneScan
  role: Comparator gene finder for benchmarking sensitivity/precision on the same
    read sets
- name: MetaGeneMark
  role: Comparator gene finder for benchmarking sensitivity/precision on the same
    read sets
- name: MetaGeneAnnotator
  role: Comparator gene finder on error-free simulated reads
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_002
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkr1067
    title: Gene prediction with Glimmer for metagenomic sequences augmented by classification
      and clustering
  source_content_hash: sha256:8371b677505d61953751b608d70271dec2e325aa9902a71f1d90ec83001f3e2e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/davek44/Glimmer-MG
  tools:
  - 454 Life Sciences
  - FragGeneScan
  - Glimmer
  - Glimmer-MG
  - MetaGeneAnnotator
  - MetaGeneMark
  - Phymm
  - Scimm
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# metagenomic-gene-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predict protein-coding genes in environmental shotgun (metagenomic) sequences using Glimmer-MG, which augments the Glimmer interpolated Markov model (IMM) framework with Phymm phylogenetic classification, Scimm clustering, per-cluster retraining, a gene-length model for truncated genes, and explicit modeling of indel and substitution sequencing errors.

## When to use

Use when input is unassembled metagenomic DNA reads (e.g. Sanger, 454, or Illumina shotgun reads) and the goal is nucleotide-level gene prediction with sensitivity/precision scoring against known gene coordinates.

## When NOT to use

- Input is a single complete prokaryotic genome — vanilla Glimmer routinely identifies >99% of genes there and Glimmer-MG's extra classification/clustering machinery is unnecessary and more computationally expensive
- Reads are already assembled into long contigs or a finished genome where metagenomic fragmentation handling (gene-length model, truncated-gene modeling) adds no benefit
- The task requires exact performance parity with prior methods under tight compute budgets — the paper notes Glimmer-MG is more computationally expensive than existing software

## Inputs

- Unassembled metagenomic shotgun DNA sequences (e.g. simulated or real 454/Sanger/Illumina reads)
- Reference GenBank genomes with known gene coordinates for simulation and ground-truth scoring
- Glimmer-MG installation with manual.pdf and shipped training models

## Outputs

- gene_predictions.gff — predicted coding regions in valid GFF format for all input sequences
- Per-tool nucleotide-level 5'/3' sensitivity and precision metrics (e.g. table1_sensitivity_precision.csv, table2_sensitivity_precision.csv)
- Translated predicted coding regions for downstream BLAST validation against NCBI nr

## How to apply

First classify input sequences with Phymm to select appropriate training genomes for IMM parameterization, instead of relying on GC-content. Then cluster sequences by likely organism of origin using Scimm (PhyScimm, combining sequence similarity with Phymm classifications), and retrain the IMM, RBS, and start-codon models on initial Glimmer gene predictions for each cluster containing at least 80 Kb of predicted coding sequence; smaller clusters fall back to likelihood-ratio model selection. Run the final prediction pass incorporating a probabilistic gene-length and start/stop-codon model for truncated genes on short fragments, and a model that branches into a different reading frame at predicted insertions/deletions (e.g. homopolymer-run indels in 454 reads) and stop-codon-introducing substitutions. Score outputs against ground-truth gene coordinates computing nucleotide-level 5'/3' sensitivity and precision per read type and error rate, as in the paper's Tables 1 and 2. For real reads (e.g. human gut 454 samples TS28/TS50), validate by translating predicted coding regions and BLAST-searching against NCBI nr with E < 0.001 to count protein-matching predictions.

## Related tools

- **Glimmer-MG** (Primary gene predictor: IMM-based coding-region identification with classification-, clustering-, and error-model-augmented retraining) — github.com/davek44/Glimmer-MG
- **Glimmer** (Underlying IMM gene-prediction framework used to seed initial predictions before per-cluster retraining)
- **Phymm** (Phylogenetic classifier used to parameterize gene prediction models per sequence (replacing GC-content-based model selection))
- **Scimm** (Clustering method (PhyScimm with Phymm augmentation) grouping sequences likely from the same organism to enable per-cluster retraining)
- **FragGeneScan** (Comparator gene finder for benchmarking sensitivity/precision on the same read sets)
- **MetaGeneMark** (Comparator gene finder for benchmarking sensitivity/precision on the same read sets)
- **MetaGeneAnnotator** (Comparator gene finder on error-free simulated reads)

## Examples

```
See manual.pdf for instructions on installation and running Glimmer-MG.
```

## Evaluation signals

- gene_predictions.gff exists, parses as valid GFF, and contains predicted features for all input sequences
- All computed sensitivity and precision values fall in [0, 100] for every predictor, read length, and error-rate combination
- On error-free simulated reads, Glimmer-MG shows greatest sensitivity for every read length, with 3' sensitivity 1.3–1.8% above the second-best method
- On 454 reads at 1.0% error rate, indel-prediction sensitivity and precision approximate the reported 23.2% and 63.8%; at 2.0% error, overall sensitivity ≈ 73.2% and precision ≈ 54.7%
- On real 454 gut microbiome reads (TS28/TS50), BLAST-hit counts against NCBI nr (E < 0.001) show Glimmer-MG predictions exceeding FragGeneScan by ~4.5% and MetaGeneMark by ~6.5%

## Limitations

- Predicting genes on error-prone sequences, particularly those with indels, remains a difficult problem — at 2.0% 454 error, precision is only 54.7%, indicating many genes are predicted incorrectly
- Glimmer-MG is more computationally expensive than existing software
- Highly diverged organisms where optimal model parameters are unclear present a challenge
- Substantial further gains likely depend on reducing the error rate via improved assembly and base-calling rather than computation
- No software version is stated for Glimmer-MG or the compared tools, complicating exact reproduction

## Evidence

- [readme] Glimmer-MG can predict insertions and deletions in the sequence by branching into a different frame at low quality base calls such as homopolymer runs in 454 sequences: [evidence span withheld — anchor status: external_doc]

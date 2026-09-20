---
name: simulated-metagenome-benchmarking
description: Use when evaluating a metagenomic gene predictor on simulated sequencing
  reads with known reference annotations, especially to quantify accuracy gains from
  an unsupervised retraining step as sequencing error rate increases.
when_to_use_negative:
- The goal is gene prediction on real (non-simulated) metagenomes, where no reference
  annotation exists to compute sensitivity/precision against truth
- Comparing clustering methods themselves (Scimm) rather than quantifying downstream
  gene-prediction retraining gains
- Reads are long contiguous genomic sequences (e.g. complete prokaryotic genomes),
  where Glimmer's standard iterative training regime applies instead of the metagenomic
  simulation benchmark
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3174
tools:
- name: Glimmer-MG
  role: Metagenomic gene prediction system whose initial (post-classification) and
    final (post-retraining) prediction sets are compared on simulated reads
  repo: https://github.com/davek44/Glimmer-MG
- name: Phymm
  role: Phylogenetic classifier used to parameterize the gene prediction models that
    produce the initial predictions being benchmarked
- name: Scimm
  role: Unsupervised clustering method grouping reads by likely organism of origin,
    enabling the retraining step whose gains this benchmark quantifies
- name: Glimmer
  role: Underlying gene prediction framework on which Glimmer-MG is built; provides
    the IMM-based prediction machinery evaluated in the simulation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkr1067
    title: Gene prediction with Glimmer for metagenomic sequences augmented by classification
      and clustering
  source_content_hash: sha256:f3c015ed2d6e32d35bcf233b135a6ac753844c55cdeca00b25f0b2ca19b1ae7c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/davek44/Glimmer-MG
  repo_url: https://github.com/davek44/Glimmer-MG
  tools:
  - Glimmer
  - Glimmer-MG
  - Phymm
  - Scimm
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# simulated-metagenome-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark gene prediction accuracy (sensitivity, precision) of Glimmer-MG's initial (post-classification) versus final (post-clustering retraining) predictions on realistically simulated metagenomes across read technologies, lengths, and error rates, quantifying how much the Scimm-based retraining step improves predictions.

## When to use

Use when evaluating a metagenomic gene predictor on simulated sequencing reads with known reference annotations, especially to quantify accuracy gains from an unsupervised retraining step as sequencing error rate increases.

## When NOT to use

- The goal is gene prediction on real (non-simulated) metagenomes, where no reference annotation exists to compute sensitivity/precision against truth
- Comparing clustering methods themselves (Scimm) rather than quantifying downstream gene-prediction retraining gains
- Reads are long contiguous genomic sequences (e.g. complete prokaryotic genomes), where Glimmer's standard iterative training regime applies instead of the metagenomic simulation benchmark

## Inputs

- Simulated metagenomic read sets (FASTA/FASTQ) with per-condition metadata (read technology, read length, error rate; e.g. 454 reads at 0%, 2.0% error)
- Simulated reference genome annotations for alignment
- Glimmer-MG initial gene predictions (models parameterized after Phymm phylogenetic classification)
- Glimmer-MG final gene predictions (after Scimm clustering and unsupervised retraining)

## Outputs

- Per-condition sensitivity and precision for initial and final predictions (gene-level and nucleotide-level, plus 3'-end accuracy for error-prone reads)
- Delta table (e.g. retraining_gains.csv) with final-minus-initial sensitivity/precision per simulated condition (read technology, read length, error rate)

## How to apply

Simulate metagenomic reads (e.g. 454 reads where low-quality base calls arise from homopolymer runs) at multiple error rates, then run Glimmer-MG twice: once using predictions from models parameterized after Phymm phylogenetic classification, and once after Scimm clusters sequences and prediction models are retrained on initial predictions within each cluster. Align both prediction sets against the simulated reference annotations and compute gene-level/nucleotide-level sensitivity and precision per condition, including 3'-end accuracy for error-prone reads. Compute the per-condition delta (final minus initial) to isolate the retraining gain; the paper reports retraining raised 454-read sensitivity by 0.8% without errors and by 2.8% at a 2.0% error rate. Validate against known operating points, e.g. simulated 454 reads with 2.0% error should yield Glimmer-MG sensitivity of 73.2% and precision of 54.7%.

## Related tools

- **Glimmer-MG** (Metagenomic gene prediction system whose initial (post-classification) and final (post-retraining) prediction sets are compared on simulated reads) — https://github.com/davek44/Glimmer-MG
- **Phymm** (Phylogenetic classifier used to parameterize the gene prediction models that produce the initial predictions being benchmarked)
- **Scimm** (Unsupervised clustering method grouping reads by likely organism of origin, enabling the retraining step whose gains this benchmark quantifies)
- **Glimmer** (Underlying gene prediction framework on which Glimmer-MG is built; provides the IMM-based prediction machinery evaluated in the simulation)

## Evaluation signals

- Deltas must reproduce the direction of the paper-reported gains: positive sensitivity improvement after clustering-based retraining across all tested simulated conditions
- At the reported operating point (454 reads, 2.0% error rate), sensitivity should be ~73.2% and precision ~54.7%
- Retraining gain magnitude should increase with sequencing error rate (paper: +0.8% sensitivity without errors vs +2.8% at 2.0% error on 454 reads)
- Output table has exactly one row per simulated condition keyed by (read technology, read length, error rate) with non-missing delta values
- Sensitivity and precision computed by aligning predictions against the simulated reference annotations, not by self-consistency between prediction sets

## Limitations

- Predicting genes on error-prone sequences, particularly those with indels, remains a difficult problem; many genes on simulated 454 reads with 2.0% error are predicted incorrectly (precision 54.7%)
- Glimmer-MG is more computationally expensive than existing software
- No software version stated for Glimmer-MG or compared tools, complicating exact reproduction
- Substantial further gains depend on reducing the error rate via improvements to assembly and base-calling accuracy rather than computation

## Evidence

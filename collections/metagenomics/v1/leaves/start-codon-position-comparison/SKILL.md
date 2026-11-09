---
name: start-codon-position-comparison
description: Use when you need to quantify how accurately a gene caller predicts start
  codons (not just gene presence) by scoring exact and near-correct start-site calls
  against a curated annotation truth set.
when_to_use_negative:
- Input is metagenomic sequence fragments without a curated whole-genome truth set
  — use Glimmer-MG's simulated/real metagenome evaluation instead
- The goal is only gene-region detection (whole-gene accuracy), not start-codon-position
  accuracy — a simpler whole-gene sensitivity/precision score suffices
- Sequences are highly error-prone reads with indels where the paper notes accuracy
  evaluation of start sites is dominated by frame-tracking errors
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Glimmer
  role: Whole-genome gene prediction pipeline (Glimmer3.0 iterated and Glimmer3.1
    with length/adjacency filtering) whose start-site predictions are scored
- name: Glimmer-MG
  role: Metagenomic gene prediction system whose predictions are evaluated against
    the same reference annotations
  repo: https://github.com/davek44/Glimmer-MG
- name: Phymm
  role: Phylogenetic classifier used to parameterize Glimmer-MG gene-prediction models
    prior to scoring
- name: Scimm
  role: Clustering method enabling unsupervised retraining of Glimmer-MG models before
    final predictions are scored
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkr1067
    title: Gene prediction with Glimmer for metagenomic sequences augmented by classification
      and clustering
  source_content_hash: sha256:eeecc8f2772f29518966f1ea636acf0e3d7ff325ffd508ca5a82f5320675fd90
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
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# start-codon-position-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking procedure that evaluates gene-prediction pipelines (Glimmer3.0, Glimmer3.1, Glimmer-MG) by comparing predicted gene start positions against curated reference annotations, computing 3' (start-site) and 5' sensitivity and precision per genome.

## When to use

Use when you need to quantify how accurately a gene caller predicts start codons (not just gene presence) by scoring exact and near-correct start-site calls against a curated annotation truth set.

## When NOT to use

- Input is metagenomic sequence fragments without a curated whole-genome truth set — use Glimmer-MG's simulated/real metagenome evaluation instead
- The goal is only gene-region detection (whole-gene accuracy), not start-codon-position accuracy — a simpler whole-gene sensitivity/precision score suffices
- Sequences are highly error-prone reads with indels where the paper notes accuracy evaluation of start sites is dominated by frame-tracking errors

## Inputs

- Reference prokaryotic genomes in FASTA format (e.g., 12 complete genomes)
- Curated gene annotations (gene coordinates and start codon positions) serving as the truth set
- Predicted gene coordinate files (contig/gene start/stop positions) from each pipeline: Glimmer3.0 iterated, Glimmer3.1 filtered, Glimmer-MG

## Outputs

- Per-genome 3' (start-site) sensitivity and precision values
- Per-genome 5' sensitivity and precision values
- Averaged sensitivity/precision per pipeline across all genomes
- Summary table (e.g., supplementary_table_s1_accuracy.csv)

## How to apply

Collect complete prokaryotic genomes in FASTA with their curated gene annotations as the truth set (the paper uses 12 reference genomes). Run each pipeline under comparison — the iterated Glimmer3.0 pipeline, the Glimmer3.1 pipeline with length/adjacency filtering that discards predictions entirely contained within longer predictions, and Glimmer-MG with Phymm-based classification and Scimm cluster retraining — on the same genomes. Score each pipeline by counting exact start codon matches and near-correct start calls (nearby alternative start sites) against the reference, yielding 3' (start-site) and 5' sensitivity and precision per genome. The rationale for the Glimmer3.1 comparison is that the length model removes erroneous short predictions (42 predictions <150 bp per genome vs 68 for Glimmer3.0), boosting 3' precision by ~1.3% on average while preserving high 3' sensitivity. Average per-genome values across all genomes per pipeline and export per-genome plus averaged results to a CSV (e.g., supplementary_table_s1_accuracy.csv) for direct comparison.

## Related tools

- **Glimmer** (Whole-genome gene prediction pipeline (Glimmer3.0 iterated and Glimmer3.1 with length/adjacency filtering) whose start-site predictions are scored)
- **Glimmer-MG** (Metagenomic gene prediction system whose predictions are evaluated against the same reference annotations) — https://github.com/davek44/Glimmer-MG
- **Phymm** (Phylogenetic classifier used to parameterize Glimmer-MG gene-prediction models prior to scoring)
- **Scimm** (Clustering method enabling unsupervised retraining of Glimmer-MG models before final predictions are scored)

## Evaluation signals

- Whole-gene sensitivity should routinely exceed 99% on complete prokaryotic genomes for Glimmer-based pipelines
- Glimmer3.1 should reduce short spurious predictions: ~42 predictions <150 bp per genome versus 68 for Glimmer3.0
- 3' precision for Glimmer3.1 should improve ~1.3% on average over Glimmer3.0 while 3' sensitivity stays high
- Per-genome sensitivity/precision rows must be present for all 12 reference genomes plus an averaged row per pipeline
- Exact and near-correct start codon calls should be counted separately so near-start alternatives are not scored as full misses

## Limitations

- Predicting genes on error-prone sequences, particularly those with indels, remains a difficult problem — on simulated 454 reads with 2.0% error, Glimmer-MG precision is only 54.7%
- Highly diverged organisms where the optimal model parameters are unclear present a challenge to accurate start-site prediction
- No software versions are stated for Glimmer-MG or compared tools, so exact reproducibility of accuracy numbers may vary

## Evidence

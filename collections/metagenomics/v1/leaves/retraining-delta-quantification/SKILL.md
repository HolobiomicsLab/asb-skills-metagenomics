---
name: retraining-delta-quantification
description: Use when comparing Glimmer-MG's initial gene predictions (models parameterized
  after Phymm phylogenetic classification) against its final predictions (after Scimm
  clustering and retraining) to measure the accuracy gain from the retraining step.
when_to_use_negative:
- The task is absolute accuracy benchmarking against other gene finders (e.g. FragGeneScan)
  rather than isolating the within-tool retraining gain
- Only real metagenomic reads are available without simulated reference annotations,
  since deltas must be computed against ground truth
- The pipeline used only the initial classification-parameterized predictions with
  no clustering/retraining step, so no delta exists
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3174
tools:
- name: Glimmer-MG
  role: Produces both the initial (post-classification) and final (post-clustering/retraining)
    gene predictions being compared
  repo: https://github.com/davek44/Glimmer-MG
- name: Scimm
  role: Clusters reads by likely organism of origin, enabling the unsupervised retraining
    step whose gain is quantified
- name: Phymm
  role: Phylogenetic classifier used to parameterize the models behind the initial
    (baseline) predictions
- name: Glimmer
  role: Underlying gene-prediction framework on which Glimmer-MG is built
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkr1067
    title: Gene prediction with Glimmer for metagenomic sequences augmented by classification
      and clustering
  source_content_hash: sha256:fb711cc486059337cd94e946d45746caa924f0d1e5a611ddfd63f8cf6b027d6d
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# retraining-delta-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how much Glimmer-MG's Scimm-based unsupervised retraining step improves gene prediction over its initial (post-Phymm-classification) predictions by computing per-condition deltas in sensitivity and precision on simulated metagenomes, including error-prone 454 reads.

## When to use

Use when comparing Glimmer-MG's initial gene predictions (models parameterized after Phymm phylogenetic classification) against its final predictions (after Scimm clustering and retraining) to measure the accuracy gain from the retraining step.

## When NOT to use

- The task is absolute accuracy benchmarking against other gene finders (e.g. FragGeneScan) rather than isolating the within-tool retraining gain
- Only real metagenomic reads are available without simulated reference annotations, since deltas must be computed against ground truth
- The pipeline used only the initial classification-parameterized predictions with no clustering/retraining step, so no delta exists

## Inputs

- Initial Glimmer-MG gene predictions (post-phylogenetic-classification models) per simulated condition
- Final Glimmer-MG gene predictions (post-Scimm clustering and retraining) per simulated condition
- Simulated reference genome annotations for the metagenome conditions (read technology, read length, error rate, e.g. 454 reads at 0% and 2.0% error)

## Outputs

- retraining_gains.csv with one row per simulated condition (read technology, read length, error rate) and per-condition deltas in sensitivity and precision (final minus initial)

## How to apply

Load both prediction sets — the initial predictions made with models parameterized by phylogenetic classification, and the final predictions made after Scimm clustering and unsupervised retraining within each cluster — for each simulated metagenome condition (read technology, read length, error rate). Align both prediction sets against the simulated reference annotations and compute gene-level/nucleotide-level sensitivity and precision, including 3'-end accuracy for error-prone 454 reads where low-quality base calls arise from homopolymer runs. Compute the per-condition delta (final minus initial) for each metric. Rationale: the paper reports retraining yields larger gains as sequencing errors increase (e.g. +0.8% sensitivity without errors vs +2.8% at a 2.0% 454 error rate), so per-condition deltas reveal error-rate-dependent benefits.

## Related tools

- **Glimmer-MG** (Produces both the initial (post-classification) and final (post-clustering/retraining) gene predictions being compared) — https://github.com/davek44/Glimmer-MG
- **Scimm** (Clusters reads by likely organism of origin, enabling the unsupervised retraining step whose gain is quantified)
- **Phymm** (Phylogenetic classifier used to parameterize the models behind the initial (baseline) predictions)
- **Glimmer** (Underlying gene-prediction framework on which Glimmer-MG is built)

## Examples

```
glimmer-mg --sequences simulated_454.fna --initial-predictions initial.gff --final-predictions final.gff; python compute_deltas.py --ref reference.gff --initial initial.gff --final final.gff --out retraining_gains.csv
```

## Evaluation signals

- Computed sensitivity/precision deltas are positive after clustering-based retraining across all tested simulated conditions
- Deltas reproduce the reported error-rate trend: larger sensitivity gain at higher sequencing error (+0.8% without errors vs +2.8% at 2.0% on 454 reads)
- Final predictions at the 2.0% 454-error operating point match the reported 73.2% sensitivity / 54.7% precision
- retraining_gains.csv contains exactly one row per simulated condition, keyed by read technology, read length, and error rate
- Precision deltas should be checked alongside sensitivity deltas, since the paper notes many genes are still predicted incorrectly at high error rates

## Limitations

- Predicting genes on error-prone sequences, particularly those with indels, remains a difficult problem
- At a 2.0% error rate on simulated 454 reads, precision is only 54.7%, indicating many genes are predicted incorrectly even after retraining
- Substantial further gains depend on reducing the error rate via improved assembly and base-calling rather than computation
- Glimmer-MG is more computationally expensive than existing gene-prediction software
- No software version is stated for Glimmer-MG or compared tools, complicating exact reproduction

## Evidence

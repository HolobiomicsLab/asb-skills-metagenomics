---
name: unsupervised-gene-model-retraining-per-cluster
description: Use when predicting genes in fragmented metagenomic reads where no reference
  genomes for training are available, and sequences should first be grouped by likely
  organism of origin so models can be retrained unsupervised per cluster.
when_to_use_negative:
- The input is a single complete prokaryotic genome, where Glimmer's iterative whole-genome
  training works and clustering is unnecessary
- Sequences lack enough contiguous per-organism coverage so every cluster falls below
  the 80 Kb retraining threshold, leaving no cluster eligible for retraining
- The task requires low computational cost — Glimmer-MG is more computationally expensive
  than existing gene prediction software
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: Glimmer-MG
  role: End-to-end system that executes the classify -> cluster -> retrain -> predict
    loop using IMMs, the gene-length/start-stop codon model, and sequencing-error
    modeling
  repo: github.com/davek44/Glimmer-MG
- name: Phymm
  role: Phylogenetic classifier whose classifications parameterize gene prediction
    models and augment Scimm clustering (PhyScimm)
- name: Scimm
  role: Unsupervised clustering method that groups sequences likely from the same
    organism so prediction models can be retrained within each cluster
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkr1067
    title: Gene prediction with Glimmer for metagenomic sequences augmented by classification
      and clustering
  source_content_hash: sha256:6956f6035930c4d13ac446aceafb5abc55e5c9c278aeb59a77cc8430937a0ade
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/davek44/Glimmer-MG
  tools:
  - Glimmer-MG
  - Phymm
  - Scimm
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# unsupervised-gene-model-retraining-per-cluster

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Organizes Glimmer-MG gene prediction by clustering metagenomic sequences with Scimm (PhyScimm) and retraining IMM, RBS, and start-codon models within each cluster on initial Glimmer gene predictions before making final predictions, boosting full-gene accuracy without supervised training data.

## When to use

Use when predicting genes in fragmented metagenomic reads where no reference genomes for training are available, and sequences should first be grouped by likely organism of origin so models can be retrained unsupervised per cluster.

## When NOT to use

- The input is a single complete prokaryotic genome, where Glimmer's iterative whole-genome training works and clustering is unnecessary
- Sequences lack enough contiguous per-organism coverage so every cluster falls below the 80 Kb retraining threshold, leaving no cluster eligible for retraining
- The task requires low computational cost — Glimmer-MG is more computationally expensive than existing gene prediction software

## Inputs

- Metagenomic shotgun DNA sequences (e.g., 454 or Illumina reads from environmental or human gut microbiome samples)
- Phymm phylogenetic classifications of the input sequences
- Initial Glimmer gene predictions on each sequence

## Outputs

- Per-cluster retrained gene prediction models (IMM, RBS, and start-codon models)
- Final gene predictions (gene_predictions.gff) covering all input sequences
- Predicted sequencing errors (insertions, deletions, stop codon-introducing substitutions) tracking coding frames

## How to apply

First classify the input sequences with Phymm to parameterize gene prediction models by phylogenetic origin rather than GC-content. Make initial gene predictions on all sequences with the Glimmer IMM framework to seed retraining. Cluster the sequences by likely organism of origin using Scimm with phylogenetic augmentation (PhyScimm), which combines sequence similarity with Phymm classifications. For each cluster containing at least 80 Kb of predicted coding sequence, retrain the IMM, RBS, and start-codon models on the cluster's initial gene predictions; clusters below this threshold skip retraining and keep their initial predictions, falling back to likelihood-ratio selection over available models. Finally run the Glimmer-MG prediction pass with per-cluster retrained models, including the probabilistic gene-length and start/stop-codon model and explicit modeling of insertion/deletion and substitution errors (e.g., branching into a different frame at homopolymer runs in 454 reads).

## Related tools

- **Glimmer-MG** (End-to-end system that executes the classify -> cluster -> retrain -> predict loop using IMMs, the gene-length/start-stop codon model, and sequencing-error modeling) — github.com/davek44/Glimmer-MG
- **Phymm** (Phylogenetic classifier whose classifications parameterize gene prediction models and augment Scimm clustering (PhyScimm))
- **Scimm** (Unsupervised clustering method that groups sequences likely from the same organism so prediction models can be retrained within each cluster)

## Evaluation signals

- Check that gene_predictions.gff exists, parses as valid GFF, and contains predicted features for every input sequence
- Verify the 80 Kb data-sufficiency rule: clusters with at least 80 Kb of predicted coding sequence have retrained models; clusters below it keep their initial Glimmer gene predictions
- Compare sensitivity and precision against Glimmer/previous systems — expect Glimmer-MG to be most sensitive and precise across read lengths and error rates (e.g., sensitivity 73.2% and precision 54.7% on simulated 454 reads with 2.0% error)
- Confirm predicted indel positions track homopolymer-run errors in 454 reads more accurately than FragGeneScan

## Limitations

- Per-cluster retraining is gated by a data-sufficiency rule: clusters with less than 80 Kb of predicted coding sequence are not retrained and keep initial gene predictions
- Predicting genes on error-prone sequences, particularly those with indels, remains a difficult problem (e.g., precision of 54.7% on simulated 454 reads with 2.0% error)
- Glimmer-MG is more computationally expensive than existing gene prediction software
- Highly diverged organisms where the optimal model parameters are unclear present a challenge
- Substantial further gains are more likely to come from reducing error rates via improved assembly and base-calling than from computation

## Evidence

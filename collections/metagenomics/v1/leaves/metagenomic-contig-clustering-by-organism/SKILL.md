---
name: metagenomic-contig-clustering-by-organism
description: Use when metagenomic shotgun reads/contigs must be grouped by organism
  of origin before downstream per-group model fitting, e.g. to gate retraining of
  gene prediction IMMs within each cluster.
when_to_use_negative:
- When sequences already have reliable per-genome provenance (e.g. assembled complete
  genomes) — clustering and retraining target the metagenomic case where training
  genomes are unknown
- When GC-content-based model parameterization is sufficient for the analysis goal
  — the article shows Phymm-based classification works much better than GC-content,
  so a GC-only grouping is the wrong substitute
- When the input is a single-organism sample where Scimm clustering adds cost without
  benefit; note Glimmer-MG is also more computationally expensive than existing software
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: Scimm
  role: Unsupervised clustering of metagenomic sequences by likely organism of origin;
    its augmented mode combines sequence similarity with Phymm classifications
- name: Phymm
  role: Provides the phylogenetic classifications that augment Scimm clustering and
    parameterize gene prediction models
- name: Glimmer-MG
  role: Consumes the clusters to gate per-cluster retraining and produce final gene
    predictions
  repo: https://github.com/davek44/Glimmer-MG
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkr1067
    title: Gene prediction with Glimmer for metagenomic sequences augmented by classification
      and clustering
  source_content_hash: sha256:ceae23b33d5a7343ce00225735a96931929223c4be6a5fcdb7c3a5b4d31d0e59
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
  - Glimmer-MG
  - Phymm
  - Scimm
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# metagenomic-contig-clustering-by-organism

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Clusters metagenomic sequences by likely organism of origin using Scimm with phylogenetic augmentation from Phymm classifications (PhyScimm), enabling per-cluster retraining of Glimmer-MG gene prediction models on initial gene predictions.

## When to use

Use when metagenomic shotgun reads/contigs must be grouped by organism of origin before downstream per-group model fitting, e.g. to gate retraining of gene prediction IMMs within each cluster.

## When NOT to use

- When sequences already have reliable per-genome provenance (e.g. assembled complete genomes) — clustering and retraining target the metagenomic case where training genomes are unknown
- When GC-content-based model parameterization is sufficient for the analysis goal — the article shows Phymm-based classification works much better than GC-content, so a GC-only grouping is the wrong substitute
- When the input is a single-organism sample where Scimm clustering adds cost without benefit; note Glimmer-MG is also more computationally expensive than existing software

## Inputs

- Metagenomic shotgun sequences (contigs or reads, e.g. 454 or Illumina) in FASTA
- Phymm phylogenetic classifications of those sequences (used for phylogenetic augmentation)
- Initial gene predictions from the Glimmer IMM framework (seeding per-cluster retraining)

## Outputs

- Per-cluster sequence assignments (sequences grouped by likely organism of origin)
- Retrained per-cluster gene prediction models (IMM, RBS, start-codon) for clusters with ≥ 80 Kb
- Final Glimmer-MG predictions (gene_predictions.gff) covering all input sequences

## How to apply

First classify the input sequences with the Phymm phylogenetic classifier, since Scimm's augmented mode (PhyScimm) combines sequence-similarity clustering with Phymm's phylogenetic assignments. Cluster the sequences with Scimm so that sequences likely from the same organism are grouped together. Use the cluster memberships to organize a retraining step: for each cluster containing at least 80 Kb of sequence, retrain the IMM, RBS, and start-codon models on that cluster's initial Glimmer gene predictions; clusters below the 80 Kb threshold are left unretrained and keep their initial predictions (or fall back to likelihood-ratio model selection). Run the final Glimmer-MG prediction pass with the per-cluster retrained models. Treat clustering as an unsupervised prerequisite: the article notes classification and clustering of the sequences are required before prediction.

## Related tools

- **Scimm** (Unsupervised clustering of metagenomic sequences by likely organism of origin; its augmented mode combines sequence similarity with Phymm classifications)
- **Phymm** (Provides the phylogenetic classifications that augment Scimm clustering and parameterize gene prediction models)
- **Glimmer-MG** (Consumes the clusters to gate per-cluster retraining and produce final gene predictions) — https://github.com/davek44/Glimmer-MG

## Evaluation signals

- gene_predictions.gff exists, parses as valid GFF, and contains predicted features for all input sequences
- Clusters with ≥ 80 Kb of predicted coding sequence were retrained; clusters below the threshold retained initial predictions
- Each cluster's retrained models were fit only on that cluster's initial gene predictions
- Every input sequence is assigned to exactly one cluster before the final prediction pass
- Per-cluster retraining yields higher full-gene accuracy than unretrained initial Glimmer predictions

## Limitations

- Glimmer-MG is more computationally expensive than existing software, partly due to the classification and clustering steps
- Highly diverged organisms where optimal model parameters are unclear present a challenge to metagenomic gene prediction
- Heterogeneous clusters below the 80 Kb threshold cannot be reliably retrained and must fall back to initial predictions or likelihood-ratio model selection

## Evidence

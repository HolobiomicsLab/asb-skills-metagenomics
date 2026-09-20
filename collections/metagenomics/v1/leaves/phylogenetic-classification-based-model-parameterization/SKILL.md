---
name: phylogenetic-classification-based-model-parameterization
description: Use when training gene-prediction models on short metagenomic shotgun
  reads (e.g. 454 human gut microbiome reads) where GC-content-based genome selection
  is unreliable and organism-of-origin must be inferred per read.
when_to_use_negative:
- Input is a complete prokaryotic genome where standard Glimmer can train directly
  on the full sequence without phylogenetic classification
- Sequences are long enough that GC-content reliably identifies evolutionarily related
  training genomes (the limitation motivating classification applies to short reads)
- Analysis goal is quantification or assembly of already-predicted genes rather than
  gene prediction model training
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0084
tools:
- name: Glimmer-MG
  role: Metagenomic gene-prediction system whose IMM models are parameterized by the
    phylogenetic classifications
  repo: https://github.com/davek44/Glimmer-MG
- name: Phymm
  role: Phylogenetic classifier that assigns organism-of-origin to each read, driving
    model parameterization
- name: Scimm
  role: Unsupervised clustering method applied after classification to group same-organism
    reads for model retraining
- name: Glimmer
  role: Underlying gene prediction framework providing the IMM-based coding-region
    identification
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkr1067
    title: Gene prediction with Glimmer for metagenomic sequences augmented by classification
      and clustering
  source_content_hash: sha256:05b06730bc93f074b11744cce290fe38fedcda2d9a4a04b88464b65456d18be5
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
  - 454 Life Sciences
  - FragGeneScan
  - Glimmer
  - Glimmer-MG
  - Phymm
  - Scimm
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# phylogenetic-classification-based-model-parameterization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parameterize metagenomic gene-prediction models (Glimmer-MG interpolated Markov models) using Phymm phylogenetic classifications of individual reads rather than GC-content binning, because read-level phylogenetic assignments identify appropriate training genomes for short fragmented metagenomic sequences.

## When to use

Use when training gene-prediction models on short metagenomic shotgun reads (e.g. 454 human gut microbiome reads) where GC-content-based genome selection is unreliable and organism-of-origin must be inferred per read.

## When NOT to use

- Input is a complete prokaryotic genome where standard Glimmer can train directly on the full sequence without phylogenetic classification
- Sequences are long enough that GC-content reliably identifies evolutionarily related training genomes (the limitation motivating classification applies to short reads)
- Analysis goal is quantification or assembly of already-predicted genes rather than gene prediction model training

## Inputs

- Environmental shotgun DNA sequences (metagenomic reads, e.g. 454 human gut microbiome reads from TS28/TS50)
- FASTA-format metagenomic sequence files

## Outputs

- Per-read phylogenetic classifications from Phymm used to parameterize gene prediction models
- IMM-parameterized gene prediction models feeding the Glimmer-MG pipeline
- Predicted coding regions (genes) in the metagenomic reads

## How to apply

Run the Phymm phylogenetic classifier on each metagenomic sequence to assign likely organisms of origin, then use those classifications to select and parameterize the interpolated Markov models (1st through 8th order, 3-periodic nonhomogeneous) that Glimmer-MG uses for coding-region prediction. This replaces the Glimmer3 approach of relying on GC% to find evolutionarily related genomes for training, which the authors show works much worse on short metagenomic reads. The classification step is performed before Scimm clustering and before retraining of prediction models within each cluster on initial gene predictions. Rationale: training model selection is the main reason Glimmer3 cannot be applied to metagenomic sequences, and phylogenetic parameterization significantly improves full gene accuracy on error-prone 454 reads.

## Related tools

- **Glimmer-MG** (Metagenomic gene-prediction system whose IMM models are parameterized by the phylogenetic classifications) — https://github.com/davek44/Glimmer-MG
- **Phymm** (Phylogenetic classifier that assigns organism-of-origin to each read, driving model parameterization)
- **Scimm** (Unsupervised clustering method applied after classification to group same-organism reads for model retraining)
- **Glimmer** (Underlying gene prediction framework providing the IMM-based coding-region identification)

## Evaluation signals

- Gene prediction sensitivity and precision on simulated and real 454 metagenomes should exceed GC-content-parameterized baselines (Glimmer-MG reported 73.2% sensitivity and 54.7% precision on simulated 454 reads with 2.0% error rate)
- Classification-derived parameterization should yield more BLAST-aligning gene predictions than FragGeneScan and MetaGeneMark when predictions are translated and searched against NCBI nr with E < 0.001 (Table 3 validation)
- Every input read should receive a Phymm classification before model parameterization and Scimm clustering proceeds
- Predictions should track coding frames across low-quality bases such as 454 homopolymer-run indels, indicating the parameterized models incorporated error modeling

## Limitations

- Predicting genes on error-prone sequences, particularly those with indels, remains a difficult problem (precision of 54.7% on simulated 454 reads with 2.0% error means many predictions are incorrect)
- Glimmer-MG with Phymm-based parameterization is more computationally expensive than existing gene prediction software
- Highly diverged organisms where the optimal model parameters are unclear present a challenge to metagenomic gene prediction
- No software version stated for Glimmer-MG or compared tools in the paper

## Evidence

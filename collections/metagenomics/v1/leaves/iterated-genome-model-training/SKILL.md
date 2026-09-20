---
name: iterated-genome-model-training
description: Use when predicting protein-coding genes in assembled prokaryotic genomes
  or metagenomic contigs/read sets with Glimmer/Glimmer-MG and you need to boot-strap
  and retrain IMM models on initial predictions before final calls.
when_to_use_negative:
- Sequences are very short error-prone reads with many indels where even Glimmer-MG
  precision is limited (54.7% precision at 2.0% 454 error rate) and assembly/base-calling
  improvements are needed first
- The organism is so highly diverged that optimal model parameters are unclear and
  no close phylogenetic relatives exist for Phymm-based parameterization
- You need a fast, cheap annotation pass — Glimmer-MG is explicitly more computationally
  expensive than existing software
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Glimmer
  role: Whole-genome IMM-based gene predictor whose long-ORF model-building step is
    iterated on initial predictions (Glimmer3.0) with length/adjacency filtering (Glimmer3.1)
- name: Glimmer-MG
  role: Metagenomic extension of the Glimmer framework that classifies with Phymm,
    clusters with Scimm, retrains per cluster, and adds gene-length and sequencing-error
    models
  repo: https://github.com/davek44/Glimmer-MG
- name: Phymm
  role: Provides phylogenetic classifications of sequences used to parameterize the
    gene prediction models instead of GC-content
- name: Scimm
  role: Clusters metagenomic sequences by likely organism of origin so models can
    be retrained unsupervised within each cluster
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkr1067
    title: Gene prediction with Glimmer for metagenomic sequences augmented by classification
      and clustering
  source_content_hash: sha256:b6e48f9b0040740bdf4c5a2f58a953c6f168e9e975b041b43192fb4c42216cbb
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
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# iterated-genome-model-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Iterative self-training of interpolated Markov model (IMM) gene predictors: build an initial model from long ORFs (or Phymm-based phylogenetic classification plus Scimm clustering for metagenomes), retrain on initial gene predictions, then produce final gene calls, optionally with length/adjacency filtering as in the Glimmer3.1 whole-genome pipeline.

## When to use

Use when predicting protein-coding genes in assembled prokaryotic genomes or metagenomic contigs/read sets with Glimmer/Glimmer-MG and you need to boot-strap and retrain IMM models on initial predictions before final calls.

## When NOT to use

- Sequences are very short error-prone reads with many indels where even Glimmer-MG precision is limited (54.7% precision at 2.0% 454 error rate) and assembly/base-calling improvements are needed first
- The organism is so highly diverged that optimal model parameters are unclear and no close phylogenetic relatives exist for Phymm-based parameterization
- You need a fast, cheap annotation pass — Glimmer-MG is explicitly more computationally expensive than existing software

## Inputs

- Prokaryotic genome sequences in FASTA format with curated gene annotations as the truth set
- Metagenomic shotgun reads or contigs (e.g. simulated 454 reads, real 454 human gut microbiome reads)
- Initial long-ORF training set for the first-pass Glimmer IMM model

## Outputs

- Per-genome predicted gene coordinates and start positions (Glimmer/Glimmer-MG prediction files)
- 3' (start-site) and 5' sensitivity and precision per genome and averaged across genomes (e.g. supplementary_table_s1_accuracy.csv)
- Retrained IMM parameter sets per cluster (Glimmer-MG) or per genome iteration

## How to apply

For whole genomes, run the iterated Glimmer3.0 pipeline: train an initial long-ORF interpolated Markov model (3-periodic nonhomogeneous Markov models, 1st through 8th order, weighted by predictive power), predict genes, then iterate the model-building step on the initial predictions. To emulate Glimmer3.1, discard predicted genes entirely contained within longer predictions (length/adjacency filtering), which reduces erroneous short predictions (42 predictions <150 bp per genome vs 68 for Glimmer3.0). For metagenomic sequences, replace GC-content-based training with Phymm phylogenetic classification to parameterize models, cluster reads with Scimm so sequences from the same organism group together, retrain models within each cluster on initial predictions, and enable the gene-length/error models (indel and stop-codon substitution handling for 454 homopolymer errors). Compare predicted starts to curated annotations to compute 3' and 5' sensitivity/precision per genome.

## Related tools

- **Glimmer** (Whole-genome IMM-based gene predictor whose long-ORF model-building step is iterated on initial predictions (Glimmer3.0) with length/adjacency filtering (Glimmer3.1))
- **Glimmer-MG** (Metagenomic extension of the Glimmer framework that classifies with Phymm, clusters with Scimm, retrains per cluster, and adds gene-length and sequencing-error models) — https://github.com/davek44/Glimmer-MG
- **Phymm** (Provides phylogenetic classifications of sequences used to parameterize the gene prediction models instead of GC-content)
- **Scimm** (Clusters metagenomic sequences by likely organism of origin so models can be retrained unsupervised within each cluster)

## Examples

```
glimmer3 genome.fna longorfs.icm genome.predictions && glimmer3 genome.fna genome.predictions.icm genome.final  # iterate long-ORF training, then filter contained predictions (Glimmer3.1-style)
```

## Evaluation signals

- 3' precision of Glimmer3.1 improves ~1.3% on average over Glimmer3.0 across the 12 reference genomes while 3' sensitivity is maintained
- Erroneous short predictions drop to ~42 predictions <150 bp per genome with length filtering, versus 68 for the unfiltered Glimmer3.0 pipeline
- Glimmer identifies >99% of genes in complete prokaryotic genomes — sensitivity on complete genomes should be near this ceiling
- Predicted start codon calls match reference annotations exactly or nearly (near-correct starts counted separately) when computing 3'/5' precision
- Glimmer-MG outperforms prior systems on simulated and real metagenomes across all tested read lengths and error rates (e.g. 73.2% sensitivity / 54.7% precision on 454 reads at 2.0% error)

## Limitations

- Predicting genes on error-prone sequences, particularly those with indels, remains a difficult problem
- Glimmer-MG is more computationally expensive than existing software
- Highly diverged organisms where the optimal model parameters are unclear present a challenge to metagenomics gene prediction
- Substantial further gains depend on reducing error rate via improved assembly and base-calling rather than computation
- No software version is stated for Glimmer-MG or the compared tools

## Evidence

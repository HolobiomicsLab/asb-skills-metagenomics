---
name: metagenomic-sequence-phylogenetic-classification
description: Use when gene-predicting unassembled metagenomic reads where GC-content
  is a poor proxy for choosing training genomes, and phylogenetic classification is
  needed to parameterize per-organism gene prediction models.
when_to_use_negative:
- Input is complete prokaryotic genomes, where standard Glimmer with >99% gene identification
  and iterative retraining on the whole genome is appropriate rather than classification-augmented
  metagenomic prediction
- Sequences are already assembled and binned per organism, making classification-based
  model selection and clustering unnecessary
- Computational cost is the binding constraint, since Glimmer-MG is more computationally
  expensive than existing gene prediction software
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Phymm
  role: Phylogenetic classifier applied to metagenomic sequences to parameterize gene
    prediction models instead of using GC-content
- name: Scimm
  role: Clustering method whose clusters are augmented with Phymm classifications
    (PhyScimm) to group sequences by likely organism of origin
- name: Glimmer-MG
  role: Gene prediction system consuming the phylogenetic classifications and clusters
    to parameterize and retrain IMM, RBS, and start-codon models before final prediction
  repo: https://github.com/davek44/Glimmer-MG
- name: Glimmer
  role: Base IMM framework on which Glimmer-MG and its model parameterization are
    built
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkr1067
    title: Gene prediction with Glimmer for metagenomic sequences augmented by classification
      and clustering
  source_content_hash: sha256:0245560559f3346416d3323189a4a373ee9eaa98902769b4c71f58333fbe2d25
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
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# metagenomic-sequence-phylogenetic-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns phylogenetic classifications to environmental shotgun DNA sequences with the Phymm classifier so that appropriate training genomes can be selected to parameterize Glimmer interpolated Markov models, replacing GC-content-based model selection in metagenomic gene prediction.

## When to use

Use when gene-predicting unassembled metagenomic reads where GC-content is a poor proxy for choosing training genomes, and phylogenetic classification is needed to parameterize per-organism gene prediction models.

## When NOT to use

- Input is complete prokaryotic genomes, where standard Glimmer with >99% gene identification and iterative retraining on the whole genome is appropriate rather than classification-augmented metagenomic prediction
- Sequences are already assembled and binned per organism, making classification-based model selection and clustering unnecessary
- Computational cost is the binding constraint, since Glimmer-MG is more computationally expensive than existing gene prediction software

## Inputs

- Environmental shotgun DNA sequences (metagenomic reads, e.g., 454 or Illumina) in FASTA format
- Phymm classification models / reference genomes used for phylogenetic classification

## Outputs

- Per-sequence phylogenetic classifications used for gene prediction model parameterization
- Phylogenetically augmented sequence clusters (PhyScimm) grouping reads by likely organism of origin
- Final gene predictions (gene_predictions.gff, valid GFF with predicted features for all input sequences)

## How to apply

Run the Phymm phylogenetic classifier on each input metagenomic sequence (e.g., 454 or Illumina reads) to obtain evolutionary classifications, which are recommended over GC-content because Phymm-based parameterization 'works much better than GC-content'. Use these classifications at two downstream points: (1) to select appropriate training genomes for parameterizing the Glimmer IMM, RBS, and start-codon models, and (2) as phylogenetic augmentation to Scimm clustering (PhyScimm), combining sequence similarity with classification labels so reads likely from the same organism group together before per-cluster retraining. Classification and clustering are required before prediction; only clusters containing at least 80 Kb of predicted coding sequence proceed to retraining, while smaller clusters fall back to likelihood-ratio selection over available models. The classified-and-clustered sequences then feed the final Glimmer-MG prediction pass, which models gene length and insertion/deletion and substitution errors such as homopolymer-run indels in 454 sequences.

## Related tools

- **Phymm** (Phylogenetic classifier applied to metagenomic sequences to parameterize gene prediction models instead of using GC-content)
- **Scimm** (Clustering method whose clusters are augmented with Phymm classifications (PhyScimm) to group sequences by likely organism of origin)
- **Glimmer-MG** (Gene prediction system consuming the phylogenetic classifications and clusters to parameterize and retrain IMM, RBS, and start-codon models before final prediction) — https://github.com/davek44/Glimmer-MG
- **Glimmer** (Base IMM framework on which Glimmer-MG and its model parameterization are built)

## Examples

```
git clone https://github.com/davek44/Glimmer-MG; per README, install Glimmer-MG and see manual.pdf for instructions on installation and running Glimmer-MG
```

## Evaluation signals

- Classification-driven parameterization outperforms GC-content-based selection on simulated metagenomes (the paper shows Phymm-based parameterization works much better than GC-content)
- PhyScimm clusters enabling retraining contain at least 80 Kb of predicted coding sequence; smaller heterogeneous clusters fall back to likelihood-ratio model selection
- Validation: confirm gene_predictions.gff exists, parses as valid GFF, and contains predicted features for all input sequences
- Benchmark against simulated 454 reads with 2.0% error: Glimmer-MG sensitivity 73.2% and precision 54.7% represent the reported operating range
- Glimmer-MG predictions should show greater accuracy than previous systems and be the most sensitive and precise across tested read lengths and error rates

## Limitations

- Predicting genes on error-prone sequences, particularly those with indels (e.g., homopolymer-run indels in 454 reads), remains a difficult problem; precision is only 54.7% at 2.0% error
- Glimmer-MG is more computationally expensive than existing gene prediction software
- Highly diverged organisms where the optimal model parameters are unclear present a challenge to metagenomics gene prediction
- Substantial further gains depend on reducing error rates via improved assembly and base-calling rather than computation
- No software version stated for Glimmer-MG or compared tools

## Evidence

- [other] only clusters with at least 80 Kb of predicted coding sequence are retrained: [evidence span withheld — anchor status: not_found]

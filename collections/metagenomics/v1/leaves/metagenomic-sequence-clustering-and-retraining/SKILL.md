---
name: metagenomic-sequence-clustering-and-retraining
description: Use when predicting genes in unassembled environmental shotgun reads
  (e.g. 454 reads from the human gut microbiome) where GC-content alone is a poor
  proxy for training model parameterization and an unsupervised organism-based grouping
  plus retraining step is needed.
when_to_use_negative:
- Input is a complete prokaryotic genome, where standard Glimmer (>99% gene identification)
  is already sufficient and no clustering/retraining is needed
- Reads have been assembled into long contigs with well-estimated GC-content, where
  GC-based model parameterization may suffice without Phymm/Scimm
- Compute budget is minimal — Glimmer-MG's classification, clustering, and multiple
  iterations are more computationally expensive than existing software
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: Glimmer-MG
  role: Gene prediction system that orchestrates Phymm classification, Scimm clustering,
    and cluster-level model retraining before final prediction
  repo: github.com/davek44/Glimmer-MG
- name: Phymm
  role: Phylogenetic classifier whose sequence classifications parameterize the gene
    prediction models instead of GC-content
- name: Scimm
  role: Unsupervised clustering method that groups reads likely from the same organism
    to enable per-cluster retraining
- name: Glimmer
  role: Underlying gene prediction framework (interpolated Markov models) on which
    Glimmer-MG is built
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkr1067
    title: Gene prediction with Glimmer for metagenomic sequences augmented by classification
      and clustering
  source_content_hash: sha256:46bdc4bc8f72482a1f7e2025f087494467680ceefac69831326a5aaf1e62ad4c
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
  - Phymm
  - Scimm
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# metagenomic-sequence-clustering-and-retraining

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Clusters metagenomic shotgun reads by likely organism of origin with Scimm (after Phymm-based phylogenetic classification) and retrains Glimmer-MG gene prediction models within each cluster on initial gene predictions before making final predictions, boosting full-gene accuracy on error-prone metagenomic reads.

## When to use

Use when predicting genes in unassembled environmental shotgun reads (e.g. 454 reads from the human gut microbiome) where GC-content alone is a poor proxy for training model parameterization and an unsupervised organism-based grouping plus retraining step is needed.

## When NOT to use

- Input is a complete prokaryotic genome, where standard Glimmer (>99% gene identification) is already sufficient and no clustering/retraining is needed
- Reads have been assembled into long contigs with well-estimated GC-content, where GC-based model parameterization may suffice without Phymm/Scimm
- Compute budget is minimal — Glimmer-MG's classification, clustering, and multiple iterations are more computationally expensive than existing software

## Inputs

- Unassembled environmental shotgun metagenomic sequencing reads (e.g. 454 reads from human gut microbiome samples TS28 and TS50)
- FASTA/FASTQ metagenomic sequence files

## Outputs

- Cluster assignments of reads by likely organism of origin (Scimm clusters)
- Final Glimmer-MG gene predictions with coordinates on each read, after cluster-level retraining
- Predicted coding-region translations usable for downstream BLAST validation

## How to apply

First classify each input read with Phymm, whose phylogenetic classifications are used to parameterize the gene prediction models in place of GC-content. Then cluster the sequences using Scimm, which groups reads likely to have originated from the same organism. Within each cluster, run an initial Glimmer-MG gene prediction pass, then retrain the interpolated Markov models on those initial predictions before making the final set of predictions. Finally, run gene prediction with modeling of insertion/deletion and substitution errors so predictions can track coding frames across homopolymer-run indels characteristic of 454 sequencing. Rationale: training on a single mixed-community model fails for metagenomes; cluster-specific retraining recovers organism-specific model parameters without reference genomes.

## Related tools

- **Glimmer-MG** (Gene prediction system that orchestrates Phymm classification, Scimm clustering, and cluster-level model retraining before final prediction) — github.com/davek44/Glimmer-MG
- **Phymm** (Phylogenetic classifier whose sequence classifications parameterize the gene prediction models instead of GC-content)
- **Scimm** (Unsupervised clustering method that groups reads likely from the same organism to enable per-cluster retraining)
- **Glimmer** (Underlying gene prediction framework (interpolated Markov models) on which Glimmer-MG is built)

## Examples

```
perl glimmer-mg.pl --input gut_reads.fasta --output gene_predictions.txt
```

## Evaluation signals

- Final gene predictions on real 454 human gut reads (TS28, TS50), when translated and BLASTed against NCBI nr at E < 0.001, should match Table 3 counts — Glimmer-MG predicted the most BLAST-aligning genes, 4.5% more than FragGeneScan and 6.5% more than MetaGeneMark
- Each Scimm cluster must have its models retrained on initial gene predictions before the final prediction pass — skipping retraining loses the reported full-gene accuracy boost
- On simulated 454 reads with 2.0% error, sensitivity and precision should be in the reported range (~73.2% sensitivity, ~54.7% precision)
- Predicted genes on error-prone 454 reads should track coding frames across homopolymer-run indels (indel-error calls), outperforming FragGeneScan on indel position identification
- Per-cluster retraining should yield higher full-gene accuracy than a single untrained/community-wide model

## Limitations

- Predicting genes on error-prone sequences, particularly those with indels, remains a difficult problem (precision 54.7% at 2.0% simulated 454 error)
- Glimmer-MG is more computationally expensive than existing software
- Highly diverged organisms where optimal model parameters are unclear present a challenge
- Substantial further gains likely depend on reducing error rate via assembly and base-calling improvements rather than computation

## Evidence

- [readme] Glimmer-MG branches into a different frame at homopolymer runs to handle 454 indels: [evidence span withheld — anchor status: external_doc]

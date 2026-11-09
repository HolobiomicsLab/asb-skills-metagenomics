---
name: open-source-bioinformatics-pipeline-execution
description: Use when you have simulated or real metagenomic shotgun reads (454, Sanger,
  or Illumina) and need to run an open-source gene prediction pipeline end-to-end,
  from installation through prediction output for benchmarking.
when_to_use_negative:
- Input is complete prokaryotic genomes rather than metagenomic fragments — plain
  Glimmer routinely identifies >99% of genes there and Glimmer-MG's training machinery
  is unnecessary
- Reads are error-free and pre-assembled with no need for indel/frame-error modeling
  — the added computational cost of classification, clustering, and error modeling
  is not justified
- The task is taxonomic classification or assembly rather than gene finding — Glimmer-MG
  outputs gene coordinates, not an assembled metagenome or abundance table
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: Glimmer-MG
  role: Primary gene-prediction pipeline executed on metagenomic reads; incorporates
    classification, clustering, retraining, and sequencing-error modeling
  repo: http://www.cbcb.umd.edu/software/glimmer-mg
- name: Phymm
  role: Phylogenetic classifier used inside the pipeline to parameterize gene prediction
    models instead of GC-content
- name: Scimm
  role: Unsupervised clustering method used inside the pipeline to group reads by
    organism of origin before model retraining
- name: 454 Life Sciences
  role: Sequencing platform whose homopolymer-run error model is simulated for benchmarking
    reads processed by the pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkr1067
    title: Gene prediction with Glimmer for metagenomic sequences augmented by classification
      and clustering
  source_content_hash: sha256:267ee8bc30d11cc6f61daef5d54be81bbb8e609e849436aa3963e2a181b2adad
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
  - Glimmer-MG
  - Phymm
  - Scimm
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# open-source-bioinformatics-pipeline-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Install and execute the Glimmer-MG metagenomic gene-prediction pipeline (classification via Phymm, clustering via Scimm, then IMM-based gene prediction) on shotgun sequencing reads, reproducing the Nucleic Acids Research gkr1067 benchmark workflow.

## When to use

Use when you have simulated or real metagenomic shotgun reads (454, Sanger, or Illumina) and need to run an open-source gene prediction pipeline end-to-end, from installation through prediction output for benchmarking.

## When NOT to use

- Input is complete prokaryotic genomes rather than metagenomic fragments — plain Glimmer routinely identifies >99% of genes there and Glimmer-MG's training machinery is unnecessary
- Reads are error-free and pre-assembled with no need for indel/frame-error modeling — the added computational cost of classification, clustering, and error modeling is not justified
- The task is taxonomic classification or assembly rather than gene finding — Glimmer-MG outputs gene coordinates, not an assembled metagenome or abundance table

## Inputs

- Metagenomic shotgun sequence reads in FASTA (simulated 454/Sanger/Illumina reads or real 454 human gut microbiome reads)
- Reference GenBank genomes used to simulate reads (e.g. via FlowSim with a Pareto abundance distribution)
- manual.pdf installation/usage instructions shipped with the Glimmer-MG repository

## Outputs

- Predicted gene coordinates (start/end positions and coding frames) per input read
- Predicted insertion/deletion and stop-codon substitution positions within reads
- Benchmark CSV of nucleotide-level 5'/3' sensitivity and precision per predictor (e.g. table1_sensitivity_precision.csv)

## How to apply

Install Glimmer-MG from the shipped manual.pdf or from www.cbcb.umd.edu/software/glimmer-mg / github.com/davek44/Glimmer-MG. Feed in FASTA-format environmental shotgun sequences; Glimmer-MG first classifies each read using the Phymm phylogenetic classifier to parameterize interpolated Markov models (IMMs, 1st- through 8th-order, 3-periodic nonhomogeneous), then clusters reads with Scimm, retrains models within each cluster on initial predictions, and emits final gene calls with a probabilistic model for gene length and start/stop codon presence to handle truncated genes. For error-prone reads, enable the model that predicts insertion, deletion, and stop codon-introducing substitution errors (e.g. frame-shifts at 454 homopolymer runs) so predictions track coding frames across low-quality bases. Run the pipeline in both its recommended classification+clustering mode and Precise mode, and compare against baseline predictors such as MetaGeneAnnotator, MetaGeneMark, and FragGeneScan on the identical read set.

## Related tools

- **Glimmer-MG** (Primary gene-prediction pipeline executed on metagenomic reads; incorporates classification, clustering, retraining, and sequencing-error modeling) — http://www.cbcb.umd.edu/software/glimmer-mg
- **Phymm** (Phylogenetic classifier used inside the pipeline to parameterize gene prediction models instead of GC-content)
- **Scimm** (Unsupervised clustering method used inside the pipeline to group reads by organism of origin before model retraining)
- **454 Life Sciences** (Sequencing platform whose homopolymer-run error model is simulated for benchmarking reads processed by the pipeline)

## Evaluation signals

- The sensitivity and precision output table contains values in [0, 100] for every predictor (Glimmer-MG, MetaGeneAnnotator, MetaGeneMark, FragGeneScan) and every read length/error-rate condition
- Glimmer-MG's reported values match the paper's reference numbers, e.g. 73.2% sensitivity and 54.7% precision on simulated 454 reads with a 2.0% error rate
- Glimmer-MG achieves the greatest sensitivity for every read length tested on error-free simulated metagenomes, with 3' sensitivity 1.3–1.8% above the second-best method
- Indel positions predicted by Glimmer-MG align more accurately to the simulated indel sites than those predicted by FragGeneScan on the same reads
- Pipeline completes both the recommended classification+clustering mode and Precise mode on the identical input FASTA without installation errors reported by manual.pdf steps

## Limitations

- Predicting genes on error-prone sequences, particularly those with indels, remains a difficult problem
- Glimmer-MG is more computationally expensive than existing software
- Highly diverged organisms where the optimal model parameters are unclear present a challenge to metagenomics gene prediction
- Substantial further gains are more likely to come from reducing sequencing error rates (better assembly and base-calling) than from improved computation
- No software version is stated for Glimmer-MG or the compared tools in the paper

## Evidence

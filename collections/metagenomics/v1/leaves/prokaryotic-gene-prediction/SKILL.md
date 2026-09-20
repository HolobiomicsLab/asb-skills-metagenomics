---
name: prokaryotic-gene-prediction
description: Use when predicting genes in complete prokaryotic genomes or environmental
  shotgun/metagenomic sequences (e.g. 454/Illumina reads) with FASTA input and curated
  reference annotations, or when comparing Glimmer3.0/3.1/Glimmer-MG accuracy.
when_to_use_negative:
- Input is already a gene annotation or GFF/feature table rather than DNA sequence
- Sequences are high-quality assembled genomes with known organism — plain iterated
  Glimmer3.1 suffices and Glimmer-MG's classification/clustering overhead is unnecessary
  (it is more computationally expensive)
- Samples are RNA/protein data rather than DNA coding-region prediction
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: Glimmer
  role: Baseline whole-genome gene predictor using 3-periodic nonhomogeneous interpolated
    Markov models; iterated long-ORF training (Glimmer3.0) and length/adjacency-filtered
    variant (Glimmer3.1)
- name: Glimmer-MG
  role: Metagenomics gene prediction system built on the Glimmer framework; parameterizes
    models via Phymm classification, clusters with Scimm, retrains per cluster, and
    models indel/substitution errors
  repo: github.com/davek44/Glimmer-MG
- name: Phymm
  role: Phylogenetic classifier whose ICM-based classifications replace GC-content
    for parameterizing Glimmer-MG gene prediction models
- name: Scimm
  role: Clustering method that groups metagenomic sequences likely from the same organism,
    enabling the unsupervised retraining step
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkr1067
    title: Gene prediction with Glimmer for metagenomic sequences augmented by classification
      and clustering
  source_content_hash: sha256:712cafb8f9f0d348199e49563053839c436e8d9602d40bd9ecc37c479e830c61
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/davek44/Glimmer-MG
  tools:
  - Glimmer
  - Glimmer-MG
  - Phymm
  - Scimm
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# prokaryotic-gene-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run the Glimmer family of interpolated Markov model gene predictors — iterated Glimmer3.0, length/adjacency-filtered Glimmer3.1, and classification+clustering-augmented Glimmer-MG — to find coding regions in complete prokaryotic genomes and metagenomic sequence sets, then score 3'/5' sensitivity and precision against curated annotations.

## When to use

Use when predicting genes in complete prokaryotic genomes or environmental shotgun/metagenomic sequences (e.g. 454/Illumina reads) with FASTA input and curated reference annotations, or when comparing Glimmer3.0/3.1/Glimmer-MG accuracy.

## When NOT to use

- Input is already a gene annotation or GFF/feature table rather than DNA sequence
- Sequences are high-quality assembled genomes with known organism — plain iterated Glimmer3.1 suffices and Glimmer-MG's classification/clustering overhead is unnecessary (it is more computationally expensive)
- Samples are RNA/protein data rather than DNA coding-region prediction

## Inputs

- Prokaryotic genome sequences in FASTA format
- Curated gene annotations (truth set) for reference genomes
- Metagenomic shotgun sequences (simulated or real 454/Illumina reads)

## Outputs

- Predicted gene coordinates and start positions per genome/contig
- Per-genome and averaged 3' and 5' sensitivity and precision tables (e.g. supplementary_table_s1_accuracy.csv)

## How to apply

For complete genomes, train Glimmer on long ORFs, run the iterated Glimmer3.0 pipeline, and re-run with the Glimmer3.1 pipeline that discards predicted genes entirely contained within longer predictions (length/adjacency filtering) — this retains ~99% sensitivity while cutting erroneous short predictions from 68 to 42 per genome (<150 bp) and improving 3' precision by 1.3% on average. For metagenomic sequences, run Glimmer-MG: first classify sequences with Phymm-style interpolated Markov models (instead of GC-content) to parameterize prediction models, cluster sequences with Scimm, retrain models within each cluster on initial predictions, and apply a probabilistic model for gene length and start/stop codon presence to handle truncated genes; enable indel/stop-codon-error modeling for raw 454 reads. Score predictions by comparing predicted genes and start sites to reference annotations, computing 3' (start-site) and 5' sensitivity and precision per genome. Aggregate per-genome and averaged metrics (e.g. supplementary_table_s1_accuracy.csv) across the 12 reference prokaryotic genomes to judge improvement.

## Related tools

- **Glimmer** (Baseline whole-genome gene predictor using 3-periodic nonhomogeneous interpolated Markov models; iterated long-ORF training (Glimmer3.0) and length/adjacency-filtered variant (Glimmer3.1))
- **Glimmer-MG** (Metagenomics gene prediction system built on the Glimmer framework; parameterizes models via Phymm classification, clusters with Scimm, retrains per cluster, and models indel/substitution errors) — github.com/davek44/Glimmer-MG
- **Phymm** (Phylogenetic classifier whose ICM-based classifications replace GC-content for parameterizing Glimmer-MG gene prediction models)
- **Scimm** (Clustering method that groups metagenomic sequences likely from the same organism, enabling the unsupervised retraining step)

## Evaluation signals

- Whole-genome sensitivity should match the paper's benchmark of routinely identifying >99% of genes in complete prokaryotic genomes
- Glimmer3.1 vs Glimmer3.0: average 3' precision improves ~1.3% while 3' sensitivity is maintained; erroneous short predictions (<150 bp) drop from ~68 to ~42 per genome
- Per-genome 3'/5' sensitivity and precision values for all three pipelines reproduce the per-genome and averaged values of Supplementary Table S1 on the 12 reference genomes
- On simulated 454 reads with 2.0% error rate, Glimmer-MG should reach ~73.2% sensitivity and ~54.7% precision, outperforming alternatives such as FragGeneScan on indel position accuracy
- Output table parses as expected schema with one row per genome per pipeline (per-genome plus averaged sensitivity/precision columns)

## Limitations

- Predicting genes on error-prone sequences, particularly those with indels, remains a difficult problem — at 2.0% simulated 454 error, precision falls to 54.7%
- Glimmer-MG is more computationally expensive than existing software
- Highly diverged organisms where the optimal model parameters are unclear present a challenge
- Substantial gains depend more on reducing error rate via improved assembly and base-calling than on computation

## Evidence

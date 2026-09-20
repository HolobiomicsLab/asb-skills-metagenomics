---
name: protein-sequence-clustering
description: Use when a set of predicted viral proteins (e.g., from Prodigal on viral
  contigs) must be dereplicated into a non-redundant catalog prior to annotation with
  eggNOG-mapper, VOGDB, or DRAM-v.
when_to_use_negative:
- Input is already a non-redundant gene catalog or single representative sequences
  per gene — clustering would discard real diversity
- Sequences are nucleotide contigs rather than predicted proteins — the workflow predicts
  genes with Prodigal first
- The goal is viral discovery/binning rather than annotation preparation — use VirSorter2/VIBRANT
  or vRhyme/phamb instead
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3697
tools:
- name: MMseqs2
  role: Clusters redundant predicted protein sequences into a non-redundant gene catalog
    (identity 0.7, coverage 0.9)
  repo: https://github.com/soedinglab/MMseqs2
- name: Prodigal
  role: Predicts protein-coding genes on input viral contigs to produce the protein
    sequences to be clustered
- name: eggNOG-mapper
  role: Downstream consumer of the clustered proteins; annotates representatives against
    EggNOG5 for KEGG/PFAM/COG hits
- name: DRAM-v
  role: Downstream consumer of the clustered proteins; distills auxiliary metabolic
    gene (AMG) calls
  repo: https://github.com/WrightonLabCSU/DRAM
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1080/19490976.2023.2192522
    title: 'ViroProfiler: a containerized bioinformatics pipeline for viral metagenomic
      data analysis'
  source_content_hash: sha256:56640e05b28a73c0ea3e05b7e5127d2c8bb61feade1c0b4cf0771769420f0f07
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/deng-lab/viroprofiler
  repo_url: https://github.com/soedinglab/MMseqs2
  tools:
  - DRAM-v
  - EggNOG
  - MMseqs2
  - Prodigal
  - VOGDB
  - ViroProfiler
  - eggNOG-mapper
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# protein-sequence-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds a non-redundant gene catalog from protein-coding predictions on viral contigs by clustering redundant protein sequences with MMseqs2 before functional annotation in the ViroProfiler pipeline.

## When to use

Use when a set of predicted viral proteins (e.g., from Prodigal on viral contigs) must be dereplicated into a non-redundant catalog prior to annotation with eggNOG-mapper, VOGDB, or DRAM-v.

## When NOT to use

- Input is already a non-redundant gene catalog or single representative sequences per gene — clustering would discard real diversity
- Sequences are nucleotide contigs rather than predicted proteins — the workflow predicts genes with Prodigal first
- The goal is viral discovery/binning rather than annotation preparation — use VirSorter2/VIBRANT or vRhyme/phamb instead

## Inputs

- Protein FASTA file of predicted protein-coding sequences from viral contigs (Prodigal output)
- Nucleotide FASTA file of viral contigs (context for gene-to-contig mapping)

## Outputs

- Non-redundant protein sequence catalog (cluster representatives)
- Cluster membership mapping from redundant protein IDs to representative gene IDs

## How to apply

In the ViroProfiler workflow, first predict protein-coding genes on the input viral contigs with Prodigal to obtain nucleotide and protein FASTA files. Then cluster the redundant protein sequences with MMseqs2 using an identity threshold of 0.7 and a coverage threshold of 0.9 to produce a non-redundant gene catalog. Clustering reduces redundancy so that downstream annotation steps (eggNOG-mapper/EMAPPER against EggNOG5, VOGDB, and DRAM-v for AMG calling) run once per representative sequence, and results can later be merged back by gene ID into a tabular gene-by-annotation table. MMseqs2 is chosen because it is an ultra-fast sequence search and clustering suite suitable for huge protein sets.

## Related tools

- **MMseqs2** (Clusters redundant predicted protein sequences into a non-redundant gene catalog (identity 0.7, coverage 0.9)) — https://github.com/soedinglab/MMseqs2
- **Prodigal** (Predicts protein-coding genes on input viral contigs to produce the protein sequences to be clustered)
- **eggNOG-mapper** (Downstream consumer of the clustered proteins; annotates representatives against EggNOG5 for KEGG/PFAM/COG hits)
- **DRAM-v** (Downstream consumer of the clustered proteins; distills auxiliary metabolic gene (AMG) calls) — https://github.com/WrightonLabCSU/DRAM

## Examples

```
mmseqs easy-cluster predicted_proteins.faa nr_gene_catalog tmp --min-seq-id 0.7 -c 0.9
```

## Evaluation signals

- The clustering step completes with MMseqs2 and produces cluster representative and cluster TSV files covering all input proteins
- Every representative in the non-redundant catalog maps back to a gene predicted on an input viral contig (contig IDs match the input FASTA)
- Downstream gene_annotation_table.tsv parses as a tab-separated table and every gene ID corresponds to a gene predicted on an input viral contig
- Cluster count is strictly less than or equal to the number of input protein sequences (dereplication actually reduced redundancy)
- Merged eggNOG, VOGDB, and DRAM-v annotations resolve per gene ID without orphan representatives

## Limitations

- Excessive environmental contaminations in metagenome sequences could increase the running time of the pipeline and result in lower precision
- Identity 0.7 / coverage 0.9 thresholds determine what counts as redundant; divergent paralogs above these thresholds will remain separate entries in the catalog
- ViroProfiler is specifically designed for classifying viruses in samples with isolated viruses; contaminated host-rich inputs are outside its validated scope

## Evidence

- [readme] MMseqs2 clustering role: [evidence span withheld — anchor status: external_doc]
- [readme] MMseqs2 used for redundancy checks on protein features: [evidence span withheld — anchor status: external_doc]

---
name: annotation-table-integration
description: Use when you have protein-coding genes predicted on viral contigs and
  need per-gene functional hits from multiple databases (EggNOG5, VOGDB) plus auxiliary
  metabolic gene (AMG) calls merged into one tabular annotation table.
when_to_use_negative:
- When the goal is viral taxonomic classification or host prediction rather than gene-level
  functional annotation — ViroProfiler's annotation step targets gene products, not
  taxonomy.
- When the input sequences are not viral (e.g., whole microbial metagenome bins) —
  DRAM-v is applied here specifically to distill AMG calls for viral contigs.
- When samples contain excessive environmental contamination, since the article notes
  this could increase the pipeline running time and result in lower precision.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: Prodigal
  role: Predicts protein-coding genes on input viral contigs to produce nucleotide
    and protein sequences for annotation
  repo: https://github.com/hyattpd/Prodigal/wiki/installation
- name: MMseqs2
  role: Clusters redundant protein sequences (identity 0.7, coverage 0.9) into a non-redundant
    gene catalog before annotation
  repo: https://github.com/soedinglab/MMseqs2
- name: EggNOG-mapper
  role: Annotates clustered proteins against EggNOG5 to retrieve KEGG, PFAM, and COG
    functional hits
  repo: http://eggnog5.embl.de/
- name: VOGDB
  role: Database used to annotate the clustered proteins with viral orthologous group
    hits (configured with 4 CPUs, 40 GB memory)
- name: DRAM-v
  role: Distills auxiliary metabolic gene (AMG) calls for the viral contigs from the
    gene annotations
  repo: https://github.com/WrightonLabCSU/DRAM
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1080/19490976.2023.2192522
    title: 'ViroProfiler: a containerized bioinformatics pipeline for viral metagenomic
      data analysis'
  source_content_hash: sha256:f72d13f5ba0e4000b916fd668abef9a120ca40ca2f410d199c569fa66f16b003
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/deng-lab/viroprofiler
  tools:
  - DRAM-v
  - EggNOG
  - EggNOG-mapper
  - MMseqs2
  - Prodigal
  - VOGDB
  - ViroProfiler
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# annotation-table-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

ViroProfiler's functional-annotation integration skill: merge eggNOG-mapper, VOGDB, and DRAM-v outputs into a single gene-by-annotation table with an AMG flag, after building a non-redundant gene catalog from viral contigs.

## When to use

Use when you have protein-coding genes predicted on viral contigs and need per-gene functional hits from multiple databases (EggNOG5, VOGDB) plus auxiliary metabolic gene (AMG) calls merged into one tabular annotation table.

## When NOT to use

- When the goal is viral taxonomic classification or host prediction rather than gene-level functional annotation — ViroProfiler's annotation step targets gene products, not taxonomy.
- When the input sequences are not viral (e.g., whole microbial metagenome bins) — DRAM-v is applied here specifically to distill AMG calls for viral contigs.
- When samples contain excessive environmental contamination, since the article notes this could increase the pipeline running time and result in lower precision.

## Inputs

- Viral contig set (FASTA) identified upstream by VirSorter2/VIBRANT/DeepVirFinder
- Nucleotide and protein sequences from Prodigal gene prediction on the viral contigs

## Outputs

- gene_annotation_table.tsv — a tab-separated gene-by-annotation table with gene ID, predicted function, KEGG/PFAM/VOGDB hits, and a boolean AMG flag
- eggNOG-mapper annotations with KEGG, PFAM, and COG functional hits against EggNOG5
- DRAM-v AMG calls distilled for the viral contigs

## How to apply

First predict protein-coding genes on the input viral contigs with Prodigal to obtain nucleotide and protein sequences. Cluster redundant proteins with MMseqs2 at an identity threshold of 0.7 and coverage threshold of 0.9 to build a non-redundant gene catalog. Annotate the clustered proteins with eggNOG-mapper against the EggNOG5 database to retrieve KEGG, PFAM, and COG functional hits, and in parallel against VOGDB for viral orthologous groups; then run DRAM-v to distill auxiliary metabolic gene (AMG) calls for the viral contigs. Finally, merge the eggNOG, VOGDB, and DRAM-v annotation results by gene ID and compile a tabular gene-by-annotation table with columns for gene ID, predicted function, KEGG/PFAM/VOGDB hits, and a boolean AMG flag. Resource-wise, the pipeline configures the EMAPPER step with 22 CPUs and 300 GB memory, VOGDB with 4 CPUs and 40 GB, and the DRAMV step with 1 CPU and 200 GB, reflecting the memory-heavy database lookups.

## Related tools

- **Prodigal** (Predicts protein-coding genes on input viral contigs to produce nucleotide and protein sequences for annotation) — https://github.com/hyattpd/Prodigal/wiki/installation
- **MMseqs2** (Clusters redundant protein sequences (identity 0.7, coverage 0.9) into a non-redundant gene catalog before annotation) — https://github.com/soedinglab/MMseqs2
- **EggNOG-mapper** (Annotates clustered proteins against EggNOG5 to retrieve KEGG, PFAM, and COG functional hits) — http://eggnog5.embl.de/
- **VOGDB** (Database used to annotate the clustered proteins with viral orthologous group hits (configured with 4 CPUs, 40 GB memory))
- **DRAM-v** (Distills auxiliary metabolic gene (AMG) calls for the viral contigs from the gene annotations) — https://github.com/WrightonLabCSU/DRAM

## Evaluation signals

- gene_annotation_table.tsv exists and parses as a tab-separated table
- The table contains the required fields: gene ID, predicted function, KEGG/PFAM/VOGDB hits, and a boolean AMG flag
- Every gene ID in the table corresponds to a gene predicted on an input viral contig
- Annotations from eggNOG, VOGDB, and DRAM-v are successfully merged by gene ID (no per-tool separate deliverables left unmerged)
- The non-redundant gene catalog reflects MMseqs2 clustering at identity 0.7 / coverage 0.9, so no duplicated proteins carry separate annotation rows

## Limitations

- ViroProfiler is specifically designed for classifying viruses in samples with isolated viruses; viral fractions should be isolated before sequencing for accurate estimation.
- Excessive environmental contaminations, usually found in metagenome sequences, could increase the running time of the pipeline and result in lower precision.
- DRAM2 is currently under active development and usage is at your own risk, per its README.

## Evidence

- [discussion] Moreover, it can detect viral replication cycles, predict hosts, and identify AMGs in viral sequences.: "Moreover, it can detect viral replication cycles, predict hosts, and identify AMGs in viral sequences."
- [methods] Gene function annotation | [DRAM-v](https://github.com/WrightonLabCSU/DRAM): "Gene function annotation | [DRAM-v](https://github.com/WrightonLabCSU/DRAM)"
- [methods] withName: VOGDB { cpus = 4 memory = "40 GB": "withName: VOGDB {
        cpus = 4
        memory = "40 GB""
- [readme] DRAM2 is currently under active development and usage is at your own risk.: [evidence span withheld — anchor status: not_found]
- [abstract] Here we present ViroProfiler, an in-silico workflow for analyzing shotgun viral metagenomic data.: "Here we present ViroProfiler, an in-silico workflow for analyzing shotgun viral metagenomic data."

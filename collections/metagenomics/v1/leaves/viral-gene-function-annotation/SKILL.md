---
name: viral-gene-function-annotation
description: Use when input is a set of predicted viral contigs (from VirSorter2/VIBRANT
  etc.) and the goal is gene prediction, functional annotation (KEGG/PFAM/COG/VOG
  hits), and AMG identification.
when_to_use_negative:
- Input is raw reads or an unassembled metagenome rather than a curated set of predicted
  viral contigs — run viral discovery/assembly (e.g. metaSPAdes + VirSorter2/VIBRANT)
  first
- 'The metagenome has excessive environmental contamination: the article notes this
  can increase pipeline running time and lower precision, so viral fractions should
  be isolated before sequencing'
- The goal is taxonomy/host/replication-cycle assignment only, with no need for gene-level
  functional annotation or AMG calls
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3174
tools:
- name: ViroProfiler
  role: Containerized Nextflow pipeline orchestrating gene prediction and functional
    annotation of viral contigs
  repo: https://github.com/deng-lab/viroprofiler
- name: Prodigal
  role: Predicts protein-coding genes on viral contigs to obtain nucleotide and protein
    sequences
  repo: https://github.com/hyattpd/Prodigal
- name: MMseqs2
  role: Clusters redundant proteins (identity 0.7, coverage 0.9) into a non-redundant
    gene catalog
  repo: https://github.com/soedinglab/MMseqs2
- name: EggNOG-mapper
  role: Annotates clustered proteins against EggNOG5 to retrieve KEGG, PFAM, and COG
    functional hits (EMAPPER process)
  repo: http://eggnog5.embl.de/
- name: VOGDB
  role: Database used to annotate viral proteins with virus-specific orthologous group
    hits
- name: DRAM-v
  role: Distills auxiliary metabolic gene (AMG) calls for viral contigs (DRAMV process)
  repo: https://github.com/WrightonLabCSU/DRAM
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1080/19490976.2023.2192522
    title: 'ViroProfiler: a containerized bioinformatics pipeline for viral metagenomic
      data analysis'
  source_content_hash: sha256:4bf4d096a354fe391a0cb108237338dc98c055da6da2f1968f1b317e5d781193
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/deng-lab/viroprofiler
  repo_url: https://github.com/deng-lab/viroprofiler
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
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# viral-gene-function-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates protein-coding genes on viral contigs by clustering predicted proteins with MMseqs2 and running eggNOG-mapper, VOGDB, and DRAM-v to produce a merged gene-by-annotation table with an auxiliary metabolic gene (AMG) flag, as implemented in the ViroProfiler pipeline.

## When to use

Use when input is a set of predicted viral contigs (from VirSorter2/VIBRANT etc.) and the goal is gene prediction, functional annotation (KEGG/PFAM/COG/VOG hits), and AMG identification.

## When NOT to use

- Input is raw reads or an unassembled metagenome rather than a curated set of predicted viral contigs — run viral discovery/assembly (e.g. metaSPAdes + VirSorter2/VIBRANT) first
- The metagenome has excessive environmental contamination: the article notes this can increase pipeline running time and lower precision, so viral fractions should be isolated before sequencing
- The goal is taxonomy/host/replication-cycle assignment only, with no need for gene-level functional annotation or AMG calls

## Inputs

- Viral contigs (nucleotide FASTA) predicted from viral metagenomes, e.g. by VirSorter2 or VIBRANT
- Prodigal-predicted nucleotide and protein FASTA sequences from those contigs

## Outputs

- Non-redundant protein gene catalog clustered with MMseqs2 (identity 0.7, coverage 0.9)
- eggNOG-mapper annotations against EggNOG5 (KEGG, PFAM, COG hits)
- VOGDB annotation hits
- DRAM-v AMG calls for viral contigs
- gene_annotation_table.tsv: tab-separated gene-by-annotation table with gene ID, predicted function, KEGG/PFAM/VOGDB hits, and boolean AMG flag

## How to apply

Predict protein-coding genes on the viral contigs with Prodigal to obtain nucleotide and protein sequences, then dereplicate the proteins by clustering with MMseqs2 at an identity threshold of 0.7 and coverage threshold of 0.9 to build a non-redundant gene catalog. Annotate the clustered proteins with eggNOG-mapper against EggNOG5 (EMAPPER step; in ViroProfiler configured with 22 CPUs and 300 GB memory) to retrieve KEGG, PFAM, and COG hits, and annotate the same proteins against VOGDB (4 CPUs, 40 GB). Run DRAM-v (DRAMV step; 1 CPU, 200 GB) on the viral contigs to distill AMG calls, since DRAM-v identifies auxiliary metabolic genes by comparing viral gene annotations against host-like microbial profiles. Finally, merge the eggNOG, VOGDB, and DRAM-v results by gene ID into a tabular gene-by-annotation table with columns for gene ID, predicted function, KEGG/PFAM/VOGDB hits, and a boolean AMG flag; the rationale for the clustering step is to reduce redundant annotation effort on near-identical proteins across viral contigs.

## Related tools

- **ViroProfiler** (Containerized Nextflow pipeline orchestrating gene prediction and functional annotation of viral contigs) — https://github.com/deng-lab/viroprofiler
- **Prodigal** (Predicts protein-coding genes on viral contigs to obtain nucleotide and protein sequences) — https://github.com/hyattpd/Prodigal
- **MMseqs2** (Clusters redundant proteins (identity 0.7, coverage 0.9) into a non-redundant gene catalog) — https://github.com/soedinglab/MMseqs2
- **EggNOG-mapper** (Annotates clustered proteins against EggNOG5 to retrieve KEGG, PFAM, and COG functional hits (EMAPPER process)) — http://eggnog5.embl.de/
- **VOGDB** (Database used to annotate viral proteins with virus-specific orthologous group hits)
- **DRAM-v** (Distills auxiliary metabolic gene (AMG) calls for viral contigs (DRAMV process)) — https://github.com/WrightonLabCSU/DRAM

## Examples

```
mmseqs easy-cluster proteins.faa nr clustered_tmp --min-seq-id 0.7 -c 0.9
```

## Evaluation signals

- gene_annotation_table.tsv exists and parses as a tab-separated table
- Table contains all required fields: gene ID, predicted function, KEGG/PFAM/VOGDB hits, and AMG flag
- Every gene ID in the table corresponds to a gene predicted on an input viral contig
- Non-redundant gene catalog was built with MMseqs2 clustering at identity 0.7 and coverage 0.9
- AMG flag values are boolean and traceable to DRAM-v distillation output

## Limitations

- ViroProfiler is specifically designed for classifying viruses in samples with isolated viruses; excessive environmental contamination increases running time and lowers precision, so viral fractions should be isolated before sequencing
- The annotation chain is resource-intensive: the EMAPPER step alone is configured for 22 CPUs / 300 GB memory and DRAMV for 200 GB, which may limit local execution
- DRAM2 (successor to the DRAM used here) is under active development and 'usage is at your own risk' per its README, so tool versions should be pinned for reproducibility

## Evidence

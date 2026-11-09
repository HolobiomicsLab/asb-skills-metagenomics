---
name: auxiliary-metabolic-gene-identification
description: Use when you have a set of predicted viral contigs (post VirSorter2/VIBRANT)
  and need to annotate their genes and flag which encode auxiliary metabolic genes
  (AMGs).
when_to_use_negative:
- Input is raw shotgun metagenomic reads rather than assembled, virus-identified contigs
  — ViroProfiler is specifically designed for classifying viruses in samples with
  isolated viruses, so isolate viral fractions and run viral identification/binning
  (VirSorter2/VIBRANT, vRhyme/phamb) first
- You need to annotate microbial (non-viral) MAGs rather than viral contigs — DRAM-v
  AMG distillation is tailored to viral sequences
- Excessive environmental contamination is present in the metagenome sequences, which
  the paper notes increases running time and lowers precision
edam_operation: http://edamontology.org/operation_1777
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: ViroProfiler
  role: Containerized Nextflow pipeline that orchestrates the gene prediction, clustering,
    annotation, and AMG-calling workflow
  repo: https://github.com/deng-lab/viroprofiler
- name: Prodigal
  role: Predicts protein-coding genes on input viral contigs to produce nucleotide
    and protein sequences
  repo: https://github.com/hyattpd/Prodigal/wiki/installation
- name: MMseqs2
  role: Clusters redundant predicted proteins (identity 0.7, coverage 0.9) to build
    the non-redundant gene catalog
  repo: https://github.com/soedinglab/MMseqs2
- name: eggNOG-mapper
  role: Annotates clustered proteins against EggNOG5 to retrieve KEGG, PFAM, and COG
    functional hits (EMAPPER step)
- name: VOGDB
  role: Virus orthologous groups database used as an additional annotation source
    for the clustered proteins
- name: DRAM-v
  role: Distills auxiliary metabolic gene (AMG) calls from the annotations of viral
    contigs (DRAMV step)
  repo: https://github.com/WrightonLabCSU/DRAM
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1080/19490976.2023.2192522
    title: 'ViroProfiler: a containerized bioinformatics pipeline for viral metagenomic
      data analysis'
  source_content_hash: sha256:e4cfa0ed07b9f1e5834b0d39ed9d1e178594bc71832696cb8f5a4af47b7a6d11
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
  - MMseqs2
  - Prodigal
  - VOGDB
  - ViroProfiler
  - eggNOG-mapper
  edam_operation: http://edamontology.org/operation_1777
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# auxiliary-metabolic-gene-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Functional annotation of protein-coding genes on viral contigs within ViroProfiler, combining eggNOG-mapper (EggNOG5), VOGDB, and DRAM-v to distill auxiliary metabolic gene (AMG) calls into a gene-by-annotation table.

## When to use

Use when you have a set of predicted viral contigs (post VirSorter2/VIBRANT) and need to annotate their genes and flag which encode auxiliary metabolic genes (AMGs).

## When NOT to use

- Input is raw shotgun metagenomic reads rather than assembled, virus-identified contigs — ViroProfiler is specifically designed for classifying viruses in samples with isolated viruses, so isolate viral fractions and run viral identification/binning (VirSorter2/VIBRANT, vRhyme/phamb) first
- You need to annotate microbial (non-viral) MAGs rather than viral contigs — DRAM-v AMG distillation is tailored to viral sequences
- Excessive environmental contamination is present in the metagenome sequences, which the paper notes increases running time and lowers precision

## Inputs

- Set of viral contigs (FASTA) predicted on viral metagenomic assemblies (e.g., via VirSorter2/VIBRANT/metaSPAdes)
- Protein and nucleotide sequences predicted from viral contigs with Prodigal

## Outputs

- Non-redundant protein gene catalog (MMseqs2 clusters, identity 0.7 / coverage 0.9)
- eggNOG-mapper annotations (KEGG, PFAM, COG hits) against EggNOG5
- VOGDB annotations
- DRAM-v AMG calls for the viral contigs
- gene_annotation_table.tsv: tab-separated table with gene ID, predicted function, KEGG/PFAM/VOGDB hits, and boolean AMG flag

## How to apply

Predict protein-coding genes on the viral contigs with Prodigal, yielding nucleotide and protein sequences. Cluster redundant proteins with MMseqs2 at an identity threshold of 0.7 and coverage threshold of 0.9 to build a non-redundant gene catalog. Annotate the clustered proteins with eggNOG-mapper against the EggNOG5 database (EMAPPER step, configured with 22 CPUs and 300 GB memory) to retrieve KEGG, PFAM, and COG hits, and against VOGDB (4 CPUs, 40 GB), then run DRAM-v (DRAMV step, 1 CPU, 200 GB) to distill AMG calls. Finally, merge the eggNOG, VOGDB, and DRAM-v results by gene ID into a tabular gene-by-annotation table with a boolean AMG flag.

## Related tools

- **ViroProfiler** (Containerized Nextflow pipeline that orchestrates the gene prediction, clustering, annotation, and AMG-calling workflow) — https://github.com/deng-lab/viroprofiler
- **Prodigal** (Predicts protein-coding genes on input viral contigs to produce nucleotide and protein sequences) — https://github.com/hyattpd/Prodigal/wiki/installation
- **MMseqs2** (Clusters redundant predicted proteins (identity 0.7, coverage 0.9) to build the non-redundant gene catalog) — https://github.com/soedinglab/MMseqs2
- **eggNOG-mapper** (Annotates clustered proteins against EggNOG5 to retrieve KEGG, PFAM, and COG functional hits (EMAPPER step))
- **VOGDB** (Virus orthologous groups database used as an additional annotation source for the clustered proteins)
- **DRAM-v** (Distills auxiliary metabolic gene (AMG) calls from the annotations of viral contigs (DRAMV step)) — https://github.com/WrightonLabCSU/DRAM

## Evaluation signals

- gene_annotation_table.tsv exists and parses as a tab-separated table
- The table contains the required fields: gene ID, predicted function, KEGG/PFAM/VOGDB hits, and a boolean AMG flag
- Every gene ID in the table corresponds to a gene predicted on an input viral contig
- The non-redundant gene catalog was built with MMseqs2 at identity threshold 0.7 and coverage threshold 0.9
- AMG flags originate from the DRAMV distillation step and merge cleanly with eggNOG and VOGDB hits by gene ID

## Limitations

- Excessive environmental contaminations, usually found in metagenome sequences, could increase the running time of the pipeline and result in lower precision
- ViroProfiler is specifically designed for classifying viruses in samples with isolated viruses — viral fractions should be isolated before sequencing for accurate estimation
- AMG calling depends on the bundled databases (EggNOG5, VOGDB) and the DRAMV distillation, so functions absent from these sources will not be annotated
- The DRAMV step is configured with only 1 CPU and 200 GB memory in the pipeline, which can make AMG distillation a resource-heavy bottleneck

## Evidence

- [intro] ViroProfiler takes advantage of the most recently developed viral metagenomic analysis tools and databases to improve the taxonomy and functional annotation of viruses and their gene products.: "ViroProfiler takes advantage of the most recently developed viral metagenomic analysis tools and databases to improve the taxonomy and functional annotation of viruses"
- [discussion] Moreover, it can detect viral replication cycles, predict hosts, and identify AMGs in viral sequences.: "Moreover, it can detect viral replication cycles, predict hosts, and identify AMGs in viral sequences."
- [methods] Gene function annotation | [DRAM-v](https://github.com/WrightonLabCSU/DRAM): "Gene function annotation | [DRAM-v](https://github.com/WrightonLabCSU/DRAM)"
- [methods] withName: VOGDB { cpus = 4 memory = "40 GB" }: "withName: VOGDB {
        cpus = 4
        memory = "40 GB""
- [abstract] ViroProfiler is freely available at https://github.com/deng-lab/viroprofiler.: "ViroProfiler is freely available at https://github.com/deng-lab/viroprofiler."

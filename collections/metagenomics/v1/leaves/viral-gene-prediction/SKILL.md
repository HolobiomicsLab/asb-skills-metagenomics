---
name: viral-gene-prediction
description: Use when you have a set of identified viral contigs (e.g., from VirSorter2/VIBRANT/DeepVirFinder/CheckV
  in ViroProfiler) and need gene predictions, KEGG/PFAM/VOGDB functional hits, or
  AMG identification.
when_to_use_negative:
- Input is raw metagenomic reads rather than identified viral contigs — run viral
  identification (VirSorter2/VIBRANT/DeepVirFinder) and CheckV first, as ViroProfiler
  is designed to work on viral contig sets.
- Samples contain isolated viruses only — the article notes ViroProfiler is 'specifically
  designed for classifying viruses in samples with isolated viruses', so bacterial
  (non-viral) gene annotation workflows are out of scope.
- Excessive environmental contamination is present in the metagenome sequences, which
  the article warns can increase running time and lower precision.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_0621
tools:
- name: ViroProfiler
  role: containerized Nextflow pipeline orchestrating gene prediction and annotation
    steps
  repo: https://github.com/deng-lab/viroprofiler
- name: DRAM-v
  role: distills auxiliary metabolic gene (AMG) calls for viral contigs (DRAMV step,
    1 CPU, 200 GB)
  repo: https://github.com/WrightonLabCSU/DRAM
- name: EggNOG-mapper
  role: annotates clustered proteins against EggNOG5 for KEGG, PFAM, and COG hits
    (EMAPPER step, 22 CPUs, 300 GB)
- name: VOGDB
  role: database for viral orthologous group annotation of clustered proteins (4 CPUs,
    40 GB)
- name: MMseqs2
  role: clusters redundant proteins at 0.7 identity / 0.9 coverage to build the non-redundant
    gene catalog
- name: Prodigal
  role: predicts protein-coding genes on input viral contigs, producing nucleotide
    and protein sequences
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1080/19490976.2023.2192522
    title: 'ViroProfiler: a containerized bioinformatics pipeline for viral metagenomic
      data analysis'
  source_content_hash: sha256:6489bbd527e0cc4a63e21eb7def85ac548382afb47207d504fb8cb66656c3584
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
  - BACPHLIP
  - DRAM-v
  - EggNOG
  - EggNOG-mapper
  - MMseqs2
  - Prodigal
  - Replidec
  - VOGDB
  - ViroProfiler
  - iPhoP
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# viral-gene-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predicts protein-coding genes on viral contigs and functionally annotates them with DRAM-v, eggNOG-mapper (EggNOG5), and VOGDB within ViroProfiler, distilling auxiliary metabolic gene (AMG) calls.

## When to use

Use when you have a set of identified viral contigs (e.g., from VirSorter2/VIBRANT/DeepVirFinder/CheckV in ViroProfiler) and need gene predictions, KEGG/PFAM/VOGDB functional hits, or AMG identification.

## When NOT to use

- Input is raw metagenomic reads rather than identified viral contigs — run viral identification (VirSorter2/VIBRANT/DeepVirFinder) and CheckV first, as ViroProfiler is designed to work on viral contig sets.
- Samples contain isolated viruses only — the article notes ViroProfiler is 'specifically designed for classifying viruses in samples with isolated viruses', so bacterial (non-viral) gene annotation workflows are out of scope.
- Excessive environmental contamination is present in the metagenome sequences, which the article warns can increase running time and lower precision.

## Inputs

- viral contig set (FASTA) identified upstream by VirSorter2, VIBRANT, DeepVirFinder and CheckV
- protein-coding gene predictions (nucleotide and protein FASTA from Prodigal)
- EggNOG5 database
- VOGDB database

## Outputs

- nucleotide and protein sequences of predicted genes
- non-redundant gene catalog (MMseqs2 clusters at 0.7 identity / 0.9 coverage)
- gene_annotation_table.tsv: tab-separated gene-by-annotation table with gene ID, predicted function, KEGG/PFAM/VOGDB hits, and boolean AMG flag

## How to apply

Take the viral contig set produced upstream in ViroProfiler and predict protein-coding genes with Prodigal, yielding nucleotide and protein sequences. Cluster redundant proteins with MMseqs2 at an identity threshold of 0.7 and coverage threshold of 0.9 to build a non-redundant gene catalog. Annotate the clustered proteins with eggNOG-mapper against EggNOG5 (EMAPPER step, 22 CPUs, 300 GB) for KEGG, PFAM, and COG hits, and against VOGDB (4 CPUs, 40 GB); run DRAM-v (DRAMV step, 1 CPU, 200 GB) to distill AMG calls. Merge the eggNOG, VOGDB, and DRAM-v results by gene ID into a gene-by-annotation table with gene ID, predicted function, KEGG/PFAM/VOGDB hits, and a boolean AMG flag. Rationale: ViroProfiler consolidates recently developed tools and databases to improve functional annotation of viruses and their gene products, with containerization ensuring reproducibility.

## Related tools

- **ViroProfiler** (containerized Nextflow pipeline orchestrating gene prediction and annotation steps) — https://github.com/deng-lab/viroprofiler
- **DRAM-v** (distills auxiliary metabolic gene (AMG) calls for viral contigs (DRAMV step, 1 CPU, 200 GB)) — https://github.com/WrightonLabCSU/DRAM
- **EggNOG-mapper** (annotates clustered proteins against EggNOG5 for KEGG, PFAM, and COG hits (EMAPPER step, 22 CPUs, 300 GB))
- **VOGDB** (database for viral orthologous group annotation of clustered proteins (4 CPUs, 40 GB))
- **MMseqs2** (clusters redundant proteins at 0.7 identity / 0.9 coverage to build the non-redundant gene catalog)
- **Prodigal** (predicts protein-coding genes on input viral contigs, producing nucleotide and protein sequences)

## Examples

```
nextflow run deng-lab/viroprofiler --input viral_contigs.fasta -profile docker -c custom.config
```

## Evaluation signals

- gene_annotation_table.tsv exists and parses as a tab-separated table
- The table contains the required fields: gene ID, predicted function, KEGG/PFAM/VOGDB hits, and AMG flag
- Every gene ID in the table corresponds to a gene predicted on an input viral contig
- Pipeline processes EMAPPER (22 CPUs / 300 GB), VOGDB (4 CPUs / 40 GB), and DRAMV (1 CPU / 200 GB) completed without error
- The non-redundant gene catalog reflects MMseqs2 clustering at identity 0.7 and coverage 0.9 (no redundant proteins remain)

## Limitations

- Excessive environmental contaminations, usually found in metagenome sequences, could increase the running time of the pipeline and result in lower precision; viral fractions should be isolated before sequencing.
- DRAM2 (DRAM successor) is 'currently under active development and usage is at your own risk' per its README.
- Resource requirements are substantial (EMAPPER alone is configured with 22 CPUs and 300 GB memory), which may constrain smaller compute environments.

## Evidence

- [methods] Workflow: gene prediction and annotation: [evidence span withheld — anchor status: not_found]
- [methods] Workflow: gene clustering: [evidence span withheld — anchor status: not_found]
- [methods] Workflow: annotation and AMG distillation: [evidence span withheld — anchor status: not_found]
- [readme] DRAM2 caveat: [evidence span withheld — anchor status: not_found]

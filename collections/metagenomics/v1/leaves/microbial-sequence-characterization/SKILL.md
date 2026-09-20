---
name: microbial-sequence-characterization
description: Use when host-filtered paired-end FASTQ files from RNA-seq data must
  be assigned taxonomically to detect microbial (bacterial, archaeal, viral) content,
  e.g. in metagenomic analysis of SSc skin RNA-seq samples like SRR5074528.
when_to_use_negative:
- When raw RNA-seq reads have not yet been host-filtered — the classification stage
  expects exogenous reads only; run the filtering/classification step (run.FilterClassify)
  pipeline instead of kraken2 alone on unfiltered input
- When the goal is differential gene expression or host transcriptome analysis rather
  than microbial taxonomic characterization
- When the sample content is expected to include non-bacterial/archaeal/viral taxa
  (e.g. fungal or parasitic genomes), since the K2DB database used here is constructed
  only from bacterial, archaeal, and viral genomes
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: kraken2
  role: Taxonomic classifier applied to host-filtered exogenous reads against the
    bacterial/archaeal/viral K2DB database
- name: nf-rnaSeqMetagen
  role: Nextflow pipeline orchestrating host filtering and kraken2 classification
    (run.FilterClassify mode)
  repo: https://github.com/phelelani/nf-rnaSeqMetagen
- name: Nextflow
  role: Workflow management system executing the nf-rnaSeqMetagen pipeline steps
  repo: http://nextflow.io/
- name: Singularity
  role: Containerization of pipeline applications to ensure reproducibility and portability
  repo: http://singularity.lbl.gov/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.medmic.2020.100011
    title: 'nf-rnaSeqMetagen: A nextflow metagenomics pipeline for identifying and
      characterizing microbial sequences from RNA-seq data'
  source_content_hash: sha256:594faee05516e7ec9ff41db1642a5899a77d1f5e36787649eab01a01de521423
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/phelelani/nf-rnaSeqMetagen
  repo_url: https://github.com/phelelani/nf-rnaSeqMetagen
  tools:
  - Nextflow
  - Singularity
  - kraken2
  - nf-rnaSeqMetagen
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# microbial-sequence-characterization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Taxonomically characterize exogenous (non-host) RNA-seq reads remaining after host filtering by running kraken2 against a database built from bacterial, archaeal, and viral genomes within the nf-rnaSeqMetagen Nextflow pipeline, producing per-read classification files and per-sample taxon reports.

## When to use

Use when host-filtered paired-end FASTQ files from RNA-seq data must be assigned taxonomically to detect microbial (bacterial, archaeal, viral) content, e.g. in metagenomic analysis of SSc skin RNA-seq samples like SRR5074528.

## When NOT to use

- When raw RNA-seq reads have not yet been host-filtered — the classification stage expects exogenous reads only; run the filtering/classification step (run.FilterClassify) pipeline instead of kraken2 alone on unfiltered input
- When the goal is differential gene expression or host transcriptome analysis rather than microbial taxonomic characterization
- When the sample content is expected to include non-bacterial/archaeal/viral taxa (e.g. fungal or parasitic genomes), since the K2DB database used here is constructed only from bacterial, archaeal, and viral genomes

## Inputs

- Host-filtered (exogenous) paired-end FASTQ files per sample (fastq | fastq.gz | fastq.bz2 | fq | fq.gz | fq.bz2)
- Prebuilt kraken2 database folder (e.g. $PWD/K2DB) constructed from bacterial, archaeal, and viral genomes
- User configuration file (main.config) specifying --data, --db, --out, and --mode run.FilterClassify

## Outputs

- Per-read kraken2 classification output file in each per-sample analysis directory
- Kraken2 report of detected taxa per sample
- Downstream Krona chart of detected organisms per sample (e.g. SRR5074528)

## How to apply

Take the host-filtered (exogenous) paired-end FASTQ files produced by the pipeline's host-filtering step as input to the classification stage. Run the nf-rnaSeqMetagen pipeline in run.FilterClassify mode (e.g. with the slurm profile and Singularity containers) via Nextflow, pointing --db at the prebuilt kraken2 database (K2DB) constructed from bacterial, archaeal, and viral genomes (built beforehand via prep.KrakenDB). The pipeline emits, into each per-sample analysis directory under the output folder, a per-read kraken2 classification output file and a kraken2 report of detected taxa. Downstream, feed the reports into Krona charts of detected organisms per sample (e.g. SRR5074528). Rationale: after the pipeline filters out reads belonging to the host organism, kraken2 taxonomic binning of the remaining reads is the method used to identify and characterize microbial sequences in RNA-seq data.

## Related tools

- **kraken2** (Taxonomic classifier applied to host-filtered exogenous reads against the bacterial/archaeal/viral K2DB database)
- **nf-rnaSeqMetagen** (Nextflow pipeline orchestrating host filtering and kraken2 classification (run.FilterClassify mode)) — https://github.com/phelelani/nf-rnaSeqMetagen
- **Nextflow** (Workflow management system executing the nf-rnaSeqMetagen pipeline steps) — http://nextflow.io/
- **Singularity** (Containerization of pipeline applications to ensure reproducibility and portability) — http://singularity.lbl.gov/

## Examples

```
nextflow run nf-rnaSeqMetagen -profile slurm --mode run.FilterClassify -c myparams.config
```

## Evaluation signals

- Each sample's analysis directory under the output folder contains both a kraken2 per-read classification file and a kraken2 taxon report
- Reported taxa in the kraken2 report are consistent with the downstream Krona chart of detected organisms for the same sample (e.g. SRR5074528)
- The kraken2 database used was built with prep.KrakenDB (K2DB) from bacterial, archaeal, and viral genomes, not an arbitrary database
- Input FASTQ files passed to classification are the host-filtered exogenous reads, not the original raw reads
- Pipeline ran in run.FilterClassify mode with the intended profile (e.g. slurm) and Singularity containers pulled via prep.Containers

## Limitations

- The kraken2 database (K2DB) covers only bacterial, archaeal, and viral genomes, so taxa outside these domains will not be characterized
- Characterization depends on the preceding host-filtering step: reads misclassified as host will be absent from microbial results
- Containerized execution requires Singularity images downloaded beforehand (prep.Containers); missing containers will stall the run

## Evidence

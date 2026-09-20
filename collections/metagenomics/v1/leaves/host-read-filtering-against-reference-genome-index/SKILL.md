---
name: host-read-filtering-against-reference-genome-index
description: Use when RNA-seq FASTQ reads from a host organism must be separated from
  microbial (bacterial, archaeal, viral) content before taxonomic classification or
  metagenomic reporting.
when_to_use_negative:
- Input is not host-derived RNA-seq data (e.g. shotgun metagenomic reads with no host
  background) — host filtering adds no value
- No reference genome FASTA/GTF or indexes are available for the host organism and
  they cannot be generated
- The goal is de novo host transcriptome analysis, where removing host reads would
  discard the signal of interest
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0091
tools:
- name: nf-rnaSeqMetagen
  role: Nextflow pipeline that orchestrates host-read filtering and exogenous-read
    classification (run.FilterClassify mode)
  repo: https://github.com/phelelani/nf-rnaSeqMetagen
- name: Nextflow
  role: Workflow management system executing the filtering/classification steps
  repo: https://github.com/phelelani/nf-rnaSeqMetagen
- name: Singularity
  role: Containerization of all filtering and classification applications for portability
    and reproducibility
- name: STAR
  role: Provides the reference genome index used to filter host reads from the RNA-seq
    input
- name: kraken2
  role: Classifies the remaining exogenous reads against a bacterial/archaeal/viral
    database
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.medmic.2020.100011
    title: 'nf-rnaSeqMetagen: A nextflow metagenomics pipeline for identifying and
      characterizing microbial sequences from RNA-seq data'
  source_content_hash: sha256:62a51be9b4e206f17e8117e4d31c989bb327c20b44ec75ac98aed590424f84fe
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
  - STAR
  - Singularity
  - kraken2
  - nf-rnaSeqMetagen
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# host-read-filtering-against-reference-genome-index

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Filters host-organism reads out of RNA-seq FASTQ input by aligning against a reference genome index, then classifies the remaining exogenous reads with kraken2, as implemented in the nf-rnaSeqMetagen Nextflow/Singularity pipeline.

## When to use

Use when RNA-seq FASTQ reads from a host organism must be separated from microbial (bacterial, archaeal, viral) content before taxonomic classification or metagenomic reporting.

## When NOT to use

- Input is not host-derived RNA-seq data (e.g. shotgun metagenomic reads with no host background) — host filtering adds no value
- No reference genome FASTA/GTF or indexes are available for the host organism and they cannot be generated
- The goal is de novo host transcriptome analysis, where removing host reads would discard the signal of interest

## Inputs

- Paired-end (or single-end) RNA-seq FASTQ files, e.g. SRR5074528_1/2 from the H3ABioNet RNA-seq test dataset
- Host reference genome FASTA (.fa/.fasta) and annotation GTF file, e.g. Mus_musculus.GRCm38 (Ensembl release-68)
- Pre-built STAR/bowtie2 reference genome indexes
- Kraken2 database (e.g. K2DB built from bacterial, archaeal, and viral genomes)

## Outputs

- Filtered (host-removed) FASTQ reads for downstream classification
- kraken2 taxonomic classification of exogenous reads
- Interactive Krona HTML report of raw-read taxonomy (e.g. SRR5074528_reads.html) and assembled-read taxonomy (SRR5074528_fasta.html) under <output_directory>/SRR5074528

## How to apply

Obtain the host reference genome FASTA and GTF annotation (e.g. Mus_musculus.GRCm38 from Ensembl release-68) and build STAR (and optionally bowtie2) indexes. Prepare the Singularity containers (prep.Containers) and optionally build a kraken2 database from bacterial, archaeal, and viral genomes (prep.KrakenDB --db $PWD/K2DB). Run the run.FilterClassify mode via `nextflow run nf-rnaSeqMetagen -profile slurm --mode run.FilterClassify -c myparams.config`, which containerizes the host-read filtering against the STAR index and then classifies remaining exogenous reads with kraken2. The rationale is that host reads dominate RNA-seq data, so removing them reduces false-positive microbial calls and shrinks the search space for kraken2 classification of exogenous reads.

## Related tools

- **nf-rnaSeqMetagen** (Nextflow pipeline that orchestrates host-read filtering and exogenous-read classification (run.FilterClassify mode)) — https://github.com/phelelani/nf-rnaSeqMetagen
- **Nextflow** (Workflow management system executing the filtering/classification steps) — https://github.com/phelelani/nf-rnaSeqMetagen
- **Singularity** (Containerization of all filtering and classification applications for portability and reproducibility)
- **STAR** (Provides the reference genome index used to filter host reads from the RNA-seq input)
- **kraken2** (Classifies the remaining exogenous reads against a bacterial/archaeal/viral database)

## Examples

```
nextflow run nf-rnaSeqMetagen -profile slurm --mode run.FilterClassify -c myparams.config
```

## Evaluation signals

- Both Krona HTML outputs (SRR5074528_reads.html and SRR5074528_fasta.html) exist under examples/output/SRR5074528/ and open as self-contained interactive hierarchical taxonomic charts
- Filtered output FASTQ files contain substantially fewer reads than the input, consistent with removal of host-origin reads
- kraken2 classification runs only after filtering, on the remaining exogenous reads, against the prepared K2DB database
- The pipeline completes under the chosen executor profile (e.g. slurm) with Singularity-containerized steps without failures
- Reference index and Kraken2 database prep modes (prep.GenomeIndexes, prep.KrakenDB) completed successfully before run.FilterClassify was invoked

## Limitations

- Requires installed software (Nextflow, Singularity), a reference genome, annotation file, and pre-built STAR/bowtie2 indexes as dependencies
- Resource-capped by default at max_memory 200.GB, max_cpus 24, max_time 24.h; runs must fit within these limits or be overridden
- Filtering fidelity depends on the host reference: reads from regions absent from Mus_musculus.GRCm38 may survive filtering and be misclassified as microbial

## Evidence

- [abstract] Host read filtering rationale: "The pipeline takes RNA-seq reads as input and filters out reads belonging to the host organism"
- [abstract] Exogenous read classification: "The remaining exogenous reads are then characterized using the kraken2 database constructed from bacterial, archaeal, and viral genomes"
- [methods] run.FilterClassify invocation: "nextflow run nf-rnaSeqMetagen -profile slurm --mode run.FilterClassify -c myparams.config"
- [methods] Reference genome source: "wget -c -O reference/genome.fa.gz ftp://ftp.ensembl.org/pub/release-68/fasta/mus_musculus/dna/Mus_musculus.GRCm38.68.dna.toplevel.fa.gz"
- [readme] Index dependency: "Reference genome indexes (`bowtie2` & `STAR` - see *1.3.* below on how to generate the indexes)"

---
name: host-read-filtering-rnaseq
description: Use when RNA-seq FASTQ data must be stripped of host-organism reads before
  microbial taxonomic characterization, e.g. paired-end SSc skin RNA-seq against Mus_musculus.GRCm38.
when_to_use_negative:
- Input is already filtered exogenous FASTQ or a taxonomy table rather than raw RNA-seq
  reads
- Reads are metagenomic shotgun data with no host component to remove — the host alignment
  stage adds no value
- No host reference genome/annotation is available, since STAR/bowtie2 indexes cannot
  be generated
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3308
tools:
- name: nf-rnaSeqMetagen
  role: Nextflow pipeline orchestrating host-read filtering and classification; run
    via prep.* and run.FilterClassify modes
  repo: https://github.com/phelelani/nf-rnaSeqMetagen
- name: Nextflow
  role: Workflow management system used to orchestrate the filtering steps
- name: Singularity
  role: Containerization of all applications; containers fetched with prep.Containers
- name: STAR
  role: Host-genome index generation (prep.GenomeIndexes) and alignment used to remove
    host RNA-seq reads
- name: bowtie2
  role: Second alignment index used alongside STAR for host-read filtering
- name: kraken2
  role: Classifies the exogenous reads remaining after host filtering using a bacterial,
    archaeal and viral genome database
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1016/j.medmic.2020.100011
    title: 'nf-rnaSeqMetagen: A nextflow metagenomics pipeline for identifying and
      characterizing microbial sequences from RNA-seq data'
  source_content_hash: sha256:c549746b541a13b954bcb6dd83af173cbfd2a4f9e4742c5ed568568f9675f41e
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
  - bowtie2
  - kraken2
  - nf-rnaSeqMetagen
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# host-read-filtering-rnaseq

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Filters host-organism reads out of RNA-seq FASTQ input using the nf-rnaSeqMetagen pipeline, which aligns reads against STAR and bowtie2 indexes of the host reference genome so that only exogenous reads pass onward to kraken2 taxonomic classification.

## When to use

Use when RNA-seq FASTQ data must be stripped of host-organism reads before microbial taxonomic characterization, e.g. paired-end SSc skin RNA-seq against Mus_musculus.GRCm38.

## When NOT to use

- Input is already filtered exogenous FASTQ or a taxonomy table rather than raw RNA-seq reads
- Reads are metagenomic shotgun data with no host component to remove — the host alignment stage adds no value
- No host reference genome/annotation is available, since STAR/bowtie2 indexes cannot be generated

## Inputs

- Paired-end or single-end RNA-seq FASTQ files (fastq | fastq.gz | fastq.bz2 | fq | fq.gz | fq.bz2)
- Host reference genome FASTA (e.g. Mus_musculus.GRCm38.68.dna.toplevel.fa from Ensembl release-68)
- Host genome annotation GTF (e.g. Mus_musculus.GRCm38.68.gtf)
- Nextflow params config file setting --data, --out, --genome, --genes and --db paths
- kraken2 database folder built from bacterial, archaeal and viral genomes

## Outputs

- Per-sample analysis directory under <output_directory>/<sample_id> containing non-host (filtered) FASTQ files
- MultiQC report with alignment summary statistics showing host reads removed from raw input
- kraken2/Krona classification reports for downstream taxonomic tallying

## How to apply

Pull the pipeline with 'nextflow pull https://github.com/phelelani/nf-rnaSeqMetagen' and download the host reference genome and annotation, e.g. Mus_musculus.GRCm38 FASTA and genes.gtf from Ensembl release-68. Pull Singularity containers ('--mode prep.Containers') and generate the reference indexes with '--mode prep.GenomeIndexes --genome reference/genome.fa --genes reference/genes.gtf' (bowtie2 indexes are also built for filtering). Run the filtering/classification stage with 'nextflow run nf-rnaSeqMetagen -profile slurm --mode run.FilterClassify -c myparams.config', where the config file sets params.data (FASTQ folder), params.out, params.genome, params.genes and params.db; filtering is performed by aligning input reads against both the bowtie2 and STAR host-genome indexes, and host reads are removed so exogenous reads pass onward for kraken2 taxonomic characterization. Per-sample analysis directories contain the non-host (filtered) FASTQ files, and the MultiQC report summarizes raw vs filtered read counts.

## Related tools

- **nf-rnaSeqMetagen** (Nextflow pipeline orchestrating host-read filtering and classification; run via prep.* and run.FilterClassify modes) — https://github.com/phelelani/nf-rnaSeqMetagen
- **Nextflow** (Workflow management system used to orchestrate the filtering steps)
- **Singularity** (Containerization of all applications; containers fetched with prep.Containers)
- **STAR** (Host-genome index generation (prep.GenomeIndexes) and alignment used to remove host RNA-seq reads)
- **bowtie2** (Second alignment index used alongside STAR for host-read filtering)
- **kraken2** (Classifies the exogenous reads remaining after host filtering using a bacterial, archaeal and viral genome database)

## Examples

```
nextflow run nf-rnaSeqMetagen -profile slurm --mode run.FilterClassify -c myparams.config
```

## Evaluation signals

- Per-sample output directory exists under <output_directory>/<sample_id> (e.g. SRR5074528) containing the filtered (non-host) FASTQ files
- MultiQC report contains alignment summary statistics showing host reads were removed from the raw input
- STAR and bowtie2 reference indexes were built beforehand (prep.GenomeIndexes run without error against the .fa and .gtf files)
- kraken2 reports exist for each sample and only contain exogenous (non-host) assigned reads
- For the SSc use case, tallying kraken2 reports across all 25 samples recovers reads assigned to Arthrobacter, Bacillus, Brachybacterium, Dietzia and Pseudarthrobacter

## Limitations

- Only host reads matching the supplied reference genome are removed — organisms absent from the STAR/bowtie2 indexes cannot be filtered
- The kraken2 database is limited to bacterial, archaeal and viral genomes, so eukaryotic exogenous contamination is not classified
- Execution requires pre-installed Nextflow, Singularity, and pre-built reference genome/annotation and indexes; available schedulers are limited to 'standard' and 'slurm' profiles
- Discussion content is absent from the source, so no documented limitations or reproducibility gaps are extractable beyond the dependencies note

## Evidence

- [abstract] Host filtering via STAR/bowtie2 indexes: "The pipeline takes RNA-seq reads as input and filters out reads belonging to the host organism"
- [abstract] kraken2 characterization of remaining reads: "The remaining exogenous reads are then characterized using the kraken2 database constructed from bacterial, archaeal, and viral genomes"
- [methods] Run filtering/classification command: "nextflow run nf-rnaSeqMetagen -profile slurm --mode run.FilterClassify -c myparams.config"
- [methods] Generate STAR genome indexes: "nextflow run nf-rnaSeqMetagen -profile slurm --mode prep.GenomeIndexes --genome "$PWD/reference/genome.fa" --genes "$PWD/reference/genes.gtf""
- [readme] Reference genome indexes required: "Reference genome indexes (`bowtie2` & `STAR` - see *1.3.* below on how to generate the indexes)."

---
name: nextflow-pipeline-execution
description: Use when a multi-step bioinformatics workflow (e.g. RNA-seq host read
  filtering plus metagenomic classification) must be run reproducibly with Nextflow,
  Singularity containers, and a scheduler such as SLURM.
when_to_use_negative:
- Input is already classified taxonomy output or a feature table rather than raw RNA-seq
  FASTQ reads needing host filtering
- Only a single preparatory resource changed (e.g. rebuilding indexes) — rerun.FilterClassify
  would be unnecessary; rerun only the relevant prep mode
- Execution environment lacks Nextflow, Singularity, or a scheduler profile the pipeline
  supports (standard/slurm)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0769
- http://edamontology.org/topic_3174
tools:
- name: nf-rnaSeqMetagen
  role: The Nextflow workflow being executed; modes prep.Containers, prep.GenomeIndexes,
    prep.KrakenDB and run.FilterClassify orchestrate filtering and classification
  repo: https://github.com/phelelani/nf-rnaSeqMetagen
- name: Nextflow
  role: Workflow management system orchestrating pipeline applications and scheduling
    via -profile standard/slurm
  repo: http://nextflow.io/
- name: Singularity
  role: Containerization of all pipeline applications to enable parallelization, portability
    and reproducibility; images fetched via --mode prep.Containers
  repo: http://singularity.lbl.gov/
- name: STAR
  role: Builds reference genome indexes (prep.GenomeIndexes) used to filter host reads
    during run.FilterClassify
- name: bowtie2
  role: Second reference genome index type (prep.BowtieIndex) used alongside STAR
    for host read filtering
- name: kraken2
  role: Classifies exogenous reads against the K2DB database produced by --mode prep.KrakenDB
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_003
  - task_001
  source_papers:
  - doi: 10.1016/j.medmic.2020.100011
    title: 'nf-rnaSeqMetagen: A nextflow metagenomics pipeline for identifying and
      characterizing microbial sequences from RNA-seq data'
  source_content_hash: sha256:bfd31042c8df27e06eacd4f67285d9971e442d9d13adcac81ef8ccc8d590f2e3
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0769
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# nextflow-pipeline-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Executing containerized Nextflow workflows, exemplified by nf-rnaSeqMetagen, which orchestrates Singularity-containerized STAR/bowtie2 host filtering and kraken2 taxonomic classification of RNA-seq reads across slurm or standard execution profiles.

## When to use

Use when a multi-step bioinformatics workflow (e.g. RNA-seq host read filtering plus metagenomic classification) must be run reproducibly with Nextflow, Singularity containers, and a scheduler such as SLURM.

## When NOT to use

- Input is already classified taxonomy output or a feature table rather than raw RNA-seq FASTQ reads needing host filtering
- Only a single preparatory resource changed (e.g. rebuilding indexes) — rerun.FilterClassify would be unnecessary; rerun only the relevant prep mode
- Execution environment lacks Nextflow, Singularity, or a scheduler profile the pipeline supports (standard/slurm)

## Inputs

- Paired-end (or single-end) FASTQ files (fastq | fastq.gz | fastq.bz2 | fq | fq.gz | fq.bz2) in a data folder
- Reference genome FASTA (.fasta/.fa/.fna) e.g. Mus_musculus.GRCm38.68.dna.toplevel.fa from Ensembl release-68
- Genome annotation GTF file e.g. Mus_musculus.GRCm38.68.gtf
- User params config file (myparams.config) setting data, out, genome, genes and db paths
- Kraken2 database folder (e.g. $PWD/K2DB) built from bacterial, archaeal and viral genomes

## Outputs

- Per-sample analysis directory under <output_directory>/<sample> (e.g. SRR5074528) with host-filtered (exogenous) FASTQ files
- Per-read kraken2 classification file and kraken2 taxon report per sample
- Interactive Krona HTML charts (e.g. SRR5074528_reads.html and SRR5074528_fasta.html) per sample
- MultiQC report with raw vs filtered read-count alignment statistics
- STAR/bowtie2 genome indexes and Singularity images from the prep modes

## How to apply

Pull the pipeline with `nextflow pull https://github.com/phelelani/nf-rnaSeqMetagen` so it lands in $HOME/.nextflow/assets, then prepare resources in staged modes: `--mode prep.Containers` downloads the Singularity images (kraken2, multiQC, star, trinity, upset recipes exist in the containers/ folder), `--mode prep.GenomeIndexes` builds STAR (and bowtie2) indexes from a reference FASTA plus GTF annotation (e.g. Mus_musculus.GRCm38 from Ensembl release-68), and `--mode prep.KrakenDB --db $PWD/K2DB` builds the kraken2 database from bacterial, archaeal, and viral genomes. Run the analysis with `--mode run.FilterClassify -c myparams.config`, a config setting params.data (FASTQ folder), params.out, params.genome, params.genes and params.db, selecting `-profile slurm` (or `standard` for local execution) and `--pairedEnd`/`--singleEnd` as appropriate. Rationale: mode-based invocation separates idempotent preparation (indexes, containers, database) from per-sample analysis, and mandatory arguments are validated via `nextflow run nf-rnaSeqMetagen --help` (profile, mode, data, genome, genes, db) with optional --out, --max_memory (default 200.GB), --max_cpus (default 24) and --max_time (default 24.h).

## Related tools

- **nf-rnaSeqMetagen** (The Nextflow workflow being executed; modes prep.Containers, prep.GenomeIndexes, prep.KrakenDB and run.FilterClassify orchestrate filtering and classification) — https://github.com/phelelani/nf-rnaSeqMetagen
- **Nextflow** (Workflow management system orchestrating pipeline applications and scheduling via -profile standard/slurm) — http://nextflow.io/
- **Singularity** (Containerization of all pipeline applications to enable parallelization, portability and reproducibility; images fetched via --mode prep.Containers) — http://singularity.lbl.gov/
- **STAR** (Builds reference genome indexes (prep.GenomeIndexes) used to filter host reads during run.FilterClassify)
- **bowtie2** (Second reference genome index type (prep.BowtieIndex) used alongside STAR for host read filtering)
- **kraken2** (Classifies exogenous reads against the K2DB database produced by --mode prep.KrakenDB)

## Examples

```
nextflow run nf-rnaSeqMetagen -profile slurm --mode run.FilterClassify -c myparams.config
```

## Evaluation signals

- Per-sample output directory exists under <output_directory>/SRR5074528 (or the corresponding sample ID) after run.FilterClassify
- Each sample analysis directory contains both the kraken2 per-read classification file and the kraken2 taxon report
- Krona HTML charts (SRR5074528_reads.html and SRR5074528_fasta.html) exist under examples/output/SRR5074528/ and open as self-contained interactive hierarchical taxonomic charts
- MultiQC report contains alignment summary statistics showing host reads removed relative to raw input read counts
- Help output `nextflow run nf-rnaSeqMetagen --help` reports the expected mandatory arguments (profile, mode, data, genome, genes, db) before any run is attempted

## Limitations

- Pipeline requires external software dependencies: Nextflow, Singularity, plus a reference genome, annotation and generated indexes; detailed documentation lives at https://phelelani.github.io/nf-rnaSeqMetagen/
- Only standard (local) and slurm executor profiles are offered; other schedulers are not covered in the README
- nextflow.config is pipeline-owned ('DO NOT EDIT'); user settings must go in main.config/myparams.config, and misplacing parameters there is a failure mode

## Evidence

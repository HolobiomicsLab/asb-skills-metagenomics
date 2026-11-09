---
name: metagenomic-taxonomy-profiling-of-rna-seq-reads
description: 'Use when RNA-seq FASTQ reads must be screened for microbial (bacterial,
  archaeal, viral) content: host reads are filtered against a STAR reference index
  and the exogenous remainder is classified taxonomically with kraken2.'
when_to_use_negative:
- Input is not RNA-seq reads but an already-assembled contig set or an existing taxonomic
  abundance table — kraken2 read-level classification via this pipeline is unnecessary.
- The target microbial community is not bacterial, archaeal, or viral (e.g., fungal/eukaryotic
  profiling), since the documented K2DB is constructed from bacterial, archaeal, and
  viral genomes only.
- No host reference genome/index is available and host filtering cannot be configured,
  making the FilterClassify step inapplicable.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: nf-rnaSeqMetagen
  role: Nextflow pipeline orchestrating host-read filtering and kraken2 taxonomic
    classification of exogenous RNA-seq reads
  repo: https://github.com/phelelani/nf-rnaSeqMetagen
- name: kraken2
  role: Classifies the remaining exogenous reads against a database constructed from
    bacterial, archaeal, and viral genomes
- name: STAR
  role: Host genome index used to filter out host organism reads from the RNA-seq
    input (e.g., Mus_musculus.GRCm38 index)
- name: Nextflow
  role: Workflow management system executing the prep.KrakenDB and run.FilterClassify
    modes
- name: Singularity
  role: Containerization of all pipeline applications for portability and reproducibility
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1016/j.medmic.2020.100011
    title: 'nf-rnaSeqMetagen: A nextflow metagenomics pipeline for identifying and
      characterizing microbial sequences from RNA-seq data'
  source_content_hash: sha256:77113f38c70347cace1b899acea047f2330f693fc55a1c97ca3940e7b2e8f215
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
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# metagenomic-taxonomy-profiling-of-rna-seq-reads

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Profile the taxonomic composition of RNA-seq samples by filtering out host reads and classifying the remaining exogenous reads with kraken2, as implemented in the nf-rnaSeqMetagen Nextflow pipeline with Singularity-containerized steps.

## When to use

Use when RNA-seq FASTQ reads must be screened for microbial (bacterial, archaeal, viral) content: host reads are filtered against a STAR reference index and the exogenous remainder is classified taxonomically with kraken2.

## When NOT to use

- Input is not RNA-seq reads but an already-assembled contig set or an existing taxonomic abundance table — kraken2 read-level classification via this pipeline is unnecessary.
- The target microbial community is not bacterial, archaeal, or viral (e.g., fungal/eukaryotic profiling), since the documented K2DB is constructed from bacterial, archaeal, and viral genomes only.
- No host reference genome/index is available and host filtering cannot be configured, making the FilterClassify step inapplicable.

## Inputs

- Paired-end RNA-seq FASTQ files (fastq | fastq.gz | fastq.bz2 | fq | fq.gz | fq.bz2), e.g., the H3ABioNet RNA-seq test dataset for SRR5074528
- Host reference genome FASTA and GTF annotation (e.g., Mus_musculus.GRCm38, Ensembl release-68)
- Kraken2 database built from bacterial, archaeal, and viral genomes (K2DB)

## Outputs

- Per-sample kraken2 per-read classification file and kraken2 report of detected taxa in <output_directory>/<sample>
- Interactive Krona HTML taxonomic charts, e.g., SRR5074528_reads.html (raw reads) and SRR5074528_fasta.html (assembled reads)
- Host-filtered (exogenous) paired-end FASTQ files passed to classification

## How to apply

Prepare a kraken2 database restricted to bacterial, archaeal, and viral genomes via the pipeline's prep.KrakenDB mode (nextflow run nf-rnaSeqMetagen -profile slurm --mode prep.KrakenDB --db $PWD/K2DB), and build a STAR index of the host reference genome (e.g., Mus_musculus.GRCm38, Ensembl release-68) with prep.GenomeIndexes. Then run the combined filtering and classification step in run.FilterClassify mode with a user config file (--mode run.FilterClassify -c myparams.config) and the slurm profile, using Singularity-containerized processes. Host reads are first removed against the STAR index, and the surviving exogenous paired-end FASTQ reads are classified per-read against the K2DB database. Per-sample outputs (e.g., under <output_directory>/SRR5074528) include the kraken2 per-read classification file, a kraken2 report of detected taxa, and interactive Krona HTML charts (e.g., SRR5074528_reads.html for raw reads and SRR5074528_fasta.html for assembled reads). The rationale is that RNA-seq data contains a mixture of host and potential microbial reads, so host subtraction followed by kraken2 classification against a curated microbial database yields a defensible taxonomic profile.

## Related tools

- **nf-rnaSeqMetagen** (Nextflow pipeline orchestrating host-read filtering and kraken2 taxonomic classification of exogenous RNA-seq reads) — https://github.com/phelelani/nf-rnaSeqMetagen
- **kraken2** (Classifies the remaining exogenous reads against a database constructed from bacterial, archaeal, and viral genomes)
- **STAR** (Host genome index used to filter out host organism reads from the RNA-seq input (e.g., Mus_musculus.GRCm38 index))
- **Nextflow** (Workflow management system executing the prep.KrakenDB and run.FilterClassify modes)
- **Singularity** (Containerization of all pipeline applications for portability and reproducibility)

## Examples

```
nextflow run nf-rnaSeqMetagen -profile slurm --mode run.FilterClassify -c myparams.config
```

## Evaluation signals

- Each per-sample output directory (e.g., examples/output/SRR5074528/) contains both a kraken2 per-read classification file and a kraken2 taxon report.
- Interactive Krona HTML charts exist for the sample (SRR5074528_reads.html for raw reads and SRR5074528_fasta.html for assembled reads) and each opens as a self-contained hierarchical taxonomic chart.
- Taxa reported in the kraken2 report are consistent with the organisms displayed in the downstream Krona chart for the same sample.
- Host-filtered (exogenous) paired-end FASTQ files are present as inputs to the kraken2 classification stage.
- The run completed via run.FilterClassify mode with the slurm profile and Singularity containers, matching the documented command pattern.

## Limitations

- The documented kraken2 database (K2DB) is built from bacterial, archaeal, and viral genomes only, so taxa outside these groups are not characterized.
- Classification quality depends on the completeness of the microbial database and the availability of the host reference genome, annotation, and STAR/bowtie2 indexes.
- The tutorial's Krona reports were demonstrated for a single sample (SRR5074528) from the H3ABioNet RNA-seq test dataset; other samples require their own per-sample output directories and validation.

## Evidence

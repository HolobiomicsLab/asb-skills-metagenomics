---
name: read-taxonomic-functional-profiling-workflow
description: 'Use when paired-end shotgun metagenomic reads require adapter trimming,
  host removal, a Kraken2 taxonomic profile, and a gene-length-normalized KEGG ortholog
  abundance profile from UniRef90 read mapping.

  '
license: CC-BY-4.0
metadata:
  kind: composite-workflow
  techniques: []
  stage_count: 5
  member_skills:
  - metagenomic-read-quality-control--333f0c268bb84080
  - fastqc-quality-reporting
  - multiqc-report-aggregation
  - short-read-quality-trimming--534731b241401dae
  - illumina-adapter-trimming-bbduk
  - host-read-filtering
  - short-read-alignment-to-host-reference
  - fastq-read-count-summary-statistics
  - host-read-alignment-against-human-genome
  - metagenomic-taxonomic-profiling
  - taxonomic-profile-generation
  - metagenomic-taxonomy-profiling-of-rna-seq-reads
  - kraken2-taxonomy-charting
  - metagenomic-read-gene-annotation
  - shotgun-metagenome-gene-abundance-estimation
  - metagenomic-hit-summarization
  - alignment-hits-tabular-output-processing
  - kegg-orthology-annotation
  - metagenomic-gene-to-ortholog-mapping
  - many-to-many-mapping-table-parsing
  - fractional-ortholog-abundance-counting
  member_tools:
  - BBTools
  - FastQC
  - MultiQC
  - Bowtie 2
  - Conda
  - MetaLAFFA
  - CoverM
  - FEAST
  - Kraken2
  - Python
  - SourceApp
  - decOM
  - fastp
  - kmtricks
  - metaSourceTracker
  - DIAMOND
  - Snakemake
  coverage_gaps: []
  derived_from_workflows: []
  bound_by: perspicacite-semantic
  collection: https://w3id.org/holobiomicslab/asb-skill/collection/metagenomics/v1
schema_version: 0.3.0
attribution:
  generator: AgenticScienceBuilder
  promoter: Louis-Félix Nothias
  sponsor: CNRS & Université Côte d'Azur
---

# Read-based Taxonomic and Ortholog Profiling

## Summary

Produce sample-level Kraken2 taxonomic profiles and fractionally counted KO abundance profiles from the same trimmed, host-filtered short reads.


## When to use

Use when paired-end shotgun metagenomic reads require adapter trimming, host removal, a Kraken2 taxonomic profile, and a gene-length-normalized KEGG ortholog abundance profile from UniRef90 read mapping.


## When NOT to use

- The data is not this platform.
- You need a single atomic step, not the full pipeline (use the leaf skill directly via the router).

## Stages

### Stage 1 — trim_and_qc

**Goal:** remove adapters and low-quality sequence and summarize read quality

**EDAM operation:** http://edamontology.org/operation_3219

**Inputs:** paired-fastq/raw · **Outputs:** fastq/trimmed, html/read-qc-report

**Candidate leaf skills:** `metagenomic-read-quality-control--333f0c268bb84080` (primary), `fastqc-quality-reporting`, `multiqc-report-aggregation`, `short-read-quality-trimming--534731b241401dae`, `illumina-adapter-trimming-bbduk`

**Tools (primary):** BBTools, FastQC, MultiQC


**Grounding:** 1 KB(s); DOIs: 10.1093/nargab/lqac070

### Stage 2 — remove_host

**Goal:** discard reads aligning above threshold to the configured host reference

**EDAM operation:** http://edamontology.org/operation_3187

**Inputs:** fastq/trimmed, database/host-bowtie2-index · **Outputs:** fastq/host-filtered, tsv/host-removal-statistics

**Candidate leaf skills:** `host-read-filtering` (primary), `short-read-alignment-to-host-reference`, `fastq-read-count-summary-statistics`, `host-read-alignment-against-human-genome`

**Tools (primary):** Bowtie 2, Conda, MetaLAFFA

**Other candidate tools:** Bowtie2, Hostile, Minimap2, Samtools, Snakemake, FLASH, Python 2.7, SAMSA, Trimmomatic

**Grounding:** 3 KB(s); DOIs: 10.1093/bioinformatics/btad728, 10.1186/s12859-016-1270-8, 10.1186/s12859-020-03815-9

### Stage 3 — build_taxonomic_profile

**Goal:** classify host-filtered reads into sample-level Kraken2 taxonomic profiles

**EDAM operation:** http://edamontology.org/operation_0222

**Inputs:** fastq/host-filtered, database/kraken2-taxonomy · **Outputs:** tsv/taxonomic-profile

**Candidate leaf skills:** `metagenomic-taxonomic-profiling` (primary), `taxonomic-profile-generation`, `metagenomic-taxonomy-profiling-of-rna-seq-reads`, `kraken2-taxonomy-charting`

**Tools (primary):** CoverM, FEAST, Kraken2, Python, SourceApp, decOM, fastp, kmtricks, metaSourceTracker

**Other candidate tools:** DeepMicrobes, Kaiju, Kraken, R, Nextflow, STAR, Singularity, nf-rnaSeqMetagen, MetaPhage, MetaQuast, MultiQC

**Grounding:** 4 KB(s); DOIs: 10.1016/j.medmic.2020.100011, 10.1021/acs.est.5c03603, 10.1093/nargab/lqaa009, 10.1128/msystems.00741-22

### Stage 4 — quantify_genes

**Goal:** map reads to UniRef90 and derive fractional gene-length-normalized abundances

**EDAM operation:** http://edamontology.org/operation_0292

**Inputs:** fastq/host-filtered, database/uniref90-protein · **Outputs:** tsv/filtered-read-gene-hits, tsv/gene-abundance

**Candidate leaf skills:** `metagenomic-read-gene-annotation` (primary), `shotgun-metagenome-gene-abundance-estimation`, `metagenomic-hit-summarization`, `alignment-hits-tabular-output-processing`

**Tools (primary):** DIAMOND, MetaLAFFA, Python, Snakemake


**Grounding:** 1 KB(s); DOIs: 10.1186/s12859-020-03815-9

### Stage 5 — aggregate_orthologs

**Goal:** fractionally aggregate gene abundances into conserved KO profiles

**EDAM operation:** http://edamontology.org/operation_0362

**Inputs:** tsv/gene-abundance, tsv/gene-to-ko-map · **Outputs:** tsv/ko-abundance-profile

**Candidate leaf skills:** `kegg-orthology-annotation` (primary), `metagenomic-gene-to-ortholog-mapping`, `many-to-many-mapping-table-parsing`, `fractional-ortholog-abundance-counting`

**Tools (primary):** MetaLAFFA, Python, Snakemake

**Other candidate tools:** UniRef90

**Grounding:** 1 KB(s); DOIs: 10.1186/s12859-020-03815-9

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.

---
name: assembly-mag-quality-taxonomy-workflow
description: 'Use when paired-end short-read metagenomes must be trimmed, assembled
  per sample, binned into MAGs, split by BUSCO quality, and assigned taxonomy with
  GTDB-Tk or the CAT/BAT fallback route.

  '
license: CC-BY-4.0
metadata:
  kind: composite-workflow
  techniques: []
  stage_count: 5
  member_skills:
  - metagenomic-read-quality-control--333f0c268bb84080
  - short-read-quality-trimming--534731b241401dae
  - adapter-trimming-quality-control
  - fastqc-quality-reporting
  - metagenome-de-novo-assembly
  - assembled-contig-fasta-handling
  - assembly-contiguity-quality-assessment
  - paired-end-read-assembly--a5035386db5a753c
  - assembled-contig-length-and-count-qc
  - metabat2-contig-binning
  - metagenomic-contig-binning--7530fd71afe1cdc5
  - metagenomic-contig-binning--aa6be0a34a35d65a
  - metagenomic-contig-depth-estimation
  - metagenome-binning-quality-assessment
  - metagenome-assembled-genome-quality-assessment--b4335ce8bdbe4c98
  - marker-gene-completeness-contamination-estimation
  - per-bin-summary-table-generation
  - gtdbtk-genome-taxonomy-assignment
  - mag-taxonomic-classification--8bedab30d38f7e0c
  - catbat-bin-taxonomic-classification
  - mag-taxonomic-classification--6714e56ce1419c81
  member_tools:
  - BBTools
  - FastQC
  - MultiQC
  - MEGAHIT
  - QUAST
  - SPAdes
  - hybridSPAdes
  - nf-test
  - CAMITAX
  - GTDBtk
  - MegaHIT v1.2.9
  - MetaBAT2
  - MetaBAT2 (v2:2.15)
  - Sickle
  - kMetaShot
  - metaSPAdes v3.15.2
  - BUSCO
  - CheckM2
  - GTDB-Tk
  - Nextflow
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

# Short-read Metagenome Assembly, MAG Quality, and Taxonomy

## Summary

Produce per-sample MAG bins, assembly and BUSCO quality metrics, and a merged taxonomy table whose method records quality-gated GTDB-Tk or CAT/BAT assignment.


## When to use

Use when paired-end short-read metagenomes must be trimmed, assembled per sample, binned into MAGs, split by BUSCO quality, and assigned taxonomy with GTDB-Tk or the CAT/BAT fallback route.


## When NOT to use

- The data is not this platform.
- You need a single atomic step, not the full pipeline (use the leaf skill directly via the router).

## Stages

### Stage 1 — clean_reads

**Goal:** remove adapters and low-quality tails and summarize read quality

**EDAM operation:** http://edamontology.org/operation_3219

**Inputs:** paired-fastq/raw · **Outputs:** fastq/trimmed, html/read-qc-report

**Candidate leaf skills:** `metagenomic-read-quality-control--333f0c268bb84080` (primary), `short-read-quality-trimming--534731b241401dae`, `adapter-trimming-quality-control`, `fastqc-quality-reporting`

**Tools (primary):** BBTools, FastQC, MultiQC

**Other candidate tools:** SortMeRNA

**Grounding:** 1 KB(s); DOIs: 10.1093/nargab/lqac070

### Stage 2 — assemble_contigs

**Goal:** assemble short reads per sample and compare contiguity metrics

**EDAM operation:** http://edamontology.org/operation_0525

**Inputs:** fastq/trimmed · **Outputs:** fasta/per-sample-contigs, tsv/assembly-qc

**Candidate leaf skills:** `metagenome-de-novo-assembly` (primary), `assembled-contig-fasta-handling`, `assembly-contiguity-quality-assessment`, `paired-end-read-assembly--a5035386db5a753c`, `assembled-contig-length-and-count-qc`

**Tools (primary):** MEGAHIT, QUAST, SPAdes, hybridSPAdes, nf-test

**Other candidate tools:** Docker, Python, SPAdes 3.15.3, metaSPAdes, virMine 2.0, Nextflow, ABySS, FastQC, SRA Toolkit, Trimmomatic, Velvet, WebQUAST

**Grounding:** 3 KB(s); DOIs: 10.1093/nar/gkad406, 10.1093/nargab/lqac007, 10.1128/mra.00107-22

### Stage 3 — bin_contigs

**Goal:** group sufficiently long contigs into per-sample MAG FASTA bins

**EDAM operation:** http://edamontology.org/operation_0310

**Inputs:** fasta/per-sample-contigs · **Outputs:** directory/mag-bins

**Candidate leaf skills:** `metabat2-contig-binning` (primary), `metagenomic-contig-binning--7530fd71afe1cdc5`, `metagenomic-contig-binning--aa6be0a34a35d65a`, `metagenomic-contig-depth-estimation`

**Tools (primary):** CAMITAX, FastQC, GTDBtk, MegaHIT v1.2.9, MetaBAT2, MetaBAT2 (v2:2.15), Sickle, kMetaShot, metaSPAdes v3.15.2

**Other candidate tools:** Bowtie2, CheckM, Conda, GCC, MetaCortex, SRA toolkit, trim-galore, Mash, MegaHIT, metaSPAdes

**Grounding:** 3 KB(s); DOIs: 10.1093/bib/bbae680, 10.1093/bioinformatics/btad020, 10.1093/nargab/lqac007

### Stage 4 — assess_mag_quality

**Goal:** estimate marker-based completeness and contamination and split classifier routes

**EDAM operation:** http://edamontology.org/operation_0362

**Inputs:** directory/mag-bins, yaml/mag-quality-thresholds · **Outputs:** tsv/busco-mag-quality, directory/gtdbtk-eligible-mags, directory/catbat-fallback-mags

**Candidate leaf skills:** `metagenome-binning-quality-assessment` (primary), `metagenome-assembled-genome-quality-assessment--b4335ce8bdbe4c98`, `marker-gene-completeness-contamination-estimation`, `per-bin-summary-table-generation`

**Tools (primary):** BUSCO, CheckM2, GTDB-Tk

**Other candidate tools:** BIgMAG, MultiQC, QUAST, Nextflow

**Grounding:** 1 KB(s); DOIs: 10.1093/nargab/lqac007

### Stage 5 — classify_mags

**Goal:** classify passing bins with GTDB-Tk and remaining bins with CAT/BAT

**EDAM operation:** http://edamontology.org/operation_0363

**Inputs:** tsv/busco-mag-quality, directory/gtdbtk-eligible-mags, directory/catbat-fallback-mags, database/gtdb-reference, database/catbat-reference · **Outputs:** tsv/mag-taxonomy

**Candidate leaf skills:** `gtdbtk-genome-taxonomy-assignment` (primary), `mag-taxonomic-classification--8bedab30d38f7e0c`, `catbat-bin-taxonomic-classification`, `mag-taxonomic-classification--6714e56ce1419c81`

**Tools (primary):** BUSCO, GTDB-Tk, Nextflow

**Other candidate tools:** CAMITAX, CMDbench, FastQC, GTDBtk, Mash, MegaHIT, MetaBAT2, Sickle, kMetaShot, metaSPAdes

**Grounding:** 2 KB(s); DOIs: 10.1093/bib/bbae680, 10.1093/nargab/lqac007

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.

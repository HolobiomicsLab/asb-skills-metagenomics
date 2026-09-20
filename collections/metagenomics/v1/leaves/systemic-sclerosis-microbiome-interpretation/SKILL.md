---
name: systemic-sclerosis-microbiome-interpretation
description: Use when kraken2 per-sample reports exist for paired SSc-patient and
  healthy-control RNA-seq samples and the goal is to compute SSc-exclusive taxa (set-difference
  presence/absence analysis) and confirm clinically relevant genera.
when_to_use_negative:
- Input is already a presence/absence or feature table that does not require kraken2
  classification of raw RNA-seq reads
- Samples lack group metadata distinguishing SSc patients from healthy controls, so
  the set difference cannot be computed
- The goal is host transcriptome expression analysis rather than exogenous microbial
  read characterization
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: nf-rnaSeqMetagen
  role: Nextflow pipeline that filters host reads and classifies exogenous reads from
    RNA-seq data (run.FilterClassify mode)
  repo: https://github.com/phelelani/nf-rnaSeqMetagen
- name: kraken2
  role: Characterizes remaining exogenous reads against a database constructed from
    bacterial, archaeal, and viral genomes to produce per-sample taxonomic calls
- name: UpSetR
  role: R package to generate UpSet plots for cross-checking set intersections between
    the SSc and control taxon sets
  repo: https://github.com/hms-dbmi/UpSetR
- name: UpSet
  role: Interactive web-based visualization technique for analyzing the set-based
    SSc vs control taxon membership data
  repo: https://github.com/VCG/upset/archive/master.zip
- name: Nextflow
  role: Workflow management system orchestrating the pipeline applications
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.medmic.2020.100011
    title: 'nf-rnaSeqMetagen: A nextflow metagenomics pipeline for identifying and
      characterizing microbial sequences from RNA-seq data'
  source_content_hash: sha256:e6134b931c65fb7dfeb14f787ffefea62dc6d8aba2ba3bfcaf4f61662612e1f7
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
  - UpSet
  - UpSetR
  - kraken2
  - nf-rnaSeqMetagen
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# systemic-sclerosis-microbiome-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Interprets kraken2 taxonomic classifications from the nf-rnaSeqMetagen run.FilterClassify mode on systemic sclerosis (SSc) skin RNA-seq data to identify pathogens present in SSc patients but absent from healthy controls, such as Epstein-Barr virus and the clinically relevant genera Arthrobacter, Bacillus, Brachybacterium, Dietzia and Pseudarthrobacter.

## When to use

Use when kraken2 per-sample reports exist for paired SSc-patient and healthy-control RNA-seq samples and the goal is to compute SSc-exclusive taxa (set-difference presence/absence analysis) and confirm clinically relevant genera.

## When NOT to use

- Input is already a presence/absence or feature table that does not require kraken2 classification of raw RNA-seq reads
- Samples lack group metadata distinguishing SSc patients from healthy controls, so the set difference cannot be computed
- The goal is host transcriptome expression analysis rather than exogenous microbial read characterization

## Inputs

- Per-sample kraken2 taxonomic classification reports from nf-rnaSeqMetagen run.FilterClassify mode
- Sample metadata assigning samples to SSc patient vs healthy control groups from the systemic sclerosis skin RNA-seq dataset
- FASTQ input files (fastq/fastq.gz/fastq.bz2/fq/fq.gz/fq.bz2) for the RNA-seq samples

## Outputs

- pathogen_presence_absence.tsv — taxon x sample-group presence/absence table
- Optional UpSetR set-intersection plot of the SSc versus control taxon sets

## How to apply

Run nf-rnaSeqMetagen in run.FilterClassify mode so host reads are filtered out and exogenous reads are classified against a kraken2 database built from bacterial, archaeal, and viral genomes. Load the per-sample kraken2 reports and assign each sample to its group (SSc patient vs healthy control) using the systemic sclerosis skin RNA-seq sample metadata. Build a taxon x sample binary presence/absence matrix, treating a taxon as present when kraken2 reports classified reads for it in that sample. Compute the set difference between the SSc taxon set and the control taxon set to obtain SSc-exclusive taxa, and optionally cross-check memberships with an UpSet/UpSetR set-intersection view of the two groups. Write the resulting taxon x sample-group table as pathogen_presence_absence.tsv.

## Related tools

- **nf-rnaSeqMetagen** (Nextflow pipeline that filters host reads and classifies exogenous reads from RNA-seq data (run.FilterClassify mode)) — https://github.com/phelelani/nf-rnaSeqMetagen
- **kraken2** (Characterizes remaining exogenous reads against a database constructed from bacterial, archaeal, and viral genomes to produce per-sample taxonomic calls)
- **UpSetR** (R package to generate UpSet plots for cross-checking set intersections between the SSc and control taxon sets) — https://github.com/hms-dbmi/UpSetR
- **UpSet** (Interactive web-based visualization technique for analyzing the set-based SSc vs control taxon membership data) — https://github.com/VCG/upset/archive/master.zip
- **Nextflow** (Workflow management system orchestrating the pipeline applications)

## Examples

```
nextflow run nf-rnaSeqMetagen -profile slurm --mode run.FilterClassify -c myparams.config
```

## Evaluation signals

- pathogen_presence_absence.tsv exists in TSV format
- The output table contains a row for Epstein-Barr virus
- Each of the five clinically relevant SSc genera (Arthrobacter, Bacillus, Brachybacterium, Dietzia, Pseudarthrobacter) is marked present in the SSc group and absent from the control group
- The presence/absence matrix rows reconcile with the kraken2 per-sample classified-read calls used to build it
- UpSet/UpSetR set-intersection view agrees with the computed set difference between SSc and control taxon sets

## Limitations

- Detection depends on kraken2 classified reads, so taxa with few or ambiguous exogenous reads may be missed or misclassified
- Pipeline requires software dependencies (Nextflow, Singularity, reference genome, annotation and indexes) that must be prepared via prep.Containers, prep.STARIndex/prep.BowtieIndex and prep.KrakenDB modes before analysis
- Presence/absence calls reflect reads classified against the kraken2 database; contaminants or low-abundance taxa common in SSc patients should be interpreted with clinical relevance in mind

## Evidence

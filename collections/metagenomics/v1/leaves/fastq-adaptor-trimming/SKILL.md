---
name: fastq-adaptor-trimming
description: Use when raw paired-end FASTQ metatranscriptome reads contain adaptor
  contamination or low-quality bases and must be trimmed and mate-joined before annotation,
  e.g. preparing 100 bp paired-end stool RNA-seq files for MG-RAST submission.
when_to_use_negative:
- Input reads are already long (>100 bp) single-end sequences that satisfy the pipeline's
  minimum-length requirement, so paired-end joining with FLASH/PEAR is unnecessary
- Reads have already been adaptor-trimmed and quality-filtered, and you would only
  be re-running Trimmomatic redundantly
- You are running SAMSA2's local pipeline and want its recommended merge tool (PEAR
  with multithreading, plus SortMeRNA ribosomal filtering) rather than the SAMSA v1
  Trimmomatic+FLASH step
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3308
tools:
- name: Trimmomatic
  role: Cleans raw FASTQ reads by removing adaptor contamination and low-quality sequences
    before pair joining
  repo: http://www.usadellab.org/cms/?page=trimmomatic
- name: FLASH
  role: Joins/aligns the trimmed paired-end fastq files into merged reads that exceed
    the 100 bp minimum for annotation (SAMSA v1)
  repo: github.com/transcript/SAMSA
- name: PEAR
  role: Merges two paired-end FASTQ files into a single file of extended fragments
    (SAMSA2 replacement for FLASH)
  repo: https://sco.h-its.org/exelixis/web/software/pear/
- name: SAMSA
  role: Pipeline package whose SAMSA_pre_annotation_pipeline.py orchestrates the Trimmomatic
    + FLASH preprocessing step
  repo: github.com/transcript/SAMSA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-016-1270-8
    title: 'SAMSA: a comprehensive metatranscriptome analysis pipeline'
  source_content_hash: sha256:8ecd1308f7080371480d743eea50383de8a2d1b87a28bf5a0862d7a1ef8befa8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FLASH
  - PEAR
  - SAMSA
  - Trimmomatic
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# fastq-adaptor-trimming

## Summary

Preprocessing phase of the SAMSA metatranscriptome pipeline: raw FASTQ reads are cleaned of adaptor contamination and low-quality bases with Trimmomatic, then paired-end mates are joined with FLASH (PEAR in SAMSA2) so reads exceed the 100 bp minimum needed for downstream MG-RAST annotation.

## When to use

Use when raw paired-end FASTQ metatranscriptome reads contain adaptor contamination or low-quality bases and must be trimmed and mate-joined before annotation, e.g. preparing 100 bp paired-end stool RNA-seq files for MG-RAST submission.

## When NOT to use

- Input reads are already long (>100 bp) single-end sequences that satisfy the pipeline's minimum-length requirement, so paired-end joining with FLASH/PEAR is unnecessary
- Reads have already been adaptor-trimmed and quality-filtered, and you would only be re-running Trimmomatic redundantly
- You are running SAMSA2's local pipeline and want its recommended merge tool (PEAR with multithreading, plus SortMeRNA ribosomal filtering) rather than the SAMSA v1 Trimmomatic+FLASH step

## Inputs

- Raw paired-end FASTQ sequencing files (metatranscriptome samples, e.g. 100 bp paired-end)
- Trimmomatic adaptor FASTA file (SAMSA2: 'Trimmomatic removes adapter contamination according to a specific fasta file')
- Installed dependencies: Trimmomatic, FLASH (or PEAR for SAMSA2), Python 2.7, R

## Outputs

- Trimmed FASTQ files with adaptor contamination and low-quality bases removed
- Merged FASTQ file of extended fragments exceeding the 100 bp minimum length
- Preprocessing metrics (fraction of raw reads aligned, average aligned read length, e.g. ~32-54% and ~178 bp) written to preprocessing_stats.tsv
- Prepared sequences ready for MG-RAST annotation submission

## How to apply

Run SAMSA's SAMSA_pre_annotation_pipeline.py, which starts from raw fastq input files, performs cleaning and removal of adaptor contamination with Trimmomatic, and then aligns/joins the paired-end files with FLASH if mates are specified. SAMSA expects the dependencies FLASH, Trimmomatic, Python 2.7, and R; the merged output must exceed the ~100 bp minimum read length the pipeline requires for annotation. For 100 bp paired-end input, judge success by the fraction of raw reads that align after joining (the paper's pilot metatranscriptome samples fell in the ~32-54% range) and an average aligned read length near ~178 bp. Write computed metrics (aligned fraction, average aligned length) to a stats table and keep the merged fastq files for upload to MG-RAST for annotation. If using SAMSA2 locally, PEAR replaces FLASH for merging and SortMeRNA removes ribosomal reads after trimming.

## Related tools

- **Trimmomatic** (Cleans raw FASTQ reads by removing adaptor contamination and low-quality sequences before pair joining) — http://www.usadellab.org/cms/?page=trimmomatic
- **FLASH** (Joins/aligns the trimmed paired-end fastq files into merged reads that exceed the 100 bp minimum for annotation (SAMSA v1)) — github.com/transcript/SAMSA
- **PEAR** (Merges two paired-end FASTQ files into a single file of extended fragments (SAMSA2 replacement for FLASH)) — https://sco.h-its.org/exelixis/web/software/pear/
- **SAMSA** (Pipeline package whose SAMSA_pre_annotation_pipeline.py orchestrates the Trimmomatic + FLASH preprocessing step) — github.com/transcript/SAMSA

## Examples

```
python SAMSA_pre_annotation_pipeline.py -usage
```

## Evaluation signals

- Fraction of raw reads successfully aligned after FLASH joining falls within the paper's reported ~32-54% range for pilot metatranscriptome samples
- Average aligned (merged) read length is near ~178 bp for 100 bp paired-end input
- Merged reads exceed the ~100 bp minimum length required for MG-RAST annotation
- Output FASTQ files no longer show adaptor contamination (Trimmomatic cleaned against the adaptor FASTA)
- Trimmed/merged FASTQ files are retained and format-valid for downstream MG-RAST upload, with metrics logged to preprocessing_stats.tsv

## Limitations

- Each sample needs 40-50 million raw sequences to yield the 5-10 million annotated reads necessary for accurate abundance measures; shallower input undermines downstream analysis
- Ribosomal RNA depletion does not equally deplete ribosomes from all species, and remaining rRNA sequences must be discarded after preprocessing and before further analysis
- Sequences shorter than ~100 bp cannot be annotated unless paired-end mates are successfully joined; unmerged short reads are effectively lost

## Evidence

- [methods] Preprocessing phase overview: "it performs cleaning and removal of adaptor contamination, and then aligns paired-end files (if specified)"
- [methods] Dependency list for the trimming/joining step: "SAMSA has several dependencies, including the programs FLASH and Trimmomatic, Python version 2.7, and R"
- [readme] Preprocessing script runs Trimmomatic and FLASH: "SAMSA_pre_annotation_pipeline.py - runs Trimmomatic, FLASH (if paired-end files), prepares raw sequences for upload to MG-RAST"
- [abstract] Minimum read length requirement: "Sequences need to be either long reads (longer than 100 bp) or joined paired-end reads."
- [readme] SAMSA2 merge tool: [evidence span withheld — anchor status: external_doc]

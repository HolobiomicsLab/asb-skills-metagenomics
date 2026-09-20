---
name: low-quality-base-trimming
description: Use when raw paired-end metatranscriptome FASTQ files show adaptor contamination
  or low-quality bases and must be cleaned before read joining (FLASH) and submission
  to an annotation server such as MG-RAST.
when_to_use_negative:
- Input reads are already cleaned adapter-free single-end/long reads longer than 100
  bp — SAMSA notes sequences need to be either long reads (>100 bp) or joined paired-end
  reads, so merging/trimming may be unnecessary.
- The goal is ribosomal read removal — that is a separate filtering step (ribodepletion
  checks / discarding remaining rRNA reads), not quality trimming.
- Running SAMSA2 with local DIAMOND annotation where preprocessing is orchestrated
  by master_script.bash (which uses PEAR instead of FLASH) and Trimmomatic is invoked
  internally rather than as a standalone trimming pass.
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0080
tools:
- name: Trimmomatic
  role: Removes adaptor contamination and low-quality sequences/bases from raw FASTQ
    reads as the trimming step of SAMSA preprocessing
- name: FLASH
  role: Joins/aligns the trimmed paired-end FASTQ files into merged reads exceeding
    the ~100 bp minimum length for annotation
- name: SAMSA
  role: Wrapper pipeline (SAMSA_pre_annotation_pipeline.py) that runs Trimmomatic
    and FLASH and prepares the merged reads for MG-RAST upload
  repo: github.com/transcript/SAMSA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-016-1270-8
    title: 'SAMSA: a comprehensive metatranscriptome analysis pipeline'
  source_content_hash: sha256:6af3ca34492087d998d9afde3520aed81284f37d335757bdc31d747f3a1d3d39
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FLASH
  - SAMSA
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# low-quality-base-trimming

## Summary

Cleaning of raw metatranscriptomic FASTQ reads by removing adaptor contamination and low-quality bases with Trimmomatic as the first step of the SAMSA preprocessing phase, prior to paired-end joining and MG-RAST annotation.

## When to use

Use when raw paired-end metatranscriptome FASTQ files show adaptor contamination or low-quality bases and must be cleaned before read joining (FLASH) and submission to an annotation server such as MG-RAST.

## When NOT to use

- Input reads are already cleaned adapter-free single-end/long reads longer than 100 bp — SAMSA notes sequences need to be either long reads (>100 bp) or joined paired-end reads, so merging/trimming may be unnecessary.
- The goal is ribosomal read removal — that is a separate filtering step (ribodepletion checks / discarding remaining rRNA reads), not quality trimming.
- Running SAMSA2 with local DIAMOND annotation where preprocessing is orchestrated by master_script.bash (which uses PEAR instead of FLASH) and Trimmomatic is invoked internally rather than as a standalone trimming pass.

## Inputs

- Raw paired-end metatranscriptome FASTQ files (100 bp paired-end recommended)
- Adaptor/adapter contaminant sequences in FASTA format (used by Trimmomatic)

## Outputs

- Trimmed FASTQ files (adapter- and low-quality-base-cleaned reads)
- FLASH-merged FASTQ of joined paired-end reads (>~100 bp fragments)
- Preprocessing statistics (e.g., preprocessing_stats.tsv) with fraction of reads aligned and average aligned read length

## How to apply

Start from the raw paired-end FASTQ files for each metatranscriptome sample (SAMSA recommends 100 bp paired-end format with 40-50 million raw sequences per sample). Run Trimmomatic to perform cleaning and removal of adaptor contamination according to a specific adapter FASTA file, and to trim low-quality sequences/bases. Keep the trimmed FASTQ files and pass them to FLASH to join the paired-end reads into merged fragments — merged reads must exceed the ~100 bp minimum length required for annotation. Record the fraction of raw reads that successfully align after joining and the average aligned read length to a preprocessing_stats file; the paper's pilot samples yielded ~32-54% aligned reads with an average aligned length of ~178 bp from 100 bp paired-end input. Retain the cleaned, merged FASTQ for downstream MG-RAST annotation; discarding the remaining rRNA reads (e.g., after ribodepletion) is a separate downstream step.

## Related tools

- **Trimmomatic** (Removes adaptor contamination and low-quality sequences/bases from raw FASTQ reads as the trimming step of SAMSA preprocessing)
- **FLASH** (Joins/aligns the trimmed paired-end FASTQ files into merged reads exceeding the ~100 bp minimum length for annotation)
- **SAMSA** (Wrapper pipeline (SAMSA_pre_annotation_pipeline.py) that runs Trimmomatic and FLASH and prepares the merged reads for MG-RAST upload) — github.com/transcript/SAMSA

## Examples

```
python SAMSA_pre_annotation_pipeline.py -input_file sample_R1.fastq -input_file2 sample_R2.fastq -trimmed_file sample_trimmed.fastq -joined_file sample_joined.fastq
```

## Evaluation signals

- Fraction of raw reads successfully aligned after trimming + FLASH joining falls within the reported ~32-54% range for the pilot metatranscriptome samples
- Average aligned read length is near the reported ~178 bp for 100 bp paired-end input
- Merged read lengths exceed the ~100 bp minimum required for annotation
- Trimmed FASTQ files contain no residual adaptor contamination and pass Trimmomatic quality filtering
- Preprocessing stats (e.g., preprocessing_stats.tsv) are populated and merged FASTQ files are retained for MG-RAST submission

## Limitations

- Only a fraction (~32-54%) of raw reads survive trimming and joining; the remainder are lost from downstream analysis
- Each sample needs 40-50 million raw sequences to yield the 5-10 million annotated reads necessary for accurate abundance measures — trimming/joining losses must be budgeted for in sequencing depth
- In SAMSA2, FLASH is replaced by PEAR and Trimmomatic adapter removal depends on a specific adapter FASTA file, so adapter content must match the library preparation

## Evidence

- [methods] SAMSA has several dependencies, including the programs FLASH and Trimmomatic, Python version 2.7, and R: "SAMSA has several dependencies, including the programs FLASH and Trimmomatic, Python version 2.7, and R"
- [methods] it performs cleaning and removal of adaptor contamination, and then aligns paired-end files (if specified): "it performs cleaning and removal of adaptor contamination, and then aligns paired-end files (if specified)"
- [abstract] Sequences need to be either long reads (longer than 100 bp) or joined paired-end reads.: "Sequences need to be either long reads (longer than 100 bp) or joined paired-end reads."
- [abstract] Each sample needs 40-50 million raw sequences, which can be expected to yield the 5-10 million annotated reads necessary for accurate abundance measures.: "Each sample needs 40-50 million raw sequences, which can be expected to yield the 5-10 million annotated reads necessary for accurate abundance measures."
- [readme] SAMSA_pre_annotation_pipeline.py - runs Trimmomatic, FLASH (if paired-end files), prepares raw sequences for upload to MG-RAST: "SAMSA_pre_annotation_pipeline.py - runs Trimmomatic, FLASH (if paired-end files), prepares raw sequences for upload to MG-RAST"
- [readme] Trimmomatic removes adapter contamination according to a specific fasta file.: [evidence span withheld — anchor status: external_doc]

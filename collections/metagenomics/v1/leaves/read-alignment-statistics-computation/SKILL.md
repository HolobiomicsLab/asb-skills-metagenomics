---
name: read-alignment-statistics-computation
description: Use when preprocessing paired-end FASTQ metatranscriptome reads with
  Trimmomatic + FLASh and you need to quantify the merged-read fraction and average
  merged read length before annotation submission.
when_to_use_negative:
- Input reads are already single-end long reads longer than 100 bp, so no paired-end
  joining step is needed and merge-fraction statistics do not apply.
- Reads have already been merged and quality-trimmed upstream; recomputing trimming/joining
  statistics on preprocessed files would double-count losses.
- The goal is annotation or statistical comparison (MG-RAST annotation, DESeq2 analysis),
  which are downstream SAMSA stages, not preprocessing alignment statistics.
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3174
tools:
- name: Trimmomatic
  role: Removes adaptor contamination and low-quality bases from paired-end reads
    prior to join-statistics computation
  repo: https://github.com/transcript/SAMSA
- name: FLASH
  role: Joins/aligns trimmed paired-end files; merged read counts and lengths feed
    the aligned-fraction and average-length metrics
  repo: https://github.com/transcript/SAMSA
- name: SAMSA
  role: Wraps the preprocessing pipeline (SAMSA_pre_annotation_pipeline.py) that runs
    Trimmomatic and FLASH and prepares merged reads for MG-RAST upload
  repo: https://github.com/transcript/SAMSA
- name: Python 2.7
  role: Environment for running the SAMSA preprocessing and statistics scripts
  repo: https://github.com/transcript/SAMSA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-016-1270-8
    title: 'SAMSA: a comprehensive metatranscriptome analysis pipeline'
  source_content_hash: sha256:cd2c79b2d0b4fc3ee27af38d3121d8898e69240c210381baa42e321ef8a7a5ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/transcript/SAMSA
  tools:
  - FLASH
  - Python 2.7
  - SAMSA
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# read-alignment-statistics-computation

## Summary

Computes the fraction of raw paired-end reads successfully aligned (joined) after Trimmomatic trimming and FLASh merging in the SAMSA preprocessing phase, plus the average aligned read length (~178 bp for 100 bp paired-end input).

## When to use

Use when preprocessing paired-end FASTQ metatranscriptome reads with Trimmomatic + FLASh and you need to quantify the merged-read fraction and average merged read length before annotation submission.

## When NOT to use

- Input reads are already single-end long reads longer than 100 bp, so no paired-end joining step is needed and merge-fraction statistics do not apply.
- Reads have already been merged and quality-trimmed upstream; recomputing trimming/joining statistics on preprocessed files would double-count losses.
- The goal is annotation or statistical comparison (MG-RAST annotation, DESeq2 analysis), which are downstream SAMSA stages, not preprocessing alignment statistics.

## Inputs

- raw paired-end FASTQ files from a metatranscriptome sample
- Trimmomatic-trimmed FASTQ mate-pair files (R1/R2)
- FLASH-merged FASTQ of joined reads

## Outputs

- fraction of raw reads successfully aligned after FLASh joining (percentage)
- average aligned (merged) read length in base pairs
- preprocessing_stats.tsv summarizing the metrics
- merged FASTQ retained for MG-RAST submission

## How to apply

Load the raw paired-end FASTQ files and run Trimmomatic to remove adaptor contamination and low-quality bases. Run FLASH to join the trimmed mate-pair files into merged reads. From the read counts before and after joining, compute the fraction of raw reads successfully aligned (the paper's pilot metatranscriptome samples fell in the ~32-54% range) and the average aligned read length (~178 bp from 100 bp paired-end input). Record these metrics (e.g., preprocessing_stats.tsv) and retain the trimmed/merged FASTQ files for downstream MG-RAST annotation. Validation hinges on the merged reads exceeding the 100 bp minimum length SAMSA requires for annotation.

## Related tools

- **Trimmomatic** (Removes adaptor contamination and low-quality bases from paired-end reads prior to join-statistics computation) — https://github.com/transcript/SAMSA
- **FLASH** (Joins/aligns trimmed paired-end files; merged read counts and lengths feed the aligned-fraction and average-length metrics) — https://github.com/transcript/SAMSA
- **SAMSA** (Wraps the preprocessing pipeline (SAMSA_pre_annotation_pipeline.py) that runs Trimmomatic and FLASH and prepares merged reads for MG-RAST upload) — https://github.com/transcript/SAMSA
- **Python 2.7** (Environment for running the SAMSA preprocessing and statistics scripts) — https://github.com/transcript/SAMSA

## Examples

```
python SAMSA_pre_annotation_pipeline.py
```

## Evaluation signals

- Aligned-read fraction falls in the ~32-54% range reported for the pilot metatranscriptome samples.
- Average aligned (merged) read length is near ~178 bp for 100 bp paired-end input.
- Merged reads exceed the ~100 bp minimum length required for annotation.
- Trimmed/merged FASTQ outputs are retained and formatted for MG-RAST submission.
- Preprocessing metrics are written to preprocessing_stats.tsv consistent with per-sample read counts.

## Limitations

- Aligned-fraction and read-length yields depend on input read length and paired-end format; the ~32-54% / ~178 bp figures were measured on 100 bp paired-end pilot metatranscriptome samples.
- Merged reads must exceed 100 bp; shorter joins will not be usable by the annotation stage.
- MG-RAST annotation, the downstream consumer of the merged reads, requires queue waits of days to weeks, so validation of downstream results is slow.

## Evidence

- [methods] it performs cleaning and removal of adaptor contamination, and then aligns paired-end files (if specified): "it performs cleaning and removal of adaptor contamination, and then aligns paired-end files (if specified)"
- [abstract] Sequences need to be either long reads (longer than 100 bp) or joined paired-end reads: "Sequences need to be either long reads (longer than 100 bp) or joined paired-end reads."
- [methods] SAMSA has several dependencies, including the programs FLASH and Trimmomatic, Python version 2.7, and R: "SAMSA has several dependencies, including the programs FLASH and Trimmomatic, Python version 2.7, and R"
- [readme] SAMSA_pre_annotation_pipeline.py - runs Trimmomatic, FLASH (if paired-end files), prepares raw sequences for upload to MG-RAST: "SAMSA_pre_annotation_pipeline.py - runs Trimmomatic, FLASH (if paired-end files), prepares raw sequences for upload to MG-RAST"

---
name: paired-end-read-merging
description: Use when raw paired-end FASTQ reads from stool metatranscriptomes must
  be joined into longer single fragments (longer than 100 bp) before annotation, as
  the SAMSA pipeline aligns paired-end files after Trimmomatic cleaning.
when_to_use_negative:
- Input is already single-end or long reads longer than 100 bp — SAMSA states sequences
  need to be either long reads (longer than 100 bp) or joined paired-end reads, so
  merging is unnecessary
- Input FASTQ files have not yet been cleaned — SAMSA performs cleaning and removal
  of adaptor contamination before aligning paired-end files, so merging raw untrimmed
  reads deviates from the pipeline order
- The goal is shotgun metagenomic population-size measurement rather than metatranscriptome
  annotation — merging alone cannot address the pipeline's noted limitations on measuring
  microbial population shifts
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0196
tools:
- name: FLASH
  role: Merges/joins Trimmomatic-cleaned paired-end FASTQ files into single merged
    reads in the SAMSA v1 preprocessing step
  repo: https://github.com/transcript/SAMSA
- name: PEAR
  role: 'Alternative paired-end merger used in SAMSA2: merges two paired-end FASTQ
    files to make a single file of extended fragments'
  repo: https://github.com/transcript/samsa2
- name: Trimmomatic
  role: Runs immediately before merging to remove adaptor contamination and low-quality
    bases from the paired-end FASTQ input
  repo: https://github.com/transcript/SAMSA
- name: SAMSA
  role: Pipeline that orchestrates cleaning, paired-end joining via SAMSA_pre_annotation_pipeline.py,
    and preparation of merged reads for MG-RAST upload
  repo: https://github.com/transcript/SAMSA
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1186/s12859-016-1270-8
    title: 'SAMSA: a comprehensive metatranscriptome analysis pipeline'
  source_content_hash: sha256:bb994ce74571cfbe29d98d193d9ca1bf677f283af9cb84da25728721542cca2f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/transcript/SAMSA
  tools:
  - FLASH
  - PEAR
  - SAMSA
  - Trimmomatic
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# paired-end-read-merging

## Summary

Merging (joining) overlapping paired-end FASTQ reads into single, longer fragments as done in the SAMSA preprocessing phase with FLASH (or PEAR in SAMSA2), after Trimmomatic cleaning, so that merged reads exceed the ~100 bp minimum required for MG-RAST/DIAMOND annotation.

## When to use

Use when raw paired-end FASTQ reads from stool metatranscriptomes must be joined into longer single fragments (longer than 100 bp) before annotation, as the SAMSA pipeline aligns paired-end files after Trimmomatic cleaning.

## When NOT to use

- Input is already single-end or long reads longer than 100 bp — SAMSA states sequences need to be either long reads (longer than 100 bp) or joined paired-end reads, so merging is unnecessary
- Input FASTQ files have not yet been cleaned — SAMSA performs cleaning and removal of adaptor contamination before aligning paired-end files, so merging raw untrimmed reads deviates from the pipeline order
- The goal is shotgun metagenomic population-size measurement rather than metatranscriptome annotation — merging alone cannot address the pipeline's noted limitations on measuring microbial population shifts

## Inputs

- Raw paired-end FASTQ files for a metatranscriptome sample (e.g. 100 bp paired-end stool metatranscriptomic reads)
- Trimmomatic-trimmed paired-end FASTQ files (R1 and R2 after adaptor and quality trimming)

## Outputs

- Merged (joined) FASTQ file of extended fragments longer than 100 bp, ready for MG-RAST annotation
- Preprocessing statistics (fraction of raw reads successfully aligned/joined, e.g. ~32-54%, and average aligned read length, e.g. ~178 bp), e.g. preprocessing_stats.tsv

## How to apply

First run Trimmomatic on the raw paired-end FASTQ files to remove adaptor contamination and low-quality bases, since SAMSA cleans reads before aligning paired-end files. Then run the merger (FLASH in SAMSA v1, invoked via SAMSA_pre_annotation_pipeline.py; PEAR in SAMSA2) on the trimmed mate files to produce merged reads longer than 100 bp, the minimum length SAMSA requires for annotation. Compute the fraction of raw reads that successfully joined/aligned (the pilot metatranscriptome samples reported ~32-54%) and the average aligned read length (~178 bp for 100 bp paired-end input). Keep the merged FASTQ files for downstream submission to MG-RAST annotation and write the joining metrics (e.g. preprocessing_stats.tsv) for validation. Note the SAMSA2 alternative 'master_script_preserving_unmerged.sh' handles unmerged pairs by concatenating forward and reverse reads separated by 20 Ns.

## Related tools

- **FLASH** (Merges/joins Trimmomatic-cleaned paired-end FASTQ files into single merged reads in the SAMSA v1 preprocessing step) — https://github.com/transcript/SAMSA
- **PEAR** (Alternative paired-end merger used in SAMSA2: merges two paired-end FASTQ files to make a single file of extended fragments) — https://github.com/transcript/samsa2
- **Trimmomatic** (Runs immediately before merging to remove adaptor contamination and low-quality bases from the paired-end FASTQ input) — https://github.com/transcript/SAMSA
- **SAMSA** (Pipeline that orchestrates cleaning, paired-end joining via SAMSA_pre_annotation_pipeline.py, and preparation of merged reads for MG-RAST upload) — https://github.com/transcript/SAMSA

## Examples

```
python SAMSA_pre_annotation_pipeline.py -usage
```

## Evaluation signals

- Fraction of raw reads successfully joined/aligned after Trimmomatic + FLASH falls in the reported ~32-54% range for pilot metatranscriptome samples
- Average aligned read length is near ~178 bp for 100 bp paired-end input
- All merged reads exceed the ~100 bp minimum length SAMSA requires for annotation
- Merged FASTQ output is accepted by downstream MG-RAST submission without format errors
- Joining metrics (aligned fraction, average length) are recorded, e.g. in preprocessing_stats.tsv, matching the paper's reported values

## Limitations

- MG-RAST annotation servers (downstream of merging) require queue waits of days to weeks, and SAMSA v1.0.0 has no install routine — each program must be run from the command line individually or via a bash batch script
- SAMSA2 (which uses PEAR instead of FLASH) still relies on Python2 for aggregation scripts and 'currently works mostly with Python3, although there may be some errors not yet caught'
- Merging cannot compensate for insufficient sequencing depth — each sample needs 40-50 million raw sequences to yield the 5-10 million annotated reads necessary for accurate abundance measures

## Evidence

- [methods] it performs cleaning and removal of adaptor contamination, and then aligns paired-end files (if specified): "it performs cleaning and removal of adaptor contamination, and then aligns paired-end files (if specified)"
- [abstract] Sequences need to be either long reads (longer than 100 bp) or joined paired-end reads.: "Sequences need to be either long reads (longer than 100 bp) or joined paired-end reads."
- [readme] SAMSA_pre_annotation_pipeline.py - runs Trimmomatic, FLASH (if paired-end files), prepares raw sequences for upload to MG-RAST: "SAMSA_pre_annotation_pipeline.py - runs Trimmomatic, FLASH (if paired-end files), prepares raw sequences for upload to MG-RAST"
- [readme] PEAR - merges two paired-end FASTQ sequencing files together to make a single file of extended fragments.: [evidence span withheld — anchor status: external_doc]
- [readme] the forward read and the reverse (complement) read are concatenated with a string of 20 Ns in the middle: [evidence span withheld — anchor status: external_doc]

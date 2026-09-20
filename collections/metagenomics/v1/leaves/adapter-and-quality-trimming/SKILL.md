---
name: adapter-and-quality-trimming
description: Use when raw paired-end RNA-seq/metatranscriptome FASTQ files have been
  merged (e.g. with PEAR) and still contain adaptor contamination and low-quality
  bases that would hurt downstream DIAMOND annotation.
when_to_use_negative:
- Input reads have not yet been merged from paired-end form (run PEAR first, as SAMSA2
  recommends PEAR for paired-end data)
- Sequences are not raw FASTQ reads but already-annotated results or abundance summary
  tables
- The goal is rRNA depletion rather than quality/adapter cleaning (that is SortMeRNA's
  role in SAMSA2)
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3308
tools:
- name: Trimmomatic
  role: Flexible read cleaner used to remove low-quality sequences and adaptor contamination
    from merged FASTQ reads
  repo: http://www.usadellab.org/cms/?page=trimmomatic
- name: SAMSA2
  role: Standalone metatranscriptome pipeline whose preprocessing (master_script.bash)
    orchestrates and parameterizes the Trimmomatic trimming step
  repo: https://github.com/transcript/samsa2
- name: PEAR
  role: Upstream tool that merges paired-end FASTQ files into the single file of extended
    fragments that Trimmomatic then trims
  repo: https://sco.h-its.org/exelixis/web/software/pear/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-018-2189-z
    title: 'SAMSA2: a standalone metatranscriptome analysis pipeline'
  source_content_hash: sha256:9131ac426c3e189a08ee2ce074298f785c536786705b54b93285c5b666bf508b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PEAR
  - SAMSA2
  - SortMeRNA
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# adapter-and-quality-trimming

## Summary

Cleaning of merged metatranscriptome FASTQ reads with Trimmomatic to remove adaptor contamination and low-quality sequences, as performed in the SAMSA2 pipeline between PEAR merging and SortMeRNA rRNA filtering.

## When to use

Use when raw paired-end RNA-seq/metatranscriptome FASTQ files have been merged (e.g. with PEAR) and still contain adaptor contamination and low-quality bases that would hurt downstream DIAMOND annotation.

## When NOT to use

- Input reads have not yet been merged from paired-end form (run PEAR first, as SAMSA2 recommends PEAR for paired-end data)
- Sequences are not raw FASTQ reads but already-annotated results or abundance summary tables
- The goal is rRNA depletion rather than quality/adapter cleaning (that is SortMeRNA's role in SAMSA2)

## Inputs

- PEAR-merged paired-end metatranscriptome FASTQ read files

## Outputs

- Adaptor- and low-quality-trimmed FASTQ read files carried forward to SortMeRNA rRNA filtering

## How to apply

Run Trimmomatic on the PEAR-merged FASTQ files so that low-quality sequences and adaptor contamination are removed before rRNA filtering and annotation. In SAMSA2's master_script.bash, adaptor removal is guided by a specific fasta file of adapter sequences, and all read and program locations plus options are specified in the first section of the script. In SAMSA2 v2.2.0 the step is multithreaded and writes a checkpoint entry when finished so the CPU-intensive step is skipped on reruns. The trimmed output feeds directly into the SortMeRNA ribosomal filtering step, whose non-rRNA reads become input to DIAMOND annotation.

## Related tools

- **Trimmomatic** (Flexible read cleaner used to remove low-quality sequences and adaptor contamination from merged FASTQ reads) — http://www.usadellab.org/cms/?page=trimmomatic
- **SAMSA2** (Standalone metatranscriptome pipeline whose preprocessing (master_script.bash) orchestrates and parameterizes the Trimmomatic trimming step) — https://github.com/transcript/samsa2
- **PEAR** (Upstream tool that merges paired-end FASTQ files into the single file of extended fragments that Trimmomatic then trims) — https://sco.h-its.org/exelixis/web/software/pear/

## Evaluation signals

- The step's output is a valid FASTQ file whose reads no longer contain adaptor contamination (checked against the adapter fasta file configured in the script)
- The SAMSA2 checkpoint file records the trimming step as finished, so a master_script rerun skips it
- Trimmed reads remain consistent in format with the merged input (single file of extended fragments) and carry forward to the SortMeRNA step
- Disk usage stays minimal because SAMSA2 removes unnecessary intermediate files once a step is performed

## Limitations

- Adaptor removal depends on a specific fasta file of adapter sequences supplied to Trimmomatic; a wrong or missing adapter fasta will leave contamination
- In the preserving-unmerged variant of the master script, unmerged reads are concatenated with 20 Ns between forward and reverse-complement reads, which changes the reads Trimmomatic sees
- SAMSA2 v2.2.0 relies on Python2 for aggregation scripts and was only tested on Linux, which constrains where the trimming pipeline can be run

## Evidence

- [readme] Trimmomatic is used for the removal of adaptor contamination and low-quality reads.: "Trimmomatic is used for the removal of adaptor contamination and low-quality reads."
- [readme] Trimmomatic removes adapter contamination according to a specific fasta file.: "Trimmomatic removes adapter contamination according to a specific fasta file."
- [readme] Multithreading added for PEAR, Trimmomatic, SortMeRNA: "Multithreading added for PEAR, Trimmomatic, SortMeRNA"
- [methods] the first 3 of 4 steps in the SAMSA2 pipeline (preprocessing, annotation, aggregation): "the first 3 of 4 steps in the SAMSA2 pipeline (preprocessing, annotation, aggregation)"

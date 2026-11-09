---
name: stool-host-transcriptome-contamination-assessment
description: Use when raw paired-end stool metatranscriptome FASTQ files (ribodepleted
  or poly(A)-enriched) need screening for human/host read contamination, or when deciding
  if host transcriptomes are recoverable from a gut microbiome sequencing run.
when_to_use_negative:
- Input is already a feature/abundance table or MG-RAST annotation output rather than
  raw FASTQ reads
- Sequencing reads are short single-end reads under 100 bp, since SAMSA requires long
  reads or joined paired-end reads
- The goal is within-sample microbiome functional annotation rather than host contamination
  assessment — MG-RAST is not tailored to work with eukaryotic organisms
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0080
tools:
- name: SAMSA
  role: Pipeline providing the preprocessing stage (cleaning, adapter removal, paired-end
    joining) whose cleaned reads are then aligned against the human genome for host
    contamination counting
  repo: https://github.com/transcript/SAMSA
- name: Trimmomatic
  role: Performs cleaning and removal of adaptor contamination on each FASTQ file
    before host alignment
- name: FLASH
  role: Joins cleaned paired-end reads so sequences exceed the 100 bp minimum required
    by SAMSA
- name: Python
  role: Runs the SAMSA preprocessing scripts and tabulates human-matching read counts
    and percentages
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-016-1270-8
    title: 'SAMSA: a comprehensive metatranscriptome analysis pipeline'
  source_content_hash: sha256:e169e2dd9f2358fec1f302dc51627df8ec0c4296aacaab80ff886de4811b2adf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/transcript/SAMSA
  tools:
  - FLASH
  - Python
  - SAMSA
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# stool-host-transcriptome-contamination-assessment

## Summary

Quantifies how many sequenced stool metatranscriptome reads align to the human genome, using SAMSA preprocessing (Trimmomatic cleaning, FLASH paired-end joining) followed by human reference alignment, to judge whether host transcriptomes can be recovered from ribodepleted or poly(A)-enriched stool libraries.

## When to use

Use when raw paired-end stool metatranscriptome FASTQ files (ribodepleted or poly(A)-enriched) need screening for human/host read contamination, or when deciding if host transcriptomes are recoverable from a gut microbiome sequencing run.

## When NOT to use

- Input is already a feature/abundance table or MG-RAST annotation output rather than raw FASTQ reads
- Sequencing reads are short single-end reads under 100 bp, since SAMSA requires long reads or joined paired-end reads
- The goal is within-sample microbiome functional annotation rather than host contamination assessment — MG-RAST is not tailored to work with eukaryotic organisms

## Inputs

- Raw paired-end FASTQ files from stool metatranscriptomes (ribodepleted, e.g. P1 replicates)
- Raw paired-end FASTQ files from poly(A)-enriched stool metatranscriptomes (e.g. P4 replicates)
- Human genome reference sequence for read alignment

## Outputs

- human_read_alignment_counts.tsv with per-replicate human-matching read counts (expected 417, 340, 2273, 2100)
- Per-sample percentage of total reads aligning to the human genome
- Cleaned and FLASH-joined intermediate FASTQ files from Trimmomatic/FLASH preprocessing

## How to apply

Retrieve the raw paired-end FASTQ files (e.g., ribodepleted P1 and poly(A)-enriched P4 replicates) and run SAMSA preprocessing: Trimmomatic performs cleaning and removal of adaptor contamination, then FLASH joins cleaned paired-end reads, since SAMSA requires sequences longer than 100 bp (long reads or joined paired-end reads). Align the resulting joined reads against the human genome reference to identify host-derived reads. Count human-matching reads per replicate and compute each as a percentage of total reads; in the source dataset, ribodepleted samples yielded only 417 and 340 human-matching reads (0.0019% and 0.0085%) versus 2273 and 2100 for poly(A)-enriched libraries (0.116% and 0.097%). Record the four counts (expected: 417, 340, 2273, 2100) in a human_read_alignment_counts.tsv table. The qualitative decision rule is that host transcriptomes cannot be reliably extracted when only a tiny fraction of total stool reads align to the human genome in both library-preparation conditions.

## Related tools

- **SAMSA** (Pipeline providing the preprocessing stage (cleaning, adapter removal, paired-end joining) whose cleaned reads are then aligned against the human genome for host contamination counting) — https://github.com/transcript/SAMSA
- **Trimmomatic** (Performs cleaning and removal of adaptor contamination on each FASTQ file before host alignment)
- **FLASH** (Joins cleaned paired-end reads so sequences exceed the 100 bp minimum required by SAMSA)
- **Python** (Runs the SAMSA preprocessing scripts and tabulates human-matching read counts and percentages)

## Examples

```
python SAMSA_pre_annotation_pipeline.py --input P1_R1.fastq --paired P1_R2.fastq  # Trimmomatic clean + FLASH join, then align joined reads against human genome
```

## Evaluation signals

- Tabulated human-matching read counts match the reported values exactly: 417 and 340 for ribodepleted, 2273 and 2100 for poly(A)-enriched
- Computed percentages fall at 0.0019% and 0.0085% (ribodepleted) and 0.116% and 0.097% (poly(A)-enriched) of total reads
- Qualitative criterion holds: only a tiny fraction of total stool reads align to the human genome in both library-preparation conditions, confirming host transcriptomes are not reliably recoverable
- Joined reads after FLASH exceed the 100 bp length threshold required by SAMSA
- All four P1/P2 replicates (two ribodepleted, two poly(A)) produce counts recorded in human_read_alignment_counts.tsv

## Limitations

- Host transcriptomes cannot be reliably extracted from stool metatranscriptomes sequenced using standard methods — the absolute number of human-matching reads is negligible in both conditions
- Ribosomal RNA depletion does not equally deplete ribosomes from all species within a sample, and remaining rRNA sequences should be discarded before further analysis
- SAMSA links to the MG-RAST server, whose annotation queue can be slow; the pipeline requires an internet connection and an MG-RAST account
- Each sample needs 40-50 million raw sequences to yield the 5-10 million annotated reads necessary for accurate abundance measures

## Evidence

- [methods] cleaning and removal of adaptor contamination, and then aligns paired-end files: "it performs cleaning and removal of adaptor contamination, and then aligns paired-end files (if specified)"
- [abstract] long reads or joined paired-end reads requirement: "Sequences need to be either long reads (longer than 100 bp) or joined paired-end reads."
- [readme] SAMSA input format and preprocessing flow: "Starting with a raw sequenced file in fastq format, it performs cleaning
and removal of adaptor contamination, and then aligns paired-end files (if specified)."
- [abstract] recommended stool metatranscriptome sequencing design: "We further recommend that stool metatranscriptomes be ribodepleted and sequenced in a 100 bp paired end format with a minimum of 40 million reads per sample."
- [discussion] discard residual rRNA before analysis: "Therefore, we recommend discarding all remaining ribosomal reads before performing further analysis on these samples."

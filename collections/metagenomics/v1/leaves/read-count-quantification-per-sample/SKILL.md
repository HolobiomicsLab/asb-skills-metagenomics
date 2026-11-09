---
name: read-count-quantification-per-sample
description: Use when raw paired-end FASTQ metatranscriptome reads must be reduced
  to per-sample counts of reads matching a reference genome (e.g., host/human) to
  test whether host transcriptomes are recoverable.
when_to_use_negative:
- Input is already a per-sample annotation or abundance summary table produced by
  analysis_counter.py or DIAMOND_analysis_counter.py — counting has already been done.
- Input sequences are short single-end reads shorter than 100 bp — SAMSA requires
  long reads or joined paired-end reads longer than 100 bp.
- The goal is organism/function annotation of the microbiome via MG-RAST or DIAMOND
  rather than counting reads against a host/reference genome.
edam_operation: http://edamontology.org/operation_0236
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Trimmomatic
  role: SAMSA dependency used for cleaning and adapter removal on each FASTQ file
    before joining and counting
  repo: https://github.com/transcript/SAMSA
- name: FLASH
  role: joins cleaned paired-end reads into long reads so that paired-end files can
    be aligned for read counting
  repo: https://github.com/transcript/SAMSA
- name: SAMSA
  role: pipeline wrapper whose pre-annotation stage (SAMSA_pre_annotation_pipeline.py)
    runs Trimmomatic and FLASH before alignment
  repo: https://github.com/transcript/SAMSA
- name: Python 2.7
  role: environment in which SAMSA aggregation and counting scripts run
  repo: https://github.com/transcript/SAMSA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-016-1270-8
    title: 'SAMSA: a comprehensive metatranscriptome analysis pipeline'
  source_content_hash: sha256:7043a7fb3c3d57b0208afeefc23cfcb80a598c3e95910a30640dc96f57e4481c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/transcript/SAMSA
  tools:
  - FLASH
  - Python 2.7
  - SAMSA
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_0236
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# read-count-quantification-per-sample

## Summary

Quantify per-sample read counts against a reference genome by cleaning paired-end FASTQ metatranscriptome reads with Trimmomatic, joining mates with FLASH, aligning to the reference (e.g., human genome), and tabulating matched read counts and percentages per replicate.

## When to use

Use when raw paired-end FASTQ metatranscriptome reads must be reduced to per-sample counts of reads matching a reference genome (e.g., host/human) to test whether host transcriptomes are recoverable.

## When NOT to use

- Input is already a per-sample annotation or abundance summary table produced by analysis_counter.py or DIAMOND_analysis_counter.py — counting has already been done.
- Input sequences are short single-end reads shorter than 100 bp — SAMSA requires long reads or joined paired-end reads longer than 100 bp.
- The goal is organism/function annotation of the microbiome via MG-RAST or DIAMOND rather than counting reads against a host/reference genome.

## Inputs

- raw paired-end FASTQ files for each sample replicate (ribodepleted and poly(A)-enriched stool metatranscriptomes)
- cleaned and FLASH-joined read files (reads longer than 100 bp)
- reference genome to align against (e.g., human genome)

## Outputs

- per-replicate read counts matching the reference genome (e.g., 417, 340, 2273, 2100)
- per-replicate percentages of matching reads relative to total reads (e.g., 0.0019-0.116%)
- a tab-delimited table such as human_read_alignment_counts.tsv

## How to apply

Start from the raw paired-end FASTQ files for each replicate (e.g., two ribodepleted P1 and two poly(A)-enriched P4 stool metatranscriptomes) and clean them with Trimmomatic, the SAMSA pipeline's adapter-removal and quality step. Join the cleaned paired-end reads with FLASH, because SAMSA aligns paired-end files only after joining, and sequences must be long reads or joined paired-end reads longer than 100 bp. Align the resulting joined reads against the human genome reference to identify host-derived reads. Count reads matching the reference for each replicate and compute each count as a percentage of total reads; write the counts to a table such as human_read_alignment_counts.tsv. Validate by checking that the tabulated counts reproduce expected values (e.g., 417 and 340 ribodepleted; 2273 and 2100 poly(A)-enriched) and that the percentages are a tiny fraction of total reads, confirming host transcriptomes cannot be reliably extracted.

## Related tools

- **Trimmomatic** (SAMSA dependency used for cleaning and adapter removal on each FASTQ file before joining and counting) — https://github.com/transcript/SAMSA
- **FLASH** (joins cleaned paired-end reads into long reads so that paired-end files can be aligned for read counting) — https://github.com/transcript/SAMSA
- **SAMSA** (pipeline wrapper whose pre-annotation stage (SAMSA_pre_annotation_pipeline.py) runs Trimmomatic and FLASH before alignment) — https://github.com/transcript/SAMSA
- **Python 2.7** (environment in which SAMSA aggregation and counting scripts run) — https://github.com/transcript/SAMSA

## Examples

```
python SAMSA_pre_annotation_pipeline.py -usage
```

## Evaluation signals

- Tabulated human-matching read counts must match the reported values: 417 and 340 for ribodepleted, 2273 and 2100 for poly(A)-enriched.
- Matching reads must be a tiny fraction of total reads in both library-preparation conditions (e.g., 0.0019-0.116%), confirming host transcriptomes are not reliably recoverable from stool.
- Joined read lengths must exceed 100 bp, as SAMSA requires long reads or joined paired-end reads.
- Each of the four replicates (two P1, two P4) yields its own count and percentage row in the output TSV.
- Output file is a valid tab-delimited table (e.g., human_read_alignment_counts.tsv) with one row per replicate.

## Limitations

- Stool metatranscriptomes sequenced with standard methods yield too few human-matching reads for reliable host transcriptome recovery, so this skill quantifies host signal rather than enabling downstream host analysis.
- Ribosomal RNA depletion does not equally deplete ribosomes from all species within a sample, so remaining rRNA sequences should be discarded before further analysis.
- SAMSA version 1.0.0 has no install routine; each program must be run individually from the command line or via a bash batch script, and MG-RAST annotation queues can delay processing by days to weeks.

## Evidence

- [abstract] Sequences need to be either long reads (longer than 100 bp) or joined paired-end reads.: "Sequences need to be either long reads (longer than 100 bp) or joined paired-end reads."
- [methods] it performs cleaning and removal of adaptor contamination, and then aligns paired-end files (if specified): "it performs cleaning and removal of adaptor contamination, and then aligns paired-end files (if specified)"
- [abstract] Each sample needs 40-50 million raw sequences, expected to yield 5-10 million annotated reads: "Each sample needs 40-50 million raw sequences, which can be expected to yield the 5-10 million annotated reads necessary for accurate abundance measures."
- [readme] SAMSA_pre_annotation_pipeline.py runs Trimmomatic and FLASH: "SAMSA_pre_annotation_pipeline.py - runs Trimmomatic, FLASH (if paired-end files), prepares raw sequences for upload to MG-RAST"

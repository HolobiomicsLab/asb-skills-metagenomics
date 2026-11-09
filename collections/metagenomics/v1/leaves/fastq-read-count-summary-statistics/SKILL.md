---
name: fastq-read-count-summary-statistics
description: Use when you have run a MetaLAFFA QC step (e.g., Bowtie 2 host filtering
  against the hs37d5 reference) and need per-sample read counts plus average read
  length and base-quality summaries to document how many reads were removed.
when_to_use_negative:
- Your input is already a reduced result (gene counts, KO/pathway abundances) rather
  than FASTQ reads — there are no per-read counts to summarize.
- You need to modify reads (e.g., Trimmomatic MAXINFO quality trimming or PICARD MarkDuplicates
  duplicate removal) — this skill only summarizes counts, lengths, and qualities.
- You want the percentage of reads with a match in the UniRef90 database or per-gene
  abundances — that comes from the read mapping and gene counting phase, not per-step
  FASTQ summaries.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: MetaLAFFA
  role: Pipeline whose per-step summarizing functionality ('Summarizing results of
    steps in MetaLAFFA') computes per-sample FASTQ read counts, average read length,
    and average base quality
  repo: https://github.com/borenstein-lab/MetaLAFFA
- name: Bowtie 2
  role: Aligns reads to the hs37d5 human reference with 1000 Genomes decoys during
    host filtering; the human reads it identifies are what the summary statistics
    record as removed
- name: Snakemake
  role: Implements the MetaLAFFA pipeline steps whose per-step summary statistics
    are generated
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-020-03815-9
    title: 'MetaLAFFA: a flexible, end-to-end, distributed computing-compatible metagenomic
      functional annotation pipeline'
  source_content_hash: sha256:82425e98249c4596f27d141a6c05af66287c93f5d74f2351a70b85d6203e6117
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/borenstein-lab/MetaLAFFA
  tools:
  - Bowtie 2
  - Conda
  - MetaLAFFA
  - Snakemake
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# fastq-read-count-summary-statistics

## Summary

Computes per-sample FASTQ summary statistics — number of reads, average read length, and average base quality — at MetaLAFFA pipeline steps such as host read filtering, so users can see exactly how many reads each quality-control operation removed and judge pre-processing quality.

## When to use

Use when you have run a MetaLAFFA QC step (e.g., Bowtie 2 host filtering against the hs37d5 reference) and need per-sample read counts plus average read length and base-quality summaries to document how many reads were removed.

## When NOT to use

- Your input is already a reduced result (gene counts, KO/pathway abundances) rather than FASTQ reads — there are no per-read counts to summarize.
- You need to modify reads (e.g., Trimmomatic MAXINFO quality trimming or PICARD MarkDuplicates duplicate removal) — this skill only summarizes counts, lengths, and qualities.
- You want the percentage of reads with a match in the UniRef90 database or per-gene abundances — that comes from the read mapping and gene counting phase, not per-step FASTQ summaries.

## Inputs

- Input FASTQ reads named per MetaLAFFA convention (<sample>.R1.fastq.gz, <sample>.R2.fastq.gz, <sample>.S.fastq.gz)
- Filtered FASTQ output from a pipeline step (e.g., reads surviving Bowtie 2 alignment against the hs37d5 + 1000 Genomes decoy index)

## Outputs

- Per-sample summary statistics table reporting number of reads, average read length, and average base quality for input and filtered FASTQs
- Per-step record of reads removed (e.g., number of host reads removed during host filtering)

## How to apply

After each pipeline step — e.g., host read filtering, where Bowtie 2 maps reads to the hs37d5 human genome reference with 1000 Genomes decoy sequences and reads whose match qualities exceed the filtering threshold are discarded — MetaLAFFA computes per-sample summary statistics over both the input and the filtered FASTQs. The statistics are the number of reads, average read length, and average base quality, tabulated per step so the pipeline records how many reads each operation removed (e.g., the number of host reads removed for SRS017307). Input files must follow MetaLAFFA's naming convention (<sample>.R1.fastq.gz, <sample>.R2.fastq.gz, <sample>.S.fastq.gz) so per-sample tables join correctly. Interpretation is context-dependent: for the post-HMP quality-control HMP stool samples (90–130 million reads), the article found minimal reads discarded during QC, so near-zero deltas are expected rather than a failure. Validate by confirming the host-filtered FASTQ exists and the summary table reports the read counts removed during host filtering.

## Related tools

- **MetaLAFFA** (Pipeline whose per-step summarizing functionality ('Summarizing results of steps in MetaLAFFA') computes per-sample FASTQ read counts, average read length, and average base quality) — https://github.com/borenstein-lab/MetaLAFFA
- **Bowtie 2** (Aligns reads to the hs37d5 human reference with 1000 Genomes decoys during host filtering; the human reads it identifies are what the summary statistics record as removed)
- **Snakemake** (Implements the MetaLAFFA pipeline steps whose per-step summary statistics are generated)

## Evaluation signals

- A per-step summary table exists and reports the read counts removed during host filtering for the sample (validated for SRS017307 in the task workflow).
- Read counts in the summary equal the number of FASTQ records in the corresponding input and host-filtered FASTQ files.
- For post-HMP quality-controlled stool samples (90–130 million reads), removed-read counts during QC are minimal; large losses indicate a misconfigured step.
- Filtered counts are less than or equal to input counts, and average read length/base quality are reported for both input and filtered FASTQs.
- The host-filtered FASTQ output exists before summaries are generated, and files follow the <sample>.R1/R2/S.fastq.gz naming convention.

## Limitations

- When input data are already post-HMP quality control, only minimal reads are discarded during QC, so a small removed-read count is normal and must not be read as step failure.
- Input FASTQs must follow MetaLAFFA's naming convention or per-sample summary statistics cannot be matched to samples.
- Default statistics reflect filtering against the hs37d5 human reference plus 1000 Genomes decoy sequences; substituting another host database changes what the removed-read counts represent.

## Evidence

- [abstract] MetaLAFFA also generates summary statistics for each step in the pipeline so that users can better understand pre-processing and annotation quality: "MetaLAFFA also generates summary statistics for each step in the pipeline so that users can better understand pre-processing and annotation quality"
- [intro] MetaLAFFA removes host reads by mapping reads to the hs37d5 human genome reference... using Bowtie 2 [12] and then discarding any reads identified as human: [evidence span withheld — anchor status: not_found]
- [methods] Host read filtering is usually performed by aligning reads to one or more reference genomes for the host species and removing reads with match qualities above a certain threshold.: "Host read filtering is usually performed by aligning reads to one or more reference genomes for the host species and removing reads with match qualities above a certain threshold."
- [results] Since these data files are post-HMP quality control, minimal reads were discarded from each sample during MetaLAFFA's quality control phase.: [evidence span withheld — anchor status: not_found]
- [readme] This process consists of several steps to go from raw FASTQs (with sequencing adapters removed) to functional profiles: "This process consists of several steps to go from raw FASTQs (with sequencing adapters removed) to functional profiles"
- [readme] 10. Summarizing results of steps in MetaLAFFA: "10. Summarizing results of steps in MetaLAFFA"

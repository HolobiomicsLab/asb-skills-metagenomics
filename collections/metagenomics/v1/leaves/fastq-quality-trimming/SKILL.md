---
name: fastq-quality-trimming
description: Use when shotgun metagenomic FASTQs (host-filtered paired and singleton
  .fastq.gz) must be quality-trimmed before read mapping, and low-quality bases at
  read ends and overly short reads need removal.
when_to_use_negative:
- Input data are already post-quality-control (e.g. post-HMP QC), where minimal additional
  trimming is expected and the step may add little value
- Reads still require host read removal or duplicate read filtering first — MetaLAFFA
  orders host filtering (Bowtie 2 vs. hs37d5) and duplicate filtering (PICARD MarkDuplicates)
  before quality trimming
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: Trimmomatic
  role: Performs quality trimming using the MAXINFO criterion and filters out reads
    below the minimum length threshold
  repo: https://github.com/borenstein-lab/MetaLAFFA
- name: MetaLAFFA
  role: Wraps the quality_filter step, orchestrates Trimmomatic execution, and generates
    per-sample summary statistics for the trimmed FASTQs
  repo: https://github.com/borenstein-lab/MetaLAFFA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-020-03815-9
    title: 'MetaLAFFA: a flexible, end-to-end, distributed computing-compatible metagenomic
      functional annotation pipeline'
  source_content_hash: sha256:ca3e9e6425be57d4e462971df8ede8b2b095abf96854c6284e8b49cfe3294cf4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/borenstein-lab/MetaLAFFA
  tools:
  - MetaLAFFA
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# fastq-quality-trimming

## Summary

MetaLAFFA's quality trimming and filtering step uses Trimmomatic with the MAXINFO criterion to trim low-quality bases from read ends and discard reads falling below a 60bp minimum length threshold, preparing quality-controlled FASTQs for read mapping.

## When to use

Use when shotgun metagenomic FASTQs (host-filtered paired and singleton .fastq.gz) must be quality-trimmed before read mapping, and low-quality bases at read ends and overly short reads need removal.

## When NOT to use

- Input data are already post-quality-control (e.g. post-HMP QC), where minimal additional trimming is expected and the step may add little value
- Reads still require host read removal or duplicate read filtering first — MetaLAFFA orders host filtering (Bowtie 2 vs. hs37d5) and duplicate filtering (PICARD MarkDuplicates) before quality trimming

## Inputs

- Host-filtered paired-end FASTQ files (<sample_id>.<type_id>.fastq.gz)
- Host-filtered singleton FASTQ files (<sample_id>.<type_id>.fastq.gz)

## Outputs

- Compressed quality-filtered FASTQ files per sample
- Per-sample summary statistics (number of reads, average read length, average base quality) merged into the step summary table

## How to apply

Load the host-filtered paired and singleton FASTQ files following the <sample_id>.<type_id>.fastq.gz naming convention from the initial data directory. Run Trimmomatic via the MetaLAFFA quality_filter step using its maximum information (MAXINFO) criterion, which balances base quality and read length to trim low-quality bases from one or both ends of each read. Discard trimmed reads shorter than the 60bp minimum length threshold. Write the quality-filtered FASTQs in compressed form to the pipeline output directory, then generate per-sample summary statistics (number of reads, average read length, average base quality) via MetaLAFFA's automatic summary statistic generation and merge them into the step summary table.

## Related tools

- **Trimmomatic** (Performs quality trimming using the MAXINFO criterion and filters out reads below the minimum length threshold) — https://github.com/borenstein-lab/MetaLAFFA
- **MetaLAFFA** (Wraps the quality_filter step, orchestrates Trimmomatic execution, and generates per-sample summary statistics for the trimmed FASTQs) — https://github.com/borenstein-lab/MetaLAFFA

## Examples

```
conda activate metalaffa && snakemake quality_filter  # Trimmomatic MAXINFO trimming, min length 60bp, on <sample_id>.<type_id>.fastq.gz inputs
```

## Evaluation signals

- Quality-filtered FASTQ files exist in the pipeline output directory in compressed form
- Summary table reports post-trim average base quality and average read length for each sample
- Reads shorter than the 60bp minimum length threshold are absent from the output FASTQs
- Number of reads after trimming is consistent with trimming of low-quality bases from read ends without discarding excessive reads

## Limitations

- For data already subjected to upstream quality control (e.g. HMP QC), minimal reads are discarded during this phase, so the step's benefit may be limited
- The 60bp minimum length threshold is a default; overly aggressive trimming can shorten reads below the threshold and discard them entirely

## Evidence

- [intro] Quality trimming and filtering step description: "quality trimming and filtering, which removes low-quality bases from the ends of reads and then discards reads that are too short. This is performed using Trimmomatic [2] and the MAXINFO trimming"
- [methods] Methods description of trimming and length threshold: "trimming bases with quality below a certain threshold from one or both ends of a read, as well as removing any reads that fall below a specific length threshold"
- [readme] Pipeline overview listing quality trimming as step 3: "1.  Host read filtering (e.g. removing human DNA)
2.  Duplicate read filtering
3.  Quality trimming and filtering"
- [results] Results note on already-QC'd data: "Since these data files are post-HMP quality control, minimal reads were discarded from each sample during MetaLAFFA’s quality control phase."
- [abstract] Summary statistics generation: "MetaLAFFA also generates summary statistics for each step in the pipeline so that users can better understand pre-processing and annotation quality"

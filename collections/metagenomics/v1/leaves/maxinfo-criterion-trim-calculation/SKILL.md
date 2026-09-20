---
name: maxinfo-criterion-trim-calculation
description: Use when quality-controlled paired/singleton FASTQs (e.g. host-filtered
  <sample_id>.<type_id>.fastq.gz files) still contain low-quality read ends and short
  reads that must be trimmed before read-to-gene mapping.
when_to_use_negative:
- Input FASTQs are already post-HMP quality control — in the article's HMP stool samples,
  minimal reads were discarded during this phase, so heavy trimming is unnecessary
- The goal is host read removal or duplicate removal — MetaLAFFA performs those before
  this step using Bowtie 2/hs37d5 and PICARD MarkDuplicates, respectively
- You need adapter removal — MetaLAFFA expects raw FASTQs with sequencing adapters
  already removed as pipeline input
edam_operation: http://edamontology.org/operation_3237
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3174
tools:
- name: Trimmomatic
  role: Performs the quality trimming (MAXINFO criterion) and length-based read filtering
- name: MetaLAFFA
  role: 'Pipeline wrapper: hosts the quality_filter step, applies the 60bp minimum
    length threshold, and generates per-step summary statistics'
  repo: https://github.com/borenstein-lab/MetaLAFFA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-020-03815-9
    title: 'MetaLAFFA: a flexible, end-to-end, distributed computing-compatible metagenomic
      functional annotation pipeline'
  source_content_hash: sha256:c49a9c73dc54d2ecdb8bdbfaffb943b631d4c61268babef2efeda24821da9de5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/borenstein-lab/MetaLAFFA
  tools:
  - MetaLAFFA
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_3237
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# maxinfo-criterion-trim-calculation

## Summary

Quality trimming and filtering of shotgun metagenomic FASTQs as implemented in MetaLAFFA: Trimmomatic with the MAXINFO criterion trims low-quality bases from read ends, and reads shorter than the 60bp length threshold are discarded, with per-sample summary statistics generated afterward.

## When to use

Use when quality-controlled paired/singleton FASTQs (e.g. host-filtered <sample_id>.<type_id>.fastq.gz files) still contain low-quality read ends and short reads that must be trimmed before read-to-gene mapping.

## When NOT to use

- Input FASTQs are already post-HMP quality control — in the article's HMP stool samples, minimal reads were discarded during this phase, so heavy trimming is unnecessary
- The goal is host read removal or duplicate removal — MetaLAFFA performs those before this step using Bowtie 2/hs37d5 and PICARD MarkDuplicates, respectively
- You need adapter removal — MetaLAFFA expects raw FASTQs with sequencing adapters already removed as pipeline input

## Inputs

- host-filtered paired-end FASTQ files (<sample_id>.paired.1.fastq.gz, <sample_id>.paired.2.fastq.gz)
- singleton FASTQ files (<sample_id>.singleton.fastq.gz) after host and duplicate read filtering

## Outputs

- quality-filtered compressed FASTQ files in the pipeline output directory
- per-sample summary statistics (number of reads, average read length, average base quality post-trim) merged into the step summary table

## How to apply

Load the host-filtered paired and singleton compressed FASTQs following the <sample_id>.<type_id>.fastq.gz naming convention from the initial data directory. Run Trimmomatic via the MetaLAFFA quality_filter step using the MAXINFO trimming criterion, which balances base quality and read length to decide how many bases to remove from one or both ends of each read. Discard any trimmed reads falling below the 60bp minimum length threshold. Write the quality-filtered FASTQs in compressed form to the pipeline output directory, then let MetaLAFFA's automatic summary statistic generation compute per-sample number of reads, average read length, and average base quality, merging them into the step summary table. Rationale: MetaLAFFA incorporates this pre-processing into the pipeline so users need not run common quality control separately, and this step sits after host and duplicate read filtering but before mapping reads to genes with DIAMOND.

## Related tools

- **Trimmomatic** (Performs the quality trimming (MAXINFO criterion) and length-based read filtering)
- **MetaLAFFA** (Pipeline wrapper: hosts the quality_filter step, applies the 60bp minimum length threshold, and generates per-step summary statistics) — https://github.com/borenstein-lab/MetaLAFFA

## Examples

```
trimmomatic PE <sample_id>.paired.1.fastq.gz <sample_id>.paired.2.fastq.gz <out>/paired.trimmed.1.fastq.gz <out>/unpaired.1.fastq.gz <out>/paired.trimmed.2.fastq.gz <out>/unpaired.2.fastq.gz MAXINFO MINLEN:60
```

## Evaluation signals

- The trimmed FASTQ exists in the output directory and is gzipped
- The step summary table reports post-trim average base quality and average read length for each sample
- No reads shorter than 60bp remain in the quality-filtered FASTQs
- Average base quality in the output FASTQs is higher than in the host-filtered input
- Read counts in the trimmed FASTQs are less than or equal to input read counts, with the decrease consistent with short-read discards

## Limitations

- The MAXINFO criterion and minimum length threshold (60bp by default) are configurable per step in config/steps/*.py, so default behavior may differ from a user's customized configuration
- If input data were already quality filtered (e.g. post-HMP quality control), minimal reads are discarded and the step adds little value
- Trimming too aggressively with MAXINFO can discard reads entirely, reducing downstream gene-mapping coverage

## Evidence

- [intro] Quality trimming and filtering using Trimmomatic with MAXINFO criterion: "quality trimming and filtering, which removes low-quality bases from the ends of reads and then discards reads that are too short. This is performed using Trimmomatic [2] and the MAXINFO trimming"
- [methods] Methods description of the filter step: "trimming bases with quality below a certain threshold from one or both ends of a read, as well as removing any reads that fall below a specific length threshold"
- [abstract] Summary statistics rationale: "MetaLAFFA also generates summary statistics for each step in the pipeline so that users can better understand pre-processing and annotation quality"
- [readme] Pipeline position of the step in the README: "several steps to go from raw FASTQs (with sequencing adapters removed) to functional profiles"
- [results] Already quality-controlled data caveat: "Since these data files are post-HMP quality control, minimal reads were discarded from each sample during MetaLAFFA’s quality control phase."

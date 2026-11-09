---
name: metagenomic-read-quality-filtering
description: Use when quality-controlling shotgun metagenomic FASTQs as part of an
  end-to-end functional annotation pipeline, after host and duplicate read filtering
  and before mapping reads to a gene database.
when_to_use_negative:
- Input reads have not yet had host reads and duplicate reads removed — MetaLAFFA
  orders host filtering and duplicate filtering before quality trimming
- Input is already a functional profile or gene abundance table — this skill operates
  on raw/host-filtered FASTQ read data only
- Input data have already undergone stringent external quality control (e.g. post-HMP
  quality control), where minimal reads would be discarded during this step
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0091
tools:
- name: Trimmomatic
  role: Performs the quality trimming (MAXINFO criterion, trimming low-quality bases
    from read ends) and length-based read filtering
- name: MetaLAFFA
  role: Wraps the quality_filter step within its Snakemake pipeline, applies the 60bp
    minimum-length threshold, and generates per-sample summary statistics for the
    step
  repo: https://github.com/borenstein-lab/MetaLAFFA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-020-03815-9
    title: 'MetaLAFFA: a flexible, end-to-end, distributed computing-compatible metagenomic
      functional annotation pipeline'
  source_content_hash: sha256:e9385e3b7f76b45b3249423424af30d5942f0a3093c04c3fcef5fd2479050f09
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/borenstein-lab/MetaLAFFA
  tools:
  - MetaLAFFA
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# metagenomic-read-quality-filtering

## Summary

MetaLAFFA's third pre-processing step trims low-quality bases from read ends and discards short reads using Trimmomatic's MAXINFO criterion, producing quality-filtered FASTQs plus per-sample summary statistics before read mapping.

## When to use

Use when quality-controlling shotgun metagenomic FASTQs as part of an end-to-end functional annotation pipeline, after host and duplicate read filtering and before mapping reads to a gene database.

## When NOT to use

- Input reads have not yet had host reads and duplicate reads removed — MetaLAFFA orders host filtering and duplicate filtering before quality trimming
- Input is already a functional profile or gene abundance table — this skill operates on raw/host-filtered FASTQ read data only
- Input data have already undergone stringent external quality control (e.g. post-HMP quality control), where minimal reads would be discarded during this step

## Inputs

- Host-filtered paired-end FASTQ files (<sample_id>.<type_id>.fastq.gz)
- Host-filtered singleton FASTQ files (<sample_id>.<type_id>.fastq.gz)

## Outputs

- Quality-filtered compressed FASTQ files (gzipped) in the pipeline output directory
- Per-sample summary statistics: number of reads, average read length, average base quality post-trimming

## How to apply

Load the host-filtered paired and singleton FASTQ files following the <sample_id>.<type_id>.fastq.gz naming convention. Run Trimmomatic via the MetaLAFFA quality_filter step using its maximum information (MAXINFO) criterion to trim low-quality bases from one or both ends of each read; the rationale is that trimming bases with quality below a threshold from read ends improves downstream read-to-gene mapping. Discard trimmed reads that fall below the 60bp minimum length threshold. Write the resulting quality-filtered FASTQs to the pipeline output directory in compressed form. Generate per-sample summary statistics (number of reads, average read length, average base quality post-trim) and merge them into the step summary table so pre-processing quality can be inspected.

## Related tools

- **Trimmomatic** (Performs the quality trimming (MAXINFO criterion, trimming low-quality bases from read ends) and length-based read filtering)
- **MetaLAFFA** (Wraps the quality_filter step within its Snakemake pipeline, applies the 60bp minimum-length threshold, and generates per-sample summary statistics for the step) — https://github.com/borenstein-lab/MetaLAFFA

## Evaluation signals

- Quality-filtered FASTQ files exist in the expected output location in compressed (.gz) form for each sample and read type
- The step summary table reports post-trim average base quality and average read length for each sample
- No reads shorter than the 60bp minimum length threshold remain in the output FASTQs
- Per-sample read counts in the summary statistics do not exceed the input host-filtered read counts
- Step order is respected: host filtering and duplicate read filtering outputs (not raw FASTQs) are the inputs to this step

## Limitations

- When input data are already post quality control (e.g. post-HMP quality control), minimal reads are discarded at this step, so it adds little value
- The 60bp minimum length and MAXINFO settings are MetaLAFFA defaults; per-step operating parameters are configurable via the Python config system (config/steps/*.py)
- Requires adapter removal upstream — the pipeline expects raw FASTQs with sequencing adapters already removed

## Evidence

- [intro] Quality trimming removes low-quality bases and discards short reads via Trimmomatic MAXINFO: "quality trimming and filtering, which removes low-quality bases from the ends of reads and then discards reads that are too short. This is performed using Trimmomatic [2] and the MAXINFO trimming"
- [methods] Methods define trimming and length-threshold filtering semantics: "trimming bases with quality below a certain threshold from one or both ends of a read, as well as removing any reads that fall below a specific length threshold"
- [abstract] Summary statistics support judging pre-processing quality: "MetaLAFFA also generates summary statistics for each step in the pipeline so that users can better understand pre-processing and annotation quality"
- [readme] Quality trimming and filtering is step 3 in the documented workflow: "1.  Host read filtering (e.g. removing human DNA)
2.  Duplicate read filtering
3.  Quality trimming and filtering
4.  Mapping reads to genes"

---
name: mg-rast-annotation-submission
description: Use when raw metatranscriptome reads have been cleaned (Trimmomatic)
  and optionally paired-end joined (FLASH), and organism/functional annotation via
  a public server is required before abundance aggregation.
when_to_use_negative:
- When offline/local annotation is required or MG-RAST queue waits of days to weeks
  are unacceptable — SAMSA2 with DIAMOND against a local RefSeq/SEED database is the
  alternative
- When the metatranscriptome is dominated by eukaryotic organisms — MG-RAST is not
  tailored to work with eukaryotic organisms, leading to increased mismatch rates
- When input files are already annotated (e.g. existing MG-RAST results) — this skill
  covers submission, not re-analysis; proceed directly to the aggregation phase
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3941
tools:
- name: SAMSA_pre_annotation_pipeline.py
  role: Runs Trimmomatic cleaning and FLASH paired-end joining to prepare raw sequences
    for upload to MG-RAST
  repo: github.com/transcript/SAMSA
- name: MG-RAST
  role: Annotation server (version 3.6) that performs organism-level and functional
    annotation of submitted metatranscriptome files
- name: MG-RAST_API_downloader.py
  role: Batch downloads of annotation results files from MG-RAST via API commands
  repo: github.com/transcript/SAMSA
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1186/s12859-016-1270-8
    title: 'SAMSA: a comprehensive metatranscriptome analysis pipeline'
  source_content_hash: sha256:0986f926e7ca4df9d7ba43b1f8452ae5d9fe024e5b94a2d6cb3e59f69a55e53d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FLASH
  - MG-RAST
  - MG-RAST_API_downloader.py
  - R
  - SAMSA
  - SAMSA_pre_annotation_pipeline.py
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3941
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# MG-RAST annotation submission

## Summary

Submit SAMSA-preprocessed metatranscriptome FASTQ files to the MG-RAST annotation server (version 3.6) for organism-level and functional annotation, then batch-download the tab-delimited annotation results for SAMSA aggregation.

## When to use

Use when raw metatranscriptome reads have been cleaned (Trimmomatic) and optionally paired-end joined (FLASH), and organism/functional annotation via a public server is required before abundance aggregation.

## When NOT to use

- When offline/local annotation is required or MG-RAST queue waits of days to weeks are unacceptable — SAMSA2 with DIAMOND against a local RefSeq/SEED database is the alternative
- When the metatranscriptome is dominated by eukaryotic organisms — MG-RAST is not tailored to work with eukaryotic organisms, leading to increased mismatch rates
- When input files are already annotated (e.g. existing MG-RAST results) — this skill covers submission, not re-analysis; proceed directly to the aggregation phase

## Inputs

- Preprocessed metatranscriptome FASTQ files (adaptor-contamination removed by Trimmomatic, paired-end reads joined by FLASH)
- MG-RAST user account credentials

## Outputs

- Tab-delimited MG-RAST annotation results files per sample (organism-level and functional annotations)
- MG-RAST internal m5nr IDs usable by analysis_counter_w-md5s.py and md5-R-results_query.py

## How to apply

First run SAMSA_pre_annotation_pipeline.py, which performs cleaning and adaptor-contamination removal with Trimmomatic and aligns paired-end files with FLASH if specified; only then submit the prepared files to MG-RAST for annotation. An MG-RAST account is required, and the pipeline was used with MG-RAST version 3.6 (current as of May 2016). After annotation completes, batch-download the annotation results files using MG-RAST_API_downloader.py via API commands. The downloaded tab-delimited annotation outputs feed directly into analysis_counter.py-style aggregation, so preserve the MG-RAST output format. Plan for server latency: MG-RAST queues can delay processing by days to weeks, so batch submissions and downloads should be scripted rather than run interactively.

## Related tools

- **SAMSA_pre_annotation_pipeline.py** (Runs Trimmomatic cleaning and FLASH paired-end joining to prepare raw sequences for upload to MG-RAST) — github.com/transcript/SAMSA
- **MG-RAST** (Annotation server (version 3.6) that performs organism-level and functional annotation of submitted metatranscriptome files)
- **MG-RAST_API_downloader.py** (Batch downloads of annotation results files from MG-RAST via API commands) — github.com/transcript/SAMSA

## Examples

```
python SAMSA_pre_annotation_pipeline.py  # prepare FASTQs; upload to MG-RAST v3.6, then: python MG-RAST_API_downloader.py
```

## Evaluation signals

- All preprocessed samples appear as annotated jobs in the MG-RAST account after the queue wait resolves
- Downloaded annotation results files are tab-delimited and parse line-by-line in the aggregation step (analysis_counter.py)
- Reference match percentage is in a plausible range — the pilot dataset achieved 77.2% paired-end vs 65.9% single-end reference database matches
- A substantial fraction of raw reads survive QC and annotation — the paper expects 40-50 million raw sequences per sample to yield 5-10 million annotated reads
- analysis_counter_w-md5s.py can recover m5nr IDs from the downloaded files, confirming MG-RAST output format was preserved

## Limitations

- MG-RAST annotation servers require queue waits of days to weeks, making the annotation step slow
- MG-RAST is not tailored to work with eukaryotic organisms, which may increase mismatch rates
- An internet connection and an MG-RAST account are mandatory; there is no local annotation fallback in SAMSA v1
- Sequences need to be either long reads (longer than 100 bp) or joined paired-end reads for reliable annotation

## Evidence

- [methods] The files must then be submitted to MG-RAST for annotation.: "The files must then be submitted to MG-RAST for annotation."
- [readme] SAMSA links with MG-RAST, and is currently used with version 3.6 - current as of May 2016.: "SAMSA links with MG-RAST, and is currently used with version 3.6 - current as of May 
2016."
- [readme] MG-RAST_API_downloader.py - allows for batch downloads of annotation results files from MG-RAST via API commands: "MG-RAST_API_downloader.py - allows for batch downloads of annotation results files from MG-RAST via API commands"
- [discussion] MG-RAST's annotation servers require waiting in a queue, and processing may be slowed by days to weeks.: "MG-RAST’s annotation servers require waiting in a queue, and processing may be slowed by days to weeks."
- [readme] Users of the SAMSA pipeline must have an MG-RAST account for assigning annotations.: "Users of the SAMSA pipeline must have an MG-RAST account
for assigning annotations."

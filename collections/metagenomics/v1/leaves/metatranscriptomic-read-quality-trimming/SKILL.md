---
name: metatranscriptomic-read-quality-trimming
description: Use when starting from raw paired-end metatranscriptomic FASTQ files
  (e.g. ribodepleted or poly(A)-enriched stool RNA-seq) that need adapter trimming
  and quality cleaning before read joining and downstream annotation.
when_to_use_negative:
- Input reads are already cleaned, adapter-trimmed, and merged, or are not FASTQ raw
  sequence data
- Sequences are short single-end reads (<100 bp) that cannot meet SAMSA's minimum
  read length requirement even after merging
- The goal is rRNA removal rather than adapter/quality trimming — use a dedicated
  ribosomal filter (SortMeRNA) instead
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3168
tools:
- name: Trimmomatic
  role: Performs cleaning, adapter removal, and low-quality sequence trimming on raw
    FASTQ input in both SAMSA v1 and SAMSA2
- name: FLASH
  role: Joins cleaned paired-end reads into single extended fragments in SAMSA v1's
    preprocessing pipeline
- name: PEAR
  role: Merges paired-end FASTQ files in SAMSA2's preprocessing (with multithreading
    support)
- name: SAMSA
  role: Pipeline wrapper (SAMSA_pre_annotation_pipeline.py) that orchestrates Trimmomatic
    and FLASH preprocessing before MG-RAST submission
  repo: github.com/transcript/SAMSA
- name: SortMeRNA
  role: Post-trimming ribosomal read filter in SAMSA2 so rRNA does not pollute the
    mRNA profile
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-016-1270-8
    title: 'SAMSA: a comprehensive metatranscriptome analysis pipeline'
  source_content_hash: sha256:ca06c64ebaf5a0aa03db0d3099eeb88aa324d776db141651f103f04384884206
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FLASH
  - PEAR
  - SAMSA
  - SortMeRNA
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# metatranscriptomic-read-quality-trimming

## Summary

Preprocessing of raw metatranscriptomic FASTQ data by adapter/contaminant removal and low-quality read trimming with Trimmomatic, followed by paired-end merging with FLASH (or PEAR), as implemented in the SAMSA pipeline before annotation.

## When to use

Use when starting from raw paired-end metatranscriptomic FASTQ files (e.g. ribodepleted or poly(A)-enriched stool RNA-seq) that need adapter trimming and quality cleaning before read joining and downstream annotation.

## When NOT to use

- Input reads are already cleaned, adapter-trimmed, and merged, or are not FASTQ raw sequence data
- Sequences are short single-end reads (<100 bp) that cannot meet SAMSA's minimum read length requirement even after merging
- The goal is rRNA removal rather than adapter/quality trimming — use a dedicated ribosomal filter (SortMeRNA) instead

## Inputs

- Raw paired-end FASTQ metatranscriptome files (e.g. ribodepleted P1 and poly(A)-enriched P4 stool samples)
- Adapter sequence FASTA file for Trimmomatic

## Outputs

- Cleaned FASTQ files with adapters and low-quality sequences removed
- Joined single-file FASTQ of merged paired-end fragments (>100 bp) suitable for annotation or reference alignment

## How to apply

Run SAMSA's preprocessing (SAMSA_pre_annotation_pipeline.py), which uses Trimmomatic to perform cleaning and removal of adapter contamination and low-quality sequences from each raw FASTQ file. Because SAMSA requires sequences longer than 100 bp, join the cleaned paired-end reads using FLASH (SAMSA v1, with MG-RAST) or PEAR (SAMSA2). Note that in SAMSA2, SortMeRNA is applied after trimming to remove ribosomal reads, since rRNA does not contribute to the mRNA functional profile. Only after this preprocessing are the joined reads ready for annotation (MG-RAST submission or DIAMOND) or, as in task_004, for alignment against a reference genome. Trimmomatic removes adapter contamination according to a specified fasta adapter file; multithreading is available in SAMSA2.

## Related tools

- **Trimmomatic** (Performs cleaning, adapter removal, and low-quality sequence trimming on raw FASTQ input in both SAMSA v1 and SAMSA2)
- **FLASH** (Joins cleaned paired-end reads into single extended fragments in SAMSA v1's preprocessing pipeline)
- **PEAR** (Merges paired-end FASTQ files in SAMSA2's preprocessing (with multithreading support))
- **SAMSA** (Pipeline wrapper (SAMSA_pre_annotation_pipeline.py) that orchestrates Trimmomatic and FLASH preprocessing before MG-RAST submission) — github.com/transcript/SAMSA
- **SortMeRNA** (Post-trimming ribosomal read filter in SAMSA2 so rRNA does not pollute the mRNA profile)

## Examples

```
python SAMSA_pre_annotation_pipeline.py --input raw_paired_end.fastq
```

## Evaluation signals

- Output must be a single joined FASTQ file with merged fragments longer than 100 bp, per SAMSA's sequence length requirement
- No residual adapter contamination should remain detectable in the cleaned FASTQ
- Downstream per-sample read counts must remain sufficient — the pipeline expects 40-50 million raw sequences yielding 5-10 million annotated reads per sample
- Trimmed-and-joined reads should be usable for reference alignment; in the task_004 validation, only 417/340 ribodepleted and 2273/2100 poly(A) reads (0.0019-0.116%) matched the human genome, so massive host-alignment rates would indicate a preprocessing error
- SAMSA2 writes a checkpoint file after each preprocessing step; reruns should skip completed trimming rather than repeat it

## Limitations

- Sequence reads must be either longer than 100 bp or joined paired-end reads; short unmerged reads fail SAMSA's input requirement
- The pipeline expects ~40-50 million raw sequences per sample; under-sequenced samples may not yield the 5-10 million annotated reads needed for accurate abundance measures
- SAMSA v1 (Python 2.7) requires an internet connection and an MG-RAST account after preprocessing; SAMSA2 is mostly Python3-oriented and Python2 aggregation scripts may still error

## Evidence

- [methods] it performs cleaning and removal of adaptor contamination, and then aligns paired-end files (if specified): "it performs cleaning and removal of adaptor contamination, and then aligns paired-end files (if specified)"
- [methods] SAMSA has several dependencies, including the programs FLASH and Trimmomatic, Python version 2.7, and R: "SAMSA has several dependencies, including the programs FLASH and Trimmomatic, Python version 2.7, and R"
- [abstract] Sequences need to be either long reads (longer than 100 bp) or joined paired-end reads.: "Sequences need to be either long reads (longer than 100 bp) or joined paired-end reads."
- [readme] Trimmomatic - removes low-quality sequences and/or adaptor contamination.: [evidence span withheld — anchor status: external_doc]
- [readme] PEAR - merges two paired-end FASTQ sequencing files together to make a single file of extended fragments.: [evidence span withheld — anchor status: external_doc]

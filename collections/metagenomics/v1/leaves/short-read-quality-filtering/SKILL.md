---
name: short-read-quality-filtering
description: Use when raw paired-end short-read (Illumina) FASTQ files must be cleaned
  of low-quality reads, adapters and poor-quality trailing bases before hybrid assembly
  or downstream polishing.
when_to_use_negative:
- The input is long (ONT/PacBio) reads — Plassembler uses chopper with -l 500 and
  -q 9 for long-read filtering, not fastp
- The paired short reads have already been adapter-trimmed and quality-filtered and
  Plassembler is run with --skip_qc
- The data are single-end or non-Illumina short reads where paired-end adapter/quality
  trimming does not apply
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: fastp
  role: Filters and trims the paired-end short-read FASTQs in Plassembler's QC step
    (quality filtering, adapter trimming, per-read quality cutting), producing gzip-compressed
    cleaned read pairs
  repo: https://github.com/OpenGene/fastp
- name: Plassembler
  role: Pipeline whose short-read QC stage invokes fastp on the input FASTQ pair;
    skippable via --skip_qc
  repo: https://github.com/gbouras13/plassembler
- name: chopper
  role: 'Contrast tool: Plassembler''s QC filters the long-read FASTQ with chopper
    (-l 500, -q 9), not with fastp'
  repo: https://github.com/wdecoster/chopper
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad409
    title: 'Plassembler: an automated bacterial plasmid assembly tool'
  source_content_hash: sha256:e3ca267215addb622b4b2021f05d1f7e49f7b092f6afc71b4630200465838f1c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OpenGene/fastp
  tools:
  - Plassembler
  - chopper
  - fastp
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# short-read-quality-filtering

## Summary

Quality-controls Illumina paired-end FASTQs before hybrid plasmid assembly by filtering and trimming them with fastp (>=0.24.2), exactly as Plassembler does inside its QC step (skippable via --skip_qc).

## When to use

Use when raw paired-end short-read (Illumina) FASTQ files must be cleaned of low-quality reads, adapters and poor-quality trailing bases before hybrid assembly or downstream polishing.

## When NOT to use

- The input is long (ONT/PacBio) reads — Plassembler uses chopper with -l 500 and -q 9 for long-read filtering, not fastp
- The paired short reads have already been adapter-trimmed and quality-filtered and Plassembler is run with --skip_qc
- The data are single-end or non-Illumina short reads where paired-end adapter/quality trimming does not apply

## Inputs

- Paired-end Illumina FASTQ files, gzip-compressed (R1 and R2, e.g. Acinetobacter_baumannii_1.fastq.gz and Acinetobacter_baumannii_2.fastq.gz)
- fastp version >= 0.24.2

## Outputs

- cleaned_1.fastq.gz (filtered/trimmed R1, gzip FASTQ)
- cleaned_2.fastq.gz (filtered/trimmed R2, gzip FASTQ)
- fastp QC report (JSON/HTML describing filtering and trimming results)

## How to apply

Run fastp (>=0.24.2) in paired-end mode on the R1/R2 gzip-compressed FASTQ pair, relying on its default quality filtering, automatic adapter detection and per-read sliding-window quality trimming to yield cleaned reads. Write gzip-compressed paired outputs (e.g. cleaned_1.fastq.gz, cleaned_2.fastq.gz). This mirrors Plassembler's short-read QC stage, which filters and trims input paired-end FASTQs with fastp before mapping and Unicycler assembly; the stage can be skipped with --skip_qc if reads are already clean. Long reads are deliberately NOT handled here — Plassembler filters those with chopper (-l 500, -q 9) instead.

## Related tools

- **fastp** (Filters and trims the paired-end short-read FASTQs in Plassembler's QC step (quality filtering, adapter trimming, per-read quality cutting), producing gzip-compressed cleaned read pairs) — https://github.com/OpenGene/fastp
- **Plassembler** (Pipeline whose short-read QC stage invokes fastp on the input FASTQ pair; skippable via --skip_qc) — https://github.com/gbouras13/plassembler
- **chopper** (Contrast tool: Plassembler's QC filters the long-read FASTQ with chopper (-l 500, -q 9), not with fastp) — https://github.com/wdecoster/chopper

## Examples

```
fastp -i Acinetobacter_baumannii_1.fastq.gz -I Acinetobacter_baumannii_2.fastq.gz -o cleaned_1.fastq.gz -O cleaned_2.fastq.gz
```

## Evaluation signals

- Both cleaned paired outputs (cleaned_1.fastq.gz and cleaned_2.fastq.gz) exist and are valid gzip FASTQ files
- R1 and R2 contain matching numbers of reads (pairing preserved after filtering)
- fastp's JSON/HTML report records adapter trimming and quality filtering statistics consistent with the input read counts
- No retained read exceeds the original read count (filtering only removes reads/bases, never adds)
- Long reads were filtered separately (e.g. chopper -l 500 -q 9) and never passed through fastp

## Limitations

- fastp is intended for short reads; for Nanopore/PacBio long reads a different filter (chopper in Plassembler) is required
- The QC step adds runtime and can be bypassed with Plassembler's --skip_qc, so cleaned outputs may be absent if QC was skipped

## Evidence

- [methods] Plassembler's short-read QC stage filters and trims the input paired-end FASTQ files using fastp, as part of its quality control step: "* [fastp](https://github.com/OpenGene/fastp) >=0.24.2"
- [readme] fastp's purpose is all-in-one FASTQ preprocessing and QC: [evidence span withheld — anchor status: external_doc]
- [readme] fastp's filtering behavior (bad-read removal): [evidence span withheld — anchor status: external_doc]
- [readme] fastp's automatic adapter detection removes the need to supply adapter sequences: [evidence span withheld — anchor status: external_doc]

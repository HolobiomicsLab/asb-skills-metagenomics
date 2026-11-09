---
name: long-read-preprocessing-for-plasmid-assembly
description: Use when a pooled or subsampled ONT long-read FASTQ must be length- and
  quality-filtered before Plassembler plasmid assembly, or when reproducing Plassembler's
  chopper defaults of 500 bp / Q9.
when_to_use_negative:
- Input is Illumina short reads — Plassembler handles short reads with fastp and Unicycler
  polishing, not chopper long-read filtering
- The goal is coverage subsampling (e.g. 50x/60x) rather than quality/length filtering
  — that is rasusa's job in the Plassembler workflow (toggleable via --no_subsample)
- Reads need error correction or depth-based subsampling before assembly — canu -correct
  (error correction and subsampling to 100x estimated depth) is the appropriate tool
  instead
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: chopper
  role: Filters (and optionally trims) the long-read FASTQ by minimum length 500 bp
    and average quality Q9; the Rust NanoFilt+NanoLyse reimplementation that replaced
    NanoFilt in Plassembler's QC step
  repo: https://github.com/wdecoster/chopper
- name: Plassembler
  role: Plasmid assembly pipeline whose long-read quality control step embeds this
    chopper filtering before the Flye/Raven long-read-only assembly
  repo: https://github.com/gbouras13/plassembler
- name: NanoFilt
  role: Python filter that chopper replaces; the legacy long-read filtering tool in
    earlier Plassembler versions
  repo: https://github.com/wdecoster/nanofilt
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad409
    title: 'Plassembler: an automated bacterial plasmid assembly tool'
  source_content_hash: sha256:ce157a2dd2ab7cb7f9ba04cec6b7cbd6811a538f9d18d62f81da6ea19e65c886
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wdecoster/chopper
  tools:
  - NanoFilt
  - Plassembler
  - chopper
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# long-read-preprocessing-for-plasmid-assembly

## Summary

Quality- and length-filter gzipped ONT long-read FASTQs with chopper (>=0.11.0) before Plassembler plasmid assembly, applying the pipeline's default cutoffs of 500 bp minimum read length and average Q-score 9.

## When to use

Use when a pooled or subsampled ONT long-read FASTQ must be length- and quality-filtered before Plassembler plasmid assembly, or when reproducing Plassembler's chopper defaults of 500 bp / Q9.

## When NOT to use

- Input is Illumina short reads — Plassembler handles short reads with fastp and Unicycler polishing, not chopper long-read filtering
- The goal is coverage subsampling (e.g. 50x/60x) rather than quality/length filtering — that is rasusa's job in the Plassembler workflow (toggleable via --no_subsample)
- Reads need error correction or depth-based subsampling before assembly — canu -correct (error correction and subsampling to 100x estimated depth) is the appropriate tool instead

## Inputs

- Gzipped ONT long-read FASTQ, e.g. the pooled, subsampled Klebsiella_variicola_subsampled60x.fastq.gz from the plassembler benchmarking deposit at zenodo.org/record/7996690

## Outputs

- Gzip-compressed filtered FASTQ (filtered_longreads.fastq.gz) containing only reads >=500 bp with average Q-score >=9

## How to apply

Stream the gzipped pooled/subsampled ONT long-read FASTQ (e.g. Klebsiella_variicola_subsampled60x.fastq.gz from the plassembler benchmarking deposit at zenodo.org/record/7996690) into chopper >=0.11.0 via stdin with -l 500 (keep reads >=500 bp) and -q 9 (keep reads with average Phred quality >=9). Compress chopper's stdout with gzip to produce filtered_longreads.fastq.gz. These are the defaults Plassembler itself uses for long-read QC — chopper replaced NanoFilt in Plassembler — so running them standalone reproduces the filtering that precedes the Flye (or Raven) long-read-only assembly used to identify non-chromosome contigs. Filtering is per-read on average read quality, minimum/maximum length and optionally GC content; it does not error-correct. Verify the output is a valid gzip FASTQ in which every retained record meets both the 500 bp and Q9 thresholds.

## Related tools

- **chopper** (Filters (and optionally trims) the long-read FASTQ by minimum length 500 bp and average quality Q9; the Rust NanoFilt+NanoLyse reimplementation that replaced NanoFilt in Plassembler's QC step) — https://github.com/wdecoster/chopper
- **Plassembler** (Plasmid assembly pipeline whose long-read quality control step embeds this chopper filtering before the Flye/Raven long-read-only assembly) — https://github.com/gbouras13/plassembler
- **NanoFilt** (Python filter that chopper replaces; the legacy long-read filtering tool in earlier Plassembler versions) — https://github.com/wdecoster/nanofilt

## Examples

```
zcat Klebsiella_variicola_subsampled60x.fastq.gz | chopper -q 9 -l 500 | gzip > filtered_longreads.fastq.gz
```

## Evaluation signals

- gzip integrity: `gzip -t filtered_longreads.fastq.gz` exits 0 and the file parses as FASTQ
- threshold invariant: every retained record is >=500 bp with mean Phred >=9 (spot-check with seqkit fx2tab or NanoStat)
- monotonicity: output read count and total bases never exceed the input read set (filtering only removes reads)
- version check: chopper is >=0.11.0, matching the benchmarked workflow
- pipeline equivalence: running Plassembler on the same FASTQ passes reads through the same 500 bp / Q9 defaults in its internal QC step

## Limitations

- chopper filters on average read quality, not per-base quality, so reads containing long low-quality stretches can still pass Q9; it does not error-correct
- Plassembler swapped NanoFilt for chopper ('Replaces nanofilt with chopper'), so filtering behaviour and speed differ across Plassembler versions
- chopper can also trim reads (e.g. fixed head-crop and other trimming modes) when trimming flags are set, which changes more than the retained read set

## Evidence

- [discussion] Replaces nanofilt with chopper: "* Replaces nanofilt with chopper"
- [other] Run chopper (>=0.11.0) on the FASTQ with -l 500 (minimum read length 500 bp) and -q 9 (minimum average Q-score 9): [evidence span withheld — anchor status: not_found]
- [readme] Filtering is based on average read quality, minimum or maximum read length, and GC content percentage.: [evidence span withheld — anchor status: external_doc]
- [readme] This tool, intended for long read sequencing such as PacBio or ONT, filters and trims a fastq file.: [evidence span withheld — anchor status: external_doc]

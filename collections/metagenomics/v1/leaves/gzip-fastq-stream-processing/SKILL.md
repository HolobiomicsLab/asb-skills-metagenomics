---
name: gzip-fastq-stream-processing
description: Use when raw or pooled long-read (ONT/PacBio) FASTQ.gz files must be
  filtered for minimum read length and average Phred quality before assembly or benchmarking,
  e.g. pre-processing ONT reads for Plassembler.
when_to_use_negative:
- Input is short-read (Illumina/MGI) FASTQ that needs adapter/polyG trimming — that
  is the domain of fastp, not chopper
- The FASTQ has already been quality- and length-filtered and only needs downstream
  processing (e.g. subsampling with rasusa or minimap2 mapping)
- The objective is trimming/adaptor removal rather than whole-record quality/length
  filtering
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: chopper
  role: Rust NanoFilt/NanoLyse replacement that filters long-read FASTQ records by
    average read quality and minimum read length, reading gzipped FASTQ from stdin
    and writing gzip-compressed output
  repo: https://github.com/wdecoster/chopper
- name: Plassembler
  role: hybrid plasmid-assembly pipeline whose quality-control step applies these
    chopper filters to long reads before the Flye/Raven long-read-only assembly
  repo: https://github.com/gbouras13/plassembler
- name: rasusa
  role: upstream subsampling step (e.g. to 50x/60x coverage) that produces the subsampled
    FASTQ inputs this filtering skill operates on
  repo: https://github.com/mbhall88/rasusa
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad409
    title: 'Plassembler: an automated bacterial plasmid assembly tool'
  source_content_hash: sha256:d7d7913fd994a418a5b1deb61a6ba465964e52bc126542cba3f2d4003bd12c52
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wdecoster/chopper
  tools:
  - Plassembler
  - chopper
  - rasusa
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# gzip-fastq-stream-processing

## Summary

Quality- and length-filter pooled, gzipped ONT long-read FASTQs by streaming from stdin to stdout with chopper, retaining only reads meeting a minimum length (500 bp) and minimum average Q-score (9), as done in the Plassembler QC/benchmarking workflow.

## When to use

Use when raw or pooled long-read (ONT/PacBio) FASTQ.gz files must be filtered for minimum read length and average Phred quality before assembly or benchmarking, e.g. pre-processing ONT reads for Plassembler.

## When NOT to use

- Input is short-read (Illumina/MGI) FASTQ that needs adapter/polyG trimming — that is the domain of fastp, not chopper
- The FASTQ has already been quality- and length-filtered and only needs downstream processing (e.g. subsampling with rasusa or minimap2 mapping)
- The objective is trimming/adaptor removal rather than whole-record quality/length filtering

## Inputs

- pooled, subsampled ONT long-read FASTQ in gzip-compressed form (e.g. Klebsiella_variicola_subsampled60x.fastq.gz from zenodo.org/record/7996690)
- minimum length threshold (-m/-l, default 500 bp) and minimum average quality threshold (-q, default Q9)

## Outputs

- gzip-compressed filtered FASTQ (filtered_longreads.fastq.gz) containing only records >=500 bp with average Q-score >=9
- record count / length summary for verifying the filter effect

## How to apply

Stream the gzip-compressed ONT FASTQ through chopper (>=0.11.0), reading the gzipped input from stdin and writing gzip-compressed filtered reads to stdout, applying the minimum length threshold (500 bp, default '-m'/-l parameter) and minimum average Q-score (9 via '-q'). Retrieve the input — e.g. a pooled, rasusa-subsampled ONT FASTQ like Klebsiella_variicola_subsampled60x.fastq.gz from zenodo.org/record/7996690 — decompress on the fly, and save the gzip-compressed output as filtered_longreads.fastq.gz. Chopper is the Rust replacement for NanoFilt chosen by Plassembler for this step, so no intermediate uncompressed files are needed. The rationale is that low-quality or very short reads degrade the downstream Flye/Raven long-read-only assembly used to identify putative plasmid contigs, so only records meeting both thresholds should reach assembly.

## Related tools

- **chopper** (Rust NanoFilt/NanoLyse replacement that filters long-read FASTQ records by average read quality and minimum read length, reading gzipped FASTQ from stdin and writing gzip-compressed output) — https://github.com/wdecoster/chopper
- **Plassembler** (hybrid plasmid-assembly pipeline whose quality-control step applies these chopper filters to long reads before the Flye/Raven long-read-only assembly) — https://github.com/gbouras13/plassembler
- **rasusa** (upstream subsampling step (e.g. to 50x/60x coverage) that produces the subsampled FASTQ inputs this filtering skill operates on) — https://github.com/mbhall88/rasusa

## Examples

```
gunzip -c Klebsiella_variicola_subsampled60x.fastq.gz | chopper -l 500 -q 9 | gzip > filtered_longreads.fastq.gz
```

## Evaluation signals

- Output is a valid gzip FASTQ that decompresses without error (e.g. gunzip -t filtered_longreads.fastq.gz)
- Every retained record satisfies both thresholds: length >=500 bp and average Q-score >=9
- Retained-read count is <= input-read count, with reads removed corresponding to short or low-quality records
- Filter runs as a stream (no uncompressed intermediates) and preserves FASTQ 4-line record structure
- Downstream long-read-only assembly (Flye/Raven in Plassembler) accepts the filtered FASTQ without format errors

## Limitations

- Chopper filtering alone does not remove chromosomal reads or plasmid-misclassified reads — Plassembler's mapping-based chromosomal-read removal is a separate step
- Thresholds 500 bp / Q9 are Plassembler defaults; hard length cutoffs may discard legitimately short plasmid-relevant reads from very degraded libraries
- The replacement of NanoFilt with chopper reflects a performance choice (Rust vs Python), so outputs may differ slightly from older NanoFilt-based pipelines

## Evidence

- [discussion] chopper as Rust NanoFilt replacement filtering long reads: "* Replaces nanofilt with chopper"
- [readme] chopper README: filtering basis on quality and length: [evidence span withheld — anchor status: external_doc]
- [readme] chopper README: intended input type: [evidence span withheld — anchor status: external_doc]
- [other] workflow: chopper 500 bp / Q9 with stdin and gzip output: [evidence span withheld — anchor status: not_found]

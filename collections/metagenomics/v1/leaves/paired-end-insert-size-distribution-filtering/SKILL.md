---
name: paired-end-insert-size-distribution-filtering
description: Use when profiling metagenomic read mappings against reference genomes
  and needing to discard improperly paired or aberrant-insert-size reads before computing
  coverage, ANI, or SNVs.
when_to_use_negative:
- Input alignments are single-end reads with no mate information, since all un-paired
  reads are removed by default and filters operate on read pairs
- The goal is only raw read counting or abundance estimation (e.g. with CoverM), not
  accurate microdiversity/SNV profiling where erroneous mappings must be reduced
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: inStrain
  role: implements the paired-read filtering pipeline and writes read_report.tsv
  repo: github.com/MrOlm/instrain
- name: Samtools
  role: used by inStrain for interacting with .bam and .sam files during read loading
    and filtering
  repo: https://instrain.readthedocs.io
- name: Pandas
  role: aggregates pass/fail counts per filter stage and writes read_report.tsv
- name: Numpy
  role: numeric computation supporting per-pair filtering statistics
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:752c6637be721c7391a6a55aa7c864ecb9856677fc828e80d7766a061e07e11b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Numpy
  - Pandas
  - Samtools
  - inStrain
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# paired-end-insert-size-distribution-filtering

## Summary

Filters metagenomic read pairs in a sorted .bam file during 'inStrain profile' by removing un-paired reads and keeping only pairs mapped in proper orientation with insert size within an expected distribution (>50 bp and <3x the median), reducing erroneous mappings.

## When to use

Use when profiling metagenomic read mappings against reference genomes and needing to discard improperly paired or aberrant-insert-size reads before computing coverage, ANI, or SNVs.

## When NOT to use

- Input alignments are single-end reads with no mate information, since all un-paired reads are removed by default and filters operate on read pairs
- The goal is only raw read counting or abundance estimation (e.g. with CoverM), not accurate microdiversity/SNV profiling where erroneous mappings must be reduced

## Inputs

- sorted .bam file of reads mapped to a reference genome
- reference genome .fasta file (used for downstream nucleotide-identity filtering of the surviving pairs)
- optional user parameters: min_insert (default 50 bp), max_insert_relative (default 3), minimum mapQ (default -1), minimum read-pair ANI (default 0.95)

## Outputs

- filtered set of read pairs passing orientation/insert-size criteria
- read_report.tsv with per-sequence per-filter pass/fail counts
- inStrain profile object with coverage and nucleotide diversity computed on filtered pairs

## How to apply

Load the sorted .bam file with Samtools/pysam, joining read pairs by name and recording each pair's mapping coordinates, orientation, insert size, and mapQ. First remove all un-paired reads by default, flagging them in the report, because inStrain's filters are applied to pairs of reads in combination rather than to individual reads. Then apply the orientation/insert-size filter: keep only pairs mapped in proper orientation with insert size greater than min_insert (default 50 bp) and less than max_insert_relative (default 3) times the median insert size. Subsequent filters (mapQ using the higher of the two reads, default minimum -1; and read-pair nucleotide identity, default 0.95 applied with strict >) operate only on pairs that survive this step. Aggregate pass/fail counts per filter stage with pandas and write read_report.tsv with per-sequence filtering counts so the filtering outcome is auditable.

## Related tools

- **inStrain** (implements the paired-read filtering pipeline and writes read_report.tsv) — github.com/MrOlm/instrain
- **Samtools** (used by inStrain for interacting with .bam and .sam files during read loading and filtering) — https://instrain.readthedocs.io
- **Pandas** (aggregates pass/fail counts per filter stage and writes read_report.tsv)
- **Numpy** (numeric computation supporting per-pair filtering statistics)

## Examples

```
inStrain profile mapping.bam genome_file.fasta -o inStrain_profile1
```

## Evaluation signals

- read_report.tsv exists and contains per-sequence filtering counts
- Each read appears in exactly one pass/fail category per filter, and per-filter counts sum to the total reads in the .bam
- No un-paired reads remain among pairs used for downstream coverage/SNV computation
- All retained pairs have insert size > 50 bp and < 3x the median insert size, and are mapped in proper orientation

## Limitations

- Un-paired reads are discarded by default, so single-end datasets cannot be profiled with these defaults
- Variant sites near the ends of genome fragments can be missed with inStrain
- Careful filtering trades sensitivity for specificity: reads erroneously mapped are reduced, but some true pairs failing the insert-size/orientation expectations are also removed

## Evidence

- [readme] README quick-start profile command: [evidence span withheld — anchor status: external_doc]

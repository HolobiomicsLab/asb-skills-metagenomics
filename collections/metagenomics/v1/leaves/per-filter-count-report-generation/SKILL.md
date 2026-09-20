---
name: per-filter-count-report-generation
description: Use when a read-filtering pipeline must account for exactly how each
  .bam read (pair) was removed — e.g. un-paired, insert size, mapQ, or nucleotide
  identity — and per-filter counts must be reported per sequence and validated.
when_to_use_negative:
- The analysis works with single-end reads only — inStrain removes all un-paired reads
  by default, so nearly everything would be discarded
- You need coverage/breadth or SNV calls rather than an account of which filter removed
  each read pair (use inStrain profile's downstream outputs instead)
- Input is raw reads rather than an already-mapped, sorted .bam file — mapping must
  be done first (e.g. with Bowtie2)
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: inStrain
  role: Implements the read-filtering module and writes read_report.tsv during 'inStrain
    profile'
  repo: github.com/MrOlm/instrain
- name: Samtools
  role: Interacting with .bam and .sam files to load and filter mapped read pairs
- name: Pandas
  role: Aggregating pass/fail counts per filter stage and writing read_report.tsv
- name: Numpy
  role: Numerical backing for count aggregation and identity computations
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:fb091d3696f96d3685f864de3de5edd4a5f75d11dc66b234c60ea35167370d8e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Numpy
  - Pandas
  - Samtools
  - inStrain
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# per-filter-count-report-generation

## Summary

Generates a per-sequence, per-stage read-filtering report (read_report.tsv) that aggregates pass/fail counts for each filter applied to .bam read pairs, following inStrain profile's four-stage read-filtering scheme.

## When to use

Use when a read-filtering pipeline must account for exactly how each .bam read (pair) was removed — e.g. un-paired, insert size, mapQ, or nucleotide identity — and per-filter counts must be reported per sequence and validated.

## When NOT to use

- The analysis works with single-end reads only — inStrain removes all un-paired reads by default, so nearly everything would be discarded
- You need coverage/breadth or SNV calls rather than an account of which filter removed each read pair (use inStrain profile's downstream outputs instead)
- Input is raw reads rather than an already-mapped, sorted .bam file — mapping must be done first (e.g. with Bowtie2)

## Inputs

- Sorted .bam (or .sam) file of paired-end reads mapped to a reference genome
- Reference genome .fasta (for computing per-pair nucleotide identity)
- User-defined filter parameters: minimum mapQ (default -1), min_insert (default 50 bp), max_insert_relative (default 3), minimum read-pair ANI (default 0.95)

## Outputs

- read_report.tsv with per-sequence filtering counts
- Pass/fail category counts for each filter stage (un-paired removal, orientation/insert-size, mapQ, nucleotide identity)

## How to apply

Load the sorted .bam with Samtools/pysam, joining read pairs by name while recording each pair's mapping coordinates, orientation, insert size, and mapQ score. Remove all un-paired reads by default, flagging them in the report. Apply the remaining filters to pairs of reads in combination: keep pairs in proper orientation with insert size > 50 bp (min_insert) and < 3x the median insert size (max_insert_relative = 3); require the higher mapQ of the two reads to exceed the user-defined minimum (default: -1); and keep pairs strictly above the minimum read-pair ANI of 0.95 computed as 1 - (mismatches / read length), using > not >=. Aggregate pass/fail counts per filter stage with pandas (numpy-backed) per reference sequence and write read_report.tsv. The rationale, per Olm et al. 2021, is that careful filtering of reads in the .bam file reduces the probability of reads being erroneously mapped.

## Related tools

- **inStrain** (Implements the read-filtering module and writes read_report.tsv during 'inStrain profile') — github.com/MrOlm/instrain
- **Samtools** (Interacting with .bam and .sam files to load and filter mapped read pairs)
- **Pandas** (Aggregating pass/fail counts per filter stage and writing read_report.tsv)
- **Numpy** (Numerical backing for count aggregation and identity computations)

## Examples

```
inStrain profile mapping.bam genome_file.fasta -o inStrain_profile1
```

## Evaluation signals

- read_report.tsv exists after the profile run
- Each read appears in exactly one pass/fail category per filter
- Per-filter counts sum to the total reads in the .bam
- Counts are internally consistent per filter stage (no read double-counted across pass/fail within a stage)
- Pairs retained after the ANI filter all exceed 0.95 nucleotide identity (strict >, not >=)

## Limitations

- Variant sites near the ends of genome fragments can be missed with inStrain, so filtering cannot fully rescue edge-of-fragment signal
- The Python implementation is slow; performance may be limiting on very large .bam files
- Insert-size filter relies on the median insert size of the mapping (max_insert_relative = 3), so highly variable libraries may lose many pairs

## Evidence

- [readme] inStrain profile CLI invocation from README: [evidence span withheld — anchor status: external_doc]

---
name: mapping-quality-based-read-pair-filtering
description: Use when processing a mapped metagenomic .bam file for microdiversity
  or strain-level analysis and you need to remove un-paired reads and apply pair-level
  filters (orientation, insert size, mapQ, nucleotide identity) before downstream
  profiling.
when_to_use_negative:
- Input reads are single-end/un-paired and you intend to retain them — inStrain removes
  all un-paired reads by default
- You only need raw coverage/breadth estimates without filtering — a faster tool like
  CoverM may be more appropriate than pair-level filtering
- You need per-read filtering statistics rather than pair-level decisions — inStrain
  applies filters to pairs of reads in combination, not individual reads
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: inStrain
  role: Implements the four-stage read-pair filtering of the .bam file during 'inStrain
    profile'
  repo: https://github.com/MrOlm/instrain
- name: Samtools
  role: Used for interacting with .bam and .sam files to load and parse read pairs
- name: Pandas
  role: Aggregates pass/fail counts per filter stage and writes read_report.tsv
- name: Numpy
  role: Numerical support for filtering computations
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:b98ad8e98c138b0391896cd9bdfdd0edb80364a9f38078f976b3af8011d8a7cd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/MrOlm/instrain
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
yes_rate: 0.7272727272727273
n_claims: 11
---

# mapping-quality-based-read-pair-filtering

## Summary

Reconstructs inStrain's four-stage read filtering of a sorted .bam file: un-paired reads are removed by default and the orientation/insert-size, mapQ, and nucleotide-identity filters are applied to read pairs in combination rather than to individual reads, reducing erroneous mappings before coverage, SNV, and linkage analysis.

## When to use

Use when processing a mapped metagenomic .bam file for microdiversity or strain-level analysis and you need to remove un-paired reads and apply pair-level filters (orientation, insert size, mapQ, nucleotide identity) before downstream profiling.

## When NOT to use

- Input reads are single-end/un-paired and you intend to retain them — inStrain removes all un-paired reads by default
- You only need raw coverage/breadth estimates without filtering — a faster tool like CoverM may be more appropriate than pair-level filtering
- You need per-read filtering statistics rather than pair-level decisions — inStrain applies filters to pairs of reads in combination, not individual reads

## Inputs

- sorted .bam file of metagenomic reads mapped to reference genomes (produced e.g. with Bowtie 2)
- reference genome .fasta file (for computing read-pair nucleotide identity)
- optional user parameters: minimum mapQ, minimum nucleotide identity, min_insert, max_insert_relative

## Outputs

- read_report.tsv with per-sequence filtering counts for each filter stage
- set of filtered read pairs retained for downstream coverage, SNV, and linkage calculations

## How to apply

Load the sorted .bam with Samtools/pysam, joining read pairs by name and recording each pair's mapping coordinates, orientation, insert size, and mapQ score. Remove all un-paired reads by default and flag them in the report. Apply the orientation/insert-size filter — keep pairs mapped in proper orientation with insert size > 50 bp (min_insert) and < 3x the median insert size (max_insert_relative = 3). Apply the mapQ filter using the higher mapQ of the two reads in each pair against the user-defined minimum (default -1), then compute each pair's nucleotide identity as 1 - (mismatches / read length) and keep pairs strictly above the minimum read-pair ANI cutoff (default 0.95, applied with > not >=). Rationale: filtering in combination at the pair level reduces the probability of reads being erroneously mapped, which would otherwise inflate coverage, diversity, and SNV calls.

## Related tools

- **inStrain** (Implements the four-stage read-pair filtering of the .bam file during 'inStrain profile') — https://github.com/MrOlm/instrain
- **Samtools** (Used for interacting with .bam and .sam files to load and parse read pairs)
- **Pandas** (Aggregates pass/fail counts per filter stage and writes read_report.tsv)
- **Numpy** (Numerical support for filtering computations)

## Examples

```
inStrain profile mapping.bam genome_file.fasta -o inStrain_profile1
```

## Evaluation signals

- read_report.tsv exists and its per-filter counts are internally consistent: each read appears in exactly one pass/fail category per filter
- Per-filter counts sum to the total reads in the .bam
- All retained pairs satisfy orientation and insert-size bounds (> 50 bp and < 3x median insert size)
- Retained pairs have mapQ at or above the user-defined minimum (default -1) and nucleotide identity strictly above 0.95
- Un-paired reads are absent from the filtered set and are flagged in the report

## Limitations

- All un-paired reads are removed by default, so single-end data cannot be profiled with this pipeline
- Filters operate on read pairs in combination (e.g. the higher mapQ of the pair), so per-read quality distinctions are not preserved
- The nucleotide-identity cutoff is strict inequality (default 0.95 with > not >=), so pairs exactly at the cutoff are removed
- Variant sites near the ends of genome fragments can be missed with inStrain

## Evidence

- [readme] inStrain profile mapping.bam genome_file.fasta: [evidence span withheld — anchor status: external_doc]

---
name: bam-file-parsing-and-read-pair-joining
description: Use when a metagenomic read-mapping .bam file must be filtered before
  profile coverage/SNV analysis and paired-end read structure (orientation, insert
  size, mapQ) should drive filtering.
when_to_use_negative:
- The input is raw FASTQ reads that have not yet been mapped to a reference — mapping
  (e.g. with Bowtie 2) must happen first
- The goal is single-read coverage estimation (e.g. CoverM-style breadth/abundance)
  rather than pair-aware filtering for microdiversity and SNV analysis
- Reads are single-end, since all un-paired reads are removed by default and no read-pair
  filters can be applied
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: inStrain
  role: Python package implementing the four-stage read-filtering of 'inStrain profile'
    on the .bam file
  repo: github.com/MrOlm/instrain
- name: Samtools
  role: Required for interacting with .bam and .sam files and calculating coverage
    during read filtering/loading
- name: Pandas
  role: Aggregates pass/fail counts per filter stage and writes read_report.tsv
- name: Numpy
  role: Numerical computation underlying pair-level metrics (insert size, identity)
    during filtering
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:40a564ce582b92b72caf47ceb37afd9063ddbf4611ac7addb1a4ef627897d310
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
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# bam-file-parsing-and-read-pair-joining

## Summary

Parses a sorted .bam file with Samtools/pysam, joins read pairs by name, and applies inStrain's read filters at the read-pair level rather than to individual reads, so that erroneous mappings are removed before coverage, SNV, and diversity calculations.

## When to use

Use when a metagenomic read-mapping .bam file must be filtered before profile coverage/SNV analysis and paired-end read structure (orientation, insert size, mapQ) should drive filtering.

## When NOT to use

- The input is raw FASTQ reads that have not yet been mapped to a reference — mapping (e.g. with Bowtie 2) must happen first
- The goal is single-read coverage estimation (e.g. CoverM-style breadth/abundance) rather than pair-aware filtering for microdiversity and SNV analysis
- Reads are single-end, since all un-paired reads are removed by default and no read-pair filters can be applied

## Inputs

- Sorted .bam file of reads mapped to a reference genome
- Reference genome .fasta file (for nucleotide identity calculation)

## Outputs

- Set of filtered read pairs retained for downstream coverage/SNV profiling
- read_report.tsv with per-sequence and per-filter pass/fail counts

## How to apply

Load the sorted .bam file with Samtools/pysam, joining read pairs by name and recording each pair's mapping coordinates, orientation, insert size, and mapQ score. First remove all un-paired reads by default, flagging them in the report, because inStrain applies all filters to pairs of reads in combination. Then keep pairs mapped in the proper orientation with insert size > 50 bp (min_insert) and < 3x the median insert size (max_insert_relative = 3). Apply the mapQ filter using the higher mapQ of the two reads in each pair (default minimum -1), then compute each pair's nucleotide identity versus the reference .fasta as 1 - (mismatches / read length) and keep pairs strictly above the minimum read-pair ANI cutoff of 0.95 (applied with >, not >=). Aggregate pass/fail counts per filter stage with pandas and write read_report.tsv with per-sequence filtering counts.

## Related tools

- **inStrain** (Python package implementing the four-stage read-filtering of 'inStrain profile' on the .bam file) — github.com/MrOlm/instrain
- **Samtools** (Required for interacting with .bam and .sam files and calculating coverage during read filtering/loading)
- **Pandas** (Aggregates pass/fail counts per filter stage and writes read_report.tsv)
- **Numpy** (Numerical computation underlying pair-level metrics (insert size, identity) during filtering)

## Examples

```
inStrain profile mapping.bam genome_file.fasta -o inStrain_profile1
```

## Evaluation signals

- read_report.tsv exists and contains per-filter counts for the .bam file
- Per-filter counts are internally consistent: each read appears in exactly one pass/fail category per filter and counts sum to the total reads in the .bam
- No un-paired reads are present in the retained read set (all are removed by default and flagged in the report)
- Retained pairs have proper orientation, insert size > 50 bp and < 3x median insert size, and pair ANI strictly above 0.95

## Limitations

- All un-paired reads are removed by default, so single-end mappings cannot be profiled this way
- Variant sites near the ends of genome fragments can be missed with inStrain
- Filters are deliberately strict to reduce the probability of reads being erroneously mapped, which may discard usable but imperfectly paired reads

## Evidence

- [discussion] All un-paired reads are removed by default, and filters are applied to pairs of reads in combination: "i) All un-paired reads are removed by default, and filters are applied to pairs of reads in combination"
- [discussion] Careful filtering of reads in the .bam file is performed: "Careful filtering of reads in the .bam file is performed to reduce the probability of reads being erroneously mapped"
- [discussion] InStrain requires Samtools for interacting with .bam and .sam files: "InStrain requires Samtools 52 for interacting with .bam and .sam files"
- [discussion] Paired reads must be mapped in the proper orientation within an expected insert size: "ii) Paired reads must be mapped in the proper orientation within an expected insert size"
- [discussion] Pairs must be above a user-defined minimum nucleotide identity value: "iv) Pairs must be above a user-defined minimum nucleotide identity value"
- [readme] inStrain profile mapping.bam genome_file.fasta -o inStrain_profile1: [evidence span withheld — anchor status: external_doc]

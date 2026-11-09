---
name: read-pair-nucleotide-identity-calculation
description: Use when filtering mapped metagenomic reads before SNV calling or coverage
  profiling, and you must discard read pairs whose alignment to the reference .fasta
  falls below a user-defined minimum nucleotide identity.
when_to_use_negative:
- Input .bam contains single-end (un-paired) reads — all un-paired reads are removed
  by default, so this skill will discard everything.
- You need per-individual-read filtering rather than pair-level filtering — inStrain's
  filters are explicitly applied to pairs of reads in combination.
- You are at the SNV-calling stage (minimum 5 reads per position, >5% variant reads,
  null-model FDR) rather than the read-pair mapping-fidelity stage — those are separate
  criteria applied to filtered reads.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: inStrain
  role: Python package that implements the read-pair identity filter and writes the
    read report during 'inStrain profile'
  repo: https://github.com/MrOlm/instrain
- name: Samtools
  role: Used by inStrain to load and interact with the sorted .bam file when joining
    read pairs and computing mapping coordinates
- name: Pandas
  role: Aggregates pass/fail counts per filter stage and writes read_report.tsv
- name: Numpy
  role: Array computation supporting the identity and filter calculations
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:e2848b45da2f61f149d62aa65afe9ad2f5abb8c4534bc775685374253289a20a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/MrOlm/instrain
  tools:
  - Numpy
  - Pandas
  - Samtools
  - inStrain
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# read-pair-nucleotide-identity-calculation

## Summary

Compute per-pair nucleotide identity (ANI vs. the reference .fasta) for read pairs within inStrain's read-filtering pipeline, applying the identity cutoff at the read-pair level to reduce erroneous mappings in metagenomic .bam files.

## When to use

Use when filtering mapped metagenomic reads before SNV calling or coverage profiling, and you must discard read pairs whose alignment to the reference .fasta falls below a user-defined minimum nucleotide identity.

## When NOT to use

- Input .bam contains single-end (un-paired) reads — all un-paired reads are removed by default, so this skill will discard everything.
- You need per-individual-read filtering rather than pair-level filtering — inStrain's filters are explicitly applied to pairs of reads in combination.
- You are at the SNV-calling stage (minimum 5 reads per position, >5% variant reads, null-model FDR) rather than the read-pair mapping-fidelity stage — those are separate criteria applied to filtered reads.

## Inputs

- Sorted .bam file of paired metagenomic reads mapped to a reference genome
- Reference genome .fasta file
- Optional user parameters: minimum mapQ score (default -1), minimum insert size (default 50 bp), max_insert_relative (default 3), minimum read-pair ANI cutoff (default 0.95)

## Outputs

- Filtered set of read pairs passing all filters (orientation/insert size, mapQ, nucleotide identity)
- read_report.tsv with pass/fail counts per filter stage and per-sequence filtering counts

## How to apply

Load the sorted .bam with Samtools/pysam, joining read pairs by name and recording each pair's mapping coordinates, orientation, insert size, and mapQ score; remove all un-paired reads by default since inStrain applies its filters to pairs of reads in combination. Apply upstream filters first — proper orientation with insert size > 50 bp (min_insert) and < 3x the median insert size (max_insert_relative = 3), then the mapQ filter using the higher mapQ of the two reads (default minimum: -1). For each surviving pair, compute nucleotide identity as 1 - (mismatches / read length) against the provided reference .fasta, and keep pairs strictly above the minimum read-pair ANI cutoff (default 0.95, applied with > not >=). Aggregate pass/fail counts per filter stage with pandas and write read_report.tsv with per-sequence filtering counts. This pair-level identity filter reduces the probability of reads being erroneously mapped before downstream microdiversity and SNV analyses.

## Related tools

- **inStrain** (Python package that implements the read-pair identity filter and writes the read report during 'inStrain profile') — https://github.com/MrOlm/instrain
- **Samtools** (Used by inStrain to load and interact with the sorted .bam file when joining read pairs and computing mapping coordinates)
- **Pandas** (Aggregates pass/fail counts per filter stage and writes read_report.tsv)
- **Numpy** (Array computation supporting the identity and filter calculations)

## Examples

```
inStrain profile mapping.bam genome_file.fasta -o inStrain_profile1
```

## Evaluation signals

- read_report.tsv exists and its per-filter counts are internally consistent: each read pair appears in exactly one pass/fail category per filter, and counts sum to the total reads in the .bam.
- Un-paired reads flagged as removed in the report match the count of singletons present in the input .bam.
- All retained pairs satisfy insert size > 50 bp and < 3x the median insert size, and pass the mapQ filter using the higher mapQ of the two reads.
- Every retained pair's computed identity 1 - (mismatches / read length) is strictly greater than the minimum ANI cutoff (default 0.95) — boundary cases at exactly 0.95 must be excluded.
- Per-sequence filtering counts in read_report.tsv reconcile with the per-filter totals across sequences.

## Limitations

- Variant sites near the ends of genome fragments can be missed with inStrain (Supplemental Figure S3).
- The identity cutoff is strict (> not >=), so pairs exactly at the threshold are removed — users should be aware when tuning the default 0.95.
- The mapQ filter default of -1 means mapQ filtering is effectively off unless a user-defined minimum is supplied.
- inStrain removes all un-paired reads by default, so single-end data cannot be profiled with this filter chain as-is.

## Evidence

- [readme] inStrain profile mapping.bam genome_file.fasta -o inStrain_profile1: [evidence span withheld — anchor status: external_doc]

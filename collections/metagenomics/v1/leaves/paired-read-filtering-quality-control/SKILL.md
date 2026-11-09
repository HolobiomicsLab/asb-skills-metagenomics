---
name: paired-read-filtering-quality-control
description: Use when preparing .bam files for inStrain profile/compare so that genome
  coverage, SNV calls, and strain-level comparisons (ConANI/popANI) rest on correctly
  mapped read pairs rather than spurious single-end or mis-oriented alignments.
when_to_use_negative:
- Input .bam contains only single-end reads — un-paired reads are removed by default,
  so no usable data would remain for profiling
- The goal is raw read-count quantification (e.g. with CoverM) where aggressive pair
  filtering would discard signal; inStrain's filters target genome-comparison accuracy,
  not general coverage estimation
- Sequencing technology or library layout differs such that proper-orientation and
  expected-insert-size assumptions do not hold for the read pairs
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: inStrain
  role: Applies the paired-read filters (paired-only, proper orientation/insert size,
    minimum mapQ, minimum read-pair ANI 0.95) when running inStrain profile
  repo: https://github.com/MrOlm/instrain
- name: Bowtie2
  role: Maps paired-end reads to the reference .fasta, with the -X flag increased
    up to 1000 for longer-insert datasets before filtering
- name: Samtools
  role: Used by inStrain for interacting with .bam and .sam files during read filtering
    and coverage calculation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:e880157dd9650c8b181e83895aa1e6d3a1e8f9b8f7f5037ea3a73c381d34f65f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/MrOlm/instrain
  tools:
  - Bowtie2
  - MIDAS
  - Samtools
  - dRep
  - inStrain
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# Paired-read filtering quality control

## Summary

A read-filtering QC step used by inStrain before profiling metagenomic .bam files: un-paired reads are removed and read pairs must pass mapping-orientation, insert-size, mapQ, and nucleotide-identity criteria so that downstream coverage, SNV, and popANI calculations are not skewed by erroneously mapped reads.

## When to use

Use when preparing .bam files for inStrain profile/compare so that genome coverage, SNV calls, and strain-level comparisons (ConANI/popANI) rest on correctly mapped read pairs rather than spurious single-end or mis-oriented alignments.

## When NOT to use

- Input .bam contains only single-end reads — un-paired reads are removed by default, so no usable data would remain for profiling
- The goal is raw read-count quantification (e.g. with CoverM) where aggressive pair filtering would discard signal; inStrain's filters target genome-comparison accuracy, not general coverage estimation
- Sequencing technology or library layout differs such that proper-orientation and expected-insert-size assumptions do not hold for the read pairs

## Inputs

- Paired-end .bam file produced by Bowtie2 mapping to a reference genome .fasta (e.g. concatenated ZymoBIOMICS.STD.refseq.v2 references)
- Reference genome .fasta file
- User-defined filter parameters: minimum mapQ score, expected insert size, minimum read-pair nucleotide identity (default 0.95)

## Outputs

- Filtered set of properly paired, correctly oriented read pairs within expected insert size used for downstream profiling
- inStrain profile output (IS_profile) with per-position coverage, nucleotide diversity, and SNVs computed on filtered reads

## How to apply

After mapping reads with Bowtie2 (default settings; increase the -X flag up to 1000 for longer-insert datasets) against a concatenated reference such as ZymoBIOMICS.STD.refseq.v2, apply inStrain's filters to the .bam: (i) remove all un-paired reads by default and apply filters to read pairs in combination; (ii) require pairs to be mapped in the proper orientation within an expected insert size; (iii) require pairs to meet a user-defined minimum mapQ score; and (iv) require pairs to exceed a user-defined minimum nucleotide identity value (default minimum read-pair ANI 0.95). This careful filtering reduces the probability that reads are erroneously mapped, which is critical because coverage, nucleotide diversity, SNV calling, and pairwise popANI comparisons are all computed from the filtered read set. Run `inStrain profile` on the filtered .bam against the reference .fasta and confirm that only properly paired, high-identity alignments contribute to coverage and breadth metrics.

## Related tools

- **inStrain** (Applies the paired-read filters (paired-only, proper orientation/insert size, minimum mapQ, minimum read-pair ANI 0.95) when running inStrain profile) — https://github.com/MrOlm/instrain
- **Bowtie2** (Maps paired-end reads to the reference .fasta, with the -X flag increased up to 1000 for longer-insert datasets before filtering)
- **Samtools** (Used by inStrain for interacting with .bam and .sam files during read filtering and coverage calculation)

## Examples

```
inStrain profile mapping.bam ZymoBIOMICS.STD.refseq.v2.fasta -o inStrain_profile1
```

## Evaluation signals

- inStrain profile runs to completion on the .bam and reports coverage and breadth per reference genome from filtered read pairs only
- No un-paired reads contribute to coverage: profiles depend exclusively on read pairs passing proper-orientation and expected-insert-size filters
- Read pairs below the default minimum nucleotide identity of 0.95 are excluded from SNV and diversity calculations
- Downstream inStrain compare produces pairwise popANI values and coverage overlap between samples consistent with filtered profiles
- Detection of reference genomes (>= 0.5 unmaskedBreadth) reflects only high-confidence paired alignments against the ZymoBIOMICS references

## Limitations

- Variant sites near the ends of genome fragments can be missed with inStrain
- The Python implementation is slow; the changelog notes it should call the node2vec C++ library in the future
- Filter parameters (mapQ, insert size, nucleotide identity) are user-defined and must be adjusted for datasets with different insert sizes or error rates; the null model for SNV calling can be adjusted for technologies with different sequencing error rates (FDR default 1e-6)

## Evidence

- [methods] paired reads only, proper orientation, expected insert size, minimum read-pair ANI 0.95: [evidence span withheld — anchor status: not_found]

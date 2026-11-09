---
name: metagenomic-read-mapping
description: Use when metagenomic paired-end FASTQ reads need to be mapped to a multi-genome
  reference .fasta to quantify genome detection, coverage, microdiversity, or strain
  sharing (e.g., ZymoBIOMICS mock-community benchmarking).
when_to_use_negative:
- Input is already a .bam/.sam alignment or a coverage table — proceed directly to
  `inStrain profile` instead of re-mapping
- Only genome-level relative abundance/coverage is needed without microdiversity or
  SNV data — a fast tool like CoverM may be more appropriate than full inStrain profiling
- Reads are single-end or the reference genomes are not dereplicated, since un-paired
  reads are removed by default and highly similar genomes cause erroneous multi-mapping
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Bowtie2
  role: Maps paired-end metagenomic reads to the concatenated reference genome set
    with default settings (increase -X up to 1000 for longer-insert datasets)
- name: inStrain
  role: Profiles filtered .bam files for coverage, breadth, SNVs, and pairwise popANI/ConANI
    population comparisons
  repo: github.com/MrOlm/instrain
- name: Samtools
  role: Required by inStrain for interacting with .bam and .sam files and calculating
    per-position coverage
- name: dRep
  role: Benchmarked competing strain-comparison tool; also used for genome dereplication
    at 98% ANI to reduce multi-mapped reads
  repo: github.com/MrOlm/drep
- name: CoverM
  role: Alternative fast tool for calculating coverage and breadth of genomes from
    mapped reads
  repo: github.com/wwood/CoverM
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:08d223802bf976b4ca2efc9eef2a35cb490406d7ae6fabe2f532ddf1783e34db
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bowtie2
  - CoverM
  - MIDAS
  - Samtools
  - dRep
  - inStrain
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# metagenomic-read-mapping

## Summary

Maps metagenomic paired-end reads to a concatenated reference genome set with Bowtie2 and profiles the resulting .bam files with inStrain's default read filters, producing coverage, breadth, SNV, and population-comparison data suitable for strain-level benchmarking against known community composition.

## When to use

Use when metagenomic paired-end FASTQ reads need to be mapped to a multi-genome reference .fasta to quantify genome detection, coverage, microdiversity, or strain sharing (e.g., ZymoBIOMICS mock-community benchmarking).

## When NOT to use

- Input is already a .bam/.sam alignment or a coverage table — proceed directly to `inStrain profile` instead of re-mapping
- Only genome-level relative abundance/coverage is needed without microdiversity or SNV data — a fast tool like CoverM may be more appropriate than full inStrain profiling
- Reads are single-end or the reference genomes are not dereplicated, since un-paired reads are removed by default and highly similar genomes cause erroneous multi-mapping

## Inputs

- Paired-end metagenomic FASTQ reads (e.g., ZymoBIOMICS mock-community reads)
- Concatenated reference genome .fasta (e.g., ZymoBIOMICS.STD.refseq.v2.zip genomes)
- Bowtie2 index of the concatenated reference
- inStrain profile objects (.IS_profile) per sample for comparison

## Outputs

- Sorted .bam alignment files per sample
- inStrain profile outputs with per-genome/scaffold/gene coverage, nucleotide diversity, breadth, and SNV tables
- Pairwise popANI/ConANI and coverage-overlap comparison table from `inStrain compare`
- Genome detection table (detection defined as >= 0.5 unmaskedBreadth) for accuracy benchmarking against known community composition

## How to apply

Concatenate the reference genomes (e.g., ZymoBIOMICS.STD.refseq.v2) into a single .fasta, then map paired-end reads with Bowtie2 using default settings, increasing the -X flag up to 1000 for longer-insert datasets. Run `inStrain profile` on each resulting .bam against the same reference .fasta, applying inStrain's default read filters: un-paired reads removed, pairs mapped in proper orientation within expected insert size, user-defined minimum mapQ, and minimum read-pair ANI of 0.95. SNV calling requires at least 5 reads per position, more than 5% of reads carrying the variant base, and variant reads exceeding a null model with FDR default 1e-6. Filtered mapping is critical because careful filtering of the .bam reduces the probability of reads being erroneously mapped, especially when reference genomes share similarity (genome de-replication at 98% ANI reduces multi-mapped reads). Downstream, run `inStrain compare` on the IS_profiles to compute pairwise ConANI/popANI and coverage overlap, defining genome detection as >= 0.5 unmaskedBreadth.

## Related tools

- **Bowtie2** (Maps paired-end metagenomic reads to the concatenated reference genome set with default settings (increase -X up to 1000 for longer-insert datasets))
- **inStrain** (Profiles filtered .bam files for coverage, breadth, SNVs, and pairwise popANI/ConANI population comparisons) — github.com/MrOlm/instrain
- **Samtools** (Required by inStrain for interacting with .bam and .sam files and calculating per-position coverage)
- **dRep** (Benchmarked competing strain-comparison tool; also used for genome dereplication at 98% ANI to reduce multi-mapped reads) — github.com/MrOlm/drep
- **CoverM** (Alternative fast tool for calculating coverage and breadth of genomes from mapped reads) — github.com/wwood/CoverM

## Examples

```
bowtie2 -x ZymoBIOMICS.STD.refseq.v2 -1 sample_R1.fq -2 sample_R2.fq -X 1000 | samtools sort -o sample.bam; inStrain profile sample.bam ZymoBIOMICS.STD.refseq.v2.fasta -o IS_profile_sample
```

## Evaluation signals

- Detection of each reference genome is defined as >= 0.5 unmaskedBreadth — verify detection calls meet this threshold
- All read pairs survive inStrain's default filters: paired, proper orientation, expected insert size, minimum read-pair ANI 0.95
- SNV calls satisfy the criteria: >= 5 reads at position, > 5% variant base, and exceed the null model at FDR 1e-6
- Compiled per-tool accuracy values against known ZymoBIOMICS composition reproduce the Figure 2c comparison and Supplemental Table S1 format
- Pairwise popANI and coverage-overlap values are computed for every pair of samples by `inStrain compare`

## Limitations

- Variant sites near the ends of genome fragments can be missed with inStrain
- Genome de-replication at 98% ANI is needed to reduce multi-mapped reads when reference genomes are similar
- inStrain's Python implementation is slow in places (e.g., node2vec should call the C++ library)

## Evidence

- [readme] inStrain profile CLI from README: [evidence span withheld — anchor status: external_doc]

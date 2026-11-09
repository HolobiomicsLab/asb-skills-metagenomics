---
name: mock-community-benchmark-accuracy-evaluation
description: Use when a new strain- or genome-comparison method needs quantitative
  accuracy validation on community data with known ground-truth composition, e.g.
  reproducing the inStrain vs dRep/MIDAS/StrainPhlAn benchmark on ZymoBIOMICS.
when_to_use_negative:
- The community composition is unknown, so there is no ground truth against which
  to score comparison accuracy
- The goal is functional profiling or abundance estimation rather than strain-level
  genomic comparison accuracy benchmarking
- Reads cannot be confidently mapped to complete reference genomes for the community
  members (e.g. only de novo assemblies exist)
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: Bowtie2
  role: Maps mock-community paired-end reads to the concatenated ZymoBIOMICS reference
    genome set (default settings; -X up to 1000 for longer inserts)
- name: inStrain
  role: Profiles each sample .bam (`inStrain profile`) and computes pairwise popANI/coverage
    overlap between samples (`inStrain compare`)
  repo: github.com/MrOlm/instrain
- name: Samtools
  role: Required by inStrain for interacting with .bam and .sam mapping files
- name: dRep
  role: Competing genome-comparison tool benchmarked on the same reads and reference
    genomes
  repo: github.com/MrOlm/drep
- name: MIDAS
  role: Competing strain-comparison tool benchmarked on the same reads and reference
    genomes
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:58aecaecebcd562730677da5dfa6a76804f2019c20e7c861c004da1b1ce10384
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bowtie2
  - MIDAS
  - Samtools
  - dRep
  - inStrain
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# mock-community-benchmark-accuracy-evaluation

## Summary

Benchmark the accuracy of strain/genome comparison tools against a mock community of known composition (ZymoBIOMICS Microbial Community Standard, Catalog #D6300), scoring each tool's pairwise genomic comparisons against ground truth as in Strain Figure 2c.

## When to use

Use when a new strain- or genome-comparison method needs quantitative accuracy validation on community data with known ground-truth composition, e.g. reproducing the inStrain vs dRep/MIDAS/StrainPhlAn benchmark on ZymoBIOMICS.

## When NOT to use

- The community composition is unknown, so there is no ground truth against which to score comparison accuracy
- The goal is functional profiling or abundance estimation rather than strain-level genomic comparison accuracy benchmarking
- Reads cannot be confidently mapped to complete reference genomes for the community members (e.g. only de novo assemblies exist)

## Inputs

- ZymoBIOMICS mock community paired-end Illumina metagenomic reads
- ZymoBIOMICS.STD.refseq.v2 concatenated reference genome .fasta (https://s3.amazonaws.com/zymo-files/BioPool/ZymoBIOMICS.STD.refseq.v2.zip)
- Known ZymoBIOMICS community composition (ground truth)
- Bowtie2 index / mapping .bam files per sample

## Outputs

- inStrain IS_profile directories per sample (coverage, microdiversity, SNVs)
- Pairwise popANI and coverage-overlap table from `inStrain compare`
- Supplemental Table S1-format table of raw per-comparison accuracy values for each benchmarked tool

## How to apply

Map mock-community paired-end reads to the concatenated ZymoBIOMICS.STD.refseq.v2 reference set using Bowtie2 with default settings (increase the -X flag up to 1000 for longer-insert datasets). Run `inStrain profile` on each sample .bam against the reference .fasta with default read filters (paired reads only, proper orientation within expected insert size, minimum read-pair ANI 0.95), then run `inStrain compare` on all IS_profiles to compute pairwise popANI and coverage overlap. Define detection of each reference genome as >= 0.5 unmaskedBreadth, and run the competing tools (dRep, MIDAS, StrainPhlAn) on the same reads and reference genomes. Compile raw per-comparison accuracy values of each tool against the known community composition into a Supplemental Table S1-format table; the result reproduces the tool-accuracy comparison of Figure 2c, demonstrating that microdiversity-aware population comparison dramatically increases genomic comparison accuracy.

## Related tools

- **Bowtie2** (Maps mock-community paired-end reads to the concatenated ZymoBIOMICS reference genome set (default settings; -X up to 1000 for longer inserts))
- **inStrain** (Profiles each sample .bam (`inStrain profile`) and computes pairwise popANI/coverage overlap between samples (`inStrain compare`)) — github.com/MrOlm/instrain
- **Samtools** (Required by inStrain for interacting with .bam and .sam mapping files)
- **dRep** (Competing genome-comparison tool benchmarked on the same reads and reference genomes) — github.com/MrOlm/drep
- **MIDAS** (Competing strain-comparison tool benchmarked on the same reads and reference genomes)

## Examples

```
inStrain profile mapping.bam ZymoBIOMICS.STD.refseq.v2.fasta -o IS_profile1; inStrain compare IS_profile1 IS_profile2
```

## Evaluation signals

- Each reference genome's detection is called consistently with the >= 0.5 unmaskedBreadth criterion
- The compiled accuracy table matches Supplemental Table S1 format with raw per-comparison values
- The benchmark reproduces the Figure 2c tool-accuracy comparison, with inStrain popANI showing the highest genomic comparison accuracy
- inStrain profile used default read filters: paired reads only, proper orientation, expected insert size, minimum read-pair ANI 0.95

## Limitations

- Raw benchmarking values are deferred to Supplemental Table S1 and not included in the main text
- Variant sites near the ends of genome fragments can be missed by inStrain
- Genome de-replication at 98% ANI is recommended to reduce multi-mapped reads; multi-mapping can otherwise distort comparisons
- The Python implementation is slow; the changelog notes it should call the node2vec C++ library

## Evidence

- [readme] Pairwise comparison module: [evidence span withheld — anchor status: external_doc]
- [readme] Profiling invocation: [evidence span withheld — anchor status: external_doc]

---
name: large-scale-metagenome-compute-management
description: Use when many (e.g., cohort-scale) paired-end metagenome FASTQ files
  must each be de novo assembled and assembly statistics (contig count, total bp,
  N50) must be reported or reproduced.
when_to_use_negative:
- The goal is read-level profiling against an existing reference — use VIRGO's runMapping
  workflow instead of re-assembling
- Only a single small metagenome is being processed and no cohort-scale compute planning
  is needed — a plain local IDBA-UD run suffices without this management overhead
- Input data are metatranscriptomic or amplicon reads where de novo assembly is not
  the intended route
edam_operation: http://edamontology.org/operation_0526
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0196
tools:
- name: IDBA-UD
  role: De novo assembler run with k-mer values spanning 20 to 100 to assemble each
    preprocessed vaginal metagenome
- name: VIRGO
  role: Downstream gene catalog whose construction consumed the assembled contigs;
    repository provides mapping and annotation modules for the resulting database
  repo: github.com/ravel-lab/VIRGO
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-14677-3
    title: A comprehensive non-redundant gene catalog reveals extensive within-community
      intraspecies diversity in the human vagina
  source_content_hash: sha256:7c1967667a1c45280348371ef8ed1b6a936d7f00863619cf657bc0b96b8e3296
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IDBA-UD
  - VIRGO
  edam_operation: http://edamontology.org/operation_0526
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# large-scale-metagenome-compute-management

## Summary

De novo assembles a large cohort of preprocessed vaginal metagenomes with IDBA-UD (k-mer 20–100) and tracks the combined assembly statistics — number of contigs, total assembled bases, and N50 — that feed construction of the VIRGO gene catalog.

## When to use

Use when many (e.g., cohort-scale) paired-end metagenome FASTQ files must each be de novo assembled and assembly statistics (contig count, total bp, N50) must be reported or reproduced.

## When NOT to use

- The goal is read-level profiling against an existing reference — use VIRGO's runMapping workflow instead of re-assembling
- Only a single small metagenome is being processed and no cohort-scale compute planning is needed — a plain local IDBA-UD run suffices without this management overhead
- Input data are metatranscriptomic or amplicon reads where de novo assembly is not the intended route

## Inputs

- Preprocessed paired-end vaginal metagenome reads (FASTQ) for the assembly cohort
- IDBA-UD v1.0 executable

## Outputs

- contigs.fa — final assembled contigs per sample in FASTA format
- Assembly statistics: total number of contigs (combined ~1.2 million), total assembled bases (~2.8 billion bp), and N50 (6.2 kbp)

## How to apply

Obtain the preprocessed paired-end vaginal metagenome reads for the assembly cohort, then run IDBA-UD v1.0 in de novo assembly mode with k-mer values spanning 20 to 100 on each preprocessed sample. Collect the final contigs from the IDBA-UD output and save them as contigs.fa in FASTA format. Compute assembly statistics directly from contigs.fa: total number of contigs, total assembled bases in bp, and N50. Validate that contigs.fa exists and is valid FASTA and that the computed statistics are consistent with the reported VIRGO assembly values of 1.2 million contigs, 2.8 billion bp total, and N50 6.2 kbp. Plan compute accordingly — the article notes that de novo assembly of metagenomic datasets typically requires substantial computational resources, so cohort-scale runs should be queued per-sample rather than attempted in one job.

## Related tools

- **IDBA-UD** (De novo assembler run with k-mer values spanning 20 to 100 to assemble each preprocessed vaginal metagenome)
- **VIRGO** (Downstream gene catalog whose construction consumed the assembled contigs; repository provides mapping and annotation modules for the resulting database) — github.com/ravel-lab/VIRGO

## Examples

```
idba_ud -r paired_reads.fa --mink 20 --maxk 100 -o vaginal_assembly
```

## Evaluation signals

- contigs.fa exists and parses as valid FASTA for every assembled sample
- Combined assembly statistics match the reported VIRGO values: ~1.2 million contigs, ~2.8 billion bp total length, N50 of 6.2 kbp
- IDBA-UD was run with the specified k-mer range (20 to 100) as recorded in logs/parameters
- Per-sample contigs.fa outputs are present for all samples in the assembly cohort (no missing samples)
- Computed N50, contig count, and total length are internally consistent with the contigs.fa contents

## Limitations

- De novo assembly of metagenomic or metatranscriptomic datasets typically requires rather substantial computational resources and complicates integration of metagenomic and metatranscriptomic data
- This skill reproduces cohort-level assembly for building VIRGO; VIRGO itself later acknowledged limitations of its reference-based downstream approach for gene-level de-redundancy

## Evidence

- [intro] De novo assembly of metagenomic or metatranscriptomic datasets typically requires rather substantial computational resources and complicates integration of metagenomic and metatranscriptomic data.: "De novo assembly of metagenomic or metatranscriptomic datasets typically requires rather substantial computational resources and complicates integration of metagenomic and metatranscriptomic data."
- [other] In the VIRGO assembly step, each preprocessed vaginal metagenome was de novo assembled, yielding a combined set of 1.2 million contigs totaling 2.8 billion bp with an N50 of 6.2 kbp: [evidence span withheld — anchor status: not_found]
- [readme] The complete VIRGO and VOG databases, along with the set of vaginal metagenomes used to build and test VIRGO are available for download: "The complete VIRGO and VOG databases, along with the set of vaginal metagenomes used to build and test VIRGO are available for download at: http://virgo.igs.umaryland.edu/"

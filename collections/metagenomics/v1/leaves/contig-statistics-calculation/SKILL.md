---
name: contig-statistics-calculation
description: Use when you have assembled contigs (e.g., a FASTA like contigs.fa from
  IDBA-UD) and must report assembly quality metrics such as contig count, total length,
  and N50.
when_to_use_negative:
- Input reads have not yet been assembled — this skill computes statistics on existing
  contigs, it does not perform the IDBA-UD assembly itself
- You need per-sample gene abundance or mapping statistics against VIRGO (use VIRGO's
  runMapping.step1.sh / runMapping.step2.sh summarization instead)
- The assembly is of non-vaginal data and the goal is comparison against the VIRGO
  reference values (1.2 million contigs, 2.8 billion bp, N50 6.2 kbp), which are cohort-specific
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0193
- http://edamontology.org/topic_0622
tools:
- name: IDBA-UD
  role: De novo metagenomic assembler run in de novo assembly mode with k-mer values
    spanning 20 to 100 to produce the contigs whose statistics are computed
- name: seqtk
  role: FASTA sequence utility (required VIRGO dependency, version 1.0+) usable to
    parse contigs.fa and obtain per-contig sequence lengths for computing contig count,
    total length, and N50
- name: VIRGO
  role: The downstream gene catalog whose reported assembly statistics (1.2 million
    contigs, 2.8 billion bp, N50 6.2 kbp) serve as the validation reference for the
    computed metrics
  repo: github.com/ravel-lab/VIRGO
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-14677-3
    title: A comprehensive non-redundant gene catalog reveals extensive within-community
      intraspecies diversity in the human vagina
  source_content_hash: sha256:95e9b50f0e536e75fe6b35f25a8bc38a9984501f39f91bbd520984db9cec1fb4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IDBA-UD
  - VIRGO
  - seqtk
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0193
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# contig-statistics-calculation

## Summary

Compute de novo assembly statistics (number of contigs, total assembled length in bp, and N50) from metagenomic assemblies, as done for the vaginal metagenomes assembled to construct the VIRGO gene catalog.

## When to use

Use when you have assembled contigs (e.g., a FASTA like contigs.fa from IDBA-UD) and must report assembly quality metrics such as contig count, total length, and N50.

## When NOT to use

- Input reads have not yet been assembled — this skill computes statistics on existing contigs, it does not perform the IDBA-UD assembly itself
- You need per-sample gene abundance or mapping statistics against VIRGO (use VIRGO's runMapping.step1.sh / runMapping.step2.sh summarization instead)
- The assembly is of non-vaginal data and the goal is comparison against the VIRGO reference values (1.2 million contigs, 2.8 billion bp, N50 6.2 kbp), which are cohort-specific

## Inputs

- contigs.fa — assembled contigs in FASTA format, output of IDBA-UD de novo assembly (k-mer 20-100) of preprocessed paired-end vaginal metagenome reads
- preprocessed paired-end metagenome reads (FASTQ) for the assembly cohort (input to the assembler)

## Outputs

- Assembly statistics table: number of contigs, total assembled length in bp, N50 in kbp
- Validated contigs.fa in FASTA format

## How to apply

Obtain preprocessed paired-end vaginal metagenome reads and run IDBA-UD v1.0 in de novo assembly mode with k-mer values spanning 20 to 100, then collect the final assembled contigs into a FASTA file (contigs.fa). Parse contigs.fa to compute three statistics: the total number of contigs, the total assembled bases in bp, and the N50 (the shortest contig length at which half of the total assembled bases are contained in contigs of that length or longer). Validation requires confirming contigs.fa exists and is valid FASTA, and that the computed values are consistent with the reported VIRGO assembly values of 1.2 million contigs, 2.8 billion bp, and an N50 of 6.2 kbp. Utilities from the VIRGO toolchain (e.g., seqtk 1.0+) can be used to parse FASTA sequences and their lengths.

## Related tools

- **IDBA-UD** (De novo metagenomic assembler run in de novo assembly mode with k-mer values spanning 20 to 100 to produce the contigs whose statistics are computed)
- **seqtk** (FASTA sequence utility (required VIRGO dependency, version 1.0+) usable to parse contigs.fa and obtain per-contig sequence lengths for computing contig count, total length, and N50)
- **VIRGO** (The downstream gene catalog whose reported assembly statistics (1.2 million contigs, 2.8 billion bp, N50 6.2 kbp) serve as the validation reference for the computed metrics) — github.com/ravel-lab/VIRGO

## Examples

```
seqtk comp contigs.fa | awk '{t+=$2; n++; l[n]=$2} END{asort(l); h=t/2; c=0; for(i=1;i<=n;i++){c+=l[i]; if(c>=h){print n, t, l[i]; exit}}}'
```

## Evaluation signals

- contigs.fa exists and passes a valid-FASTA check (headers plus A/C/G/T/N sequences only)
- Computed number of contigs matches the reported ~1.2 million combined contigs
- Computed total assembled length matches the reported 2.8 billion bp
- Computed N50 matches the reported 6.2 kbp
- Per-contig lengths parsed from contigs.fa sum exactly to the reported total assembled bases (internal consistency check)

## Limitations

- De novo assembly of metagenomic datasets requires substantial computational resources and complicates integration of metagenomic and metatranscriptomic data
- The reference values (1.2 million contigs, 2.8 billion bp, N50 6.2 kbp) are specific to the combined vaginal metagenome assembly cohort used to build VIRGO and will not match other datasets
- The assembler version (IDBA-UD v1.0) and k-mer range (20-100) must match the VIRGO construction protocol for reproducibility

## Evidence

- [methods] IDBA-UD de novo assembly with k 20-100: [evidence span withheld — anchor status: not_found]
- [methods] assembly statistics definition: [evidence span withheld — anchor status: not_found]
- [other] reported VIRGO assembly statistics: [evidence span withheld — anchor status: not_found]
- [readme] seqtk dependency: "> BLAST 2.2.3+ 
> Bowtie 1.1.2+
> GNU Awk 3.1+
> seqtk 1.0+"
- [intro] computational burden of assembly: "De novo assembly of metagenomic or metatranscriptomic datasets typically requires rather substantial computational resources"

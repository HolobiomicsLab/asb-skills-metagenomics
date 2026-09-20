---
name: assembly-quality-assessment
description: Use when preprocessed paired-end metagenomic reads must be de novo assembled
  and the resulting contigs must be quality-checked via contig count, total length
  (bp), and N50, e.g. to reproduce VIRGO's assembly cohort statistics.
when_to_use_negative:
- Input is already a set of assembled contigs or a finished gene catalog (e.g. VIRGO's
  0_db sequences) — mapping or annotation skills apply instead
- The goal is taxonomic/functional profiling of a sample against VIRGO — use the reference-based
  mapping workflow rather than re-assembling
- Working with metatranscriptomic-only data where the article notes de novo assembly
  'complicates integration of metagenomic and metatranscriptomic data'
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: IDBA-UD
  role: De novo assembler run with k-mer values 20-100 on preprocessed vaginal metagenome
    reads to produce contigs.fa
- name: VIRGO
  role: Downstream non-redundant gene catalog and toolkit whose assembly step output
    statistics (1.2M contigs, 2.8 Gbp, N50 6.2 kbp) serve as the validation target
  repo: github.com/ravel-lab/VIRGO
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-14677-3
    title: A comprehensive non-redundant gene catalog reveals extensive within-community
      intraspecies diversity in the human vagina
  source_content_hash: sha256:02b61cc3acc59945cddec53586da6a8398cfe4a8a80fe60a35fb779777a240e7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IDBA-UD
  - VIRGO
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# assembly-quality-assessment

## Summary

De novo assembly of vaginal metagenome reads with IDBA-UD (k-mer 20-100) followed by computation of assembly statistics — number of contigs, total assembled bp, and N50 — used to build and validate the VIRGO non-redundant gene catalog.

## When to use

Use when preprocessed paired-end metagenomic reads must be de novo assembled and the resulting contigs must be quality-checked via contig count, total length (bp), and N50, e.g. to reproduce VIRGO's assembly cohort statistics.

## When NOT to use

- Input is already a set of assembled contigs or a finished gene catalog (e.g. VIRGO's 0_db sequences) — mapping or annotation skills apply instead
- The goal is taxonomic/functional profiling of a sample against VIRGO — use the reference-based mapping workflow rather than re-assembling
- Working with metatranscriptomic-only data where the article notes de novo assembly 'complicates integration of metagenomic and metatranscriptomic data'

## Inputs

- Preprocessed paired-end vaginal metagenome reads (FASTQ) from the assembly cohort
- IDBA-UD v1.0 de novo assembler

## Outputs

- contigs.fa — final assembled contigs in FASTA format
- Assembly statistics: number of contigs, total assembled length in bp, and N50

## How to apply

Obtain the preprocessed paired-end vaginal metagenome reads for the assembly cohort, then run IDBA-UD v1.0 in de novo assembly mode with k-mer values spanning 20 to 100, which allows the assembler to resolve both low- and high-coverage regions of the heterogeneous vaginal community. Collect the final contigs from the IDBA-UD output and save them as contigs.fa in FASTA format. Compute assembly statistics directly from contigs.fa by sorting contig lengths and calculating the total number of contigs, total assembled bases in bp, and the N50 (shortest contig length at which 50% of total assembled bases are contained in contigs of that length or longer). Validate by confirming contigs.fa exists and is valid FASTA and that the computed statistics are consistent with the reported VIRGO assembly values: 1.2 million contigs, 2.8 billion bp, and N50 of 6.2 kbp. These contigs feed the downstream gene prediction and de-redundancy that yield VIRGO's 0.95 million non-redundant genes and the VOG protein families.

## Related tools

- **IDBA-UD** (De novo assembler run with k-mer values 20-100 on preprocessed vaginal metagenome reads to produce contigs.fa)
- **VIRGO** (Downstream non-redundant gene catalog and toolkit whose assembly step output statistics (1.2M contigs, 2.8 Gbp, N50 6.2 kbp) serve as the validation target) — github.com/ravel-lab/VIRGO

## Examples

```
idba_ud -k 20,40,60,80,100 -r preprocessed_vaginal_reads.fa -o virgo_asm && assembly_stats virgo_asm/contigs.fa
```

## Evaluation signals

- contigs.fa exists and parses as valid FASTA
- Computed contig count is consistent with the reported ~1.2 million contigs
- Total assembled bases are consistent with the reported 2.8 billion bp
- N50 is consistent with the reported 6.2 kbp
- Downstream gene prediction/de-redundancy on these contigs recovers a catalog on the order of VIRGO's 0.95 million non-redundant genes

## Limitations

- De novo assembly of metagenomic datasets 'typically requires rather substantial computational resources and complicates integration of metagenomic and metatranscriptomic data'
- VIRGO's reference-based downstream approach has acknowledged limitations: it is focused on gene-level de-redundancy, is primarily bacterial, and has 'limited inclusion of viral and fungal gene sequences'
- Statistics reported are cohort-level aggregates; per-sample assembly statistics will differ

## Evidence

- [intro] De novo assembly of metagenomic or metatranscriptomic datasets typically requires rather substantial computational resources and complicates integration of metagenomic and metatranscriptomic data.: "De novo assembly of metagenomic or metatranscriptomic datasets typically requires rather substantial computational resources and complicates integration of metagenomic and metatranscriptomic data."
- [abstract] we assemble a comprehensive human vaginal non-redundant gene catalog (VIRGO): "we assemble a comprehensive human vaginal non-redundant gene catalog (VIRGO)"
- [readme] VIRGO is a non-redundant gene catalog for the microbial communities that inhabit the human vagina. It was built using a combination of metagenomic and urogential isolate genome sequences.: "**VIRGO** is a non-redundant gene catalog for the microbial communities that inhabit the human vagina. It was built using a combination of metagenomic and urogential isolate genome sequences."
- [readme] The complete VIRGO and VOG databases, along with the set of vaginal metagenomes used to build and test VIRGO are available for download: "The complete VIRGO and VOG databases, along with the set of vaginal metagenomes used to build and test VIRGO are available for download at: http://virgo.igs.umaryland.edu/"

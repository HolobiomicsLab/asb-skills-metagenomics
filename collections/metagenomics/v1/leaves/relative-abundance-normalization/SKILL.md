---
name: relative-abundance-normalization
description: Use when you have reconstructed full-length SSU rRNA sequences (e.g.
  metagenome_SSU_sequences.fasta from EMIRGE/MetaRib) plus the original quality-controlled
  reads and need per-sequence relative abundances for taxonomic profiling.
when_to_use_negative:
- Input is a short-read amplicon classification task where taxa should be assigned
  directly to reads — direct classification of short Illumina reads yields a high
  rate of false positives in this pipeline's benchmarking
- Reads have not been adapter-trimmed and quality-filtered with BBTOOLS first, since
  the mapper expects the pipeline's quality-controlled trimmed reads
- You need copy-number-corrected absolute abundances — RiboTaxa intentionally does
  not use 16S rRNA gene copy number correction
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: BBMap
  role: Maps the unfiltered high-quality reads onto the reconstructed SSU rRNA sequences
    so per-sequence read counts can be tallied for relative abundance
  repo: https://jgi.doe.gov/data-and-tools/bbtools/
- name: BBTools
  role: Tool suite providing BBMap and the upstream trimming (adapter removal, minlength,
    trimq, qtrim, maxns) that yields the quality-controlled reads used as mapping
    input
  repo: https://jgi.doe.gov/data-and-tools/bbtools/
- name: RiboTaxa
  role: Pipeline orchestrating SSU reconstruction and downstream abundance estimation
    before 97% VSEARCH clustering and QIIME 2 sklearn taxonomic affiliation
  repo: https://github.com/oschakoory/RiboTaxa
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nargab/lqac070
    title: 'RiboTaxa: combined approaches for rRNA genes taxonomic resolution down
      to the species level from metagenomics data revealing novelties'
  source_content_hash: sha256:976b1c795773ab6483d9d8c7efe4dc14c0964d61f5d7691f5a57672f3cd0f4d1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/oschakoory/RiboTaxa
  tools:
  - BBMap
  - BBTools
  - RiboTaxa
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# relative-abundance-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies the relative abundance of reconstructed full-length SSU rRNA sequences in the RiboTaxa pipeline by mapping unfiltered high-quality reads onto the reconstructed sequences with BBMap (BBTools) and normalizing per-sequence read counts to percentages before 97% clustering and taxonomic affiliation.

## When to use

Use when you have reconstructed full-length SSU rRNA sequences (e.g. metagenome_SSU_sequences.fasta from EMIRGE/MetaRib) plus the original quality-controlled reads and need per-sequence relative abundances for taxonomic profiling.

## When NOT to use

- Input is a short-read amplicon classification task where taxa should be assigned directly to reads — direct classification of short Illumina reads yields a high rate of false positives in this pipeline's benchmarking
- Reads have not been adapter-trimmed and quality-filtered with BBTOOLS first, since the mapper expects the pipeline's quality-controlled trimmed reads
- You need copy-number-corrected absolute abundances — RiboTaxa intentionally does not use 16S rRNA gene copy number correction

## Inputs

- Quality-controlled trimmed metagenomic reads (FASTQ, single-end or paired-end, .fastq or .fastq.gz) from BBTOOLS trimming
- Reconstructed full-length SSU rRNA sequences file (metagenome_SSU_sequences.fasta) produced by EMIRGE/MetaRib

## Outputs

- metagenome_SSU_taxonomy_abundance.tsv with columns Sequence_ID, Assigned reads, Relative_Abundance(%)

## How to apply

Load the quality-controlled trimmed reads produced by BBTOOLS and the reconstructed full-length SSU sequences file (metagenome_SSU_sequences.fasta). Map the short reads onto the SSU sequences with BBMap from BBTools, limiting RAM to approximately 80% of available memory (e.g. RAM = 12 on a 16 GB machine). Tally the number of reads assigned to each reconstructed SSU sequence, then normalize by dividing each sequence's assigned read count by the sum of all assigned reads and multiplying by 100 so relative abundance is expressed in %. Write per-sequence abundances to metagenome_SSU_taxonomy_abundance.tsv with columns Sequence_ID, Assigned reads, and Relative_Abundance(%). Note that RiboTaxa deliberately applies no 16S rRNA gene copy number correction, so abundances reflect read-mapping proportions only.

## Related tools

- **BBMap** (Maps the unfiltered high-quality reads onto the reconstructed SSU rRNA sequences so per-sequence read counts can be tallied for relative abundance) — https://jgi.doe.gov/data-and-tools/bbtools/
- **BBTools** (Tool suite providing BBMap and the upstream trimming (adapter removal, minlength, trimq, qtrim, maxns) that yields the quality-controlled reads used as mapping input) — https://jgi.doe.gov/data-and-tools/bbtools/
- **RiboTaxa** (Pipeline orchestrating SSU reconstruction and downstream abundance estimation before 97% VSEARCH clustering and QIIME 2 sklearn taxonomic affiliation) — https://github.com/oschakoory/RiboTaxa

## Examples

```
bbmap.sh ref=metagenome_SSU_sequences.fasta in=trimmed_reads.fastq.gz out=mapping.sam ram=12
```

## Evaluation signals

- Output file metagenome_SSU_taxonomy_abundance.tsv exists and contains the required headers: Sequence_ID, Assigned reads, Relative_Abundance(%)
- Relative abundance values sum to approximately 100%
- Every Sequence_ID in the abundance table corresponds to a sequence present in metagenome_SSU_sequences.fasta
- Assigned read counts per sequence are non-negative integers and their sum equals the number of mapped reads reported by BBMap
- Abundances are computed on the reconstructed SSU sequences (pre-97% VSEARCH clustering), not on clustered OTU representatives

## Limitations

- No 16S rRNA gene copy number normalization is applied, so abundances can be biased by differing operon copy numbers
- Artificial in-silico diversity created during full-length sequence reconstruction cannot be excluded and may fragment abundance across spurious sequences
- Occasional small indel errors in reconstructed sequences can occur, though in practice these rare indels have little effect on taxonomic classification
- RiboTaxa is slower than short-read classifiers such as Kraken2 and Centrifuge

## Evidence

- [methods] the reconstructed SSU sequences serve as the reference: unfiltered high-quality reads are mapped onto them with BBmap v38.87: [evidence span withheld — anchor status: not_found]
- [methods] Map the short reads onto the SSU sequences with BBMap from BBTOOLS, limiting RAM to approximately 80% of available memory (e.g. RAM = 12 for a 16 GB machine).: [evidence span withheld — anchor status: not_found]

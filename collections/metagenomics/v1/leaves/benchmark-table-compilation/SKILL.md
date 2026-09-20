---
name: benchmark-table-compilation
description: Use when two or more assembly versions (or assemblers) must be compared
  under identical settings and the results tabulated, e.g. contig/scaffold N50, coverage
  percentages, or per-sequence representation rates of a reference set.
when_to_use_negative:
- The comparison involves metagenomic or single-cell data where MEGAHIT rather than
  SOAPdenovo is the appropriate assembler
- Input is a single assembly with no predecessor version to serve as a baseline for
  side-by-side tabulation
- The reference set (e.g. SD sequences) is not fixed between versions, so per-sequence
  coverage rates would not be comparable
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: SOAPdenovo2
  role: Assembles the updated YH v2 genome whose contigs are aligned to SD sequences
    and benchmarked on Assemblathon1 and GAGE datasets
  repo: github.com/aquaskyline/SOAPdenovo2
- name: SOAPdenovo1.05
  role: Predecessor assembler whose Assemblathon1 output provides the baseline column
    for the benchmark comparison
- name: GapCloser
  role: SOAPdenovo2 module whose improvements contribute to the gap-closure column
    of the benchmark table
- name: ALLPATHS-LG
  role: External assembler whose Assemblathon1 N50 values are tabulated against SOAPdenovo2
    (1.53x contig, 1.84x scaffold longer)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/2047-217x-1-18
    title: 'SOAPdenovo2: an empirically improved memory-efficient short-read <i>de
      novo</i> assembler'
  source_content_hash: sha256:a6947d2c8130d1629c34af9958803177630a435024f85052c978054aecc6d7b7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ALLPATHS-LG
  - GapCloser
  - SOAPdenovo1.05
  - SOAPdenovo2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# benchmark-table-compilation

## Summary

Compiles side-by-side benchmark tables that quantify assembly improvement of a new assembler release against its predecessor, as done when SOAPdenovo2-updated YH assembly was tabulated against the 2008 YH v1 for SD sequence representation (21.5% vs ~99%), N50, genome coverage, and error rates.

## When to use

Use when two or more assembly versions (or assemblers) must be compared under identical settings and the results tabulated, e.g. contig/scaffold N50, coverage percentages, or per-sequence representation rates of a reference set.

## When NOT to use

- The comparison involves metagenomic or single-cell data where MEGAHIT rather than SOAPdenovo is the appropriate assembler
- Input is a single assembly with no predecessor version to serve as a baseline for side-by-side tabulation
- The reference set (e.g. SD sequences) is not fixed between versions, so per-sequence coverage rates would not be comparable

## Inputs

- Contigs/scaffolds of the first-version YH (2008) assembly (FASTA)
- Contigs/scaffolds of the SOAPdenovo2-updated YH v2 assembly (FASTA *.contig, *.scafSeq)
- 134 Mb of published human SD sequences from the hg19 UCSC segmental duplication track
- NCBI human reference genome hg19
- Illumina short-read libraries configured via SOAPdenovo config file (FASTA/FASTQ/BAM)

## Outputs

- Two-column comparison table of SD representation rates (YH v1: 21.5% vs YH v2: ~99% at ≥90% coverage per SD sequence)
- Per-assembly N50 table (~20.9 kbp contig / ~22 Mbp scaffold for YH v2)
- Genome coverage comparison (81.16% vs 93.91% vs hg19)
- Error-class reduction table (structural −90.12%, substitution −92.13%, copy number −69.47% vs SOAPdenovo1)

## How to apply

Run each assembler (or assembler version) on the same input reads, e.g. SOAPdenovo2 vs SOAPdenovo1.05 on the YH Asian genome, producing *.contig and *.scafSeq outputs. Align each assembly's contigs to a fixed reference set — here 134 Mb of published human SD sequences from the hg19 segmental duplication annotation — under identical aligner settings for both versions, recording per-sequence coverage. Compute per-assembly summary fractions (e.g. fraction of SD sequences covered at ≥90% of length) and global metrics: contig and scaffold N50, genome coverage by alignment to NCBI human reference hg19, and error classes (structural, substitution, copy number). Tabulate version-paired values side by side (YH v1: 21.5% SD representation, 81.16% coverage; YH v2: ~99%, 93.91%). Validate by checking the table contains both versions' rates and reproduces the reported directional contrast.

## Related tools

- **SOAPdenovo2** (Assembles the updated YH v2 genome whose contigs are aligned to SD sequences and benchmarked on Assemblathon1 and GAGE datasets) — github.com/aquaskyline/SOAPdenovo2
- **SOAPdenovo1.05** (Predecessor assembler whose Assemblathon1 output provides the baseline column for the benchmark comparison)
- **GapCloser** (SOAPdenovo2 module whose improvements contribute to the gap-closure column of the benchmark table)
- **ALLPATHS-LG** (External assembler whose Assemblathon1 N50 values are tabulated against SOAPdenovo2 (1.53x contig, 1.84x scaffold longer))

## Examples

```
SOAPdenovo2 all -s config_file -K 63 -R -o graph_prefix 1>ass.log 2>ass.err
```

## Evaluation signals

- Output table contains both SD representation rates and reproduces the reported contrast: 21.5% for YH v1 vs up to ~99% for YH v2
- YH v2 contig and scaffold N50 (~20.9 kbp and ~22 Mbp) are approximately 3-fold and 50-fold longer than the first published version
- Genome coverage values match reported 81.16% (v1) vs 93.91% (v2) when aligned to hg19
- Error reductions vs SOAPdenovo1 fall in reported ranges: structural −90.12%, substitution −92.13%, copy number −69.47%
- Both assembly versions were aligned to the same 134 Mb SD reference under identical aligner settings

## Limitations

- SOAPdenovo2 for human-sized genomes requires about 150 GB physical memory and a 64-bit Linux system
- K-mer size is bounded (min 13, max 63/127), and multi-kmer use requires the -m option; README FAQ separately states odd values 13–31, an inconsistency to verify per build
- Benchmark contrasts are specific to the YH genome, Assemblathon1, and GAGE datasets; rates may not transfer to other genomes or read sets

## Evidence

- [results] YH v1 vs v2 SD representation: "we aligned the contigs of the first version and the new version to 134 Mb of published human SD sequences"
- [abstract] N50 improvement: "the contig and scaffold N50 of the YH genome were ~20.9 kbp and ~22 Mbp, respectively, which is 3-fold and 50-fold longer than the first published version"
- [abstract] benchmark scope: "Benchmark using the Assemblathon1 and GAGE datasets showed that SOAPdenovo2 greatly surpasses its predecessor SOAPdenovo and is competitive to other assemblers on both assembly length and accuracy."
- [readme] identical run recipe: [evidence span withheld — anchor status: external_doc]
- [readme] memory caveat: [evidence span withheld — anchor status: external_doc]

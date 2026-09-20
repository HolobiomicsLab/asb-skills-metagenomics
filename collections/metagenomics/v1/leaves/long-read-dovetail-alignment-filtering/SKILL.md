---
name: long-read-dovetail-alignment-filtering
description: 'Use when long reads must join strain-separated contigs into scaffolds:
  you have a strain-oblivious/strain-separated contig FASTA, a long-read FASTQ, and
  need dovetail read-to-contig links to scaffold within a strain-resolved pipeline.'
when_to_use_negative:
- The input is a raw long-read metagenome that still needs de novo strain separation
  (strain separation via variant calling and haplotyping must precede scaffolding).
- High strain variability requires splitting a strain-oblivious contig into a variable
  number of strains within the same contig — a case Strainberry does not handle.
- Only short-read data are available; the skill relies on error-prone long-read dovetail
  alignments.
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: Strainberry
  role: 'Performs the strain-aware scaffolding: filters dovetail alignments, builds
    and trims the bi-directed string graph, and writes assembly.scaffolds.fa and assembly.scaffolds.bam'
  repo: https://github.com/rvicedomini/strainberry
- name: minimap2
  role: Aligns long reads to the strain-separated contigs (and later to the scaffolds)
    to supply the read-to-contig dovetail alignments
- name: samtools
  role: Sorts and indexes the minimap2 alignment BAM and indexes the contig FASTA
    so filtered dovetail links can be traversed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-24515-9
    title: 'Strainberry: automated strain separation in low-complexity metagenomes
      using long reads'
  source_content_hash: sha256:31ad3b0210ba658ca6f9e8499b32b6180ccbaac0a3371d7e1b2a523266ef82a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rvicedomini/strainberry
  tools:
  - Strainberry
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Long-read dovetail alignment filtering for strain-aware scaffolding

## Summary

Filters high-confidence long-read-to-contig dovetail alignments (e.g. MAPQ > 40) and uses them to build and trim a bi-directed weighted string graph that joins strain-separated contigs into longer scaffolds, as done by Strainberry's scaffolding component.

## When to use

Use when long reads must join strain-separated contigs into scaffolds: you have a strain-oblivious/strain-separated contig FASTA, a long-read FASTQ, and need dovetail read-to-contig links to scaffold within a strain-resolved pipeline.

## When NOT to use

- The input is a raw long-read metagenome that still needs de novo strain separation (strain separation via variant calling and haplotyping must precede scaffolding).
- High strain variability requires splitting a strain-oblivious contig into a variable number of strains within the same contig — a case Strainberry does not handle.
- Only short-read data are available; the skill relies on error-prone long-read dovetail alignments.

## Inputs

- Strain-oblivious or strain-separated metagenome assembly in FASTA format (e.g. metaFlye/ssFlye contigs), indexed with samtools faidx
- Long-read FASTQ reads (PacBio or Oxford Nanopore)
- Coordinate-sorted, samtools-indexed read-to-contig alignment in BAM format (produced with minimap2 and samtools sort/index)

## Outputs

- Strain-aware scaffolds in assembly.scaffolds.fa (with .fai index)
- Minimap2-based read alignment of input reads on the scaffolds in assembly.scaffolds.bam (with .bai index)

## How to apply

First align the input long reads to the strain-separated contigs with minimap2 and coordinate-sort and index the result with samtools, so Strainberry consumes a coordinate-sorted BAM plus a samtools-faidx-indexed FASTA. Filter the read-to-contig dovetail alignments, keeping only high-confidence matches with MAPQ > 40. Build the bi-directed weighted string graph over contig ends, admitting edges according to Strainberry's four edge-allowance rules, then trim the graph by removing transitive edges and weak edges supported by fewer than 10 reads or less than 90% of a contig end's outgoing read support. Trace maximal unambiguous paths through the trimmed graph to define scaffolds, estimating inter-contig gaps as the median distance between reads spanning adjacent contigs; outputs are written to assembly.scaffolds.fa with a minimap2-based read alignment in assembly.scaffolds.bam.

## Related tools

- **Strainberry** (Performs the strain-aware scaffolding: filters dovetail alignments, builds and trims the bi-directed string graph, and writes assembly.scaffolds.fa and assembly.scaffolds.bam) — https://github.com/rvicedomini/strainberry
- **minimap2** (Aligns long reads to the strain-separated contigs (and later to the scaffolds) to supply the read-to-contig dovetail alignments)
- **samtools** (Sorts and indexes the minimap2 alignment BAM and indexes the contig FASTA so filtered dovetail links can be traversed)

## Examples

```
minimap2 -ax map-pb -t 12 ./flye_out/assembly.fasta reads.fq.gz | samtools sort >./flye_out/alignment.sorted.bam
```

## Evaluation signals

- Every retained dovetail alignment has MAPQ > 40; alignments at or below this threshold are excluded from the graph.
- Weak edges are absent: all graph edges are supported by at least 10 reads and at least 90% of a contig end's outgoing read support.
- Output directory contains assembly.scaffolds.fa (with .fai) and assembly.scaffolds.bam (with .bai), and scaffolds are longer than the input strain-separated contigs.
- Benchmarking against reference strain genomes yields near-complete reference coverage and approximately 99.9% base accuracy on mock communities such as Mock3.

## Limitations

- Limited read length (around N50 of 5 Kbp, constrained by DNA extraction methods) restricts the reads available for dovetail links between contig ends.
- Haplotype phasing's even-coverage-split assumption does not hold in metagenomes where strains have different abundances; separation is demonstrated only for a limited number of well-represented strains.
- In presence of high strain variability, strain-oblivious sequences might need separation into a variable number of strains within the same contig, which is not handled.

## Evidence

- [intro] Strainberry combines a strain-oblivious assembler with the careful use of a long-read variant calling and haplotyping tool, followed by a component that performs long-read metagenome scaffolding.: "Strainberry combines a strain-oblivious assembler with the careful use of a long-read variant calling and haplotyping tool, followed by a component that performs long-read metagenome scaffolding."
- [discussion] Strainberry makes use of well-established tools for haplotype phasing and genome assembly and introduces a strain-aware scaffolding component: "Strainberry makes use of well-established tools for haplotype phasing and genome assembly and introduces a strain-aware scaffolding component"
- [readme] Both FASTA and BAM files are expected to be indexed with the samtools faidx and samtools index commands, respectively.: "Both `FASTA` and `BAM` files are expected to be indexed with the `samtools faidx` and `samtools index` commands, respectively."
- [readme] Strainberry output assembly is stored in the assembly.scaffolds.fa file.: "Strainberry output assembly is stored in the `assembly.scaffolds.fa` file."
- [methods] minimap2 -ax map-pb -t 12 ./flye_out/assembly.fasta reads.fq.gz | samtools sort >./flye_out/alignment.sorted.bam: "minimap2 -ax map-pb -t 12 ./flye_out/assembly.fasta reads.fq.gz | samtools sort >./flye_out/alignment.sorted.bam"

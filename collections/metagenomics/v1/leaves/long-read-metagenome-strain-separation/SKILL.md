---
name: long-read-metagenome-strain-separation
description: Use when a low-complexity single-sample long-read metagenome dataset
  (PacBio BAM or Oxford Nanopore BAM, '--nanopore' flag for ONT) needs strain-resolved
  assemblies rather than a single consensus, e.g., to quantify coverage of individual
  reference strains like L. helveticus NWC_2_3/NWC_2_4.
when_to_use_negative:
- High-complexity metagenomes with many strains or uneven abundance — Strainberry
  is designed for single-sample low-complexity metagenomes and a limited number of
  well-represented strains
- Contigs needing separation into a variable number of strains within the same contig
  (high strain variability) — not handled by the pipeline
- Only short-read data available — all strain-level assembly steps rely uniquely on
  long reads and cannot reuse short-read methods
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: Strainberry
  role: 'Core pipeline: variant calling, haplotype phasing, and strain-aware scaffolding
    on the strain-oblivious assembly; outputs assembly.scaffolds.fa'
  repo: https://github.com/rvicedomini/strainberry
- name: metaFlye
  role: Generates the strain-oblivious metagenome assembly input (e.g., 'flye --meta
    --pacbio-raw reads.fq.gz --out-dir flye_out --genome-size 300k --threads 12')
- name: minimap2
  role: Aligns long reads to the strain-oblivious assembly ('-ax map-pb' for PacBio)
    and to the output scaffolds for coverage evaluation
- name: samtools
  role: Sorts alignments, indexes the assembly FASTA ('samtools faidx') and the BAM
    ('samtools index') as required inputs to Strainberry
- name: Canu
  role: Alternative strain-oblivious long-read assembler used to produce assembly/BAM
    pairs for the NWC2 comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-24515-9
    title: 'Strainberry: automated strain separation in low-complexity metagenomes
      using long reads'
  source_content_hash: sha256:df278223fa965799ea7f0cf2c772b78c56bdf76c6ec98ca190d3f00cd8549249
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rvicedomini/strainberry
  tools:
  - Canu
  - Strainberry
  - metaFlye
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# long-read-metagenome-strain-separation

## Summary

A pipeline skill for strain-separated de novo metagenome assembly from a single sample using error-prone long reads, implemented by Strainberry on top of a strain-oblivious assembly (e.g., metaFlye/Canu) plus read alignment. It recovers strain-level sequence (haplotype-level) assemblies and additional genomic material that conventional strain-oblivious assemblers miss.

## When to use

Use when a low-complexity single-sample long-read metagenome dataset (PacBio BAM or Oxford Nanopore BAM, '--nanopore' flag for ONT) needs strain-resolved assemblies rather than a single consensus, e.g., to quantify coverage of individual reference strains like L. helveticus NWC_2_3/NWC_2_4.

## When NOT to use

- High-complexity metagenomes with many strains or uneven abundance — Strainberry is designed for single-sample low-complexity metagenomes and a limited number of well-represented strains
- Contigs needing separation into a variable number of strains within the same contig (high strain variability) — not handled by the pipeline
- Only short-read data available — all strain-level assembly steps rely uniquely on long reads and cannot reuse short-read methods

## Inputs

- Strain-oblivious long-read metagenome assembly in FASTA format (e.g., metaFlye assembly.fasta), indexed with samtools faidx
- Coordinate-sorted long-read alignment in BAM format (minimap2 map-pb for PacBio or ONT preset), indexed with samtools index
- Long reads in FASTQ (PacBio or Oxford Nanopore; use --nanopore for ONT)

## Outputs

- Strain-resolved assembly 'assembly.scaffolds.fa' in the Strainberry output directory (plus assembly.scaffolds.bam/.bai and .fa.fai)
- Intermediate per-multiplicity directories strainberry_n{k} containing variant VCFs (e.g., from mapping reads to contigs), haplotype blocks, and log files
- Per-strain reference-genome coverage metrics (e.g., fraction of L. helveticus NWC_2_3/NWC_2_4 covered; additional genomic material recovered vs. strain-oblivious assembly)

## How to apply

First generate a strain-oblivious metagenome assembly (e.g., metaFlye with 'flye --meta --pacbio-raw reads.fq.gz'), align long reads with minimap2 ('-ax map-pb' for PacBio or the ONT preset), sort with samtools, and index both the FASTA ('samtools faidx') and the coordinate-sorted BAM ('samtools index') — Strainberry expects both indexed. Then run 'strainberry -r assembly.fasta -b alignment.sorted.bam -o sberry_out -c 4' with defaults (max-strains 5, snv-density 0.1), adding '--nanopore' for Oxford Nanopore reads; Strainberry iterates strain multiplicities (strainberry_n2 … nK) and combines variant calling, haplotype phasing, and strain-aware scaffolding. Collect the final 'assembly.scaffolds.fa' from each run, align reads back with minimap2, and compute per-strain reference-genome coverage. Judge success by comparing against the strain-oblivious baseline: on the NWC2 PacBio dataset, Flye missed 23.03% (NWC_2_3) and 19.99% (NWC_2_4) of L. helveticus references, while Strainberry scaffolds reduced this to 9.87% and 0.64%, with 20% (PacBio) to 25–43% (Nanopore) additional genomic material on individual strain genomes and ~99.9% base accuracy on mock communities.

## Related tools

- **Strainberry** (Core pipeline: variant calling, haplotype phasing, and strain-aware scaffolding on the strain-oblivious assembly; outputs assembly.scaffolds.fa) — https://github.com/rvicedomini/strainberry
- **metaFlye** (Generates the strain-oblivious metagenome assembly input (e.g., 'flye --meta --pacbio-raw reads.fq.gz --out-dir flye_out --genome-size 300k --threads 12'))
- **minimap2** (Aligns long reads to the strain-oblivious assembly ('-ax map-pb' for PacBio) and to the output scaffolds for coverage evaluation)
- **samtools** (Sorts alignments, indexes the assembly FASTA ('samtools faidx') and the BAM ('samtools index') as required inputs to Strainberry)
- **Canu** (Alternative strain-oblivious long-read assembler used to produce assembly/BAM pairs for the NWC2 comparison)

## Examples

```
strainberry -r flye_out/assembly.fasta -b flye_out/alignment.sorted.bam -o sberry_out -c 4
```

## Evaluation signals

- Output directory contains 'assembly.scaffolds.fa' (with .fai) plus 'assembly.scaffolds.bam'/'.bai'; per-multiplicity subdirectories strainberry_n2 … nK exist after a successful run
- Scaffolds cover individual reference strain genomes substantially better than the strain-oblivious baseline — e.g., missing fraction of L. helveticus NWC_2_3 drops from 23.03% (Flye) to 9.87% (Strainberry, PacBio)
- Strainberry generates additional genomic material on individual strain genomes (20% PacBio, 25–43% Nanopore vs. conventional assemblies; up to 118% reported overall)
- Base-level accuracy of strain-resolved assemblies approaches the benchmarked ~99.9% on mock communities
- Read-depth separation: strain scaffolds show roughly halved coverage relative to the merged contigs for biallelic strains, with sequence identity exceeding 99.9% at 60–80x depth

## Limitations

- Haplotype phasing assumes read coverage evenly split among haplotypes, which fails in metagenomic settings where strains have different abundance/coverage levels — though separation of a limited number of well-represented strains still worked
- In high strain variability, strain-oblivious sequences might need separation into a variable number of strains within the same contig, which Strainberry does not handle
- Read length is limited (around N50 of 5 Kbp) by lack of easy DNA extraction methods producing high molecular weight DNA for all bacteria
- Ability to resolve plasmid genomes was not evaluated (left for future work); relationship between depth of coverage, read length, and haplotype resolution merits further study

## Evidence

- [intro] Strainberry combines a strain-oblivious assembler with the careful use of a long-read variant calling and haplotyping tool, followed by a component that performs long-read metagenome scaffolding.: "Strainberry combines a strain-oblivious assembler with the careful use of a long-read variant calling and haplotyping tool, followed by a component that performs long-read metagenome scaffolding."
- [readme] strainberry -r assembly.fasta -b alignment.sorted.bam -o sberry_out -c 4: "strainberry -r assembly.fasta -b alignment.sorted.bam -o sberry_out -c 4"
- [readme] Both `FASTA` and `BAM` files are expected to be indexed with the `samtools faidx` and `samtools index` commands, respectively.: "Both `FASTA` and `BAM` files are expected to be indexed with the `samtools faidx` and `samtools index` commands, respectively."
- [abstract] we benchmarked Strainberry on mock communities for which it produces strain-resolved assemblies with near-complete reference coverage and 99.9% base accuracy.: "We benchmarked Strainberry on mock communities for which it produces strain-resolved assemblies with near-complete reference coverage and 99.9% base accuracy."
- [abstract] Strainberry on real datasets for which it improved assemblies generating 20-118% additional genomic material than conventional metagenome assemblies on individual strain genomes.: "We also applied Strainberry on real datasets for which it improved assemblies generating 20-118% additional genomic material than conventional metagenome assemblies on individual strain genomes."
- [discussion] An average depth of coverage of 60–80× suffices to assemble individual strains of low-complexity metagenomes with almost complete coverage and sequence identity exceeding 99.9%.: "An average depth of coverage of 60–80× suffices to assemble individual strains of low-complexity metagenomes with almost complete coverage and sequence identity exceeding 99.9%."

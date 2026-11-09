---
name: strain-haplotype-separation
description: Use when long PacBio or Nanopore reads from a single low-complexity metagenomic
  sample must be separated into conspecific strain-level assemblies, rather than left
  collapsed in a strain-oblivious metagenome assembly.
when_to_use_negative:
- The metagenome is high-complexity — Strainberry is designed for single-sample low-complexity
  metagenomes
- Only short reads are available — the pipeline relies uniquely on long-read data
  and cannot reuse short-read strain methods
- The input is a species-level profile or marker-gene abundance table (e.g., StrainPhlAn/StrainEst
  output) rather than raw long reads plus an assembly
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: Strainberry
  role: Core pipeline performing strain separation via long-read variant calling,
    haplotype phasing, and strain-aware scaffolding
  repo: github.com/rvicedomini/strainberry
- name: metaFlye
  role: Generates the strain-oblivious metagenome assembly that Strainberry separates
    (flye --meta --pacbio-raw)
- name: minimap2
  role: Aligns long reads back to the assembly with the PacBio preset (-ax map-pb)
    to produce the input BAM
- name: samtools
  role: Sorts the alignment and indexes both the assembly FASTA (faidx) and BAM (index)
    as required inputs
- name: conda
  role: Installs Strainberry dependencies in an isolated 'sberry' environment (conda
    env create -n sberry --file environment.yml)
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-24515-9
    title: 'Strainberry: automated strain separation in low-complexity metagenomes
      using long reads'
  source_content_hash: sha256:c5265e699e29e189b818d8f3aa828bfebae548bafd82f79d4b0260e96f57ae28
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Strainberry
  - conda
  - metaFlye
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# strain-haplotype-separation

## Summary

Automated strain separation of low-complexity metagenomes from error-prone long reads: Strainberry takes a strain-oblivious metaFlye assembly plus a sorted, indexed long-read BAM and reconstructs individual strain haplotypes via variant calling, haplotype phasing, and strain-aware scaffolding.

## When to use

Use when long PacBio or Nanopore reads from a single low-complexity metagenomic sample must be separated into conspecific strain-level assemblies, rather than left collapsed in a strain-oblivious metagenome assembly.

## When NOT to use

- The metagenome is high-complexity — Strainberry is designed for single-sample low-complexity metagenomes
- Only short reads are available — the pipeline relies uniquely on long-read data and cannot reuse short-read strain methods
- The input is a species-level profile or marker-gene abundance table (e.g., StrainPhlAn/StrainEst output) rather than raw long reads plus an assembly

## Inputs

- PacBio or Oxford Nanopore long reads from a single low-complexity metagenomic sample (FASTQ/FASTQ.GZ)
- Strain-oblivious metagenome assembly (e.g., metaFlye assembly.fasta)
- Coordinate-sorted, indexed long-read alignment BAM (minimap2 -ax map-pb | samtools sort, indexed with samtools index)
- samtools faidx index of the assembly FASTA

## Outputs

- Strain-resolved scaffolds in OUTPUT_DIR/assembly.scaffolds.fa
- Read alignments on the strain-resolved assembly in OUTPUT_DIR/assembly.scaffolds.bam (with .bai index)
- Intermediate per-iteration results (VCFs, logs) in strainberry_n{k} sub-directories
- Per-strain evaluation table of reference coverage, ANI, and duplication ratio

## How to apply

First assemble the long reads with metaFlye in metagenome mode (flye --meta --pacbio-raw reads.fq.gz --out-dir flye_out --genome-size 300k --threads 12; --genome-size not required from metaFlye 2.8). Align the reads back to the assembly with minimap2 using the PacBio preset (-ax map-pb -t 12), sort with samtools sort, and index both the FASTA (samtools faidx) and the coordinate-sorted BAM (samtools index). Then run Strainberry (strainberry -r assembly.fasta -b alignment.sorted.bam -o sberry_out -c 4) inside the 'sberry' conda environment; it defaults to a maximum strain multiplicity of 5 (-n) and a minimum SNV density of 0.1% (-s), and internally iterates over increasing strain counts (strainberry_n2 ... strainberry_nK). Collect the final strain-resolved contigs from OUTPUT_DIR/assembly.scaffolds.fa and the read alignments on them from assembly.scaffolds.bam, then evaluate per-strain reference coverage, ANI, and duplication ratio against known references. An average depth of 60–80x suffices to reach near-complete coverage and >99.9% sequence identity.

## Related tools

- **Strainberry** (Core pipeline performing strain separation via long-read variant calling, haplotype phasing, and strain-aware scaffolding) — github.com/rvicedomini/strainberry
- **metaFlye** (Generates the strain-oblivious metagenome assembly that Strainberry separates (flye --meta --pacbio-raw))
- **minimap2** (Aligns long reads back to the assembly with the PacBio preset (-ax map-pb) to produce the input BAM)
- **samtools** (Sorts the alignment and indexes both the assembly FASTA (faidx) and BAM (index) as required inputs)
- **conda** (Installs Strainberry dependencies in an isolated 'sberry' environment (conda env create -n sberry --file environment.yml))

## Examples

```
strainberry -r assembly.fasta -b alignment.sorted.bam -o sberry_out -c 4
```

## Evaluation signals

- For the two-strain E. coli K12/W example, assembly.scaffolds.fa contains exactly two scaffolds — one closer to K12 and one closer to W
- Per-strain ANI reaches ~99.9% (99.97% for both K-12 and W with ssFlye on Mock3) with near-complete reference coverage
- Duplication ratios are close to 1 for the metaFlye-based run (Canu baseline expected at 1.23/1.39)
- Strain separation succeeds for up to five conspecific E. coli strains, with degradation beyond that (coverage dropping from 95% at 3 strains to 75% at 5 strains)
- In the benchmarked mock communities at 60–80x average depth, assemblies show almost complete coverage with sequence identity exceeding 99.9%

## Limitations

- Haplotype phasing assumes read coverage is evenly split among haplotypes, which does not hold in metagenomes where strains differ in abundance
- Coverage separation was demonstrated only for a limited number of well-represented strains
- A single contig needing separation into a variable number of strains (high strain variability) is not handled
- Ability to resolve plasmid genomes was not evaluated
- DNA extraction limits read length to around N50 of 5 Kbp, and the relationship between depth, read length, and haplotype resolution merits further study

## Evidence

- [intro] Strainberry combines a strain-oblivious assembler with the careful use of a long-read variant calling and haplotyping tool, followed by a component that performs long-read metagenome scaffolding.: "Strainberry combines a strain-oblivious assembler with the careful use of a long-read variant calling and haplotyping tool, followed by a component that performs long-read metagenome scaffolding."
- [methods] strainberry -r assembly.fasta -b alignment.sorted.bam -o sberry_out -c 4: "strainberry -r assembly.fasta -b alignment.sorted.bam -o sberry_out -c 4"
- [discussion] An average depth of coverage of 60–80× suffices to assemble individual strains of low-complexity metagenomes with almost complete coverage and sequence identity exceeding 99.9%.: "An average depth of coverage of 60–80× suffices to assemble individual strains of low-complexity metagenomes with almost complete coverage and sequence identity exceeding 99.9%."
- [readme] After a successful run of Strainberry, in the `sberry_out` directory, the file `assembly.scaffolds.fa` should contain two scaffolds (one closer to strain K12, the other closer to strain W).: "After a successful run of Strainberry, in the `sberry_out` directory, the file `assembly.scaffolds.fa` should contain two scaffolds (one closer to strain K12, the other closer to strain W)."
- [discussion] Haplotype phasing in single genomes benefits from read coverage being evenly split among the different haplotypes. This assumption no longer holds in a metagenomic setting: "Haplotype phasing in single genomes benefits from read coverage being evenly split among the different haplotypes. This assumption no longer holds in a metagenomic setting"

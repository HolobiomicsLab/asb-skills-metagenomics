---
name: coverage-metrics-tabulation-across-assemblers
description: Use when comparing strain-level genome coverage recovered by a strain-resolved
  assembly against a strain-oblivious assembly, to compute missing percentages and
  additional genomic material per reference strain.
when_to_use_negative:
- 'The metagenome is high-complexity rather than low-complexity: Strainberry strain
  separation, and hence the coverage gains being tabulated, is designed for single-sample
  low-complexity metagenomes'
- No strain-level reference genomes are available, so per-strain coverage fractions
  cannot be computed and only de novo assembly completeness could be assessed
- The input is already a processed coverage/evaluation table rather than assemblies
  and read alignments
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: Strainberry
  role: Produces the strain-resolved assemblies whose per-strain coverage is tabulated
    against the strain-oblivious baseline
  repo: https://github.com/rvicedomini/strainberry
- name: metaFlye
  role: Generates the strain-oblivious metagenome assembly used as the baseline coverage
    comparison
- name: minimap2
  role: Aligns long reads back to each output assembly (map-pb preset for PacBio,
    ONT preset for Nanopore) to compute coverage fractions
- name: samtools
  role: Sorts, indexes (samtools index) and faidx-indexes (samtools faidx) alignments
    and assembly FASTAs required before coverage computation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-24515-9
    title: 'Strainberry: automated strain separation in low-complexity metagenomes
      using long reads'
  source_content_hash: sha256:b3a99d93dd5bfdf98f88b73074572a1fa9f5bf8c2badbedafa5480a07bc8aeb3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rvicedomini/strainberry
  tools:
  - Strainberry
  - metaFlye
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# coverage-metrics-tabulation-across-assemblers

## Summary

Quantify how much of each strain reference genome is covered by strain-oblivious assemblies (Flye, Canu) versus Strainberry strain-resolved scaffolds, across PacBio and Nanopore datasets, and tabulate the additional genomic material recovered (e.g. nwc2_coverage_metrics.csv mirroring Fig. 5 / Supplementary Data 2).

## When to use

Use when comparing strain-level genome coverage recovered by a strain-resolved assembly against a strain-oblivious assembly, to compute missing percentages and additional genomic material per reference strain.

## When NOT to use

- The metagenome is high-complexity rather than low-complexity: Strainberry strain separation, and hence the coverage gains being tabulated, is designed for single-sample low-complexity metagenomes
- No strain-level reference genomes are available, so per-strain coverage fractions cannot be computed and only de novo assembly completeness could be assessed
- The input is already a processed coverage/evaluation table rather than assemblies and read alignments

## Inputs

- Strain-oblivious metagenome assembly FASTA (e.g. metaFlye assembly.fasta, Canu assembly)
- Coordinate-sorted long-read alignment BAM indexed with samtools index
- Long-read FASTQ datasets (PacBio and Oxford Nanopore) for realignment
- Reference genomes of the individual strains (L. helveticus NWC_2_3 and NWC_2_4, L. delbrueckii strains)
- Strainberry strain-resolved output assembly.scaffolds.fa per assembler/technology combination

## Outputs

- Per-strain reference genome coverage fractions for each assembly (strain-oblivious and strain-resolved)
- Percentage of each reference genome missed by each assembly (e.g. 23.03% vs 9.87% for NWC_2_3 on PacBio)
- Additional genomic material recovered relative to the strain-oblivious assembly (20% PacBio, 25-43% Nanopore)
- Coverage metrics table (nwc2_coverage_metrics.csv) mirroring Fig. 5 / Supplementary Data 2

## How to apply

Index each strain-oblivious assembly FASTA with 'samtools faidx' and its coordinate-sorted read alignment with 'samtools index', then run Strainberry on each assembly/BAM pair ('strainberry -r assembly.fasta -b alignment.sorted.bam -o sberry_out -c 4', defaults max-strains 5 and snv-density 0.1, adding '--nanopore' for ONT alignments). Collect the strain-resolved 'assembly.scaffolds.fa' from each run (Flye-PacBio, Flye-Nanopore, Canu-PacBio, Canu-Nanopore), realign the long reads back to each output with minimap2 ('-ax map-pb' for PacBio or the appropriate ONT preset), and sort/index with samtools. Compute the fraction of each L. helveticus (and L. delbrueckii) strain reference genome covered by the output scaffolds and the additional genomic material recovered relative to the strain-oblivious assembly. Compile these coverage metrics into a CSV mirroring Fig. 5 / Supplementary Data 2 and validate against reported values (Flye missing 23.03% of NWC_2_3 vs Strainberry/ssFlye missing 9.87% on PacBio; 20% PacBio and 25-43% Nanopore additional material).

## Related tools

- **Strainberry** (Produces the strain-resolved assemblies whose per-strain coverage is tabulated against the strain-oblivious baseline) — https://github.com/rvicedomini/strainberry
- **metaFlye** (Generates the strain-oblivious metagenome assembly used as the baseline coverage comparison)
- **minimap2** (Aligns long reads back to each output assembly (map-pb preset for PacBio, ONT preset for Nanopore) to compute coverage fractions)
- **samtools** (Sorts, indexes (samtools index) and faidx-indexes (samtools faidx) alignments and assembly FASTAs required before coverage computation)

## Examples

```
minimap2 -ax map-pb -t 12 ./flye_out/assembly.fasta reads.fq.gz | samtools sort >./flye_out/alignment.sorted.bam; samtools faidx ./flye_out/assembly.fasta; samtools index ./flye_out/alignment.sorted.bam
```

## Evaluation signals

- Flye-PacBio baseline shows 23.03% of L. helveticus NWC_2_3 and 19.99% of NWC_2_4 reference genomes missing; Strainberry (ssFlye) reduces this to 9.87% and 0.64%
- Strainberry generates 20% additional genomic material on individual strain genomes with PacBio and 25-43% with Nanopore relative to conventional assemblies
- Each assembly FASTA and BAM is successfully indexed with samtools faidx and samtools index before Strainberry runs
- Strainberry output directory contains assembly.scaffolds.fa (plus .fai, .bam, .bam.bai) and strainberry_n{k} iteration sub-directories
- Compiled CSV metrics match the values reported in Fig. 5 / Supplementary Data 2 of the paper

## Limitations

- No explicit version of Strainberry is stated in the article (the analyses repository pins version 1.1)
- Haplotype phasing assumes evenly split read coverage, which does not hold in metagenomic settings where strains have different abundance/coverage levels
- In presence of high strain variability, strain-oblivious sequences might need separation into a variable number of strains within the same contig, which is not handled
- Limited read length (N50 around 5 Kbp) due to DNA extraction constraints may affect strain separation and thus coverage gains

## Evidence

- [abstract] On real datasets, Strainberry generated 20-118% additional genomic material than conventional metagenome assemblies on individual strain genomes.: "We also applied Strainberry on real datasets for which it improved assemblies generating 20-118% additional genomic material than conventional metagenome assemblies on individual strain genomes."
- [readme] Indexing requirement before Strainberry runs.: "Both `FASTA` and `BAM` files are expected to be indexed with the `samtools faidx` and `samtools index` commands, respectively."
- [readme] Strainberry run command with CPUs parameter.: "$ strainberry -r assembly.fasta -b alignment.sorted.bam -o sberry_out -c 4"
- [readme] Output assembly file used for coverage computation.: [evidence span withheld — anchor status: not_found]
- [discussion] Coverage-depth requirement for near-complete strain coverage.: "An average depth of coverage of 60–80× suffices to assemble individual strains of low-complexity metagenomes with almost complete coverage and sequence identity exceeding 99.9%."
- [discussion] Uneven strain coverage caveat in metagenomic phasing.: "Haplotype phasing in single genomes benefits from read coverage being evenly split among the different haplotypes. This assumption no longer holds in a metagenomic setting where haplotypes/strains"

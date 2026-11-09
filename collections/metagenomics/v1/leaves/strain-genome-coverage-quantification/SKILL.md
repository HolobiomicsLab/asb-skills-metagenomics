---
name: strain-genome-coverage-quantification
description: Use when you have a strain-oblivious long-read metagenome assembly, a
  strain-resolved assembly (e.g., Strainberry scaffolds), and closed strain reference
  genomes, and you need per-strain coverage fractions or recovered additional genomic
  material.
when_to_use_negative:
- No closed strain reference genomes are available to compute per-strain coverage
  fractions against
- 'The metagenome is high-complexity rather than low-complexity: Strainberry separation
  is demonstrated for a limited number of well-represented strains and haplotype phasing
  assumes strains can be separated by SNV density'
- The goal is de novo assembly itself rather than benchmarking coverage recovery of
  an existing strain-resolved assembly
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: Strainberry
  role: Produces the strain-resolved scaffolds (assembly.scaffolds.fa) whose coverage
    is quantified against each strain reference
  repo: https://github.com/rvicedomini/strainberry
- name: metaFlye
  role: Generates the strain-oblivious baseline metagenome assembly (flye --meta)
    used as the comparison point
- name: minimap2
  role: Aligns long reads back to each output assembly ('-ax map-pb' for PacBio or
    the ONT preset) to support coverage computation
- name: samtools
  role: Indexes assemblies (samtools faidx) and alignments (samtools index) and sorts
    minimap2 output BAMs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-24515-9
    title: 'Strainberry: automated strain separation in low-complexity metagenomes
      using long reads'
  source_content_hash: sha256:fe5546870dc854a4c189b80cffb8a7a286c85ec755322979b22cf6807206491f
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
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# strain-genome-coverage-quantification

## Summary

Quantifies how much of each strain reference genome is covered by strain-oblivious versus strain-resolved long-read assemblies (e.g., Flye vs. Strainberry scaffolds) by mapping long reads back to the assemblies and computing per-strain reference coverage fractions, mirroring Fig. 5 / Supplementary Data 2 of the Strainberry paper.

## When to use

Use when you have a strain-oblivious long-read metagenome assembly, a strain-resolved assembly (e.g., Strainberry scaffolds), and closed strain reference genomes, and you need per-strain coverage fractions or recovered additional genomic material.

## When NOT to use

- No closed strain reference genomes are available to compute per-strain coverage fractions against
- The metagenome is high-complexity rather than low-complexity: Strainberry separation is demonstrated for a limited number of well-represented strains and haplotype phasing assumes strains can be separated by SNV density
- The goal is de novo assembly itself rather than benchmarking coverage recovery of an existing strain-resolved assembly

## Inputs

- Strain-oblivious long-read metagenome assembly FASTA (e.g., metaFlye assembly.fasta)
- Coordinate-sorted long-read alignment BAM (indexed with samtools index)
- Strainberry strain-resolved output assembly 'assembly.scaffolds.fa'
- Long reads in FASTQ (PacBio or Oxford Nanopore)
- Closed/circular reference genomes for each strain (e.g., L. helveticus NWC_2_3 and NWC_2_4)

## Outputs

- Per-strain reference genome coverage fractions for strain-oblivious and strain-resolved assemblies (e.g., nwc2_coverage_metrics.csv)
- Additional genomic material recovered by the strain-resolved assembly relative to the conventional assembly
- Sorted/indexed BAM alignments of long reads against each output assembly

## How to apply

Index each strain-oblivious assembly (Flye/Canu) with 'samtools faidx' and its coordinate-sorted read alignment with 'samtools index', then run Strainberry ('strainberry -r assembly.fasta -b alignment.sorted.bam -o sberry_out -c 4') with default parameters (max-strains 5, snv-density 0.1), adding '--nanopore' for ONT alignments. Collect the strain-resolved 'assembly.scaffolds.fa' from each run (Flye-PacBio, Flye-Nanopore, Canu-PacBio, Canu-Nanopore). Align the long reads back to each output assembly with minimap2 ('-ax map-pb' for PacBio or the ONT preset) and sort/index with samtools. Compute the fraction of each reference strain genome (e.g., L. helveticus NWC_2_3 and NWC_2_4) covered by the scaffolds and the additional genomic material recovered relative to the strain-oblivious assembly, compiling metrics into a CSV such as nwc2_coverage_metrics.csv. The rationale is that a strain-oblivious assembler collapses conspecific haplotypes into a consensus, so per-reference read-mapping coverage reveals strain content lost to the consensus.

## Related tools

- **Strainberry** (Produces the strain-resolved scaffolds (assembly.scaffolds.fa) whose coverage is quantified against each strain reference) — https://github.com/rvicedomini/strainberry
- **metaFlye** (Generates the strain-oblivious baseline metagenome assembly (flye --meta) used as the comparison point)
- **minimap2** (Aligns long reads back to each output assembly ('-ax map-pb' for PacBio or the ONT preset) to support coverage computation)
- **samtools** (Indexes assemblies (samtools faidx) and alignments (samtools index) and sorts minimap2 output BAMs)

## Examples

```
strainberry -r assembly.fasta -b alignment.sorted.bam -o sberry_out -c 4 --nanopore
```

## Evaluation signals

- On the NWC2 PacBio dataset, conventional Flye should miss ~23.03% of L. helveticus NWC_2_3 and ~19.99% of NWC_2_4, while Strainberry (ssFlye) should miss only ~9.87% and ~0.64% respectively
- Strainberry should generate ~20% additional genomic material on individual strain genomes for PacBio and ~25-43% for Nanopore data (article-wide range 20-118%)
- The strain-resolved output 'assembly.scaffolds.fa' must exist after a successful Strainberry run, with indexed .fai/.bai companions
- Read-mapping BAMs against each output assembly must be coordinate-sorted and indexed (samtools sort/index) before coverage computation
- Compiled coverage metrics should match the reported values in Fig. 5 / Supplementary Data 2 when reproduced with default Strainberry parameters (max-strains 5, snv-density 0.1)

## Limitations

- An average depth of coverage of 60-80x is needed to assemble individual strains with almost complete coverage and >99.9% sequence identity; lower coverage will reduce recovered fraction
- Haplotype phasing assumes read coverage is evenly split among haplotypes, which does not hold for unevenly abundant strains, though separation of a limited number of well-represented strains still works
- In presence of high strain variability, strain-oblivious sequences might need separation into a variable number of strains within the same contig, which Strainberry does not handle
- Short read lengths (around N50 of 5 Kbp, limited by DNA extraction) constrain strain separation; plasmid genome resolution was not evaluated

## Evidence

- [results] On the NWC2 PacBio dataset, Strainberry scaffolds cover almost fully both L. helveticus reference strains, leaving only 9.87% of strain NWC_2_3 and 0.64% of strain NWC_2_4 missing, versus 23.03% and 19.99% for the Flye assembly.: [evidence span withheld — anchor status: not_found]
- [readme] Both FASTA and BAM files are expected to be indexed with the samtools faidx and samtools index commands, respectively.: "Both `FASTA` and `BAM` files are expected to be indexed with the `samtools faidx` and `samtools index` commands, respectively."
- [readme] The strain-resolved output assembly is stored as assembly.scaffolds.fa in the output directory.: "Strainberry output assembly is stored in the `assembly.scaffolds.fa` file."
- [abstract] We also applied Strainberry on real datasets for which it improved assemblies generating 20-118% additional genomic material than conventional metagenome assemblies on individual strain genomes.: "it improved assemblies generating 20-118% additional genomic material than conventional metagenome assemblies on individual strain genomes"
- [discussion] An average depth of coverage of 60-80x suffices to assemble individual strains of low-complexity metagenomes with almost complete coverage and sequence identity exceeding 99.9%.: "An average depth of coverage of 60–80× suffices to assemble individual strains of low-complexity metagenomes with almost complete coverage and sequence identity exceeding 99.9%."

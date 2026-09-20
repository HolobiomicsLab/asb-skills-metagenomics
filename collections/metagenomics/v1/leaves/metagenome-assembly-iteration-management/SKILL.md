---
name: metagenome-assembly-iteration-management
description: Use when running Strainberry on a strain-oblivious long-read assembly
  (FASTA plus sorted, indexed BAM) and you need to control iteration count via the
  Hamming rate improvement stopping rule and --max-strains multiplicity cap.
when_to_use_negative:
- When the metagenome is high-complexity rather than single-sample low-complexity
  — Strainberry relies uniquely on long-read data and is designed for low-complexity
  metagenomes
- When strain-oblivious contigs would need separation into a variable number of strains
  within the same contig (high strain variability) — not handled by the method
- When input reads are short reads — existing strain-level assembly approaches based
  on short reads cannot be reused for long reads, and Strainberry requires long-read
  alignments
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0196
tools:
- name: Strainberry
  role: Pipeline that performs the iterative haplotype phasing, per-group assembly,
    and strain-aware scaffolding; exposes --max-strains and --snv-density parameters
    that govern the iteration loop
  repo: https://github.com/rvicedomini/strainberry
- name: minimap2
  role: Aligns long reads to the current scaffolds for re-phasing at each iteration
    and produces the final assembly.scaffolds.bam read alignment
- name: samtools
  role: Sorts, indexes (samtools index), and faidx-indexes the BAM/FASTA inputs required
    before running the iterative loop
- name: metaFlye
  role: Generates the strain-oblivious assembly (flye --meta) that seeds the first
    iteration
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-24515-9
    title: 'Strainberry: automated strain separation in low-complexity metagenomes
      using long reads'
  source_content_hash: sha256:96e75f8e25b756a867ea237bb3c0cb271ebd29c220eabff74d08c292a0be3589
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
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# metagenome-assembly-iteration-management

## Summary

Manages Strainberry's iterative strain-separation loop, deciding when to trigger an additional phasing/assembly/scaffolding round based on average Hamming rate improvement and when to stop (criterion failure or maximum strain multiplicity).

## When to use

Use when running Strainberry on a strain-oblivious long-read assembly (FASTA plus sorted, indexed BAM) and you need to control iteration count via the Hamming rate improvement stopping rule and --max-strains multiplicity cap.

## When NOT to use

- When the metagenome is high-complexity rather than single-sample low-complexity — Strainberry relies uniquely on long-read data and is designed for low-complexity metagenomes
- When strain-oblivious contigs would need separation into a variable number of strains within the same contig (high strain variability) — not handled by the method
- When input reads are short reads — existing strain-level assembly approaches based on short reads cannot be reused for long reads, and Strainberry requires long-read alignments

## Inputs

- Strain-oblivious metagenome assembly FASTA (e.g., metaFlye assembly.fasta, indexed with samtools faidx)
- Coordinate-sorted long-read alignment BAM (indexed with samtools index)
- Maximum strain multiplicity n via --max-strains (default 5)
- Minimum SNV percentage for haplotype blocks via --snv-density (default 0.1)

## Outputs

- Final strain-separated scaffolds in assembly.scaffolds.fa
- minimap2-based read alignment of the output assembly (assembly.scaffolds.bam with .bai index)
- Per-iteration intermediate directories strainberry_n2/, strainberry_n3/, ..., strainberry_nK/ (VCFs of variants, log files)

## How to apply

Start from a strain-oblivious assembly (e.g., metaFlye FASTA) and its coordinate-sorted, samtools-indexed BAM alignment. For each iteration k=2..K, re-map and re-phase long reads against the current scaffolds using minimap2-based alignment and long-read variant calling/haplotyping, with haplotype blocks defined by the --snv-density threshold of 0.1 (minimum SNV percentage). For each read overlapping a phased region by at least 3 Kbp, compute the Hamming rate to the closest haplotype; trigger an additional strain separation only when the global average Hamming rate improves by at least 1%, applying local per-sequence improvement checks. Store per-iteration intermediates in strainberry_n2/, strainberry_n3/, ... subdirectories, cap iterations with --max-strains (default 5), and stop when the improvement criterion fails or the maximum multiplicity is reached. On Mock3 ssFlye input with n=2 strains, the loop terminates after n-1 = 1 separation iteration.

## Related tools

- **Strainberry** (Pipeline that performs the iterative haplotype phasing, per-group assembly, and strain-aware scaffolding; exposes --max-strains and --snv-density parameters that govern the iteration loop) — https://github.com/rvicedomini/strainberry
- **minimap2** (Aligns long reads to the current scaffolds for re-phasing at each iteration and produces the final assembly.scaffolds.bam read alignment)
- **samtools** (Sorts, indexes (samtools index), and faidx-indexes the BAM/FASTA inputs required before running the iterative loop)
- **metaFlye** (Generates the strain-oblivious assembly (flye --meta) that seeds the first iteration)

## Examples

```
strainberry -r assembly.fasta -b alignment.sorted.bam -o sberry_out -c 4
```

## Evaluation signals

- The output directory contains per-iteration subdirectories strainberry_n2/ ... strainberry_nK plus assembly.scaffolds.fa and assembly.scaffolds.bam/.bai
- On Mock3 ssFlye input with n=2 strains, the loop terminates after n-1 = 1 separation iteration
- No additional iteration is launched when the global average Hamming rate improves by less than 1% (stopping criterion respected)
- Number of iteration directories never exceeds the --max-strains cap (default 5)
- On the E. coli K12/W example, assembly.scaffolds.fa contains two scaffolds, one closer to strain K12 and the other closer to strain W

## Limitations

- Haplotype phasing assumes evenly split read coverage among haplotypes; this does not hold in metagenomes where strains have different abundance/coverage levels
- In presence of high strain variability, strain-oblivious sequences might need separation into a variable number of strains within the same contig, which is not handled
- Relationship between depth of coverage, read length, and haplotype resolution merits further study; short read lengths (N50 around 5 Kbp due to DNA extraction limits) constrain separation

## Evidence

- [readme] README command-line options for iteration control: "-n int, --max-strains int
                        Attempt strain-separation at most for the provided
                        strain multiplicity [5]"
- [readme] README SNV density threshold: "-s float, --snv-density float
                        Minimum SNV percentage to consider haplotype blocks
                        [0.1]"
- [readme] README per-iteration output directories: "All sub-directories named `strainberry_n{k}` contain intermediate results of Strainberry iterations (e.g. the VCF of variants found by mapping reads to contigs) and log files."
- [intro] Article workflow description: "Strainberry combines a strain-oblivious assembler with the careful use of a long-read variant calling and haplotyping tool, followed by a component that performs long-read metagenome scaffolding."
- [discussion] Article coverage assumption caveat: "Haplotype phasing in single genomes benefits from read coverage being evenly split among the different haplotypes. This assumption no longer holds in a metagenomic setting where haplotypes/strains"

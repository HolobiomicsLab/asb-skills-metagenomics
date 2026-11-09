---
name: long-read-haplotype-phasing
description: Use when you have a strain-oblivious long-read metagenome assembly (FASTA)
  plus a coordinate-sorted indexed BAM of long reads and need to separate reads/scaffolds
  by strain using haplotype phasing rather than short reads or references.
when_to_use_negative:
- The community is high-complexity — Strainberry is designed for single-sample low-complexity
  metagenomes, and phasing assumes a limited number of well-represented strains
- Only short reads are available — all Strainberry separation relies uniquely on long-read
  data and short-read strain methods cannot be re-used
- A reference-genome-based strain typing is sufficient — tools like StrainEst or StrainPhlAn
  serve that goal; this skill is de novo, reference-free separation
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0194
tools:
- name: Strainberry
  role: 'Runs the iterative phasing/separation: long-read variant calling and haplotyping
    over the strain-oblivious assembly, then strain-aware scaffolding of per-strain
    groups'
  repo: https://github.com/rvicedomini/strainberry
- name: minimap2
  role: Aligns long reads (e.g. -ax map-pb for PacBio) to the assembly to produce
    the BAM input and the final assembly.scaffolds.bam read alignment
- name: samtools
  role: Sorts minimap2 output into a coordinate-sorted BAM and creates the required
    faidx/BAM indexes
- name: metaFlye (flye --meta)
  role: Generates the strain-oblivious metagenome assembly that phasing is performed
    against
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-24515-9
    title: 'Strainberry: automated strain separation in low-complexity metagenomes
      using long reads'
  source_content_hash: sha256:63e12427d7440d37f807b9195f41570ea68884db7e1fae157e03a98121364c98
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rvicedomini/strainberry
  tools:
  - Strainberry
  - metaFlye (flye --meta)
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0194
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# long-read-haplotype-phasing

## Summary

Phases strain haplotypes from error-prone long reads by mapping them to a strain-oblivious metagenome assembly, calling variants, and using haplotyping to partition reads into strain groups — the core separation engine of Strainberry for low-complexity metagenomes.

## When to use

Use when you have a strain-oblivious long-read metagenome assembly (FASTA) plus a coordinate-sorted indexed BAM of long reads and need to separate reads/scaffolds by strain using haplotype phasing rather than short reads or references.

## When NOT to use

- The community is high-complexity — Strainberry is designed for single-sample low-complexity metagenomes, and phasing assumes a limited number of well-represented strains
- Only short reads are available — all Strainberry separation relies uniquely on long-read data and short-read strain methods cannot be re-used
- A reference-genome-based strain typing is sufficient — tools like StrainEst or StrainPhlAn serve that goal; this skill is de novo, reference-free separation

## Inputs

- Strain-oblivious long-read metagenome assembly in FASTA format (e.g. metaFlye assembly.fasta)
- Coordinate-sorted long-read alignment in BAM format (minimap2 output sorted with samtools)
- Samtools faidx index of the FASTA and samtools index of the BAM

## Outputs

- assembly.scaffolds.fa — final strain-separated scaffolds
- assembly.scaffolds.bam (+ .bai) — minimap2-based alignment of input reads on the output assembly
- strainberry_n2/, strainberry_n3/, ... per-iteration intermediate directories (VCFs of variants, phasing results, logs)

## How to apply

Build inputs with metaFlye (--meta mode) and minimap2, then index the assembly with samtools faidx and the BAM with samtools index before running Strainberry. Strainberry combines the strain-oblivious assembler with long-read variant calling and haplotyping, phasing reads over haplotype blocks defined by the --snv-density threshold (default 0.1 SNV percentage). Only reads overlapping a phased region by at least 3 Kbp are assigned to a haplotype via their Hamming rate to the closest haplotype. Iteratively re-map and re-phase against the latest scaffolds, triggering a further separation (up to --max-strains n=5) only when the global average Hamming rate improves by at least 1%; stop when the improvement criterion fails or the maximum multiplicity is reached.

## Related tools

- **Strainberry** (Runs the iterative phasing/separation: long-read variant calling and haplotyping over the strain-oblivious assembly, then strain-aware scaffolding of per-strain groups) — https://github.com/rvicedomini/strainberry
- **minimap2** (Aligns long reads (e.g. -ax map-pb for PacBio) to the assembly to produce the BAM input and the final assembly.scaffolds.bam read alignment)
- **samtools** (Sorts minimap2 output into a coordinate-sorted BAM and creates the required faidx/BAM indexes)
- **metaFlye (flye --meta)** (Generates the strain-oblivious metagenome assembly that phasing is performed against)

## Examples

```
strainberry -r assembly.fasta -b alignment.sorted.bam -o sberry_out -c 4
```

## Evaluation signals

- Output directory contains the per-iteration subdirectories strainberry_n2/, strainberry_n3/, ... plus assembly.scaffolds.fa, assembly.scaffolds.bam, .bai and .fai files
- On the Mock3 ssFlye input with n=2 strains the loop terminates after n-1 = 1 separation iteration
- In the README example, assembly.scaffolds.fa contains two scaffolds, one closer to E. coli strain K12 and one closer to strain W
- Separation iterations continue only while the global average Hamming rate improves by at least 1%, with local per-sequence improvement checks
- Neither input FASTA nor BAM lacks indexes — run fails or misbehaves unless samtools faidx and samtools index were applied beforehand

## Limitations

- Haplotype phasing assumes read coverage is evenly split among haplotypes, which does not hold in metagenomic settings where strains have different abundances; demonstrated only for a limited number of well-represented strains
- In presence of high strain variability, strain-oblivious sequences might need separation into a variable number of strains within the same contig, which is not handled
- Ability to resolve plasmid genomes was not evaluated
- Long-read length is limited by DNA extraction (~N50 of 5 Kbp), and the relationship between coverage, read length and haplotype resolution merits further study

## Evidence

- [intro] Strainberry combines a strain-oblivious assembler with the careful use of a long-read variant calling and haplotyping tool, followed by a component that performs long-read metagenome scaffolding.: "Strainberry combines a strain-oblivious assembler with the careful use of a long-read variant calling and haplotyping tool, followed by a component that performs long-read metagenome scaffolding."
- [discussion] Strainberry finds strain variations and separates reads by exploiting state-of-the-art haplotype phasing techniques that were designed for genomes: "Strainberry finds strain variations and separates reads by exploiting state-of-the-art haplotype phasing techniques that were designed for genomes"
- [readme] -s float, --snv-density float Minimum SNV percentage to consider haplotype blocks [0.1]: "-s float, --snv-density float
                        Minimum SNV percentage to consider haplotype blocks
                        [0.1]"
- [readme] Both `FASTA` and `BAM` files are expected to be indexed with the `samtools faidx` and `samtools index` commands, respectively.: "Both `FASTA` and `BAM` files are expected to be indexed with the `samtools faidx` and `samtools index` commands, respectively."

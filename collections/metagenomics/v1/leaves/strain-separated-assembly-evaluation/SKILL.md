---
name: strain-separated-assembly-evaluation
description: Reference for judging whether Strainberry correctly separated two or
  more conspecific strains from a long-read metagenome assembly, using ANI, reference
  coverage, and duplication-ratio metrics per recovered strain.
when_to_use_negative:
- The sample is a high-complexity microbiome rather than a low-complexity metagenome
  — Strainberry and this evaluation target low-complexity communities with well-represented
  strains
- Only short-read data are available — the workflow relies uniquely on error-prone
  long reads and strain-oblivious long-read assemblers
- You only need a raw strain-oblivious assembly without strain separation or per-strain
  quality assessment
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: Strainberry
  role: Performs the strain separation whose outputs (assembly.scaffolds.fa, assembly.scaffolds.bam)
    are evaluated; version 1.1 used in the analysis workflows
  repo: https://github.com/rvicedomini/strainberry
- name: metaFlye
  role: Generates the strain-oblivious metagenome assembly (flye --meta --pacbio-raw)
    that serves as Strainberry input and ssFlye baseline
- name: minimap2
  role: Aligns long reads (preset map-pb for PacBio) to assemblies for the required
    BAM input and the output-assembly alignments
- name: samtools
  role: Sorts alignments and produces the samtools faidx / samtools index files Strainberry
    requires
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-24515-9
    title: 'Strainberry: automated strain separation in low-complexity metagenomes
      using long reads'
  source_content_hash: sha256:85a76ed53bea906de3947a92ca26d0ab8fb5d395d4fff8d2bdade1ff85f1870c
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
yes_rate: 0.6428571428571429
n_claims: 14
---

# strain-separated-assembly-evaluation

## Summary

Evaluate Strainberry strain-separated metagenome assemblies against reference genomes by computing per-strain average nucleotide identity (ANI), reference coverage, and duplication ratio, mirroring the Fig. 2 / Supplementary Data 1 tables of the Mock3 and Mock9 benchmarks.

## When to use

Reference for judging whether Strainberry correctly separated two or more conspecific strains from a long-read metagenome assembly, using ANI, reference coverage, and duplication-ratio metrics per recovered strain.

## When NOT to use

- The sample is a high-complexity microbiome rather than a low-complexity metagenome — Strainberry and this evaluation target low-complexity communities with well-represented strains
- Only short-read data are available — the workflow relies uniquely on error-prone long reads and strain-oblivious long-read assemblers
- You only need a raw strain-oblivious assembly without strain separation or per-strain quality assessment

## Inputs

- PacBio (or Oxford Nanopore) long-read FASTQ of a low-complexity mock community (e.g. Mock3 E. coli K-12 + W)
- Strain-oblivious metagenome assembly FASTA (e.g. metaFlye assembly.fasta or Canu baseline)
- Coordinate-sorted, indexed long-read alignment BAM (minimap2 map-pb, samtools sort/index)
- Reference genomes of the constituent strains (e.g. E. coli K-12 and W) for ANI/coverage computation

## Outputs

- Strainberry strain-separated scaffolds (assembly.scaffolds.fa) per assembler baseline
- Minimap2-based read alignments on the output assembly (assembly.scaffolds.bam)
- Per-strain evaluation table with ANI, reference coverage, and duplication ratio for each recovered strain and each run (mirroring Fig. 2 / Supplementary Data 1)

## How to apply

First generate a strain-oblivious assembly from PacBio (or Oxford Nanopore, using Strainberry's --nanopore flag) reads with metaFlye ('flye --meta --pacbio-raw reads.fq.gz --out-dir flye_out --threads 12'; --genome-size is unnecessary for metaFlye >= 2.8). Align the raw long reads back to the assembly with minimap2 ('minimap2 -ax map-pb -t 12 assembly.fasta reads.fq.gz'), sort with samtools sort, and index the FASTA and BAM with 'samtools faidx' and 'samtools index' as Strainberry requires. Run Strainberry on the metaFlye assembly and on a baseline assembly of the same reads (e.g. Canu), collecting assembly.scaffolds.fa and assembly.scaffolds.bam from each output directory. Compute per-strain ANI against reference genomes, reference coverage, and duplication ratio for each recovered scaffold, compiling the values into a table like the ssFlye/ssCanu vs Flye/Canu comparison in the paper. Accept the run when ANI reaches ~99.97% for both E. coli K-12 and W with near-complete coverage and duplication ratios close to 1 (ssFlye), matching the published 99.64%/99.74% ANI and 1.23/1.39 duplication ratios for the Canu baseline.

## Related tools

- **Strainberry** (Performs the strain separation whose outputs (assembly.scaffolds.fa, assembly.scaffolds.bam) are evaluated; version 1.1 used in the analysis workflows) — https://github.com/rvicedomini/strainberry
- **metaFlye** (Generates the strain-oblivious metagenome assembly (flye --meta --pacbio-raw) that serves as Strainberry input and ssFlye baseline)
- **minimap2** (Aligns long reads (preset map-pb for PacBio) to assemblies for the required BAM input and the output-assembly alignments)
- **samtools** (Sorts alignments and produces the samtools faidx / samtools index files Strainberry requires)

## Examples

```
strainberry -r assembly.fasta -b alignment.sorted.bam -o sberry_out -c 4
```

## Evaluation signals

- ssFlye evaluation reaches ANI of 99.97% for both E. coli K-12 and W with near-complete reference coverage
- Duplication ratios of the metaFlye-based separated strains are close to 1
- Canu baseline reproduces the published 99.64%/99.74% ANI and 1.23/1.39 duplication ratios
- Output directory contains assembly.scaffolds.fa, its .fai index, assembly.scaffolds.bam, and .bam.bai after a successful run
- Both assembly FASTA and BAM inputs are indexed (samtools faidx / samtools index) before Strainberry runs

## Limitations

- An average depth of coverage of 60-80x is needed to assemble individual strains with almost complete coverage and sequence identity exceeding 99.9%
- Haplotype phasing assumes evenly split read coverage, which does not hold when strains have different abundance levels; only a limited number of well-represented strains were demonstrated
- Strain-oblivious contigs with high strain variability might need separation into a variable number of strains within the same contig, which is not handled
- Lack of easy-to-use DNA extraction methods producing high molecular weight DNA limits read length (around N50 of 5 Kbp); plasmid genome resolution was not evaluated

## Evidence

- [readme] Strainberry output assembly is stored in the `assembly.scaffolds.fa` file.: "Strainberry output assembly is stored in the `assembly.scaffolds.fa` file.
A minimap2-based alignment of input reads on the output assembly is also available"
- [methods] minimap2 alignment command for BAM input: "minimap2 -ax map-pb -t 12 ./flye_out/assembly.fasta reads.fq.gz | samtools sort >./flye_out/alignment.sorted.bam"
- [abstract] benchmark accuracy claim: "We benchmarked Strainberry on mock communities for which it produces strain-resolved assemblies with near-complete reference coverage and 99.9% base accuracy."
- [discussion] coverage requirement for strain assembly: "An average depth of coverage of 60–80× suffices to assemble individual strains of low-complexity metagenomes with almost complete coverage and sequence identity exceeding 99.9%."
- [readme] readme example run command: [evidence span withheld — anchor status: external_doc]

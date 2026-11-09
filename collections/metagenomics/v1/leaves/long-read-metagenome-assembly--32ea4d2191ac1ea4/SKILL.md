---
name: long-read-metagenome-assembly
description: Use when starting from raw long reads (PacBio .fq.gz or ONT) of a low-complexity
  metagenome and you need a strain-oblivious assembly plus indexed read alignment
  as input to strain-resolved assembly.
when_to_use_negative:
- Input data is short-read sequencing — the article states all prior strain-level
  assembly works were short-read based and 'cannot be re-used' with long reads
- The aim is assembly of a single isolate genome rather than a low-complexity metagenome
  with conspecific strains
- The metagenome is high-complexity with high strain variability requiring a variable
  number of strains separated within the same contig, which Strainberry does not handle
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: metaFlye
  role: Strain-oblivious metagenome assembler run in --meta mode on raw PacBio (or
    ONT) long reads
  repo: https://github.com/rvicedomini/strainberry-analyses
- name: minimap2
  role: Aligns long reads to the strain-oblivious assembly using technology-specific
    presets (-ax map-pb) to feed variant calling/haplotyping
- name: samtools
  role: Coordinate-sorts alignments, and produces the mandatory FASTA (faidx) and
    BAM (index) indexes required by Strainberry
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_006
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-24515-9
    title: 'Strainberry: automated strain separation in low-complexity metagenomes
      using long reads'
  source_content_hash: sha256:86cd3a56f74f81160b5dbc9a36ad63e7bcda29d1df63e69af105e855cbb3cea1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rvicedomini/strainberry-analyses
  tools:
  - Strainberry
  - conda
  - metaFlye
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# long-read-metagenome-assembly

## Summary

Generates a strain-oblivious metagenome assembly from error-prone long reads (PacBio or Oxford Nanopore) with metaFlye, then aligns reads back with minimap2 and indexes the outputs with samtools, producing the FASTA+BAM inputs required by Strainberry for strain separation.

## When to use

Use when starting from raw long reads (PacBio .fq.gz or ONT) of a low-complexity metagenome and you need a strain-oblivious assembly plus indexed read alignment as input to strain-resolved assembly.

## When NOT to use

- Input data is short-read sequencing — the article states all prior strain-level assembly works were short-read based and 'cannot be re-used' with long reads
- The aim is assembly of a single isolate genome rather than a low-complexity metagenome with conspecific strains
- The metagenome is high-complexity with high strain variability requiring a variable number of strains separated within the same contig, which Strainberry does not handle

## Inputs

- Long reads in FASTQ (optionally gzipped), e.g. PacBio reads.fq.gz or Oxford Nanopore reads
- Optional genome-size estimate for metaFlye (e.g. --genome-size 300k, not required from metaFlye 2.8)
- Threads/CPU budget (e.g. 12 threads for flye and minimap2)

## Outputs

- Strain-oblivious metagenome assembly FASTA (flye_out/assembly.fasta)
- Coordinate-sorted long-read alignment BAM (flye_out/alignment.sorted.bam)
- Indexed FASTA (assembly.fasta.fai via samtools faidx) and BAM index (alignment.sorted.bam.bai via samtools index)

## How to apply

Generate the strain-oblivious assembly with metaFlye in metagenome mode (flye --meta --pacbio-raw reads.fq.gz --out-dir flye_out --threads 12; the --genome-size estimate is not required for metaFlye 2.8 or later). Align the long reads back to the assembly with minimap2 using the read-technology preset (e.g. minimap2 -ax map-pb for PacBio, add --nanopore semantics when reads are ONT), coordinate-sort with samtools sort, then index the assembly FASTA with samtools faidx and the sorted BAM with samtools index — both index files are mandatory for Strainberry. The rationale is that Strainberry requires a strain-oblivious metagenome assembly (e.g. generated with metaFlye) and a coordinate-sorted, indexed long-read alignment in BAM format; the variant calling and haplotyping steps consume these exact inputs. Expect ~60–80x average depth of coverage per strain to be sufficient for near-complete coverage and >99.9% sequence identity downstream.

## Related tools

- **metaFlye** (Strain-oblivious metagenome assembler run in --meta mode on raw PacBio (or ONT) long reads) — https://github.com/rvicedomini/strainberry-analyses
- **minimap2** (Aligns long reads to the strain-oblivious assembly using technology-specific presets (-ax map-pb) to feed variant calling/haplotyping)
- **samtools** (Coordinate-sorts alignments, and produces the mandatory FASTA (faidx) and BAM (index) indexes required by Strainberry)

## Examples

```
flye --meta --pacbio-raw reads.fq.gz --out-dir flye_out --genome-size 300k --threads 12
```

## Evaluation signals

- The output directory contains both flye_out/assembly.fasta and flye_out/alignment.sorted.bam, plus .fai and .bai index files from samtools faidx/index
- On the K12/W example, assembly.fasta contains a single consensus sequence of a small region of E. coli strains K12 and W, and the downstream Strainberry run yields two scaffolds (one closer to K12, one closer to W)
- Downstream strain separation reaches near-complete reference coverage and ~99.9% base accuracy when average per-strain depth of coverage is 60–80x
- Per-strain metrics (ANI, reference coverage, duplication ratio) computed against known references match reported values (e.g. ANI 99.97% for both E. coli K-12 and W in ssFlye on Mock3)
- The minimap2 preset matches the sequencing chemistry (-ax map-pb for PacBio, --nanopore flag passed to Strainberry for ONT data)

## Limitations

- Read length is limited to around N50 of 5 Kbp because DNA extraction methods producing high molecular weight DNA for all bacteria are lacking
- An average depth of coverage of 60–80x per strain is needed; lower depth degrades strain resolution
- Relationship between depth of coverage, read length, and ability to resolve haplotypes merits further study
- In the presence of high strain variability, strain-oblivious sequences might need separation into a variable number of strains within the same contig, which is not handled

## Evidence

- [methods] metaFlye assembly generation: "flye --meta --pacbio-raw reads.fq.gz --out-dir flye_out --genome-size 300k --threads 12"
- [methods] Read alignment and sorting: "minimap2 -ax map-pb -t 12 ./flye_out/assembly.fasta reads.fq.gz | samtools sort >./flye_out/alignment.sorted.bam"
- [readme] Indexing requirements: "Both `FASTA` and `BAM` files are expected to be indexed with the `samtools faidx` and `samtools index` commands, respectively."
- [abstract] Long-read-only reliance: "Strainberry, a metagenome assembly pipeline that performs strain separation in single-sample low-complexity metagenomes and that relies uniquely on long-read data."
- [discussion] Sufficient coverage depth: "An average depth of coverage of 60–80× suffices to assemble individual strains of low-complexity metagenomes with almost complete coverage and sequence identity exceeding 99.9%."
- [readme] Example input description: "The file `assembly.fasta` contains a single sequence which is a consensus of a small region of *E. coli* strains K12 and W."

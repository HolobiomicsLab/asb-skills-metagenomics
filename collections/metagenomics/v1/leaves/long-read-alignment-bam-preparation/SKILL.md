---
name: long-read-alignment-bam-preparation
description: Use when a strain-oblivious metagenome assembly exists (or is being built
  with metaFlye) and Strainberry requires a coordinate-sorted, indexed long-read alignment
  BAM paired with an indexed FASTA.
when_to_use_negative:
- The input is already a Strainberry output directory with assembly.scaffolds.bam
  — no realignment is needed
- The goal is strain separation with short reads, which Strainberry does not support
  (all prior strain-assembly methods were short-read based and could not be reused)
- The assembly is already strain-separated rather than strain-oblivious; Strainberry
  expects a metaFlye-style consensus reference
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0102
tools:
- name: metaFlye
  role: Generates the strain-oblivious metagenome assembly that serves as the alignment
    reference (flye --meta --pacbio-raw)
  repo: https://github.com/rvicedomini/strainberry
- name: minimap2
  role: Aligns long reads to the strain-oblivious assembly in PacBio mode (-ax map-pb),
    piped to samtools sort
  repo: https://github.com/rvicedomini/strainberry
- name: samtools
  role: Sorts alignments into a coordinate-sorted BAM (samtools sort) and creates
    the required .fai (samtools faidx) and .bai (samtools index) indexes
  repo: https://github.com/rvicedomini/strainberry
- name: Badread
  role: Simulates the PacBio long reads (v0.2.0, --quantity 50x --error_model pacbio2016)
    that are aligned in benchmark datasets
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-24515-9
    title: 'Strainberry: automated strain separation in low-complexity metagenomes
      using long reads'
  source_content_hash: sha256:a711a1b3b2d368dc6be30e2fbbce3710dc6127f5ef1f42df06c2c971bfc44f5d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rvicedomini/strainberry
  tools:
  - Badread
  - Strainberry
  - metaFlye
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# long-read-alignment-bam-preparation

## Summary

Prepares the Strainberry input pair: a strain-oblivious metagenome assembly (metaFlye) and a coordinate-sorted, indexed long-read BAM produced by aligning PacBio or Nanopore reads with minimap2 and samtools.

## When to use

Use when a strain-oblivious metagenome assembly exists (or is being built with metaFlye) and Strainberry requires a coordinate-sorted, indexed long-read alignment BAM paired with an indexed FASTA.

## When NOT to use

- The input is already a Strainberry output directory with assembly.scaffolds.bam — no realignment is needed
- The goal is strain separation with short reads, which Strainberry does not support (all prior strain-assembly methods were short-read based and could not be reused)
- The assembly is already strain-separated rather than strain-oblivious; Strainberry expects a metaFlye-style consensus reference

## Inputs

- Long-read FASTQ reads (PacBio simulated reads from Badread, e.g. --error_model pacbio2016 --identity 90,97,3 --length 6100,3700, or real Nanopore reads such as the HSM dataset)
- Strain-oblivious metagenome assembly FASTA generated with metaFlye (flye --meta --pacbio-raw)

## Outputs

- Coordinate-sorted, indexed BAM of long reads aligned to the strain-oblivious assembly (alignment.sorted.bam plus .bai)
- Indexed assembly FASTA (assembly.fasta.fai from samtools faidx)
- Input pair (FASTA + BAM) ready for strainberry -r ... -b ...

## How to apply

First generate the strain-oblivious reference with metaFlye in metagenome mode (flye --meta --pacbio-raw reads.fq.gz --out-dir flye_out --genome-size 300k --threads 12). Align the long reads to assembly.fasta with minimap2 in the mode matching the read chemistry — -ax map-pb for PacBio reads; Nanopore reads are instead handled downstream by Strainberry's --nanopore flag — and pipe through samtools sort to yield alignment.sorted.bam. Index both artifacts before invoking Strainberry: samtools faidx on the assembly FASTA and samtools index on the sorted BAM, since Strainberry refuses to run without these indexes. The resulting FASTA+BAM pair is then passed as -r and -b to strainberry.

## Related tools

- **metaFlye** (Generates the strain-oblivious metagenome assembly that serves as the alignment reference (flye --meta --pacbio-raw)) — https://github.com/rvicedomini/strainberry
- **minimap2** (Aligns long reads to the strain-oblivious assembly in PacBio mode (-ax map-pb), piped to samtools sort) — https://github.com/rvicedomini/strainberry
- **samtools** (Sorts alignments into a coordinate-sorted BAM (samtools sort) and creates the required .fai (samtools faidx) and .bai (samtools index) indexes) — https://github.com/rvicedomini/strainberry
- **Badread** (Simulates the PacBio long reads (v0.2.0, --quantity 50x --error_model pacbio2016) that are aligned in benchmark datasets)

## Examples

```
minimap2 -ax map-pb -t 12 ./flye_out/assembly.fasta reads.fq.gz | samtools sort >./flye_out/alignment.sorted.bam; samtools faidx ./flye_out/assembly.fasta; samtools index ./flye_out/alignment.sorted.bam
```

## Evaluation signals

- alignment.sorted.bam is coordinate-sorted and has a companion .bai from samtools index
- assembly.fasta has a companion .fai from samtools faidx — Strainberry requires both files indexed
- Alignment command matches read technology: minimap2 -ax map-pb for PacBio reads; for Nanopore input the Strainberry call must carry the --nanopore flag
- On the README example (E. coli K12/W consensus region), a successful downstream run yields two scaffolds in assembly.scaffolds.fa, one closer to K12, one closer to W
- In the Mock3 downsampling series, every simulated dataset in the manifest has a non-empty FASTQ and a coordinate-sorted indexed BAM

## Limitations

- Both the FASTA and BAM must be indexed (samtools faidx / samtools index) or Strainberry will not run
- For metaFlye versions earlier than 2.8, --genome-size must be supplied (e.g. 300k in the example); it is not required from version 2.8
- The strain-oblivious assembly must be a consensus that mixes strains; per the paper, high strain variability within a contig may require separation into a variable number of strains, which is not handled

## Evidence

- [methods] minimap2 -ax map-pb -t 12 ./flye_out/assembly.fasta reads.fq.gz | samtools sort >./flye_out/alignment.sorted.bam: "minimap2 -ax map-pb -t 12 ./flye_out/assembly.fasta reads.fq.gz | samtools sort >./flye_out/alignment.sorted.bam"
- [methods] Generate strain-oblivious metagenome assembly with metaFlye: "flye --meta --pacbio-raw reads.fq.gz --out-dir flye_out --genome-size 300k --threads 12"
- [readme] Indexed inputs required by Strainberry: "Both `FASTA` and `BAM` files are expected to be indexed with the `samtools faidx` and `samtools index` commands, respectively."
- [readme] Alignment input chemistry handled downstream: "--nanopore            Input consists of Oxford Nanopore reads"
- [intro] Pipeline position of the strain-oblivious assembly: "Strainberry combines a strain-oblivious assembler with the careful use of a long-read variant calling and haplotyping tool"

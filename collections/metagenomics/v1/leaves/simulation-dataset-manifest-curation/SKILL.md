---
name: simulation-dataset-manifest-curation
description: Use when benchmarking strain-resolved assembly tools on simulated low-complexity
  metagenomes and you need reproducible read simulations, coverage series, aligned
  BAM artifacts, and a manifest enumerating all datasets.
when_to_use_negative:
- The analysis targets real (non-simulated) long-read datasets such as the human stool
  microbiome, Mock9, or NWC2 runs handled by the strainberry-analyses Snakefiles
- Strain variability is high enough that strain-oblivious contigs would need separation
  into a variable number of strains, a case Strainberry does not handle
- The community is high-complexity rather than low-complexity with a limited number
  of well-represented strains
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Badread
  role: Simulates PacBio long reads for each conspecific-strain community with pacbio2016
    error model, 50x quantity, and specified identity/length distributions
- name: metaFlye
  role: Generates the strain-oblivious metagenome reference assembly for each simulated
    dataset (flye --meta --pacbio-raw)
- name: minimap2
  role: Aligns simulated PacBio reads to each strain-oblivious assembly in map-pb
    mode
- name: samtools
  role: Sorts the minimap2 alignments into coordinate-sorted BAMs and indexes them
    with faidx and index for downstream Strainberry input
- name: Strainberry
  role: Downstream consumer of the FASTA/BAM artifacts prepared by this skill; the
    manifest enables the coverage and strain-number benchmarks
  repo: https://github.com/rvicedomini/strainberry
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-24515-9
    title: 'Strainberry: automated strain separation in low-complexity metagenomes
      using long reads'
  source_content_hash: sha256:0b05f46bc1a2eca92a6196ada70820b22ed4d6c41e1fba75199bc6b40f600f68
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
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# simulation-dataset-manifest-curation

## Summary

Simulates a panel of conspecific-strain metagenome communities (seven two-strain E. coli divergence communities, Mock3, and 2-5 strain communities) with Badread v0.2.0 PacBio reads, downsamples Mock3 to 5x-50x, prepares strain-oblivious assemblies and BAM alignments, and compiles a manifest TSV enumerating every dataset and artifact.

## When to use

Use when benchmarking strain-resolved assembly tools on simulated low-complexity metagenomes and you need reproducible read simulations, coverage series, aligned BAM artifacts, and a manifest enumerating all datasets.

## When NOT to use

- The analysis targets real (non-simulated) long-read datasets such as the human stool microbiome, Mock9, or NWC2 runs handled by the strainberry-analyses Snakefiles
- Strain variability is high enough that strain-oblivious contigs would need separation into a variable number of strains, a case Strainberry does not handle
- The community is high-complexity rather than low-complexity with a limited number of well-represented strains

## Inputs

- NCBI reference genomes for seven two-strain E. coli divergence communities, the Mock3 community, and 2-5 conspecific E. coli strain communities (Supplementary Table 2)
- Badread parameters: --quantity 50x, --error_model pacbio2016, --identity 90,97,3, --length 6100,3700
- Mock3 simulated FASTQ reads to be subsampled at 5x-50x coverage

## Outputs

- Simulated PacBio FASTQ read files per community and per downsampling level
- Strain-oblivious metaFlye assemblies (assembly.fasta) per dataset
- Coordinate-sorted, indexed BAM alignments (alignment.sorted.bam plus .bai) per dataset
- Manifest TSV with community composition, genome accessions, coverage, Badread parameters, and FASTQ/BAM file paths

## How to apply

Retrieve the reference genomes enumerated in Supplementary Table 2 (seven two-strain E. coli divergence communities, Mock3, and the 2-5 conspecific E. coli strain communities) from NCBI. Simulate PacBio long reads for each community with Badread v0.2.0 using --quantity 50x --error_model pacbio2016 --identity 90,97,3 --length 6100,3700, then subsample the Mock3 simulated reads to coverage levels from 5x to 50x to build the downsampling series. For each dataset, generate a strain-oblivious reference by running metaFlye in metagenome mode (flye --meta --pacbio-raw), align the simulated reads with minimap2 in PacBio mode (minimap2 -ax map-pb), pipe through samtools sort, and index with samtools faidx and samtools index. Finally, compile a manifest TSV recording, for every dataset, the community composition, source genome accessions, sequencing coverage, Badread parameters, and file paths to the FASTQ and BAM artifacts.

## Related tools

- **Badread** (Simulates PacBio long reads for each conspecific-strain community with pacbio2016 error model, 50x quantity, and specified identity/length distributions)
- **metaFlye** (Generates the strain-oblivious metagenome reference assembly for each simulated dataset (flye --meta --pacbio-raw))
- **minimap2** (Aligns simulated PacBio reads to each strain-oblivious assembly in map-pb mode)
- **samtools** (Sorts the minimap2 alignments into coordinate-sorted BAMs and indexes them with faidx and index for downstream Strainberry input)
- **Strainberry** (Downstream consumer of the FASTA/BAM artifacts prepared by this skill; the manifest enables the coverage and strain-number benchmarks) — https://github.com/rvicedomini/strainberry

## Examples

```
flye --meta --pacbio-raw reads.fq.gz --out-dir flye_out --genome-size 300k --threads 12; minimap2 -ax map-pb -t 12 flye_out/assembly.fasta reads.fq.gz | samtools sort > flye_out/alignment.sorted.bam; samtools index flye_out/alignment.sorted.bam
```

## Evaluation signals

- Every simulated dataset in the manifest has a non-empty FASTQ file
- Every dataset has a coordinate-sorted, indexed BAM (alignment.sorted.bam with .bai present)
- The manifest enumerates all seven divergence communities
- The manifest enumerates all Mock3 downsampling levels from 5x to 50x
- The manifest enumerates all 2-5 conspecific E. coli strain communities, and each row records accessions, coverage, and Badread parameters

## Limitations

- Read length is bounded by DNA-extraction limits, around an N50 of 5 Kbp, reflected in the simulated 6100,3700 length distribution
- Haplotype phasing assumes evenly split read coverage, which does not hold when strains exhibit different abundance/coverage levels
- The downsampling series starts at 5x; below 20x coverage the reference representation and sequence identity degrade (near-complete >95% representation with >99.8% identity requires at least 20x)

## Evidence

- [supplementary] Simulate PacBio long reads for each community with Badread v0.2.0 using --quantity 50x --error_model pacbio2016 --identity 90,97,3 --length 6100,3700.: [evidence span withheld — anchor status: not_found]
- [results] Strainberry separation quality plateaus after 30x coverage: [evidence span withheld — anchor status: not_found]
- [supplementary] Build a strain-oblivious reference for each dataset by running metaFlye in metagenome mode: [evidence span withheld — anchor status: not_found]
- [methods] Align long reads to assembly with minimap2 and sort with samtools: "minimap2 -ax map-pb -t 12 ./flye_out/assembly.fasta reads.fq.gz | samtools sort >./flye_out/alignment.sorted.bam"
- [readme] Both `FASTA` and `BAM` files are expected to be indexed: "Both `FASTA` and `BAM` files are expected to be indexed with the `samtools faidx` and `samtools index` commands, respectively."

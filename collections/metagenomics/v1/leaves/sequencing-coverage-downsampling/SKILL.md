---
name: sequencing-coverage-downsampling
description: Use when benchmarking how coverage depth affects strain separation quality,
  e.g. building a Mock3 5x–50x downsampling series of simulated PacBio reads before
  strain-oblivious assembly and Strainberry separation.
when_to_use_negative:
- The goal is strain separation itself rather than benchmarking depth sensitivity
  — a single adequately covered dataset (60–80x) suffices
- Input reads are real experimental data whose coverage cannot be controlled by simulation
  parameters; subsampling still works but Badread simulation steps are unnecessary
- The metagenome is high-complexity — Strainberry is designed for low-complexity communities
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0196
tools:
- name: Badread
  role: Simulates PacBio long reads (--quantity 50x --error_model pacbio2016 --identity
    90,97,3 --length 6100,3700) that are then subsampled to build the coverage series
- name: metaFlye
  role: Produces the strain-oblivious metagenome assembly (flye --meta --pacbio-raw)
    for each downsampling level
- name: minimap2
  role: Aligns simulated reads to each assembly in PacBio mode (-ax map-pb) to generate
    BAM input for Strainberry
- name: samtools
  role: Sorts alignments and indexes assembly FASTA (faidx) and sorted BAM (index)
    as required by Strainberry
- name: Strainberry
  role: Performs the strain separation evaluated across each coverage level; consumes
    the indexed FASTA and coordinate-sorted BAM
  repo: https://github.com/rvicedomini/strainberry
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-24515-9
    title: 'Strainberry: automated strain separation in low-complexity metagenomes
      using long reads'
  source_content_hash: sha256:50ba9648118266e4120c22f8a01e4d6a38a4e4e7f05ea50a4be3dee9d89d2cca
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# sequencing-coverage-downsampling

## Summary

Builds a series of coverage-reduced datasets by subsampling simulated long reads (5x–50x from a 50x Mock3 PacBio simulation) to quantify how sequencing depth affects strain-resolved assembly quality with Strainberry.

## When to use

Use when benchmarking how coverage depth affects strain separation quality, e.g. building a Mock3 5x–50x downsampling series of simulated PacBio reads before strain-oblivious assembly and Strainberry separation.

## When NOT to use

- The goal is strain separation itself rather than benchmarking depth sensitivity — a single adequately covered dataset (60–80x) suffices
- Input reads are real experimental data whose coverage cannot be controlled by simulation parameters; subsampling still works but Badread simulation steps are unnecessary
- The metagenome is high-complexity — Strainberry is designed for low-complexity communities

## Inputs

- Reference genomes for conspecific E. coli strain communities and the Mock3 community (NCBI accessions from Supplementary Table 2)
- Simulated PacBio long reads (FASTQ) generated with Badread v0.2.0 at 50x coverage
- Downsampled FASTQ series at 5x–50x coverage
- Strain-oblivious metaFlye assembly (FASTA)
- Coordinate-sorted, indexed BAM of reads aligned to the assembly

## Outputs

- Downsampling series of subsampled FASTQ files spanning 5x–50x coverage
- Per-coverage-level metaFlye assemblies and indexed BAM alignments
- Manifest table (TSV) with community composition, genome accessions, coverage, Badread parameters, and FASTQ/BAM file paths
- Coverage-dependent Strainberry separation quality metrics (reference representation %, sequence identity, strain number)

## How to apply

First simulate PacBio long reads for each conspecific-strain community with Badread v0.2.0 using --quantity 50x, --error_model pacbio2016, --identity 90,97,3, and --length 6100,3700, so all downsampling levels derive from one consistent read set. Then subsample the Mock3 simulated reads to coverage levels from 5x to 50x, keeping the read-length and error distributions intact. For each coverage level, build a strain-oblivious reference by running metaFlye in metagenome mode (flye --meta --pacbio-raw), align reads with minimap2 (-ax map-pb), sort with samtools, and index the FASTA (samtools faidx) and sorted BAM (samtools index) before running Strainberry. The rationale is that the article reports near-complete reference representation (>95%) with >99.8% sequence identity requires at least 20x coverage, while an average depth of 60–80x suffices for almost complete coverage with >99.9% identity, so the 5x–50x series brackets the transition from insufficient to sufficient depth.

## Related tools

- **Badread** (Simulates PacBio long reads (--quantity 50x --error_model pacbio2016 --identity 90,97,3 --length 6100,3700) that are then subsampled to build the coverage series)
- **metaFlye** (Produces the strain-oblivious metagenome assembly (flye --meta --pacbio-raw) for each downsampling level)
- **minimap2** (Aligns simulated reads to each assembly in PacBio mode (-ax map-pb) to generate BAM input for Strainberry)
- **samtools** (Sorts alignments and indexes assembly FASTA (faidx) and sorted BAM (index) as required by Strainberry)
- **Strainberry** (Performs the strain separation evaluated across each coverage level; consumes the indexed FASTA and coordinate-sorted BAM) — https://github.com/rvicedomini/strainberry

## Examples

```
badread simulate --quantity 50x --error_model pacbio2016 --identity 90,97,3 --length 6100,3700 --reference mock3_genomes.fa > mock3_50x.fq.gz
```

## Evaluation signals

- Every simulated dataset in the manifest has a non-empty FASTQ and a coordinate-sorted, indexed BAM
- Manifest enumerates all seven two-strain divergence communities, all 5x–50x Mock3 downsampling levels, and all 2–5 conspecific strain communities
- Downsampled FASTQ coverage levels fall in the 5x–50x range with read length/error distributions matching the parent Badread simulation
- Coverage-response curve shows the expected trend: quality plateaus after 30x, and >95% reference representation with >99.8% identity occurs only at ≥20x
- At 60–80x, assemblies approach almost complete coverage with sequence identity exceeding 99.9%

## Limitations

- Haplotype phasing benefits from evenly split read coverage, an assumption that no longer holds in metagenomic settings where strains have different abundance/coverage levels
- Separation quality was demonstrated only for a limited number of well-represented strains
- Read length is limited (~N50 5 Kbp) by DNA extraction methods, affecting the coverage-resolvability relationship, which the authors note merits further study

## Evidence

- [discussion] Coverage sufficiency for strain assembly: "An average depth of coverage of 60–80× suffices to assemble individual strains of low-complexity metagenomes with almost complete coverage and sequence identity exceeding 99.9%."
- [discussion] Coverage/read-length relationship open question: "A key aspect that would merit further study is the relationship between depth of coverage of the sample, read length, and ability to resolve haplotypes."
- [methods] Downstream pipeline per coverage level: "minimap2 -ax map-pb -t 12 ./flye_out/assembly.fasta reads.fq.gz | samtools sort >./flye_out/alignment.sorted.bam"
- [readme] Indexing requirement for Strainberry inputs: "Both `FASTA` and `BAM` files are expected to be indexed with the `samtools faidx` and `samtools index` commands, respectively."

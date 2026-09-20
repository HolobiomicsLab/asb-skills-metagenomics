---
name: metagenome-community-composition-design
description: Use when you need simulated long-read metagenome communities of known
  strain composition (e.g. 2-5 conspecific E. coli strains, Mock3 downsampling at
  5x-50x) to benchmark strain-separation tools like Strainberry.
when_to_use_negative:
- The input is real (non-simulated) sequencing data with unknown strain composition
  — this skill designs reference-truth communities, it does not characterize real
  samples
- Short-read data or Illumina simulation is required — the workflow uses Badread PacBio
  2016 error modelling and PacBio-mode minimap2 alignment
- High-complexity metagenomes with many strains are the target — Strainberry benchmarks
  target low-complexity communities (2-5 conspecific strains)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: Badread
  role: Simulates PacBio long reads for each community with error model pacbio2016,
    identity 90,97,3 and length 6100,3700 at 50x quantity
- name: metaFlye
  role: Builds the strain-oblivious metagenome reference assembly from simulated reads
    (flye --meta --pacbio-raw)
- name: minimap2
  role: Aligns simulated PacBio reads to the metaFlye reference in PacBio mode (-ax
    map-pb)
- name: samtools
  role: Sorts alignments, indexes the BAM (samtools index) and the assembly FASTA
    (samtools faidx)
- name: Strainberry
  role: Downstream strain-separation benchmark target consuming the strain-oblivious
    FASTA and coordinate-sorted BAM
  repo: https://github.com/rvicedomini/strainberry
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-24515-9
    title: 'Strainberry: automated strain separation in low-complexity metagenomes
      using long reads'
  source_content_hash: sha256:f4a8be9643648526e2accfdfae7a0a333d563b486729a2c475c99926319b78a4
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# metagenome-community-composition-design

## Summary

Design and simulate conspecific-strain metagenome communities (PacBio long reads) with Badread to benchmark strain-resolved assembly: build strain-oblivious references with metaFlye and read alignments with minimap2/samtools, catalogued in a manifest TSV. This recreates the Strainberry benchmark datasets for coverage (5x-50x Mock3), divergence, and strain-number (2-5 E. coli strains) tests.

## When to use

Use when you need simulated long-read metagenome communities of known strain composition (e.g. 2-5 conspecific E. coli strains, Mock3 downsampling at 5x-50x) to benchmark strain-separation tools like Strainberry.

## When NOT to use

- The input is real (non-simulated) sequencing data with unknown strain composition — this skill designs reference-truth communities, it does not characterize real samples
- Short-read data or Illumina simulation is required — the workflow uses Badread PacBio 2016 error modelling and PacBio-mode minimap2 alignment
- High-complexity metagenomes with many strains are the target — Strainberry benchmarks target low-complexity communities (2-5 conspecific strains)

## Inputs

- Reference genomes (NCBI accessions) for seven two-strain E. coli divergence communities, the Mock3 community, and 2-5 conspecific E. coli strain communities (Supplementary Table 2)
- Community composition specification (strain identities, expected coverage levels)
- Simulated PacBio read FASTQ files produced by Badread v0.2.0

## Outputs

- Badread-simulated long-read FASTQ files at 50x (--quantity 50x, --error_model pacbio2016, --identity 90,97,3, --length 6100,3700)
- Mock3 downsampling series spanning 5x to 50x coverage
- Strain-oblivious metaFlye metagenome assembly FASTA per dataset (flye --meta --pacbio-raw)
- Coordinate-sorted indexed BAM alignments (minimap2 -ax map-pb | samtools sort; samtools index; samtools faidx)
- Manifest TSV recording community composition, source genome accessions, coverage, Badread parameters, and FASTQ/BAM file paths

## How to apply

Retrieve reference genomes from NCBI for the seven two-strain E. coli divergence communities, the Mock3 community, and the 2-5 conspecific E. coli strain communities enumerated in Supplementary Table 2. Simulate PacBio long reads per community with Badread v0.2.0 using --quantity 50x --error_model pacbio2016 --identity 90,97,3 --length 6100,3700; build a downsampling series by subsampling the Mock3 reads from 5x to 50x. For each dataset, build a strain-oblivious reference by running metaFlye in metagenome mode (flye --meta --pacbio-raw), then align simulated reads with minimap2 PacBio mode (minimap2 -ax map-pb), sort with samtools sort, and index with samtools faidx and samtools index. Compile a manifest TSV per dataset recording community composition, source genome accessions, coverage, Badread parameters, and paths to the FASTQ and BAM artifacts; validate that every entry has a non-empty FASTQ, a coordinate-sorted indexed BAM, and complete enumeration of all divergence communities, 5x-50x Mock3 levels, and 2-5 strain communities. Coverage design matters because separation plateaus after 30x, while >95% reference representation with >99.8% sequence identity requires at least 20x.

## Related tools

- **Badread** (Simulates PacBio long reads for each community with error model pacbio2016, identity 90,97,3 and length 6100,3700 at 50x quantity)
- **metaFlye** (Builds the strain-oblivious metagenome reference assembly from simulated reads (flye --meta --pacbio-raw))
- **minimap2** (Aligns simulated PacBio reads to the metaFlye reference in PacBio mode (-ax map-pb))
- **samtools** (Sorts alignments, indexes the BAM (samtools index) and the assembly FASTA (samtools faidx))
- **Strainberry** (Downstream strain-separation benchmark target consuming the strain-oblivious FASTA and coordinate-sorted BAM) — https://github.com/rvicedomini/strainberry

## Examples

```
badread simulate --quantity 50x --error_model pacbio2016 --identity 90,97,3 --length 6100,3700
```

## Evaluation signals

- Every simulated dataset in the manifest has a non-empty FASTQ file
- Every dataset has a coordinate-sorted, indexed BAM (samtools index succeeded)
- Manifest enumerates all seven two-strain divergence communities
- Manifest enumerates all Mock3 downsampling levels from 5x to 50x
- Manifest enumerates all 2-5 conspecific E. coli strain communities and records Badread parameters and file paths for each dataset

## Limitations

- Separation quality plateaus after 30x coverage; near-complete (>95%) reference representation with >99.8% sequence identity requires at least 20x coverage, so datasets below ~20x will underperform
- Read length is limited (~N50 of 5 Kbp) by lack of easy DNA extraction methods producing high molecular weight DNA from all bacteria
- Strainberry was not evaluated on plasmid genomes, and in presence of high strain variability a single strain-oblivious contig may need separation into a variable number of strains, which is not handled

## Evidence

- [supplementary] Simulate PacBio long reads for each community with Badread v0.2.0 using --quantity 50x --error_model pacbio2016 --identity 90,97,3 --length 6100,3700: [evidence span withheld — anchor status: not_found]
- [supplementary] Retrieve reference genomes from NCBI (Supplementary Table 2) for divergence, Mock3 and 2-5 strain communities: [evidence span withheld — anchor status: not_found]
- [readme] README example command aligning reads to the metaFlye assembly and sorting with samtools: "minimap2 -ax map-pb -t 12 ./flye_out/assembly.fasta reads.fq.gz | samtools sort >./flye_out/alignment.sorted.bam"
- [discussion] Article finding on coverage sufficiency for assembling individual strains: "An average depth of coverage of 60–80× suffices to assemble individual strains of low-complexity metagenomes with almost complete coverage and sequence identity exceeding 99.9%."
- [abstract] Article benchmark claim of near-complete reference coverage and 99.9% base accuracy on mock communities: "We benchmarked Strainberry on mock communities for which it produces strain-resolved assemblies with near-complete reference coverage and 99.9% base accuracy."

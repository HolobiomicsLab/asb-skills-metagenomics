---
name: long-read-read-simulation
description: Use when a benchmark needs controlled long-read metagenome communities
  — e.g. varying strain divergence, strain number, or coverage depth — and real reads
  cannot provide known ground-truth strain genomes.
when_to_use_negative:
- The goal is to benchmark on real sequencing data — simulation only provides in-silico
  ground truth and does not capture real sample complexity
- Strain multiplicity exceeds what the method handles — Strainberry attempts strain-separation
  at most 5 strains by default (-n/--max-strains 5)
- A strain-oblivious assembly or read alignment already exists — this skill only generates
  the raw simulated reads, not downstream assembly
edam_operation: http://edamontology.org/operation_3559
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Badread
  role: PacBio long-read simulator used to generate reads with --error_model pacbio2016
    and per-strain identity drawn from 90,97,3
- name: metaFlye
  role: Builds a strain-oblivious reference (flye --meta --pacbio-raw) on the simulated
    reads so alignments can be made for downstream Strainberry runs
- name: minimap2
  role: Aligns simulated reads to the metaFlye reference in PacBio mode (minimap2
    -ax map-pb)
- name: samtools
  role: Sorts and indexes alignments (samtools sort, samtools faidx, samtools index)
    to produce coordinate-sorted BAM inputs
- name: Strainberry
  role: Downstream strain-separation assembler consuming the simulated reads/alignments;
    the simulated communities are its benchmark inputs
  repo: https://github.com/rvicedomini/strainberry
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-24515-9
    title: 'Strainberry: automated strain separation in low-complexity metagenomes
      using long reads'
  source_content_hash: sha256:216e7a844f1b00137253f90caec59c17b01ca51ad6f6482e6926b98e044dfd31
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
  edam_operation: http://edamontology.org/operation_3559
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# long-read-read-simulation

## Summary

Simulate PacBio long reads for in-silico metagenome communities with Badread using the Strainberry benchmark parameters (--quantity 50x, --error_model pacbio2016, --identity 90,97,3, --length 6100,3700), then subsample (e.g. Mock3 at 5x-50x) to build controlled coverage series for strain-assembly benchmarks.

## When to use

Use when a benchmark needs controlled long-read metagenome communities — e.g. varying strain divergence, strain number, or coverage depth — and real reads cannot provide known ground-truth strain genomes.

## When NOT to use

- The goal is to benchmark on real sequencing data — simulation only provides in-silico ground truth and does not capture real sample complexity
- Strain multiplicity exceeds what the method handles — Strainberry attempts strain-separation at most 5 strains by default (-n/--max-strains 5)
- A strain-oblivious assembly or read alignment already exists — this skill only generates the raw simulated reads, not downstream assembly

## Inputs

- NCBI reference genomes for conspecific E. coli strain communities (7 two-strain divergence communities, Mock3, and 2-5 strain communities per Supplementary Table 2)
- Badread v0.2.0 simulation parameters (quantity 50x, pacbio2016 error model)
- Target subsampling coverage levels (5x-50x for the Mock3 downsampling series)

## Outputs

- Simulated PacBio long-read FASTQ files per community
- Subsampled FASTQ files at each coverage level (5x-50x)
- Manifest TSV with community composition, accessions, coverage, Badread parameters, and artifact paths

## How to apply

Retrieve the reference genomes for each community (the seven two-strain E. coli divergence communities, the Mock3 community, and the 2-5 conspecific E. coli strain communities enumerated in Supplementary Table 2) from NCBI. Simulate PacBio reads for each community with Badread v0.2.0 using --quantity 50x --error_model pacbio2016 --identity 90,97,3 --length 6100,3700, which reproduces the error and length profile used in the Strainberry benchmarks. For coverage-depth benchmarks, subsample the Mock3 simulated reads to a series of coverage levels from 5x to 50x (separation plateaus after 30x; >95% reference representation with >99.8% identity needs at least 20x). Compile a manifest TSV per dataset recording community composition, source accessions, coverage, Badread parameters, and FASTQ/BAM paths. Validate that every simulated dataset has a non-empty FASTQ and that the manifest enumerates all communities and subsampling levels.

## Related tools

- **Badread** (PacBio long-read simulator used to generate reads with --error_model pacbio2016 and per-strain identity drawn from 90,97,3)
- **metaFlye** (Builds a strain-oblivious reference (flye --meta --pacbio-raw) on the simulated reads so alignments can be made for downstream Strainberry runs)
- **minimap2** (Aligns simulated reads to the metaFlye reference in PacBio mode (minimap2 -ax map-pb))
- **samtools** (Sorts and indexes alignments (samtools sort, samtools faidx, samtools index) to produce coordinate-sorted BAM inputs)
- **Strainberry** (Downstream strain-separation assembler consuming the simulated reads/alignments; the simulated communities are its benchmark inputs) — https://github.com/rvicedomini/strainberry

## Examples

```
badread simulate --quantity 50x --error_model pacbio2016 --identity 90,97,3 --length 6100,3700 reference.fasta > simulated_reads.fq
```

## Evaluation signals

- Every simulated dataset in the manifest has a non-empty FASTQ file
- Every dataset has a coordinate-sorted, indexed BAM alignment
- The manifest enumerates all seven two-strain divergence communities
- The manifest enumerates all 5x-50x Mock3 downsampling levels
- The manifest enumerates all 2-5 conspecific E. coli strain communities with correct source accessions

## Limitations

- Simulated reads rely on Badread's error/length models (pacbio2016, identity 90,97,3, length 6100,3700), which may not fully reflect real library characteristics
- Simulated coverage of 50x per community is the upper bound of the benchmark; deeper or shallower regimes are not covered here
- Downsampling is only performed for the Mock3 dataset; other communities are simulated at a single coverage level

## Evidence

- [supplementary] Badread parameters: [evidence span withheld — anchor status: not_found]
- [supplementary] coverage downsampling: [evidence span withheld — anchor status: not_found]
- [results] Strainberry coverage finding: "An average depth of coverage of 60–80× suffices to assemble individual strains of low-complexity metagenomes with almost complete coverage and sequence identity exceeding 99.9%."
- [readme] Strainberry installation: "git clone https://github.com/rvicedomini/strainberry.git
 cd strainberry
 conda env create -n sberry --file environment.yml"
- [supplementary] reference genome retrieval: [evidence span withheld — anchor status: not_found]

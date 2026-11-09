---
name: coverage-and-error-rate-parameterization
description: Use when benchmarking a contig extension or assembly tool on known viral
  reference genomes (e.g. BASV, BKV, HIV-1) and you need controlled variation in read
  length, sequencing error rate, coverage depth, and coverage unevenness.
when_to_use_negative:
- Your goal is to evaluate performance on real metagenomic samples — in silico simulation
  is a proof-of-concept surrogate and does not capture real sample complexity
- You need to simulate reads from non-viral or eukaryotic genomes not covered by the
  NCBI viral reference set used here
- You cannot install or run pIRS and only need paired-end reads without controlled
  error/coverage parameterization
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0781
tools:
- name: pIRS
  role: Simulates paired-end 100 bp Illumina reads from viral reference genomes at
    a specified error rate (0.05) for the controlled-parameterization benchmark
- name: ContigExtender
  role: Downstream contig-extension tool evaluated on the parameterized synthetic
    read sets
  repo: https://github.com/dengzac/contig-extender
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-021-04038-2
    title: 'ContigExtender: a new approach to improving de novo sequence assembly
      for viral metagenomics data'
  source_content_hash: sha256:2e6f8fbd48671f85d0af28f4b82a93cd351d434e1a08177fbdaadb694163166c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dengzac/contig-extender
  tools:
  - ContigExtender
  - pIRS
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0781
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# coverage-and-error-rate-parameterization

## Summary

Simulate in silico viral read datasets with controlled read lengths (100/250 bp), error rates (1%/5%), and coverage depths (10x/20x/50x) — including 50x coverage-peak spike-ins every 1 kb — to benchmark contig-extension tools like ContigExtender under realistic uneven metagenomic coverage.

## When to use

Use when benchmarking a contig extension or assembly tool on known viral reference genomes (e.g. BASV, BKV, HIV-1) and you need controlled variation in read length, sequencing error rate, coverage depth, and coverage unevenness.

## When NOT to use

- Your goal is to evaluate performance on real metagenomic samples — in silico simulation is a proof-of-concept surrogate and does not capture real sample complexity
- You need to simulate reads from non-viral or eukaryotic genomes not covered by the NCBI viral reference set used here
- You cannot install or run pIRS and only need paired-end reads without controlled error/coverage parameterization

## Inputs

- Viral reference nucleotide sequences (BASV, BKV, HIV-1) from NCBI viral reference databases (FASTA)
- pIRS simulator
- Simulation parameter grid: read lengths (100 bp, 250 bp), error rates (1%, 5%), coverage depths (10x, 20x, 50x)

## Outputs

- Per-genome/parameter FASTQ read sets (e.g. BKV_250_50_0.01_0_.fastq), with and without 1 kb 50x spike-in peaks
- Paired-end 100 bp Illumina read sets simulated by pIRS at 5% error rate
- Seed contigs for extension testing (e.g. BKV_seed_1000_867.fa)

## How to apply

Retrieve viral reference nucleotide sequences (BASV, BKV, HIV-1) from the NCBI viral reference databases. Run pIRS to simulate paired-end 100 bp Illumina reads from each genome at an error rate of 0.05, and generate additional unpaired 250 bp read sets at error rates of 1% and 5% and coverage depths of 10x, 20x, and 50x, using the naming convention exemplified by BKV_250_50_0.01_0_.fastq (read length 250, coverage 50, error 0.01). To emulate abundant-virus coverage profiles, produce replicate read sets with 50x spike-in peaks by placing 250 bp reads every 1 kb at elevated depth. Write all combinations as per-genome FASTQ files alongside seed contigs (e.g. BKV_seed_1000_867.fa), and confirm each FASTQ carries the intended read length, error rate, coverage depth, and presence/absence of the 1 kb spike-in configuration.

## Related tools

- **pIRS** (Simulates paired-end 100 bp Illumina reads from viral reference genomes at a specified error rate (0.05) for the controlled-parameterization benchmark)
- **ContigExtender** (Downstream contig-extension tool evaluated on the parameterized synthetic read sets) — https://github.com/dengzac/contig-extender

## Examples

```
./dist/extender_wrapper --coverage 50 examples/BKV_seed_1000_867.fa examples/BKV_250_50_0.01_0_.fastq
```

## Evaluation signals

- Each expected FASTQ (genome × read length × error rate × depth × spike-in/no-spike-in) exists and is non-empty
- Read lengths in each FASTQ match the simulated parameter (100 bp for pIRS paired-end sets, 250 bp for unpaired sets)
- Per-file coverage depth matches the target (10x, 20x, or 50x), verifiable by genome-size-normalized base counts
- Spike-in replicates show ~50x peaks every 1 kb when coverage is plotted along the reference genome
- Corresponding seed contig files (e.g. BKV_seed_1000_867.fa) are present and consistent with the reference genome

## Limitations

- Unevenness is modeled only as 50x peaks every 1 kb, which is a simplified emulation of real abundant-virus coverage profiles
- Only three viral genomes (BASV, BKV, HIV-1) were used; generalization to broader viral diversity is untested in this context
- Simulation parameters are restricted to the stated grid (100/250 bp; 1%/5% error; 10x/20x/50x)

## Evidence

- [methods] proof-of-concept in silico synthetic dataset parameter grid: [evidence span withheld — anchor status: not_found]
- [methods] unevenness emulation: [evidence span withheld — anchor status: not_found]
- [methods] pIRS paired-end simulation at 5% error: [evidence span withheld — anchor status: not_found]
- [readme] example simulated dataset naming: "The ```examples``` folder contains a simulated dataset from the BKV genome, with a set of reads and a seed contig"
- [readme] seed contig and example file names: "BKV_seed_1000_867.fa examples/BKV_250_50_0.01_0_.fastq"

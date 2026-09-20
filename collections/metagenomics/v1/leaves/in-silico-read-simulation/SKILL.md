---
name: in-silico-read-simulation
description: Use when benchmarking a metagenomic assembler or contig extender and
  a ground-truth-controlled read set is needed spanning known read lengths, error
  rates, coverage depths, and uneven coverage profiles.
when_to_use_negative:
- The goal is to benchmark on real metagenomic samples — this skill yields synthetic
  data with known ground truth, not biological complexity of clinical/animal metagenomes.
- The analysis target is host genome assembly or eukaryotic reference genomes — the
  article's simulation scope is viral reference genomes only.
- Reads have already been sequenced; simulation is only needed when controlled ground-truth
  read sets must be generated.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0797
tools:
- name: pIRS
  role: Simulates paired-end 100 bp Illumina reads from viral reference genomes with
    a specified base error rate (0.05 in the benchmark)
- name: ContigExtender
  role: Downstream consumer of the simulated FASTQ reads and seed contigs; the synthetic
    datasets exist to benchmark its recursive extension (r-OLC) accuracy
  repo: https://github.com/dengzac/contig-extender
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-021-04038-2
    title: 'ContigExtender: a new approach to improving de novo sequence assembly
      for viral metagenomics data'
  source_content_hash: sha256:c7766dc1530eee37577606c330aa330eb87b2de5e9057b65d22d8ead2d960a32
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dengzac/contig-extender
  tools:
  - ContigExtender
  - pIRS
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# in-silico-read-simulation

## Summary

Generate synthetic FASTQ read sets from viral reference genomes (BASV, BKV, HIV-1) with pIRS and custom simulations across read lengths (100 bp/250 bp), error rates (1%/5%), coverage depths (10x/20x/50x), and 1 kb spike-in coverage peaks to benchmark contig extension under controlled conditions.

## When to use

Use when benchmarking a metagenomic assembler or contig extender and a ground-truth-controlled read set is needed spanning known read lengths, error rates, coverage depths, and uneven coverage profiles.

## When NOT to use

- The goal is to benchmark on real metagenomic samples — this skill yields synthetic data with known ground truth, not biological complexity of clinical/animal metagenomes.
- The analysis target is host genome assembly or eukaryotic reference genomes — the article's simulation scope is viral reference genomes only.
- Reads have already been sequenced; simulation is only needed when controlled ground-truth read sets must be generated.

## Inputs

- Viral reference nucleotide sequences from NCBI (e.g., BASV, BKV, HIV-1 genomes) in FASTA format
- pIRS simulator configuration (paired-end 100 bp Illumina reads, error rate 0.05)

## Outputs

- Paired-end 100 bp FASTQ read sets per genome (pIRS, error rate 0.05)
- Unpaired 250 bp FASTQ read sets per genome/error/depth combination (e.g., BKV_250_50_0.01_0_.fastq)
- Replicate read sets with 50x spike-in peaks every 1 kb mimicking uneven coverage
- Seed contigs for extension benchmarking (e.g., BKV_seed_1000_867.fa)

## How to apply

Retrieve viral reference nucleotide sequences (e.g., BASV, BKV, HIV-1) from NCBI viral reference databases. Run pIRS to simulate paired-end 100 bp Illumina reads from each reference genome at an error rate of 0.05, and generate additional synthetic 250 bp unpaired read sets at error rates of 1% and 5% and depths of 10x, 20x, and 50x, using the naming convention exemplified by BKV_250_50_0.01_0_.fastq (read length_coverage_error). To emulate uneven coverage from abundant viruses, produce replicate read sets with 50x coverage peaks by placing 250 bp reads every 1 kb at elevated depth. Write each genome/parameter combination as a separate FASTQ file alongside a seed contig (e.g., BKV_seed_1000_867.fa) so that extension accuracy can be judged against the provided reference genome (BKV.fasta).

## Related tools

- **pIRS** (Simulates paired-end 100 bp Illumina reads from viral reference genomes with a specified base error rate (0.05 in the benchmark))
- **ContigExtender** (Downstream consumer of the simulated FASTQ reads and seed contigs; the synthetic datasets exist to benchmark its recursive extension (r-OLC) accuracy) — https://github.com/dengzac/contig-extender

## Examples

```
./dist/extender_wrapper --coverage 50 examples/BKV_seed_1000_867.fa examples/BKV_250_50_0.01_0_.fastq
```

## Evaluation signals

- Each output FASTQ exists with the specified read length (100 bp or 250 bp), error-rate setting, and coverage depth (10x/20x/50x), with and without the 1 kb spike-in peak configuration.
- File names follow the convention <genome>_<readlen>_<coverage>_<error>_<flag>.fastq (e.g., BKV_250_50_0.01_0_.fastq), so parameters are auditable from filenames.
- Reads from the 50x spike-in replicate sets show elevated local depth at ~1 kb intervals when mapped back to the reference genome.
- Simulated reads align back to their source viral reference genome (e.g., BKV.fasta) at the intended error rate, allowing extension accuracy to be scored against ground truth.
- Seed contigs such as BKV_seed_1000_867.fa are present and correspond to the simulated genome so extension output can be compared to the reference.

## Limitations

- Spike-in peak configuration is a coarse model of uneven coverage (250 bp reads every 1 kb at 50x), which may not capture real abundance variation in metagenomes.
- Only Illumina-style reads at the tested combinations (100/250 bp, 1%/5% error, 10x/20x/50x depth) were simulated; other platforms or parameter ranges are out of scope.
- Simulated datasets lack biological complexity (no host background or community mixture), so results may not transfer to real metagenomic data.

## Evidence

- [results] in silico synthetic dataset parameters: [evidence span withheld — anchor status: not_found]
- [results] spike-in coverage peaks: [evidence span withheld — anchor status: not_found]
- [methods] pIRS simulation: [evidence span withheld — anchor status: not_found]
- [readme] BKV example dataset in repo: "The ```examples``` folder contains a simulated dataset from the BKV genome, with a set of reads and a seed contig"
- [readme] reference genome for accuracy check: "To verify the accuracy of the extended contig, the reference genome is provided in ```BKV.fasta```"
- [discussion] simulated reads used in evaluation: "Supplementary Table S2 shows contig lengths produced by ContigExtender on simulated paired-end reads from pIRS"

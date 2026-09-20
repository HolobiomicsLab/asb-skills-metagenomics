---
name: spike-in-coverage-peak-library-construction
description: Use when benchmarking contig extension or de novo assembly tools on viral
  metagenomic data and you need a controlled in silico ground truth with uneven, spike-like
  coverage profiles mimicking abundant-virus read distributions.
when_to_use_negative:
- The goal is real metagenomic (non-simulated) analysis — this skill only produces
  synthetic ground-truth datasets for benchmarking.
- Platform is not Illumina-like short reads; pIRS simulates paired-end Illumina reads
  and the 100/250 bp parameterization assumes short-read chemistry.
- You need realistic coverage unevenness with quantitative community abundances rather
  than the stylized 50x-every-1kb spike pattern used here.
edam_operation: http://edamontology.org/operation_0304
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: pIRS
  role: Generates paired-end 100 bp Illumina reads from each viral reference genome
    at error rate 0.05, forming the uniform background of the simulated datasets
- name: ContigExtender
  role: Downstream consumer used to validate that the simulated spike-in libraries
    actually support contig extension (e.g. extending BKV_seed_1000_867.fa)
  repo: https://github.com/dengzac/contig-extender
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-021-04038-2
    title: 'ContigExtender: a new approach to improving de novo sequence assembly
      for viral metagenomics data'
  source_content_hash: sha256:d94a4b88cc0ca8d8d559c874e2bbcd1736e0915475d5d339459c4b8899c6311b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dengzac/contig-extender
  tools:
  - ContigExtender
  - pIRS
  edam_operation: http://edamontology.org/operation_0304
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# spike-in-coverage-peak-library-construction

## Summary

Construct in silico viral metagenome read libraries with deliberate coverage unevenness — 250 bp spike-in reads placed every 1 kb at ~50x depth on top of uniform simulated backgrounds — plus factorial variation in read length (100/250 bp), error rate (1%/5%), and depth (10x/20x/50x), so that contig-extension tools like ContigExtender can be benchmarked on realistic uneven-coverage data.

## When to use

Use when benchmarking contig extension or de novo assembly tools on viral metagenomic data and you need a controlled in silico ground truth with uneven, spike-like coverage profiles mimicking abundant-virus read distributions.

## When NOT to use

- The goal is real metagenomic (non-simulated) analysis — this skill only produces synthetic ground-truth datasets for benchmarking.
- Platform is not Illumina-like short reads; pIRS simulates paired-end Illumina reads and the 100/250 bp parameterization assumes short-read chemistry.
- You need realistic coverage unevenness with quantitative community abundances rather than the stylized 50x-every-1kb spike pattern used here.

## Inputs

- Viral reference nucleotide sequences (FASTA) from NCBI viral reference databases (BASV, BKV, HIV-1)
- Simulation parameter choices: read length (100 bp or 250 bp), error rate (1% or 5%, or 0.05 for pIRS paired-end), coverage depth (10x/20x/50x)
- Spike-in configuration: 250 bp reads placed every 1 kb at ~50x coverage

## Outputs

- Per-genome simulated FASTQ read sets (e.g. BKV_250_50_0.01_0_.fastq) for each length/error/depth combination
- Spike-in variant FASTQs with 1 kb-interval 50x coverage peaks
- Seed contig FASTA files for extension testing (e.g. BKV_seed_1000_867.fa)
- Reference genome FASTA retained for validating extended-contig accuracy (e.g. BKV.fasta)

## How to apply

Retrieve viral reference nucleotide sequences (e.g. BASV, BKV, HIV-1) from the NCBI viral reference databases, then run pIRS to simulate paired-end 100 bp Illumina reads per genome at an error rate of 0.05. Separately generate unpaired 250 bp read sets at error rates of 1% and 5% and coverage depths of 10x, 20x, and 50x, using the naming convention exemplified by BKV_250_50_0.01_0_.fastq (genome_read-length_coverage_error_0). For each genome, build the unevenness configuration by placing 250 bp reads every 1 kb at elevated (~50x) depth to emulate abundant-virus coverage peaks, and export every genome/parameter combination as its own FASTQ. Also produce the seed contig files (e.g. BKV_seed_1000_867.fa) that will serve as extension inputs, and keep the reference genome (e.g. BKV.fasta) alongside for accuracy checking.

## Related tools

- **pIRS** (Generates paired-end 100 bp Illumina reads from each viral reference genome at error rate 0.05, forming the uniform background of the simulated datasets)
- **ContigExtender** (Downstream consumer used to validate that the simulated spike-in libraries actually support contig extension (e.g. extending BKV_seed_1000_867.fa)) — https://github.com/dengzac/contig-extender

## Examples

```
./dist/extender_wrapper --coverage 50 examples/BKV_seed_1000_867.fa examples/BKV_250_50_0.01_0_.fastq
```

## Evaluation signals

- Every genome/parameter FASTQ exists with the specified read length (100 bp or 250 bp), error-rate setting, and depth (10x/20x/50x), with and without the 1 kb spike-in configuration.
- Spike-in libraries show ~50x coverage peaks at 250 bp intervals of 1 kb across each reference genome, versus flat 10x/20x/50x backgrounds.
- Each output FASTQ is accompanied by a seed contig FASTA (e.g. BKV_seed_1000_867.fa) usable as extension input.
- Filename convention holds: GENOME_length_coverage_error_0_.fastq (e.g. BKV_250_50_0.01_0_.fastq encodes 250 bp, 50x, 1% error).
- Extended contigs from the simulated reads align to the retained reference genome (e.g. BKV.fasta), confirming read simulation fidelity.

## Limitations

- Simulations cover only a few viral reference genomes (BASV, BKV, HIV-1), so results may not generalize to broader viral diversity.
- The 50x-spike-every-1-kb pattern is a stylized proxy for abundant-virus coverage profiles, not an empirically derived distribution.
- High-error settings (5%) and forced extension parameters increase the chance of misassembly/false extension in downstream evaluation (per README, raising --extend-tolerance above 2.5 'risks higher chances of false extension').

## Evidence

- [discussion] Supplementary Table S2 shows contig lengths produced by ContigExtender on simulated paired-end reads from pIRS: "Supplementary Table S2 shows contig lengths produced by ContigExtender on simulated paired-end reads from pIRS"
- [discussion] in silico reads of the BKV and HIV viruses: "in silico reads of the BKV and HIV viruses"
- [methods] The ```examples``` folder contains a simulated dataset from the BKV genome, with a set of reads and a seed contig: "The ```examples``` folder contains a simulated dataset from the BKV genome, with a set of reads and a seed contig"
- [readme] To verify the accuracy of the extended contig, the reference genome is provided in ```BKV.fasta```.: "To verify the accuracy of the extended contig, the reference genome is provided in ```BKV.fasta```."

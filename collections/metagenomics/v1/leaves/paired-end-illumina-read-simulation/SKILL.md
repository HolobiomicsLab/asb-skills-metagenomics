---
name: paired-end-illumina-read-simulation
description: Use when building synthetic FASTQ benchmarks from known viral reference
  genomes (e.g. BASV, BKV, HIV-1) to evaluate contig extension or assembly accuracy
  under varied read length, error rate, and depth conditions.
when_to_use_negative:
- Your goal is evaluating on real metagenomics data — the article separately benchmarks
  ContigExtender on real animal and human metagenomics datasets, so synthetic reads
  are only needed for controlled parameter sweeps
- You need realistic background-community complexity — these simulations come from
  single reference genomes and cannot model mixed viral populations or host contamination
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0781
tools:
- name: pIRS
  role: Paired-end Illumina read simulator used to generate 100 bp synthetic reads
    at controlled error rate and depth from each viral reference genome
- name: ContigExtender
  role: Downstream consumer of the simulated FASTQ reads and seed contigs; extension
    accuracy against the known reference validates the simulation setup
  repo: https://github.com/dengzac/contig-extender
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-021-04038-2
    title: 'ContigExtender: a new approach to improving de novo sequence assembly
      for viral metagenomics data'
  source_content_hash: sha256:cd9a0ab4fbc431753e6b98e61cb995e777d98af7a1b7aa4de36e471701378fc7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dengzac/contig-extender
  tools:
  - ContigExtender
  - pIRS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0781
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# paired-end-illumina-read-simulation

## Summary

Generates in silico synthetic Illumina FASTQ read sets with pIRS to benchmark ContigExtender under controlled read lengths (100 bp and 250 bp), error rates (1% and 5%), coverage depths (10x/20x/50x), and uneven-coverage spike-in peaks (50x 250 bp reads every 1 kb).

## When to use

Use when building synthetic FASTQ benchmarks from known viral reference genomes (e.g. BASV, BKV, HIV-1) to evaluate contig extension or assembly accuracy under varied read length, error rate, and depth conditions.

## When NOT to use

- Your goal is evaluating on real metagenomics data — the article separately benchmarks ContigExtender on real animal and human metagenomics datasets, so synthetic reads are only needed for controlled parameter sweeps
- You need realistic background-community complexity — these simulations come from single reference genomes and cannot model mixed viral populations or host contamination

## Inputs

- Viral reference nucleotide sequences in FASTA (BASV, BKV, HIV-1 from NCBI viral reference databases)
- Simulation parameters: read length (100 bp or 250 bp), error rate (0.01 or 0.05), coverage depth (10x/20x/50x)
- Spike-in configuration for uneven coverage: 250 bp reads placed every 1 kb at 50x depth

## Outputs

- Paired-end 100 bp Illumina FASTQ files simulated with pIRS at error rate 0.05
- Unpaired 250 bp FASTQ files per genome/parameter combination, named per convention (e.g. BKV_250_50_0.01_0_.fastq)
- Replicate FASTQ files with 1 kb spike-in 50x coverage peaks
- Seed contig FASTA files (e.g. BKV_seed_1000_867.fa) for downstream extension testing

## How to apply

Retrieve viral reference nucleotide sequences (BASV, BKV, HIV-1) from NCBI viral reference databases, then run pIRS to simulate paired-end 100 bp Illumina reads (error rate 0.05) from each genome. Generate additional unpaired 250 bp read sets at error rates of 1% and 5% and depths of 10x, 20x, and 50x, using the naming convention exemplified by BKV_250_50_0.01_0_.fastq (read length 250, coverage 50, error 0.01). To emulate abundant-virus unevenness, produce replicate sets with 50x coverage spike-in peaks by placing 250 bp reads every 1 kb at elevated depth. Write each genome/parameter combination as a separate FASTQ alongside a seed contig (e.g. BKV_seed_1000_867.fa). Because the true reference genome is known (e.g. BKV.fasta), simulated reads support exact accuracy verification of extended contigs via nucleotide BLAST against the reference.

## Related tools

- **pIRS** (Paired-end Illumina read simulator used to generate 100 bp synthetic reads at controlled error rate and depth from each viral reference genome)
- **ContigExtender** (Downstream consumer of the simulated FASTQ reads and seed contigs; extension accuracy against the known reference validates the simulation setup) — https://github.com/dengzac/contig-extender

## Examples

```
./dist/extender_wrapper --coverage 50 examples/BKV_seed_1000_867.fa examples/BKV_250_50_0.01_0_.fastq
```

## Evaluation signals

- Each output FASTQ exists for every genome/parameter combination: read length (100 bp or 250 bp), error rate (0.01/0.05), and depth (10x/20x/50x), with and without spike-in peaks
- Filename encodes the simulation parameters correctly per the BKV_250_50_0.01_0_.fastq convention (length, coverage, error)
- Spike-in variant shows 50x coverage peaks at ~1 kb intervals across the reference when reads are re-aligned
- A seed contig FASTA (e.g. BKV_seed_1000_867.fa) accompanies each simulated read set
- Extended contigs produced from simulated reads can be verified against the known reference genome (BKV.fasta) via nucleotide BLAST

## Limitations

- Simulated reads derive from single reference genomes, so they do not capture real metagenomic community complexity
- Spike-in unevenness only models the abundant-virus profile (50x peaks every 1 kb), not other uneven coverage patterns
- Simulated-read results are reported in supplementary material (Supplementary Table S2 for pIRS simulated paired-end reads) not fully included in the article, limiting reproducibility of exact numbers

## Evidence

- [discussion] in silico reads of the BKV and HIV viruses: "in silico reads of the BKV and HIV viruses"
- [readme] The examples folder contains a simulated dataset from the BKV genome: "The ```examples``` folder contains a simulated dataset from the BKV genome, with a set of reads and a seed contig"
- [discussion] Supplementary Table S2 shows contig lengths produced by ContigExtender on simulated paired-end reads from pIRS: "Supplementary Table S2 shows contig lengths produced by ContigExtender on simulated paired-end reads from pIRS"
- [readme] To verify the accuracy of the extended contig, the reference genome is provided in BKV.fasta: "To verify the accuracy of the extended contig, the reference genome is provided in ```BKV.fasta```."

---
name: mycobacterial-genome-read-simulation
description: 'Use when benchmarking host read-removal tools on known-composition microbial
  data: you need simulated bacterial reads with a ground truth so that false-removal
  (retention) percentages of Hostile versus HRRT can be measured exactly.'
when_to_use_negative:
- You want to measure human read removal sensitivity on real human data — use 1000
  Genomes Project samples instead of simulated bacterial reads
- Your goal is microbial taxonomic profiling or assembly, not decontamination benchmarking
- You lack the ground-truth assemblies, so retention percentages cannot be computed
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: DWGSIM
  role: Simulates paired Illumina short reads from each of the 140 mycobacterial assemblies
- name: PBSIM2
  role: Simulates Oxford Nanopore long reads from the same assemblies
- name: Hostile
  role: Decontaminates simulated reads (Bowtie2 mode for Illumina, Minimap2 mode for
    ONT) against human-t2t-hla; retention computed as reads_out/reads_in
  repo: https://github.com/bede/hostile
- name: HRRT
  role: NCBI human read removal tool run on both simulated read sets as the comparison
    baseline
  repo: https://github.com/ncbi/sra-human-scrubber
- name: Bowtie2
  role: Alignment engine used by Hostile for short-read decontamination
- name: Minimap2
  role: Alignment engine used by Hostile for long-read decontamination
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad728
    title: 'Hostile: accurate decontamination of microbial host sequences'
  source_content_hash: sha256:29c4836c3e7f79c7388e4848a2842d2eb5e5bcf00ef326dfb71bf81f2ca9dd90
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bede/hostile
  tools:
  - Bowtie2
  - DWGSIM
  - HRRT
  - Hostile
  - Minimap2
  - PBSIM2
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# mycobacterial-genome-read-simulation

## Summary

Simulate paired Illumina short reads and Oxford Nanopore long reads from complete mycobacterial assemblies (DWGSIM and PBSIM2) to benchmark how much host-decontamination tools such as Hostile and HRRT remove non-host microbial reads.

## When to use

Use when benchmarking host read-removal tools on known-composition microbial data: you need simulated bacterial reads with a ground truth so that false-removal (retention) percentages of Hostile versus HRRT can be measured exactly.

## When NOT to use

- You want to measure human read removal sensitivity on real human data — use 1000 Genomes Project samples instead of simulated bacterial reads
- Your goal is microbial taxonomic profiling or assembly, not decontamination benchmarking
- You lack the ground-truth assemblies, so retention percentages cannot be computed

## Inputs

- 140 complete mycobacterial genome assemblies (FASTA)
- Human reference index for decontamination: human-t2t-hla (T2T-CHM13v2.0 + IPD-IMGT/HLA v3.51)
- HRRT pre-built human k-mer database downloaded via init_db.sh

## Outputs

- Simulated paired Illumina FASTQ files (DWGSIM)
- Simulated ONT long-read FASTQ files (PBSIM2)
- Cleaned FASTQ files per tool per platform
- Comparison table of four retention percentages: Hostile vs HRRT on Illumina and ONT

## How to apply

Simulate paired Illumina short reads from each of 140 complete mycobacterial assemblies with DWGSIM and Oxford Nanopore long reads from the same assemblies with PBSIM2, yielding read sets whose true origin is known. Decontaminate the simulated Illumina reads with Hostile in Bowtie2 mode against the default human-t2t-hla index (T2T-CHM13v2.0 + IPD-IMGT/HLA v3.51), and the simulated ONT reads with Hostile in Minimap2 mode against the same index, computing retention as reads_out relative to reads_in. Run HRRT (ncbi/sra-human-scrubber) on both read sets in parallel and compute the corresponding retained proportions. Tabulate the four retention values (Hostile vs HRRT, Illumina vs ONT) into a single comparison table. The point is to quantify false positives: Hostile is precise by default and should retain ≥99.989% of simulated bacterial reads, whereas HRRT's k-mer database (Eukaryota k-mers minus non-Eukaryota RefSeq k-mers) tends to be more aggressive.

## Related tools

- **DWGSIM** (Simulates paired Illumina short reads from each of the 140 mycobacterial assemblies)
- **PBSIM2** (Simulates Oxford Nanopore long reads from the same assemblies)
- **Hostile** (Decontaminates simulated reads (Bowtie2 mode for Illumina, Minimap2 mode for ONT) against human-t2t-hla; retention computed as reads_out/reads_in) — https://github.com/bede/hostile
- **HRRT** (NCBI human read removal tool run on both simulated read sets as the comparison baseline) — https://github.com/ncbi/sra-human-scrubber
- **Bowtie2** (Alignment engine used by Hostile for short-read decontamination)
- **Minimap2** (Alignment engine used by Hostile for long-read decontamination)

## Examples

```
hostile clean --fastq1 reads_1.fastq.gz --fastq2 reads_2.fastq.gz --index human-t2t-hla --threads 8
```

## Evaluation signals

- Hostile Illumina retention reproduces ~99.99998% and ONT retention ~99.99948% of the reported figures
- Hostile retention meets the abstract-level guarantee of at least 99.989% of simulated bacterial reads
- HRRT removes roughly 15 times more mycobacterial reads than Hostile (99.99864% ONT retention, 99.999751% Illumina retention)
- Retention is computed consistently as reads_out relative to reads_in for every tool/platform combination

## Limitations

- HRRT tends to be aggressive about identifying human reads since its database contains k-mers common to primates and other taxa, at the cost of removing more bacterial reads
- HRRT masks identified human reads with 'N' rather than removing them, so output read counts differ from removal-based tools
- Simulation does not capture real-world read error and adapter profiles perfectly; results apply to the simulated read models used (DWGSIM, PBSIM2)

## Evidence

- [abstract] Hostile retains simulated bacterial reads at near-identity level: "removes at least 99.6% of real human reads and retains at least 99.989% of simulated bacterial reads"
- [methods] Default index composition: "The default index `human-t2t-hla` comprises [T2T-CHM13v2.0](https://www.ncbi.nlm.nih.gov/assembly/11828891) and [IPD-IMGT/HLA](https://www.ebi.ac.uk/ipd/imgt/hla/) v3.51"
- [intro] Hostile operates on FASTQ streams: "Hostile removes host sequences from short and long read (meta)genomes, consuming single or paired FASTQ from files or stdin."
- [readme] HRRT database design makes it aggressive: [evidence span withheld — anchor status: external_doc]
- [readme] HRRT output behavior: [evidence span withheld — anchor status: external_doc]

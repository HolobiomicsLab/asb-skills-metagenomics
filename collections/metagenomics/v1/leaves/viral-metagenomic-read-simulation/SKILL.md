---
name: viral-metagenomic-read-simulation
description: Use when benchmarking a viral read-recruitment or haplotype-assembly
  pipeline and a ground-truth simulated metagenome with known haplotype counts and
  coverage is needed.
when_to_use_negative:
- The analysis targets real clinical or environmental metagenomic data rather than
  a controlled benchmark with known ground truth
- The sequencer/platform differs from MiSeq paired-end 250 bp chemistry and ART-illumina
  error models would misrepresent it
- A completely new virus sharing no conservation with any reference genome must be
  detected, since simulation from known references cannot represent it
edam_operation: http://edamontology.org/operation_0242
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: ART-illumina
  role: Generates the error-containing MiSeq paired-end simulated reads from the mutated
    haplotype and decoy genomes
- name: Karect
  role: Corrects sequencing errors in the simulated reads so exact overlap detection
    in TAR-VIR works
  repo: https://github.com/aminallam/karect
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1186/s12859-019-2878-2
    title: 'TAR-VIR: a pipeline for TARgeted VIRal strain reconstruction from metagenomic
      data'
  source_content_hash: sha256:2822b0233b02f7ecfdf07b2529b64d4aeba054ed3f5c269c8e7ecb0f2006d4c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aminallam/karect
  tools:
  - ART-illumina
  - BWA
  - Karect
  - TAR-VIR
  edam_operation: http://edamontology.org/operation_0242
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# viral-metagenomic-read-simulation

## Summary

Simulate error-containing MiSeq paired-end viral metagenomic read sets with ART-illumina so that downstream recruitment/assembly pipelines (e.g. TAR-VIR) can be benchmarked against a known ground-truth composition of haplotypes and off-target viruses.

## When to use

Use when benchmarking a viral read-recruitment or haplotype-assembly pipeline and a ground-truth simulated metagenome with known haplotype counts and coverage is needed.

## When NOT to use

- The analysis targets real clinical or environmental metagenomic data rather than a controlled benchmark with known ground truth
- The sequencer/platform differs from MiSeq paired-end 250 bp chemistry and ART-illumina error models would misrepresent it
- A completely new virus sharing no conservation with any reference genome must be detected, since simulation from known references cannot represent it

## Inputs

- Reference genome FASTA(s) for the target virus (e.g. SARS-CoV NC_004718.3) for haplotype construction
- Decoy viral genome FASTAs for specificity testing (e.g. Influenza NC_002023.1, HCV NC_004102.1)
- ART-illumina MiSeq paired-end profile settings (read length 250 bp, insert 600 bp, SD 150 bp)

## Outputs

- Paired-end FASTQ read files simulating the viral metagenomic sample (173,703 reads in the reference study)
- Ground-truth mapping of reads to source haplotypes (119,002 SARS-CoV-derived reads) enabling per-haplotype coverage and contamination scoring

## How to apply

Create the target haplotypes by mutating bases at randomly selected locations in a reference genome (here 5 SARS-CoV haplotypes from NC_004718.3) and mix them with decoy viral genomes (Influenza NC_002023.1, HCV NC_004102.1) to test specificity. Run ART-illumina in error-containing MiSeq paired-end mode with 250 bp reads, 600 bp mean insert size, and 150 bp SD, scaled to yield a defined total (here 173,703 reads, of which 119,002 derive from the 5 SARS-CoV haplotypes). Retain the read-to-genome truth mapping so per-haplotype recruited counts and genome coverage can be scored downstream. Optionally include a closely related but distinct reference (bat coronavirus NC_014470.1) to mimic a remotely related seed-alignment reference.

## Related tools

- **ART-illumina** (Generates the error-containing MiSeq paired-end simulated reads from the mutated haplotype and decoy genomes)
- **Karect** (Corrects sequencing errors in the simulated reads so exact overlap detection in TAR-VIR works) — https://github.com/aminallam/karect

## Evaluation signals

- Total simulated read count matches the planned design (173,703 reads in the reference study)
- Per-source read proportions match design: 119,002 of 173,703 reads derive from the five SARS-CoV haplotypes
- Read length and insert statistics reflect the MiSeq settings used (250 bp reads, 600 bp mean insert, 150 bp SD)
- Downstream recruitment on the simulated data yields no reads from decoy genomes (no Influenza or HCV contamination in recruited reads)
- Mutated haplotypes differ from the NC_004718.3 reference only at the intended random mutation positions

## Limitations

- Simulation from known references cannot capture a completely new virus that shares no conservation with any reference genome, which such a pipeline would not detect
- ART-illumina's error model may not perfectly reflect real MiSeq error profiles, so benchmark results may overstate performance on real data
- Haplotypes are generated by randomly located base mutations, which may not reflect the real mutation spectrum of fast-mutating RNA viruses

## Evidence

- [results] Simulated MiSeq paired-end reads with ART-illumina (250 bp reads, 600 bp insert, 150 bp SD): "simulated by ART-illumina [24] as error-containing MiSeq paired-end reads, with the read length of 250 bp, the average insert size of 600 bp"
- [results] Simulated dataset composition: "In total, there are 173,703 simulated reads, of which 119,002 reads are from the five SARS-CoV haplotypes."
- [results] Haplotype construction by random mutation: "The SARS-CoV haplotypes were created from the SARS-CoV reference (NC_004718.3) genome by mutating bases at randomly selected locations."
- [readme] Simulated test data in the repository: "This data set contains simulated viral reads from HIV-1, HCV genotype 1, and HGV."

---
name: selection-trajectory-parameter-configuration
description: Use when simulating longitudinal metagenomic samples that require injected
  within-strain mutations to follow prescribed per-timepoint allele-frequency trajectories
  emulating positive or purifying selection with STEMSim.
when_to_use_negative:
- Real metagenomic data on low-depth or poorly characterized species — STEMSim only
  works well on species with high depth and well-characterized genomes
- Static single-timepoint communities without a longitudinal design, since trajectories
  are defined per timepoint across samples
- When the goal is simulating de novo community abundance rather than within-strain
  mutation injection — use CAMISIM alone
tools:
- name: STEMSim
  role: Simulator that injects mutations and realizes configured allele frequencies
    via Bernoulli read-carrier selection
  repo: https://github.com/BoyanZhou/STEMSim
- name: CAMISIM
  role: Optional generator of simulated microbial-community raw reads used as input
    for camisim mode
  repo: https://github.com/CAMI-challenge/CAMISIM
- name: ART
  role: Optional generator of simulated raw reads from a reference genome used as
    input for reads mode
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad302
    title: 'STEMSIM: a simulator of within-strain short-term evolutionary mutations
      for longitudinal metagenomic data'
  source_content_hash: sha256:9d2fc80510ba36d8e7ceaa0269b7f751ab589deae9aea6e5d147f457689d5af8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BoyanZhou/STEMSim
  tools:
  - ART
  - CAMISIM
  - Python
  - STEMSim
  - numpy
  - samtools
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# selection-trajectory-parameter-configuration

## Summary

Configure STEMSim config.txt files to specify allele-frequency trajectories of injected within-strain mutations across longitudinal timepoints (e.g., increasing 0.2/0.5/0.8/1 for positive selection; decreasing 0.8/0.5/0.3/0 for purifying selection), so that downstream metagenomic reads carry mutations at the intended frequencies.

## When to use

Use when simulating longitudinal metagenomic samples that require injected within-strain mutations to follow prescribed per-timepoint allele-frequency trajectories emulating positive or purifying selection with STEMSim.

## When NOT to use

- Real metagenomic data on low-depth or poorly characterized species — STEMSim only works well on species with high depth and well-characterized genomes
- Static single-timepoint communities without a longitudinal design, since trajectories are defined per timepoint across samples
- When the goal is simulating de novo community abundance rather than within-strain mutation injection — use CAMISIM alone

## Inputs

- STEMSim config.txt file defining longitudinal allele-frequency trajectories across timepoints
- Longitudinal simulated raw reads from CAMISIM (output directory + genome_to_id.tsv) or from ART (FASTQ files)
- Reference genome FASTA (.fas) and genome ID (for reads mode)

## Outputs

- Modified reads with within-strain evolutionary mutations at the configured timepoint frequencies
- VCF of true (injected) mutations with observed per-timepoint allele frequencies
- STEMSim log file

## How to apply

Prepare a STEMSim config file (template config.txt) that defines the mutation-frequency trajectory across longitudinal timepoints, such as an increasing trajectory (0.2/0.5/0.8/1) emulating positive selection or a decreasing trajectory (0.8/0.5/0.3/0) emulating purifying selection. Generate simulated longitudinal raw reads with CAMISIM (camisim mode) or ART (reads mode), then run stemsim.py, which realizes each configured frequency by randomly selecting the reads carrying each mutation via a Bernoulli process at each timepoint. Choose the mode based on input: 'camisim' takes a CAMISIM output directory plus genome_to_id.tsv, while 'reads' takes per-sample FASTQs (paired files comma-separated within a sample, samples colon-separated) plus a reference genome and genome ID. Parameter details for config.txt are documented in the STEMSim wiki.

## Related tools

- **STEMSim** (Simulator that injects mutations and realizes configured allele frequencies via Bernoulli read-carrier selection) — https://github.com/BoyanZhou/STEMSim
- **CAMISIM** (Optional generator of simulated microbial-community raw reads used as input for camisim mode) — https://github.com/CAMI-challenge/CAMISIM
- **ART** (Optional generator of simulated raw reads from a reference genome used as input for reads mode)

## Examples

```
python stemsim.py -m camisim -f PATH/config.txt -s test_subject1 --input_from_camisim PATH_to_output_directory_of_CAMISIM/ --genome_to_id PATH_to_genome_to_id/genome_to_id.tsv -o PATH_to_output_directory_of_STEMSim/
```

## Evaluation signals

- Observed per-timepoint allele frequencies extracted from the VCF of true mutations should track the configured trajectories (e.g., rise from 0.2 to 1 for the positive-selection case, fall from 0.8 to 0 for the purifying-selection case)
- Joined table observed_vs_specified_allele_frequencies.tsv should contain one row per mutation per timepoint with both specified and observed frequencies
- Read-level support recovered via Bowtie2 alignment and samtools/pysam at injected sites should be consistent with Bernoulli sampling at the configured frequencies
- STEMSim log file should confirm the mode, subject ID, and config file used for each run

## Limitations

- STEMSim is designed for simulated longitudinal raw sequencing reads; on real metagenomic data it only works well on species with high depth and well-characterized genomes
- CAMISIM is required only for camisim mode, and the authors recommend CAMISIM output as input because it is a well-established community-simulation pipeline

## Evidence

- [other] STEMSim realizes specified allele frequencies by randomly selecting the reads that carry each mutation via a Bernoulli process: [evidence span withheld — anchor status: not_found]
- [abstract] STEMSim outputs modified reads with within-strain evolutionary mutations and the relevant information of these mutations.: [evidence span withheld — anchor status: not_found]
- [methods] stemsim takes the output directory of CAMISIM as input directory, and generate longitudinal mutations on the simulated raw sequencing data: "stemsim takes the output directory of CAMISIM as input directory, and generate longitudinal mutations on the simulated raw sequencing data"
- [readme] The tutorial of parameter setting can be found https://github.com/BoyanZhou/STEMSim/wiki: "The tutorial of parameter setting can be found https://github.com/BoyanZhou/STEMSim/wiki"
- [readme] Although it can also works on real metagenomic data, it only works well on species with high depth and well-characterized genomes.: "Although it can also works on real metagenomic data, it only works well on species with high depth and well-characterized genomes."

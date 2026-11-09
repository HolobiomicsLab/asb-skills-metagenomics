---
name: bernoulli-read-carrier-mutation-injection
description: Use when you need simulated longitudinal metagenomic reads carrying within-strain
  mutations with user-specified allele-frequency trajectories across timepoints, to
  benchmark mutation-detection tools.
when_to_use_negative:
- Your input is real metagenomic data from species with low sequencing depth or poorly
  characterized genomes — the README states STEMSim 'only works well on species with
  high depth and well-characterized genomes'
- You need between-strain or long-term evolutionary divergence rather than within-strain
  short-term evolutionary mutations in longitudinal samples
- Your raw reads are not simulated reads from a single reference genome or CAMISIM
  community output (the two supported input modes)
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: STEMSim
  role: 'Core simulator: injects mutations and selects read carriers via Bernoulli
    sampling at configured timepoint frequencies; emits mutated reads and true-mutation
    VCF'
  repo: https://github.com/BoyanZhou/STEMSim
- name: CAMISIM
  role: Optional generator of longitudinal simulated microbial community raw reads
    used as input in 'camisim' mode
  repo: https://github.com/CAMI-challenge/CAMISIM
- name: ART
  role: Optional generator of simulated raw reads from one reference genome used as
    input in 'reads' mode
- name: Bowtie2
  role: Aligns mutated reads to the reference genome to recover read-level support
    at each injected site
- name: samtools
  role: Processes Bowtie2 alignments (with pysam) to tally read-level support at injected
    sites per timepoint
- name: pysam
  role: Python library used alongside samtools to parse alignments and quantify read
    support at each injected mutation site
- name: SnpEff
  role: Optional annotation of the VCF output of true mutations
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad302
    title: 'STEMSIM: a simulator of within-strain short-term evolutionary mutations
      for longitudinal metagenomic data'
  source_content_hash: sha256:bed7caf7d61c1d91e1c64f6e0b5fe448a13c565240aba74a548bd134da5bcf78
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BoyanZhou/STEMSim
  tools:
  - ART
  - Bowtie2
  - CAMISIM
  - Python
  - STEMSim
  - SnpEff
  - numpy
  - pysam
  - samtools
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# bernoulli-read-carrier-mutation-injection

## Summary

A STEMSIM-based method for injecting within-strain short-term evolutionary mutations into longitudinal metagenomic reads by Bernoulli-sampling the read carriers at each configured per-timepoint allele frequency, so observed trajectories match specified ones (e.g., increasing 0.2/0.5/0.8/1 under positive selection, decreasing 0.8/0.5/0.3/0 under purifying selection).

## When to use

Use when you need simulated longitudinal metagenomic reads carrying within-strain mutations with user-specified allele-frequency trajectories across timepoints, to benchmark mutation-detection tools.

## When NOT to use

- Your input is real metagenomic data from species with low sequencing depth or poorly characterized genomes — the README states STEMSim 'only works well on species with high depth and well-characterized genomes'
- You need between-strain or long-term evolutionary divergence rather than within-strain short-term evolutionary mutations in longitudinal samples
- Your raw reads are not simulated reads from a single reference genome or CAMISIM community output (the two supported input modes)

## Inputs

- STEMSim config file (template config.txt) specifying per-timepoint mutation allele-frequency trajectories
- CAMISIM output directory plus genome_to_id.tsv (camisim mode), each line a genome_id and absolute path to its fasta separated by Tab
- Longitudinal raw read FASTQ files from another simulator such as ART (reads mode), paired reads separated by ',' within samples and samples separated by ':'
- Reference genome FASTA (.fas) from which raw reads were simulated, with a genome_id

## Outputs

- Modified reads carrying within-strain evolutionary mutations per longitudinal timepoint
- VCF of true mutations with per-timepoint observed allele frequencies for each injected mutation
- Combined table joining specified vs observed allele frequencies per mutation per timepoint (observed_vs_specified_allele_frequencies.tsv)

## How to apply

Prepare two STEMSim config files (template config.txt) defining longitudinal mutation-frequency trajectories across four timepoints: an increasing trajectory (0.2/0.5/0.8/1) emulating positive selection and a decreasing trajectory (0.8/0.5/0.3/0) emulating purifying selection. Generate raw reads per trajectory either with CAMISIM (camisim mode, supplying the CAMISIM output directory and genome_to_id.tsv) or with ART (reads mode, supplying paired/single FASTQs separated by ',' within samples and ':' across samples, plus --reference_genome and --genome_id). Run 'python stemsim.py -m camisim|reads -f config.txt -s subject_id ...', which selects read carriers for each injected mutation via Bernoulli sampling at the configured timepoint frequencies. Align mutated reads to the reference with Bowtie2, process alignments with samtools/pysam to recover read-level support at each injected site, and extract observed per-timepoint allele frequencies from the VCF of true mutations emitted by STEMSim. Finally, join specified trajectory values with observed frequencies per mutation per timepoint (observed_vs_specified_allele_frequencies.tsv) to confirm the Bernoulli carrier selection realized the configured frequencies.

## Related tools

- **STEMSim** (Core simulator: injects mutations and selects read carriers via Bernoulli sampling at configured timepoint frequencies; emits mutated reads and true-mutation VCF) — https://github.com/BoyanZhou/STEMSim
- **CAMISIM** (Optional generator of longitudinal simulated microbial community raw reads used as input in 'camisim' mode) — https://github.com/CAMI-challenge/CAMISIM
- **ART** (Optional generator of simulated raw reads from one reference genome used as input in 'reads' mode)
- **Bowtie2** (Aligns mutated reads to the reference genome to recover read-level support at each injected site)
- **samtools** (Processes Bowtie2 alignments (with pysam) to tally read-level support at injected sites per timepoint)
- **pysam** (Python library used alongside samtools to parse alignments and quantify read support at each injected mutation site)
- **SnpEff** (Optional annotation of the VCF output of true mutations)

## Examples

```
python /path_to_stemsim_directory/stemsim.py -m reads -f PATH/config.txt -s test_subject1 --input_reads PATH/sample0_R1.fq,PATH/sample0_R2.fq:PATH/sample1.fq:PATH/sample2.fq -o PATH_to_output_directory_of_STEMSim/ --reference_genome PATH/species0_strain0.fas --genome_id species0_strain0 -l Path/stemsim_test1_log
```

## Evaluation signals

- Observed per-timepoint allele frequencies extracted from the STEMSim true-mutation VCF match the configured trajectory values (e.g., 0.2/0.5/0.8/1 for the positive-selection trajectory) within expected sampling noise
- The combined observed_vs_specified_allele_frequencies.tsv contains one row per mutation per timepoint with both specified and observed frequencies populated
- Every injected mutation site shows non-zero read-level support in the Bowtie2/samtools alignments at timepoints where the configured frequency is > 0
- The decreasing trajectory (0.8/0.5/0.3/0) shows monotonically decreasing observed frequencies and ends with no carriers at the final timepoint (configured frequency 0)
- STEMSim log file (-l) completes without errors and output directory contains mutated reads plus the VCF of true mutations for all timepoints

## Limitations

- STEMSim 'only works well on species with high depth and well-characterized genomes'; it is designed for simulated longitudinal/concurrent raw sequencing reads, though it can run on real metagenomic data
- CAMISIM is only necessary for 'camisim' mode; the 'reads' mode is restricted to raw reads simulated from one reference genome
- Observed frequencies reflect Bernoulli sampling of read carriers, so finite read depth introduces sampling deviation from the exact specified frequencies

## Evidence

- [results] STEMSim realizes specified allele frequencies by randomly selecting the reads that carry each mutation via a Bernoulli process, so the observed per-timepoint frequencies recorded in the output reflect the configured trajectories.: [evidence span withheld — anchor status: not_found]
- [abstract] we introduce STEMSIM, a user-friendly command-line simulator of short-term evolutionary mutations for longitudinal metagenomic data: "we introduce STEMSIM, a user-friendly command-line simulator of short-term evolutionary mutations for longitudin"
- [readme] The 'camisim' model is designed for the simulated microbial communities by CAMISIM. The 'reads' model is designed for the simulated raw reads from one reference genome.: "The "camisim" model is designed for the simulated microbial communities by CAMISIM. The "reads" model is designe"
- [readme] is input raw read files from longitudinal samples separated by ':'; for paired reads from the same sample, they are separated by ',': "is input raw read files from longitudinal samples separated by ':'; for paired reads from the same sample, they"
- [readme] Although it can also works on real metagenomic data, it only works well on species with high depth and well-characterized genomes.: "Although it can also works on real metagenomic data, it only works well on species with high depth and well-char"
- [abstract] The output is the modified reads with within-strain evolutionary mutations and the relevant information of these mutations.: "The output is the modified reads with within-strain evolutionary mutations and the relevant information of these"

---
name: runtime-benchmarking-wallclock-timing
description: Use when a reproducibility task asks to confirm a published runtime claim
  for STEMSim, e.g. reproducing that one million reads are processed in approximately
  three minutes on a server with a 2.40 GHz CPU.
when_to_use_negative:
- The goal is to evaluate STEMSim's mutation simulation accuracy (e.g. mutation rates
  or VCF correctness) rather than its processing speed.
- The input is real metagenomic data rather than simulated raw reads — STEMSim only
  works well on species with high depth and well-characterized genomes, so timing
  on such data does not reproduce the benchmark.
- Hardware differs substantially from the reference server (2.40 GHz CPU), making
  a direct comparison to the reported ~3-minute runtime unreliable.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_0622
tools:
- name: STEMSim
  role: Tool whose wall-clock runtime is being benchmarked; invoked via stemsim.py
    in reads or camisim mode
  repo: https://github.com/BoyanZhou/STEMSim
- name: Bowtie2
  role: Internal alignment dependency STEMSim uses during the benchmarked run
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: samtools
  role: Internal file-handling dependency STEMSim uses during the benchmarked run
  repo: http://www.htslib.org/doc/
- name: pysam
  role: Python package (tested with 0.16.0.1) required by STEMSim during the benchmarked
    run
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad302
    title: 'STEMSIM: a simulator of within-strain short-term evolutionary mutations
      for longitudinal metagenomic data'
  source_content_hash: sha256:01fdf89eda71bd17079fd5edfd4fd2794c4faff6f4926c514ba9a2e6caffa073
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BoyanZhou/STEMSim
  tools:
  - Bowtie2
  - Python
  - STEMSim
  - numpy
  - pysam
  - samtools
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# runtime-benchmarking-wallclock-timing

## Summary

Benchmark the wall-clock runtime of STEMSim by wrapping a fixed-size simulation (one million simulated sequencing reads) in a wall-clock timer on a comparable server CPU, to verify the tool meets the ~3 minutes runtime reported in Zhou & Li (2023).

## When to use

Use when a reproducibility task asks to confirm a published runtime claim for STEMSim, e.g. reproducing that one million reads are processed in approximately three minutes on a server with a 2.40 GHz CPU.

## When NOT to use

- The goal is to evaluate STEMSim's mutation simulation accuracy (e.g. mutation rates or VCF correctness) rather than its processing speed.
- The input is real metagenomic data rather than simulated raw reads — STEMSim only works well on species with high depth and well-characterized genomes, so timing on such data does not reproduce the benchmark.
- Hardware differs substantially from the reference server (2.40 GHz CPU), making a direct comparison to the reported ~3-minute runtime unreliable.

## Inputs

- One-million-read simulated longitudinal metagenomic example dataset (FASTQ read files, e.g. sample0_R1.fq, sample0_R2.fq, sample1.fq, sample2.fq)
- Reference genome FASTA (.fas) used to simulate the raw reads
- STEMSim config file prepared from the config.txt template with tutorial-default parameters
- genome ID string for the reference genome

## Outputs

- runtime_report.txt containing a single numeric elapsed wall-clock time
- stemsim_test1_log run log confirming the fixed one-million-read input was processed without errors
- STEMSim output directory with modified reads carrying within-strain evolutionary mutations and mutation information

## How to apply

Obtain the one-million-read simulated metagenomic example dataset and its reference genome from the STEMSim GitHub wiki tutorial (github.com/BoyanZhou/STEMSim/wiki). Prepare a config file from the provided config.txt template, keeping all parameter settings identical to the tutorial defaults so that input size is the only variable. Invoke stemsim.py in the appropriate working mode (-m reads or -m camisim), supplying the longitudinal read files, reference genome, genome ID, and the log file stemsim_test1_log via the -l flag; STEMSim internally uses Bowtie2 for alignment and samtools for file handling under Python 3.6.5 with pysam 0.16.0.1. Wrap the invocation in a wall-clock timer on a comparable server CPU and confirm the run completes, writing modified reads, mutation information, and the run log. Record the measured elapsed time in runtime_report.txt.

## Related tools

- **STEMSim** (Tool whose wall-clock runtime is being benchmarked; invoked via stemsim.py in reads or camisim mode) — https://github.com/BoyanZhou/STEMSim
- **Bowtie2** (Internal alignment dependency STEMSim uses during the benchmarked run) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **samtools** (Internal file-handling dependency STEMSim uses during the benchmarked run) — http://www.htslib.org/doc/
- **pysam** (Python package (tested with 0.16.0.1) required by STEMSim during the benchmarked run)

## Examples

```
time python /path_to_stemsim_directory/stemsim.py -m reads -f PATH/config.txt -s test_subject1 --input_reads PATH/sample0_R1.fq,PATH/sample0_R2.fq:PATH/sample1.fq:PATH/sample2.fq -o PATH_to_output_directory_of_STEMSim/ --reference_genome PATH/species0_strain0.fas --genome_id species0_strain0 -l Path/stemsim_test1_log
```

## Evaluation signals

- runtime_report.txt exists and states a single numeric elapsed wall-clock time.
- stemsim_test1_log confirms the run processed the fixed one-million-read input without errors.
- The measured elapsed time is approximately three minutes when run on a comparable server with a 2.40 GHz CPU.
- The STEMSim output directory contains the modified reads with mutations plus the mutation information files, confirming a complete successful run.
- The config file used matches the tutorial defaults from the STEMSim wiki, so input size is the only variable in the benchmark.

## Limitations

- The reported ~3-minute runtime is tied to a 2.40 GHz server CPU; wall-clock time will differ on other hardware.
- STEMSim is designed for longitudinal or concurrent simulated raw sequencing reads; although it can run on real metagenomic data, it only works well on species with high depth and well-characterized genomes.
- Runtime depends on the installed dependency versions (Bowtie2, samtools 1.9, Python 3.6.5, numpy 1.19.5, pysam 0.16.0.1) matching those the paper was tested with.

## Evidence

- [full_text] STEMSIM is fast enough for benchmark-scale use, processing one million reads in approximately three minutes when run on a server with a 2.40 GHz CPU.: [evidence span withheld — anchor status: not_found]
- [readme] This code was written and tested on python 3.6.5, and requires the following packages: "This code was written and tested on python 3.6.5, and requires the following packages"
- [readme] -l Path/stemsim_test1_log is the absolute of log file: "-l Path/stemsim_test1_log is the absolute of log file"
- [abstract] The output is the modified reads with within-strain evolutionary mutations and the relevant information of these mutations.: "The output is the modified reads with within-strain evolutionary mutations and the relevant information of these mutations."

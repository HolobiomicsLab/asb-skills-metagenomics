---
name: hiv-amplicon-read-processing
description: Use when benchmarking ShoRAH's error-correction (diri_sampler) and frequency-estimation
  (freqEst) stages on large HIV-1 amplicon/pyrosequencing read sets, or when validating
  reported runtimes against the paper's hardware configuration.
when_to_use_negative:
- Input reads are already error-corrected or a haplotype frequency table already exists
  — re-running the timed steps adds no information
- Global haplotype reconstruction is required on ShoRAH2, where global mode is disabled
  (use ShoRAH v1.1.3 or V-pipe global options instead)
- A packaged binary/container or Windows-native execution is required without WSL/Cygwin,
  and compiling from the command line is not feasible
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3168
tools:
- name: ShoRAH
  role: Package whose error-correction (diri_sampler) and frequency-estimation (freqEst)
    steps are timed for the benchmark
  repo: http://www.cbg.ethz.ch/software/shorah
- name: shorah.py
  role: Wrapper that calls the four major steps; its overhead is verified to remain
    negligible relative to the timed steps
  repo: github.com/cbg-ethz/shorah
- name: diri_sampler
  role: Gibbs sampler for error correction via Dirichlet process mixture, invoked
    on overlapping MSA windows; the step benchmarked at <12 h
  repo: github.com/cbg-ethz/shorah
- name: shorah amplicon
  role: Entry point for amplicon-based local analysis that cuts windows and invokes
    diri_sampler on them
  repo: github.com/cbg-ethz/shorah
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/1471-2105-12-119
    title: 'ShoRAH: estimating the genetic diversity of a mixed sample from next-generation
      sequencing data'
  source_content_hash: sha256:09bceeaade4ef9c415f1a0f4e1cfaa84f1657ccd1e1a625f8d229bacdb75c90e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ShoRAH
  - diri_sampler
  - shorah amplicon
  - shorah.py
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# hiv-amplicon-read-processing

## Summary

A runtime-benchmark skill for ShoRAH-based amplicon read processing: it reproduces the paper's computational-efficiency measurements for the error-correction (local haplotype reconstruction) and frequency-estimation steps on a large pyrosequencing sample, recording hardware configuration and wall-clock timings.

## When to use

Use when benchmarking ShoRAH's error-correction (diri_sampler) and frequency-estimation (freqEst) stages on large HIV-1 amplicon/pyrosequencing read sets, or when validating reported runtimes against the paper's hardware configuration.

## When NOT to use

- Input reads are already error-corrected or a haplotype frequency table already exists — re-running the timed steps adds no information
- Global haplotype reconstruction is required on ShoRAH2, where global mode is disabled (use ShoRAH v1.1.3 or V-pipe global options instead)
- A packaged binary/container or Windows-native execution is required without WSL/Cygwin, and compiling from the command line is not feasible

## Inputs

- Large pyrosequencing sample of 40,000 reads of average length 300 bp (HIV-1 pol gene type data)
- Sorted BAM/CRAM alignment file of reads aligned to the reference of the amplified region
- Reference sequence of the amplified region

## Outputs

- runtime_benchmark.csv with measured error-correction and freqEst timings plus hardware configuration (6 CPUs at 2.8 GHz)
- Corrected reads / local haplotype reconstruction from diri_sampler windows
- Haplotype frequency estimates from freqEst EM output

## How to apply

Load a large pyrosequencing sample of 40,000 reads of average length ~300 bp (HIV-1 pol gene type data) and confirm read count and length distribution match the benchmark conditions. Run the error-correction/local haplotype reconstruction step by invoking diri_sampler on windows cut from the multiple sequence alignment (via shorah amplicon or shorah shotgun), parallelized across 6 CPUs at 2.8 GHz, and record the wall-clock time (target: less than 12 hours). Feed corrected reads into freqEst, running the EM algorithm with a log-likelihood convergence threshold of 10^-6 and a maximum of 5000 iterations, and record its runtime (reported: about two days). Verify that mm.py and the shorah.py wrapper overhead remain negligible relative to the two timed steps. Write measured timings and hardware configuration to runtime_benchmark.csv and compare against the paper's targets.

## Related tools

- **ShoRAH** (Package whose error-correction (diri_sampler) and frequency-estimation (freqEst) steps are timed for the benchmark) — http://www.cbg.ethz.ch/software/shorah
- **shorah.py** (Wrapper that calls the four major steps; its overhead is verified to remain negligible relative to the timed steps) — github.com/cbg-ethz/shorah
- **diri_sampler** (Gibbs sampler for error correction via Dirichlet process mixture, invoked on overlapping MSA windows; the step benchmarked at <12 h) — github.com/cbg-ethz/shorah
- **shorah amplicon** (Entry point for amplicon-based local analysis that cuts windows and invokes diri_sampler on them) — github.com/cbg-ethz/shorah

## Examples

```
shorah amplicon  # run local amplicon-mode analysis (error correction via diri_sampler on MSA windows) on a sorted bam input
```

## Evaluation signals

- Measured error-correction (diri_sampler) wall-clock time is below 12 hours on 6 CPUs at 2.8 GHz for 40,000 pyrosequencing reads
- freqEst EM runtime is on the order of two days, with log-likelihood convergence threshold 10^-6 and at most 5000 iterations
- Read count and length distribution of the input match benchmark conditions (40,000 reads, average ~300 bp, HIV-1 pol gene type data)
- mm.py and shorah.py wrapper overhead is negligible relative to the two timed steps
- runtime_benchmark.csv contains timings, hardware configuration (CPU count and clock speed), and comparisons against the reported targets

## Limitations

- Benchmark runtimes are hardware-specific (6 CPUs at 2.8 GHz); they may not transfer to different CPU counts, clock speeds, or read volumes
- ShoRAH must be compiled and run from the command line; SAMtools and pysam are not included in the package and must be obtained separately
- ShoRAH has been tested on Linux and Mac OS X only; Windows users need WSL or Cygwin

## Evidence

- [methods] Runtime benchmark conditions and result: "the error correction step was completed in less than 12 hours using 6 CPUs at 2.8 GHz for a large sample consisting of 40,000 pyrosequencing reads"
- [methods] Four major workflow steps performed by the wrapper: "These are: 1) alignment; 2) error correction (local haplotype reconstruction); 3) global haplotype reconstruction and 4) frequency estimation"
- [intro] Error correction by read clustering in genome windows: "This software corrects sequencing errors by clustering all reads that overlap the same region of the genome of length approximately equal to the read length"
- [readme] How local amplicon analysis invokes the sampler: "They work by cutting window from the multiple sequence alignment, invoking `diri_sampler` on the windows and calling `shorah snv` for the SNV calling"
- [readme] Input format for local analysis: "The input is a sorted bam file. Analysis can be performed in local mode."

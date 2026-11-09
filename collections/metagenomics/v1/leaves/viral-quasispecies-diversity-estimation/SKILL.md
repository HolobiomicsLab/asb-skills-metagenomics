---
name: viral-quasispecies-diversity-estimation
description: Use when deep sequencing reads from a genetically heterogeneous viral
  sample must be error-corrected, clustered into local/global haplotypes, and their
  frequencies estimated, especially to detect rare variants.
when_to_use_negative:
- Only the consensus sequence is needed and low-frequency variants are irrelevant
  — Sanger-style consensus analysis suffices
- Global haplotype reconstruction is required while using ShoRAH2, which has global
  mode disabled; use ShoRAH1 v1.1.3 or V-pipe's global options instead
- Data are not from deep sequencing of a mixed viral sample (e.g. single-isolate sequencing
  or non-viral variant tasks outside quasispecies scope)
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: ShoRAH
  role: 'Main pipeline: error correction, haplotype reconstruction and frequency estimation
    of variants in a mixed sample'
  repo: http://www.cbg.ethz.ch/software/shorah
- name: shorah shotgun
  role: 'Runs the local analysis: cuts windows from the MSA, invokes diri_sampler
    on windows and calls shorah snv'
  repo: https://github.com/cbg-ethz/shorah
- name: diri_sampler
  role: Gibbs sampler for error correction via Dirichlet process mixture clustering
    of reads per window
  repo: https://github.com/cbg-ethz/shorah
- name: V-pipe
  role: Workflow providing global haplotype reconstruction options when ShoRAH2's
    global mode is unavailable
  repo: https://github.com/cbg-ethz/V-pipe
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/1471-2105-12-119
    title: 'ShoRAH: estimating the genetic diversity of a mixed sample from next-generation
      sequencing data'
  source_content_hash: sha256:48ec0e0a76e63ac55b147865075cd82c74b78e7d19deb73979aca74ea19fb24e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - ShoRAH
  - V-pipe
  - diri_sampler
  - shorah shotgun
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# viral-quasispecies-diversity-estimation

## Summary

Estimate the genetic diversity of a mixed viral sample and quantify constituent haplotypes from deep sequencing data using ShoRAH, which corrects sequencing errors via Dirichlet-process clustering of overlapping reads. Enables detection of rare variants, e.g. haplotypes at 0.1% frequency at 5000x coverage in simulated HIV-1 pol data.

## When to use

Use when deep sequencing reads from a genetically heterogeneous viral sample must be error-corrected, clustered into local/global haplotypes, and their frequencies estimated, especially to detect rare variants.

## When NOT to use

- Only the consensus sequence is needed and low-frequency variants are irrelevant — Sanger-style consensus analysis suffices
- Global haplotype reconstruction is required while using ShoRAH2, which has global mode disabled; use ShoRAH1 v1.1.3 or V-pipe's global options instead
- Data are not from deep sequencing of a mixed viral sample (e.g. single-isolate sequencing or non-viral variant tasks outside quasispecies scope)

## Inputs

- Sorted BAM/CRAM/SAM file of deep sequencing reads aligned to a reference (e.g. HIV-1 pol)
- Reference FASTA sequence of the amplified region
- Configuration of window width (~average read length) and diri_sampler parameters (iterations, alpha)

## Outputs

- Error-corrected (snv-corrected) reads
- Local and global reconstructed haplotype sequences
- Estimated haplotype frequencies in the population
- SNV calls from shorah snv (e.g. SNVs_0.010000.txt)

## How to apply

Follow ShoRAH's four steps: 1) alignment of reads to the reference sequence of the amplified region (e.g. HIV-1 subtype B pol gene) and construction of a multiple sequence alignment; 2) error correction / local haplotype reconstruction by clustering all reads overlapping the same genome window of length approximately equal to the read length using diri_sampler (Gibbs sampling with a Dirichlet process mixture model), tuning iterations and alpha for proper mixing; 3) global haplotype reconstruction from adjacent overlapping reads (only in ShoRAH1 v1.1.3, disabled in ShoRAH2); 4) frequency estimation from the number of reads associated with each cluster. For shotgun data invoke 'shorah shotgun', which cuts windows from the MSA, runs diri_sampler, and calls shorah snv. The error correction reduces the error rate approximately five-fold (0.25% to 0.05%) and detects haplotypes as rare as 0.1% at 5000x coverage.

## Related tools

- **ShoRAH** (Main pipeline: error correction, haplotype reconstruction and frequency estimation of variants in a mixed sample) — http://www.cbg.ethz.ch/software/shorah
- **shorah shotgun** (Runs the local analysis: cuts windows from the MSA, invokes diri_sampler on windows and calls shorah snv) — https://github.com/cbg-ethz/shorah
- **diri_sampler** (Gibbs sampler for error correction via Dirichlet process mixture clustering of reads per window) — https://github.com/cbg-ethz/shorah
- **V-pipe** (Workflow providing global haplotype reconstruction options when ShoRAH2's global mode is unavailable) — https://github.com/cbg-ethz/V-pipe

## Examples

```
shorah shotgun
```

## Evaluation signals

- Simulated ground-truth check: haplotypes at 0.1% frequency detected and their frequencies correctly estimated at a coverage of 5000 (10 HIV-1 subtype B pol variants)
- Error rate after correction drops roughly five-fold, from ~0.25% to ~0.05%
- Cluster/frequency sanity: the number of reads associated with each cluster matches the known mixture proportions of simulated variants
- Pipeline completes end-to-end: alignment, window cutting, diri_sampler mixing, and frequency estimation output files are produced
- Benchmark feasibility: error correction finishes in less than 12 hours on 6 CPUs at 2.8 GHz for 40,000 pyrosequencing reads

## Limitations

- ShoRAH2 has global haplotype reconstruction disabled; use ShoRAH1 v1.1.3 (e.g. via bioconda) or V-pipe global options for global reconstruction
- SAMtools/pysam-style dependencies (HTSlib, zlib, Boost, Biopython, NumPy) must be installed separately; software must be compiled from command line (Linux/Mac OS X tested, Windows via WSL/Cygwin)
- Performance depends on proper tuning of diri_sampler iterations and hyper-parameter alpha for mixing; windows must be sized near the average read length

## Evidence

- [methods] Haplotypes with frequencies as low as 0.1% were detected at a coverage of 5000 and their frequency correctly estimated: "Haplotypes with frequencies as low as 0.1% were detected at a coverage of 5000 and their frequency correctly estimated"
- [intro] This software corrects sequencing errors by clustering all reads that overlap the same region of the genome of length approximately equal to the read length: "This software corrects sequencing errors by clustering all reads that overlap the same region of the genome of length approximately equal to the read length"
- [methods] These are: 1) alignment; 2) error correction (local haplotype reconstruction); 3) global haplotype reconstruction and 4) frequency estimation: "These are: 1) alignment; 2) error correction (local haplotype reconstruction); 3) global haplotype reconstruction and 4) frequency estimation"
- [readme] The local analysis alone can be run invoking `shorah shotgun` or `shorah amplicon`: "The local analysis alone can be run invoking `shorah shotgun` or `shorah amplicon`"
- [methods] simulating reads from 10 different variants of the HIV-1 subtype B pol gene: "simulating reads from 10 different variants of the HIV-1 subtype B pol gene"

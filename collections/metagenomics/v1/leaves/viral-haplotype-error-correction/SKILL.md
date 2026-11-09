---
name: viral-haplotype-error-correction
description: Use when error-prone NGS reads (e.g. 454 Titanium pyrosequencing) from
  a genetically heterogeneous viral sample are aligned to a reference (e.g. HIV-1
  subtype B pol) and low-frequency variants (~0.1%) must be separated from sequencing
  errors.
when_to_use_negative:
- Global (whole-genome-length) haplotype reconstruction is required and ShoRAH2 is
  installed — global mode is disabled there; use ShoRAH v1.1.3 or the global options
  in V-pipe instead
- The input is not an alignment of reads to a reference (e.g. only unaligned FASTQ
  or a finished consensus sequence) — the MSA must be built first, or a different
  tool is needed
- Only single-nucleotide variants with strand-bias correction are wanted from a clean
  sample, without read error correction — call 'shorah snv' directly rather than running
  the full correction pipeline
edam_operation: http://edamontology.org/operation_0491
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3308
tools:
- name: ShoRAH
  role: Overall suite for error correction, haplotype reconstruction and frequency
    estimation from deep sequencing of a mixed sample; local analysis invoked via
    'shorah shotgun' or 'shorah amplicon'
  repo: http://www.cbg.ethz.ch/software/shorah
- name: diri_sampler
  role: Gibbs sampling for error correction via Dirichlet process mixture; clusters
    reads in each window into local haplotypes
  repo: https://github.com/cbg-ethz/shorah
- name: b2w
  role: Splits the shotgun sequencing .BAM alignment into multiple overlapping windows
    feeding diri_sampler
  repo: https://github.com/cbg-ethz/shorah
- name: shorah snv
  role: Downstream single-nucleotide variant detection called after window-based clustering,
    taking strand bias into account
  repo: https://github.com/cbg-ethz/shorah
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/1471-2105-12-119
    title: 'ShoRAH: estimating the genetic diversity of a mixed sample from next-generation
      sequencing data'
  source_content_hash: sha256:9e975e2c254aa3cb1a57d1ac2e48f20d525707270b2dc64806175f293113c314
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - ShoRAH
  - b2w
  - diri_sampler
  - shorah shotgun
  - shorah snv
  - shorah.py
  edam_operation: http://edamontology.org/operation_0491
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# viral-haplotype-error-correction

## Summary

ShoRAH-based error correction for deep sequencing of mixed viral samples: reads overlapping the same genome window (~read length) are clustered with a Dirichlet process mixture (diri_sampler) to reconstruct local haplotypes, and original reads are corrected by majority vote across windows.

## When to use

Use when error-prone NGS reads (e.g. 454 Titanium pyrosequencing) from a genetically heterogeneous viral sample are aligned to a reference (e.g. HIV-1 subtype B pol) and low-frequency variants (~0.1%) must be separated from sequencing errors.

## When NOT to use

- Global (whole-genome-length) haplotype reconstruction is required and ShoRAH2 is installed — global mode is disabled there; use ShoRAH v1.1.3 or the global options in V-pipe instead
- The input is not an alignment of reads to a reference (e.g. only unaligned FASTQ or a finished consensus sequence) — the MSA must be built first, or a different tool is needed
- Only single-nucleotide variants with strand-bias correction are wanted from a clean sample, without read error correction — call 'shorah snv' directly rather than running the full correction pipeline

## Inputs

- Sorted .BAM/.CRAM file of shotgun or amplicon NGS reads aligned to a reference (e.g. 454 Titanium reads vs. HIV-1 subtype B pol)
- Multiple sequence alignment of reads against the reference (produced by ShoRAH's alignment step)
- Window definitions over the MSA (each position covered three times, width ≈ average read length)

## Outputs

- Corrected reads (error-corrected via majority vote across overlapping windows)
- Local haplotype reconstructions per window with estimated frequencies from cluster read counts
- Residual sequencing error rate estimate for the corrected reads

## How to apply

Build a multiple sequence alignment of reads against the reference (pairwise alignment to the reference followed by MSA via gap padding) as the first ShoRAH step. Split the MSA into overlapping windows with dec.py/b2w so that every alignment position is covered three times, setting the window width approximately equal to the average read length. Run diri_sampler on each window — a Gibbs-sampling Dirichlet process mixture — choosing the number of iterations and the alpha hyperparameter large enough to guarantee proper mixing, i.e. so that solutions proposing new clusters are actually reached. Correct the original reads by majority vote based on the local haplotype reconstructions from the overlapping windows; the number of reads per cluster estimates haplotype prevalence. Drive the local analysis with the wrapper 'shorah shotgun' (or 'shorah amplicon') on a sorted BAM, and optionally call shorah snv afterwards. Validate by computing the residual error rate of corrected reads (expect a five-fold decrease, 0.25% to 0.05%) and checking that clones with frequencies as low as 0.1% are recovered at coverage ~5000–6000.

## Related tools

- **ShoRAH** (Overall suite for error correction, haplotype reconstruction and frequency estimation from deep sequencing of a mixed sample; local analysis invoked via 'shorah shotgun' or 'shorah amplicon') — http://www.cbg.ethz.ch/software/shorah
- **diri_sampler** (Gibbs sampling for error correction via Dirichlet process mixture; clusters reads in each window into local haplotypes) — https://github.com/cbg-ethz/shorah
- **b2w** (Splits the shotgun sequencing .BAM alignment into multiple overlapping windows feeding diri_sampler) — https://github.com/cbg-ethz/shorah
- **shorah snv** (Downstream single-nucleotide variant detection called after window-based clustering, taking strand bias into account) — https://github.com/cbg-ethz/shorah

## Examples

```
shorah shotgun
```

## Evaluation signals

- Residual error rate of corrected reads shows a five-fold decrease, from 0.25% to 0.05%
- Individual haplotypes/clones with frequencies as low as 0.1% are detected and their frequencies correctly estimated at coverage around 5000 (validation run used coverage ~6000)
- Window scheme verifies each alignment position is covered three times, with window width approximately equal to the average read length
- diri_sampler trajectories show proper mixing: solutions proposing new clusters are reached during Gibbs sampling (sufficient iterations and alpha)
- Corrected reads reproduce the majority-vote local haplotype consensus across all overlapping windows covering each position

## Limitations

- The error correction step is expensive: less than 12 hours using 6 CPUs at 2.8 GHz was needed for 40,000 pyrosequencing reads
- ShoRAH2 has global haplotype reconstruction disabled; the last version with global mode is v1.1.3, otherwise V-pipe's global haplotype options are suggested
- ShoRAH must be compiled and run from the command line, requires external dependencies (HTSlib, Biopython, NumPy, Boost, zlib), and the original version was only tested on Linux and Mac OS X
- diri_sampler results depend on iteration count and the alpha hyperparameter: if set too small, mixing fails and new clusters (rare haplotypes) are never proposed

## Evidence

- [intro] Error correction mechanism: clustering of overlapping reads in windows of ~read length: "This software corrects sequencing errors by clustering all reads that overlap the same region of the genome of length approximately equal to the read length"
- [readme] diri_sampler implements Gibbs sampling under a Dirichlet process mixture for error correction: "`diri_sampler` | Gibbs sampling for error correction via Dirichlet process mixture"
- [methods] Validation threshold: five-fold error-rate reduction: "A five-fold decrease of the error rate, from 0.25% to 0.05%, was achieved"
- [methods] Sensitivity threshold: 0.1% haplotypes recovered at coverage 5000: "Haplotypes with frequencies as low as 0.1% were detected at a coverage of 5000 and their frequency correctly estimated"
- [methods] Pipeline structure: four steps driven by the shorah.py wrapper: "Full analysis with ShoRAH includes four major steps that are performed by calling the wrapper shorah.py"
- [readme] b2w generates the overlapping windows consumed by the clustering step: "`b2w`          | splitting shotgun sequencing .BAM into multiple overlapping windows"

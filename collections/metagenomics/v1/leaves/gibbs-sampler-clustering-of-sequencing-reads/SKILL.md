---
name: gibbs-sampler-clustering-of-sequencing-reads
description: Use when error-prone shotgun or amplicon NGS reads (e.g. 454 Titanium
  pyrosequencing of HIV-1 pol) aligned to a reference must be partitioned into local-haplotype
  clusters to correct sequencing errors before quantifying genetic diversity.
when_to_use_negative:
- You need global (full-length) haplotype reconstruction from adjacent overlapping
  reads — ShoRAH2 has global mode disabled; use ShoRAH v1.1.3 or V-pipe's global haplotype
  options instead
- The goal is single-nucleotide variant calling with strand-bias accounting — that
  is shorah snv's role, not read clustering for error correction
- Input reads are not yet aligned to a reference — alignment is a prerequisite step
  before windowing and clustering
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: diri_sampler
  role: Performs the Gibbs sampling for error correction via Dirichlet process mixture
    on each sequence window
  repo: http://www.cbg.ethz.ch/software/shorah
- name: shorah shotgun
  role: 'Wraps the local analysis: cuts windows from the MSA, invokes diri_sampler
    on the windows, and calls shorah snv'
  repo: https://github.com/cbg-ethz/shorah
- name: b2w
  role: Splits shotgun-sequencing BAM input into multiple overlapping windows prior
    to clustering
  repo: https://github.com/cbg-ethz/shorah
- name: V-pipe
  role: Alternative route for global haplotype reconstruction options when local clustering
    alone is insufficient
  repo: https://github.com/cbg-ethz/V-pipe
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/1471-2105-12-119
    title: 'ShoRAH: estimating the genetic diversity of a mixed sample from next-generation
      sequencing data'
  source_content_hash: sha256:ceb6752c59892f395099714aa839d601868a50124e9afbae5846ff47dbf0c37a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - ShoRAH
  - V-pipe
  - b2w
  - diri_sampler
  - shorah shotgun
  - shorah.py
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# Gibbs-sampler clustering of sequencing reads

## Summary

Clusters error-prone deep-sequencing reads by genome window using a Dirichlet process mixture with Gibbs sampling (ShoRAH's diri_sampler), so that read clusters define local haplotypes whose majority-vote consensus corrects sequencing errors and whose cluster sizes estimate haplotype prevalence.

## When to use

Use when error-prone shotgun or amplicon NGS reads (e.g. 454 Titanium pyrosequencing of HIV-1 pol) aligned to a reference must be partitioned into local-haplotype clusters to correct sequencing errors before quantifying genetic diversity.

## When NOT to use

- You need global (full-length) haplotype reconstruction from adjacent overlapping reads — ShoRAH2 has global mode disabled; use ShoRAH v1.1.3 or V-pipe's global haplotype options instead
- The goal is single-nucleotide variant calling with strand-bias accounting — that is shorah snv's role, not read clustering for error correction
- Input reads are not yet aligned to a reference — alignment is a prerequisite step before windowing and clustering

## Inputs

- Sorted BAM/CRAM file of reads aligned to a reference sequence (e.g. 454 Titanium reads vs. HIV-1 subtype B pol)
- Multiple sequence alignment windows of width ~read length, each genome position covered three times (produced by b2w/dec.py window splitting)
- diri_sampler run parameters: number of Gibbs iterations and Dirichlet process hyperparameter alpha

## Outputs

- Cluster assignments of reads per window (local haplotype reconstructions from the Dirichlet process mixture)
- Haplotype prevalence estimates per cluster (number of reads associated with each cluster)
- Error-corrected reads obtained by majority vote across overlapping windows

## How to apply

Split the multiple sequence alignment (built against the reference, e.g. HIV-1 subtype B pol) into overlapping windows with width approximately equal to the average read length, so every alignment position is covered three times. On each window run diri_sampler, a model-based probabilistic clustering algorithm using a Dirichlet process mixture with Gibbs sampling. Set the number of iterations and the hyperparameter alpha large enough to guarantee proper mixing — solutions with new clusters must be proposed, otherwise rare variants collapse into existing clusters. The number of reads associated with each cluster estimates the prevalence of the haplotype in the population, and overlapping-window cluster solutions are combined by majority vote to correct the original reads. At coverage around 5000-6000, success is a five-fold residual error-rate decrease (0.25% to 0.05%) and correct recovery of clones with frequencies as low as 0.1%.

## Related tools

- **diri_sampler** (Performs the Gibbs sampling for error correction via Dirichlet process mixture on each sequence window) — http://www.cbg.ethz.ch/software/shorah
- **shorah shotgun** (Wraps the local analysis: cuts windows from the MSA, invokes diri_sampler on the windows, and calls shorah snv) — https://github.com/cbg-ethz/shorah
- **b2w** (Splits shotgun-sequencing BAM input into multiple overlapping windows prior to clustering) — https://github.com/cbg-ethz/shorah
- **V-pipe** (Alternative route for global haplotype reconstruction options when local clustering alone is insufficient) — https://github.com/cbg-ethz/V-pipe

## Examples

```
shorah shotgun
```

## Evaluation signals

- Residual sequencing error rate of the corrected reads shows a five-fold decrease, from 0.25% to 0.05%
- Individual clones with frequencies as low as 0.1% are inferred with correct frequency estimates at coverage ~5000
- Every alignment position is covered by three overlapping windows of width approximately equal to the read length
- Trace of the Gibbs sampler shows new clusters being proposed, confirming proper mixing given the chosen iterations and alpha
- Runtime sanity: error correction of ~40,000 pyrosequencing reads should complete in less than 12 hours using 6 CPUs at 2.8 GHz

## Limitations

- Global haplotype reconstruction is disabled in ShoRAH2; the last version with global mode is v1.1.3
- If the number of iterations or hyperparameter alpha is too small, the Dirichlet process fails to propose new clusters and rare haplotypes are not separated
- Tested on Linux and Mac OS X platforms (Windows only via WSL or Cygwin); Windows support is not native
- Error correction is computationally expensive — under 12 hours on 6 CPUs for 40,000 pyrosequencing reads

## Evidence

- [intro] Clustering corrects sequencing errors by read overlap: "This software corrects sequencing errors by clustering all reads that overlap the same region of the genome of length approximately equal to the read length"
- [intro] Prevalence from cluster sizes: "The number of reads associated with the cluster estimates the prevalence of the haplotype in the population"
- [methods] Five-fold error rate decrease: "A five-fold decrease of the error rate, from 0.25% to 0.05%, was achieved"
- [methods] Rare clone recovery: "Haplotypes with frequencies as low as 0.1% were detected at a coverage of 5000 and their frequency correctly estimated"
- [methods] Runtime of error correction step: "the error correction step was completed in less than 12 hours using 6 CPUs at 2.8 GHz for a large sample consisting of 40,000 pyrosequencing reads"
- [readme] diri_sampler role in the suite: "`diri_sampler` | Gibbs sampling for error correction via Dirichlet process mixture"
- [readme] Local analysis invocation: "The local analysis alone can be run invoking `shorah shotgun` or `shorah amplicon`"

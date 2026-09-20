---
name: illumina-short-read-error-correction
description: Use when deep-sequencing reads (BAM/CRAM aligned to a reference such
  as HIV-1 subtype B pol) from a genetically heterogeneous mixed sample must be error-corrected
  and rare variants down to 0.1% frequency quantified.
when_to_use_negative:
- 'When global (full-length) haplotype reconstruction is required: current ShoRAH2
  has global mode disabled; use the v1.1.3 tarball with global mode or V-pipe''s global
  haplotype options instead.'
- When the sample is homogeneous or low-frequency variants are not of interest — the
  error-correction/clustering machinery targets heterogeneous mixed samples, not consensus-only
  analyses.
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0102
tools:
- name: ShoRAH
  role: 'overall package: aligns reads, clusters overlapping-window reads, reconstructs
    local haplotypes, and estimates frequencies'
  repo: https://github.com/cbg-ethz/shorah
- name: diri_sampler
  role: Gibbs sampler with Dirichlet process mixture for error correction on each
    alignment window
  repo: https://github.com/cbg-ethz/shorah
- name: b2w
  role: splits shotgun-sequencing .BAM files into multiple overlapping windows before
    sampling
  repo: https://github.com/cbg-ethz/shorah
- name: shorah shotgun
  role: 'wrapper driving the local analysis: window cutting, diri_sampler invocation,
    and SNV calling'
  repo: https://github.com/cbg-ethz/shorah
- name: shorah snv
  role: called after diri_sampler to detect single nucleotide variants, taking strand
    bias into account
  repo: https://github.com/cbg-ethz/shorah
- name: V-pipe
  role: recommended downstream workflow offering global haplotype reconstruction options
    not in ShoRAH2
  repo: https://github.com/cbg-ethz/V-pipe
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/1471-2105-12-119
    title: 'ShoRAH: estimating the genetic diversity of a mixed sample from next-generation
      sequencing data'
  source_content_hash: sha256:48f9221303ed3f1693372d033c96ca0b4de9737a7857ea335a16098c9e1a7726
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cbg-ethz/shorah
  tools:
  - Python
  - ShoRAH
  - V-pipe
  - b2w
  - diri_sampler
  - shorah shotgun
  - shorah snv
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# illumina-short-read-error-correction

## Summary

Correct Illumina short-read sequencing errors and reconstruct local haplotypes by clustering reads that overlap the same genomic window via Gibbs sampling with a Dirichlet process mixture (ShoRAH's diri_sampler), then estimate haplotype frequencies from cluster read counts.

## When to use

Use when deep-sequencing reads (BAM/CRAM aligned to a reference such as HIV-1 subtype B pol) from a genetically heterogeneous mixed sample must be error-corrected and rare variants down to 0.1% frequency quantified.

## When NOT to use

- When global (full-length) haplotype reconstruction is required: current ShoRAH2 has global mode disabled; use the v1.1.3 tarball with global mode or V-pipe's global haplotype options instead.
- When the sample is homogeneous or low-frequency variants are not of interest — the error-correction/clustering machinery targets heterogeneous mixed samples, not consensus-only analyses.

## Inputs

- sorted BAM/CRAM file of aligned reads (e.g., test_aln.cram)
- reference sequence for the amplified/mapped region (e.g., HIV-1 subtype B pol)
- read length and desired window coverage (windows such that each position is covered 3 times)

## Outputs

- corrected reads (majority-vote consensus per window)
- inferred local haplotypes with estimated population frequencies
- single nucleotide variant calls with strand bias taken into account (SNV.txt / SNVs_0.010000.txt)

## How to apply

Align the reads to the reference (e.g., HIV-1 subtype B pol) and build a multiple sequence alignment with ShoRAH's alignment step. Split the MSA into overlapping windows — each alignment position is covered three times, with window width approximately equal to the average read length (the b2w/dec.py step for shotgun BAM input). Run diri_sampler, a Gibbs sampler with a Dirichlet process mixture model, on each window, tuning the number of iterations and hyper-parameter alpha to be large enough for proper mixing of the sampling. Correct original reads by majority vote based on the local reconstructions and aggregate cluster assignments into inferred haplotypes with frequencies estimated from the number of reads per cluster. The whole local analysis can be driven by the wrapper `shorah shotgun`; expect roughly a five-fold error-rate decrease (0.25% to 0.05%) and sensitivity to haplotypes at 0.1% frequency when coverage reaches 5000.

## Related tools

- **ShoRAH** (overall package: aligns reads, clusters overlapping-window reads, reconstructs local haplotypes, and estimates frequencies) — https://github.com/cbg-ethz/shorah
- **diri_sampler** (Gibbs sampler with Dirichlet process mixture for error correction on each alignment window) — https://github.com/cbg-ethz/shorah
- **b2w** (splits shotgun-sequencing .BAM files into multiple overlapping windows before sampling) — https://github.com/cbg-ethz/shorah
- **shorah shotgun** (wrapper driving the local analysis: window cutting, diri_sampler invocation, and SNV calling) — https://github.com/cbg-ethz/shorah
- **shorah snv** (called after diri_sampler to detect single nucleotide variants, taking strand bias into account) — https://github.com/cbg-ethz/shorah
- **V-pipe** (recommended downstream workflow offering global haplotype reconstruction options not in ShoRAH2) — https://github.com/cbg-ethz/V-pipe

## Examples

```
shorah shotgun -b test_aln.cram
```

## Evaluation signals

- Haplotypes with frequencies as low as 0.1% are detected at a coverage of 5000 and their frequencies match the known ground-truth mixture proportions of simulated variants.
- Error rate of corrected reads drops about five-fold, from ~0.25% to ~0.05%.
- Every alignment position is covered exactly three times by the overlapping windows, and window width approximates the average read length.
- Haplotype prevalence estimates track the number of reads associated with each cluster (invariant: frequency ≈ reads_in_cluster / total_reads).
- diri_sampler iterations and alpha are large enough that the Gibbs sampling shows proper mixing (stable cluster assignments across chains).

## Limitations

- Global haplotype reconstruction is disabled in ShoRAH2; the last version with global mode is v1.1.3.
- Requires external dependencies not always packaged: HTSlib, zlib, Boost (C++), Biopython and NumPy (Python 3); SAMtools/pysam must be obtained separately.
- Tested only on Linux and Mac OS X; Windows requires WSL or Cygwin and the software must be compiled from the command line.
- Hyper-parameter alpha and the number of Gibbs iterations must be tuned per run; insufficient values cause improper mixing of the sampler.
- Error correction is computationally heavy — under the reported pyrosequencing test, 40,000 reads took under 12 hours on 6 CPUs at 2.8 GHz.

## Evidence

- [intro] error correction by window clustering: "This software corrects sequencing errors by clustering all reads that overlap the same region of the genome of length approximately equal to the read length"
- [methods] sensitivity threshold: "Haplotypes with frequencies as low as 0.1% were detected at a coverage of 5000 and their frequency correctly estimated"
- [methods] error-rate reduction: "A five-fold decrease of the error rate, from 0.25% to 0.05%, was achieved"
- [readme] tool roles: "`diri_sampler` | Gibbs sampling for error correction via Dirichlet process mixture"
- [readme] local analysis entry point: "The local analysis alone can be run invoking `shorah shotgun` or `shorah amplicon`"
- [readme] global mode caveat: "Currently in ShoRAH2 global haplotype reconstruction is disabled."

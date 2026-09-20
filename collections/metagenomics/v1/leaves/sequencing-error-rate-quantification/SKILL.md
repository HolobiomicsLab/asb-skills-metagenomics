---
name: sequencing-error-rate-quantification
description: Use when error-prone NGS reads (e.g., 454 Titanium pyrosequencing of
  HIV-1 subtype B pol) from a genetically heterogeneous sample must be error-corrected
  before estimating haplotype frequencies or clone prevalence.
when_to_use_negative:
- The sample is expected to be clonal or only a consensus sequence is needed — Sanger-style
  consensus would suffice and low-frequency variants are not of interest
- Global (full-length) haplotype reconstruction is required with current ShoRAH2 —
  global mode is disabled there; use ShoRAH1 v1.1.3 or V-pipe's global haplotype options
  instead
- The goal is strand-bias-aware SNV detection alone without read error correction
  — shorah snv addresses SNV calling separately from the error-correction clustering
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_0625
tools:
- name: shorah shotgun
  role: 'Runs the local (window-based) analysis pipeline: cuts windows from the MSA,
    invokes diri_sampler, and calls shorah snv'
  repo: https://github.com/cbg-ethz/shorah
- name: diri_sampler
  role: Gibbs sampling for error correction via Dirichlet process mixture — clusters
    reads within each window into local haplotypes
  repo: https://github.com/cbg-ethz/shorah
- name: b2w / dec.py
  role: Splits the multiple sequence alignment (or shotgun BAM) into overlapping windows,
    each position covered three times, with width ~ read length
  repo: https://github.com/cbg-ethz/shorah
- name: shorah snv
  role: Detects single nucleotide variants after error correction, taking strand bias
    into account
  repo: https://github.com/cbg-ethz/shorah
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/1471-2105-12-119
    title: 'ShoRAH: estimating the genetic diversity of a mixed sample from next-generation
      sequencing data'
  source_content_hash: sha256:871560aa5a0cd8d106cf66edd11e3b0f1a4a5311d0f1298e7853c1f738e72e72
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cbg-ethz/shorah
  tools:
  - Python
  - ShoRAH
  - b2w / dec.py
  - diri_sampler
  - shorah shotgun
  - shorah snv
  - shorah.py
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# sequencing-error-rate-quantification

## Summary

Quantifies and corrects sequencing errors in mixed deep-sequencing samples by clustering overlapping reads per genome window with ShoRAH's Dirichlet-process mixture (diri_sampler), reducing the residual error rate five-fold (0.25% to 0.05%) so low-frequency variants become quantifiable.

## When to use

Use when error-prone NGS reads (e.g., 454 Titanium pyrosequencing of HIV-1 subtype B pol) from a genetically heterogeneous sample must be error-corrected before estimating haplotype frequencies or clone prevalence.

## When NOT to use

- The sample is expected to be clonal or only a consensus sequence is needed — Sanger-style consensus would suffice and low-frequency variants are not of interest
- Global (full-length) haplotype reconstruction is required with current ShoRAH2 — global mode is disabled there; use ShoRAH1 v1.1.3 or V-pipe's global haplotype options instead
- The goal is strand-bias-aware SNV detection alone without read error correction — shorah snv addresses SNV calling separately from the error-correction clustering

## Inputs

- Sorted BAM/CRAM file of shotgun or amplicon NGS reads (e.g., 454 Titanium pyrosequencing reads, ~40,000 reads at ~6000x coverage)
- Reference sequence of the amplified region (e.g., HIV-1 subtype B pol gene)

## Outputs

- Error-corrected reads (majority-vote consensus per read from overlapping window haplotypes)
- Local haplotype clusters per window with associated read counts (frequencies)
- Residual sequencing error rate estimate (expected five-fold decrease, 0.25% to 0.05%)
- SNV call files (e.g., SNVs_0.010000.txt) from shorah snv

## How to apply

Align the reads to the reference of the amplified region (e.g., pairwise alignment to the HIV-1 subtype B pol reference followed by MSA construction via gap padding). Split the MSA into overlapping windows using dec.py (or b2w for BAM input) so every alignment position is covered three times, with window width approximately equal to the average read length. Run diri_sampler on each window — a Gibbs-sampling Dirichlet process mixture — setting the iteration count and hyperparameter alpha large enough that solutions proposing new clusters are explored (proper mixing). Correct each original read by majority vote over the local haplotype reconstructions from overlapping windows, then compute the residual error rate of the corrected reads and the lowest clone frequency recovered at high coverage (around 5000–6000).

## Related tools

- **shorah shotgun** (Runs the local (window-based) analysis pipeline: cuts windows from the MSA, invokes diri_sampler, and calls shorah snv) — https://github.com/cbg-ethz/shorah
- **diri_sampler** (Gibbs sampling for error correction via Dirichlet process mixture — clusters reads within each window into local haplotypes) — https://github.com/cbg-ethz/shorah
- **b2w / dec.py** (Splits the multiple sequence alignment (or shotgun BAM) into overlapping windows, each position covered three times, with width ~ read length) — https://github.com/cbg-ethz/shorah
- **shorah snv** (Detects single nucleotide variants after error correction, taking strand bias into account) — https://github.com/cbg-ethz/shorah

## Examples

```
shorah shotgun
```

## Evaluation signals

- Residual error rate of corrected reads shows the expected five-fold decrease, from 0.25% to 0.05%
- Haplotypes with frequencies as low as 0.1% are detected at coverage ~5000 and their frequencies are correctly estimated
- Every alignment position is covered by exactly three overlapping windows of width approximately equal to the average read length
- diri_sampler chains are properly mixed: runs with sufficiently large alpha and iteration count propose new clusters rather than collapsing onto initial assignments
- Performance sanity check: error correction of ~40,000 pyrosequencing reads completes in under 12 hours on 6 CPUs at 2.8 GHz

## Limitations

- Compute-intensive: error correction needed less than 12 hours on 6 CPUs at 2.8 GHz for 40,000 pyrosequencing reads, so large samples require parallel resources
- ShoRAH2 has global haplotype reconstruction disabled; only the local (window) analysis runs by default, and ShoRAH1 v1.1.3 is the last version with global mode
- Hyperparameters alpha and the number of iterations must be set large enough to guarantee proper mixing of the Dirichlet-process Gibbs sampler; too-small values miss solutions with new clusters
- Software must be compiled and run from the command line; Linux/Mac OS X are the tested platforms, with WSL or Cygwin required for Windows

## Evidence

- [intro] This software corrects sequencing errors by clustering all reads that overlap the same region of the genome of length approximately equal to the read length: "This software corrects sequencing errors by clustering all reads that overlap the same region of the genome of length approximately equal to the read length"
- [methods] A five-fold decrease of the error rate, from 0.25% to 0.05%, was achieved: "A five-fold decrease of the error rate, from 0.25% to 0.05%, was achieved"
- [methods] Haplotypes with frequencies as low as 0.1% were detected at a coverage of 5000 and their frequency correctly estimated: "Haplotypes with frequencies as low as 0.1% were detected at a coverage of 5000 and their frequency correctly estimated"
- [readme] Gibbs sampling for error correction via Dirichlet process mixture: "diri_sampler | Gibbs sampling for error correction via Dirichlet process mixture"
- [readme] They work by cutting window from the multiple sequence alignment, invoking `diri_sampler` on the windows and calling `shorah snv` for the SNV calling.: "They work by cutting window from the multiple sequence alignment, invoking `diri_sampler` on the windows and calling `shorah snv` for the SNV calling."

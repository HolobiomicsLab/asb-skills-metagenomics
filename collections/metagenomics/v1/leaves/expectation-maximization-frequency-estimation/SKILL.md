---
name: expectation-maximization-frequency-estimation
description: Use when you have globally reconstructed viral haplotypes (e.g., from
  ShoRAH's mm.py) and need maximum-likelihood estimates of their population frequencies,
  including low-frequency variants.
when_to_use_negative:
- 'Running ShoRAH2 for global analysis: global haplotype reconstruction (and hence
  EM frequency estimation of global haplotypes) is disabled there; use ShoRAH1 v1.1.3
  or V-pipe''s global options'
- Only local haplotype/SNV calling is needed (shorah shotgun, shorah amplicon, or
  shorah snv) — frequency estimation of global haplotypes is unnecessary
- The sample is not genetically heterogeneous, so there is no mixed population to
  deconvolve
edam_operation: http://edamontology.org/operation_3197
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: ShoRAH
  role: Package whose four-step pipeline (alignment, error correction, global haplotype
    reconstruction, frequency estimation) includes the EM frequency estimation as
    the final step; run via the shorah.py wrapper
  repo: github.com/cbg-ethz/shorah
- name: freqEst
  role: Expectation Maximization program estimating haplotype frequencies by maximum
    likelihood (threshold 10^-6 on log-likelihood difference, max 5000 iterations)
  repo: github.com/cbg-ethz/shorah/tree/v1.1.3
- name: V-pipe
  role: Recommended workflow providing global haplotype options for performing global
    reconstruction and frequency estimation in place of ShoRAH1
  repo: github.com/cbg-ethz/V-pipe
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/1471-2105-12-119
    title: 'ShoRAH: estimating the genetic diversity of a mixed sample from next-generation
      sequencing data'
  source_content_hash: sha256:65c8831e63fddb3bd92f90e94bd78a2a525d95a800607930221b590443d5ffdf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - ShoRAH
  - V-pipe
  - freqEst
  - mm.py
  - shorah.py
  edam_operation: http://edamontology.org/operation_3197
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# expectation-maximization-frequency-estimation

## Summary

Estimates population frequencies of reconstructed global haplotypes from a mixed deep-sequencing sample using freqEst, an Expectation Maximization algorithm that finds maximum-likelihood haplotype abundances, the final step of the ShoRAH pipeline.

## When to use

Use when you have globally reconstructed viral haplotypes (e.g., from ShoRAH's mm.py) and need maximum-likelihood estimates of their population frequencies, including low-frequency variants.

## When NOT to use

- Running ShoRAH2 for global analysis: global haplotype reconstruction (and hence EM frequency estimation of global haplotypes) is disabled there; use ShoRAH1 v1.1.3 or V-pipe's global options
- Only local haplotype/SNV calling is needed (shorah shotgun, shorah amplicon, or shorah snv) — frequency estimation of global haplotypes is unnecessary
- The sample is not genetically heterogeneous, so there is no mixed population to deconvolve

## Inputs

- Error-corrected reads (output of ShoRAH's local error-correction stage)
- Reconstructed global haplotype sequences (output of mm.py maximum-weight-matching reconstruction)
- Sorted BAM/CRAM alignment file (input to the ShoRAH pipeline)

## Outputs

- haplotype_frequencies.tsv — reconstructed global haplotype sequences with their EM-estimated frequencies

## How to apply

After global haplotype reconstruction, run freqEst on the error-corrected reads and reconstructed haplotypes: the Expectation Maximization algorithm iteratively assigns reads to haplotypes and refines frequency estimates by maximum likelihood, using a convergence threshold of 10^-6 on the log-likelihood difference between successive iterations and a cap of 5000 iterations. This step follows alignment, error correction (local haplotype reconstruction), and global haplotype reconstruction in the four-step ShoRAH pipeline, and is invoked via the shorah.py wrapper in global mode. Note that global reconstruction and frequency estimation are only available in ShoRAH1 v1.1.3; ShoRAH2 has global mode disabled, so pin the old version or use V-pipe's global haplotype options. The rationale is that cluster/assignment-based read counts provide prevalence estimates, and EM refines them so haplotypes with frequencies as low as 0.1% at a coverage of 5000 are correctly estimated.

## Related tools

- **ShoRAH** (Package whose four-step pipeline (alignment, error correction, global haplotype reconstruction, frequency estimation) includes the EM frequency estimation as the final step; run via the shorah.py wrapper) — github.com/cbg-ethz/shorah
- **freqEst** (Expectation Maximization program estimating haplotype frequencies by maximum likelihood (threshold 10^-6 on log-likelihood difference, max 5000 iterations)) — github.com/cbg-ethz/shorah/tree/v1.1.3
- **V-pipe** (Recommended workflow providing global haplotype options for performing global reconstruction and frequency estimation in place of ShoRAH1) — github.com/cbg-ethz/V-pipe

## Examples

```
conda install shorah=1.1.3; freqEst
```

## Evaluation signals

- The EM run terminates either by convergence (log-likelihood difference below 10^-6) or after the 5000-iteration cap
- haplotype_frequencies.tsv is produced and haplotype frequencies resolve low-frequency variants (the paper reports detection and correct estimation of haplotypes at frequencies as low as 0.1% at coverage 5000)
- Estimated frequencies are consistent with read-support counts from the underlying clusters/assignments
- The run uses the global-mode branch of the pipeline (ShoRAH1 v1.1.3 or V-pipe global options), not ShoRAH2, which has global mode disabled

## Limitations

- Currently in ShoRAH2 global haplotype reconstruction is disabled; frequency estimation of global haplotypes requires ShoRAH1 v1.1.3 or the V-pipe global haplotype options
- Frequency estimation is only as good as the upstream error correction and global reconstruction; sequencing errors must be properly treated before EM assignment
- Frequency estimates for very low-frequency haplotypes depend on sufficient coverage (detection at 0.1% frequency was demonstrated only at coverage 5000)

## Evidence

- [methods] These are: 1) alignment; 2) error correction (local haplotype reconstruction); 3) global haplotype reconstruction and 4) frequency estimation: "These are: 1) alignment; 2) error correction (local haplotype reconstruction); 3) global haplotype reconstruction and 4) frequency estimation"
- [methods] Haplotypes with frequencies as low as 0.1% were detected at a coverage of 5000 and their frequency correctly estimated: "Haplotypes with frequencies as low as 0.1% were detected at a coverage of 5000 and their frequency correctly estimated"
- [intro] The number of reads associated with the cluster estimates the prevalence of the haplotype in the population: "The number of reads associated with the cluster estimates the prevalence of the haplotype in the population"
- [readme] Currently in ShoRAH2 global haplotype reconstruction is disabled.: "Currently in ShoRAH2 global haplotype reconstruction is disabled."

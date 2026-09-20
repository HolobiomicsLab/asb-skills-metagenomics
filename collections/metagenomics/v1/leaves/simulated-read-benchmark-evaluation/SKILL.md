---
name: simulated-read-benchmark-evaluation
description: Use when you must quantify a reconstruction method's sensitivity and
  accuracy on known ground truth, e.g., verifying that rare variants down to 0.1%
  frequency at ~5000x coverage are recovered from simulated reads.
when_to_use_negative:
- Only real wet-lab data are available and no ground-truth clone frequencies exist
  — sensitivity to a specific 0.1% variant cannot be verified without simulated or
  spiked-in mixtures
- The goal is global (full-length) haplotype reconstruction rather than local window-based
  evaluation
- Coverage is far below the ~5000x level at which 0.1%-frequency haplotype detection
  was validated in this setup
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0622
tools:
- name: ShoRAH
  role: 'Pipeline under test: local haplotype reconstruction and frequency estimation
    on the simulated reads'
  repo: http://www.cbg.ethz.ch/software/shorah
- name: shorah shotgun
  role: 'Runs the local analysis: cuts windows from the MSA and invokes diri_sampler
    on each window'
  repo: https://github.com/cbg-ethz/shorah
- name: Python
  role: Used for the simulation setup and window-splitting step (dec.py)
- name: diri_sampler
  role: Gibbs sampler with Dirichlet process mixture model applied per window for
    error correction / local haplotype reconstruction
  repo: https://github.com/cbg-ethz/shorah
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/1471-2105-12-119
    title: 'ShoRAH: estimating the genetic diversity of a mixed sample from next-generation
      sequencing data'
  source_content_hash: sha256:6e46d084537b2cca51df673cdece0f3ec5dae8e0a75bd0fd88e19e07271fa20e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - ShoRAH
  - diri_sampler
  - shorah shotgun
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# Simulated-read benchmark evaluation

## Summary

Benchmark a haplotype-inference pipeline by simulating NGS reads from a known ground-truth mixture of variants (e.g., 10 HIV-1 subtype B pol variants) and testing whether low-frequency clones are detected and correctly quantified against the known mixture proportions.

## When to use

Use when you must quantify a reconstruction method's sensitivity and accuracy on known ground truth, e.g., verifying that rare variants down to 0.1% frequency at ~5000x coverage are recovered from simulated reads.

## When NOT to use

- Only real wet-lab data are available and no ground-truth clone frequencies exist — sensitivity to a specific 0.1% variant cannot be verified without simulated or spiked-in mixtures
- The goal is global (full-length) haplotype reconstruction rather than local window-based evaluation
- Coverage is far below the ~5000x level at which 0.1%-frequency haplotype detection was validated in this setup

## Inputs

- Known set of ground-truth variant sequences (e.g., 10 HIV-1 subtype B pol gene variants)
- Ground-truth mixture proportions (including a 0.1%-frequency variant)
- Simulated Illumina reads aligned to the HIV-1 subtype B pol reference (sorted BAM/MSA input)
- HIV-1 subtype B pol reference sequence

## Outputs

- Inferred local haplotypes with estimated population frequencies
- Comparison of inferred vs. ground-truth frequencies (minimum detectable frequency, frequency estimation error)
- Error-rate assessment of the reconstruction (e.g., five-fold decrease of error rate from 0.25% to 0.05%)

## How to apply

Simulate sequencing reads from a known set of variants (here, 10 HIV-1 subtype B pol gene variants) at controlled coverage, then run the full inference pipeline exactly as for real data: align simulated reads to the HIV-1 subtype B pol reference, build the multiple sequence alignment, split it into overlapping windows (each position covered three times, window width approximately equal to the average read length) with dec.py, and run diri_sampler (Gibbs sampler with a Dirichlet process mixture model) per window, tuning the number of iterations and the hyper-parameter alpha for proper mixing. Correct the original reads by majority vote using the local reconstructions, aggregate cluster assignments into inferred haplotypes, and estimate frequencies from the number of reads per cluster. Compare inferred haplotypes and frequencies directly against the known ground-truth mixture proportions, focusing the acceptance criterion on the rarest variant (0.1% frequency at coverage 5000).

## Related tools

- **ShoRAH** (Pipeline under test: local haplotype reconstruction and frequency estimation on the simulated reads) — http://www.cbg.ethz.ch/software/shorah
- **shorah shotgun** (Runs the local analysis: cuts windows from the MSA and invokes diri_sampler on each window) — https://github.com/cbg-ethz/shorah
- **Python** (Used for the simulation setup and window-splitting step (dec.py))
- **diri_sampler** (Gibbs sampler with Dirichlet process mixture model applied per window for error correction / local haplotype reconstruction) — https://github.com/cbg-ethz/shorah

## Examples

```
shorah shotgun
```

## Evaluation signals

- Haplotypes at 0.1% frequency are detected at coverage 5000
- Estimated frequencies of detected haplotypes match the known ground-truth mixture proportions of the 10 simulated variants
- Every alignment position is covered by three overlapping windows with window width approximately equal to the average read length
- Post-correction error rate is substantially reduced relative to raw reads (paper reports 0.25% to 0.05%)
- diri_sampler mixing is adequate, verified by tuning iterations and hyper-parameter alpha

## Limitations

- Sensitivity result (0.1% at coverage 5000) is specific to the simulated dataset of 10 HIV-1 subtype B pol variants; it does not guarantee equivalent performance on real data with unmodeled error profiles
- diri_sampler requires careful tuning of the number of iterations and hyper-parameter alpha; insufficient tuning prevents proper mixing of the Gibbs sampler
- ShoRAH must be compiled and run from the command line; SAMtools and pysam must be obtained separately, and only Linux and Mac OS X platforms were tested

## Evidence

- [methods] Haplotypes with frequencies as low as 0.1% were detected at a coverage of 5000 and their frequency correctly estimated: "Haplotypes with frequencies as low as 0.1% were detected at a coverage of 5000 and their frequency correctly estimated"
- [methods] simulating reads from 10 different variants of the HIV-1 subtype B pol gene: "simulating reads from 10 different variants of the HIV-1 subtype B pol gene"
- [intro] number of reads associated with the cluster estimates the prevalence: "The number of reads associated with the cluster estimates the prevalence of the haplotype in the population"
- [methods] invoking diri_sampler on the windows: "invoking diri_sampler on the windows and calling shorah snv for the SNV calling"
- [readme] README: local analysis entry point: "The local analysis alone can be run invoking `shorah shotgun` or `shorah amplicon`"

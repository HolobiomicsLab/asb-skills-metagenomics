---
name: kernel-density-smoothing-of-length-histograms
description: Use when modeling coding ORF length distributions for gene-finding scoring,
  when an empirical histogram is accurate for short/medium ORFs but cannot model the
  long-ORF tail that a Gamma distribution captures.
when_to_use_negative:
- The task is predicting genes on error-prone metagenomic reads with indels/substitutions
  — that requires the full Glimmer-MG error model, not just a length distribution
- Input is a set of already-predicted genes rather than a curated annotation; the
  blend must be fit to trusted annotated ORF lengths
- Only very short reads/fragments are available with no reference genome for estimating
  the length prior
tools:
- name: Glimmer-MG
  role: The gene predictor whose gene-length probabilistic scoring model this blended
    KDE+Gamma distribution reproduces; Glimmer-MG 'incorporates a thorough probabilistic
    model for gene length' to handle truncated genes on short fragments
  repo: github.com/davek44/Glimmer-MG
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkr1067
    title: Gene prediction with Glimmer for metagenomic sequences augmented by classification
      and clustering
  source_content_hash: sha256:f5bc85e3f365f200f6e0d60ab1d08f8a8acbbbd4b5158bf70b3a19a836734bae
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/davek44/Glimmer-MG
  tools:
  - Glimmer-MG
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# Kernel-density smoothing of length histograms

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Blend a kernel-smoothed (KDE) histogram of annotated coding ORF lengths with a maximum-likelihood Gamma fit, interpolating between the first and third quartiles, to build Glimmer-MG's gene-length distribution used to score ORF lengths (Figure 1, D. radiodurans R1).

## When to use

Use when modeling coding ORF length distributions for gene-finding scoring, when an empirical histogram is accurate for short/medium ORFs but cannot model the long-ORF tail that a Gamma distribution captures.

## When NOT to use

- The task is predicting genes on error-prone metagenomic reads with indels/substitutions — that requires the full Glimmer-MG error model, not just a length distribution
- Input is a set of already-predicted genes rather than a curated annotation; the blend must be fit to trusted annotated ORF lengths
- Only very short reads/fragments are available with no reference genome for estimating the length prior

## Inputs

- Reference genome annotation in GenBank and FASTA format (e.g. Deinococcus radiodurans R1)
- List/array of annotated coding ORF lengths extracted from the annotation (via Biopython)

## Outputs

- Blended ORF length-distribution curve (KDE below first quartile, Gamma MLE above last quartile, linear interpolation between)
- Fitted Gamma parameters from scipy.stats.gamma.fit
- Kernel-smoothed density estimate (scipy.stats.gaussian_kde)
- Figure overlaying Gamma fit, kernel-smoothed histogram, and blend curves (figure1_length_distributions.png)
- Adjacent-gene orientation category frequencies and intergenic distance distributions (optional companion outputs)

## How to apply

Extract lengths of all annotated coding ORFs from a reference genome annotation (e.g. D. radiodurans R1 GenBank/FASTA) with Biopython. Fit a Gamma distribution by maximum likelihood with scipy.stats.gamma.fit, and compute a kernel-smoothed histogram with scipy.stats.gaussian_kde. Construct the blended length model: use the KDE curve for the first quartile of lengths, the Gamma MLE fit for the last quartile, and linearly interpolate between the two curves from the first to the third quartile. The rationale (per Glimmer-MG) is that the kernel-smoothed histogram gives greater accuracy for short and medium ORFs but is useless for very long ORFs, which the Gamma distribution models more effectively. Overlay Gamma fit, KDE, and blend curves in Matplotlib to reproduce the published Figure 1; optionally compute adjacent-gene orientation category frequencies and intergenic distance distributions from the same annotation.

## Related tools

- **Glimmer-MG** (The gene predictor whose gene-length probabilistic scoring model this blended KDE+Gamma distribution reproduces; Glimmer-MG 'incorporates a thorough probabilistic model for gene length' to handle truncated genes on short fragments) — github.com/davek44/Glimmer-MG

## Examples

```
from scipy.stats import gamma, gaussian_kde; g = gamma.fit(orf_lengths); kde = gaussian_kde(orf_lengths)
```

## Evaluation signals

- The Blend curve closely follows the empirical kernel-smoothed histogram in the low-length (first-quartile) region
- The Blend curve closely follows the Gamma MLE curve in the high-length (last-quartile) region
- The three overlaid curves (Gamma fit, KDE, Blend) are visually consistent with the published Figure 1 for D. radiodurans R1
- Gamma parameters converge under maximum-likelihood fit (scipy.stats.gamma.fit) and the KDE is finite/dense over the observed ORF-length range
- Output figure is saved as figure1_length_distributions.png with all three curves present

## Limitations

- The kernel-smoothed histogram is 'useless for very long ORFs', which is why the Gamma tail is spliced in above the last quartile
- This length model is only one component of Glimmer-MG's gene prediction; Glimmer-MG is computationally more expensive than existing software and prediction on error-prone sequences with indels remains difficult
- Glimmer-MG's probabilistic length model targets fragmented/partial genes on short metagenomic sequences, so the fitted distribution reflects genome annotation from a complete organism (e.g. D. radiodurans R1)

## Evidence

- [other] Blend component rationale: [evidence span withheld — anchor status: not_found]

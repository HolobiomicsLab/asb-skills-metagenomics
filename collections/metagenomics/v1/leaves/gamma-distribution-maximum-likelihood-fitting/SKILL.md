---
name: gamma-distribution-maximum-likelihood-fitting
description: 'Use when building or reproducing the Glimmer-MG gene-length model: you
  have annotated ORF lengths (e.g. from a GenBank/FASTA genome annotation) and need
  a blended coding/non-coding length distribution to score ORF lengths, especially
  for truncated genes on short metagenomic reads.'
when_to_use_negative:
- Predicting genes in complete, well-annotated prokaryotic genomes where standard
  Glimmer's IMM training on the whole genome is adequate — the length model exists
  to handle fragmented genes on short metagenomic reads
- The input is already a set of predicted genes/lengths and no length-distribution
  scoring model is needed
- The ORF-length data are not from coding regions, since the blended model is specifically
  a coding/non-coding length distribution
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Glimmer-MG
  role: The gene prediction system whose probabilistic gene-length model this fitting
    procedure reproduces; the Gamma/KDE blend underlies its scoring of ORF lengths
    for partial genes
  repo: https://github.com/davek44/Glimmer-MG
- name: Glimmer
  role: Base framework on which Glimmer-MG (and its length model context) is built
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkr1067
    title: Gene prediction with Glimmer for metagenomic sequences augmented by classification
      and clustering
  source_content_hash: sha256:08cb0794b804b688db93e0b7c2dd9f97630899dcc07025123f4349c00b360fdd
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/davek44/Glimmer-MG
  repo_url: https://github.com/davek44/Glimmer-MG
  tools:
  - Glimmer
  - Glimmer-MG
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# gamma-distribution-maximum-likelihood-fitting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fit a Gamma distribution by maximum likelihood to annotated coding ORF lengths and blend it with a kernel-smoothed histogram to reproduce Glimmer-MG's gene-length probability model. This models the length distribution used to score partial genes on short metagenomic fragments, as visualized in Figure 1 for Deinococcus radiodurans R1.

## When to use

Use when building or reproducing the Glimmer-MG gene-length model: you have annotated ORF lengths (e.g. from a GenBank/FASTA genome annotation) and need a blended coding/non-coding length distribution to score ORF lengths, especially for truncated genes on short metagenomic reads.

## When NOT to use

- Predicting genes in complete, well-annotated prokaryotic genomes where standard Glimmer's IMM training on the whole genome is adequate — the length model exists to handle fragmented genes on short metagenomic reads
- The input is already a set of predicted genes/lengths and no length-distribution scoring model is needed
- The ORF-length data are not from coding regions, since the blended model is specifically a coding/non-coding length distribution

## Inputs

- Genome annotation for a reference organism (e.g. Deinococcus radiodurans R1) in GenBank/FASTA format
- List of annotated coding ORF lengths extracted from the annotation (via Biopython)

## Outputs

- Gamma MLE parameters fitted to ORF lengths (scipy.stats.gamma.fit output)
- Kernel-smoothed histogram (KDE) of ORF lengths
- Blended coding/non-coding ORF-length distribution curve (first-quartile KDE, last-quartile Gamma, linear interpolation between Q1 and Q3)
- Overlay figure of Gamma fit, kernel-smoothed histogram, and blend curve (e.g. figure1_length_distributions.png) reproducing Figure 1

## How to apply

Extract the lengths of all annotated coding ORFs from the genome annotation with Biopython, then fit a Gamma distribution to those lengths by maximum-likelihood estimation using scipy.stats.gamma.fit. In parallel, compute a kernel-smoothed histogram of the ORF lengths (e.g. scipy.stats.gaussian_kde) to capture the empirical distribution. Construct the blended length model: use the kernel-smoothed histogram for the first quartile of lengths, the Gamma MLE fit for the last quartile, and linearly interpolate the two curves between the first and third quartiles. The rationale (per the paper) is that the kernel-smoothed histogram gives greater accuracy for short and medium-sized ORFs but is useless for very long ORFs, which the Gamma distribution can model more effectively. Finally, overlay the Gamma fit, the kernel-smoothed histogram, and the blended curve with Matplotlib to reproduce the D. radiodurans R1 length-distribution curves of Figure 1.

## Related tools

- **Glimmer-MG** (The gene prediction system whose probabilistic gene-length model this fitting procedure reproduces; the Gamma/KDE blend underlies its scoring of ORF lengths for partial genes) — https://github.com/davek44/Glimmer-MG
- **Glimmer** (Base framework on which Glimmer-MG (and its length model context) is built)

## Examples

```
from scipy.stats import gamma; params = gamma.fit(orf_lengths)
```

## Evaluation signals

- The Blend curve closely follows the empirical kernel-smoothed histogram in the low-length (first-quartile) region
- The Blend curve closely follows the Gamma MLE curve in the high-length (last-quartile) region
- The Gamma fit, kernel-smoothed histogram, and blended curve are visually consistent with the published Figure 1 for D. radiodurans R1
- The three curves are rendered on a single overlay figure and saved (e.g. figure1_length_distributions.png)
- The gamma.fit maximum-likelihood optimization converges on the ORF-length data without warnings or degenerate parameter estimates

## Limitations

- The kernel-smoothed histogram is useless for very long ORFs, which is why the Gamma distribution must model the high-length tail
- Glimmer-MG (whose length model this reproduces) is more computationally expensive than existing gene-prediction software
- The length model targets fragmented/partial genes on short reads; on error-prone sequences with indels, gene prediction remains difficult regardless of the length model

## Evidence

- [full_text] The blend motivates its components: the kernel-smoothed histogram gives greater accuracy for short and medium sized ORFs but is useless for very long ORFs, which the Gamma distribution can model more effectively.: [evidence span withheld — anchor status: not_found]

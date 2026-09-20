---
name: prokaryotic-orf-length-distribution-modeling
description: Use when reconstructing or validating the ORF length and adjacent-gene
  feature models behind a prokaryotic gene predictor's length-scoring term, or when
  an annotated genome's ORF lengths must be modeled as a blended empirical+parametric
  distribution.
when_to_use_negative:
- The goal is to run gene prediction on metagenomic reads rather than to reconstruct
  or validate the underlying length-distribution model — use Glimmer-MG's prediction
  pipeline directly instead.
- Input is raw sequencing reads without a genome annotation, so no trusted coding
  ORF lengths exist to fit the model to.
- Sequenced organism is not prokaryotic or the length model would need different parametric
  families (the blend rationale is specific to prokaryotic ORF length behavior).
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0102
tools:
- name: Glimmer-MG
  role: Gene prediction system whose probabilistic model for gene length (used to
    score ORF lengths and handle truncated genes on short metagenomic fragments) is
    reconstructed by this skill; its Figure 1 length-distribution curves are the validation
    target
  repo: https://github.com/davek44/Glimmer-MG
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkr1067
    title: Gene prediction with Glimmer for metagenomic sequences augmented by classification
      and clustering
  source_content_hash: sha256:70f5178981614b77872ab173e4d79bfd677780a04e851e6960b496b87f533e9b
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
  - Glimmer-MG
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# prokaryotic-orf-length-distribution-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs the blended coding/non-coding ORF length distribution that Glimmer-MG uses to score ORF lengths, combining a kernel-smoothed empirical histogram (accurate for short/medium ORFs) with a Gamma distribution MLE fit (accurate for very long ORFs), and reproduces the D. radiodurans R1 length-distribution curves of Figure 1.

## When to use

Use when reconstructing or validating the ORF length and adjacent-gene feature models behind a prokaryotic gene predictor's length-scoring term, or when an annotated genome's ORF lengths must be modeled as a blended empirical+parametric distribution.

## When NOT to use

- The goal is to run gene prediction on metagenomic reads rather than to reconstruct or validate the underlying length-distribution model — use Glimmer-MG's prediction pipeline directly instead.
- Input is raw sequencing reads without a genome annotation, so no trusted coding ORF lengths exist to fit the model to.
- Sequenced organism is not prokaryotic or the length model would need different parametric families (the blend rationale is specific to prokaryotic ORF length behavior).

## Inputs

- Annotated prokaryotic genome in GenBank/FASTA format (e.g., Deinococcus radiodurans R1)
- List of annotated coding ORF lengths extracted from the annotation (via Biopython)
- Genome annotation features for adjacent-gene orientation and intergenic distances

## Outputs

- Gamma MLE fit parameters for the ORF length distribution
- Kernel-smoothed histogram (Gaussian KDE) of ORF lengths
- Blended ORF length model curve (KDE below Q1, Gamma above Q3, linear interpolation between Q1 and Q3)
- Adjacent-gene orientation category frequencies and intergenic distance distributions
- figure1_length_distributions.png overlaying Gamma fit, KDE, and blended curves

## How to apply

Load a well-annotated prokaryotic genome (the paper's example is Deinococcus radiodurans R1, in GenBank/FASTA format) and extract annotated coding ORF lengths with Biopython. Fit a Gamma distribution to the ORF lengths by maximum-likelihood estimation (scipy.stats.gamma.fit) and compute a Gaussian KDE of the empirical lengths (scipy.stats.gaussian_kde). Construct the blend: use the kernel-smoothed histogram over the first quartile of lengths, the Gamma fit over the last quartile, and linearly interpolate the two curves between the first and third quartiles — the kernel estimate is more accurate for short and medium ORFs, while the Gamma model handles the very long ORF tail that the KDE cannot. From the same annotation, compute adjacent-gene orientation category frequencies and intergenic distance distributions. Overlay the Gamma fit, KDE, and blended curve in Matplotlib to reproduce Figure 1 and save as figure1_length_distributions.png.

## Related tools

- **Glimmer-MG** (Gene prediction system whose probabilistic model for gene length (used to score ORF lengths and handle truncated genes on short metagenomic fragments) is reconstructed by this skill; its Figure 1 length-distribution curves are the validation target) — https://github.com/davek44/Glimmer-MG

## Examples

```
from scipy.stats import gamma; a, loc, scale = gamma.fit(orf_lengths)
```

## Evaluation signals

- The Blend curve tracks the kernel-smoothed histogram closely in the low-length (first-quartile) region of the plot
- The Blend curve tracks the Gamma MLE curve in the high-length (last-quartile) region
- All three overlaid curves (Gamma fit, KDE, blend) are visually consistent with the published Figure 1 for D. radiodurans R1
- The blended curve is continuous, with a smooth linear transition between Q1 and Q3 and no discontinuities at the quartile boundaries
- Gamma fit via scipy.stats.gamma.fit returns finite, positive shape and scale parameters for the ORF length data

## Limitations

- The kernel-smoothed histogram component is useless for very long ORFs, which is precisely why the Gamma distribution is blended in for the last quartile
- Within Glimmer-MG's broader system, predicting genes on error-prone sequences, particularly those with indels, remains a difficult problem — a good length model alone does not resolve this
- Highly diverged organisms where the optimal model parameters are unclear present a challenge, so fits from a well-annotated model organism like D. radiodurans may not transfer directly

## Evidence

- [results] The blend motivates its components: the kernel-smoothed histogram gives greater accuracy for short and medium sized ORFs but is useless for very long ORFs, which the Gamma distribution can model more effectively.: [evidence span withheld — anchor status: not_found]

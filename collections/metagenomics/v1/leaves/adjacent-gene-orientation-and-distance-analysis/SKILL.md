---
name: adjacent-gene-orientation-and-distance-analysis
description: Use when building or reproducing gene-prediction length/start-site models
  that require the empirical frequencies of adjacent-gene orientations (same-strand
  vs. opposite-strand categories) and intergenic distance distributions from an annotated
  prokaryotic genome.
when_to_use_negative:
- Input is raw unassembled metagenomic reads with no reference annotation — Glimmer-MG
  parameterizes models from phylogenetic classification (Phymm) and clustering (Scimm),
  not from an annotated complete genome
- The goal is eukaryotic gene structure prediction with introns/exons — the adjacent-gene
  orientation and intergenic distance model is specific to prokaryotic operon-style
  gene layouts
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Glimmer-MG
  role: Gene prediction system whose gene-length and adjacent-gene models are parameterized
    by the orientation frequencies and intergenic distance distributions derived here
  repo: https://github.com/davek44/Glimmer-MG
- name: Glimmer
  role: Underlying framework using interpolated Markov models for coding region identification,
    extended by Glimmer-MG
  repo: https://github.com/davek44/Glimmer-MG
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkr1067
    title: Gene prediction with Glimmer for metagenomic sequences augmented by classification
      and clustering
  source_content_hash: sha256:91b76ea26e37e48e3182d723178ec067834ec0fc02d0762fc05e6ebd477634cb
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
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# Adjacent-gene orientation and distance analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts adjacent-gene orientation category frequencies and intergenic distance distributions from a genome annotation, as used by Glimmer-MG to model downstream and intervening adjacent gene pairs when scoring predicted ORFs on metagenomic fragments.

## When to use

Use when building or reproducing gene-prediction length/start-site models that require the empirical frequencies of adjacent-gene orientations (same-strand vs. opposite-strand categories) and intergenic distance distributions from an annotated prokaryotic genome.

## When NOT to use

- Input is raw unassembled metagenomic reads with no reference annotation — Glimmer-MG parameterizes models from phylogenetic classification (Phymm) and clustering (Scimm), not from an annotated complete genome
- The goal is eukaryotic gene structure prediction with introns/exons — the adjacent-gene orientation and intergenic distance model is specific to prokaryotic operon-style gene layouts

## Inputs

- Complete prokaryotic genome annotation (GenBank flatfile plus FASTA), e.g. Deinococcus radiodurans R1
- Annotated coding ORF coordinates, strands, and lengths extracted from the annotation
- Glimmer-MG gene prediction models (interpolated Markov models) to be parameterized with the derived distributions

## Outputs

- Table of adjacent-gene orientation category frequencies
- Empirical intergenic distance distribution (binned histogram)
- Blended ORF length model (kernel-smoothed histogram + Gamma MLE fit) and figure1_length_distributions.png reproducing Figure 1 curves

## How to apply

Load a complete annotated prokaryotic genome (e.g. Deinococcus radiodurans R1 in GenBank/FASTA) with Biopython and extract the coordinates, strand, and lengths of all annotated coding ORFs. Classify each pair of adjacent genes into an orientation category (e.g. same-direction downstream gene, convergent, divergent/intervening gene) and tabulate the category frequencies. Separately compute the distribution of intergenic distances between adjacent gene pairs, binning distances so they can be combined with the ORF length model (kernel-smoothed histogram blended with a scipy.stats.gamma maximum-likelihood fit) that Glimmer-MG uses to score partial and complete genes. These distributions parameterize the probabilistic gene length and start/stop codon model that Glimmer-MG applies to fragmented metagenomic reads. Validate by overlaying the empirical and fitted curves in Matplotlib and confirming consistency with Figure 1 of the paper.

## Related tools

- **Glimmer-MG** (Gene prediction system whose gene-length and adjacent-gene models are parameterized by the orientation frequencies and intergenic distance distributions derived here) — https://github.com/davek44/Glimmer-MG
- **Glimmer** (Underlying framework using interpolated Markov models for coding region identification, extended by Glimmer-MG) — https://github.com/davek44/Glimmer-MG

## Examples

```
from scipy.stats import gamma; params = gamma.fit(orf_lengths); kde = gaussian_kde(orf_lengths); blend(q1:q3) = interp(kde, gamma.pdf)
```

## Evaluation signals

- Blend curve follows the empirical kernel-smoothed histogram closely in the low-length (first-quartile) region
- Blend curve follows the Gamma maximum-likelihood fit in the high-length (last-quartile) region
- Orientation category frequencies sum to 1 across all adjacent gene pairs in the annotation
- Overlay of Gamma fit, kernel-smoothed histogram, and blended curve is visually consistent with the published Figure 1 for D. radiodurans R1
- Output figure saved as figure1_length_distributions.png with all three length-distribution curves present

## Limitations

- Glimmer-MG is more computationally expensive than existing gene prediction software
- Highly diverged organisms where the optimal model parameters are unclear present a challenge to metagenomics gene prediction
- Predicting genes on error-prone sequences, particularly those with indels, remains a difficult problem

## Evidence

- [intro] Probabilistic gene length model: "Glimmer-MG incorporates a thorough probabilistic model for gene length and start/stop codon presence to aid prediction of truncated genes on short sequence fragments"
- [readme] Annotation-derived training: "To account for fragmented genes, Glimmer-MG incorporates a model for gene length, in which partial genes are carefully handled."
- [readme] Training data dependence: "Glimmer-MG instead finds phylogenetic classifications using Phymm and parameterizes gene prediction models using those classifications."
- [abstract] Accuracy claim: "we developed a metagenomics gene prediction system Glimmer-MG that achieves significantly greater accuracy than previous systems"

---
name: permutational-multivariate-anova-on-distance-matrices
description: Use when sample groupings must be tested for significant differences
  in multivariate dispersion (e.g. allele-frequency or community profiles) and only
  per-sample distance matrices — not raw feature tables — are available for group-wise
  hypothesis testing.
when_to_use_negative:
- The question concerns alpha-level diversity within single samples (e.g. nucleotide
  diversity per gene) rather than between-group differences in multivariate profiles
- Variants are not shared informative variants — using noisy, non-polymorphic sites
  inflates false positives and masks group signal
- Sample groups have fewer replicates than feasible for permutation testing (permutation-based
  P-values will be unstable)
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: MetaQuad
  role: Flags shared informative variants (num_of_clusters >= 2, minSample = 5% of
    samples) used to build the allele-frequency matrix prior to distance computation
  repo: https://github.com/holab-hku/MetaQuad
- name: cellSNP-lite
  role: Piles up variant allele counts (recommended minMAF = 0.02, minCOUNT = 100)
    that feed MetaQuad and the ARG allele-frequency matrix
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: ggbiplot
  role: Renders the PCoA ordination of the Bray-Curtis matrix as an ordination biplot
    grouped by treatment
  repo: https://github.com/vqv/ggbiplot
- name: Python
  role: Implements the pairwise Bray-Curtis computation, PCoA, PERMANOVA, and multilevel
    pairwise.adonis with Bonferroni adjustment
  repo: https://github.com/holab-hku/MetaQuad
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbae030
    title: 'MetaQuad: shared informative variants discovery in metagenomic samples'
  source_content_hash: sha256:cb2ff419d0bfa52e697d332b19c8f656fffc2fb293486776427859298ff5a9fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/holab-hku/MetaQuad
  tools:
  - MetaQuad
  - Python
  - cellSNP-lite
  - ggbiplot
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Permutational multivariate ANOVA on distance matrices

## Summary

Applies permutational multivariate ANOVA (PERMANOVA) with multilevel pairwise.adonis and Bonferroni adjustment to a Bray-Curtis dissimilarity matrix of shared ARG allele frequencies, testing whether antibiotic treatment groups (LEVO, CLA, OTHERS) differ in beta diversity at 6 weeks post-treatment.

## When to use

Use when sample groupings must be tested for significant differences in multivariate dispersion (e.g. allele-frequency or community profiles) and only per-sample distance matrices — not raw feature tables — are available for group-wise hypothesis testing.

## When NOT to use

- The question concerns alpha-level diversity within single samples (e.g. nucleotide diversity per gene) rather than between-group differences in multivariate profiles
- Variants are not shared informative variants — using noisy, non-polymorphic sites inflates false positives and masks group signal
- Sample groups have fewer replicates than feasible for permutation testing (permutation-based P-values will be unstable)

## Inputs

- ARG allele-frequency matrix for T+6W samples derived from cellSNP.cells_ARG.vcf.gz
- Variant filter list of shared informative variants from MetaQuad BIC_params.csv (num_of_clusters >= 2)
- Sample-to-treatment-group metadata (CLA, LEVO, OTHERS)

## Outputs

- Bray-Curtis pairwise dissimilarity matrix between samples
- PCoA ordination biplot colored by treatment group (ggbiplot-style)
- Bonferroni-adjusted pairwise P-value table (pairwise_adonis_ARG_T6W.csv)

## How to apply

Build a pairwise Bray-Curtis dissimilarity matrix from the allele-frequency profiles of shared informative variants (MetaQuad-flagged with num_of_clusters >= 2 in BIC_params.csv) extracted from cellSNP.cells_ARG.vcf.gz for the T+6W samples. Compute a PCoA ordination on this matrix and visualize grouping by treatment (LEVO, CLA, OTHERS) with a ggbiplot-style biplot to check clustering before formal testing. Run permutational multivariate ANOVA across treatment groups, then multilevel pairwise.adonis for LEVO vs OTHERS and CLA vs OTHERS, adjusting P-values with Bonferroni correction. Correct application is confirmed when the adjusted P-values reproduce the expected result (P = 0.03 LEVO vs OTHERS; P = 0.003 CLA vs OTHERS at T+6W) and no comparison is significant at other time points.

## Related tools

- **MetaQuad** (Flags shared informative variants (num_of_clusters >= 2, minSample = 5% of samples) used to build the allele-frequency matrix prior to distance computation) — https://github.com/holab-hku/MetaQuad
- **cellSNP-lite** (Piles up variant allele counts (recommended minMAF = 0.02, minCOUNT = 100) that feed MetaQuad and the ARG allele-frequency matrix) — https://github.com/single-cell-genetics/cellsnp-lite
- **ggbiplot** (Renders the PCoA ordination of the Bray-Curtis matrix as an ordination biplot grouped by treatment) — https://github.com/vqv/ggbiplot
- **Python** (Implements the pairwise Bray-Curtis computation, PCoA, PERMANOVA, and multilevel pairwise.adonis with Bonferroni adjustment) — https://github.com/holab-hku/MetaQuad

## Examples

```
metaquad -i cellSNP -o Metaquad --minSample=5
```

## Evaluation signals

- Adjusted P-values reproduce the published result: P = 0.03 for LEVO vs OTHERS and P = 0.003 for CLA vs OTHERS at T+6W
- No pairwise comparison is significant at time points other than T+6W
- pairwise_adonis_ARG_T6W.csv contains a complete LEVO/CLA/OTHERS comparison table with Bonferroni-adjusted P-values
- PCoA biplot visually separates CLA and LEVO samples from OTHERS at T+6W, consistent with the significant adonis results
- Variant set restricted to MetaQuad-passed variants with num_of_clusters >= 2 from BIC_params.csv

## Limitations

- Significance was time-point-specific: group differentiation was only observed at T+6W, so applying the test at other time points yields non-significant results
- P-values rely on permutation, so results depend on group sizes and permutation count; Bonferroni adjustment across multiple pairwise tests is required to control family-wise error
- Results are specific to shared informative ARG variants detected by MetaQuad; using all pileup variants would change the distance structure

## Evidence

- [results] At T+6W, multilevel pairwise comparison showed significant differences for LEVO versus OTHER (adjusted P = 0.03) and CLA versus OTHER (adjusted P = 0.003), whereas no comparison was significant at the other time points.: [evidence span withheld — anchor status: not_found]
- [intro] MetaQuad informative variant parameters: "Detect informative mutations using MetaQuad, with the following recommended parameters: minSample = 5% of the total sample size and num_of_clusters >= 2."
- [intro] cellsnp-lite pileup parameters: [evidence span withheld — anchor status: not_found]
- [abstract] MetaQuad false positive reduction: "MetaQuad significantly reduces the number of false positive SNPs without greatly affecting the true positive rate."
- [readme] MetaQuad density clustering rationale: "It uses a density clustering model to assess the informative variants among background noise."

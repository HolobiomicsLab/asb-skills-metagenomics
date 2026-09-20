---
name: allele-frequency-dissimilarity-computation
description: Use when comparing treatment groups (e.g. CLA, LEVO vs OTHERS) at a time
  point such as T+6W using allele-frequency profiles of shared informative variants
  from a cellSNP-lite/MetaQuad pipeline on ARG VCF data.
when_to_use_negative:
- The question is gene-level nucleotide diversity of ARGs rather than between-sample
  allele-frequency dissimilarity — use the nucleotide_diversity_code.R workflow instead.
- The variant set lacks shared informative variants (e.g. fewer than the minSample
  = 5% threshold or num_of_clusters < 2), so no informative dissimilarity signal exists
  between groups.
- Input is raw shotgun metagenomic reads not yet filtered/aligned (bwa/bowtie2) and
  piled up with cellsnp-lite (minMAF = 0.02, minCOUNT = 100) — the dissimilarity step
  presupposes the allele-frequency matrix.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MetaQuad
  role: Detects shared informative variants via density clustering; its num_of_clusters
    >= 2 flags (from BIC_params.csv) define which variants enter the allele-frequency
    matrix
  repo: https://github.com/holab-hku/MetaQuad.git
- name: cellSNP-lite
  role: Piles up variants to produce the SNP x sample AD/DP matrices and cellSNP.cells_ARG.vcf.gz
    allele-frequency input
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: ggbiplot
  role: Renders the PCoA ordination biplot of samples grouped by antibiotic treatment
  repo: https://github.com/vqv/ggbiplot
- name: Python
  role: Implements the Bray-Curtis dissimilarity computation, PCoA ordination, and
    PERMANOVA / multilevel pairwise.adonis testing
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbae030
    title: 'MetaQuad: shared informative variants discovery in metagenomic samples'
  source_content_hash: sha256:7de0b929876d171a2a3b795fc46ca116cbebfcff47d72f3c8875a1b213fcef9e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/holab-hku/MetaQuad.git
  tools:
  - MetaQuad
  - Python
  - cellSNP-lite
  - ggbiplot
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# allele-frequency-dissimilarity-computation

## Summary

Compute between-sample dissimilarities from allele-frequency profiles of shared informative variants (MetaQuad-flagged, num_of_clusters >= 2) detected across ARG loci, then ordinate with PCoA and test treatment-group differentiation using PERMANOVA with multilevel pairwise.adonis and Bonferroni-adjusted P-values.

## When to use

Use when comparing treatment groups (e.g. CLA, LEVO vs OTHERS) at a time point such as T+6W using allele-frequency profiles of shared informative variants from a cellSNP-lite/MetaQuad pipeline on ARG VCF data.

## When NOT to use

- The question is gene-level nucleotide diversity of ARGs rather than between-sample allele-frequency dissimilarity — use the nucleotide_diversity_code.R workflow instead.
- The variant set lacks shared informative variants (e.g. fewer than the minSample = 5% threshold or num_of_clusters < 2), so no informative dissimilarity signal exists between groups.
- Input is raw shotgun metagenomic reads not yet filtered/aligned (bwa/bowtie2) and piled up with cellsnp-lite (minMAF = 0.02, minCOUNT = 100) — the dissimilarity step presupposes the allele-frequency matrix.

## Inputs

- cellSNP.cells_ARG.vcf.gz — ARG allele-frequency (SNP x sample) matrix from cellsnp-lite
- BIC_params.csv — MetaQuad output with per-variant statistics used to filter num_of_clusters >= 2
- Sample metadata mapping samples to antibiotic treatment groups (CLA, LEVO, OTHERS) and time points (e.g. T+6W)

## Outputs

- Pairwise Bray-Curtis dissimilarity matrix between samples
- PCoA ordination biplot coloured/grouped by treatment (ggbiplot)
- pairwise_adonis_ARG_T6W.csv — table of Bonferroni-adjusted pairwise P-values per time point

## How to apply

Load the ARG allele-frequency matrix from cellSNP.cells_ARG.vcf.gz for the samples at the target time point (e.g. T+6W) and restrict to shared informative variants flagged by MetaQuad with num_of_clusters >= 2 in BIC_params.csv (variants were detected with minSample = 5% of the total sample size). Compute all pairwise Bray-Curtis dissimilarities between samples from their allele-frequency profiles. Run PCoA ordination on the resulting distance matrix and render the grouping by treatment using a ggbiplot-style ordination biplot. Apply permutational multivariate ANOVA (PERMANOVA) followed by multilevel pairwise.adonis across treatment groups (LEVO vs OTHERS, CLA vs OTHERS) with Bonferroni-adjusted P-values, writing the results to pairwise_adonis_ARG_T6W.csv. Judge success by reproducing the paper's pattern: at T+6W, adjusted P = 0.03 for LEVO vs OTHER and adjusted P = 0.003 for CLA vs OTHER, with no significant comparison at other time points.

## Related tools

- **MetaQuad** (Detects shared informative variants via density clustering; its num_of_clusters >= 2 flags (from BIC_params.csv) define which variants enter the allele-frequency matrix) — https://github.com/holab-hku/MetaQuad.git
- **cellSNP-lite** (Piles up variants to produce the SNP x sample AD/DP matrices and cellSNP.cells_ARG.vcf.gz allele-frequency input) — https://github.com/single-cell-genetics/cellsnp-lite
- **ggbiplot** (Renders the PCoA ordination biplot of samples grouped by antibiotic treatment) — https://github.com/vqv/ggbiplot
- **Python** (Implements the Bray-Curtis dissimilarity computation, PCoA ordination, and PERMANOVA / multilevel pairwise.adonis testing)

## Examples

```
metaquad -i cellSNP -o Metaquad --minSample=5
```

## Evaluation signals

- Bonferroni-adjusted pairwise P-values at T+6W reproduce adjusted P = 0.03 (LEVO vs OTHER) and P = 0.003 (CLA vs OTHER), with non-significance at all other time points
- Bray-Curtis distance matrix is symmetric with a zero diagonal and dimension n_samples x n_samples
- Only variants with num_of_clusters >= 2 in BIC_params.csv (MetaQuad-informative) are retained before computing dissimilarities
- PCoA biplot renders sample points grouped by CLA / LEVO / OTHERS treatment assignment
- Variant detection upstream used the recommended thresholds: cellsnp-lite minMAF = 0.02 and minCOUNT = 100; MetaQuad minSample = 5% of total sample size

## Limitations

- Variant informativeness depends on MetaQuad's density-clustering call; false positives/negatives in num_of_clusters flags propagate into the dissimilarity analysis
- cellsnp-lite upstream was described as a faster version of cellSNP 'still at testing stage so might be unstable'
- ggbiplot's active development has moved to github.com/friendly/ggbiplot, so the pinned vqv/ggbiplot version may be unmaintained
- Pairwise.adonis inference is permutation-based and relies on Bonferroni adjustment; with few samples per treatment group power is limited, as seen by non-significant comparisons at all time points except T+6W

## Evidence

- [abstract] MetaQuad detected 7591 variants across 529 antibiotic resistance genes in patients who underwent Helicobacter pylori eradication therapy.: [evidence span withheld — anchor status: not_found]
- [intro] MetaQuad recommended detection parameters: "Detect informative mutations using MetaQuad, with the following recommended parameters: minSample = 5% of the total sample size and num_of_clusters >= 2."
- [readme] cellsnp-lite pile-up thresholds: "Pile up variants using [cellsnp-lite](https://github.com/single-cell-genetics/cellsnp-lite). <br/> with the following recommended parameters: minMAF = 0.02 and minCOUNT = 100."
- [readme] MetaQuad density-clustering rationale: "It uses a density clustering model to assess the informative variants among background noise."
- [readme] ggbiplot maintenance caveat: [evidence span withheld — anchor status: external_doc]

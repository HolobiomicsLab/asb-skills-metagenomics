---
name: pcoa-ordination-of-variant-frequencies
description: Use when you have a sample-by-variant allele-frequency matrix (e.g.,
  from cellSNP.cells_ARG.vcf.gz with MetaQuad num_of_clusters >= 2 variants) and need
  to test whether predefined sample groups (e.g., antibiotic treatments) differ in
  variant composition at a given time point.
when_to_use_negative:
- The variants have not been filtered for informativeness — raw cellSNP pileups containing
  non-polymorphic/background noise sites will inflate dissimilarities spuriously;
  use MetaQuad's num_of_clusters >= 2 flag first.
- The goal is detecting which variants are informative in the first place — that is
  MetaQuad's density-clustering step, not the downstream ordination and group-comparison
  step.
- Sample groups lack replication (single sample per group), making PERMANOVA / pairwise.adonis
  testing undefined or underpowered.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: MetaQuad
  role: Detects shared informative variants (density clustering, num_of_clusters >=
    2, minSample = 5% of total sample size) that define the allele-frequency profiles
    used in the PCoA
  repo: https://github.com/holab-hku/MetaQuad.git
- name: cellSNP-lite
  role: Piles up variants (recommended minMAF = 0.02, minCOUNT = 100) producing the
    cellSNP output folder consumed by MetaQuad and providing the ARG allele-frequency
    matrix
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: ggbiplot
  role: R package used to render the PCoA ordination biplot with samples grouped by
    antibiotic treatment
  repo: https://github.com/vqv/ggbiplot
- name: Python
  role: Language in which MetaQuad is implemented and used to compute Bray-Curtis
    dissimilarities and run PERMANOVA / pairwise.adonis on the filtered allele-frequency
    matrix
  repo: https://github.com/holab-hku/MetaQuad.git
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbae030
    title: 'MetaQuad: shared informative variants discovery in metagenomic samples'
  source_content_hash: sha256:7c1876c406d4e64350a835b5235e492f11e4daf6eaab51ee58ab3b59792a17ac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/holab-hku/MetaQuad.git
  tools:
  - MetaQuad
  - Python
  - cellSNP-lite
  - ggbiplot
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# PCoA ordination of variant frequencies

## Summary

Ordinates samples by Bray-Curtis dissimilarity computed from shared informative ARG allele-frequency profiles (MetaQuad-filtered cellSNP VCF) using PCoA, and tests treatment-group separation with PERMANOVA and multilevel pairwise.adonis with Bonferroni adjustment.

## When to use

Use when you have a sample-by-variant allele-frequency matrix (e.g., from cellSNP.cells_ARG.vcf.gz with MetaQuad num_of_clusters >= 2 variants) and need to test whether predefined sample groups (e.g., antibiotic treatments) differ in variant composition at a given time point.

## When NOT to use

- The variants have not been filtered for informativeness — raw cellSNP pileups containing non-polymorphic/background noise sites will inflate dissimilarities spuriously; use MetaQuad's num_of_clusters >= 2 flag first.
- The goal is detecting which variants are informative in the first place — that is MetaQuad's density-clustering step, not the downstream ordination and group-comparison step.
- Sample groups lack replication (single sample per group), making PERMANOVA / pairwise.adonis testing undefined or underpowered.

## Inputs

- cellSNP.cells_ARG.vcf.gz — allele-frequency pileup of ARG variants per sample
- BIC_params.csv — MetaQuad output used to filter shared informative variants (num_of_clusters >= 2)
- Sample metadata table mapping samples to antibiotic treatment groups (CLA, LEVO, OTHERS) and time points (e.g., T+6W)

## Outputs

- Bray-Curtis pairwise distance matrix between samples
- PCoA ordination biplot colored by treatment group (ggbiplot-style)
- pairwise_adonis_ARG_T6W.csv — table of multilevel pairwise.adonis P-values with Bonferroni adjustment per time point

## How to apply

Subset the ARG allele-frequency matrix to the samples of the target time point (e.g., T+6W), retaining only shared informative variants flagged by MetaQuad with num_of_clusters >= 2 in BIC_params.csv. Compute pairwise Bray-Curtis dissimilarities between samples from their allele-frequency profiles, then run principal coordinates analysis (PCoA) on the resulting distance matrix and render sample grouping by treatment (CLA, LEVO, OTHERS) as a ggbiplot-style ordination biplot. Quantify group separation with permutational multivariate ANOVA (PERMANOVA) and multilevel pairwise.adonis across treatment groups with Bonferroni-adjusted P-values, writing the adjusted P-values to a table such as pairwise_adonis_ARG_T6W.csv. Interpretation follows the article's finding: at T+6W, LEVO versus OTHER (adjusted P = 0.03) and CLA versus OTHER (adjusted P = 0.003) were significant, while no comparison was significant at other time points.

## Related tools

- **MetaQuad** (Detects shared informative variants (density clustering, num_of_clusters >= 2, minSample = 5% of total sample size) that define the allele-frequency profiles used in the PCoA) — https://github.com/holab-hku/MetaQuad.git
- **cellSNP-lite** (Piles up variants (recommended minMAF = 0.02, minCOUNT = 100) producing the cellSNP output folder consumed by MetaQuad and providing the ARG allele-frequency matrix) — https://github.com/single-cell-genetics/cellsnp-lite
- **ggbiplot** (R package used to render the PCoA ordination biplot with samples grouped by antibiotic treatment) — https://github.com/vqv/ggbiplot
- **Python** (Language in which MetaQuad is implemented and used to compute Bray-Curtis dissimilarities and run PERMANOVA / pairwise.adonis on the filtered allele-frequency matrix) — https://github.com/holab-hku/MetaQuad.git

## Examples

```
metaquad -i cellSNP -o Metaquad --minSample=5
```

## Evaluation signals

- The Bonferroni-adjusted pairwise.adonis table reproduces adjusted P = 0.03 for LEVO vs OTHERS and P = 0.003 for CLA vs OTHERS at T+6W.
- No pairwise comparison between treatment groups is significant at time points other than T+6W (non-significance at all other time points is expected).
- The distance matrix contains one Bray-Curtis dissimilarity per sample pair and is symmetric with zero diagonal.
- The PCoA biplot shows samples colored/grouped by treatment (CLA, LEVO, OTHERS) using only variants with num_of_clusters >= 2 in BIC_params.csv.
- Variant filtering matches MetaQuad's recommended minSample = 5% of the total sample size and num_of_clusters >= 2.

## Limitations

- Group separation was time-point specific: no comparison was significant at the other time points, so a null PERMANOVA result at other time points is the expected outcome, not a failure.
- Analysis depends on upstream variant-calling quality; MetaQuad's density clustering is required to distinguish informative variants from background noise, and false positives/negatives propagate into the distance matrix.
- cellsnp-lite is noted as 'still at testing stage so might be unstable' in the MQuad README context, which may affect pileup reproducibility.

## Evidence

- [results] At T+6W, multilevel pairwise comparison showed significant differences for LEVO versus OTHER (adjusted P = 0.03) and CLA versus OTHER (adjusted P = 0.003), whereas no comparison was significant at the other time points.: [evidence span withheld — anchor status: not_found]
- [intro] MetaQuad recommends informative-variant parameters: "Detect informative mutations using MetaQuad, with the following recommended parameters: minSample = 5% of the total sample size and num_of_clusters >= 2."
- [readme] cellSNP pileup parameters: "Pile up variants using [cellsnp-lite](https://github.com/single-cell-genetics/cellsnp-lite). <br/> with the following recommended parameters: minMAF = 0.02 and minCOUNT = 100."
- [abstract] MetaQuad uses density clustering to separate informative variants from noise: "MetaQuad utilizes a density-based clustering technique to effectively distinguish between shared variants and non-polymorphic sites using shotgun metagenomic data."
- [methods] MetaQuad run command on cellSNP output: "metaquad  -i cellSNP -o Metaquad --minSample=5"

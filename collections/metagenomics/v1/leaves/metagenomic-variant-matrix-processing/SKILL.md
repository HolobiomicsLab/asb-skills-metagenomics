---
name: metagenomic-variant-matrix-processing
description: Use when you have per-sample AD/DP allele counts from cellsnp-lite over
  shotgun metagenomic (e.g., ARG) loci and need to filter MetaQuad informative variants
  (num_of_clusters >= 2 in BIC_params.csv) into an AF matrix for Bray-Curtis PCoA
  and pairwise.adonis group comparison.
when_to_use_negative:
- Data are single-sample or single-cell without a population of samples — OPTICS clustering
  needs per-sample allele frequencies across samples to distinguish shared variants
  from noise
- Variants fail the cellsnp-lite quality gates (minMAF = 0.02, minCOUNT = 100) or
  have insufficient non-zero-DP samples — they are noise, not shared informative variants
- The goal is de novo variant discovery per isolate rather than strain-level differentiation
  across a metagenomic sample population (use standard variant callers instead)
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3168
tools:
- name: MetaQuad
  role: Detects shared informative variants by 1-D OPTICS density clustering of per-sample
    allele frequencies; flags variants with num_of_clusters >= 2 and writes BIC_params.csv
  repo: https://github.com/holab-hku/MetaQuad
- name: cellsnp-lite
  role: Piles up per-sample AD/DP allele counts from aligned shotgun metagenomic reads
    (recommended minMAF = 0.02, minCOUNT = 100) to feed MetaQuad
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: MQuad
  role: Upstream model from which MetaQuad is extended for shotgun metagenomic data;
    defines the BIC_params.csv output convention
  repo: https://github.com/single-cell-genetics/MQuad
- name: ggbiplot
  role: R package used to render the PCoA ordination biplot of samples grouped by
    antibiotic treatment
  repo: https://github.com/vqv/ggbiplot
- name: bwa
  role: Aligns filtered shotgun metagenomic reads to a reference database, producing
    .filtered.sorted.bam inputs for cellsnp-lite pileup
provenance:
  source_task_ids:
  - task_008
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae030
    title: 'MetaQuad: shared informative variants discovery in metagenomic samples'
  source_content_hash: sha256:6a7da564fb7eb4614b753403a6190568e36fb817a0d4153f6a225bce5b0adee6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/holab-hku/MetaQuad
  tools:
  - MQuad
  - MetaQuad
  - Python
  - bwa
  - cellsnp-lite
  - ggbiplot
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# metagenomic-variant-matrix-processing

## Summary

Turns cellsnp-lite allele-count pileups from shotgun metagenomic samples into an allele-frequency matrix restricted to MetaQuad-flagged shared informative variants (num_of_clusters >= 2), enabling downstream ordination and between-group testing such as the T+6W antibiotic-treatment PCoA/PERMANOVA analysis.

## When to use

Use when you have per-sample AD/DP allele counts from cellsnp-lite over shotgun metagenomic (e.g., ARG) loci and need to filter MetaQuad informative variants (num_of_clusters >= 2 in BIC_params.csv) into an AF matrix for Bray-Curtis PCoA and pairwise.adonis group comparison.

## When NOT to use

- Data are single-sample or single-cell without a population of samples — OPTICS clustering needs per-sample allele frequencies across samples to distinguish shared variants from noise
- Variants fail the cellsnp-lite quality gates (minMAF = 0.02, minCOUNT = 100) or have insufficient non-zero-DP samples — they are noise, not shared informative variants
- The goal is de novo variant discovery per isolate rather than strain-level differentiation across a metagenomic sample population (use standard variant callers instead)

## Inputs

- cellsnp-lite output folder with per-sample AD and DP allele-count sparse matrices (.mtx) or cellSNP.cells.vcf.gz / cellSNP.cells_ARG.vcf.gz pileups
- BIC_params.csv from MetaQuad containing variant_names, num_of_clusters, mean_DP, mean_AD, and number_DP columns
- bwa- or bowtie2-aligned .filtered.sorted.bam files from shotgun metagenomic sequencing

## Outputs

- Filtered sample x variant allele-frequency matrix of shared informative variants (num_of_clusters >= 2)
- Bray-Curtis dissimilarity matrix and PCoA ordination biplot grouped by treatment (CLA, LEVO, OTHERS)
- Bonferroni-adjusted multilevel pairwise.adonis P-value table (e.g., pairwise_adonis_ARG_T6W.csv)
- BIC_params.csv with per-variant num_of_clusters, mean_DP, mean_AD, and number_DP

## How to apply

First pile up variants with cellsnp-lite using the recommended parameters minMAF = 0.02 and minCOUNT = 100 on bwa/bowtie2-aligned '.filtered.sorted.bam' files, then run MetaQuad on the cellSNP output folder with minSample = 5% of the total sample size so 1-D OPTICS density clustering assigns each variant a num_of_clusters value. Join the per-sample AD/DP counts (e.g., from cellSNP.cells_ARG.vcf.gz) with BIC_params.csv and retain only shared informative variants with num_of_clusters >= 2, computing each sample's alternate-allele frequency as AD/DP. For a given time point (e.g., T+6W after H. pylori eradication therapy), build the sample x variant allele-frequency matrix and compute pairwise Bray-Curtis dissimilarities. Ordinate the distance matrix with PCoA and render treatment-group structure (CLA, LEVO, OTHERS) with a ggbiplot-style biplot. Test group differentiation with permutational multivariate ANOVA plus multilevel pairwise.adonis (LEVO vs OTHERS, CLA vs OTHERS) using Bonferroni-adjusted P-values, writing results to a table such as pairwise_adonis_ARG_T6W.csv.

## Related tools

- **MetaQuad** (Detects shared informative variants by 1-D OPTICS density clustering of per-sample allele frequencies; flags variants with num_of_clusters >= 2 and writes BIC_params.csv) — https://github.com/holab-hku/MetaQuad
- **cellsnp-lite** (Piles up per-sample AD/DP allele counts from aligned shotgun metagenomic reads (recommended minMAF = 0.02, minCOUNT = 100) to feed MetaQuad) — https://github.com/single-cell-genetics/cellsnp-lite
- **MQuad** (Upstream model from which MetaQuad is extended for shotgun metagenomic data; defines the BIC_params.csv output convention) — https://github.com/single-cell-genetics/MQuad
- **ggbiplot** (R package used to render the PCoA ordination biplot of samples grouped by antibiotic treatment) — https://github.com/vqv/ggbiplot
- **bwa** (Aligns filtered shotgun metagenomic reads to a reference database, producing .filtered.sorted.bam inputs for cellsnp-lite pileup)

## Examples

```
metaquad -i cellSNP -o Metaquad --minSample=5
```

## Evaluation signals

- BIC_params.csv exists and contains the required columns variant_names, num_of_clusters, mean_DP, mean_AD, and number_DP, with every num_of_clusters a non-negative integer
- All retained variants satisfy num_of_clusters >= 2 (shared informative), while noise points from OPTICS are excluded
- Per-sample allele frequencies reproduce AD/DP ratios for the retained variants, and number_DP counts non-zero-DP samples correctly
- Pairwise.adonis at T+6W reproduces Bonferroni-adjusted P = 0.03 for LEVO vs OTHERS and P = 0.003 for CLA vs OTHERS, with no significant comparison at other time points
- In the published application, MetaQuad detected 7591 variants across 529 antibiotic resistance genes — orders of magnitude that can sanity-check scale on comparable data

## Limitations

- MetaQuad's OPTICS clustering enforces minSample (recommended 5% of total sample size) as a minimum cluster size, so variants present in fewer samples are labelled noise and lost
- cellsnp-lite minMAF/minCOUNT thresholds (recommended 0.02/100) trade sensitivity for computation time; a higher value is recommended to reduce computation time for subsequent steps
- MetaQuad reduces false positive SNPs 'without greatly affecting the true positive rate' — informative-variant calls are statistical clusterings, not validated genotype truth
- Group-difference significance (e.g., T+6W LEVO/CLA vs OTHERS) is time-point specific and may not hold at other sampling points

## Evidence

- [abstract] MetaQuad utilizes a density-based clustering technique to effectively distinguish between shared variants and non-polymorphic sites using shotgun metagenomic data.: "MetaQuad utilizes a density-based clustering technique to effectively distinguish between shared variants and non-polymorphic sites using shotgun metagenomic data."
- [intro] Detect informative mutations using MetaQuad, with the following recommended parameters: minSample = 5% of the total sample size and num_of_clusters >= 2.: "Detect informative mutations using MetaQuad, with the following recommended parameters: minSample = 5% of the total sample size and num_of_clusters >= 2."
- [methods] MetaQuad takes the output folder of cellSNP-lite: "MetaQuad takes the output folder of cellSNP-lite"
- [readme] Pile up variants using cellsnp-lite with the following recommended parameters: minMAF = 0.02 and minCOUNT = 100.: "Pile up variants using [cellsnp-lite](https://github.com/single-cell-genetics/cellsnp-lite). <br/> with the following recommended parameters: minMAF = 0.02 and minCOUNT = 100."

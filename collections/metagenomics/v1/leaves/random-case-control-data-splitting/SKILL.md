---
name: random-case-control-data-splitting
description: Use when validating that a differential abundance test on real microbiome
  relative abundance data (e.g. 16S taxa tables from the Bangladesh cohort) controls
  false positive rates by permuting artificial group labels under the null of no group
  effect.
when_to_use_negative:
- When the data contain a real, biologically defined group comparison (e.g. gender
  or breastfeeding status) — the split must be artificial to test null-hypothesis
  type I error control
- When the goal is effect-size estimation or meta-analysis pooling of estimates across
  studies rather than false-positive rate calibration
- When the input is a feature/abundance table from a non-microbiome data type that
  does not resemble a zero-inflated beta distribution
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: metamicrobiomeR
  role: Provides taxa.filter, taxa.compare (with propmed.rel="gamlss" invoking GAMLSS
    zero-inflated beta family), and the datasets taxtab.rm7 and sam.rm used to run
    the per-split differential abundance tests
  repo: https://github.com/nhanhocu/metamicrobiomeR
- name: R
  role: Implementation language; R 3.4.2 or higher required; random splits, iteration
    loop, and type I error summarization are scripted in R
- name: GAMLSS-BEZI
  role: Zero-inflated beta GAMLSS family used by taxa.compare to test differential
    relative abundance under each artificial case/control split
  repo: https://github.com/nhanhocu/metamicrobiomeR
- name: MaAsLin
  role: 'Reference linear-model (LMAS) comparator: arcsin square root transformed
    relative abundances tested with linear models or linear mixed models with random
    intercepts for the longitudinal design'
- name: knitr
  role: Used with kable() to render result tables in the package workflow examples
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-019-2744-2
    title: 'metamicrobiomeR: an R package for analysis of microbiome relative abundance
      data using zero-inflated beta GAMLSS and meta-analysis across studies using
      random effects models'
  source_content_hash: sha256:79ba6f59b634d26f2197760c22c88f52f6adae923f06ec4187ac84110bdf17b4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nhanhocu/metamicrobiomeR
  tools:
  - GAMLSS-BEZI
  - MaAsLin
  - R
  - knitr
  - metamicrobiomeR
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# random-case-control-data-splitting

## Summary

Empirical type I error assessment for microbiome differential abundance methods: repeatedly assign artificial case/control labels at random (preserving cross-sectional or longitudinal structure), re-run the testing pipeline (GAMLSS-BEZI via metamicrobiomeR::taxa.compare and LMAS), and summarize the proportion of taxa with p < 0.05 per iteration.

## When to use

Use when validating that a differential abundance test on real microbiome relative abundance data (e.g. 16S taxa tables from the Bangladesh cohort) controls false positive rates by permuting artificial group labels under the null of no group effect.

## When NOT to use

- When the data contain a real, biologically defined group comparison (e.g. gender or breastfeeding status) — the split must be artificial to test null-hypothesis type I error control
- When the goal is effect-size estimation or meta-analysis pooling of estimates across studies rather than false-positive rate calibration
- When the input is a feature/abundance table from a non-microbiome data type that does not resemble a zero-inflated beta distribution

## Inputs

- Published processed 16S relative abundance taxa tables of the Subramanian et al. 2014 Bangladesh cohort (from https://gordonlab.wustl.edu/Subramanian_6_14/Nature_2014_Processed_16S_rRNA_datasets.html)
- metamicrobiomeR package dataset taxtab.rm7 (taxonomy table with l2 phylum level)
- metamicrobiomeR package dataset sam.rm (sample mapping file)
- 50-sample cross-sectional and 996-sample longitudinal stool sample subsets with infant identifiers for repeated measures

## Outputs

- Per-iteration empirical type I error rates (proportion of taxa with p < 0.05) at phylum (l2) and family (l5) levels for GAMLSS-BEZI and LMAS
- Median (IQR) summary over 1000 iterations, formatted as Table 2 of the paper, written to type1_error_table.csv

## How to apply

Load the processed 16S data of the Subramanian et al. 2014 Bangladesh cohort (50-sample cross-sectional and 996-sample longitudinal stool subsets) together with the package datasets taxtab.rm7 (phylum taxonomy, level l2) and sam.rm, and filter taxa with taxa.filter (percent.filter = 0.05, relabund.filter = 0.00005). For each of 1000 iterations, randomly reassign samples to artificial case/control groups, keeping repeated samples per infant together for the longitudinal subset. Test every taxon at phylum (l2) and family (l5) levels with taxa.compare using propmed.rel="gamlss" (zero-inflated beta GAMLSS-BEZI) and longitudinal="yes", and in parallel with LMAS (linear model, or linear mixed model with random intercepts for repeated measures). Compute the per-iteration type I error as the fraction of taxa with p-value < 0.05, then report the median (IQR) across 1000 iterations, formatted to match Table 2 of the paper. Correct application is judged by median type I errors close to the nominal alpha levels of 0.01, 0.05 and 0.1 for both methods and both taxonomic levels.

## Related tools

- **metamicrobiomeR** (Provides taxa.filter, taxa.compare (with propmed.rel="gamlss" invoking GAMLSS zero-inflated beta family), and the datasets taxtab.rm7 and sam.rm used to run the per-split differential abundance tests) — https://github.com/nhanhocu/metamicrobiomeR
- **R** (Implementation language; R 3.4.2 or higher required; random splits, iteration loop, and type I error summarization are scripted in R)
- **GAMLSS-BEZI** (Zero-inflated beta GAMLSS family used by taxa.compare to test differential relative abundance under each artificial case/control split) — https://github.com/nhanhocu/metamicrobiomeR
- **MaAsLin** (Reference linear-model (LMAS) comparator: arcsin square root transformed relative abundances tested with linear models or linear mixed models with random intercepts for the longitudinal design)
- **knitr** (Used with kable() to render result tables in the package workflow examples)

## Examples

```
taxacom<-taxa.compare(taxtab=taxtab6.rm[[5]],propmed.rel="gamlss",comvar="group",adjustvar="age.sample",longitudinal="yes",p.adjust.method="fdr")
```

## Evaluation signals

- Median type I error across the 1000 iterations is close to the nominal alpha levels of 0.01, 0.05 and 0.1 at both phylum (l2) and family (l5) levels for both GAMLSS-BEZI and LMAS
- Each iteration's type I error is computed as the proportion of taxa with p-value < 0.05 under the null split, so per-iteration values must lie in [0, 1] and center near 0.05
- The 1000 per-iteration results are summarized as median (IQR) and the resulting table matches the structure of Table 2 of the paper and is written to type1_error_table.csv
- Longitudinal splits keep repeated samples per infant intact (linear mixed model random intercepts), and the cross-sectional subset is exactly 50 stool samples while the longitudinal subset is 996
- Taxa are pre-filtered with the same thresholds as the paper (percent.filter = 0.05, relabund.filter = 0.00005) before testing

## Limitations

- Type I error control was demonstrated only at phylum (l2) and family (l5) levels on the Bangladesh cohort subsets; other taxonomic levels or datasets are not covered by this validation
- LMAS relies on transformation (e.g. arcsin square root) that does not address the inflation of zero values in relative abundance data, so comparator results depend on that transformation
- Results are summarized as median (IQR) rather than mean, and performance on real data depends on the 1000-iteration split scheme preserving the study design

## Evidence

- [abstract] Both simulation studies and application to real microbiome data demonstrate that GAMLSS-BEZI well performs in testing differential relative abundances of microbial taxonomies.: "Both simulation studies and application to real microbiome data demonstrate that GAMLSS-BEZI well performs in testing differential relative abundances of microbial taxonomies."
- [intro] However, these methods do not address the actual distribution of the microbial taxonomy relative abundance data, which resemble a zero-inflated beta distribution.: "However, these methods do not address the actual distribution of the microbial taxonomy relative abundance data, which resemble a zero-inflated beta distribution."
- [methods] taxa.compare with gamlss and longitudinal options: "taxacom6.zi.rmg<-taxa.compare(taxtab=taxtab6.rm[[5]],propmed.rel="gamlss",comvar="bf",adjustvar="age.sample",longitudinal="yes",p.adjust.method="fdr")"
- [methods] taxa.filter call: "taxlist.rm<-taxa.filter(taxtab=taxtab.rm[[5]],percent.filter = 0.05, relabund.filter = 0.00005)"
- [readme] README package description: "The *metamicrobiomeR* package implements Generalized Additive Model for Location, Scale and Shape (GAMLSS) 
    with zero inflated beta (BEZI) family for analysis of microbiome relative abundance data"

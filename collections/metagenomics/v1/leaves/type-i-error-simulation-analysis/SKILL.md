---
name: type-i-error-simulation-analysis
description: Use when you need to demonstrate that a differential abundance testing
  method controls false positives at nominal alpha (e.g. 0.01, 0.05, 0.1) on real
  16S relative abundance data with known null group labels.
when_to_use_negative:
- The goal is testing real, biologically defined group differences (e.g. gender or
  breastfeeding status) rather than validating false-positive control under artificial
  null splits
- The input is raw sequencing reads or an unprocessed OTU/ASV table rather than taxa
  relative abundance tables prepared via the package's taxa.filter/taxtab workflow
- Comparing methods on simulated count data with a known generative model instead
  of real microbiome data with randomized group labels
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0091
tools:
- name: metamicrobiomeR
  role: Provides taxa.compare with propmed.rel='gamlss' (GAMLSS zero-inflated beta
    family) for differential abundance testing under the null splits, plus package
    datasets taxtab.rm7 and sam.rm
  repo: https://github.com/nhanhocu/metamicrobiomeR
- name: GAMLSS-BEZI
  role: Statistical family applied via metamicrobiomeR whose type I error is being
    assessed on real data
- name: MaAsLin
  role: LMAS-style linear model / linear mixed model (arcsin square-root transformed
    relative abundance) run in parallel as the comparison method whose type I error
    is assessed
- name: R
  role: Implementation language; requires R 3.4.2 or higher
- name: knitr
  role: Used to render result tables (e.g. via kable) in the reproducible workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-019-2744-2
    title: 'metamicrobiomeR: an R package for analysis of microbiome relative abundance
      data using zero-inflated beta GAMLSS and meta-analysis across studies using
      random effects models'
  source_content_hash: sha256:8a18746efb5e441eaef18a77aa39976de2397fe08f9fdf8d9d6e594e98fd4ff0
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# type-i-error-simulation-analysis

## Summary

Empirically assesses whether differential abundance tests (GAMLSS-BEZI via metamicrobiomeR and LMAS linear/mixed models) hold their nominal type I error rate on real microbiome data by repeatedly splitting the Bangladesh infant cohort into artificial case/control groups and tabulating the fraction of taxa called significant.

## When to use

Use when you need to demonstrate that a differential abundance testing method controls false positives at nominal alpha (e.g. 0.01, 0.05, 0.1) on real 16S relative abundance data with known null group labels.

## When NOT to use

- The goal is testing real, biologically defined group differences (e.g. gender or breastfeeding status) rather than validating false-positive control under artificial null splits
- The input is raw sequencing reads or an unprocessed OTU/ASV table rather than taxa relative abundance tables prepared via the package's taxa.filter/taxtab workflow
- Comparing methods on simulated count data with a known generative model instead of real microbiome data with randomized group labels

## Inputs

- Processed 16S relative abundance data of the Subramanian et al. 2014 Bangladesh cohort (from https://gordonlab.wustl.edu/Subramanian_6_14/Nature_2014_Processed_16S_rRNA_datasets.html)
- taxtab.rm7 taxonomy table (phylum-level, level l2) shipped with metamicrobiomeR
- sam.rm sample mapping file shipped with metamicrobiomeR
- 50-sample cross-sectional and 996-sample longitudinal stool subsets of the cohort
- Nominal alpha levels (0.01, 0.05, 0.1) and number of iterations (1000)

## Outputs

- Per-iteration empirical type I error rates (proportion of taxa with p < 0.05 per null split)
- Summary table of median (IQR) type I error per method (GAMLSS-BEZI, LMAS) and taxonomic level (phylum l2, family l5), formatted as Table 2 and written to type1_error_table.csv

## How to apply

Load the published processed 16S data from the Subramanian et al. 2014 Bangladesh cohort (gordonlab.wustl.edu) together with the package datasets taxtab.rm7 (phylum-level taxonomy, level l2) and sam.rm (sample mapping), and form the 50-sample cross-sectional and 996-sample longitudinal stool subsets. For each of 1000 iterations, randomly assign samples to artificial case/control groups while preserving the design structure (longitudinal splits keep repeated samples per infant). Test each taxon for differential relative abundance using taxa.compare from metamicrobiomeR with propmed.rel='gamlss' (zero-inflated beta GAMLSS-BEZI) and longitudinal='yes', and in parallel with LMAS (linear model for cross-sectional, linear mixed model with random intercepts for longitudinal) at phylum (l2) and family (l5) levels. At each iteration compute the empirical type I error as the proportion of taxa with p-value < 0.05 under the null split, then summarize the 1000 per-iteration rates as median (IQR) per method and taxonomic level, matching Table 2 of the paper.

## Related tools

- **metamicrobiomeR** (Provides taxa.compare with propmed.rel='gamlss' (GAMLSS zero-inflated beta family) for differential abundance testing under the null splits, plus package datasets taxtab.rm7 and sam.rm) — https://github.com/nhanhocu/metamicrobiomeR
- **GAMLSS-BEZI** (Statistical family applied via metamicrobiomeR whose type I error is being assessed on real data)
- **MaAsLin** (LMAS-style linear model / linear mixed model (arcsin square-root transformed relative abundance) run in parallel as the comparison method whose type I error is assessed)
- **R** (Implementation language; requires R 3.4.2 or higher)
- **knitr** (Used to render result tables (e.g. via kable) in the reproducible workflow)

## Examples

```
taxacom<-taxa.compare(taxtab=taxtab.rm[[5]],propmed.rel="gamlss",comvar="group",longitudinal="yes",p.adjust.method="fdr")
```

## Evaluation signals

- Median type I error at phylum and family levels falls close to the nominal alpha levels 0.01, 0.05, and 0.1 for both GAMLSS-BEZI and LMAS, in both cross-sectional and longitudinal subsets (matching Table 2)
- Exactly 1000 per-iteration null splits were run for each subset, with longitudinal splits preserving repeated samples per infant
- Empirical rate per iteration is computed as the proportion of taxa with p-value < 0.05 under the artificial case/control assignment
- Output CSV reproduces the Table 2 layout: rows for method × taxonomic level (l2, l5), values reported as median (IQR)

## Limitations

- Type I error on real data with randomized labels is design- and cohort-specific; results are specific to the 50-sample cross-sectional and 996-sample longitudinal Bangladesh stool subsets
- Only phylum (l2) and family (l5) taxonomic levels were evaluated in Table 2; lower levels (e.g. genus l6) are not covered by this skill
- The null-split approach removes real biological signal but does not test power or estimation accuracy of the log(odds ratio) estimates

## Evidence

- [abstract] GAMLSS-BEZI performs well in testing differential relative abundances of microbial taxonomies, based on simulation studies and real microbiome data: "Both simulation studies and application to real microbiome data demonstrate that GAMLSS-BEZI well performs in testing differential relative abundances of microbial taxonomies."
- [methods] taxa.compare usage for null testing: "taxacom6.zi.rmg<-taxa.compare(taxtab=taxtab6.rm[[5]],propmed.rel="gamlss",comvar="bf",adjustvar="age.sample",longitudinal="yes",p.adjust.method="fdr")"
- [intro] distribution rationale for GAMLSS-BEZI: "However, these methods do not address the actual distribution of the microbial taxonomy relative abundance data, which resemble a zero-inflated beta distribution."
- [readme] package install and data loading from README: "install_github("nhanhocu/metamicrobiomeR") library(metamicrobiomeR) #Load other needed packages library(knitr)"

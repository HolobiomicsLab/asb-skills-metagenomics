---
name: alpha-diversity-index-computation
description: Use when comparing within-sample microbial diversity between treatment
  groups on a rarefied feature table and you need per-sample alpha-diversity estimates
  (including phylogenetic indices) plus covariate-adjusted group difference tests.
when_to_use_negative:
- The analysis goal is between-sample composition comparison (beta diversity / MiRKAT),
  not within-sample richness estimation
- The input is raw sequencing reads rather than a processed feature table — data processing
  (QC, rarefaction) must happen first
- No treatment grouping or covariate metadata is available, so adjusted group comparison
  cannot be fit
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: phyloseq
  role: Provides the phyloseq object format, rarefy_even_depth for rarefaction, and
    estimate_richness for computing the alpha-diversity indices
  repo: https://github.com/wg99526/MiSurvGit
- name: picante
  role: Supplies the pd function for computing phylogenetic diversity on the rarefied
    feature table
- name: MiSurv
  role: Web cloud platform whose Module 2 performs covariate-adjusted comparative
    microbial composition (alpha-diversity) analysis between treatment groups
  repo: https://github.com/wg99526/MiSurvGit
- name: QIIME2
  role: Used upstream to obtain the feature table, taxonomic annotations, and phylogenetic
    tree consumed by this analysis
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1128/spectrum.05059-22
    title: 'MiSurv: an Integrative Web Cloud Platform for User-Friendly Microbiome
      Data Analysis with Survival Responses'
  source_content_hash: sha256:e1603e2ef255a50e69d5fb19fe494ff246b3ffc0f50b1ff6665571a32a37c8e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wg99526/MiSurvGit
  tools:
  - MiSurv
  - QIIME2
  - R
  - phyloseq
  - picante
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# alpha-diversity-index-computation

## Summary

Computes nine alpha-diversity indices per sample on a rarefied microbiome feature table using phyloseq's estimate_richness plus phylogenetic diversity via picante's pd, then fits covariate-adjusted linear regressions (e.g., treatment effect with Sex as covariate) to compare groups as in MiSurv Module 2.

## When to use

Use when comparing within-sample microbial diversity between treatment groups on a rarefied feature table and you need per-sample alpha-diversity estimates (including phylogenetic indices) plus covariate-adjusted group difference tests.

## When NOT to use

- The analysis goal is between-sample composition comparison (beta diversity / MiRKAT), not within-sample richness estimation
- The input is raw sequencing reads rather than a processed feature table — data processing (QC, rarefaction) must happen first
- No treatment grouping or covariate metadata is available, so adjusted group comparison cannot be fit

## Inputs

- phyloseq object 'biom.Rdata' (Data folder of https://github.com/wg99526/MiSurvGit)
- Four individual input files: 'otu.tab.txt', 'sam.dat.txt', 'tax.tab.txt', 'tree.tre'
- Sample metadata with treatment (control vs tylosin) and Sex covariates

## Outputs

- Per-sample values for nine alpha-diversity indices (eight via estimate_richness + phylogenetic diversity via pd)
- Summary statistics CSV of covariate-adjusted group difference estimates and p-values for all nine indices
- Alpha-diversity group comparison figure reproducing Fig. 3A

## How to apply

Load the phyloseq object 'biom.Rdata' (or assemble it from 'otu.tab.txt', 'sam.dat.txt', 'tax.tab.txt', and 'tree.tre') from the MiSurv GitHub Data folder. Rarefy the feature table to even depth using rarefy_even_depth in phyloseq so that diversity indices are comparable across samples. Compute eight non-phylogenetic alpha-diversity indices per sample with estimate_richness in phyloseq and phylogenetic diversity with the pd function in picante, yielding nine indices total. For each index, fit a linear regression of the index on treatment (control vs tylosin) with Sex as a covariate, extracting group difference estimates and p-values; assemble all nine results into a summary CSV and render the group comparison figure reproducing Fig. 3A. Validation requires all nine indices to show statistically significant, directionally consistent (lower in tylosin) differences matching Gu et al. 2023.

## Related tools

- **phyloseq** (Provides the phyloseq object format, rarefy_even_depth for rarefaction, and estimate_richness for computing the alpha-diversity indices) — https://github.com/wg99526/MiSurvGit
- **picante** (Supplies the pd function for computing phylogenetic diversity on the rarefied feature table)
- **MiSurv** (Web cloud platform whose Module 2 performs covariate-adjusted comparative microbial composition (alpha-diversity) analysis between treatment groups) — https://github.com/wg99526/MiSurvGit
- **QIIME2** (Used upstream to obtain the feature table, taxonomic annotations, and phylogenetic tree consumed by this analysis)

## Examples

```
ps <- readRDS("biom.Rdata"); ps.r <- rarefy_even_depth(ps); adf <- estimate_richness(ps.r); pd.res <- pd(as.data.frame(otu_table(ps.r)), phy_tree(ps.r))
```

## Evaluation signals

- All nine alpha-diversity indices show statistically significant control-vs-tylosin differences after Sex adjustment
- Estimated alpha-diversity is smaller in the tylosin group than the control group for every index, matching the direction in Fig. 3A of Gu et al. 2023 (Microbiology Spectrum 11(3): e05059-22)
- Rarefaction was applied via rarefy_even_depth before index computation (library sizes equalized)
- Phylogenetic diversity index computed with picante::pd on the tree.tre phylogeny, and the summary CSV contains nine rows (one per index) with estimates and p-values

## Limitations

- No version numbers are given for the R packages used (e.g., phyloseq, picante), so exact reproducibility of index values may depend on package versions
- The reference pipeline is demonstrated on the Zhang et al. T1D non-obese diabetic mouse dataset (QIITA study 10508); other datasets may need different covariate adjustments
- Rarefaction depth choice via rarefy_even_depth affects index values; the article's exact rarefaction parameters are not specified in the provided context

## Evidence

- [discussion] Compute nine alpha-diversity indices per sample using 'estimate_richness' in the 'phyloseq' package plus phylogenetic diversity via the 'pd' function in the 'picante' package: [evidence span withheld — anchor status: not_found]
- [discussion] Rarefaction step: [evidence span withheld — anchor status: not_found]
- [intro] Module 2 scope: "Module 2: Comparative analysis in microbial composition between treatment groups."
- [readme] Example data storage: "stored the all the final processed data that we used in our paper (Gu et al., 2023) in the above Data folder as ‘phyloseq’ format (see the file named ‘biom.Rdata’)"
- [discussion] Upstream QIIME 2.0 processing: "We used QIIME 2.0 [16] to obtain the feature table, taxonomic annotations, and phylogenetic tree"

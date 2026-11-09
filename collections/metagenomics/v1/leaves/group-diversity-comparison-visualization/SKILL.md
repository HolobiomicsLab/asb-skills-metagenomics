---
name: group-diversity-comparison-visualization
description: Use when comparing microbial composition between treatment groups (e.g.,
  control vs tylosin) with covariate adjustment — computing multiple alpha/beta diversity
  metrics on a rarefied feature table and rendering group-comparison figures.
when_to_use_negative:
- The question is survival analysis, microbiome–survival association, or survival
  prediction (MiSurv Modules 1, 3, or 4) rather than composition comparison between
  treatment groups — Module 2 does not involve survival data.
- Input is raw sequencing reads still requiring 16S rRNA processing (quality filtering,
  97% open-reference picking against Greengenes, chimera removal) rather than an already-built
  phyloseq object/feature table.
- Samples lack a treatment-group label and covariates such as Sex, so no covariate-adjusted
  group comparison can be fitted.
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: phyloseq
  role: rarefy_even_depth rarefaction, estimate_richness for the nine alpha indices,
    and distance functions for the five beta-diversity matrices
- name: picante
  role: provides the 'pd' function for phylogenetic diversity, one of the nine alpha-diversity
    indices
- name: MiRKAT
  role: kernel association testing of control-vs-tylosin differences for each of the
    five beta-diversity distance matrices
- name: QIIME2
  role: upstream generation of the feature table, taxonomic annotations, and phylogenetic
    tree consumed by this analysis
- name: MiSurv
  role: web cloud platform / local Shiny app whose Module 2 implements comparative
    microbial composition analysis between treatment groups
  repo: https://github.com/wg99526/MiSurvGit
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1128/spectrum.05059-22
    title: 'MiSurv: an Integrative Web Cloud Platform for User-Friendly Microbiome
      Data Analysis with Survival Responses'
  source_content_hash: sha256:bf160478c8e5de0fab6e42074e83b7d04c54ed35d4485e727f5166ecccb17089
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wg99526/MiSurvGit
  tools:
  - MiRKAT
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
yes_rate: 0.9230769230769231
n_claims: 13
---

# group-diversity-comparison-visualization

## Summary

Reproduces MiSurv Module 2-style control-vs-treatment microbiome composition comparisons: nine alpha-diversity indices and five beta-diversity distances computed on a rarefied phyloseq object, covariate-adjusted group testing, and figures reproducing Fig. 3A/3B of Gu et al. 2023 (Microbiology Spectrum 11(3): e05059-22).

## When to use

Use when comparing microbial composition between treatment groups (e.g., control vs tylosin) with covariate adjustment — computing multiple alpha/beta diversity metrics on a rarefied feature table and rendering group-comparison figures.

## When NOT to use

- The question is survival analysis, microbiome–survival association, or survival prediction (MiSurv Modules 1, 3, or 4) rather than composition comparison between treatment groups — Module 2 does not involve survival data.
- Input is raw sequencing reads still requiring 16S rRNA processing (quality filtering, 97% open-reference picking against Greengenes, chimera removal) rather than an already-built phyloseq object/feature table.
- Samples lack a treatment-group label and covariates such as Sex, so no covariate-adjusted group comparison can be fitted.

## Inputs

- phyloseq object 'biom.Rdata' from the MiSurvGit Data folder
- four individual files: 'otu.tab.txt', 'sam.dat.txt', 'tax.tab.txt', 'tree.tre'
- sample metadata with treatment group (control vs tylosin) and Sex covariate
- raw data alternative: QIITA study ID 10508 (16S rRNA V4, Greengenes 97% open-reference OTU picking)

## Outputs

- summary statistics table (CSV) of nine alpha-diversity and five beta-diversity control-vs-tylosin comparisons with estimates and p-values
- alpha-diversity group-comparison figure reproducing Fig. 3A
- beta-diversity group-comparison figure reproducing Fig. 3B

## How to apply

Load the phyloseq object 'biom.Rdata' (or the four individual files 'otu.tab.txt', 'sam.dat.txt', 'tax.tab.txt', 'tree.tre') from the Data folder of the MiSurv GitHub repository, then rarefy with 'rarefy_even_depth' in phyloseq. Compute nine alpha-diversity indices per sample using 'estimate_richness' in phyloseq plus phylogenetic diversity via the 'pd' function in picante, and compute five beta-diversity distance matrices with phyloseq's distance functions on the same processed feature table. For each alpha index, fit a covariate-adjusted linear regression of the index on treatment (control vs tylosin) with Sex as a covariate and extract group difference estimates and p-values; for each beta distance, test control-vs-tylosin association with MiRKAT kernel association tests. Assemble the nine alpha and five beta results into a summary statistics CSV and render two figures reproducing Fig. 3A/3B. Rationale: rarefaction before richness estimation equalizes library size, and covariate adjustment (Sex) isolates the treatment effect, matching the Module 2 design of Gu et al. 2023.

## Related tools

- **phyloseq** (rarefy_even_depth rarefaction, estimate_richness for the nine alpha indices, and distance functions for the five beta-diversity matrices)
- **picante** (provides the 'pd' function for phylogenetic diversity, one of the nine alpha-diversity indices)
- **MiRKAT** (kernel association testing of control-vs-tylosin differences for each of the five beta-diversity distance matrices)
- **QIIME2** (upstream generation of the feature table, taxonomic annotations, and phylogenetic tree consumed by this analysis)
- **MiSurv** (web cloud platform / local Shiny app whose Module 2 implements comparative microbial composition analysis between treatment groups) — https://github.com/wg99526/MiSurvGit

## Examples

```
ps_rar <- rarefy_even_depth(ps); div <- estimate_richness(ps_rar, measures = c("Observed", "Shannon", "Simpson")); pd_div <- pd(as.data.frame(otu_table(ps_rar)), phy_tree(ps_rar))
```

## Evaluation signals

- All nine alpha-diversity indices and all five beta-diversity distances must show statistically significant control-vs-tylosin differences with lower diversity in the tylosin group, matching Fig. 3A/3B of Gu et al. 2023.
- Alpha-diversity estimates are computed on the rarefied (rarefy_even_depth) feature table, not on the raw table.
- Each alpha-index regression includes Sex as a covariate and returns both a group difference estimate and a p-value.
- Results assemble into a single summary CSV containing all nine alpha and five beta comparison rows.
- Rendered figures visually match the direction (lower in tylosin) and significance pattern of Fig. 3A/3B.

## Limitations

- Validation is all-or-nothing: any single alpha index or beta distance failing significance/direction breaks reproduction of Fig. 3A/3B.
- No version numbers were given for R packages (phyloseq, survival, MiRKAT); local implementation needs R (>4.2.0).
- MiSurv's authors note hardware specification may need improvement, implying capacity limits on the public web server; local runs require the GitHub repo.

## Evidence

- [results] Alpha direction and significance: [evidence span withheld — anchor status: not_found]
- [readme] MiSurv data files: "We stored the all the final processed data that we used in our paper (Gu et al., 2023) in the above Data folder as ‘phyloseq’ format (see the file named ‘biom.Rdata’)"
- [readme] Module 2 scope: "Module 2 : Comparative analysis in microbial composition between treatment groups, not involving survival data, with or without covariate adjustment(s)."
- [discussion] Rarefaction step: [evidence span withheld — anchor status: not_found]
- [other] Diversity tooling: [evidence span withheld — anchor status: not_found]

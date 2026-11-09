---
name: covariate-adjusted-linear-regression
description: Use when comparing treatment groups on a continuous microbiome outcome
  (alpha-diversity index or CLR-transformed abundance) while adjusting for covariates
  such as Sex, as in MiSurv Module 2.
when_to_use_negative:
- The input is still raw sequencing data — the regression requires the processed phyloseq
  object or OTU/metadata tables
- The response is a survival outcome or a beta-diversity distance matrix — use MiSurv
  Modules 1/3 or MiRKAT instead, since regression handles one sample-level continuous
  response
- You need non-adjusted comparisons only when Sex is not a plausible confounder of
  treatment in the study design
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: MiSurv
  role: Web cloud / Shiny platform whose Module 2 implements covariate-adjusted comparative
    microbial composition analysis
  repo: https://github.com/wg99526/MiSurvGit
- name: phyloseq
  role: Provides the phyloseq object, rarefy_even_depth, estimate_richness, and CLR
    transformation feeding the regressions
  repo: https://github.com/wg99526/MiSurvGit
- name: picante
  role: pd function computes phylogenetic diversity, one of the nine alpha-diversity
    responses regressed on treatment
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1128/spectrum.05059-22
    title: 'MiSurv: an Integrative Web Cloud Platform for User-Friendly Microbiome
      Data Analysis with Survival Responses'
  source_content_hash: sha256:5cf5151ea7841478c7042de306e89b0e7f3c5568f1b1465599ea0c2f9dd6418d
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
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# covariate-adjusted-linear-regression

## Summary

Fits linear regression of a microbiome response (rarefied alpha-diversity index or CLR-transformed taxon abundance) on treatment group with covariates (e.g., Sex), used in MiSurv Module 2 to compare control vs tylosin groups and, for taxa, to call significant hits at BH FDR 5%.

## When to use

Use when comparing treatment groups on a continuous microbiome outcome (alpha-diversity index or CLR-transformed abundance) while adjusting for covariates such as Sex, as in MiSurv Module 2.

## When NOT to use

- The input is still raw sequencing data — the regression requires the processed phyloseq object or OTU/metadata tables
- The response is a survival outcome or a beta-diversity distance matrix — use MiSurv Modules 1/3 or MiRKAT instead, since regression handles one sample-level continuous response
- You need non-adjusted comparisons only when Sex is not a plausible confounder of treatment in the study design

## Inputs

- phyloseq object biom.Rdata (or the four files otu.tab.txt, sam.dat.txt, tax.tab.txt, tree.tre) from the MiSurvGit Data folder
- Sample metadata including treatment group (control vs tylosin) and Sex
- Rarefied (rarefy_even_depth) feature table and derived alpha-diversity indices or CLR-transformed taxon abundances

## Outputs

- Per-index group difference estimates and p-values for nine alpha-diversity indices (Fig. 3A-style comparison)
- significant_taxa_fdr5.tsv listing taxa from phylum to genus with BH-adjusted P < 0.05 (66 taxa expected)
- Summary statistics CSV of alpha and beta comparison results

## How to apply

Load the processed phyloseq data ('biom.Rdata' or 'otu.tab.txt', 'sam.dat.txt', 'tax.tab.txt', 'tree.tre') from the MiSurvGit Data folder and compute responses on the rarefied (rarefy_even_depth) feature table: nine alpha-diversity indices via estimate_richness plus phylogenetic diversity via picante's pd, or CLR-transformed abundances for taxa from phylum to genus. For each response, fit a linear regression of the response on treatment (control vs tylosin) with Sex as a covariate, and extract the group coefficient estimate and p-value. For alpha-diversity, interpret the sign of the group estimate (negative means lower diversity in tylosin). For taxa, collect raw P-values across all taxa and apply Benjamini-Hochberg FDR correction at a 5% threshold, retaining taxa with adjusted P < 0.05 as differentially abundant.

## Related tools

- **MiSurv** (Web cloud / Shiny platform whose Module 2 implements covariate-adjusted comparative microbial composition analysis) — https://github.com/wg99526/MiSurvGit
- **phyloseq** (Provides the phyloseq object, rarefy_even_depth, estimate_richness, and CLR transformation feeding the regressions) — https://github.com/wg99526/MiSurvGit
- **picante** (pd function computes phylogenetic diversity, one of the nine alpha-diversity responses regressed on treatment)

## Examples

```
fit <- lm(alpha_shannon ~ treatment + Sex, data = div_df); summary(fit)$coefficients['treatmenttylosin',]
```

## Evaluation signals

- All nine alpha-diversity indices show significantly smaller estimates in the tylosin group than control, matching Fig. 3A direction and significance
- significant_taxa_fdr5.tsv contains exactly 66 significant taxa rows at BH FDR 5%, matching Figures 4 and 5 of Gu et al. 2023
- Every regression includes Sex as a covariate, so group coefficients are Sex-adjusted control-vs-tylosin contrasts
- Responses are computed on the rarefied (rarefy_even_depth) feature table, not raw counts
- Adjusted P-values in the output table are all < 0.05 under Benjamini-Hochberg correction

## Limitations

- The paper provides no version numbers for R packages (e.g., phyloseq, survival, MiRKAT); R > 4.2.0 is required for local MiSurv implementation
- Sex is the only documented covariate; other confounders in a given dataset must be handled by extending the model
- CLR-based regression is compositional and assumes the rarefied/transformed table; results depend on prior quality-control and rarefaction choices

## Evidence

- [methods] alpha-diversity regression: [evidence span withheld — anchor status: not_found]
- [methods] taxa regression and FDR: [evidence span withheld — anchor status: not_found]
- [results] significance count: [evidence span withheld — anchor status: not_found]
- [results] alpha-diversity direction: [evidence span withheld — anchor status: not_found]
- [readme] data files: "stored the all the final processed data that we used in our paper (Gu et al., 2023) in the above Data folder as ‘phyloseq’ format (see the file named ‘biom.Rdata’)"

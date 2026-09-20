---
name: survival-association-kernel-testing
description: Use when testing whether gut microbiome beta-diversity (e.g., Jaccard
  dissimilarity) associates with survival/time-to-event outcomes (e.g., time-to-T1D
  onset) in a study with treatment groups and covariates like sex.
when_to_use_negative:
- The question is a control-vs-treatment comparison of microbial composition without
  a survival response — that is Module 2 territory (MiRKAT, not MiRKAT-S)
- The goal is prediction modeling of survival from microbial taxa rather than association
  testing — that is Module 4
- Input is raw sequence data still needing QIIME 2.0 processing; run the Data Processing
  module (quality control, rarefaction) first
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0625
tools:
- name: MiSurv
  role: Integrative web/shiny platform whose Module 3 performs the kernel association
    testing between microbial composition and survival responses
  repo: https://github.com/wg99526/MiSurvGit
- name: MiRKAT
  role: Kernel-based regression framework underlying MiRKAT-S association testing
    between distance kernels and survival outcomes
- name: phyloseq
  role: Provides the phyloseq object, 'distance' function (e.g., method='jaccard'),
    and 'rarefy_even_depth' for constructing the beta-diversity kernel
- name: shiny
  role: Runs the MiSurv web application locally via runGitHub
  repo: https://github.com/wg99526/MiSurvGit
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1128/spectrum.05059-22
    title: 'MiSurv: an Integrative Web Cloud Platform for User-Friendly Microbiome
      Data Analysis with Survival Responses'
  source_content_hash: sha256:2a3b25eb00b182e020f161080754ca5b5f13ba0f29b1c06f894e2cd9f90c8e96
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
  - shiny
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# survival-association-kernel-testing

## Summary

MiSurv Module 3 association testing between microbial composition (beta-diversity distance kernels, e.g., Jaccard) and survival (time-to-event) responses via MiRKAT-S, with optional covariate adjustment and subgroup analysis.

## When to use

Use when testing whether gut microbiome beta-diversity (e.g., Jaccard dissimilarity) associates with survival/time-to-event outcomes (e.g., time-to-T1D onset) in a study with treatment groups and covariates like sex.

## When NOT to use

- The question is a control-vs-treatment comparison of microbial composition without a survival response — that is Module 2 territory (MiRKAT, not MiRKAT-S)
- The goal is prediction modeling of survival from microbial taxa rather than association testing — that is Module 4
- Input is raw sequence data still needing QIIME 2.0 processing; run the Data Processing module (quality control, rarefaction) first

## Inputs

- phyloseq object 'biom.Rdata' or the four individual files 'otu.tab.txt', 'sam.dat.txt', 'tax.tab.txt', 'tree.tre'
- Sample metadata with survival time, censored/event status, follow-up period, treatment group, and covariates (e.g., Sex)
- Beta-diversity distance matrix (e.g., Jaccard) computed from the processed feature table

## Outputs

- MiRKAT-S kernel association P-value between microbiome beta-diversity and survival responses (e.g., CSV such as 'mirkat_s_jaccard_control_pvalue.csv')
- Optionally, subgroup-stratified association results (e.g., control-group Jaccard P = 0.046, Fig. 3D)

## How to apply

Load the processed phyloseq object 'biom.Rdata' (or the individual files 'otu.tab.txt', 'sam.dat.txt', 'tax.tab.txt', 'tree.tre') from the MiSurv GitHub Data folder. Optionally subset samples to a treatment-group subgroup (e.g., control subjects only) using the group column of 'sam.dat.txt', retaining survival time, event status, and covariate columns (e.g., Sex). Compute the pairwise beta-diversity dissimilarity matrix (e.g., Jaccard via phyloseq's 'distance' with method='jaccard') on the processed feature table. Run MiRKAT-S (Module 3) kernel association testing between the distance kernel and the survival response with covariate adjustment, then write the P-value to a results file (e.g., 'mirkat_s_jaccard_control_pvalue.csv'). Subgroup analysis is a deliberate design option because microbiome effects on survival may differ between treatment groups.

## Related tools

- **MiSurv** (Integrative web/shiny platform whose Module 3 performs the kernel association testing between microbial composition and survival responses) — https://github.com/wg99526/MiSurvGit
- **MiRKAT** (Kernel-based regression framework underlying MiRKAT-S association testing between distance kernels and survival outcomes)
- **phyloseq** (Provides the phyloseq object, 'distance' function (e.g., method='jaccard'), and 'rarefy_even_depth' for constructing the beta-diversity kernel)
- **shiny** (Runs the MiSurv web application locally via runGitHub) — https://github.com/wg99526/MiSurvGit

## Examples

```
library(shiny); runGitHub("MiSurvGit", "wg99526", ref = "main")
```

## Evaluation signals

- Control-group Jaccard dissimilarity vs survival association reproduces P = 0.046 (Fig. 3D) within ±0.01 rounding tolerance
- Association P-value is written to the results file (e.g., 'mirkat_s_jaccard_control_pvalue.csv') with the expected schema (kernel, subgroup, covariates, P-value)
- Subgroup subsetting yields only control-group subjects per the group column of 'sam.dat.txt', with survival time, event status, and Sex retained
- Jaccard distance matrix is square, symmetric, with zero diagonal and dimensions matching the number of subsetted samples
- Significance direction and magnitude are consistent with the published finding when rerun via the MiSurv web server (http://misurv.micloud.kr) or local shiny app

## Limitations

- No version numbers are given for R packages used (e.g., phyloseq, survival, MiRKAT), so exact reproducibility may vary
- Subgroup analyses assume microbiome effects on survival differ between treatment groups; pooled analysis may mask subgroup-specific signals
- Published P-values may only match within rounding tolerance due to rarefaction randomness and implementation details

## Evidence

- [readme] Module 3: Association testing between microbial composition and survival responses with or without covariate adjustment(s).: "Module 3: Association testing between microbial composition and survival responses with or without covariate adjustment(s)."
- [discussion] Subgroup analysis rationale: "we added the subgroup analysis user option because it is often assumed that the effects of microbiome on survival responses are different between treatment groups"
- [readme] Data files location: "We stored the all the final processed data that we used in our paper (Gu et al., 2023) in the above Data folder as ‘phyloseq’ format (see the file named ‘biom.Rdata’)"
- [methods] GitHub local implementation: "GitHub repository (local implementation): https://github.com/wg99526/MiSurvGit"

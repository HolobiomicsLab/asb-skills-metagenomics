---
name: covariate-adjusted-mirkat-s-analysis
description: Use when you have an OTU/feature table with matched survival times, censoring/event
  status, and covariates, and need a kernel-based beta-diversity-vs-survival association
  test (MiRKAT-S), optionally within a treatment-group subgroup.
when_to_use_negative:
- The question is comparative survival between treatment groups without microbiome
  data (use MiSurv Module 1) or comparative microbial composition without survival
  data (use Module 2).
- The goal is prediction modeling of survival from microbial taxa (use MiSurv Module
  4) rather than association testing.
- The response is continuous or binary rather than survival/time-to-event — MiSurv
  is specifically designed for survival responses, unlike MiCloud.
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0625
tools:
- name: MiSurv
  role: Shiny web/local platform whose Module 3 performs the covariate-adjusted MiRKAT-S
    association testing between microbiome composition and survival responses, with
    subgroup analysis option
  repo: https://github.com/wg99526/MiSurvGit
- name: MiRKAT
  role: Kernel-based regression method underlying Module 3's beta-diversity vs survival
    association test
- name: phyloseq
  role: Used to load/subset the phyloseq object and compute the pairwise Jaccard dissimilarity
    matrix via distance(method='jaccard')
- name: shiny
  role: Runtime dependency for launching MiSurv locally via runGitHub('MiSurvGit',
    'wg99526', ref = 'main')
  repo: https://github.com/wg99526/MiSurvGit
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1128/spectrum.05059-22
    title: 'MiSurv: an Integrative Web Cloud Platform for User-Friendly Microbiome
      Data Analysis with Survival Responses'
  source_content_hash: sha256:5636db5772450c7bc0c07686e822c53050371f5ed950cf0424d6f4e147c72960
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wg99526/MiSurvGit
  tools:
  - MiRKAT
  - MiSurv
  - phyloseq
  - shiny
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# covariate-adjusted-mirkat-s-analysis

## Summary

Run MiSurv Module 3 (MiRKAT-S) to test association between a microbiome beta-diversity kernel and survival (time-to-event) responses, with optional covariate adjustment and subgroup subsetting, as demonstrated on the Zhang et al. antibiotic-T1D NOD-mouse gut microbiome data.

## When to use

Use when you have an OTU/feature table with matched survival times, censoring/event status, and covariates, and need a kernel-based beta-diversity-vs-survival association test (MiRKAT-S), optionally within a treatment-group subgroup.

## When NOT to use

- The question is comparative survival between treatment groups without microbiome data (use MiSurv Module 1) or comparative microbial composition without survival data (use Module 2).
- The goal is prediction modeling of survival from microbial taxa (use MiSurv Module 4) rather than association testing.
- The response is continuous or binary rather than survival/time-to-event — MiSurv is specifically designed for survival responses, unlike MiCloud.

## Inputs

- phyloseq object 'biom.Rdata' (or 'otu.tab.txt', 'sam.dat.txt', 'tax.tab.txt', 'tree.tre') from the MiSurv Data folder
- sample metadata with survival time, censored/event indicator, treatment group, and covariate columns (e.g., Sex)
- OTU/feature table for computing the beta-diversity dissimilarity matrix (e.g., Jaccard)
- subgroup specification (e.g., control group) and covariate adjustment plan

## Outputs

- MiRKAT-S association P-value between the beta-diversity kernel and survival response (e.g., written to 'mirkat_s_jaccard_control_pvalue.csv')
- kernel-vs-survival association test statistic from MiSurv Module 3

## How to apply

Load the processed phyloseq object 'biom.Rdata' (or the four individual files 'otu.tab.txt', 'sam.dat.txt', 'tax.tab.txt', 'tree.tre') from the MiSurv GitHub Data folder. If a subgroup analysis is needed (the article added this option because microbiome effects on survival can differ between treatment groups), subset the phyloseq object to the desired treatment group using the group column of 'sam.dat.txt', keeping survival time, event status, and covariate columns such as Sex. Compute the chosen beta-diversity distance matrix (e.g., phyloseq::distance with method='jaccard') on the subset OTU table, then run Module 3 of MiSurv — MiRKAT-S association testing between the distance kernel and the survival response, adjusting for the covariate(s) (e.g., Sex). Write the resulting P-value to a results file such as 'mirkat_s_jaccard_control_pvalue.csv'; in the published example the control-subgroup, sex-adjusted Jaccard test gave P = 0.046 (Fig. 3D).

## Related tools

- **MiSurv** (Shiny web/local platform whose Module 3 performs the covariate-adjusted MiRKAT-S association testing between microbiome composition and survival responses, with subgroup analysis option) — https://github.com/wg99526/MiSurvGit
- **MiRKAT** (Kernel-based regression method underlying Module 3's beta-diversity vs survival association test)
- **phyloseq** (Used to load/subset the phyloseq object and compute the pairwise Jaccard dissimilarity matrix via distance(method='jaccard'))
- **shiny** (Runtime dependency for launching MiSurv locally via runGitHub('MiSurvGit', 'wg99526', ref = 'main')) — https://github.com/wg99526/MiSurvGit

## Examples

```
library(shiny); runGitHub("MiSurvGit", "wg99526", ref = "main")  # then use Module 3 with subgroup=control, covariate=Sex, kernel=Jaccard
```

## Evaluation signals

- Control-subgroup, sex-adjusted Jaccard MiRKAT-S P-value must match the published P = 0.046 (Fig. 3D) within rounding tolerance (~±0.01)
- The Jaccard dissimilarity matrix must be square, symmetric, with zero diagonal and dimensions equal to the number of control-group samples
- Subgroup subsetting must leave only control-group samples (verified against the group column of 'sam.dat.txt') while preserving survival time, event status, and Sex columns
- The covariate model must include Sex (no unadjusted result accepted for the published comparison)
- MiSurv runs under R > 4.2.0 with shiny installed, per the README prerequisites

## Limitations

- Subgroup analysis in Modules 3 and 4 was added because microbiome effects on survival are assumed to differ between treatment groups; running on the pooled cohort may miss subgroup-specific effects
- No R package version numbers (phyloseq, survival, MiRKAT) are given in the article/README, so exact reproducibility may vary
- MiSurv web server hardware specification may need improvement, implying capacity limits; local R (>4.2.0) implementation is an alternative

## Evidence

- [intro] Module 3: Association testing between microbial composition and survival responses: "Module 3: Association testing between microbial composition and survival responses."
- [discussion] we added the subgroup analysis user option because it is often assumed that the effects of microbiome on survival responses are different between treatment groups: "we added the subgroup analysis user option because it is often assumed that the effects of microbiome on survival responses are different between treatment groups"
- [readme] We stored the all the final processed data that we used in our paper (Gu et al., 2023) in the above Data folder as ‘phyloseq’ format: "We stored the all the final processed data that we used in our paper (Gu et al., 2023) in the above Data folder as ‘phyloseq’ format (see the file named ‘biom.Rdata’)"
- [readme] Module 3 : Association testing between microbial composition and survival responses with or without covariate adjustment(s).: "* **Module 3** : Association testing between microbial composition and survival responses with or without covariate adjustment(s)."
- [readme] For local implementation using this GitHub repository, recent version of **R (>4.2.0)** is needed.: "* For local implementation using this GitHub repository, recent version of **R (>4.2.0)** is needed."

---
name: cox-proportional-hazards-survival-analysis
description: Use when input is a phyloseq object (biom.Rdata) or its four component
  files with survival time, censoring indicator, treatment, and covariate columns,
  and the goal is a covariate-adjusted Cox model of treatment or CLR-transformed microbial
  abundance on survival.
when_to_use_negative:
- Input is raw 16S sequence data still needing QIIME 2.0 processing — use MiSurv's
  Data Processing module first (quality control, 97% OTU picking, ChimeraSlayer, rarefaction
  via rarefy_even_depth)
- The analysis goal is comparative microbial composition between treatment groups
  without survival data (Module 2) or prediction modeling of survival from microbial
  taxa (Module 4)
- The response is continuous or binary rather than a survival (time-to-event) response
  — MiCloud, not MiSurv, is the appropriate platform
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: MiSurv
  role: Web cloud or local R/Shiny platform providing Module 1 (comparative survival
    analysis) and Module 3 (microbiome–survival association testing) with subgroup
    analysis options
  repo: https://github.com/wg99526/MiSurvGit
- name: R (survival package)
  role: Fits the Cox proportional hazards model via coxph on Surv(T1Dweek, T1D) ~
    Antibiotics + Sex
- name: shiny
  role: Required package to launch the local MiSurv implementation from the GitHub
    repository
  repo: https://github.com/wg99526/MiSurvGit
- name: phyloseq
  role: Provides the phyloseq object format (biom.Rdata) holding the OTU table, sample
    data, taxonomy, and tree; also used for rarefy_even_depth in preprocessing
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1128/spectrum.05059-22
    title: 'MiSurv: an Integrative Web Cloud Platform for User-Friendly Microbiome
      Data Analysis with Survival Responses'
  source_content_hash: sha256:3d088fed378746a21be8a7db5ec3cd6fc1a6921ca62ec133d765f667571cb752
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wg99526/MiSurvGit
  tools:
  - MiSurv
  - R
  - R (survival package)
  - phyloseq
  - shiny
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cox-proportional-hazards-survival-analysis

## Summary

Fits covariate-adjusted Cox proportional hazards models for microbiome survival studies via MiSurv's Module 1 (comparative survival analysis between treatment groups) or Module 3 (microbiome–survival association testing), reproducing results like the tylosin T1D trial (HR = 1.692, P = 0.005).

## When to use

Use when input is a phyloseq object (biom.Rdata) or its four component files with survival time, censoring indicator, treatment, and covariate columns, and the goal is a covariate-adjusted Cox model of treatment or CLR-transformed microbial abundance on survival.

## When NOT to use

- Input is raw 16S sequence data still needing QIIME 2.0 processing — use MiSurv's Data Processing module first (quality control, 97% OTU picking, ChimeraSlayer, rarefaction via rarefy_even_depth)
- The analysis goal is comparative microbial composition between treatment groups without survival data (Module 2) or prediction modeling of survival from microbial taxa (Module 4)
- The response is continuous or binary rather than a survival (time-to-event) response — MiCloud, not MiSurv, is the appropriate platform

## Inputs

- phyloseq object (biom.Rdata) from the MiSurvGit Data folder
- Four individual files: otu.tab.txt, sam.dat.txt, tax.tab.txt, tree.tre
- Sample metadata columns: T1Dweek (survival time), T1D (event/censoring indicator), Antibiotics (treatment group), Sex (covariate)
- Alternative raw data source: QIITA study ID 10508

## Outputs

- cox_survival_results.csv with treatment-group hazard ratio, confidence interval, P-value, group sizes (88 control / 84 tylosin), and censored/event counts (56/116)
- hazard_ratios.csv with per-taxon exp(coef) hazard ratios from CLR-transformed abundance Cox models

## How to apply

Load the processed phyloseq object (biom.Rdata) or the equivalent otu.tab.txt/sam.dat.txt/tax.tab.txt/tree.tre files from the MiSurvGit Data folder (QIITA study ID 10508). For treatment-group comparison (Module 1), extract T1Dweek (survival time), T1D (event indicator), Antibiotics (control vs tylosin factor), and Sex, then fit a Cox proportional hazards regression Surv(T1Dweek, T1D) ~ Antibiotics + Sex using the R 'survival' package. For microbiome–survival association (Module 3), aggregate OTU counts to the target taxonomic rank, convert to relative abundance, apply a centered log-ratio (CLR) transformation, and fit one Cox model per taxon with Sex as covariate, optionally restricted to a treatment subgroup. Extract exp(coef) hazard ratios, confidence intervals, and Wald/log-rank P-values from the model summary, and validate against reference values (e.g., treatment HR = 1.692, P = 0.005; Verrucomicrobia HR = 1.253, Proteobacteria 1.236, Firmicutes 1.262, Actinobacteria 0.759, Tenericutes 0.796).

## Related tools

- **MiSurv** (Web cloud or local R/Shiny platform providing Module 1 (comparative survival analysis) and Module 3 (microbiome–survival association testing) with subgroup analysis options) — https://github.com/wg99526/MiSurvGit
- **R (survival package)** (Fits the Cox proportional hazards model via coxph on Surv(T1Dweek, T1D) ~ Antibiotics + Sex)
- **shiny** (Required package to launch the local MiSurv implementation from the GitHub repository) — https://github.com/wg99526/MiSurvGit
- **phyloseq** (Provides the phyloseq object format (biom.Rdata) holding the OTU table, sample data, taxonomy, and tree; also used for rarefy_even_depth in preprocessing)

## Examples

```
library(survival); fit <- coxph(Surv(T1Dweek, T1D) ~ Antibiotics + Sex, data = sam.dat); summary(fit)
```

## Evaluation signals

- Module 1 reproduction check: P-value = 0.005, HR = 1.692, group sizes 88 (control) / 84 (tylosin), and censored/events = 56/116 match the reported results
- Module 3 phylum HRs match Fig. 6 reference values within tolerance: Verrucomicrobia 1.253, Proteobacteria 1.236, Firmicutes 1.262, Actinobacteria 0.759, Tenericutes 0.796
- Output CSVs (cox_survival_results.csv, hazard_ratios.csv) contain hazard ratio, confidence interval, and P-value columns with non-missing values for all reported taxa
- The Cox model formula includes the Sex covariate (or declared covariates) and uses Surv(time, event) with correct event coding
- Input phyloseq object or the four component files load successfully in R (>4.2.0) and metadata columns (T1Dweek, T1D, Antibiotics, Sex) are present

## Limitations

- Subgroup analysis (Module 3/4) exists because microbiome effects on survival are often assumed to differ between treatment groups — pooled models may mask group-specific effects
- The web server (http://misurv.micloud.kr) has limited hardware specification, so heavy jobs may need local implementation
- No version numbers are given for R packages used (phyloseq, survival, MiRKAT), so exact reproducibility of package versions is not guaranteed
- Module 1 performs exploratory comparative survival analysis not involving microbiome data; microbiome-specific inference requires Module 3

## Evidence

- [readme] Module 1 definition: "Module 1 : Comparative survival analysis between treatment groups, not involving microbiome data, with or without covariate adjustment(s)."
- [readme] Module 3 definition: "Module 3 : Association testing between microbial composition and survival responses with or without covariate adjustment(s)."
- [methods] Cox model specification from reproduction workflow: [evidence span withheld — anchor status: not_found]
- [results] Expected statistics to validate against: [evidence span withheld — anchor status: not_found]
- [readme] Data availability: "The raw sequence data are deposited in QIITA (https://qiita.ucsd.edu) with the ID number 10508"
- [discussion] Subgroup rationale: "we added the subgroup analysis user option because it is often assumed that the effects of microbiome on survival responses are different between treatment groups"

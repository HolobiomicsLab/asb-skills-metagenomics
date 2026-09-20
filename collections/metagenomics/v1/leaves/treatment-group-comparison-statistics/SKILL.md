---
name: treatment-group-comparison-statistics
description: Use when comparing a survival outcome (time-to-event with censoring)
  between pre-defined treatment groups and the comparison must be adjusted for covariates
  such as Sex via Cox regression.
when_to_use_negative:
- The analysis goal involves microbiome composition rather than a pure treatment-group
  survival comparison — MiSurv Modules 2–4 handle composition, association testing,
  and prediction modeling instead; Module 1 explicitly does not involve microbiome
  data.
- The response is continuous or binary rather than a survival (time-to-event) response
  — the authors note MiSurv is designed for survival responses while MiCloud handles
  continuous or binary responses.
- No censoring structure or survival time is available in the metadata, so a Cox model
  on Surv(time, event) cannot be constructed.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MiSurv
  role: Web cloud / local R Shiny platform whose Module 1 performs the comparative
    survival analysis between treatment groups with covariate adjustment
  repo: https://github.com/wg99526/MiSurvGit
- name: R (survival package)
  role: Used to fit the covariate-adjusted Cox proportional hazards model Surv(T1Dweek,
    T1D) ~ Antibiotics + Sex and extract HR, CI, and P-value
- name: shiny
  role: Dependency used to launch the MiSurv app locally for the Module 1 workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/spectrum.05059-22
    title: 'MiSurv: an Integrative Web Cloud Platform for User-Friendly Microbiome
      Data Analysis with Survival Responses'
  source_content_hash: sha256:2fccebe0a260743de3c1e5221339a5cc1adaff4bcac111437a6be30587fdcae6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wg99526/MiSurvGit
  tools:
  - MiSurv
  - R
  - R (survival package)
  - shiny
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# treatment-group-comparison-statistics

## Summary

Fits a covariate-adjusted Cox proportional hazards model in R's 'survival' package to compare survival rates between treatment groups (e.g., control vs tylosin) in a microbiome survival-data workflow, as performed by Module 1 of the MiSurv platform.

## When to use

Use when comparing a survival outcome (time-to-event with censoring) between pre-defined treatment groups and the comparison must be adjusted for covariates such as Sex via Cox regression.

## When NOT to use

- The analysis goal involves microbiome composition rather than a pure treatment-group survival comparison — MiSurv Modules 2–4 handle composition, association testing, and prediction modeling instead; Module 1 explicitly does not involve microbiome data.
- The response is continuous or binary rather than a survival (time-to-event) response — the authors note MiSurv is designed for survival responses while MiCloud handles continuous or binary responses.
- No censoring structure or survival time is available in the metadata, so a Cox model on Surv(time, event) cannot be constructed.

## Inputs

- phyloseq object biom.Rdata (final processed data from Gu et al. 2023)
- sam.dat.txt sample metadata table with columns T1Dweek, T1D, Antibiotics, Sex
- QIITA study ID 10508 raw sequence data (https://qiita.ucsd.edu/study/description/10508)

## Outputs

- cox_survival_results.csv containing treatment HR, 95% CI, P-value, group sample sizes (88/84), and censored/event counts (56/116)
- Model summary of the fitted Cox regression with covariate-adjusted treatment effect

## How to apply

Load the final processed phyloseq object (biom.Rdata) or its component files (sam.dat.txt) from the MiSurv Data folder, also obtainable via QIITA study ID 10508. Extract the sample-data columns T1Dweek (survival time in weeks to T1D onset), T1D (event indicator), Antibiotics (treatment), and Sex, encoding Antibiotics as the control vs tylosin factor. Fit coxph with Surv(T1Dweek, T1D) ~ Antibiotics + Sex using the R 'survival' package, then extract the treatment hazard ratio, its confidence interval, and the Wald/log-rank P-value from the model summary. Also tabulate group sample sizes and the numbers of right-censored mice vs T1D-onset events, and write all statistics to cox_survival_results.csv. This corresponds to MiSurv Module 1, which performs comparative survival analysis between treatment groups 'with or without covariate adjustment(s)' and does not involve microbiome data.

## Related tools

- **MiSurv** (Web cloud / local R Shiny platform whose Module 1 performs the comparative survival analysis between treatment groups with covariate adjustment) — https://github.com/wg99526/MiSurvGit
- **R (survival package)** (Used to fit the covariate-adjusted Cox proportional hazards model Surv(T1Dweek, T1D) ~ Antibiotics + Sex and extract HR, CI, and P-value)
- **shiny** (Dependency used to launch the MiSurv app locally for the Module 1 workflow)

## Examples

```
library(survival); fit <- coxph(Surv(T1Dweek, T1D) ~ Antibiotics + Sex, data = sam.dat); summary(fit)
```

## Evaluation signals

- Reproduced Cox model P-value for the Antibiotics term equals 0.005 (significant difference between control and tylosin groups).
- Treatment-group hazard ratio equals 1.692.
- Tabulated group sizes are 88 (control) and 84 (tylosin).
- Censored-vs-event counts total 56 censored mice and 116 T1D-onset events, matching the reported Module 1 results.
- cox_survival_results.csv is written with the HR, CI, P-value, group sizes, and censored/event counts.

## Limitations

- Module 1 performs only basic exploratory data analysis and does not involve microbiome data, so it cannot establish microbiome–survival associations.
- The document gives no version numbers for the R packages used (e.g., phyloseq, survival), and R >4.2.0 is required for local implementation.
- Results are grounded in the Zhang et al. NOD-mouse T1D example; extending to other datasets requires the same T1Dweek/T1D/Antibiotics/Sex column semantics to hold.

## Evidence

- [intro] Module 1 performs the covariate-adjusted Cox comparison: "Module 1: Comparative survival analysis between treatment groups."
- [readme] Module 1 supports covariate adjustment and excludes microbiome data: "Module 1 : Comparative survival analysis between treatment groups, not involving microbiome data, with or without covariate adjustment(s)."
- [readme] Example dataset files and QIITA deposit for reproduction: "The raw sequence data are deposited in QIITA (https://qiita.ucsd.edu) with the ID number 10508"
- [discussion] Final processed data used in the paper are provided for reproduction: "we stored the all the final processed data that we used in our paper (Gu et al., 2023) in the above Data folder as ‘phyloseq’ format"
- [readme] R version requirement for local implementation: "For local implementation using this GitHub repository, recent version of **R (>4.2.0)** is needed."

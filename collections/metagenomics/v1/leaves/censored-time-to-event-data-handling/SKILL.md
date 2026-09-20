---
name: censored-time-to-event-data-handling
description: Use when the outcome is right-censored time-to-event data (survival time
  plus event/censor indicator) and the goal is comparing treatment groups with or
  without covariate adjustment via a Cox model.
when_to_use_negative:
- The research question concerns association between microbial composition and survival
  — use MiSurv Module 3 (association testing) instead of Module 1, which does not
  involve microbiome data.
- The outcome is continuous or binary rather than time-to-event — MiSurv's companion
  platform MiCloud handles continuous/binary responses, not MiSurv.
- The goal is comparative microbial composition between treatment groups without survival
  data — use MiSurv Module 2.
- Survival time lacks censoring information — a naive comparison ignoring the censored/event
  indicator would misuse this skill.
edam_operation: http://edamontology.org/operation_3575
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0625
tools:
- name: MiSurv
  role: Web cloud platform (Shiny app) whose Module 1 performs comparative survival
    analysis between treatment groups with or without covariate adjustment(s), including
    follow-up-period quality control
  repo: https://github.com/wg99526/MiSurvGit
- name: R survival package
  role: Fits the Cox proportional hazards regression Surv(T1Dweek, T1D) ~ Antibiotics
    + Sex and supplies HR, CI, and P-value
- name: shiny
  role: Required R dependency to launch the MiSurv app locally via runGitHub
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/spectrum.05059-22
    title: 'MiSurv: an Integrative Web Cloud Platform for User-Friendly Microbiome
      Data Analysis with Survival Responses'
  source_content_hash: sha256:76f9a577abc189b4fccce4add847905ea28b8d30818a59e1f23b1d654d5f888c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wg99526/MiSurvGit
  tools:
  - MiSurv
  - R
  - R survival package
  - shiny
  edam_operation: http://edamontology.org/operation_3575
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# censored-time-to-event-data-handling

## Summary

Fit covariate-adjusted Cox proportional hazards regression on right-censored time-to-event microbiome trial data (e.g., Surv(T1Dweek, T1D) ~ Antibiotics + Sex in the MiSurv T1D example), extracting hazard ratios, confidence intervals, and Wald/log-rank P-values for treatment-group survival comparison.

## When to use

Use when the outcome is right-censored time-to-event data (survival time plus event/censor indicator) and the goal is comparing treatment groups with or without covariate adjustment via a Cox model.

## When NOT to use

- The research question concerns association between microbial composition and survival — use MiSurv Module 3 (association testing) instead of Module 1, which does not involve microbiome data.
- The outcome is continuous or binary rather than time-to-event — MiSurv's companion platform MiCloud handles continuous/binary responses, not MiSurv.
- The goal is comparative microbial composition between treatment groups without survival data — use MiSurv Module 2.
- Survival time lacks censoring information — a naive comparison ignoring the censored/event indicator would misuse this skill.

## Inputs

- Sample metadata table with survival time (T1Dweek), event indicator (T1D), treatment (Antibiotics: control vs tylosin), and covariates (Sex) — e.g., sam.dat.txt
- Final processed phyloseq object (biom.Rdata) from the MiSurv GitHub Data folder
- User-specified analytic plan: survival time variable, censored/event variable, follow-up period, subgroup option (MiSurv Data Processing module)

## Outputs

- Cox model treatment hazard ratio with 95% confidence interval and Wald/log-rank P-value
- Group-level counts: sample sizes per treatment group and numbers right-censored vs events
- Statistics table written to cox_survival_results.csv

## How to apply

Extract the survival time column (e.g., T1Dweek in weeks to T1D onset), the right-censoring/event indicator (T1D), the treatment factor (Antibiotics encoded as control vs tylosin), and any covariates (Sex) from the sample metadata (sam.dat.txt) or the phyloseq object (biom.Rdata). Fit a Cox proportional hazards regression with the R 'survival' package using Surv(T1Dweek, T1D) ~ Antibiotics + Sex; the semi-parametric Cox model correctly handles right-censored observations without distributional assumptions on the survival time. Extract the treatment-group hazard ratio, its confidence interval, and the Wald/log-rank P-value from the fitted model summary, and additionally tabulate group sample sizes (control vs tylosin) and the counts of right-censored mice versus T1D-onset events. In MiSurv this corresponds to Module 1: comparative survival analysis between treatment groups, optionally with covariate adjustment(s), applied after the Data Processing module (with follow-up period quality control).

## Related tools

- **MiSurv** (Web cloud platform (Shiny app) whose Module 1 performs comparative survival analysis between treatment groups with or without covariate adjustment(s), including follow-up-period quality control) — https://github.com/wg99526/MiSurvGit
- **R survival package** (Fits the Cox proportional hazards regression Surv(T1Dweek, T1D) ~ Antibiotics + Sex and supplies HR, CI, and P-value)
- **shiny** (Required R dependency to launch the MiSurv app locally via runGitHub)

## Examples

```
R: library(survival); fit <- coxph(Surv(T1Dweek, T1D) ~ Antibiotics + Sex, data = sam.dat); summary(fit)
```

## Evaluation signals

- Reproduced Cox statistics match the published Module 1 results: P = 0.005, HR = 1.692
- Group sample sizes match 88 (control) / 84 (tylosin) and censored/events totals match 56/116
- Event indicator is binary (0/1) and the Surv object is right-censored with follow-up period respected (MiSurv quality control adjusts the follow-up period)
- cox_survival_results.csv contains HR, CI, P-value, and group/censor counts with consistent n = 172 total samples
- Local run uses R > 4.2.0 and the app launches via runGitHub('MiSurvGit', 'wg99526', ref = 'main')

## Limitations

- No version numbers are given for the R packages used (e.g., phyloseq, survival), which may hinder exact reproduction
- MiSurv has no software license mentioned and hardware specification may need improvement, implying current web-server capacity limits
- Module 1 performs basic exploratory comparative survival analysis and does not involve microbiome data; microbiome effects on survival may differ between treatment groups, which is why subgroup analysis options exist in Modules 3 and 4
- Example analysis relies on the NOD-mouse T1D trial (Zhang et al., QIITA study 10508); extending to other studies requires valid survival-time and censoring metadata

## Evidence

- [readme] Module 1 performs comparative survival analysis: "Module 1 : Comparative survival analysis between treatment groups, not involving microbiome data, with or without covariate adjustment(s)."
- [intro] Article intro module description: "Module 1: Comparative survival analysis between treatment groups."
- [readme] Example data location: "We stored the all the final processed data that we used in our paper (Gu et al., 2023) in the above Data folder as ‘phyloseq’ format (see the file named ‘biom.Rdata’)"
- [discussion] Quality control on follow-up: "we added the quality control to adjust the follow-up period"
- [discussion] Subgroup rationale: "we added the subgroup analysis user option because it is often assumed that the effects of microbiome on survival responses are different between treatment groups"

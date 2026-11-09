---
name: microbiome-sample-metadata-extraction
description: Use when a microbiome survival analysis needs clinical/design variables
  (survival time, event indicator, treatment group, covariates) pulled from a sample-data
  table such as MiSurv's sam.dat.txt or the phyloseq object biom.Rdata.
when_to_use_negative:
- The goal is microbial composition or diversity analysis (MiSurv Modules 2–4), which
  additionally requires otu.tab.txt, tax.tab.txt, and tree.tre rather than metadata
  alone.
- The input is raw 16S reads or a QIIME 2.0 feature table that still requires the
  data-processing module (quality control, rarefaction via rarefy_even_depth) before
  metadata extraction is meaningful.
- The metadata lacks survival columns (survival time and censoring indicator), so
  no Surv() object can be constructed for comparative survival analysis.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0625
tools:
- name: MiSurv
  role: Web/Shiny platform whose Data folder provides the sample metadata (sam.dat.txt,
    biom.Rdata) used for survival-response extraction
  repo: https://github.com/wg99526/MiSurvGit
- name: R
  role: Runtime for loading the phyloseq object and encoding the Antibiotics factor
    for downstream Cox regression with the survival package
- name: shiny
  role: Dependency used to launch the MiSurv app locally via runGitHub to access the
    Data folder
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/spectrum.05059-22
    title: 'MiSurv: an Integrative Web Cloud Platform for User-Friendly Microbiome
      Data Analysis with Survival Responses'
  source_content_hash: sha256:f6955a48b0e793394d6c1db6736bdf9818a50f1a33f176edf1e7790defd309d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wg99526/MiSurvGit
  tools:
  - MiSurv
  - R
  - shiny
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# microbiome-sample-metadata-extraction

## Summary

Extracts the survival-relevant columns (T1Dweek, T1D, Antibiotics, Sex) from the MiSurv sample metadata file (sam.dat.txt) of the Zhang et al. T1D NOD-mouse study (QIITA 10508) so they can feed a covariate-adjusted Cox model in R's survival package.

## When to use

Use when a microbiome survival analysis needs clinical/design variables (survival time, event indicator, treatment group, covariates) pulled from a sample-data table such as MiSurv's sam.dat.txt or the phyloseq object biom.Rdata.

## When NOT to use

- The goal is microbial composition or diversity analysis (MiSurv Modules 2–4), which additionally requires otu.tab.txt, tax.tab.txt, and tree.tre rather than metadata alone.
- The input is raw 16S reads or a QIIME 2.0 feature table that still requires the data-processing module (quality control, rarefaction via rarefy_even_depth) before metadata extraction is meaningful.
- The metadata lacks survival columns (survival time and censoring indicator), so no Surv() object can be constructed for comparative survival analysis.

## Inputs

- sam.dat.txt (sample metadata table from MiSurv Data folder)
- biom.Rdata (phyloseq-format object containing the same sample data)
- QIITA study ID 10508 raw study data

## Outputs

- Encoded data frame with columns T1Dweek (numeric survival time), T1D (event indicator), Antibiotics (control vs tylosin factor), Sex (covariate)
- cox_survival_results.csv (group sample sizes, censored/event counts, HR, CI, P-value)

## How to apply

Load either the phyloseq object biom.Rdata or the flat sample metadata file sam.dat.txt from the Data folder of the MiSurv GitHub repository (or retrieve the raw study from QIITA study ID 10508). From the sample-data table, extract the specific columns T1Dweek (survival time in weeks to T1D onset), T1D (right-censoring/event indicator), Antibiotics (treatment), and Sex (covariate), and encode Antibiotics as the control vs tylosin group factor. Verify the extracted group sizes against the published design: 88 control and 84 tylosin mice with 56 censored and 116 T1D-onset events. These encoded columns then serve directly as the response and covariates in a Cox proportional hazards model, Surv(T1Dweek, T1D) ~ Antibiotics + Sex, fitted with the R survival package.

## Related tools

- **MiSurv** (Web/Shiny platform whose Data folder provides the sample metadata (sam.dat.txt, biom.Rdata) used for survival-response extraction) — https://github.com/wg99526/MiSurvGit
- **R** (Runtime for loading the phyloseq object and encoding the Antibiotics factor for downstream Cox regression with the survival package)
- **shiny** (Dependency used to launch the MiSurv app locally via runGitHub to access the Data folder)

## Examples

```
library(shiny); runGitHub("MiSurvGit", "wg99526", ref = "main")
```

## Evaluation signals

- Group sample sizes after factor encoding must be 88 control and 84 tylosin mice.
- Censoring tally must give 56 right-censored mice and 116 T1D-onset events.
- Downstream Cox model on the extracted columns must reproduce Module 1 values: HR = 1.692, P = 0.005.
- The four required columns (T1Dweek, T1D, Antibiotics, Sex) must be present and non-missing in sam.dat.txt before fitting.
- Extracted statistics must be written to cox_survival_results.csv matching the reported Module 1 results.

## Limitations

- No version numbers are given for the R packages used (e.g., phyloseq, survival), so reproducibility depends on matching the article's environment; R > 4.2.0 is required for local MiSurv use.
- The metadata correspond to the Zhang et al. NOD-mouse T1D study; column names like T1Dweek and Antibiotics are study-specific and must be remapped for other datasets.
- The Data Processing module's quality control (e.g., adjusting the follow-up period) must be applied first; using unadjusted metadata can distort survival times.

## Evidence

- [other] Sample-data column extraction from sam.dat.txt: "with four individual files (see the files named 'otu.tab.txt,' 'sam.dat.txt,' 'tax.tab.txt,' and 'tree.tre')"
- [methods] Raw data availability in QIITA: "The raw sequence data are deposited in QIITA (https://qiita.ucsd.edu) with the ID number 10508"
- [readme] Processed phyloseq object provided: "We stored the all the final processed data that we used in our paper (Gu et al., 2023) in the above Data folder as ‘phyloseq’ format (see the file named ‘biom.Rdata’)"
- [intro] Module 1 comparative survival analysis scope: "Module 1: Comparative survival analysis between treatment groups, not involving microbiome data, with or without covariate adjustment(s)."
- [readme] Local implementation requirements: "For local implementation using this GitHub repository, recent version of **R (>4.2.0)** is needed."

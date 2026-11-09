---
name: nominal-alpha-rejection-rate-analysis
description: Use when validating that a differential abundance test controls false
  positive (type I) error on zero-inflated beta relative abundance data by simulating
  under the null and comparing empirical rejection rates at alpha=0.01/0.05/0.1 to
  nominal levels.
when_to_use_negative:
- When the goal is power estimation or differential abundance detection on real data
  with a true between-group effect (the null must hold for a type I error assessment)
- When the input is non-relative-abundance data that does not resemble a zero-inflated
  beta distribution (the article notes t-tests/Wilcoxon/linear models and arcsin sqrt
  transformation fail to address this distribution)
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: metamicrobiomeR
  role: Provides GAMLSS-BEZI differential abundance testing (taxa.compare with propmed.rel="gamlss")
    used to test simulated null data; installed from GitHub
  repo: https://github.com/nhanhocu/metamicrobiomeR
- name: devtools
  role: Used to install metamicrobiomeR from GitHub (install_github)
- name: GAMLSS-BEZI
  role: Zero-inflated beta GAMLSS family used for the between-group comparison whose
    type I error is being estimated
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-019-2744-2
    title: 'metamicrobiomeR: an R package for analysis of microbiome relative abundance
      data using zero-inflated beta GAMLSS and meta-analysis across studies using
      random effects models'
  source_content_hash: sha256:fae66a826237d8c84f98e5eeb17e98f7ee8118371ac33bd240d7bf68ceb9c692
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nhanhocu/metamicrobiomeR
  tools:
  - GAMLSS-BEZI
  - R
  - devtools
  - metamicrobiomeR
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# nominal-alpha-rejection-rate-analysis

## Summary

Empirical type I error assessment for zero-inflated beta GAMLSS (GAMLSS-BEZI) tests on microbiome relative abundance data: simulate null data, count rejections at nominal alpha levels, and verify rates match their nominal levels within Monte Carlo tolerance.

## When to use

Use when validating that a differential abundance test controls false positive (type I) error on zero-inflated beta relative abundance data by simulating under the null and comparing empirical rejection rates at alpha=0.01/0.05/0.1 to nominal levels.

## When NOT to use

- When the goal is power estimation or differential abundance detection on real data with a true between-group effect (the null must hold for a type I error assessment)
- When the input is non-relative-abundance data that does not resemble a zero-inflated beta distribution (the article notes t-tests/Wilcoxon/linear models and arcsin sqrt transformation fail to address this distribution)

## Inputs

- Simulated zero-inflated beta (BEZI) relative abundance data generated under the null (mu1=mu2=0.5, alpha1=alpha2=0.5, phi=5) at n=10, 100, 500
- metamicrobiomeR package (installed via devtools from github.com/nhanhocu/metamicrobiomeR)
- Package tutorial (Additional file 1) simulation workflow specification

## Outputs

- Empirical type I error rates per sample size and alpha level (counts of null rejections over 1000 Monte Carlo replicates)
- Results table of rejection rates (rows = sample sizes, columns = alpha levels 0.01/0.05/0.1), e.g. written to type1_error_rates.csv
- Verdict on whether rates fall within ~2 standard errors of nominal levels, replicating Table 1

## How to apply

Install metamicrobiomeR from GitHub (devtools::install_github("nhanhocu/metamicrobiomeR")) and load it with the gamlss/gamlss.dist packages, since taxa.compare with propmed.rel="gamlss" fits the zero-inflated beta GAMLSS. Following the simulation workflow in the package tutorial (Additional file 1), generate BEZI-distributed relative abundance data under the null hypothesis with mu1=mu2=0.5, alpha1=alpha2=0.5, phi=5, for sample sizes n=10, 100, and 500. For each sample size, repeat null data generation and the GAMLSS-BEZI between-group comparison for 1000 Monte Carlo replicates. Count null-hypothesis rejections at alpha=0.01, 0.05, and 0.1, convert counts to empirical type I error rates, and assemble a table (rows = sample sizes, columns = alpha levels). The rationale is that rejecting the null when no effect exists should occur at exactly the nominal rate; each empirical rate should lie within ~2 binomial standard errors of nominal, reproducing the well-controlled pattern reported in Table 1 for GAMLSS-BEZI (vs LMAS).

## Related tools

- **metamicrobiomeR** (Provides GAMLSS-BEZI differential abundance testing (taxa.compare with propmed.rel="gamlss") used to test simulated null data; installed from GitHub) — https://github.com/nhanhocu/metamicrobiomeR
- **devtools** (Used to install metamicrobiomeR from GitHub (install_github))
- **GAMLSS-BEZI** (Zero-inflated beta GAMLSS family used for the between-group comparison whose type I error is being estimated)

## Examples

```
library(devtools); install_github("nhanhocu/metamicrobiomeR"); library(metamicrobiomeR); library(gamlss.dist)
```

## Evaluation signals

- Each empirical type I error rate lies within ~2 standard errors of the binomial proportion (over 1000 replicates) of its nominal level (0.01, 0.05, or 0.1)
- Reject rates at alpha=0.1 exceed those at alpha=0.05, which exceed those at alpha=0.01, for every sample size
- Results reproduce the well-controlled type I error pattern of GAMLSS-BEZI reported in Table 1 of the paper
- The results table has the expected shape: 3 sample-size rows (n=10, 100, 500) x 3 alpha-level columns
- Each rate is computed from exactly 1000 Monte Carlo replicates per setting

## Limitations

- The simulation workflow depends on Additional file 1 (package tutorial), which is not inline in the article text and must be obtained from the paper/repository
- The GitHub package version has minor changes versus the CRAN version; the README recommends the CRAN version, which may affect exact reproducibility
- Results depend on the specific null parameters (mu=0.5, alpha=0.5, phi=5) and do not establish calibration at other parameter values or higher sample sizes

## Evidence

- [abstract] Both simulation studies and application to real microbiome data demonstrate that GAMLSS-BEZI well performs in testing differential relative abundances of microbial taxonomies.: "Both simulation studies and application to real microbiome data demonstrate that GAMLSS-BEZI well performs in testing differential relative abundances of microbial taxonomies."
- [results] Type I errors were well controlled in both GAMLSS-BEZI and LMAS in the simulation study (Table 1).: [evidence span withheld — anchor status: not_found]
- [methods] Install from GitHub as shown in the paper methods.: "install_github("nhanhocu/metamicrobiomeR") library(metamicrobiomeR)"
- [readme] README points to the published paper and simulation workflow.: "The published paper describing the methodology as well as simulations and example analyses/results can be found at BMC Bioinformatics."

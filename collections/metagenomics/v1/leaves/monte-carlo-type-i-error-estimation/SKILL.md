---
name: monte-carlo-type-i-error-estimation
description: Use when validating that a zero-inflated beta GAMLSS test controls false
  positives on simulated microbiome relative abundance data under the null hypothesis,
  e.g. reproducing Table 1 type I error comparisons at alpha = 0.01, 0.05, 0.1.
when_to_use_negative:
- Data contain real observed differential abundance (non-null effect) — this skill
  estimates false positive rates only under the null hypothesis
- The data are not zero-inflated beta microbiome relative abundances (e.g. counts
  or compositional tables without relative abundance transformation), since GAMLSS-BEZI
  is designed specifically for that distribution
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: metamicrobiomeR
  role: Provides GAMLSS-BEZI differential abundance testing (taxa.compare) used to
    compute p-values in each Monte Carlo replicate
  repo: https://github.com/nhanhocu/metamicrobiomeR
- name: R
  role: Programming language in which the simulation, replicate loop, and rate assembly
    are implemented (R 3.4.2 or higher)
- name: devtools
  role: Used to install metamicrobiomeR from GitHub via install_github
- name: GAMLSS-BEZI
  role: Zero-inflated beta GAMLSS family used both to simulate null relative abundance
    data and to test between-group differences
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-019-2744-2
    title: 'metamicrobiomeR: an R package for analysis of microbiome relative abundance
      data using zero-inflated beta GAMLSS and meta-analysis across studies using
      random effects models'
  source_content_hash: sha256:f94ca768cfcf0e07d1b99d10ad4e399854c5c9a37083fdcca24572b70e537d9d
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
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# monte-carlo-type-i-error-estimation

## Summary

A Monte Carlo simulation skill for estimating empirical type I error rates of the GAMLSS-BEZI (zero-inflated beta) differential abundance test in metamicrobiomeR, by repeatedly simulating BEZI-distributed microbiome relative abundance data under the null and counting rejections at nominal alpha levels.

## When to use

Use when validating that a zero-inflated beta GAMLSS test controls false positives on simulated microbiome relative abundance data under the null hypothesis, e.g. reproducing Table 1 type I error comparisons at alpha = 0.01, 0.05, 0.1.

## When NOT to use

- Data contain real observed differential abundance (non-null effect) — this skill estimates false positive rates only under the null hypothesis
- The data are not zero-inflated beta microbiome relative abundances (e.g. counts or compositional tables without relative abundance transformation), since GAMLSS-BEZI is designed specifically for that distribution

## Inputs

- Simulated BEZI relative abundance matrices under the null hypothesis (mu=0.5, alpha=0.5, phi=5) for n=10, 100, 500
- metamicrobiomeR package installed from GitHub (nhanhocu/metamicrobiomeR)
- gamlss and gamlss.dist R packages
- Monte Carlo replicate count (1000) and nominal alpha levels (0.01, 0.05, 0.1)

## Outputs

- Empirical type I error rate table (rows = sample sizes n=10/100/500, columns = alpha levels 0.01/0.05/0.1), e.g. type1_error_rates.csv
- Rejection counts per Monte Carlo replicate per alpha level per sample size

## How to apply

Generate BEZI-distributed relative abundance data under the null (mu1=mu2=0.5, alpha1=alpha2=0.5, phi=5) for each sample size n=10, 100, and 500, following the metamicrobiomeR package tutorial (Additional file 1). For each setting, run 1000 Monte Carlo replicates of the GAMLSS-BEZI between-group comparison via taxa.compare with propmed.rel='gamlss'. At each replicate, count null-hypothesis rejections at alpha = 0.01, 0.05, and 0.1, then convert counts to empirical type I error rates. Assemble rates into a table with rows as sample sizes and columns as alpha levels (e.g. type1_error_rates.csv) and compare against LMAS. Rationale: GAMLSS-BEZI directly addresses the zero-inflated beta distribution of microbiome relative abundance data, so its type I error control under the true generating distribution is the property being verified.

## Related tools

- **metamicrobiomeR** (Provides GAMLSS-BEZI differential abundance testing (taxa.compare) used to compute p-values in each Monte Carlo replicate) — https://github.com/nhanhocu/metamicrobiomeR
- **R** (Programming language in which the simulation, replicate loop, and rate assembly are implemented (R 3.4.2 or higher))
- **devtools** (Used to install metamicrobiomeR from GitHub via install_github)
- **GAMLSS-BEZI** (Zero-inflated beta GAMLSS family used both to simulate null relative abundance data and to test between-group differences)

## Examples

```
library(devtools); install_github("nhanhocu/metamicrobiomeR"); library(metamicrobiomeR); y <- gamlss.dist::rBEZI(n, mu=0.5, sigma=0.5, nu=0.5, tau=5)
```

## Evaluation signals

- Each empirical type I error rate lies within ~2 standard errors of the binomial proportion (over 1000 replicates) of its nominal level 0.01, 0.05, or 0.1
- Rejection counts are consistent with nominal alpha: expected counts ≈ 10, 50, and 100 per 1000 replicates at alpha = 0.01, 0.05, 0.1 respectively
- Results table has the expected shape: 3 rows (n=10, 100, 500) × 3 columns (alpha = 0.01, 0.05, 0.1)
- Empirical rates match the well-controlled pattern reported in Table 1 for both GAMLSS-BEZI and LMAS
- Simulation parameters match the null specification (mu1=mu2=0.5, alpha1=alpha2=0.5, phi=5) for all sample sizes

## Limitations

- The Additional file 1 tutorial containing the simulation workflow is referenced but not inline-accessible in the article text, so exact simulation code must be obtained from the supplementary file
- Type I error estimates carry Monte Carlo noise (~2 binomial SEs over 1000 replicates); small alpha levels like 0.01 have wide relative tolerance
- The GitHub version differs from the CRAN version of metamicrobiomeR; the README notes some updates/minor changes and recommends the CRAN version

## Evidence

- [abstract] Both simulation studies and application to real microbiome data demonstrate that GAMLSS-BEZI well performs in testing differential relative abundances of microbial taxonomies.: "Both simulation studies and application to real microbiome data demonstrate that GAMLSS-BEZI well performs in testing differential relative abundances of microbial taxonomies."
- [methods] install_github("nhanhocu/metamicrobiomeR") library(metamicrobiomeR): "install_github("nhanhocu/metamicrobiomeR") library(metamicrobiomeR)"
- [discussion] The advantages of GAMLSS-BEZI are: 1) it directly address the distribution of microbiome relative abundance data: "The advantages of GAMLSS-BEZI are: 1) it directly address the distribution of microbiome relative abundance data"
- [discussion] Other requirements: R 3.4.2 or higher.: "Programming language: R. Other requirements: R 3.4.2 or higher."
- [readme] The published paper describing the methodology as well as simulations and example analyses/results can be found at BMC Bioinformatics. 2019 Apr 16;20(1):188.: "The published paper describing the methodology as well as simulations and example analyses/results can be found at BMC Bioinformatics. 2019 Apr 16;20(1):188."

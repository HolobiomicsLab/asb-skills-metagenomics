---
name: microbiome-relabundance-null-data-generation
description: Use when evaluating the false positive control of a differential abundance
  test on zero-inflated beta relative abundance data — e.g. simulating null data with
  mu1=mu2=0.5, alpha1=alpha2=0.5, phi=5 to reproduce Table 1 type I error rates at
  alpha=0.01/0.05/0.1.
when_to_use_negative:
- The goal is estimating statistical power or differential abundance effects — this
  skill simulates strictly under the null (mu1=mu2), so rejections are false positives,
  not true discoveries
- The data are real observed microbiome relative abundances rather than simulated
  BEZI data — use the package's taxa.compare/meta.taxa analysis workflow instead
- The abundance distribution of interest is not zero-inflated beta (e.g. count data)
  — BEZI simulation parameters would not be applicable
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: metamicrobiomeR
  role: R package providing the GAMLSS-BEZI framework and tutorial (Additional file
    1) whose simulation workflow defines the null-data generation and between-group
    comparison
  repo: https://github.com/nhanhocu/metamicrobiomeR
- name: gamlss.dist
  role: Supplies the zero-inflated beta (BEZI) family used to generate null relative
    abundance data
- name: devtools
  role: Used to install the metamicrobiomeR package from GitHub via install_github
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-019-2744-2
    title: 'metamicrobiomeR: an R package for analysis of microbiome relative abundance
      data using zero-inflated beta GAMLSS and meta-analysis across studies using
      random effects models'
  source_content_hash: sha256:df0b1bc8bdbaf67ad97236447c93ed8e3b791f0d6f3730c59724586823d8b69b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nhanhocu/metamicrobiomeR
  tools:
  - R
  - devtools
  - gamlss.dist
  - metamicrobiomeR
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# microbiome-relabundance-null-data-generation

## Summary

Generates simulated microbiome relative abundance data drawn from a zero-inflated beta (BEZI) distribution under the null hypothesis (no between-group difference) so that empirical type I error rates of GAMLSS-BEZI tests can be estimated via Monte Carlo replication.

## When to use

Use when evaluating the false positive control of a differential abundance test on zero-inflated beta relative abundance data — e.g. simulating null data with mu1=mu2=0.5, alpha1=alpha2=0.5, phi=5 to reproduce Table 1 type I error rates at alpha=0.01/0.05/0.1.

## When NOT to use

- The goal is estimating statistical power or differential abundance effects — this skill simulates strictly under the null (mu1=mu2), so rejections are false positives, not true discoveries
- The data are real observed microbiome relative abundances rather than simulated BEZI data — use the package's taxa.compare/meta.taxa analysis workflow instead
- The abundance distribution of interest is not zero-inflated beta (e.g. count data) — BEZI simulation parameters would not be applicable

## Inputs

- Null-hypothesis simulation parameters: mu1=mu2=0.5, alpha1=alpha2=0.5, phi=5
- Sample sizes n=10, n=100, n=500
- Monte Carlo replicate count (1000)
- Nominal significance levels alpha = 0.01, 0.05, 0.1
- R environment with metamicrobiomeR, gamlss, gamlss.dist, devtools

## Outputs

- type1_error_rates.csv: table of empirical type I error rates (rows = sample sizes, columns = alpha levels)
- Per-setting rejection counts over 1000 Monte Carlo replicates
- Validation check of empirical rates against nominal levels within Monte Carlo tolerance

## How to apply

Following the simulation workflow in the metamicrobiomeR tutorial (Additional file 1), generate BEZI-distributed relative abundance data under the null with equal group parameters (mu1=mu2=0.5, alpha1=alpha2=0.5, phi=5) for each sample size n=10, 100, and 500, using the gamlss.dist BEZI random generation. For each sample size, repeat data generation and the GAMLSS-BEZI between-group comparison for 1000 Monte Carlo replicates, counting null-hypothesis rejections at alpha levels 0.01, 0.05, and 0.1. Convert rejection counts to empirical type I error rates, assemble them into a table with rows as sample sizes and columns as alpha levels, and validate that each empirical rate lies within Monte Carlo tolerance (~2 standard errors of a binomial proportion over 1000 replicates) of its nominal level. The rationale is that only a method that correctly models the zero-inflated beta distribution, as GAMLSS-BEZI does, will show well-controlled type I error on such data.

## Related tools

- **metamicrobiomeR** (R package providing the GAMLSS-BEZI framework and tutorial (Additional file 1) whose simulation workflow defines the null-data generation and between-group comparison) — https://github.com/nhanhocu/metamicrobiomeR
- **gamlss.dist** (Supplies the zero-inflated beta (BEZI) family used to generate null relative abundance data)
- **devtools** (Used to install the metamicrobiomeR package from GitHub via install_github)

## Examples

```
library(devtools); install_github("nhanhocu/metamicrobiomeR"); library(metamicrobiomeR); library(gamlss.dist); rBEZI(n, mu=0.5, sigma=0.5, nu=5)
```

## Evaluation signals

- Each empirical type I error rate lies within approximately 2 standard errors of the binomial proportion (over 1000 replicates) of its nominal level (0.01, 0.05, or 0.1)
- The assembled table reproduces the well-controlled type I error pattern reported in Table 1 for both GAMLSS-BEZI and LMAS
- type1_error_rates.csv has rows = sample sizes (10, 100, 500) and columns = alpha levels (0.01, 0.05, 0.1) with rates in plausible binomial ranges
- Exactly 1000 Monte Carlo replicates were run per sample-size setting, so rejection counts divided by 1000 match the reported rates

## Limitations

- The workflow depends on the package tutorial (Additional file 1), which is referenced but not inline-accessible in the article text
- The README notes the CRAN version has updates/minor changes relative to the initial GitHub version; users are recommended to use the CRAN version, which may alter simulation behavior
- Type I error control is demonstrated only for the BEZI simulation parameters studied (mu1=mu2=0.5, alpha1=alpha2=0.5, phi=5) and does not establish control under other distributional settings

## Evidence

- [full_text] Type I errors were well controlled in both GAMLSS-BEZI and LMAS in the simulation study (Table 1).: [evidence span withheld — anchor status: not_found]
- [full_text] generate BEZI null data per tutorial workflow: [evidence span withheld — anchor status: not_found]
- [methods] install from GitHub: "install_github("nhanhocu/metamicrobiomeR") library(metamicrobiomeR)"
- [discussion] GAMLSS-BEZI addresses the distribution of relative abundance data: "The advantages of GAMLSS-BEZI are: 1) it directly address the distribution of microbiome relative abundance data"
- [readme] README recommends CRAN version over GitHub version: "There are some updates/ minor changes in the R cran version as compared to this initial version of the package deposited in Github."

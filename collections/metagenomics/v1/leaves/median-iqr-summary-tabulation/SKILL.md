---
name: median-iqr-summary-tabulation
description: Use when you have per-iteration error rates (e.g. proportion of taxa
  with p < 0.05 under 1000 artificial case/control splits) and need a compact median
  (IQR) summary table per method and taxonomic level (phylum l2, family l5).
when_to_use_negative:
- You need per-taxa differential abundance effect estimates or FDR-adjusted p-values,
  not a global type I error summary (use taxcomtab.show / metatab.show instead).
- The per-iteration statistic distribution is strongly bimodal or you need full distributional
  detail rather than a robust two-number summary.
- Input is raw taxonomy relative abundance data (taxtab) rather than already-computed
  per-iteration error rates — tabulation is the final step, not the analysis itself.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: metamicrobiomeR
  role: taxa.compare with propmed.rel='gamlss' generates the per-taxon p-values from
    which per-iteration type I error rates are computed before median (IQR) tabulation
  repo: https://github.com/nhanhocu/metamicrobiomeR
- name: R
  role: Language in which the 1000-iteration loop, median()/quantile() summarization,
    and write.csv of the Table 2-formatted output are performed (R 3.4.2 or higher)
- name: knitr
  role: Used with kable() to render comparable result tables in the package tutorial
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-019-2744-2
    title: 'metamicrobiomeR: an R package for analysis of microbiome relative abundance
      data using zero-inflated beta GAMLSS and meta-analysis across studies using
      random effects models'
  source_content_hash: sha256:59a51ace731cf101281d144147427dc4b34b654cc91b755ec3c4d872787284b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nhanhocu/metamicrobiomeR
  tools:
  - MaAsLin
  - R
  - knitr
  - metamicrobiomeR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# median-iqr-summary-tabulation

## Summary

Summarizes per-iteration empirical type I error rates from repeated random case/control splits (e.g. 1000 iterations on the Subramanian 2014 Bangladesh cohort) as median (IQR) per method and taxonomic level, formatted like Table 2 of the metamicrobiomeR paper to assess control at nominal alpha levels of 0.01, 0.05 and 0.1.

## When to use

Use when you have per-iteration error rates (e.g. proportion of taxa with p < 0.05 under 1000 artificial case/control splits) and need a compact median (IQR) summary table per method and taxonomic level (phylum l2, family l5).

## When NOT to use

- You need per-taxa differential abundance effect estimates or FDR-adjusted p-values, not a global type I error summary (use taxcomtab.show / metatab.show instead).
- The per-iteration statistic distribution is strongly bimodal or you need full distributional detail rather than a robust two-number summary.
- Input is raw taxonomy relative abundance data (taxtab) rather than already-computed per-iteration error rates — tabulation is the final step, not the analysis itself.

## Inputs

- Per-iteration type I error rates (numeric vector, length 1000) for each method x taxonomic level x dataset combination
- Per-taxon p-values from taxa.compare (GAMLSS-BEZI) and LMAS linear/linear mixed model fits under each artificial split
- Nominal alpha levels for comparison: 0.01, 0.05, 0.1

## Outputs

- type1_error_table.csv: rows = method (GAMLSS-BEZI, LMAS) x taxonomic level (phylum, family), columns = median (IQR) per dataset (cross-sectional, longitudinal), formatted to match Table 2

## How to apply

For each of the 1000 iterations, compute the empirical type I error rate as the proportion of taxa with p-value < 0.05 under the null (artificial case/control) split, for each method (GAMLSS-BEZI via taxa.compare with propmed.rel='gamlss', and LMAS linear/linear mixed models) and taxonomic level (phylum 'l2', family 'l5'). Summarize the 1000 per-iteration rates as median with interquartile range (25th-75th percentiles) per method x dataset (50-sample cross-sectional, 996-sample longitudinal) x taxonomic level using R's median() and quantile(). Write the resulting table to type1_error_table.csv formatted to match Table 2 of the paper. Judge correctness by checking that reported medians fall close to the nominal alpha levels (0.01, 0.05, 0.1), as the paper reports for both GAMLSS-BEZI and LMAS on real microbiome data.

## Related tools

- **metamicrobiomeR** (taxa.compare with propmed.rel='gamlss' generates the per-taxon p-values from which per-iteration type I error rates are computed before median (IQR) tabulation) — https://github.com/nhanhocu/metamicrobiomeR
- **R** (Language in which the 1000-iteration loop, median()/quantile() summarization, and write.csv of the Table 2-formatted output are performed (R 3.4.2 or higher))
- **knitr** (Used with kable() to render comparable result tables in the package tutorial)

## Examples

```
t1 <- sapply(iters, function(it) mean(pvals[[it]] < 0.05)); c(median=median(t1), iqr=quantile(t1, c(.25,.75))); write.csv(type1_error_table, "type1_error_table.csv")
```

## Evaluation signals

- Reported median type I error values at phylum and family levels fall close to the nominal alpha levels 0.01, 0.05 and 0.1, reproducing the paper's Table 2 conclusion for both GAMLSS-BEZI and LMAS.
- The IQR is computed as the 25th-75th percentile of the 1000 per-iteration rates, and each summary cell aggregates exactly 1000 iteration values.
- Output CSV schema matches Table 2: method x taxonomic level (l2 phylum, l5 family) rows with median (IQR) per dataset (50-sample cross-sectional, 996-sample longitudinal).
- Longitudinal splits preserve repeated samples per infant (no summary computed on permuting within-individual structure).

## Limitations

- Median (IQR) is a robust two-number summary; it can mask tails or bimodality of the 1000-iteration error-rate distribution and does not itself provide a formal test of calibration.
- Type I error assessment depends on the artificial-split design: the 50-sample cross-sectional subset has limited taxa per level, and longitudinal splits must keep repeated samples per infant or the summary is invalid.
- Conclusions are specific to real 16S data of the Subramanian et al. 2014 Bangladesh cohort; medians close to nominal alpha on other datasets are not guaranteed.

## Evidence

- [results] type I error finding: [evidence span withheld — anchor status: not_found]
- [abstract] abstract performance claim: "Both simulation studies and application to real microbiome data demonstrate that GAMLSS-BEZI well performs in testing differential relative abundances of microbial taxonomies."
- [methods] per-taxon comparison call feeding the tabulation: "taxacom6.zi.rmg<-taxa.compare(taxtab=taxtab6.rm[[5]],propmed.rel="gamlss",comvar="bf",adjustvar="age.sample",longitudinal="yes",p.adjust.method="fdr")"
- [readme] readme tooling: "install_github("nhanhocu/metamicrobiomeR")
library(metamicrobiomeR)"

---
name: random-effects-meta-analysis-pvalue-computation
description: Use when you have study-specific regression coefficients and standard
  errors (effect estimates + SEs) for ≥10 studies across multiple related markers,
  and need p-values that avoid the over-conservatism of traditional random effects
  meta-analysis.
when_to_use_negative:
- You have fewer than 10 studies — mMeta.aMeta requires the number of rows (studies)
  to be greater than or equal to 10
- You only have raw sequencing data (e.g., FASTQ/OTU tables) rather than effect estimates
  and standard errors — the workflow demands regression coefficients and their standard
  errors per study per marker
- You need estimation of marker-by-marker effect sizes with high power rather than
  hypothesis testing — aMeta is limited to hypothesis testing
tools:
- name: mMeta
  role: R package providing the mMeta.aMeta function used to fit both the Han-Eskin
    modified (tau0=FALSE) and traditional (tau0=TRUE) random effects meta-analysis
    and return the p-values
  repo: https://github.com/hk1785/mMeta
- name: R
  role: Runtime environment (>= 4.0.3) in which the mMeta package and its dependencies
    (forestplot, MASS, vegan) are run
  repo: https://github.com/hk1785/mMeta
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/sim.8940
    title: 'Meta‐analysis methods for multiple related markers: Applications to microbiome
      studies with the results on multiple <i>α</i>‐diversity indices'
  source_content_hash: sha256:fefeb2931524128b7b5833398fd3789d37cac0f6bbd668d4a2d3eda41392e44e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hk1785/mMeta
  tools:
  - R
  - R package mMeta
  - mMeta
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# random-effects-meta-analysis-pvalue-computation

## Summary

Computes single-marker and cross-marker p-values for combining multiple studies across related markers (e.g., 6 α-diversity indices over 15 HIV studies) using mMeta and aMeta under the Han-Eskin modified random effects model (tau0=FALSE), which assumes no heterogeneity under H0, versus the traditional random effects null (tau0=TRUE).

## When to use

Use when you have study-specific regression coefficients and standard errors (effect estimates + SEs) for ≥10 studies across multiple related markers, and need p-values that avoid the over-conservatism of traditional random effects meta-analysis.

## When NOT to use

- You have fewer than 10 studies — mMeta.aMeta requires the number of rows (studies) to be greater than or equal to 10
- You only have raw sequencing data (e.g., FASTQ/OTU tables) rather than effect estimates and standard errors — the workflow demands regression coefficients and their standard errors per study per marker
- You need estimation of marker-by-marker effect sizes with high power rather than hypothesis testing — aMeta is limited to hypothesis testing

## Inputs

- Data.frame/matrix of study-specific regression coefficients (rows = studies, columns = markers; ≥10 studies; NAs matched to std.err)
- Data.frame/matrix of study-specific standard errors (same shape and NA locations as est)
- Parameters: tau0 (TRUE/FALSE), n.perm (default 5000), seed (e.g., 123)

## Outputs

- Pooled estimates and 95% confidence intervals per marker and across markers
- Single-marker p-values for each marker
- mMeta p-value across markers (multivariate, permutation-based)
- aMeta p-value across markers (adaptive)
- Comparison table showing Han-Eskin (tau0=FALSE) vs traditional (tau0=TRUE) p-values

## How to apply

Load the summary statistics (e.g., HIV.Sum.Data$est and HIV.Sum.Data$std.err for 15 HIV studies × 6 α-diversity indices: Richness Observed, Shannon, Simpson, PD, PE, PQE), keeping NA locations matched between est and std.err. Fit mMeta.aMeta twice: once with tau0=FALSE (Han and Eskin's modification, no heterogeneous variance under the null) and once with tau0=TRUE (traditional random effects where heterogeneity exists under H0), using the SJ heterogeneity estimator, n.perm=5000 group permutations, and seed=123 for reproducibility. Extract the single-marker p-values, the mMeta p-value across markers, and the aMeta p-value across markers from each fitted object. Compare the two p-value sets: the Han-Eskin modification should yield smaller p-values, confirming the traditional null assumption is overly conservative (as reported for Figure 2, Figure 3, and Table S2).

## Related tools

- **mMeta** (R package providing the mMeta.aMeta function used to fit both the Han-Eskin modified (tau0=FALSE) and traditional (tau0=TRUE) random effects meta-analysis and return the p-values) — https://github.com/hk1785/mMeta
- **R** (Runtime environment (>= 4.0.3) in which the mMeta package and its dependencies (forestplot, MASS, vegan) are run) — https://github.com/hk1785/mMeta

## Examples

```
out <- mMeta.aMeta(est, std.err, tau0=FALSE, n.perm=5000, seed=123)
```

## Evaluation signals

- Han-Eskin modified p-values (tau0=FALSE) should be smaller than or equal to the corresponding traditional random effects p-values (tau0=TRUE) across markers
- Re-running with the same seed=123 and n.perm=5000 reproduces identical p-values (deterministic permutations); with <15 studies all possible permutations are enumerated and seed is ignored
- Results match the reported comparison in Figure 2, Figure 3, and Table S2 of the paper
- NA locations in est and std.err are matched (schema check), and both inputs have ≥10 study rows
- Output object contains pooled estimates, 95% confidence intervals, single-marker p-values, an mMeta cross-marker p-value, and an aMeta cross-marker p-value

## Limitations

- Requires at least 10 studies; if the number of studies < 15, all possible permutations are implemented and n.perm/seed are ignored
- mMeta estimates marker-by-marker correlations non-parametrically via permutations, so its power is only moderate
- aMeta closely approaches the highest power among marker-specific meta-analyses but is limited to hypothesis testing
- Both methods require only effect estimates and their standard errors, but these summary statistics are hard to obtain in practice (e.g., many studies report only graphs or stratified summaries)

## Evidence

- [methods] HIV summary data loading step: [evidence span withheld — anchor status: not_found]

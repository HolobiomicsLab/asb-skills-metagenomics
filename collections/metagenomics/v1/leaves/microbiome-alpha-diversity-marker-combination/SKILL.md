---
name: microbiome-alpha-diversity-marker-combination
description: Use when you have effect estimates and standard errors per study and
  marker (≥10 studies), want a global conclusion on 'α-diversity' across multiple
  related markers, and have no prior marker-by-marker correlation estimates.
when_to_use_negative:
- You have fewer than 10 studies (the README requires the number of studies ≥ 10 for
  est and std.err)
- You need pooled effect estimation per marker only and already have a priori marker-by-marker
  correlation results — a standard multivariate meta-analysis approach applies instead
- Raw 16S rRNA read data or OTU tables are all you have — mMeta/aMeta require already-computed
  effect estimates and standard errors per study and marker (hard to obtain in practice
  when studies report only graphs or stratified summaries)
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: mMeta
  role: R package providing mMeta.aMeta (multi-marker and adaptive multi-marker meta-analysis
    with group permutation n.perm=5000 and Han-Eskin tau0 option) and mMeta.aMeta.plot
    (forest plot)
  repo: https://github.com/hk1785/mMeta
- name: R
  role: Runtime environment (>= 4.0.3) with devtools for GitHub installation and the
    forestplot, MASS, vegan dependencies
  repo: https://github.com/hk1785/mMeta
- name: devtools
  role: Used to install the mMeta package from GitHub via install_github('hk1785/mMeta',
    force=T)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/sim.8940
    title: 'Meta‐analysis methods for multiple related markers: Applications to microbiome
      studies with the results on multiple <i>α</i>‐diversity indices'
  source_content_hash: sha256:0d2c54ab711e57765fbcbc5ffb93b490aee3448ffd118c89e368ca1ae7ed40ed
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hk1785/mMeta
  tools:
  - R
  - R package mMeta
  - devtools
  - mMeta
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# microbiome-alpha-diversity-marker-combination

## Summary

Combine multiple microbiome studies across multiple related α-diversity markers (e.g., Richness, Shannon, Simpson, PD, PE, PQE) using only summary statistics (effect estimates and standard errors) with no a priori marker-by-marker correlations, via the mMeta and aMeta methods implemented in the R package mMeta.

## When to use

Use when you have effect estimates and standard errors per study and marker (≥10 studies), want a global conclusion on 'α-diversity' across multiple related markers, and have no prior marker-by-marker correlation estimates.

## When NOT to use

- You have fewer than 10 studies (the README requires the number of studies ≥ 10 for est and std.err)
- You need pooled effect estimation per marker only and already have a priori marker-by-marker correlation results — a standard multivariate meta-analysis approach applies instead
- Raw 16S rRNA read data or OTU tables are all you have — mMeta/aMeta require already-computed effect estimates and standard errors per study and marker (hard to obtain in practice when studies report only graphs or stratified summaries)

## Inputs

- data.frame or matrix of study-specific effect estimates (regression coefficients), rows = studies (≥10), columns = markers, NAs allowed
- data.frame or matrix of study-specific standard errors with NA locations matched to est
- Packaged datasets: Sum.Data (example) and HIV.Sum.Data (15 HIV studies × 6 α-diversity indices: Richness (Observed), Shannon, Simpson, PD, PE, PQE)

## Outputs

- Pooled estimates, 95% confidence intervals, and p-values for each marker and across markers using mMeta
- A p-value across markers using aMeta (adaptive minimum p-value statistic)
- Comparison table of p-values and 5%-level power under group permutation vs parametric asymptotic-normality p-value calculation (e.g., pvalue_method_comparison.csv)
- Forest plot per marker and across markers via mMeta.aMeta.plot

## How to apply

Install the mMeta package from GitHub with devtools (install_github('hk1785/mMeta', force=T)), which requires R (>= 4.0.3) plus forestplot, MASS, and vegan. Load a summary data set as a data.frame of study-specific estimates and matched standard errors (rows = studies, columns = markers; NAs allowed if locations match between est and std.err), e.g. the packaged Sum.Data example or HIV.Sum.Data covering 15 HIV studies and 6 α-diversity indices. Call mMeta.aMeta(est, std.err, tau0=FALSE, n.perm=5000, seed=123): mMeta uses a semi-parametric group permutation (Follmann-Proschan) method to estimate marker-by-marker correlations, and aMeta uses the same permutations to generate null statistic values of the minimum p-value statistic across markers; tau0=FALSE applies the Han-Eskin modification assuming no heterogeneity under H0. For the method comparison sub-task, run the same data under the group permutation p-value calculation and the parametric method based on asymptotic normality (mu_hat/sigma_mu ~ N(0,1)), optionally replicating the simulation design (K=10/20/30 studies; Dirichlet-multinomial counts from 100 respiratory-tract OTUs; scenarios S1 rare, S2 random, S3 common, S4 PAM) to estimate power at the 5% level. Tabulate p-values and power side by side and confirm that the group permutation method yields smaller p-values / higher power than the asymptotic-normality method, as reported in Figure 1 and Table S2. Optionally visualize per-marker and cross-marker pooled estimates, 95% CIs, and p-values with mMeta.aMeta.plot (forest plot).

## Related tools

- **mMeta** (R package providing mMeta.aMeta (multi-marker and adaptive multi-marker meta-analysis with group permutation n.perm=5000 and Han-Eskin tau0 option) and mMeta.aMeta.plot (forest plot)) — https://github.com/hk1785/mMeta
- **R** (Runtime environment (>= 4.0.3) with devtools for GitHub installation and the forestplot, MASS, vegan dependencies) — https://github.com/hk1785/mMeta
- **devtools** (Used to install the mMeta package from GitHub via install_github('hk1785/mMeta', force=T))

## Examples

```
library(mMeta); data(HIV.Sum.Data); out2 <- mMeta.aMeta(HIV.Sum.Data$est, HIV.Sum.Data$std.err, tau0=FALSE, n.perm=5000, seed=123); out2
```

## Evaluation signals

- mMeta.aMeta returns pooled estimates, 95% CIs, per-marker and cross-marker mMeta p-values, and an aMeta cross-marker p-value without erroring; the number of study rows in est and std.err is ≥ 10 and NA positions match between the two matrices
- With seed=123 and n.perm=5000 (or all possible permutations when studies < 15), repeated runs give identical p-values
- Group permutation p-values are smaller / power at the 5% level is higher than the parametric asymptotic-normality method, matching the ordering reported in Figure 1A,B < Figure 1C,D and Table S2
- Forest plots from mMeta.aMeta.plot show one panel per marker plus a cross-marker panel for all 6 α-diversity indices (Richness, Shannon, Simpson, PD, PE, PQE) in HIV.Sum.Data
- Simulations using Dirichlet-multinomial counts from 100 respiratory-tract OTUs reproduce the relative power ordering across scenarios S1 (rare), S2 (random), S3 (common), and S4 (PAM phylogenetic cluster)

## Limitations

- mMeta's power is only moderate because marker-by-marker correlations are estimated non-parametrically via permutations; aMeta closely approaches the highest power but is limited to hypothesis testing (no pooled estimate)
- Requires ≥ 10 studies; if the number of studies < 15, all possible permutations are implemented and n.perm and seed are ignored
- Effect estimates and standard errors are hard to obtain in practice — many studies report only graphs (box plots, bar/line charts with error bars) or stratified summaries (mean, median, IQR per group) rather than association estimates and p-values
- The six α-diversity indices are not always available from prior studies, and there are no set criteria for selecting candidate markers; delineate your selection rules explicitly
- No version number is stated for the mMeta R package and no changelog is available

## Evidence

---
name: missing-summary-data-handling
description: Use when the summary-statistics table has missing study-by-marker entries
  so the number of studies (Kj) varies by marker, and you need pooled estimates plus
  mMeta/aMeta p-values using all available data.
when_to_use_negative:
- The input is raw sequencing data (e.g., paired-end Illumina reads or OTU tables)
  rather than per-study, per-marker summary statistics (effect estimates and standard
  errors)
- Fewer than 10 studies are available, since mMeta.aMeta requires the number of rows
  (studies) to be >= 10
- NA positions differ between 'est' and 'std.err' (locations of NAs must be matched),
  or a single-marker conventional meta-analysis is the actual goal rather than combining
  multiple related markers
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: mMeta
  role: R package providing mMeta.aMeta and mMeta.aMeta.plot, the core functions that
    run the multi-marker and adaptive multi-marker meta-analyses on (possibly unbalanced)
    est/std.err tables
  repo: https://github.com/hk1785/mMeta
- name: R
  role: Runtime environment for loading the mMeta package, importing built-in summary
    datasets, and constructing the unbalanced est/std.err tables
  repo: https://github.com/hk1785/mMeta
- name: devtools
  role: Used to install the mMeta package from GitHub (install_github('hk1785/mMeta',
    force=T))
  repo: https://github.com/hk1785/mMeta
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/sim.8940
    title: 'Meta‐analysis methods for multiple related markers: Applications to microbiome
      studies with the results on multiple <i>α</i>‐diversity indices'
  source_content_hash: sha256:1e6bf963dfe337c228da381ab5b393d2ecf681fa8a0fcec1f4e4c0d54b296c4b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hk1785/mMeta
  tools:
  - R
  - devtools
  - mMeta
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# missing-summary-data-handling

## Summary

Implements the mMeta and aMeta multi-marker meta-analysis functions on unbalanced summary-statistic tables (study-by-marker regression coefficient estimates 'est' and standard errors 'std.err') in which selected entries are set to NA, exploiting the methods' design to robustly handle missing summary data across studies and markers.

## When to use

Use when the summary-statistics table has missing study-by-marker entries so the number of studies (Kj) varies by marker, and you need pooled estimates plus mMeta/aMeta p-values using all available data.

## When NOT to use

- The input is raw sequencing data (e.g., paired-end Illumina reads or OTU tables) rather than per-study, per-marker summary statistics (effect estimates and standard errors)
- Fewer than 10 studies are available, since mMeta.aMeta requires the number of rows (studies) to be >= 10
- NA positions differ between 'est' and 'std.err' (locations of NAs must be matched), or a single-marker conventional meta-analysis is the actual goal rather than combining multiple related markers

## Inputs

- data.frame or matrix of study-specific effect estimates (e.g., regression coefficients), rows = studies, columns = markers, NAs allowed at matched positions
- data.frame or matrix of study-specific standard errors with the same shape and matched NA locations as 'est'
- Built-in mMeta example data HIV.Sum.Data (15 HIV studies x 6 alpha-diversity indices) or Sum.Data

## Outputs

- Marker-specific pooled estimates with 95% confidence intervals and p-values
- Across-marker mMeta p-values
- Across-marker aMeta p-value
- Optionally, a forest plot per marker and across markers via mMeta.aMeta.plot, and a CSV of extracted results

## How to apply

Load the mMeta R package (installed via devtools::install_github('hk1785/mMeta')) together with forestplot, MASS, and vegan. Import a summary dataset such as the built-in HIV.Sum.Data, which holds regression coefficient estimates (est) and standard errors (std.err) for 15 HIV studies across 6 alpha-diversity indices (Richness Observed, Shannon, Simpson, PD, PE, PQE), or supply your own data.frame/matrices whose rows are studies (>= 10 rows required) and columns are markers. Construct an unbalanced version by setting selected study-by-marker entries to NA in both 'est' and 'std.err' at matched NA positions, mimicking studies that report only a subset of markers. Call mMeta.aMeta(est, std.err, seed=123), which internally applies the simple Kj modification so all available summary data are used; n.perm defaults to 5000 permutations, but if the number of studies < 15 all possible permutations are used and 'seed' is ignored. Extract marker-specific pooled estimates, 95% confidence intervals, and p-values, the across-marker mMeta p-values, and the aMeta across-marker p-value from the output object (optionally visualized with mMeta.aMeta.plot and written to CSV).

## Related tools

- **mMeta** (R package providing mMeta.aMeta and mMeta.aMeta.plot, the core functions that run the multi-marker and adaptive multi-marker meta-analyses on (possibly unbalanced) est/std.err tables) — https://github.com/hk1785/mMeta
- **R** (Runtime environment for loading the mMeta package, importing built-in summary datasets, and constructing the unbalanced est/std.err tables) — https://github.com/hk1785/mMeta
- **devtools** (Used to install the mMeta package from GitHub (install_github('hk1785/mMeta', force=T))) — https://github.com/hk1785/mMeta

## Examples

```
data(HIV.Sum.Data); out <- mMeta.aMeta(HIV.Sum.Data$est, HIV.Sum.Data$std.err, seed=123); out
```

## Evaluation signals

- mMeta.aMeta(est, std.err, seed=123) completes with no implementation error despite NA entries in the unbalanced tables
- The output object contains non-empty pooled estimates, 95% confidence interval bounds, and p-values for every marker
- An aMeta across-marker p-value is present in the output alongside the mMeta p-values
- NA positions in 'est' and 'std.err' match exactly (same row/column coordinates), as required by the function arguments
- Repeated runs with the same seed=123 return identical outcomes

## Limitations

- The number of studies (rows of est/std.err) must be >= 10; NA locations must be matched between 'est' and 'std.err'
- If the number of studies < 15, all possible permutations are implemented and 'n.perm' and 'seed' are ignored
- Effect estimates and their standard errors required by mMeta and aMeta are hard to obtain in practice; many studies report only graphs or stratified summary statistics rather than association estimates
- aMeta is limited to hypothesis testing (no pooled effect estimate), and mMeta's power is only moderate because marker-by-marker correlations are estimated non-parametrically via permutations
- No version number is stated for the mMeta R package and no changelog is available in the repository

## Evidence

- [methods] data(HIV.Sum.Data) # Summary data for 15 HIV studies and 6 alpha-diversity indices: [evidence span withheld — anchor status: not_found]

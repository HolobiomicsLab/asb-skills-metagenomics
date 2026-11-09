---
name: covariate-adjusted-longitudinal-modeling
description: Use when the input is a longitudinal master table (features x samples
  over repeated time points, e.g. genus-level fasting stool data with a Day or Time_point
  variable) and time effects must be separated from covariate effects in count, proportion,
  binary, ordinal or continuous data.
when_to_use_negative:
- The data are cross-sectional with no repeated measurements or time variable — longdat_cont()/longdat_disc()
  require longitudinal structure
- Only a metadata or feature table exists alone — the master table must first be assembled
  via make_master_table()
- 'Comparing against ANCOM: the FrederickHuangLin/ANCOM-Code-Archive functions are
  unmaintained, and LongDat outperformed ANCOM in accuracy, runtime and memory cost'
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: LongDat
  role: Core R package providing longdat_cont()/longdat_disc() pipelines, make_master_table()
    and cuneiform_plot() for covariate-sensitive longitudinal analysis
  repo: https://github.com/CCY-dev/LongDat
- name: R
  role: Runtime environment for LongDat; install with install.packages('LongDat')
  repo: https://github.com/CCY-dev/LongDat
- name: MaAsLin2
  role: Comparison benchmark; produced higher false positives than LongDat on shuffled-time
    semi-synthetic fasting data
  repo: https://github.com/biobakery/biobakery
- name: lgpr
  role: Comparison benchmark on identical shuffled datasets; showed elevated false
    positive rates vs LongDat
- name: ZIBR
  role: Comparison benchmark on identical shuffled datasets; showed elevated false
    positive rates vs LongDat
- name: ANCOM
  role: Comparison benchmark; available via the archived standalone R functions with
    feature_table_pre_process()
  repo: https://github.com/FrederickHuangLin/ANCOM-Code-Archive
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad063
    title: 'LongDat: an R package for covariate-sensitive longitudinal analysis of
      high-dimensional data'
  source_content_hash: sha256:a5d352b9bf85894bdf2fa2d20ca32a806d0ac5ac9abfc5c01ae86a099f01e03e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCY-dev/LongDat
  tools:
  - ANCOM
  - LongDat
  - MaAsLin2
  - R
  - ZIBR
  - lgpr
  - longdat_cont()
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# covariate-adjusted-longitudinal-modeling

## Summary

Analyze high-dimensional longitudinal data (e.g. microbiome count tables) for significant feature change over time while simultaneously detecting and controlling for covariates, using LongDat's longdat_cont() or longdat_disc() R pipelines, which outperform MaAsLin2, ANCOM, lgpr and ZIBR in accuracy, runtime and memory cost when covariates are numerous.

## When to use

Use when the input is a longitudinal master table (features x samples over repeated time points, e.g. genus-level fasting stool data with a Day or Time_point variable) and time effects must be separated from covariate effects in count, proportion, binary, ordinal or continuous data.

## When NOT to use

- The data are cross-sectional with no repeated measurements or time variable — longdat_cont()/longdat_disc() require longitudinal structure
- Only a metadata or feature table exists alone — the master table must first be assembled via make_master_table()
- Comparing against ANCOM: the FrederickHuangLin/ANCOM-Code-Archive functions are unmaintained, and LongDat outperformed ANCOM in accuracy, runtime and memory cost

## Inputs

- Longitudinal master table (LongDat_cont_master_table / LongDat_disc_master_table format) combining metadata and feature columns
- Metadata table (e.g. LongDat_cont_metadata_table) with sample IDs, time variable, and covariates
- Feature table (e.g. LongDat_cont_feature_table) with high-dimensional features such as genus-level microbiome counts
- data_type specification: count, proportion, binary, ordinal or continuous

## Outputs

- Result table with Signal, p values and EffectSize per feature
- Covariate table listing detected covariate effects (blank when none found)
- Randomized control table when data_type = 'count'
- Normalize_method table for measurement/other data types
- cuneiform_plot() visualization of the result table

## How to apply

Build a master table from metadata and feature tables with make_master_table(), then call longdat_cont() for a continuous time variable (e.g. test_var = 'Day') or longdat_disc() for a discrete variable (e.g. test_var = 'Time_point'), setting data_type = 'count' for count data. When data_type = 'count', set a seed beforehand so the randomized control test is reproducible; this control also guards against false positives, as validated on shuffled-time semi-synthetic fasting data where LongDat showed near-zero false positives while MaAsLin2, lgpr and ZIBR produced elevated rates (Supplementary Fig. S16). Inspect the result table's Signal column: 'OK_nc' means a significant time effect with no covariate, 'OK_nrc' means a significant effect not reducible to covariates, and 'NS' means no time effect. Check the covariate table for detected covariate effects (it is blank when none are found), use the EffectSize column to read directional trends (e.g. decreasing -0.78 for BacteriumA, increasing 0.71 for BacteriumC), and visualize with cuneiform_plot().

## Related tools

- **LongDat** (Core R package providing longdat_cont()/longdat_disc() pipelines, make_master_table() and cuneiform_plot() for covariate-sensitive longitudinal analysis) — https://github.com/CCY-dev/LongDat
- **R** (Runtime environment for LongDat; install with install.packages('LongDat')) — https://github.com/CCY-dev/LongDat
- **MaAsLin2** (Comparison benchmark; produced higher false positives than LongDat on shuffled-time semi-synthetic fasting data) — https://github.com/biobakery/biobakery
- **lgpr** (Comparison benchmark on identical shuffled datasets; showed elevated false positive rates vs LongDat)
- **ZIBR** (Comparison benchmark on identical shuffled datasets; showed elevated false positive rates vs LongDat)
- **ANCOM** (Comparison benchmark; available via the archived standalone R functions with feature_table_pre_process()) — https://github.com/FrederickHuangLin/ANCOM-Code-Archive

## Examples

```
test_cont <- longdat_cont(input = LongDat_cont_master_table, data_type = "count", test_var = "Day", ...
```

## Evaluation signals

- Feature with Signal 'OK_nc' or 'OK_nrc' has a significant time/treatment effect; 'NS' features show no time effect (e.g. BacteriumB)
- EffectSize signs match expected directionality (e.g. BacteriumA -0.78 decreasing, BacteriumC 0.71 increasing)
- Covariate table is blank when no covariate effect is detected; non-blank covariate table names the interfering covariates
- A randomized control table is returned when data_type = 'count', and setting a seed beforehand reproduces it
- On shuffled-time null data, false positive tally should be near zero for LongDat while measurably higher for MaAsLin2, lgpr and ZIBR (Supplementary Fig. S16 pattern)

## Limitations

- For count data (data_type = 'count'), LongDat performs a randomized control test that is stochastic — a seed must be set for reproducibility
- Comparison tools showed higher false positive rates on null (shuffled-time) data, so tool choice matters for FDR-sensitive biomarker searches
- ANCOM standalone code in FrederickHuangLin/ANCOM-Code-Archive is unmaintained; users are directed to ANCOMBC instead
- package namespace errors (e.g. object not exported by dependency B_package) may require installing the dependency first

## Evidence

- [abstract] abstract quote: "We introduce LongDat, an R package that analyzes longitudinal multivariable (cohort) data while simultaneously accounting for a potentially large number of covariates."
- [intro] intro seed quote: "When data_type equals to "count", please remember to set seed (as shown below) so that you'll get reproducible randomized control test."
- [results] results table quote: "If you have count data (data_type equals to "count"), then there are chances that you get a third table "randomized control table"."
- [readme] readme introduction quote: "LongDat R package takes longitudinal dataset as input data and analyzes if there is significant change of the features over time (proxy for treatments), while detects and controls for covariates"
- [full_text] task card finding quote: [evidence span withheld — anchor status: not_found]

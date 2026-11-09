---
name: count-data-randomized-control-testing
description: Use when running longdat_cont() or longdat_disc() with data_type = "count"
  on longitudinal abundance tables (e.g. species counts from a fasting MetS cohort)
  and reproducible, per-feature control testing is needed.
when_to_use_negative:
- Input data are proportion, binary, ordinal, or continuous rather than counts — the
  randomized control test and seed requirement apply when data_type equals "count"
- The goal is a cross-sectional differential abundance test rather than a longitudinal
  change-over-time analysis with covariate control
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: LongDat
  role: R package whose count-data pipelines (longdat_cont/longdat_disc) run the randomized
    control test; installed from CRAN or GitHub
  repo: https://github.com/CCY-dev/LongDat
- name: longdat_cont()
  role: Continuous-time longitudinal pipeline called with data_type = "count" and
    test_var such as 'Day'; triggers the seed-dependent randomized control test
  repo: https://github.com/CCY-dev/LongDat
- name: make_master_table()
  role: Builds the master table from metadata and feature tables that is the direct
    input to the count-data pipelines
  repo: https://github.com/CCY-dev/LongDat
- name: R
  role: Runtime environment in which set.seed() is called before count-data LongDat
    runs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad063
    title: 'LongDat: an R package for covariate-sensitive longitudinal analysis of
      high-dimensional data'
  source_content_hash: sha256:2398b10ca6d18eee4c7e4b1fb0634f47faf7e45f2ee410e8fbcceecacafcdaa8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCY-dev/LongDat
  tools:
  - LongDat
  - MaAsLin2
  - R
  - longdat_cont()
  - make_master_table()
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# count-data-randomized-control-testing

## Summary

When running LongDat's longdat_cont() or longdat_disc() on count-type data (e.g. species-level microbial abundance counts), the package performs a randomized control test that must be made reproducible by setting a seed beforehand, and can yield an extra randomized control table in the output.

## When to use

Use when running longdat_cont() or longdat_disc() with data_type = "count" on longitudinal abundance tables (e.g. species counts from a fasting MetS cohort) and reproducible, per-feature control testing is needed.

## When NOT to use

- Input data are proportion, binary, ordinal, or continuous rather than counts — the randomized control test and seed requirement apply when data_type equals "count"
- The goal is a cross-sectional differential abundance test rather than a longitudinal change-over-time analysis with covariate control

## Inputs

- LongDat master table built with make_master_table() from metadata and species-level count feature tables (data_type = "count")
- Sample metadata including time variable (e.g. Day or Time_point) and optional covariates (e.g. medication usage)
- A seed value set via set.seed() prior to running

## Outputs

- Result table with p values, effect sizes, and Signal column per feature (e.g. OK_nc, OK_nrc, NS)
- Covariate table listing detected covariate effects (blank if none)
- Randomized control table (produced when data_type = "count")

## How to apply

Before invoking longdat_cont() (or longdat_disc()) on the master table assembled with make_master_table(), set an R seed (e.g. set.seed()) whenever data_type equals "count", because the pipeline includes a randomized control test whose outcome depends on randomization. Run the pipeline with data_type = "count" and the chosen test_var (e.g. continuous 'Day' or discrete 'Time_point'); for count data the tool may return a third table, the randomized control table, alongside the result and covariate tables. Inspect the Signal column of the result table (e.g. 'OK_nc', 'OK_nrc', 'NS') and the covariate table to distinguish genuine time/treatment effects from covariate-reducible ones. In comparative reanalyses such as the Maifeld et al. 2021 fasting MetS cohort, this seed-controlled LongDat run remains robust with multiple medication covariates where MaAsLin2 models return errors, so report the covariate-disentangled significant species counts (e.g. 27 species per tool without covariates, 19 overlapping, MaAsLin2 dropping to 2 with covariates).

## Related tools

- **LongDat** (R package whose count-data pipelines (longdat_cont/longdat_disc) run the randomized control test; installed from CRAN or GitHub) — https://github.com/CCY-dev/LongDat
- **longdat_cont()** (Continuous-time longitudinal pipeline called with data_type = "count" and test_var such as 'Day'; triggers the seed-dependent randomized control test) — https://github.com/CCY-dev/LongDat
- **make_master_table()** (Builds the master table from metadata and feature tables that is the direct input to the count-data pipelines) — https://github.com/CCY-dev/LongDat
- **R** (Runtime environment in which set.seed() is called before count-data LongDat runs)

## Examples

```
set.seed(123); test_cont <- longdat_cont(input = master_table, data_type = "count", test_var = "Day")
```

## Evaluation signals

- Re-running the identical longdat_cont() call with the same seed reproduces the same result table and randomized control table (reproducibility invariant)
- Output contains a randomized control table in addition to the result and covariate tables when data_type = "count"
- Signal values in the result table are from the expected vocabulary (e.g. OK_nc, OK_nrc, NS) for every feature
- In the fasting MetS reanalysis, significant-species counts (27 per tool without covariates, 19 overlapping; MaAsLin2 only 2 with covariates) match Supplementary Tables S2–S5 and Fig. 5 expectations

## Limitations

- Omitting set.seed() when data_type = "count" makes the randomized control test non-reproducible
- Dependency installation errors (e.g. "object 'A' is not exported by 'namespace:B_package'") can block loading LongDat; the dependency must be installed first
- Count data cannot guarantee a randomized control table in every run — 'there are chances that you get a third table'

## Evidence

- [intro] When data_type equals to "count", please remember to set seed (as shown below) so that you'll get reproducible randomized control test.: "When data_type equals to "count", please remember to set seed (as shown below) so that you'll get reproducible randomized control test."
- [results] If you have count data (data_type equals to "count"), then there are chances that you get a third table "randomized control table".: "If you have count data (data_type equals to "count"), then there are chances that you get a third table "randomized control table"."
- [readme] LongDat R package takes longitudinal dataset as input data and analyzes if there is significant change of the features over time (proxy for treatments), while detects and controls for covariates at the same time.: "LongDat R package takes longitudinal dataset as input data and analyzes if there is significant change of the features over time (proxy for treatments), while detects and controls for covariates at"
- [discussion] LongDat handles multiple covariates robustly by looping over one covariate per model.: [evidence span withheld — anchor status: not_found]

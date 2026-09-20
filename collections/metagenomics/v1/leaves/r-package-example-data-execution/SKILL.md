---
name: r-package-example-data-execution
description: Use when verifying an R package installation with shipped example datasets,
  or reproducing the LongDat vignette workflow before applying the pipeline to new
  longitudinal count/proportion/binary/ordinal/continuous data.
when_to_use_negative:
- Your data is not longitudinal or lacks a repeated-measure/time variable — LongDat
  tests change of features over time (proxy for treatments).
- You only have separate metadata and feature tables and no master table — first build
  one with make_master_table(); the pipeline expects the master-table format.
- You need cross-sectional differential abundance without a time/covariate structure
  — comparison tools like ANCOM/MaAsLin2 address that design instead.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3174
tools:
- name: LongDat
  role: R package under test; provides longdat_cont(), longdat_disc(), make_master_table(),
    cuneiform_plot() and the lazily loaded example datasets
  repo: https://github.com/CCY-dev/LongDat
- name: R
  role: execution environment in which the package is installed, the seed is set,
    and the example pipeline is run
  repo: https://github.com/CCY-dev/LongDat
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad063
    title: 'LongDat: an R package for covariate-sensitive longitudinal analysis of
      high-dimensional data'
  source_content_hash: sha256:f3ee18511c3cd01101fa554c71fe5bdd46a6eca0478a26870e86cac0d7decca4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCY-dev/LongDat
  tools:
  - LongDat
  - R
  - longdat_cont()
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# r-package-example-data-execution

## Summary

Execute the LongDat R package's bundled example data (LongDat_cont_master_table / LongDat_disc_master_table) through longdat_cont() or longdat_disc() to reproduce the paper's three-stage longitudinal pipeline (null time model GLMM, per-covariate model loop, non-parametric effect-size step) and validate the documented outputs.

## When to use

Use when verifying an R package installation with shipped example datasets, or reproducing the LongDat vignette workflow before applying the pipeline to new longitudinal count/proportion/binary/ordinal/continuous data.

## When NOT to use

- Your data is not longitudinal or lacks a repeated-measure/time variable — LongDat tests change of features over time (proxy for treatments).
- You only have separate metadata and feature tables and no master table — first build one with make_master_table(); the pipeline expects the master-table format.
- You need cross-sectional differential abundance without a time/covariate structure — comparison tools like ANCOM/MaAsLin2 address that design instead.

## Inputs

- LongDat_cont_master_table (lazily loaded dummy longitudinal dataset, 2 time points: day 0 and 7)
- LongDat_disc_master_table (lazily loaded example for discrete time variables)
- data_type string (e.g. 'count'), test_var column name (e.g. 'Day' or 'Time_point')
- random seed for reproducibility when data_type = 'count'

## Outputs

- result table with Signal and EffectSize columns (longdat_cont_result_table.csv)
- covariate table (longdat_cont_covariate_table.csv; blank when no covariate effect is found)
- randomized control table (when data_type = 'count') or Normalize_method table (for measurement/others data)
- cuneiform_plot() visualization of the result table

## How to apply

Install LongDat from CRAN (install.packages('LongDat')) and use the lazily loaded example dataset LongDat_cont_master_table (a dummy longitudinal dataset with 2 time points, day 0 and 7). Because data_type equals 'count', set a random seed first so the randomized control test is reproducible. Call longdat_cont(input = LongDat_cont_master_table, data_type = 'count', test_var = 'Day') so the GLMM null time model, the covariate loop (each covariate added alone with the time variable, classified as reducible/entangled/not-reducible) and the Spearman's rho effect-size step run in fixed order. Inspect the Signal column of the result table ('OK_nc' marks a significant time effect with no covariate, 'NS' no effect) and check the covariate table; in the worked example it is blank because no covariate effect was found, with BacteriumA and BacteriumC showing 'OK_nc' and effect sizes -0.78 and 0.71. Optionally visualize with cuneiform_plot().

## Related tools

- **LongDat** (R package under test; provides longdat_cont(), longdat_disc(), make_master_table(), cuneiform_plot() and the lazily loaded example datasets) — https://github.com/CCY-dev/LongDat
- **R** (execution environment in which the package is installed, the seed is set, and the example pipeline is run) — https://github.com/CCY-dev/LongDat

## Examples

```
set.seed(1); test_cont <- longdat_cont(input = LongDat_cont_master_table, data_type = "count", test_var = "Day")
```

## Evaluation signals

- Result table Signal column matches the documented pattern: BacteriumA and BacteriumC show 'OK_nc', BacteriumB shows 'NS'.
- EffectSize column shows -0.78 for BacteriumA (decreasing over time) and 0.71 for BacteriumC (increasing over time).
- Covariate table is blank in the worked example because no covariate effect was found.
- With data_type = 'count', re-running with the same seed reproduces the same randomized control table (reproducibility check).
- Output CSVs are written as longdat_cont_result_table.csv and longdat_cont_covariate_table.csv with expected schema (p values, effect sizes, covariates per feature).

## Limitations

- When data_type equals 'count', a seed must be set or the randomized control test is not reproducible.
- Installation may fail with namespace export errors (e.g. object 'A' is not exported by 'namespace:B_package'); the dependency must be installed first.
- The covariate model test loops over covariates one at a time — a significant time effect may be classified as reducible to a single covariate rather than removed jointly.

## Evidence

- [intro] Set seed before running when data_type equals 'count' for reproducible randomized control test: "When data_type equals to "count", please remember to set seed (as shown below) so that you'll get reproducible randomized control test."
- [results] BacteriumA and BacteriumC show OK_nc with effect sizes -0.78 and 0.71: "BacteriumA and BacteriumC have decreasing and increasing patterns, respectively. From the next column "EffectSize", we know that the effect sizes are -0.78 and 0.71"
- [results] Covariate table is blank when no covariate effect is found: "Since there is no covariate effect found in this example (according to the result table), the covariate table is blank."
- [intro] Example call of longdat_cont on the master table: "test_cont <- longdat_cont(input = LongDat_cont_master_table,
data_type = "count",
test_var = "Day","
- [readme] README: install and data types supported: "LongDat is able to take in several data types as input, including count, proportion, binary, ordinal and continuous data."

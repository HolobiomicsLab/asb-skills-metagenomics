---
name: negative-control-microbiome-count-simulation
description: 'Use when validating a longitudinal microbiome method''s type-I error
  control: you need simulated null (zero-effect) count data whose library sizes differ
  systematically across time points to expose normalization-induced false positives.'
when_to_use_negative:
- Data contain a real treatment/time effect — this is a negative-control benchmark
  with effect size zero across time points
- Feature values are already relative-abundance or continuous measurements rather
  than raw counts — LongDat's randomized control test and the rarefaction step apply
  to data_type='count'
- The goal is differential-abundance power benchmarking rather than type-I error (false
  positive rate) assessment
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: LongDat
  role: Runs the count-data longitudinal pipeline (longdat_cont()/longdat_disc(),
    negative-binomial mode) on rarefied negative-control tables to measure false positives
  repo: https://github.com/CCY-dev/LongDat
- name: longdat_cont()
  role: Longitudinal analysis pipeline function applied with data_type='count' to
    the rarefied tables
  repo: https://github.com/CCY-dev/LongDat
- name: R
  role: Implementation environment for simulation, rarefaction, TSS normalization,
    linear-model comparator, and FPR computation
- name: SparseDOSSA2
  role: Generates negative-control simulated count tables with zero effect size and
    systematic sequencing-depth differences across two time points
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad063
    title: 'LongDat: an R package for covariate-sensitive longitudinal analysis of
      high-dimensional data'
  source_content_hash: sha256:963a2133901c7b596ace9324cb35d9b6f6aedfc1f79daffb19a6bf779bbf2489
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCY-dev/LongDat
  tools:
  - LongDat
  - R
  - SparseDOSSA2
  - longdat_cont()
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# negative-control-microbiome-count-simulation

## Summary

Generate simulated negative-control longitudinal microbiome count tables (zero effect size) with systematic sequencing-depth variation via SparseDOSSA2, then compare false positive rates of LongDat's rarefied negative-binomial pipeline against a non-rarefied TSS-normalized linear-model comparator, as in the LongDat Supplementary Fig. S9 benchmark.

## When to use

Use when validating a longitudinal microbiome method's type-I error control: you need simulated null (zero-effect) count data whose library sizes differ systematically across time points to expose normalization-induced false positives.

## When NOT to use

- Data contain a real treatment/time effect — this is a negative-control benchmark with effect size zero across time points
- Feature values are already relative-abundance or continuous measurements rather than raw counts — LongDat's randomized control test and the rarefaction step apply to data_type='count'
- The goal is differential-abundance power benchmarking rather than type-I error (false positive rate) assessment

## Inputs

- Simulated longitudinal microbiome count tables (features x samples, two time points, zero effect size) generated with SparseDOSSA2
- Six dataset versions differing systematically in designated sequencing depth
- Metadata/master table formatted per LongDat (as in LongDat_disc_master_table) mapping samples to time points

## Outputs

- Rarefied count tables with common library size per version
- LongDat result tables (p values, effect sizes, Signal column) from the negative-binomial count pipeline
- Linear-model p values on TSS-normalized non-rarefied tables
- Per-setting false positive rates (fraction of null features significant at nominal alpha)
- Supplementary Fig. S9-style FPR comparison figure

## How to apply

Simulate negative-control longitudinal count tables with SparseDOSSA2 using zero effect size across two time points, producing six dataset versions that differ systematically in sequencing depth. Rarefy each simulated table to its designated depth so all samples within a version share a common library size. Run LongDat's count-data longitudinal pipeline (longdat_cont() or longdat_disc() with data_type='count', negative-binomial mode) on the rarefied tables, setting a seed first because the randomized control test for count data requires one for reproducibility. Fit linear models on TSS-transformed versions of the non-rarefied tables as the compositional-normalization comparator. Compute the false positive rate per setting as the fraction of truly-null features called significant at the nominal alpha threshold, then tabulate the FPR contrast (rarefied NB vs non-rarefied linear/TSS) and render a Supplementary Fig. S9-style comparison figure.

## Related tools

- **LongDat** (Runs the count-data longitudinal pipeline (longdat_cont()/longdat_disc(), negative-binomial mode) on rarefied negative-control tables to measure false positives) — https://github.com/CCY-dev/LongDat
- **longdat_cont()** (Longitudinal analysis pipeline function applied with data_type='count' to the rarefied tables) — https://github.com/CCY-dev/LongDat
- **R** (Implementation environment for simulation, rarefaction, TSS normalization, linear-model comparator, and FPR computation)
- **SparseDOSSA2** (Generates negative-control simulated count tables with zero effect size and systematic sequencing-depth differences across two time points)

## Examples

```
set.seed(123); test_cont <- longdat_cont(input = LongDat_cont_master_table, data_type = "count", test_var = "Time_point")
```

## Evaluation signals

- Rarefied negative-binomial false positive rate is near zero, at or below the nominal alpha level, across all six depth versions
- Linear/TSS false positive rate is markedly elevated under large sequencing-depth variation, matching Supplementary Fig. S9
- Every truly-null feature has effect size zero, so any significant call at nominal alpha counts as a false positive
- All samples within a rarefied dataset version share a common library size (verify post-rarefaction column sums)
- LongDat count-mode results are reproducible across reruns because the seed was set before running

## Limitations

- Findings are qualitative and specific to the simulated six-depth negative-control design; they do not establish absolute FPR values for other designs
- The linear/TSS comparator inflation is a property of non-rarefied compositional normalization under systematic depth differences; results depend on the magnitude of depth variation
- LongDat's randomized control test for count data is stochastic unless a seed is set, so unseeded runs may not be reproducible

## Evidence

- [intro] Count data require seed for reproducible randomized control test: "When data_type equals to "count", please remember to set seed (as shown below) so that you'll get reproducible randomized control test."
- [abstract] LongDat analyzes longitudinal multivariable data accounting for covariates: "We introduce LongDat, an R package that analyzes longitudinal multivariable (cohort) data while simultaneously accounting for a potentially large number of covariates."
- [readme] LongDat installs from CRAN/GitHub: [evidence span withheld — anchor status: not_found]
- [intro] longdat_cont count pipeline invocation with test_var: "test_cont <- longdat_cont(input = LongDat_cont_master_table,
data_type = "count",
test_var = "Day","

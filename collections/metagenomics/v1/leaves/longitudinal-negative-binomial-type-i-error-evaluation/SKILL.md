---
name: longitudinal-negative-binomial-type-i-error-evaluation
description: Use when validating a longitudinal microbiome pipeline's false positive
  rate on negative-control data where sequencing depth varies systematically across
  time points but all features are truly null (zero effect size).
when_to_use_negative:
- Data contain genuine non-zero time effects — this skill calibrates Type I error
  on all-null (zero effect size) negative controls only.
- Input is already compositional (proportion/TSS-normalized) rather than raw counts
  — the negative-binomial count mode and rarefaction step assume absolute count tables.
- The analysis is cross-sectional or otherwise lacks time points/replicates needed
  for longitudinal modeling in longdat_cont()/longdat_disc().
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3050
tools:
- name: LongDat
  role: Runs the longitudinal count-data pipeline (longdat_cont()/longdat_disc(),
    negative-binomial mode) whose false positive rate is being calibrated on rarefied
    negative-control tables
  repo: https://github.com/CCY-dev/LongDat
- name: longdat_cont()
  role: Longitudinal analysis pipeline function applied to the rarefied count master
    tables with data_type='count' and a fixed seed
  repo: https://github.com/CCY-dev/LongDat
- name: SparseDOSSA2
  role: Generates the negative-control longitudinal count tables with zero effect
    size and six systematic sequencing-depth versions
- name: R
  role: Language in which rarefaction, TSS normalization, linear-model comparator
    fitting, FPR computation, and plotting are implemented
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad063
    title: 'LongDat: an R package for covariate-sensitive longitudinal analysis of
      high-dimensional data'
  source_content_hash: sha256:10dd8846c993a0cba8c43cba6e1076b57dd71056372a7aac0304154e43b4aaba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCY-dev/LongDat
  tools:
  - LongDat
  - R
  - SparseDOSSA2
  - longdat_cont()
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3050
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# longitudinal-negative-binomial-type-i-error-evaluation

## Summary

A negative-control calibration skill that quantifies false positive rate (Type I error) of longitudinal microbiome pipelines under systematic sequencing-depth differences, contrasting LongDat's negative-binomial mode on rarefied counts against linear models on TSS-normalized non-rarefied data.

## When to use

Use when validating a longitudinal microbiome pipeline's false positive rate on negative-control data where sequencing depth varies systematically across time points but all features are truly null (zero effect size).

## When NOT to use

- Data contain genuine non-zero time effects — this skill calibrates Type I error on all-null (zero effect size) negative controls only.
- Input is already compositional (proportion/TSS-normalized) rather than raw counts — the negative-binomial count mode and rarefaction step assume absolute count tables.
- The analysis is cross-sectional or otherwise lacks time points/replicates needed for longitudinal modeling in longdat_cont()/longdat_disc().

## Inputs

- SparseDOSSA2-simulated negative-control longitudinal count tables (two time points, zero effect size, six systematic sequencing-depth versions)
- Metadata/master table compatible with LongDat (same format as LongDat_cont_master_table or LongDat_disc_master_table)
- Nominal significance threshold alpha (e.g. 0.05)

## Outputs

- Per-feature result tables from longdat_cont()/longdat_disc() (p values, effect sizes, Signal column) plus randomized control table for count data
- False positive rate per setting (fraction of truly-null features called significant across all six depth versions)
- FPR contrast table and Supplementary Fig. S9-style comparison figure (rarefied negative binomial vs linear/TSS)

## How to apply

Simulate negative-control longitudinal count tables with SparseDOSSA2 using zero effect size across two time points, generating six dataset versions that differ systematically in sequencing depth. Rarefy each simulated table to its designated depth so all samples within a version share a common library size, then run LongDat's count-data pipeline (longdat_cont() or longdat_disc(), data_type='count') with a seed set for reproducibility. As a comparator, fit linear models on TSS-transformed (non-rarefied) versions of the same tables. Compute FPR per setting as the fraction of truly-null features called significant at the nominal alpha threshold across all six depth versions, and tabulate the rarefied negative-binomial vs linear/TSS contrast in a Supplementary Fig. S9-style figure. Accept the result when the rarefied negative-binomial FPR is at or below the nominal alpha (near zero) while the linear/TSS FPR is markedly elevated under large depth variation.

## Related tools

- **LongDat** (Runs the longitudinal count-data pipeline (longdat_cont()/longdat_disc(), negative-binomial mode) whose false positive rate is being calibrated on rarefied negative-control tables) — https://github.com/CCY-dev/LongDat
- **longdat_cont()** (Longitudinal analysis pipeline function applied to the rarefied count master tables with data_type='count' and a fixed seed) — https://github.com/CCY-dev/LongDat
- **SparseDOSSA2** (Generates the negative-control longitudinal count tables with zero effect size and six systematic sequencing-depth versions)
- **R** (Language in which rarefaction, TSS normalization, linear-model comparator fitting, FPR computation, and plotting are implemented)

## Examples

```
install.packages("LongDat"); set.seed(1); res <- longdat_cont(input = master_table, data_type = "count", test_var = "Time_point")
```

## Evaluation signals

- Rarefied negative-binomial FPR is near zero — at or below the nominal alpha level across all six depth versions
- Linear/TSS FPR is markedly elevated under large sequencing-depth variation, matching the qualitative pattern of Supplementary Fig. S9
- FPR is computed only from truly-null features (zero effect size) called significant at the nominal alpha threshold
- Randomized control / count-mode results are reproducible because a seed was set before running when data_type equals 'count'
- Each rarefied table has a common library size shared by all samples within a version

## Limitations

- Type I error calibration is specific to the simulation settings (two time points, the six chosen sequencing-depth versions, SparseDOSSA2 count-generating model); conclusions may not transfer to other designs or depth distributions.
- The negative-binomial count mode requires setting a seed for reproducible randomized control tests; unseeded runs will not reproduce the FPR estimates.
- The elevated FPR of the linear/TSS comparator is expected under large sequencing-depth variation, so this skill should not be used to claim the linear model is generally invalid outside depth-confounded settings.

## Evidence

- [intro] seed for reproducible count-mode runs: "When data_type equals to "count", please remember to set seed (as shown below) so that you'll get reproducible randomized control test."
- [abstract] LongDat package scope: "We introduce LongDat, an R package that analyzes longitudinal multivariable (cohort) data while simultaneously accounting for a potentially large number of covariates."
- [readme] LongDat input and purpose from README: "LongDat R package takes longitudinal dataset as input data and analyzes if there is significant change of the features over time (proxy for treatments), while detects and controls for covariates at"
- [results] count-mode pipeline output structure: "If you have count data (data_type equals to "count"), then there are chances that you get a third table "randomized control table"."

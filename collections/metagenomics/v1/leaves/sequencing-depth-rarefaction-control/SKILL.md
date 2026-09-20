---
name: sequencing-depth-rarefaction-control
description: 'Use when validating a longitudinal differential-abundance method against
  sequencing-depth bias: you need negative-control data with no true time effect but
  varying library sizes, or must quantify FPR inflation from compositional normalization.'
when_to_use_negative:
- Data contain genuine time/treatment effects — this skill is a negative-control design
  assuming all features are truly null (zero effect size), so FPR is the only meaningful
  readout
- Sequencing depth is approximately uniform across time points, so depth-driven confounding
  is not the validation target
- Analysis targets absolute-abundance or already-rarefied data where compositional
  normalization bias (TSS on non-rarefied counts) does not apply
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: LongDat
  role: Negative-binomial-mode longitudinal pipeline (longdat_cont()/longdat_disc(),
    data_type='count') run on rarefied negative-control count tables
  repo: https://github.com/CCY-dev/LongDat
- name: longdat_cont()
  role: Longitudinal analysis function applied to count data with a seed set so the
    randomized control test is reproducible
  repo: https://github.com/CCY-dev/LongDat
- name: SparseDOSSA2
  role: Simulates negative-control longitudinal count tables with zero effect size
    and systematically varying sequencing depth
- name: R
  role: Environment for simulation, rarefaction, TSS transformation, linear-model
    comparator fitting, and FPR tabulation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad063
    title: 'LongDat: an R package for covariate-sensitive longitudinal analysis of
      high-dimensional data'
  source_content_hash: sha256:ad847c66e8bcac21be3ac92a5f32e941782317ce935d1cd2113d5b0bf36b136f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCY-dev/LongDat
  tools:
  - LongDat
  - R
  - SparseDOSSA2
  - longdat_cont()
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# sequencing-depth-rarefaction-control

## Summary

A negative-control validation skill that simulates longitudinal microbiome count tables with zero true effect but systematically varying sequencing depth (SparseDOSSA2), then contrasts rarefaction + LongDat's negative-binomial count pipeline versus TSS normalization + linear models to expose false positives driven by depth differences.

## When to use

Use when validating a longitudinal differential-abundance method against sequencing-depth bias: you need negative-control data with no true time effect but varying library sizes, or must quantify FPR inflation from compositional normalization.

## When NOT to use

- Data contain genuine time/treatment effects — this skill is a negative-control design assuming all features are truly null (zero effect size), so FPR is the only meaningful readout
- Sequencing depth is approximately uniform across time points, so depth-driven confounding is not the validation target
- Analysis targets absolute-abundance or already-rarefied data where compositional normalization bias (TSS on non-rarefied counts) does not apply

## Inputs

- SparseDOSSA2-simulated negative-control longitudinal count tables (zero effect size, two time points, six versions varying in sequencing depth)
- Rarefied count tables with common library size per version
- Non-rarefied count tables for TSS transformation
- Nominal alpha threshold (e.g. 0.05)

## Outputs

- False positive rate per setting (rarefied negative-binomial vs non-rarefied linear/TSS) across the six depth versions
- FPR contrast table
- Supplementary Fig. S9-style comparison figure of FPR vs sequencing-depth setting

## How to apply

Simulate negative-control longitudinal count tables with SparseDOSSA2 using zero effect size across two time points, generating six dataset versions that differ systematically in sequencing depth. Rarefy each simulated count table so all samples within a version share a common library size, then run LongDat's count-data longitudinal pipeline (longdat_cont() or longdat_disc() with data_type='count' and a set seed for the reproducible randomized control test) on the rarefied tables. As the comparator, fit linear models on TSS-transformed versions of the non-rarefied count tables. Compute the false positive rate per setting as the fraction of truly-null features called significant at the nominal alpha threshold across all six depth versions, and tabulate the contrast between the rarefied negative-binomial and non-rarefied linear/TSS settings (e.g. as a Supplementary Fig. S9-style figure). The expected pattern is a near-zero FPR for rarefied negative-binomial (at or below nominal alpha) and a markedly elevated FPR for linear/TSS under large depth variation.

## Related tools

- **LongDat** (Negative-binomial-mode longitudinal pipeline (longdat_cont()/longdat_disc(), data_type='count') run on rarefied negative-control count tables) — https://github.com/CCY-dev/LongDat
- **longdat_cont()** (Longitudinal analysis function applied to count data with a seed set so the randomized control test is reproducible) — https://github.com/CCY-dev/LongDat
- **SparseDOSSA2** (Simulates negative-control longitudinal count tables with zero effect size and systematically varying sequencing depth)
- **R** (Environment for simulation, rarefaction, TSS transformation, linear-model comparator fitting, and FPR tabulation)

## Examples

```
set.seed(1); res <- longdat_cont(input = rarefied_master_table, data_type = "count", test_var = "Time_point")
```

## Evaluation signals

- Rarefied negative-binomial FPR is near zero, at or below the nominal alpha level
- Non-rarefied linear/TSS FPR is markedly elevated under large sequencing-depth variation
- FPR is computed as the fraction of truly-null features called significant at the nominal alpha threshold, aggregated across all six depth versions
- Rarefied tables show identical library sizes within each simulated version
- Randomized control test of LongDat on count data is reproducible because a seed was set before running

## Limitations

- Rarefaction discards reads, so it trades power for unbiasedness; the finding applies to negative-control data and may not reflect power on effect-bearing data
- The linear/TSS comparator's high FPR is expected only when sequencing-depth differences are large and systematic between time points
- The count-data mode of LongDat includes a randomized control test that is stochastic unless a seed is set, so results are not reproducible without it

## Evidence

- [supplementary] Negative-control simulation with SparseDOSSA2 at zero effect size, rarefied NB pipeline vs TSS linear comparator, FPR contrast (task_004 workflow): [evidence span withheld — anchor status: not_found]
- [intro] LongDat count mode requires seed setting for reproducible randomized control test: "When data_type equals to "count", please remember to set seed (as shown below) so that you'll get reproducible randomized control test."
- [intro] LongDat count pipeline invocation via longdat_cont() on a master table: "test_cont <- longdat_cont(input = LongDat_cont_master_table,
data_type = "count",
test_var = "Day","
- [results] LongDat count mode produces a randomized control table supporting FPR-based validation: "If you have count data (data_type equals to "count"), then there are chances that you get a third table "randomized control table"."

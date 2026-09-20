---
name: fallback-feature-selection-logic
description: Use when implementing a distance-correlation-based feature screening
  step for microbiome multivariate association testing and you need a guaranteed non-empty
  selected feature subset S from an OTU matrix and adjusted response.
when_to_use_negative:
- You already have a predefined signal set or taxa-set (e.g. taxa-sets with at least
  five taxa at genus/family/order/class/phylum level) and do not need data-driven
  screening
- You are running a competing method such as aMiSPU, OMiRKAT, OMiAT, LDM or MiHC that
  tests the full clade without feature selection
- Your outcome-feature relationship is purely linear and well captured by marginal
  score statistics, where distance-correlation screening adds little
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: AMAT
  role: R implementation providing the distance correlation learning routine used
    to screen features and select the testing subset S, including the max-DC fallback
    rule
  repo: https://github.com/kzb193/AMAT
- name: R
  role: Computing environment in which the AMAT screening and fallback selection scripts
    are run
  repo: https://github.com/kzb193/AMAT
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqab120
    title: Adaptive and powerful microbiome multivariate association analysis via
      feature selection
  source_content_hash: sha256:baeb738fff300bb7360ba740bbd9ede126e6324d1e25f4232070c0eb3e434a53
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/kzb193/AMAT
  repo_url: https://github.com/kzb193/AMAT
  tools:
  - AMAT
  - R
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# fallback-feature-selection-logic

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implements AMAT's data-adaptive screening guarantee: features are selected by comparing observed distance correlations with permuted null means, and if no feature passes, the single feature with maximum distance correlation to the adjusted response is force-selected so the testing subset S is never empty.

## When to use

Use when implementing a distance-correlation-based feature screening step for microbiome multivariate association testing and you need a guaranteed non-empty selected feature subset S from an OTU matrix and adjusted response.

## When NOT to use

- You already have a predefined signal set or taxa-set (e.g. taxa-sets with at least five taxa at genus/family/order/class/phylum level) and do not need data-driven screening
- You are running a competing method such as aMiSPU, OMiRKAT, OMiAT, LDM or MiHC that tests the full clade without feature selection
- Your outcome-feature relationship is purely linear and well captured by marginal score statistics, where distance-correlation screening adds little

## Inputs

- n x p OTU abundance/count matrix (e.g. throat microbiome OTU table with 856 OTUs from 60 samples)
- adjusted response vector (continuous or binary outcome)
- number of permutations B for the null distance-correlation distribution (article uses 10,000 permutations for testing)

## Outputs

- Selected index set S, a non-empty subset of feature indices {1,...,p}
- Results file recording the selected feature indices
- Per-feature observed distance correlations and permuted null means (intermediate artifacts)

## How to apply

Given an n x p OTU count matrix (counts can be simulated from a multinomial distribution mimicking real throat microbiome composition) and an adjusted response vector, compute the distance correlation dcj between each feature j and the adjusted response. Permute the response B times and recompute distance correlations to build a per-feature null distribution; select features whose observed dcj exceeds the mean of their B permuted values. Apply the fallback decision point: if no column of Z passes the permuted-mean threshold, select the single feature with maximum observed distance correlation as the sole member of S. This mirrors the AMAT strategy of alleviating noise accumulation via distance correlation learning before running the adaptive association test. Write the selected index set S (subset of {1,...,p}) to a results file for downstream testing under the generalized linear model framework.

## Related tools

- **AMAT** (R implementation providing the distance correlation learning routine used to screen features and select the testing subset S, including the max-DC fallback rule) — https://github.com/kzb193/AMAT
- **R** (Computing environment in which the AMAT screening and fallback selection scripts are run) — https://github.com/kzb193/AMAT

## Examples

```
devtools::install_github("kzb193/AMAT"); library(AMAT)  # then run distance correlation screening with fallback on OTU matrix and adjusted response
```

## Evaluation signals

- The script runs end-to-end without error on the simulated OTU matrix and adjusted response
- The output S is a valid, non-empty subset of feature indices {1,...,p}
- The fallback rule fires exactly when the permuted-mean threshold rule selects zero features
- Selected features' observed distance correlations exceed the mean of their B permuted distance correlations (except fallback case)
- Multinomial simulation with the Dirichlet-multinomial generative model reproduces realistic throat microbiome count structure (856 OTUs, 60 samples scale)

## Limitations

- If no feature passes the permuted-mean threshold, the fallback selects a single maximum-DC feature, which may be a noise feature and reduce downstream power
- Screening quality depends on the permutation budget B; too few permutations give unstable null means
- Naive aggregation of low-rank taxa can suffer power loss when effects have opposite directions that cancel out, which is why DC-based screening rather than simple aggregation is used

## Evidence

- [other] if none of the columns of Z are selected in step 4, the feature having the maximum distance correlation with the adjusted response is selected as the testing-subset member.: [evidence span withheld — anchor status: not_found]

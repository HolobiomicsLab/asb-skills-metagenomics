---
name: empirical-type-i-error-estimation
description: Use when comparing the statistical power and type I error of multivariate
  microbiome association tests on simulated OTU count data (e.g. Dirichlet-multinomial
  simulations with known null and alternative scenarios) and you need empirical error
  rates at alpha=0.05.
when_to_use_negative:
- Real observational data with unknown true association status — empirical power and
  type I error cannot be computed without simulated nulls and known alternatives
- Scenarios where you cannot control or run the permutation scheme (e.g. only pre-computed
  P-values are available), since calibration must be re-estimated at the same alpha
  level
- Single-method analysis with no competing tests — the skill is a comparative benchmark,
  not a standalone calibration check
edam_operation: http://edamontology.org/operation_3233
edam_topics:
- http://edamontology.org/topic_3365
tools:
- name: AMAT
  role: Proposed adaptive multivariate association test whose power and type I error
    are being benchmarked; run on each simulation scenario with default settings
  repo: https://github.com/kzb193/AMAT
- name: aMiSPU
  role: Competing adaptive microbiome-based sum of powered score test run on identical
    simulation scenarios
- name: OMiRKAT
  role: Competing optimal microbiome regression-based kernel association test run
    on identical scenarios
- name: OMiAT
  role: Competing optimal microbiome-based association test run on identical scenarios
- name: LDM
  role: Competing linear decomposition of models test run on identical scenarios
- name: MiHC
  role: Competing minimum P-value combination test (Simes + higher criticism variants)
    run on identical scenarios
- name: R
  role: Computing environment in which all simulations, tests and permutation-based
    error rate estimation are performed
  repo: https://github.com/kzb193/AMAT
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqab120
    title: Adaptive and powerful microbiome multivariate association analysis via
      feature selection
  source_content_hash: sha256:f2c123500c5b9f17c58fb4d0200fde86dfab7eb0eaadcfc37b54dd79993d2a77
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
  - LDM
  - MiHC
  - OMiAT
  - OMiRKAT
  - R
  - aMiSPU
  edam_operation: http://edamontology.org/operation_3233
  edam_topics:
  - http://edamontology.org/topic_3365
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# Empirical type I error estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarking procedure that estimates empirical type I error (under simulated nulls) and power (under alternatives) at the 0.05 significance level for microbiome association tests, using 10,000 permutations per test so that methods like AMAT, aMiSPU, OMiRKAT, OMiAT, LDM and MiHC can be compared fairly.

## When to use

Use when comparing the statistical power and type I error of multivariate microbiome association tests on simulated OTU count data (e.g. Dirichlet-multinomial simulations with known null and alternative scenarios) and you need empirical error rates at alpha=0.05.

## When NOT to use

- Real observational data with unknown true association status — empirical power and type I error cannot be computed without simulated nulls and known alternatives
- Scenarios where you cannot control or run the permutation scheme (e.g. only pre-computed P-values are available), since calibration must be re-estimated at the same alpha level
- Single-method analysis with no competing tests — the skill is a comparative benchmark, not a standalone calibration check

## Inputs

- Simulated OTU count matrix under null scenarios (Dirichlet-multinomial generated, e.g. 856 OTUs mimicking the throat microbiome)
- Simulated OTU count matrix under alternative scenarios with varying signal densities (10%, 20%, 30%)
- Outcome vector (continuous and binary) and covariates for each scenario
- R packages for each competing test: AMAT, aMiSPU, OMiAT, MiHC, LDM, OMiRKAT

## Outputs

- Empirical power estimates per method per scenario at the 0.05 level
- Empirical type I error estimates per method per null configuration at the 0.05 level
- A single CSV table of six-method-per-scenario power and type I error estimates

## How to apply

Generate or load shared simulated OTU count datasets (in this paper, Dirichlet-multinomial simulations mimicking the real throat microbiome with 856 OTUs, under continuous and binary outcome scenarios), including explicit null configurations and alternatives varying signal density (e.g. 10%, 20%, 30%). Run each candidate test (AMAT via kzb193/AMAT, plus aMiSPU, OMiAT, MiHC, LDM, OMiRKAT in their default package settings) on identical scenarios, computing P-values for all six tests with 10,000 permutations. Estimate empirical power as the rejection proportion under alternatives and empirical type I error as the rejection proportion under nulls, both at the 0.05 significance level. Assemble the per-scenario, per-method estimates into a single table (CSV). Judge correctness by checking that estimated type I errors fall near the nominal 0.05 level — as the paper reports, all tests were well calibrated except MiHC, which was conservative, especially with binary outcomes — and that AMAT's power gains are consistent with the reported scenario structure (most powerful except under extremely sparse signals).

## Related tools

- **AMAT** (Proposed adaptive multivariate association test whose power and type I error are being benchmarked; run on each simulation scenario with default settings) — https://github.com/kzb193/AMAT
- **aMiSPU** (Competing adaptive microbiome-based sum of powered score test run on identical simulation scenarios)
- **OMiRKAT** (Competing optimal microbiome regression-based kernel association test run on identical scenarios)
- **OMiAT** (Competing optimal microbiome-based association test run on identical scenarios)
- **LDM** (Competing linear decomposition of models test run on identical scenarios)
- **MiHC** (Competing minimum P-value combination test (Simes + higher criticism variants) run on identical scenarios)
- **R** (Computing environment in which all simulations, tests and permutation-based error rate estimation are performed) — https://github.com/kzb193/AMAT

## Examples

```
install.packages('remotes'); remotes::install_github('kzb193/AMAT')
```

## Evaluation signals

- Estimated type I error rates under all null configurations fall near the nominal 0.05 level (paper reports MiHC as the exception, tending conservative, especially with binary outcomes)
- All six methods use the identical simulated datasets and the same 10,000 permutations per test, so differences in error rates are attributable to the tests, not the data
- Power ranking expectations: AMAT tended to be the most powerful except when association signal density was extremely sparse
- Under Scenario I (n=100, random OTU signal-set), AMAT powers should be reproducible at ~44.7%, 49.8%, 52.5% for signal densities 10%, 20%, 30%
- Output is a single CSV with per-scenario, per-method power and type I error columns

## Limitations

- MiHC tends to be conservative, especially with binary outcomes, so its nominally low type I error does not imply good calibration
- AMAT's power advantage vanishes when the association signal density is extremely sparse
- Empirical rates depend on the simulation model (Dirichlet-multinomial mimicking the throat microbiome); results may not transfer to other community structures or sample sizes

## Evidence

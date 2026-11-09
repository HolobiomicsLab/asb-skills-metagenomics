---
name: statistical-power-comparison-across-methods
description: Use when you must show whether a new statistical test is more than existing
  multivariate microbiome association tests across controlled simulation scenarios
  at a fixed significance level.
when_to_use_negative:
- The goal is testing association on one real dataset rather than benchmarking relative
  power of competing methods across scenarios
- The outcome is neither continuous nor binary in the simulated generalized linear
  model framework, or no null scenarios are available to check type I error
- Only one method is of interest, so no cross-method power comparison can be assembled
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: AMAT
  role: Proposed data-adaptive multivariate microbiome association test under evaluation;
    R implementation from kzb193/AMAT with default settings
  repo: https://github.com/kzb193/AMAT
- name: aMiSPU
  role: Competing adaptive microbiome-based sum of powered score test run on identical
    scenarios
- name: OMiRKAT
  role: Competing optimal microbiome regression-based kernel association test run
    on identical scenarios
- name: OMiAT
  role: Competing optimal microbiome-based association test run on identical scenarios
- name: LDM
  role: Competing linear decomposition model test run on identical scenarios
- name: MiHC
  role: Competing minimum P-value combination test (Simes plus higher criticism variants)
    run on identical scenarios
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqab120
    title: Adaptive and powerful microbiome multivariate association analysis via
      feature selection
  source_content_hash: sha256:a082a812c8546dfa1079a9c3c47d2610f7ab806271b1912e9e679086e6fbbe9b
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
  edam_operation: http://edamontology.org/operation_3798
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

# statistical-power-comparison-across-methods

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark empirical power and type I error of competing association tests (here AMAT vs aMiSPU, OMiRKAT, OMiAT, LDM, MiHC) on identical simulated OTU count datasets, estimating power at the 0.05 level from 10,000 permutations per test per scenario.

## When to use

Use when you must show whether a new statistical test is more powerful than existing multivariate microbiome association tests across controlled simulation scenarios at a fixed significance level.

## When NOT to use

- The goal is testing association on one real dataset rather than benchmarking relative power of competing methods across scenarios
- The outcome is neither continuous nor binary in the simulated generalized linear model framework, or no null scenarios are available to check type I error
- Only one method is of interest, so no cross-method power comparison can be assembled

## Inputs

- Simulated OTU count datasets (Dirichlet-multinomial, mimicking the 856-OTU throat microbiome) for continuous and binary outcome scenarios
- Null and alternative simulation configurations varying signal density (10%, 20%, 30%)
- R implementations of AMAT, aMiSPU, OMiAT, MiHC, LDM, and OMiRKAT

## Outputs

- Empirical power estimates per method per scenario at the 0.05 significance level
- Type I error estimates per method under null configurations
- A single CSV table of six-method-per-scenario power and type I error estimates

## How to apply

Generate or load shared simulated OTU count datasets (Dirichlet-multinomial simulations mimicking the real throat microbiome) spanning continuous and binary outcome scenarios and multiple signal densities (e.g. 10%, 20%, 30%). Run the proposed method (AMAT from kzb193/AMAT) and each competitor (aMiSPU, OMiAT, MiHC, LDM, OMiRKAT) with default package settings on the identical scenarios. Compute P-values for all six tests using 10,000 permutations, then estimate empirical power under alternatives and type I error under nulls at the 0.05 significance level. Assemble per-scenario power and type I error estimates for all methods into a single CSV table so the comparison is reproducible and directly comparable. Judge success by well-controlled type I error for most methods and a clear power ordering across signal densities, noting any exceptions such as extremely sparse signals.

## Related tools

- **AMAT** (Proposed data-adaptive multivariate microbiome association test under evaluation; R implementation from kzb193/AMAT with default settings) — https://github.com/kzb193/AMAT
- **aMiSPU** (Competing adaptive microbiome-based sum of powered score test run on identical scenarios)
- **OMiRKAT** (Competing optimal microbiome regression-based kernel association test run on identical scenarios)
- **OMiAT** (Competing optimal microbiome-based association test run on identical scenarios)
- **LDM** (Competing linear decomposition model test run on identical scenarios)
- **MiHC** (Competing minimum P-value combination test (Simes plus higher criticism variants) run on identical scenarios)

## Examples

```
Rscript run_AMAT.R  # install AMAT from https://github.com/kzb193/AMAT, run on each simulated OTU scenario with 10,000 permutations
```

## Evaluation signals

- Type I error rates across null configurations should be near the nominal 0.05 level (MiHC may be conservative, especially for binary outcomes)
- Empirical power is estimated from 10,000 permutations per test, so per-scenario estimates should be multiples of 1/10,000 scaled to percentages
- Under Scenario I (n=100, random OTU signal-set), AMAT powers of 44.7%/49.8%/52.5% at 10%/20%/30% signal densities should exceed runner-up powers of 33.6%/35.4%/38.3%
- All six methods are run on the identical simulated scenarios with default settings, ensuring a one-to-one per-scenario comparison in the assembled CSV table
- Power advantage of AMAT should hold across scenarios except when the association signal density is extremely sparse

## Limitations

- AMAT's power advantage may not hold when the association signal density is extremely sparse
- MiHC tends to be conservative in type I error, especially with binary outcomes, which can depress its apparent power
- Power estimates depend on 10,000 permutations, limiting resolution of very small P-values

## Evidence

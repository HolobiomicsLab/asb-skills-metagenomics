---
name: microbiome-simulation-benchmarking
description: Use when comparing power/type I error of multivariate microbiome association
  tests on simulated OTU count data with continuous or binary outcomes, e.g. Dirichlet-multinomial
  scenarios varying signal density.
when_to_use_negative:
- The goal is real-data taxon-level association testing with FWER control via Bonferroni
  correction, not simulation-based power benchmarking
- Only one method is of interest — benchmarking requires running all six competing
  tests on identical scenarios to make power comparisons meaningful
- The association signal density is extremely sparse, where AMAT is known to lose
  its power advantage
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Computing environment in which AMAT and all competing tests are implemented
    and the benchmark is run
  repo: https://github.com/kzb193/AMAT
- name: AMAT
  role: Proposed data-adaptive test under evaluation; run with default settings from
    the kzb193/AMAT repository
  repo: https://github.com/kzb193/AMAT
- name: aMiSPU
  role: Competing adaptive sum of powered score test run with default settings on
    identical simulated scenarios
- name: OMiRKAT
  role: Competing optimal microbiome regression-based kernel association test run
    with default settings
- name: OMiAT
  role: Competing optimal microbiome-based association test run with default settings
- name: LDM
  role: Competing linear decomposition model test run with default settings
- name: MiHC
  role: Competing minimum P-value combination test (Simes + higher criticism variants)
    run with default settings
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqab120
    title: Adaptive and powerful microbiome multivariate association analysis via
      feature selection
  source_content_hash: sha256:672a646ed53d16be93c4825307d8326ff58f0915a9b809f654d08ade62e2602e
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
yes_rate: 0.9166666666666666
n_claims: 12
---

# microbiome-simulation-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark AMAT against existing multivariate microbiome association tests (aMiSPU, OMiRKAT, OMiAT, LDM, MiHC) using Dirichlet-multinomial OTU simulations mimicking the real throat microbiome, estimating empirical power and type I error at the 0.05 level.

## When to use

Use when comparing power/type I error of multivariate microbiome association tests on simulated OTU count data with continuous or binary outcomes, e.g. Dirichlet-multinomial scenarios varying signal density.

## When NOT to use

- The goal is real-data taxon-level association testing with FWER control via Bonferroni correction, not simulation-based power benchmarking
- Only one method is of interest — benchmarking requires running all six competing tests on identical scenarios to make power comparisons meaningful
- The association signal density is extremely sparse, where AMAT is known to lose its power advantage

## Inputs

- Simulated OTU count tables generated via Dirichlet-multinomial/multinomial sampling mimicking the throat microbiome (856 OTUs, 60 samples)
- Simulated continuous and binary outcome vectors per scenario
- Scenario configuration varying association signal density (e.g. 10%, 20%, 30%)
- R packages: AMAT, aMiSPU, OMiAT, MiHC, LDM, OMiRKAT

## Outputs

- Per-scenario empirical power estimates at the 0.05 significance level for AMAT, aMiSPU, OMiRKAT, OMiAT, LDM and MiHC
- Per-scenario type I error estimates at the 0.05 level for all six methods
- A single CSV table of six-method power and type I error estimates per scenario

## How to apply

Load shared simulated OTU count datasets generated from a multinomial model mimicking the 856-OTU, 60-sample throat microbiome, spanning continuous and binary outcome scenarios. Run AMAT from the kzb193/AMAT R repository on each scenario with default settings, then run aMiSPU, OMiAT, MiHC, LDM and OMiRKAT on the identical scenarios with their packages' default settings. Compute P-values for all six tests using 10,000 permutations per test. Estimate empirical power (under alternatives) and type I error (under nulls) at the 0.05 significance level per method per scenario, then assemble a single six-method-per-scenario CSV table. Judge correctness by confirming all methods preserve nominal type I error (~0.05) and that AMAT's power advantage reproduces the reported gaps, e.g. 44.7% vs 33.6% for the runner-up at 10% signal density under Scenario I (n=100, random OTU signal-set).

## Related tools

- **R** (Computing environment in which AMAT and all competing tests are implemented and the benchmark is run) — https://github.com/kzb193/AMAT
- **AMAT** (Proposed data-adaptive test under evaluation; run with default settings from the kzb193/AMAT repository) — https://github.com/kzb193/AMAT
- **aMiSPU** (Competing adaptive sum of powered score test run with default settings on identical simulated scenarios)
- **OMiRKAT** (Competing optimal microbiome regression-based kernel association test run with default settings)
- **OMiAT** (Competing optimal microbiome-based association test run with default settings)
- **LDM** (Competing linear decomposition model test run with default settings)
- **MiHC** (Competing minimum P-value combination test (Simes + higher criticism variants) run with default settings)

## Examples

```
Rscript -e 'AMAT(Y, X, covs); aMiSPU(Y, X, covs); LDM(Y ~ covs + X); OMiRKAT(Y, X, Ks); OMiAT(Y, X); MiHC(Y, X)'  # 10,000 permutations, alpha=0.05
```

## Evaluation signals

- All six methods show type I error rates close to the nominal 0.05 across null scenarios; MiHC may be conservative, especially with binary outcomes
- AMAT's empirical power matches reported values, e.g. 44.7%, 49.8% and 52.5% at 10%, 20% and 30% signal densities under Scenario I (n=100)
- Runner-up methods' powers fall clearly below AMAT, e.g. 33.6%, 35.4% and 38.3% in the same scenarios
- Permutation counts are exactly 10,000 per test, ensuring comparable P-value resolution across methods
- The output CSV contains power and type I error for all six methods for every simulation scenario with no missing cells

## Limitations

- AMAT's power advantage does not hold when the association signal density is extremely sparse
- MiHC tends to be conservative (under-sized), particularly for binary outcomes, so its power comparisons should be interpreted with that in mind
- Empirical power/type I error estimates are subject to Monte Carlo error from 10,000 permutations per test
- Simulations mimic specific real microbiomes (throat, 856 OTUs; gut, 557 OTUs), so results may not generalize to other community structures

## Evidence

- [results] AMAT tended to be the most powerful method compared with existing multivariate microbiome association tests, except when the association signal density was extremely sparse.: [evidence span withheld — anchor status: not_found]

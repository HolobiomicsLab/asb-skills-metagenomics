---
name: empirical-power-analysis
description: Use when validating that a new microbiome multivariate association test
  (AMAT, aMiSPU, OMiRKAT, OMiAT, LDM, MiHC) controls type I error at the nominal 0.05
  level and to compare empirical power across signal densities and effect sizes.
when_to_use_negative:
- The analysis goal is a single hypothesis test on one real dataset rather than characterizing
  test performance over simulated replicates
- The input is a real OTU table without a defined null/alternative generative model
  — type I error cannot be estimated without simulated null data
- You need multiplicity-adjusted testing across taxonomic ranks (FWER control via
  Bonferroni) rather than single-set power estimation
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: AMAT
  role: Test being evaluated; applied to each simulated replicate via distance-correlation-based
    feature selection and data-adaptive association testing
  repo: https://github.com/kzb193/AMAT
- name: R
  role: Environment in which the simulations, AMAT implementation, and empirical error/power
    calculations are performed
  repo: https://github.com/kzb193/AMAT
- name: MiHC
  role: Competing method benchmarked alongside AMAT in the type I error and power
    simulations
- name: aMiSPU
  role: Competing method benchmarked alongside AMAT in the type I error and power
    simulations
- name: OMiRKAT
  role: Competing method benchmarked alongside AMAT in the type I error and power
    simulations
- name: OMiAT
  role: Competing method benchmarked alongside AMAT in the type I error and power
    simulations
- name: LDM
  role: Competing method benchmarked alongside AMAT in the type I error and power
    simulations
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqab120
    title: Adaptive and powerful microbiome multivariate association analysis via
      feature selection
  source_content_hash: sha256:dce5913d19af49124ae7570ea74f1d6c516972f888f5c330076c19813ffae487
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
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# empirical-power-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simulate null and alternative microbiome association datasets to estimate empirical type I error and statistical power of a multivariate test (e.g., AMAT) by measuring rejection proportions at a nominal level across replicate simulated OTU count tables.

## When to use

Use when validating that a new microbiome multivariate association test (AMAT, aMiSPU, OMiRKAT, OMiAT, LDM, MiHC) controls type I error at the nominal 0.05 level and to compare empirical power across signal densities and effect sizes.

## When NOT to use

- The analysis goal is a single hypothesis test on one real dataset rather than characterizing test performance over simulated replicates
- The input is a real OTU table without a defined null/alternative generative model — type I error cannot be estimated without simulated null data
- You need multiplicity-adjusted testing across taxonomic ranks (FWER control via Bonferroni) rather than single-set power estimation

## Inputs

- Simulated OTU count tables mimicking the real throat microbiome (856 OTUs, 60 samples) drawn from a multinomial distribution with observed relative abundances
- Continuous (linear model, Equation 1) and binary (logistic model, Equation 2) outcome vectors generated with β = 0 (null) and varying signal density (10%, 20%, 30%) and effect size
- Real throat microbiome OTU table (856 OTUs, 60 samples) used as the basis for simulation parameters

## Outputs

- Empirical type I error rates per scenario at the nominal 0.05 level
- Empirical power estimates per scenario (outcome type × signal density × effect size)
- Aggregated CSV table (e.g., simulation_results.csv) with rows per scenario and columns for outcome type, scenario parameters, empirical type I error, and empirical power

## How to apply

Simulate OTU count tables mimicking the real throat microbiome (856 OTUs, 60 samples) by drawing counts from a multinomial distribution with the observed relative abundances, under both a linear model (Equation 1, continuous outcome) and a logistic model (Equation 2, binary outcome). Generate replicate datasets under the null (β = 0) and alternatives varying signal density (10%, 20%, 30%) and effect size. Apply AMAT to each replicate using distance-correlation-based feature selection followed by the data-adaptive test with B = 500 permutations and tuning-parameter candidates Γ = {2, 3, 4, 8} in R. Compute empirical type I error as the proportion of null replicates rejected at the nominal 0.05 level and empirical power as the proportion of alternative replicates rejected, then aggregate into a per-scenario CSV (simulation_results.csv) with columns for outcome type, scenario parameters, type I error, and power.

## Related tools

- **AMAT** (Test being evaluated; applied to each simulated replicate via distance-correlation-based feature selection and data-adaptive association testing) — https://github.com/kzb193/AMAT
- **R** (Environment in which the simulations, AMAT implementation, and empirical error/power calculations are performed) — https://github.com/kzb193/AMAT
- **MiHC** (Competing method benchmarked alongside AMAT in the type I error and power simulations)
- **aMiSPU** (Competing method benchmarked alongside AMAT in the type I error and power simulations)
- **OMiRKAT** (Competing method benchmarked alongside AMAT in the type I error and power simulations)
- **OMiAT** (Competing method benchmarked alongside AMAT in the type I error and power simulations)
- **LDM** (Competing method benchmarked alongside AMAT in the type I error and power simulations)

## Examples

```
devtools::install_github("kzb193/AMAT"); library(AMAT); # per replicate: apply AMAT with B = 500 permutations and gamma = c(2, 3, 4, 8)
```

## Evaluation signals

- Each empirical type I error rate falls within the binomial 95% confidence interval around the nominal 0.05 level
- All reported power and type I error values lie in [0, 1]
- Null-replicate rejection proportion is close to 0.05 (as observed: all tests well controlled except MiHC, which is conservative especially with binary outcomes)
- Power increases monotonically with signal density (AMAT: 44.7%, 49.8%, 52.5% at 10%, 20%, 30% signal density in Scenario I)
- CSV output contains one row per scenario with columns for outcome type, scenario parameters, empirical type I error, and empirical power

## Limitations

- MiHC tends to be conservative, especially with binary outcomes, so pooled comparisons of 'well-controlled' tests should treat MiHC's type I error separately
- Power results depend on the simulation basis (throat microbiome, 856 OTUs); extending to other communities requires re-estimating the multinomial abundance parameters
- Permutation cost: B = 500 permutations per test per replicate limits the number of feasible replicates and scenarios

## Evidence

- [abstract] Extensive simulation studies and real data applications demonstrate that AMAT is highly robust and often more powerful than several existing methods, while preserving the correct type I error rate.: "Extensive simulation studies and real data applications demonstrate that AMAT is highly robust and often more powerful than several existing methods, while preserving the correct type I error rate."
- [results] Multinomial OTU generation: "the observed OTU counts were randomly generated from a multinomial distribution: (Zi1, Zi2, …, Zi856) ∼ Multinomial(ni; p1i, p2i, …, p856i)."

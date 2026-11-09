---
name: microbiome-otu-data-simulation
description: Use when you must benchmark type I error calibration or statistical power
  of a microbiome multivariate association test against synthetic OTU count data generated
  to mimic a real community.
when_to_use_negative:
- The goal is testing association on the real OTU data itself (run AMAT with 10,000
  permutations on the actual table instead of simulating).
- The input is a real, already-filtered OTU table needing only QC and taxonomic classification
  (this skill generates synthetic data, not downstream annotations).
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: AMAT
  role: The association test whose empirical type I error and power are being evaluated
    on the simulated OTU tables; uses distance-correlation-based feature selection
    and a data-adaptive test under the generalized linear model framework
  repo: https://github.com/kzb193/AMAT
- name: R
  role: Computing environment in which the multinomial OTU simulations and the AMAT
    test calls are implemented
  repo: https://github.com/kzb193/AMAT
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqab120
    title: Adaptive and powerful microbiome multivariate association analysis via
      feature selection
  source_content_hash: sha256:78f07785152332e6efb1e90c2c4e5e4e8ef9cb7d8b67d207ee2f20d538b809cd
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# microbiome-otu-data-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A simulation workflow that generates synthetic OTU count tables mimicking a real microbiome (e.g., the 856-OTU throat community) via multinomial/Dirichlet-multinomial sampling, then evaluates empirical type I error (null, beta = 0) and power of microbiome association tests such as AMAT under continuous (linear) and binary (logistic) outcome models.

## When to use

Use when you must benchmark type I error calibration or statistical power of a microbiome multivariate association test against synthetic OTU count data generated to mimic a real community.

## When NOT to use

- The goal is testing association on the real OTU data itself (run AMAT with 10,000 permutations on the actual table instead of simulating).
- The input is a real, already-filtered OTU table needing only QC and taxonomic classification (this skill generates synthetic data, not downstream annotations).

## Inputs

- Real OTU count table with observed relative abundances (e.g., throat microbiome: 856 OTUs, 60 samples)
- Outcome model specification (linear for continuous, logistic for binary outcomes)
- Scenario parameters: beta (0 for null; varying effect sizes for alternatives) and signal density (10%, 20%, 30%)
- Test settings: number of permutations (B = 500 for the simulation; 10,000 used for real-data tests) and tuning parameter candidates Gamma = {2,3,4,8}

## Outputs

- Simulated OTU count matrices per replicate and scenario
- Empirical type I error per outcome type and scenario (null replicates)
- Empirical power per signal density and effect size (alternative replicates)
- Aggregated results table, e.g., simulation_results.csv

## How to apply

Start from a quality-controlled OTU table (e.g., the throat microbiome with 856 OTUs from 60 samples, 28 smokers) and estimate observed relative abundances to serve as the multinomial probabilities. Draw replicate OTU count vectors from Multinomial(n_i; p_i1, ..., p_i856) under the linear model for continuous outcomes and the logistic model for binary outcomes, generating null replicates (beta = 0) and alternatives varying signal density (10%, 20%, 30%) and effect size. Apply the test under evaluation (AMAT with distance-correlation-based feature selection, B = 500 permutations, tuning candidates Gamma = {2,3,4,8}) to each replicate. Compute empirical type I error as the proportion of null replicates rejected at the nominal 0.05 level, and empirical power as the proportion of alternative replicates rejected, then aggregate per scenario into a results table (e.g., simulation_results.csv) with rows per scenario and columns for outcome type, scenario parameters, type I error, and power.

## Related tools

- **AMAT** (The association test whose empirical type I error and power are being evaluated on the simulated OTU tables; uses distance-correlation-based feature selection and a data-adaptive test under the generalized linear model framework) — https://github.com/kzb193/AMAT
- **R** (Computing environment in which the multinomial OTU simulations and the AMAT test calls are implemented) — https://github.com/kzb193/AMAT

## Examples

```
devtools::install_github("kzb193/AMAT")  # then run the R simulation script to write simulation_results.csv
```

## Evaluation signals

- Each empirical type I error falls within the binomial 95% interval around the nominal 0.05 level.
- All reported power and type I error values lie in [0, 1].
- Null-configured simulations (beta = 0) for both continuous and binary outcomes reproduce well-controlled type I error for AMAT, with MiHC expected to be conservative.
- Power monotonically increases with signal density (10% -> 20% -> 30%) for a fixed effect size.
- Result table schema matches: one row per scenario with columns for outcome type, scenario parameters, empirical type I error, and empirical power.

## Limitations

- Empirical type I error is only meaningful with enough null replicates; small replicate counts make the binomial 95% interval around 0.05 wide.
- Simulations are anchored to specific communities (throat: 856 OTUs/60 samples; gut: 557 OTUs/98 volunteers), so results may not transfer to communities with very different sparsity or dimensionality.
- MiHC's conservatism under binary outcomes means cross-method comparisons at the null can be confounded by method-specific calibration rather than power.

## Evidence

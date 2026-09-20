---
name: synthetic-otu-count-simulation
description: Use when benchmarking microbiome feature-screening or multivariate association
  tests on controlled synthetic data — specifically when OTU counts must mimic a real
  throat microbiome composition via multinomial sampling.
when_to_use_negative:
- Analyzing an actual observed OTU table for hypothesis testing — this skill only
  fabricates synthetic counts calibrated to a real composition
- Simulating compositional or zero-inflated count data where the multinomial single-depth
  model is inadequate and a Dirichlet-multinomial or overdispersed model is required
- Association analysis on real taxa-sets restricted to at least five taxa (e.g., the
  115 genera / 557-OTU filtered gut community), where simulation is unnecessary
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0610
tools:
- name: R
  role: Computing environment in which the multinomial OTU count simulation and the
    AMAT pipeline (distance correlation screening with B permutations) are implemented
  repo: https://github.com/kzb193/AMAT
- name: AMAT
  role: 'Consumer of the simulated data: its Algorithm 1 screens features via distance
    correlation learning, selecting features whose observed DC exceeds the permuted
    mean, with a max-DC fallback rule'
  repo: https://github.com/kzb193/AMAT
- name: QIIME
  role: Pipeline that produced the denoised V1-V2 16S rRNA OTU table whose composition
    calibrates the simulation probabilities
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqab120
    title: Adaptive and powerful microbiome multivariate association analysis via
      feature selection
  source_content_hash: sha256:84cd0fa065e46c36e7d982edfb579a9dd183eae94d4ddca9d30d3d0be6582ead
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
  - QIIME
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0610
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# Synthetic OTU count simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates synthetic n x p OTU count matrices by drawing counts from a multinomial distribution calibrated to the real throat microbiome composition (856 OTUs, 60 samples), providing the simulation substrate on which the AMAT distance-correlation feature-screening procedure (Algorithm 1) is evaluated.

## When to use

Use when benchmarking microbiome feature-screening or multivariate association tests on controlled synthetic data — specifically when OTU counts must mimic a real throat microbiome composition via multinomial sampling.

## When NOT to use

- Analyzing an actual observed OTU table for hypothesis testing — this skill only fabricates synthetic counts calibrated to a real composition
- Simulating compositional or zero-inflated count data where the multinomial single-depth model is inadequate and a Dirichlet-multinomial or overdispersed model is required
- Association analysis on real taxa-sets restricted to at least five taxa (e.g., the 115 genera / 557-OTU filtered gut community), where simulation is unnecessary

## Inputs

- Real throat microbiome OTU table (856 OTUs from 60 samples, post quality-control) used to estimate multinomial probabilities p1i..p856i
- Sequencing depth vector ni per simulated sample
- Sample size n and number of taxa p for the synthetic matrix
- Adjusted response vector specification (covariate-adjusted outcome) from the paper's simulation model

## Outputs

- Synthetic n x p OTU count matrix with counts drawn from Multinomial(ni; p1i, ..., p856i)
- Known ground-truth signal-set indices used to embed association
- Simulated adjusted response vector for association testing

## How to apply

Build an OTU probability vector (p1, p2, …, p856) from the real throat microbiome OTU table (856 OTUs, 60 samples, 28 smokers) after quality control and data filtering. For each simulated sample i, draw OTU counts as (Zi1, Zi2, …, Zi856) ~ Multinomial(ni; p1i, p2i, …, p856i), choosing sample size n (e.g., n=100 in the paper's Scenario I) and signal density (10%, 20%, or 30%) to embed a known signal set. Pair the simulated count matrix with an adjusted response vector per the paper's simulation model so that association strength is known. Feed the simulated matrix and response into the AMAT R implementation: compute per-feature distance correlations dcj against the adjusted response, build per-feature null distributions via B permutations, screen features whose observed DC exceeds the permuted mean, and apply the fallback rule (select the maximum-DC feature) if nothing passes. The known ground-truth signal set then lets you compare AMAT's power against aMiSPU, OMiRKAT, OMiAT, LDM, and MiHC while checking type I error control.

## Related tools

- **R** (Computing environment in which the multinomial OTU count simulation and the AMAT pipeline (distance correlation screening with B permutations) are implemented) — https://github.com/kzb193/AMAT
- **AMAT** (Consumer of the simulated data: its Algorithm 1 screens features via distance correlation learning, selecting features whose observed DC exceeds the permuted mean, with a max-DC fallback rule) — https://github.com/kzb193/AMAT
- **QIIME** (Pipeline that produced the denoised V1-V2 16S rRNA OTU table whose composition calibrates the simulation probabilities)

## Examples

```
sim <- t(rmultinom(n = 100, size = ni, prob = p_hat))  # (Zi1,...,Zi856) ~ Multinomial(ni; p1i,...,p856i)
```

## Evaluation signals

- Each simulated sample's counts sum exactly to the specified sequencing depth ni, and the matrix has n rows and p columns of non-negative integers
- Empirical per-OTU frequencies across samples approximate the throat microbiome probability vector used for the multinomial draws
- The embedded ground-truth signal set (e.g., 10%/20%/30% signal density at n=100) is recoverable by AMAT's screening stage without error
- The screening output S is a valid non-empty subset of {1,...,p}; the fallback (maximum distance correlation) fires whenever no feature exceeds its permuted mean
- Downstream AMAT tests on the simulated data preserve the correct type I error rate, as reported for the paper's extensive simulation studies

## Limitations

- The multinomial model assumes a fixed sequencing depth ni per sample and independent draws, so it does not capture overdispersion or zero inflation beyond what the empirical composition implies
- Calibration inherits the composition of a single real dataset (throat microbiome, 856 OTUs / 60 samples, 28 smokers); conclusions may not transfer to gut or other communities without re-estimating probabilities
- Power estimates from the simulation are conditional on the chosen signal densities (10%, 20%, 30%) and n=100 configuration

## Evidence

---
name: microbiome-feature-screening
description: Use when a high-dimensional OTU/taxa count matrix (e.g. 856 OTUs x 60
  samples) and an adjusted response vector must be reduced to a non-empty subset S
  of taxa likely associated with the outcome, prior to a data-adaptive multivariate
  association test under a generalized linear model.
when_to_use_negative:
- The taxa set is already small and pre-specified — the paper restricts association
  analysis to taxa-sets containing at least five taxa, where screening adds no value
- The response has not been covariate-adjusted; screening is defined on the adjusted
  response, and an unadjusted outcome would confound the distance correlations
- The goal is per-taxon (univariate) testing rather than multivariate clade/community
  association analysis
edam_topics:
- http://edamontology.org/topic_0610
- http://edamontology.org/topic_3697
tools:
- name: AMAT
  role: R implementation providing the distance correlation learning routine and Algorithm
    1 screening with the max-DC fallback rule
  repo: https://github.com/kzb193/AMAT
- name: R
  role: Computing environment in which AMAT screening is executed (synthetic data
    generation, permutation, selection, and output of S)
  repo: https://github.com/kzb193/AMAT
- name: QIIME
  role: Upstream pipeline used to construct the OTUs (denoised 16S rRNA V1-V2 sequences)
    that feed the OTU count matrix screened by this skill
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqab120
    title: Adaptive and powerful microbiome multivariate association analysis via
      feature selection
  source_content_hash: sha256:e549f2c9224fab56190aba478e2b5bb159dd7051fae6173e5d1b14e693e9e993
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
  - http://edamontology.org/topic_0610
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# microbiome-feature-screening

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A distance-correlation-based, data-adaptive feature screening stage (Algorithm 1 of AMAT) that shrinks noise taxa to exactly zero before multivariate microbiome association testing, selecting a data-driven testing subset S of outcome-associated taxa from an OTU matrix and adjusted response.

## When to use

Use when a high-dimensional OTU/taxa count matrix (e.g. 856 OTUs x 60 samples) and an adjusted response vector must be reduced to a non-empty subset S of taxa likely associated with the outcome, prior to a data-adaptive multivariate association test under a generalized linear model.

## When NOT to use

- The taxa set is already small and pre-specified — the paper restricts association analysis to taxa-sets containing at least five taxa, where screening adds no value
- The response has not been covariate-adjusted; screening is defined on the adjusted response, and an unadjusted outcome would confound the distance correlations
- The goal is per-taxon (univariate) testing rather than multivariate clade/community association analysis

## Inputs

- n x p OTU count matrix (e.g. 856 OTUs from 60 throat microbiome samples, or simulated via multinomial/Dirichlet-multinomial mimicking real composition)
- Adjusted response vector (continuous or binary outcome, covariate-adjusted)
- Number of permutations B for building per-feature null distributions of distance correlation

## Outputs

- Selected index set S, a valid non-empty subset of {1,...,p}
- Per-feature permuted distance-correlation null distributions (B per feature)
- Screening script / results file recording S for downstream data-adaptive association testing

## How to apply

Generate or load an n x p OTU count matrix (e.g. counts drawn from a multinomial distribution mimicking throat microbiome composition: (Zi1,...,Zi856) ~ Multinomial(ni; pi1,...,pi856)) and an adjusted response vector. For each feature j = 1..p, compute the distance correlation dcj between the feature's abundance vector and the adjusted response using the distance correlation learning routine in the AMAT R implementation (https://github.com/kzb193/AMAT). Permute the response B times (the paper uses 10,000 permutations for testing) and recompute distance correlations per feature to build per-feature null distributions. Select features whose observed dcj exceeds the mean of its B permuted distance correlations; apply the fallback rule — if no feature passes, select the single feature with maximum observed distance correlation. The rationale is that AMAT 'alleviates the burden of noise accumulation via distance correlation learning', and setting noise weights to exactly zero avoids the power loss that small non-zero score-statistic weights on noise taxa cause in competing methods.

## Related tools

- **AMAT** (R implementation providing the distance correlation learning routine and Algorithm 1 screening with the max-DC fallback rule) — https://github.com/kzb193/AMAT
- **R** (Computing environment in which AMAT screening is executed (synthetic data generation, permutation, selection, and output of S)) — https://github.com/kzb193/AMAT
- **QIIME** (Upstream pipeline used to construct the OTUs (denoised 16S rRNA V1-V2 sequences) that feed the OTU count matrix screened by this skill)

## Examples

```
remotes::install_github("kzb193/AMAT"); library(AMAT)  # then run Algorithm 1: per-feature distance correlation vs adjusted response with B-permutation screening and max-DC fallback to obtain S
```

## Evaluation signals

- The selected index set S is a valid, non-empty subset of {1,...,p} for every run
- The fallback rule fires whenever the mean-threshold rule selects no feature, guaranteeing non-emptiness of S
- The screening script runs end-to-end without error on both simulated (multinomial OTU counts) and real OTU tables
- In simulation scenarios with known signals, features in S should include true signal taxa at a rate well above chance, consistent with AMAT's reported power gains (e.g. 44.7%-52.5% vs 33.6%-38.3% for runner-ups under Scenario I)
- Type I error remains well controlled downstream, indicating S is not systematically importing noise taxa under the global null

## Limitations

- The fallback rule forces selection of the maximum-DC feature even when no feature shows signal, so S can contain pure noise taxa under the global null
- Screening quality depends on B permutation replicates per feature; too few permutations make per-feature means unstable and thresholds noisy
- Distance correlation with a single adjusted response may miss features whose effects cancel in direction, a cancellation issue the paper notes for low-rank aggregation approaches

## Evidence

- [full_text] Algorithm 1 fallback rule (source card): [evidence span withheld — anchor status: not_found]

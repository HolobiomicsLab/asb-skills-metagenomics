---
name: distance-correlation-computation
description: 'Use when performing feature screening for microbiome multivariate association
  testing: given a high-dimensional OTU count matrix (e.g. 856 taxa) and an adjusted
  response, decide which taxa enter the testing subset.'
when_to_use_negative:
- The taxon set has already been reduced to a small, pre-specified clade and the article's
  taxa-sets-of-at-least-five-taxa scope is met without data-adaptive screening
- You need a single omnibus P-value for an already fixed feature set (e.g. direct
  use of OMiRKAT/aMiSPU) rather than selection of which taxa to test
- The response is multivariate and no adjusted single-response vector has been constructed
  for the distance correlation computation
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Computing environment for running the distance correlation learning routine
    and permutation-based screening
  repo: https://github.com/kzb193/AMAT
- name: AMAT
  role: Provides the distance correlation learning implementation used to rank taxa
    and select the testing subset (Algorithm 1)
  repo: https://github.com/kzb193/AMAT
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqab120
    title: Adaptive and powerful microbiome multivariate association analysis via
      feature selection
  source_content_hash: sha256:25aa55b08a5dcee44edef264be67c9dc1a4569945e06e5b5a1dd1f6e874be984
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# distance-correlation-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes distance correlations between individual OTU/taxon abundance vectors and an adjusted response to build the data-adaptive feature screening step of AMAT (Algorithm 1), which selects the testing subset S before the multivariate association test.

## When to use

Use when performing feature screening for microbiome multivariate association testing: given a high-dimensional OTU count matrix (e.g. 856 taxa) and an adjusted response, decide which taxa enter the testing subset.

## When NOT to use

- The taxon set has already been reduced to a small, pre-specified clade and the article's taxa-sets-of-at-least-five-taxa scope is met without data-adaptive screening
- You need a single omnibus P-value for an already fixed feature set (e.g. direct use of OMiRKAT/aMiSPU) rather than selection of which taxa to test
- The response is multivariate and no adjusted single-response vector has been constructed for the distance correlation computation

## Inputs

- n x p OTU count matrix (e.g. multinomial-simulated counts mimicking the real throat microbiome composition, or a real OTU table such as the 856-OTU / 60-sample throat microbiome data set)
- adjusted response vector (continuous or binary outcome)

## Outputs

- Selected index set S (a non-empty subset of {1,...,p}) of taxa for the downstream data-adaptive association test
- Per-feature distance correlation statistics and their permutation-based null means

## How to apply

For each taxon j = 1..p, compute the distance correlation dcj between the taxon's abundance vector and the adjusted response using the distance correlation learning routine from the AMAT R implementation (github.com/kzb193/AMAT). Permute the response B times and recompute distance correlations on each permuted dataset to build a per-feature null distribution; screen features whose observed dcj exceeds the mean of its B permuted correlations. Apply the fallback rule from Algorithm 1: if no column of Z passes the permuted-mean threshold, select the single feature with the maximum observed distance correlation. The rationale is to shrink noise features to exactly zero and alleviate noise accumulation that would otherwise deteriorate the downstream data-adaptive test under the generalized linear model framework.

## Related tools

- **R** (Computing environment for running the distance correlation learning routine and permutation-based screening) — https://github.com/kzb193/AMAT
- **AMAT** (Provides the distance correlation learning implementation used to rank taxa and select the testing subset (Algorithm 1)) — https://github.com/kzb193/AMAT

## Examples

```
Rscript -e 'remotes::install_github("kzb193/AMAT"); library(AMAT)'
```

## Evaluation signals

- The screening script runs end-to-end without error
- S is a valid, non-empty subset of feature indices {1,...,p}
- The fallback rule fires exactly when no feature exceeds its permuted mean threshold, and then S contains the single maximum-DC feature
- Selection precision and recall computed against a known simulated signal set lie in [0,1] and can be benchmarked against aMiSPU, OMiRKAT, OMiAT, LDM and MiHC (Supplementary Tables S1-S2)

## Limitations

- With mostly noise taxa, enough power to detect the association signal is the central challenge; the screening must not drop truly associated taxa, and recall can suffer if the permuted-mean threshold is too strict
- Type I error must be re-checked downstream: in the article's simulations all tests had well-controlled type I error rates except MiHC, which was conservative especially with binary outcomes
- Screening quality depends on the OTU count distribution model; simulated counts were drawn from a multinomial distribution mimicking real throat microbiome composition

## Evidence

- [abstract] Distance correlation learning drives AMAT screening: "AMAT first alleviates the burden of noise accumulation via distance correlation learning, and then conducts a data-adaptive association test"
- [abstract] AMAT R implementation location: "A free implementation of AMAT in R computing environment is available at https://github.com/kzb193/AMAT."
- [full_text] Fallback rule of Algorithm 1: [evidence span withheld — anchor status: not_found]
- [results] Simulation model for OTU counts used to benchmark screening: "the observed OTU counts were randomly generated from a multinomial distribution: (Zi1, Zi2, …, Zi856) ∼ Multinomial(ni; p1i, p2i, …, p856i)."
- [full_text] Superiority of the DC-based selection over competing feature-selection strategies: [evidence span withheld — anchor status: not_found]

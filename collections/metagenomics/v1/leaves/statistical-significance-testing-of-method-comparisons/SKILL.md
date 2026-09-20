---
name: statistical-significance-testing-of-method-comparisons
description: Use when comparing competing source-tracking methods on identical simulated
  or benchmark inputs and you need significance levels for accuracy differences, e.g.,
  testing MSE gaps against P < 1e-6 thresholds across varying unknown-source fractions.
when_to_use_negative:
- Your data are real (not simulated) sinks without known ground-truth mixing proportions,
  so MSE and false positive rate cannot be computed
- You are evaluating a single method in isolation rather than a head-to-head comparison
  on identical inputs — significance testing of differences is meaningless without
  a paired competitor run
- Only one candidate source environment is being considered, so sparsity/source-selection
  error (the driver of MSE differences in this benchmark) does not arise
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3053
- http://edamontology.org/topic_3301
tools:
- name: STENSL
  role: Method under test; EM algorithm with L1-regularized Lasso source selection
    whose MSE is compared against competitors
  repo: https://github.com/cozygene/FEAST/tree/STENSL
- name: FEAST
  role: Competitor source tracking method run on identical simulated sinks; STENSL's
    MSE is tested as significantly lower (P < 1.01e-6); STENSL is a branch of the
    FEAST codebase
  repo: https://github.com/cozygene/FEAST
- name: SourceTracker2
  role: Competitor source tracking method whose MSE is tested against STENSL's (P
    < 9.13e-7)
- name: R
  role: Environment in which FEAST/STENSL are installed (devtools) and run to produce
    the per-sink mixing proportions entering the comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/msystems.00995-21
    title: 'STENSL: Microbial Source Tracking with ENvironment SeLection'
  source_content_hash: sha256:f01585014d1fa7d328a1a994f62ad82cdc550649eaa22d85d8c27af5e01ec701
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FEAST
  - R
  - STENSL
  - SourceTracker2
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3053
  - http://edamontology.org/topic_3301
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Statistical significance testing of method comparisons

## Summary

Benchmark multiple microbial source tracking methods (STENSL, FEAST, SourceTracker2, RAD) on identical simulated sinks and attach statistical significance to their performance differences, so claims of superior accuracy (e.g., lower MSE) are quantitatively supported rather than anecdotal.

## When to use

Use when comparing competing source-tracking methods on identical simulated or benchmark inputs and you need significance levels for accuracy differences, e.g., testing MSE gaps against P < 1e-6 thresholds across varying unknown-source fractions.

## When NOT to use

- Your data are real (not simulated) sinks without known ground-truth mixing proportions, so MSE and false positive rate cannot be computed
- You are evaluating a single method in isolation rather than a head-to-head comparison on identical inputs — significance testing of differences is meaningless without a paired competitor run
- Only one candidate source environment is being considered, so sparsity/source-selection error (the driver of MSE differences in this benchmark) does not arise

## Inputs

- Simulated sink-source sets: EMP-derived synthetic sinks as convex mixtures of K=10 true sources from M=50 candidates, unknown contribution 0%-90%
- Per-sink estimated mixing proportion vectors from each method (rows summing to 1, including an unknown-source column)
- Ground-truth contributing source labels and true unknown contribution per simulated sink

## Outputs

- simulation_benchmark_results.csv with MSE of mixing proportions, FPR of identified sources, and unknown-contribution absolute error for all four methods
- Significance assessments of pairwise MSE differences (STENSL vs SourceTracker2 P < 9.13e-7; vs FEAST P < 1.01e-6; vs RAD P < 9.13e-7)

## How to apply

First generate a controlled benchmark: draw source environments from Earth Microbiome Project samples, simulate synthetic sinks as convex mixtures of K=10 contributing sources selected from M=50 candidate sources, and sweep the unknown-source contribution from 0% to 90%. Run every competitor (STENSL with its EM + L1-regularized Lasso source selection at lambda = 1e-6 with cross-validation, FEAST, SourceTracker2, RAD) on the identical sink-source sets so differences reflect the algorithms, not the data. For each method compute the MSE of estimated mixing proportions, the false positive rate of identified contributing sources, and the absolute error of the estimated unknown contribution. Then test the MSE differences for significance — in the STENSL paper the reported thresholds are STENSL vs SourceTracker2 P < 9.13e-7, vs FEAST P < 1.01e-6, vs RAD P < 9.13e-7. Compile all per-method metrics and significance outcomes into a results table (e.g., simulation_benchmark_results.csv), validating against Table S1 / Fig 2 expectations.

## Related tools

- **STENSL** (Method under test; EM algorithm with L1-regularized Lasso source selection whose MSE is compared against competitors) — https://github.com/cozygene/FEAST/tree/STENSL
- **FEAST** (Competitor source tracking method run on identical simulated sinks; STENSL's MSE is tested as significantly lower (P < 1.01e-6); STENSL is a branch of the FEAST codebase) — https://github.com/cozygene/FEAST
- **SourceTracker2** (Competitor source tracking method whose MSE is tested against STENSL's (P < 9.13e-7))
- **R** (Environment in which FEAST/STENSL are installed (devtools) and run to produce the per-sink mixing proportions entering the comparison)

## Examples

```
FEAST_output <- FEAST(C = otus, metadata = metadata, different_sources_flag = 1, dir_path = "~/FEAST/Data_files/", outfile="demo")
```

## Evaluation signals

- The compiled results table reports all three metrics (MSE, FPR, unknown absolute error) for all four methods, matching Table S1 / Fig 2 scope
- STENSL's MSE is significantly lower than each competitor at the stated P-value thresholds (P < 9.13e-7 vs SourceTracker2 and RAD; P < 1.01e-6 vs FEAST)
- Per-method mixing proportion estimates include an unknown-source fraction and each method's output rows sum to 1 across sources plus unknown
- Unknown-contribution absolute error is assessed across the full 0%-90% unknown sweep, with STENSL the only method consistently estimating real source contributions
- False positive rate is computed over the full M=50 candidate source pool, not just the K=10 true contributors

## Limitations

- The detailed simulation protocol is delegated to Supplementary Text S1 and is not included in the main document, so the exact significance test used must be sourced from the supplement
- The Lasso hyperparameter was fixed to a single value (lambda = 1e-6) without a reported sensitivity analysis, so benchmark conclusions are conditional on that choice
- P-values are stated as thresholds (e.g., P < 9.13e-7) without a formal citation of the underlying test statistic in the main text

## Evidence

- [methods] Assess significance of MSE differences (STENSL vs SourceTracker2 P < 9.13e-7, vs FEAST P < 1.01e-6, vs RAD P < 9.13e-7): [evidence span withheld — anchor status: not_found]
- [methods] Per-method metrics computed on identical simulated sinks: [evidence span withheld — anchor status: not_found]
- [results] STENSL consistently more accurate in EMP simulations: [evidence span withheld — anchor status: not_found]
- [abstract] Abstract-level claim of significant accuracy advantage: "STENSL is significantly more accurate in identifying the contributing sources as well as the unknown source, even when considering hundreds of potential source environments"

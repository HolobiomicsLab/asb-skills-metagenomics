---
name: roc-auc-evaluation
description: Use when you need to benchmark a differential abundance method's classification
  performance against a comparator on simulated taxa with known null/differential
  status, requiring ROC curves, AUC with 95% CI, and a DeLong test for correlated
  ROC curves.
when_to_use_negative:
- Analyzing real microbiome data without known ground-truth null/differential labels
  — ROC/AUC classification metrics are not computable
- Comparing methods on independently simulated datasets (DeLong's test requires correlated
  ROC curves from the same paired data)
- The goal is meta-analysis pooling of effect estimates across studies (use random-effects
  meta-analysis via meta.taxa instead)
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: metamicrobiomeR
  role: Provides taxa.compare with propmed.rel='gamlss' to fit zero-inflated beta
    GAMLSS and produce differential abundance p-values for the ROC/AUC analysis
  repo: https://github.com/nhanhocu/metamicrobiomeR
- name: GAMLSS-BEZI
  role: Zero-inflated beta modeling family whose p-values serve as the ranking score
    for one ROC curve
  repo: https://github.com/nhanhocu/metamicrobiomeR
- name: LMAS
  role: Comparator differential abundance method whose p-values provide the second
    ROC curve for paired AUC comparison
- name: pROC
  role: Computes ROC curves, AUC point estimates, 95% confidence intervals, and DeLong's
    test for correlated ROC curves
- name: ggplot2
  role: Plots the ROC curves for visual comparison of both methods
- name: R
  role: Runtime environment for simulation, fitting, ROC computation, and plotting
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-019-2744-2
    title: 'metamicrobiomeR: an R package for analysis of microbiome relative abundance
      data using zero-inflated beta GAMLSS and meta-analysis across studies using
      random effects models'
  source_content_hash: sha256:9e09fcf7ba831630a22d3e55190126847f1ab6e57091a8ef30b67f49dc739650
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nhanhocu/metamicrobiomeR
  tools:
  - GAMLSS-BEZI
  - LMAS
  - R
  - ggplot2
  - metamicrobiomeR
  - pROC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# roc-auc-evaluation

## Summary

Benchmark differential abundance methods (e.g., GAMLSS-BEZI vs LMAS) on simulated zero-inflated beta relative abundance data with known ground truth by computing ROC curves, AUC estimates with 95% CIs, and comparing paired AUCs via DeLong's test.

## When to use

Use when you need to benchmark a differential abundance method's classification performance against a comparator on simulated taxa with known null/differential status, requiring ROC curves, AUC with 95% CI, and a DeLong test for correlated ROC curves.

## When NOT to use

- Analyzing real microbiome data without known ground-truth null/differential labels — ROC/AUC classification metrics are not computable
- Comparing methods on independently simulated datasets (DeLong's test requires correlated ROC curves from the same paired data)
- The goal is meta-analysis pooling of effect estimates across studies (use random-effects meta-analysis via meta.taxa instead)

## Inputs

- Simulated zero-inflated beta relative abundance matrix for 800 species (400 null, 400 differential across four mu-shift settings) with known ground-truth labels
- Case/control sample grouping metadata (n=100 per group)
- Differential abundance p-values from GAMLSS-BEZI via taxa.compare
- Differential abundance p-values from LMAS on the same simulated datasets

## Outputs

- ROC curves for GAMLSS-BEZI and LMAS (ggplot2 plot, e.g., Fig 1a)
- AUC point estimates with 95% confidence intervals per method
- DeLong test p-value for the paired AUC comparison
- CSV record of AUC estimates, 95% CIs, and DeLong p-value

## How to apply

Simulate zero-inflated beta relative abundance data in R for 800 species with known ground truth — 400 null and 400 differential species allocated evenly across four mu-shift settings, with n=100 samples per case/control group. Obtain differential abundance p-values from GAMLSS-BEZI by fitting each taxon with metamicrobiomeR's taxa.compare (propmed.rel='gamlss', p.adjust.method='fdr'), and run LMAS on the identical simulated datasets for comparable p-values. Compute ROC curves and AUC statistics for both methods with pROC, then compare the paired AUCs using DeLong's test for correlated ROC curves (both methods score the same simulated species). Plot both ROC curves with ggplot2 and write the AUC estimates, 95% CIs, and DeLong p-value to a CSV record. The rationale is grounded in the paper's simulation framework demonstrating that GAMLSS-BEZI, which explicitly models the zero-inflated beta distribution of relative abundances, outperforms methods that do not address zero inflation.

## Related tools

- **metamicrobiomeR** (Provides taxa.compare with propmed.rel='gamlss' to fit zero-inflated beta GAMLSS and produce differential abundance p-values for the ROC/AUC analysis) — https://github.com/nhanhocu/metamicrobiomeR
- **GAMLSS-BEZI** (Zero-inflated beta modeling family whose p-values serve as the ranking score for one ROC curve) — https://github.com/nhanhocu/metamicrobiomeR
- **LMAS** (Comparator differential abundance method whose p-values provide the second ROC curve for paired AUC comparison)
- **pROC** (Computes ROC curves, AUC point estimates, 95% confidence intervals, and DeLong's test for correlated ROC curves)
- **ggplot2** (Plots the ROC curves for visual comparison of both methods)
- **R** (Runtime environment for simulation, fitting, ROC computation, and plotting)

## Examples

```
taxacom<-taxa.compare(taxtab=taxtab6.rm[[5]],propmed.rel="gamlss",comvar="bf",adjustvar="age.sample",longitudinal="yes",p.adjust.method="fdr")
```

## Evaluation signals

- Reproduced GAMLSS-BEZI AUC equals ~95.6 with 95% CI [94.2, 97.1]
- Reproduced LMAS AUC equals ~92.9 with 95% CI [91.1, 94.7]
- DeLong test p-value for the paired AUC difference is < 2.2e-16, with the invariant that GAMLSS-BEZI outperforms LMAS
- Both ROC curves are computed on the identical 800-species simulation (400 null, 400 differential across four mu-shift settings, n=100 per group), so the p-value inputs are paired
- AUC estimates include 95% confidence intervals from pROC and are persisted to a CSV record alongside the DeLong p-value

## Limitations

- Validation values (AUC 95.6 vs 92.9, p < 2.2e-16) come from a single simulation configuration (800 species, four mu-shift settings, n=100 per group); other simulation settings may shift the AUC magnitudes
- GitHub package version has minor changes relative to the CRAN version, which users are recommended to use; results may differ slightly between versions
- Methods that do not address the zero-inflated beta distribution of relative abundances (e.g., t-tests, Wilcoxon tests, linear models with arcsin square-root transformation) may rank poorly, which is part of the expected signal rather than a pipeline error

## Evidence

- [abstract] Simulation performance claim: "Both simulation studies and application to real microbiome data demonstrate that GAMLSS-BEZI well performs in testing differential relative abundances of microbial taxonomies."
- [intro] Justification for BEZI-based ranking: "However, these methods do not address the actual distribution of the microbial taxonomy relative abundance data, which resemble a zero-inflated beta distribution."
- [methods] taxa.compare invocation used to obtain p-values: "taxacom6.zi.rmg<-taxa.compare(taxtab=taxtab6.rm[[5]],propmed.rel="gamlss",comvar="bf",adjustvar="age.sample",longitudinal="yes",p.adjust.method="fdr")"
- [readme] Simulation results documented in the paper: "The published paper describing the methodology as well as simulations and example analyses/results can be found at BMC Bioinformatics."

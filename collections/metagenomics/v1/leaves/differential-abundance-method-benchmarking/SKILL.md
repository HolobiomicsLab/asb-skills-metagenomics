---
name: differential-abundance-method-benchmarking
description: Use when you need to quantitatively compare two or more differential
  abundance testing methods on simulated microbiome relative abundance data with known
  differential/null species labels, to decide which method has detection power.
when_to_use_negative:
- Input is real observational data without known ground truth (null vs differential
  taxa) — ROC/AUC benchmarking requires true labels from simulation
- The goal is a single-method differential abundance analysis rather than comparing
  competing methods
- Data are absolute counts or compositional data you do not intend to model as zero-inflated
  beta relative abundances
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: metamicrobiomeR
  role: Provides taxa.compare with GAMLSS-BEZI (propmed.rel='gamlss') to obtain differential
    abundance p-values for the candidate method
  repo: https://github.com/nhanhocu/metamicrobiomeR
- name: GAMLSS-BEZI
  role: Statistical family directly modeling zero-inflated beta relative abundance
    distribution used by the candidate method
  repo: https://github.com/nhanhocu/metamicrobiomeR
- name: LMAS
  role: Comparator differential abundance method applied to identical simulated datasets
- name: R
  role: Implementation language for simulation, model fitting, ROC computation, and
    plotting
  repo: https://github.com/nhanhocu/metamicrobiomeR
- name: ggplot2
  role: Plotting library for rendering the comparison ROC curves
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-019-2744-2
    title: 'metamicrobiomeR: an R package for analysis of microbiome relative abundance
      data using zero-inflated beta GAMLSS and meta-analysis across studies using
      random effects models'
  source_content_hash: sha256:269dff7e09ed4e62c50249f1b06a8d42208a96a994397f673e783058c65392d0
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# differential-abundance-method-benchmarking

## Summary

A simulation-based benchmarking workflow that compares differential abundance methods (GAMLSS-BEZI via metamicrobiomeR vs LMAS) on zero-inflated beta relative abundance data with known ground truth, using ROC/AUC and DeLong's test for correlated ROC curves.

## When to use

Use when you need to quantitatively compare two or more differential abundance testing methods on simulated microbiome relative abundance data with known differential/null species labels, to decide which method has superior detection power.

## When NOT to use

- Input is real observational data without known ground truth (null vs differential taxa) — ROC/AUC benchmarking requires true labels from simulation
- The goal is a single-method differential abundance analysis rather than comparing competing methods
- Data are absolute counts or compositional data you do not intend to model as zero-inflated beta relative abundances

## Inputs

- Simulated zero-inflated beta relative abundance matrix with known ground-truth labels (null vs differential species)
- Differential abundance p-value tables from each method under comparison (e.g., metamicrobiomeR taxa.compare output and LMAS output)
- Binary ground-truth classification vector per species

## Outputs

- ROC curves per method (plotted with pROC/ggplot2)
- AUC estimates with 95% confidence intervals per method
- DeLong test p-value comparing correlated ROC curves
- CSV record of AUCs, CIs, and p-value

## How to apply

Simulate zero-inflated beta relative abundance data with known ground truth (e.g., 800 species: 400 null and 400 differential spread evenly across four mu-shift settings, with n=100 samples per group). Apply each candidate method to the identical simulated datasets: fit GAMLSS with the zero-inflated beta (BEZI) family using metamicrobiomeR's taxa.compare (propmed.rel='gamlss', p.adjust.method='fdr') and run LMAS in parallel. Compute ROC curves and AUC statistics from the p-values with pROC, then compare paired AUCs using DeLong's test for correlated ROC curves. Report each method's AUC with 95% confidence intervals, the DeLong p-value, and overlay the ROC curves (e.g., with ggplot2) to justify the choice of method.

## Related tools

- **metamicrobiomeR** (Provides taxa.compare with GAMLSS-BEZI (propmed.rel='gamlss') to obtain differential abundance p-values for the candidate method) — https://github.com/nhanhocu/metamicrobiomeR
- **GAMLSS-BEZI** (Statistical family directly modeling zero-inflated beta relative abundance distribution used by the candidate method) — https://github.com/nhanhocu/metamicrobiomeR
- **LMAS** (Comparator differential abundance method applied to identical simulated datasets)
- **R** (Implementation language for simulation, model fitting, ROC computation, and plotting) — https://github.com/nhanhocu/metamicrobiomeR
- **ggplot2** (Plotting library for rendering the comparison ROC curves)

## Examples

```
taxacom<-taxa.compare(taxtab=simtab,propmed.rel="gamlss",comvar="group",longitudinal="no",p.adjust.method="fdr")
```

## Evaluation signals

- Reproduced AUCs match the published benchmark: GAMLSS-BEZI AUC = 95.6 (95% CI 94.2–97.1) and LMAS AUC = 92.9 (95% CI 91.1–94.7)
- DeLong's test for the paired ROC curves yields p < 2.2e-16, confirming GAMLSS-BEZI significantly outperforms LMAS
- Simulation design invariants hold: 800 species total, 400 null and 400 differential evenly split across four mu-shift settings, n=100 per group
- Both methods are evaluated on identical simulated datasets so paired DeLong comparison is valid
- FDR correction (p.adjust.method='fdr') is applied to the taxa.compare p-values before ROC construction

## Limitations

- Results depend on the simulation settings (mu-shift magnitudes, group size n=100); performance may differ under other effect sizes or sample sizes
- GAMLSS-BEZI benchmarking presumes data resemble a zero-inflated beta distribution; other distributions may favor other methods
- Only two methods compared (GAMLSS-BEZI vs LMAS); conclusions do not generalize to all differential abundance tools

## Evidence

- [results] GAMLSS-BEZI vs LMAS AUC benchmark: [evidence span withheld — anchor status: not_found]
- [methods] taxa.compare invocation: "taxacom6.zi.rmg<-taxa.compare(taxtab=taxtab6.rm[[5]],propmed.rel="gamlss",comvar="bf",adjustvar="age.sample",longitudinal="yes",p.adjust.method="fdr")"
- [abstract] simulation performance finding: "Both simulation studies and application to real microbiome data demonstrate that GAMLSS-BEZI well performs in testing differential relative abundances of microbial taxonomies."
- [discussion] GAMLSS-BEZI rationale: "The advantages of GAMLSS-BEZI are: 1) it directly address the distribution of microbiome relative abundance data"
- [readme] install and load package: "install_github("nhanhocu/metamicrobiomeR")
library(metamicrobiomeR)"

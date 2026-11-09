---
name: delong-correlated-roc-comparison
description: Use when two DA methods have been applied to the same samples with known
  ground-truth labels and you need a statistically valid paired comparison of their
  ROC AUCs (e.g., 800 simulated bacterial species, 400 null / 400 differential).
when_to_use_negative:
- The two methods were run on DIFFERENT datasets or taxa sets (curves are not correlated,
  so paired DeLong is inappropriate)
- The task is calibration, effect-size, or FDR-power assessment rather than ranking/discrimination
  via ROC (p-values and effect estimates need other summaries)
- Data are not relative abundances resembling a zero-inflated beta distribution (e.g.,
  raw counts or compositional data transformed to log-ratios) — the simulation context
  here assumes zero-inflated beta relative abundance
tools:
- name: pROC
  role: Computes ROC curves, AUC statistics with 95% confidence intervals, and DeLong's
    test for correlated ROC curves
- name: metamicrobiomeR
  role: Fits GAMLSS with zero-inflated beta (BEZI) family per taxon via taxa.compare
    (propmed.rel='gamlss', p.adjust.method='fdr') to produce the primary method's
    p-values
  repo: https://github.com/nhanhocu/metamicrobiomeR
- name: LMAS
  role: Comparator differential-abundance method run on the identical simulated datasets
    to yield paired p-values
- name: ggplot2
  role: Plots the paired ROC curves for GAMLSS-BEZI and LMAS
- name: R
  role: Runtime environment for the simulation, fitting, ROC computation, and DeLong
    testing (R 3.4.2 or higher)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-019-2744-2
    title: 'metamicrobiomeR: an R package for analysis of microbiome relative abundance
      data using zero-inflated beta GAMLSS and meta-analysis across studies using
      random effects models'
  source_content_hash: sha256:6e56f3cceec434822af7b80df885768b15c7871ba891432fe0e9cfb4aec98d37
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nhanhocu/metamicrobiomeR
  tools:
  - LMAS
  - R
  - ggplot2
  - metamicrobiomeR
  - pROC
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# delong-correlated-roc-comparison

## Summary

A benchmarking skill for comparing two differential-abundance classifiers that score the SAME simulated taxa (correlated/paired predictions): compute ROC curves and AUCs with pROC in R, then compare paired AUCs with DeLong's test. Used in metamicrobiomeR's simulation to show GAMLSS-BEZI (AUC 95.6) significantly outperforms LMAS (AUC 92.9) on zero-inflated beta relative abundance data.

## When to use

Use when two DA methods have been applied to the same samples with known ground-truth labels and you need a statistically valid paired comparison of their ROC AUCs (e.g., 800 simulated bacterial species, 400 null / 400 differential).

## When NOT to use

- The two methods were run on DIFFERENT datasets or taxa sets (curves are not correlated, so paired DeLong is inappropriate)
- The task is calibration, effect-size, or FDR-power assessment rather than ranking/discrimination via ROC (p-values and effect estimates need other summaries)
- Data are not relative abundances resembling a zero-inflated beta distribution (e.g., raw counts or compositional data transformed to log-ratios) — the simulation context here assumes zero-inflated beta relative abundance

## Inputs

- Paired differential-abundance p-value vectors from two methods applied to the same taxa (e.g., GAMLSS-BEZI p-values from metamicrobiomeR taxa.compare and LMAS p-values)
- Binary ground-truth labels for each taxon (null vs differential), e.g., 400 null and 400 differential species from the mu-shift simulation
- Simulated zero-inflated beta relative abundance data (800 species, four mu-shift settings, n=100 per group)

## Outputs

- ROC curves for each method with AUC estimates and 95% confidence intervals (via pROC)
- DeLong test p-value for the paired AUC comparison
- ROC curve plot (e.g., ggplot2) and a CSV record of AUCs, CIs, and p-value

## How to apply

Simulate ground-truthed zero-inflated beta relative abundance data (e.g., 800 species: 400 null, 400 differential across four mu-shift settings, n=100 per group). Fit each method on the identical datasets — e.g., metamicrobiomeR's taxa.compare with propmed.rel='gamlss' (zero-inflated beta BEZI family) and p.adjust.method='fdr' to get GAMLSS-BEZI p-values, and the comparator (LMAS) on the same taxa — then compute ROC curves and AUCs with pROC using the known null/differential labels. Because both methods score the same taxa, the ROC curves are correlated, so use DeLong's test for correlated ROC curves (pROC roc.test) rather than treating AUCs as independent. Plot the paired ROC curves (e.g., ggplot2) and record both AUCs with 95% confidence intervals and the DeLong p-value to a CSV. The pairing is the rationale: DeLong's covariance estimate accounts for the shared data, giving valid inference (here p < 2.2e-16 in favor of GAMLSS-BEZI).

## Related tools

- **pROC** (Computes ROC curves, AUC statistics with 95% confidence intervals, and DeLong's test for correlated ROC curves)
- **metamicrobiomeR** (Fits GAMLSS with zero-inflated beta (BEZI) family per taxon via taxa.compare (propmed.rel='gamlss', p.adjust.method='fdr') to produce the primary method's p-values) — https://github.com/nhanhocu/metamicrobiomeR
- **LMAS** (Comparator differential-abundance method run on the identical simulated datasets to yield paired p-values)
- **ggplot2** (Plots the paired ROC curves for GAMLSS-BEZI and LMAS)
- **R** (Runtime environment for the simulation, fitting, ROC computation, and DeLong testing (R 3.4.2 or higher))

## Examples

```
roc.test(roc.gamlss.bezi, roc.lmas, method="delong", paired=TRUE)
```

## Evaluation signals

- GAMLSS-BEZI AUC should reproduce ~95.6 with 95% CI [94.2, 97.1]; LMAS AUC ~92.9 with 95% CI [91.1, 94.7]
- DeLong's test on the paired AUCs should yield p < 2.2e-16 favoring GAMLSS-BEZI
- pROC AUCs must lie on the 0-100 scale consistent with the reported CIs, and each method's predictions must be row-paired with the same 800 taxa and their ground-truth labels
- p-values passed to the ROC must come from FDR-adjusted taxa.compare output (p.adjust.method='fdr') so the two methods' scores are directly comparable
- The CSV record must contain both AUC estimates, both 95% CIs, and the DeLong p-value; the ROC plot must overlay both methods on the same axis

## Limitations

- The DeLong p-value depends on the simulation design (800 species, balanced 400 null/400 differential, n=100 per group); different imbalance or sample sizes will shift AUCs and power
- LMAS p-values are only comparable because LMAS was applied to the exact same simulated datasets; substituting a different simulation breaks the pairing
- In metamicrobiomeR, the CRAN version has updates/minor changes versus the GitHub version; results may differ between package versions (README recommends the CRAN version)

## Evidence

- [other] Compare the paired AUCs using the DeLong test for correlated ROC curves.: [evidence span withheld — anchor status: not_found]
- [other] GAMLSS-BEZI AUC 95.6 CI [94.2,97.1] outperforms LMAS 92.9, DeLong p<2.2e-16: "GAMLSS-BEZI (AUC = 95.6, 95% CI = [94.2, 97.1%]) significantly outperformed LMAS (AUC = 92.9"
- [abstract] GAMLSS-BEZI validated on simulations; applies GAMLSS with zero-inflated beta BEZI family: "Both simulation studies and application to real microbiome data demonstrate that GAMLSS-BEZI well performs in testing differential relative abundances of microbial taxonomies."
- [methods] methods: taxa.compare call producing GAMLSS-BEZI p-values: "taxacom6.zi.rmg<-taxa.compare(taxtab=taxtab6.rm[[5]],propmed.rel="gamlss",comvar="bf",adjustvar="age.sample",longitudinal="yes",p.adjust.method="fdr")"
- [readme] readme: package implements GAMLSS with zero-inflated beta for relative abundance analysis: "The *metamicrobiomeR* package implements Generalized Additive Model for Location, Scale and Shape (GAMLSS)"

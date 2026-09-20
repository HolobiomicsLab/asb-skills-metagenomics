---
name: cross-validated-phenotype-trait-prediction
description: Use when predicting phenotypic traits (e.g. aerobe, anaerobe, gramneg,
  thermophile) from comparative-genomics feature matrices (COG/eggNOG presence-absence
  profiles) and need cross-validated, kernel-comparable SVM accuracy and runtime figures.
when_to_use_negative:
- Input genomes are far below 60-70% completeness — the article notes most traits
  can only be reliably predicted at 60-70% completeness, so incomplete bins below
  this range will yield unreliable predictions
- The target trait lacks sufficient labeled training data — the article states good
  performance for further ecologically important traits is pending availability of
  sufficient training data
- The goal is a detailed functional analysis of individual genome features rather
  than genome-level phenotypic trait classification
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: PICA
  role: Framework for training and comparing machine-learning phenotype predictors
    on COG presence/absence matrices
  repo: github.com/univieCUBE/PICA
- name: PICA support vector machine plug-in
  role: Plug-in providing SVM classification of phenotypic traits; the linear kernel
    matched or beat RBF, sigmoid, and polynomial kernels for all ten traits
  repo: github.com/univieCUBE/PICA
- name: libSVM
  role: Underlying SVM implementation invoked by the PICA plug-in for cross-validated
    trait classification
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1186/1471-2105-16-s14-s1
    title: Prediction of microbial phenotypes based on comparative genomics
  source_content_hash: sha256:c421af089e5e5df464078f5e1e185541fd07b60f7a5818e2b514f180f240b9cc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PICA
  - PICA support vector machine plug-in
  - libSVM
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# cross-validated-phenotype-trait-prediction

## Summary

Trains and evaluates microbial phenotypic trait predictors from COG presence/absence profiles using PICA's SVM plug-in (libSVM) with 5-fold cross-validation repeated 10 times, scoring balanced accuracy per trait. It supports both kernel comparisons (linear vs RBF/sigmoid/polynomial) and SVM-only vs CPAR2SVM feature-selection comparisons.

## When to use

Use when predicting phenotypic traits (e.g. aerobe, anaerobe, gramneg, thermophile) from comparative-genomics feature matrices (COG/eggNOG presence-absence profiles) and need cross-validated, kernel-comparable SVM accuracy and runtime figures.

## When NOT to use

- Input genomes are far below 60-70% completeness — the article notes most traits can only be reliably predicted at 60-70% completeness, so incomplete bins below this range will yield unreliable predictions
- The target trait lacks sufficient labeled training data — the article states good performance for further ecologically important traits is pending availability of sufficient training data
- The goal is a detailed functional analysis of individual genome features rather than genome-level phenotypic trait classification

## Inputs

- COG/eggNOG presence/absence feature matrix for genomes from the PICA example dataset
- Binary phenotypic trait labels for the ten example traits (aerobe, anaerobe, facult, gramneg, halo, motile, photo, psychro, spore, therm)
- Choice of SVM kernel (linear, RBF, sigmoid, polynomial) or CPAR2SVM feature-selection pipeline

## Outputs

- kernel_balanced_accuracy.csv with one row per trait and one column per kernel (mean balanced accuracy over 10x5-fold CV)
- svm_vs_cpar2svm_comparison.csv with mean balanced accuracy and mean runtime columns for SVM-only and CPAR2SVM per trait
- Per-replicate balanced-accuracy and wall-clock-runtime records for each fold fit

## How to apply

Load the eggNOG/COG presence-absence feature matrix and the ten trait labels (aerobe, anaerobe, facult, gramneg, halo, motile, photo, psychro, spore, therm) from the PICA distribution (github.com/univieCUBE/PICA). For each trait, train a linear-kernel SVM via PICA's libSVM plug-in and evaluate with 5-fold cross-validation repeated 10 replicates, recording balanced accuracy per replicate. To compare kernels, repeat the identical protocol with RBF, sigmoid, and polynomial kernels and aggregate replicate scores into mean balanced accuracy per trait and kernel. To compare against the feature-selection pipeline, run CPAR2SVM (association-rule feature selection followed by libSVM) with the same 10x5-fold protocol and also record wall-clock runtime, since SVM-only runs roughly 3x faster with no accuracy loss. Write per-trait comparisons to a CSV table and check that the linear kernel's balanced accuracy is at least as high as every alternative kernel's for all ten traits, mirroring the paper's Figure 3 result.

## Related tools

- **PICA** (Framework for training and comparing machine-learning phenotype predictors on COG presence/absence matrices) — github.com/univieCUBE/PICA
- **PICA support vector machine plug-in** (Plug-in providing SVM classification of phenotypic traits; the linear kernel matched or beat RBF, sigmoid, and polynomial kernels for all ten traits) — github.com/univieCUBE/PICA
- **libSVM** (Underlying SVM implementation invoked by the PICA plug-in for cross-validated trait classification)

## Examples

```
phenotrex predict --classifier MY_TRAIT.pkl genome.fna > predictions.tsv
```

## Evaluation signals

- kernel_balanced_accuracy.csv contains one row for each of the ten traits and one column per kernel (linear, RBF, sigmoid, polynomial), with all values populated
- For every trait, the linear kernel's mean balanced accuracy is >= the RBF, sigmoid, and polynomial kernels' accuracies, reproducing the paper's Figure 3 outcome
- svm_vs_cpar2svm_comparison.csv has rows for all ten traits with both balanced-accuracy and runtime metrics populated
- SVM-only mean balanced accuracy shows no significant difference versus CPAR2SVM across the ten traits, while mean SVM-only runtime is approximately 3x lower averaged over the ten traits

## Limitations

- Most traits can be reliably predicted only in 60-70% complete genomes; predictions in genome bins below this completeness may be unreliable
- Performance for further ecologically important traits is pending availability of sufficient training data (ongoing work in the article)
- The original PICA codebase is succeeded by the phenotrex package, which builds on scikit-learn and adds new classifiers (e.g. XGboost)

## Evidence

- [abstract] The PICA framework facilitates application and comparison of different machine learning techniques for phenotypic trait prediction.: "The PICA framework facilitates application and comparison of different machine learning techniques for phenotypic trait prediction."
- [abstract] We have improved and extended PICA's support vector machine plug-in and suggest its applicability to large-scale genome databases and incomplete genome sequences.: "We have improved and extended PICA's support vector machine plug-in and suggest its applicability to large-scale genome databases and incomplete genome sequences."
- [abstract] Most of the traits can be reliably predicted in only 60-70% complete genomes.: "Most of the traits can be reliably predicted in only 60-70% complete genomes."
- [readme] End-to-end Microbial Phenotypic Trait Prediction.: [evidence span withheld — anchor status: external_doc]

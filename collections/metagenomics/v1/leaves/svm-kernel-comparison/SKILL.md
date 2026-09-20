---
name: svm-kernel-comparison
description: Use when choosing an SVM kernel for phenotype prediction from protein-family
  (COG) presence/absence feature matrices, or when verifying that a linear kernel
  matches or beats RBF, sigmoid, and polynomial alternatives under repeated cross-validation.
when_to_use_negative:
- The input is not a genotype (COG presence/absence) feature matrix paired with trait
  labels — e.g. raw sequence or metagenomic bin data requiring prior trait-model prediction
  rather than SVM training/comparison
- The goal is de novo phenotype prediction on new genomes rather than benchmarking
  kernel choices — use a trained phenotrex model (e.g. `phenotrex predict`) instead
- Training data are insufficient for the trait — the article notes performance for
  further ecologically important traits is pending availability of sufficient training
  data
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: PICA
  role: Framework used to train and cross-validate SVM-based phenotype prediction
    models on COG feature matrices and compare kernels
  repo: https://github.com/univieCUBE/PICA
- name: PICA support vector machine plug-in
  role: Extended plug-in providing the SVM classifier interface through which the
    linear, RBF, sigmoid, and polynomial kernels are evaluated
  repo: https://github.com/univieCUBE/PICA
- name: libSVM
  role: Backend SVM library invoked by the PICA plug-in to fit and evaluate each kernel
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1186/1471-2105-16-s14-s1
    title: Prediction of microbial phenotypes based on comparative genomics
  source_content_hash: sha256:74e2b91d81b4954e20273a0a7eb595f9b11b2476335b2e1fb9184bc714ebedb9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/univieCUBE/PICA
  tools:
  - PICA
  - PICA support vector machine plug-in
  - libSVM
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# svm-kernel-comparison

## Summary

Benchmark and compare SVM kernel choices (linear vs. RBF, sigmoid, polynomial) for phenotypic trait prediction from COG presence/absence profiles using PICA's libSVM-backed plug-in, as in the Feldbauer et al. 2015 evaluation of PICA.

## When to use

Use when choosing an SVM kernel for phenotype prediction from protein-family (COG) presence/absence feature matrices, or when verifying that a linear kernel matches or beats RBF, sigmoid, and polynomial alternatives under repeated cross-validation.

## When NOT to use

- The input is not a genotype (COG presence/absence) feature matrix paired with trait labels — e.g. raw sequence or metagenomic bin data requiring prior trait-model prediction rather than SVM training/comparison
- The goal is de novo phenotype prediction on new genomes rather than benchmarking kernel choices — use a trained phenotrex model (e.g. `phenotrex predict`) instead
- Training data are insufficient for the trait — the article notes performance for further ecologically important traits is pending availability of sufficient training data

## Inputs

- COG presence/absence feature matrix (genomes x protein families) from the PICA distribution
- Binary trait labels for the phenotypic traits being predicted (e.g. the ten example traits)
- PICA SVM plug-in configuration with libSVM backend and kernel choice (linear, RBF, sigmoid, polynomial)

## Outputs

- Per-trait kernel-comparison table of mean balanced accuracy (e.g. kernel_balanced_accuracy.csv, one row per trait, one column per kernel)
- Balanced-accuracy scores per cross-validation replicate for each trait-kernel combination

## How to apply

Load the COG presence/absence feature matrices and trait labels for the traits of interest from the PICA distribution (github.com/univieCUBE/PICA). For each trait, train SVMs via PICA's support vector machine plug-in (libSVM backend) with the linear kernel and evaluate with 5-fold cross-validation repeated 10 times, recording balanced accuracy per replicate. Repeat the identical 5-fold x 10-replicate protocol for the RBF, sigmoid, and polynomial kernels so kernel comparisons are matched on data splits. Aggregate replicate scores into a mean balanced accuracy per trait per kernel and write the comparison table (one row per trait, one column per kernel, e.g. kernel_balanced_accuracy.csv). Validate by checking that the linear kernel's mean balanced accuracy is greater than or equal to each alternative kernel's for every trait, mirroring the Figure 3 outcome where the linear kernel performed better than or at least as well as RBF, sigmoid, and polynomial for all ten traits.

## Related tools

- **PICA** (Framework used to train and cross-validate SVM-based phenotype prediction models on COG feature matrices and compare kernels) — https://github.com/univieCUBE/PICA
- **PICA support vector machine plug-in** (Extended plug-in providing the SVM classifier interface through which the linear, RBF, sigmoid, and polynomial kernels are evaluated) — https://github.com/univieCUBE/PICA
- **libSVM** (Backend SVM library invoked by the PICA plug-in to fit and evaluate each kernel)

## Examples

```
phenotrex predict --classifier MY_TRAIT.pkl genome.fna > predictions.tsv
```

## Evaluation signals

- The mean balanced accuracy of the linear kernel is >= that of RBF, sigmoid, and polynomial kernels for every trait evaluated (as reported for all ten traits, Figure 3)
- The comparison table has exactly one row per trait and one column per kernel, with mean balanced accuracy aggregated over the 5-fold x 10-replicate cross-validation
- All kernels are evaluated on identical replicated cross-validation splits (5-fold, repeated 10 times) so scores are directly comparable
- Balanced accuracy values fall within the plausible [0, 1] range and per-replicate scores are retained before aggregation
- Scaling behavior remains computationally feasible at large matrix sizes (e.g. 5,000 genomes x ~200,000 COGs), consistent with the paper's feasibility benchmark

## Limitations

- Reliable trait prediction requires at least 60-70% genome completeness; predictions in less complete bins may be unreliable
- Performance for further ecologically important traits is pending availability of sufficient training data (ongoing work)
- PICA has been superseded by the phenotrex package, which builds upon the SciPy stack (scikit-learn, pandas), adds new classifiers such as XGboost, and supports Python 3.7+ via pip

## Evidence

- [abstract] PICA framework role: "The PICA framework facilitates application and comparison of different machine learning techniques for phenotypic trait prediction."
- [abstract] Extended SVM plug-in applicability: "We have improved and extended PICA's support vector machine plug-in and suggest its applicability to large-scale genome databases and incomplete genome sequences."
- [intro] libSVM plug-in: "It includes plug-ins for CPAR and libSVM [19] among others."
- [readme] phenotrex prediction CLI: [evidence span withheld — anchor status: external_doc]
- [discussion] Genome completeness limitation: "Most of the traits can be reliably predicted in only 60-70% complete genomes, which allows reasonable predictions in genome bins from metagenomes."

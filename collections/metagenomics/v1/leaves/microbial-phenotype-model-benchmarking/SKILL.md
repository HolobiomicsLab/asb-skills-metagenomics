---
name: microbial-phenotype-model-benchmarking
description: 'Use when a comparative-genomics phenotype prediction task must select
  an SVM kernel: input is a COG presence/absence feature matrix with binary trait
  labels, and the goal is kernel comparison via balanced accuracy to justify a default
  kernel choice.'
when_to_use_negative:
- Input is already a trained model or single genome sequence to annotate — use phenotrex/PhenDB-style
  prediction instead of kernel benchmarking
- The task requires predicting traits in only 60-70% complete genomes for which no
  sufficient training data exist; the paper notes performance for further ecologically
  important traits is pending availability of sufficient training data
- Non-SVM classifiers (e.g., CPAR or newer classifiers like XGBoost in phenotrex)
  are the target of the comparison — this skill benchmarks only libSVM kernels within
  PICA
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: PICA
  role: Framework providing the phenotype-prediction workflow and the COG presence/absence
    training data used for the kernel comparison
  repo: github.com/univieCUBE/PICA
- name: PICA support vector machine plug-in
  role: Executes the SVM training and cross-validation with switchable kernels (linear,
    RBF, sigmoid, polynomial)
  repo: github.com/univieCUBE/PICA
- name: libSVM
  role: Underlying SVM backend invoked by PICA's support vector machine plug-in
- name: phenotrex
  role: Successor package to PICA with an improved SciPy/scikit-learn-based stack
    for microbial phenotypic trait prediction
  repo: univieCUBE/phenotrex
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/1471-2105-16-s14-s1
    title: Prediction of microbial phenotypes based on comparative genomics
  source_content_hash: sha256:954846a314a60bdda8370e14d6c1734c7df4f465ef32a26bf06a02e415835d06
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PICA
  - PICA support vector machine plug-in
  - libSVM
  - phenotrex
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# microbial-phenotype-model-benchmarking

## Summary

Benchmarks SVM kernel choices (linear, RBF, sigmoid, polynomial) for predicting microbial phenotypic traits from COG presence/absence feature matrices using PICA's libSVM plug-in, verifying whether the default linear kernel matches or outperforms alternatives under repeated 5-fold cross-validation.

## When to use

Use when a comparative-genomics phenotype prediction task must select an SVM kernel: input is a COG presence/absence feature matrix with binary trait labels, and the goal is kernel comparison via balanced accuracy to justify a default kernel choice.

## When NOT to use

- Input is already a trained model or single genome sequence to annotate — use phenotrex/PhenDB-style prediction instead of kernel benchmarking
- The task requires predicting traits in only 60-70% complete genomes for which no sufficient training data exist; the paper notes performance for further ecologically important traits is pending availability of sufficient training data
- Non-SVM classifiers (e.g., CPAR or newer classifiers like XGBoost in phenotrex) are the target of the comparison — this skill benchmarks only libSVM kernels within PICA

## Inputs

- COG (protein family) presence/absence feature matrices for microbial genomes
- Binary phenotypic trait labels for the ten example traits (e.g., from the PICA distribution)
- PICA distribution including its libSVM plug-in

## Outputs

- kernel_balanced_accuracy.csv: per-trait kernel-comparison table (one row per trait, one column per kernel)
- Mean balanced accuracy per trait and SVM kernel (linear, RBF, sigmoid, polynomial)

## How to apply

Load the protein-family (COG) presence/absence feature matrices and binary trait labels for the target phenotypic traits from the PICA distribution (github.com/univieCUBE/PICA). For each trait, train a linear-kernel SVM through PICA's libSVM plug-in and evaluate with 5-fold cross-validation repeated 10 times, recording balanced accuracy per replicate; then repeat the identical 5-fold x 10-replicate protocol with the RBF, sigmoid, and polynomial kernels. Aggregate replicate scores into a mean balanced accuracy per trait and kernel, and write a per-trait comparison table (one row per trait, one column per kernel) to kernel_balanced_accuracy.csv. Accept the benchmark only if the linear kernel's balanced accuracy is greater than or equal to every alternative kernel for each trait, mirroring the paper's Figure 3 result that the linear kernel performed better than or at least as well as RBF, sigmoid, and polynomial for all ten traits.

## Related tools

- **PICA** (Framework providing the phenotype-prediction workflow and the COG presence/absence training data used for the kernel comparison) — github.com/univieCUBE/PICA
- **PICA support vector machine plug-in** (Executes the SVM training and cross-validation with switchable kernels (linear, RBF, sigmoid, polynomial)) — github.com/univieCUBE/PICA
- **libSVM** (Underlying SVM backend invoked by PICA's support vector machine plug-in)
- **phenotrex** (Successor package to PICA with an improved SciPy/scikit-learn-based stack for microbial phenotypic trait prediction) — univieCUBE/phenotrex

## Examples

```
phenotrex predict --classifier MY_TRAIT.pkl genome.fna > predictions.tsv
```

## Evaluation signals

- kernel_balanced_accuracy.csv contains exactly one row per trait and one column per kernel (linear, RBF, sigmoid, polynomial)
- For every trait, linear-kernel mean balanced accuracy >= RBF, sigmoid, and polynomial mean balanced accuracy (Figure 3 invariant)
- Balanced accuracy is computed per replicate under 5-fold cross-validation repeated 10 times (i.e., 50 replicate scores per trait-kernel pair before aggregation)
- The paper reports that the linear kernel performed better than or at least as well as RBF, sigmoid, and polynomial for all ten phenotypic traits — any trait violating this should trigger re-examination
- Per-replicate balanced accuracies fall in the [0, 1] range and means are consistent with the replicate-level records

## Limitations

- Reliable trait prediction requires at least 60-70% genome completeness; most traits can be reliably predicted only in 60-70% complete genomes
- Performance for further ecologically important traits is pending availability of sufficient training data (ongoing work)
- The comparison is restricted to SVM kernels via libSVM within PICA; the PICA README notes the successor package phenotrex with new classifiers (e.g., XGBoost), so the kernel ranking may not transfer to other model families

## Evidence

- [abstract] linear kernel matches or beats alternatives: "We have improved and extended PICA's support vector machine plug-in and suggest its applicability to large-scale genome databases and incomplete genome sequences."
- [full_text] task protocol: [evidence span withheld — anchor status: not_found]
- [readme] successor package: "Please checkout the new `phenotrex` package, which succeeds PICA"
- [readme] tooling basis: "Builds upon the battle-tested SciPy stack (`scikit-learn`, `pandas`, etc.)"

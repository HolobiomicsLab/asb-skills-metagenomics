---
name: repeated-cross-validation-accuracy-estimation
description: Use when a binary phenotypic trait model (e.g. obligate intracellular
  vs free-living) trained on COG presence/absence vectors must be validated with a
  robust, replicate-averaged accuracy estimate rather than a single train/test split.
when_to_use_negative:
- The goal is prediction on new metagenomic bins with an already-trained model — use
  phenotrex predict rather than re-running cross-validation
- Genomes fail the marker-COG completeness screen (fewer than 39/40 marker COGs);
  the paper's traits are only reliably predictable in ~60-70% complete genomes, so
  incomplete inputs should not enter training/evaluation
- The analysis needs method comparison across different ML algorithms rather than
  replicate accuracy estimation for one classifier
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0196
tools:
- name: PICA
  role: Framework hosting the trait-prediction workflow and the support vector machine
    plug-in used to train and cross-validate the phenotype model
  repo: https://github.com/univieCUBE/PICA
- name: PICA support vector machine plug-in
  role: SVM implementation applied to the COG presence/absence matrix for trait classification
    and fold-wise prediction
  repo: https://github.com/univieCUBE/PICA
- name: libSVM
  role: Backend SVM library invoked by the PICA plug-in during classifier training
    and cross-validation
- name: phenotrex
  role: Successor package of PICA for training, evaluation and prediction of phenotypic
    trait models on genome/metagenomic-bin data
  repo: https://github.com/univieCUBE/phenotrex
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/1471-2105-16-s14-s1
    title: Prediction of microbial phenotypes based on comparative genomics
  source_content_hash: sha256:5be5a7bb230ae83b409f42f63d64257b4c5f602ef81fb4023434d8283856128a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/univieCUBE/PICA
  tools:
  - PICA
  - PICA support vector machine plug-in
  - libSVM
  - phenotrex
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8636363636363636
n_claims: 22
---

# Repeated cross-validation accuracy estimation

## Summary

Estimates generalization accuracy of an SVM-based phenotypic trait classifier by aggregating balanced accuracy over 5-fold cross-validation repeated 10 times (e.g. PICA/libSVM on COG presence/absence matrices), reporting mean ± standard deviation and the per-fold misclassification pattern.

## When to use

Use when a binary phenotypic trait model (e.g. obligate intracellular vs free-living) trained on COG presence/absence vectors must be validated with a robust, replicate-averaged accuracy estimate rather than a single train/test split.

## When NOT to use

- The goal is prediction on new metagenomic bins with an already-trained model — use phenotrex predict rather than re-running cross-validation
- Genomes fail the marker-COG completeness screen (fewer than 39/40 marker COGs); the paper's traits are only reliably predictable in ~60-70% complete genomes, so incomplete inputs should not enter training/evaluation
- The analysis needs method comparison across different ML algorithms rather than replicate accuracy estimation for one classifier

## Inputs

- COG presence/absence feature matrix for complete bacterial genomes (e.g. 97 genomes × 30,455 COGs)
- Phenotypic trait labels per genome from a curated annotation table (e.g. Additional File 1 Table S1: obligate intracellular / facultative / free-living)
- Marker-COG completeness screen results (39/40 marker COGs threshold)

## Outputs

- Per-fold predicted label for every genome across 10 replicates (e.g. cv_results.csv)
- Mean balanced accuracy ± standard deviation across replicates (reference: 0.997 ± 0.010)
- List of misclassified genomes with their misclassification frequency across folds

## How to apply

Load the COG presence/absence feature matrix (in the reference case: 97 complete bacterial genomes × 30,455 COGs, labelled 43 obligate intracellular, 6 facultative, 48 free-living) and attach trait labels from the annotation table. Apply the marker-COG completeness filter (39/40 marker COGs) so only passing genomes enter training. Train the classifier with the PICA support vector machine plug-in backed by libSVM, then run 5-fold cross-validation with 10 replicates, recording balanced accuracy and each genome's predicted label in every fold. Aggregate the per-fold balanced accuracies into mean ± standard deviation (reference result: 0.997 ± 0.010) and tabulate genomes that are consistently misclassified (reference: Cand. Pelagibacter sp. IMCC9063 wrong in 3 of 10 folds). Write per-fold predictions and the summary metric (e.g. cv_results.csv) so the run is auditable and reproducible.

## Related tools

- **PICA** (Framework hosting the trait-prediction workflow and the support vector machine plug-in used to train and cross-validate the phenotype model) — https://github.com/univieCUBE/PICA
- **PICA support vector machine plug-in** (SVM implementation applied to the COG presence/absence matrix for trait classification and fold-wise prediction) — https://github.com/univieCUBE/PICA
- **libSVM** (Backend SVM library invoked by the PICA plug-in during classifier training and cross-validation)
- **phenotrex** (Successor package of PICA for training, evaluation and prediction of phenotypic trait models on genome/metagenomic-bin data) — https://github.com/univieCUBE/phenotrex

## Examples

```
$ phenotrex predict --classifier MY_TRAIT.pkl genome.fna > predictions.tsv
```

## Evaluation signals

- Mean balanced accuracy across the 10 replicates matches the reference value 0.997 ± 0.010
- All positives and negatives are predicted correctly in all folds except the documented outlier (Cand. Pelagibacter sp. IMCC9063 misclassified as obligate intracellular in 3 of 10 folds)
- cv_results.csv contains one predicted label per genome per fold (97 genomes × 5 folds × 10 replicates) plus the summary metric
- Every genome entering the training set passes the 39/40 marker-COG completeness filter
- Per-fold balanced accuracy is computed with balanced class weighting given the 43/6/48 label distribution

## Limitations

- Reliable trait prediction requires at least 60-70% genome completeness; models trained this way should not be applied to highly fragmented assemblies
- Performance for further ecologically important traits is pending availability of sufficient training data
- Accuracy estimates are trait- and database-dependent; the paper notes the predictive power is stable and not perturbed by the rapid growth of genome databases, but the misclassification of Cand. Pelagibacter sp. IMCC9063 shows genome-reduction traits can confuse the model for free-living species with reduced genomes

## Evidence

- [abstract] PICA facilitates trait prediction: "The PICA framework facilitates application and comparison of different machine learning techniques for phenotypic trait prediction."
- [abstract] SVM plug-in extension: "We have improved and extended PICA's support vector machine plug-in and suggest its applicability to large-scale genome databases and incomplete genome sequences."
- [discussion] Completeness limitation: "Most of the traits can be reliably predicted in only 60-70% complete genomes, which allows reasonable predictions in genome bins from metagenomes."
- [readme] phenotrex prediction output format: [evidence span withheld — anchor status: external_doc]

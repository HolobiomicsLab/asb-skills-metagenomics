---
name: cog-feature-matrix-handling
description: 'Use when training or validating SVM phenotype predictors from comparative-genomics
  data: inputs are complete/partial bacterial genomes or COG presence/absence tables
  plus trait labels, and features must be assembled and completeness-screened.'
when_to_use_negative:
- Input is already a filtered, labeled feature table ready for the PICA/libSVM classifier
  — re-assembling matrices would be redundant.
- 'The trait cannot be learned from generic COG content: the article notes deeper
  interpretation requires additional specific models beyond the generic COG-based
  first layer.'
- Genome completeness is far below the 60-70% level at which most traits remain reliably
  predictable and the 39/40 marker-COG screen fails.
edam_operation: http://edamontology.org/operation_2995
tools:
- name: PICA
  role: Framework for loading COG feature matrices, attaching trait labels, and applying
    the marker-COG completeness filter before classification
  repo: github.com/univieCUBE/PICA
- name: PICA support vector machine plug-in
  role: Consumes the filtered COG presence/absence matrix to train and cross-validate
    SVM phenotype models via libSVM
  repo: github.com/univieCUBE/PICA
- name: libSVM
  role: SVM backend used by the PICA plug-in on the COG feature matrix
- name: phenotrex
  role: Successor package to PICA for end-to-end microbial phenotypic trait prediction
    on genome FASTAs
  repo: univieCUBE/phenotrex
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1186/1471-2105-16-s14-s1
    title: Prediction of microbial phenotypes based on comparative genomics
  source_content_hash: sha256:48ab0f8e65c298363df671223a2fdb0e84b3a989afe41d44a2e3a092d0520223
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PICA
  - PICA support vector machine plug-in
  - libSVM
  - phenotrex
  edam_operation: http://edamontology.org/operation_2995
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# COG feature matrix handling

## Summary

Build and prepare COG presence/absence feature matrices for SVM-based phenotypic trait prediction in the PICA framework, including applying marker-COG genome completeness screening before model training. The matrix rows are genomes and columns are COGs (e.g. 97 complete bacterial genomes against 30,455 COGs), with trait labels attached per genome.

## When to use

Use when training or validating SVM phenotype predictors from comparative-genomics data: inputs are complete/partial bacterial genomes or COG presence/absence tables plus trait labels, and features must be assembled and completeness-screened.

## When NOT to use

- Input is already a filtered, labeled feature table ready for the PICA/libSVM classifier — re-assembling matrices would be redundant.
- The trait cannot be learned from generic COG content: the article notes deeper interpretation requires additional specific models beyond the generic COG-based first layer.
- Genome completeness is far below the 60-70% level at which most traits remain reliably predictable and the 39/40 marker-COG screen fails.

## Inputs

- COG presence/absence feature matrix (genomes x COGs, e.g. 97 genomes x 30,455 COGs)
- Trait label table per genome (e.g. Additional File 1 Table S1: obligate intracellular / facultative / free-living)
- Set of 40 marker COGs used for genome completeness screening (threshold: >=39/40 present)

## Outputs

- Completeness-filtered feature matrix with attached trait labels ready for SVM training
- Cross-validation results table (cv_results.csv) with per-fold predictions and mean +/- SD balanced accuracy (e.g. 0.997 +/- 0.010 for intracellular lifestyle)

## How to apply

Load the COG presence/absence feature matrix (rows = genomes, columns = COGs; e.g. the 97-genome x 30,455-COG matrix used for intracellular lifestyle) and attach binary trait labels from the reference table (Additional File 1 Table S1: 43 obligate intracellular, 6 facultative, 48 free-living). Apply the marker-COG completeness filter — requiring at least 39 of 40 marker COGs — so that only genomes passing the completeness screen enter the training set; this is what enables reliable prediction in 60-70% complete genomes. Feed the filtered matrix into the PICA support vector machine plug-in backed by libSVM, and evaluate with 5-fold cross-validation repeated 10 times, recording balanced accuracy per replicate. PICA has been superseded by the phenotrex package (pip installable, built on scikit-learn), which the README recommends for new work.

## Related tools

- **PICA** (Framework for loading COG feature matrices, attaching trait labels, and applying the marker-COG completeness filter before classification) — github.com/univieCUBE/PICA
- **PICA support vector machine plug-in** (Consumes the filtered COG presence/absence matrix to train and cross-validate SVM phenotype models via libSVM) — github.com/univieCUBE/PICA
- **libSVM** (SVM backend used by the PICA plug-in on the COG feature matrix)
- **phenotrex** (Successor package to PICA for end-to-end microbial phenotypic trait prediction on genome FASTAs) — univieCUBE/phenotrex

## Examples

```
pip install phenotrex[fasta]; phenotrex predict --classifier MY_TRAIT.pkl genome.fna > predictions.tsv
```

## Evaluation signals

- All training-set genomes pass the marker-COG completeness screen (>=39 of 40 marker COGs present).
- Feature matrix dimensions match the source dataset (e.g. 97 genomes covering 30,455 COGs; label counts 43 obligate intracellular / 6 facultative / 48 free-living).
- 5-fold cross-validation repeated 10 times yields mean balanced accuracy consistent with the reported 0.997 +/- 0.010 for intracellular lifestyle.
- Only Cand. Pelagibacter sp. IMCC9063 is inconsistently misclassified (in 3 of 10 folds), reproducing the paper's error pattern.
- Output cv_results.csv contains one row per fold with predicted labels and balanced accuracy.

## Limitations

- Most traits can be reliably predicted only in 60-70% complete genomes; the marker-COG filter (39/40) gates which genomes may be used.
- Performance for further ecologically important traits is pending availability of sufficient training data (ongoing work in the article).
- Generic COG-based methods are described as a first layer only; deeper interpretation requires additional specific models.
- PICA itself is deprecated in favor of phenotrex, which changes the exact tooling for this workflow.

## Evidence

- [abstract] We have improved and extended PICA's support vector machine plug-in and suggest its applicability to large-scale genome databases and incomplete genome sequences.: "We have improved and extended PICA's support vector machine plug-in and suggest its applicability to large-scale genome databases and incomplete genome sequences."
- [intro] It includes plug-ins for CPAR and libSVM [19] among others.: "It includes plug-ins for CPAR and libSVM [19] among others."
- [discussion] Most of the traits can be reliably predicted in only 60-70% complete genomes, which allows reasonable predictions in genome bins from metagenomes.: "Most of the traits can be reliably predicted in only 60-70% complete genomes, which allows reasonable predictions in genome bins from metagenomes."
- [readme] End-to-end Microbial Phenotypic Trait Prediction.: [evidence span withheld — anchor status: external_doc]

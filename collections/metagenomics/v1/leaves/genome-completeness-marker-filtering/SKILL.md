---
name: genome-completeness-marker-filtering
description: Use when a COG presence/absence feature matrix (e.g. 97 genomes x 30,455
  COGs) is being prepared for trait-prediction training and each genome must pass
  a marker-COG completeness screen before model fitting.
when_to_use_negative:
- Input is already a feature matrix whose genomes have all passed a 39/40 marker-COG
  completeness screen — re-filtering adds nothing.
- Input is raw sequencing reads or unannotated metagenomic contigs without COG annotation,
  so marker-COG presence cannot be tabulated.
- Goal is predicting traits on incomplete genome bins rather than building a training
  set — the article notes reliable prediction needs only 60-70% completeness, so a
  39/40 hard filter for prediction-time bins is unnecessarily strict.
edam_operation: http://edamontology.org/operation_3695
tools:
- name: PICA
  role: Framework whose training pipeline consumes the completeness-filtered COG matrix
    for phenotype trait prediction
  repo: https://github.com/univieCUBE/PICA
- name: PICA support vector machine plug-in
  role: SVM plug-in that trains on the filtered feature matrix for obligate intracellular
    lifestyle prediction
  repo: https://github.com/univieCUBE/PICA
- name: libSVM
  role: Backend classifier used by the PICA SVM plug-in after completeness filtering
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/1471-2105-16-s14-s1
    title: Prediction of microbial phenotypes based on comparative genomics
  source_content_hash: sha256:67fd327e6c27f00571a0a78d8953879ced6c2e494f13bde6cef76bf5034d217f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/univieCUBE/PICA
  tools:
  - PICA
  - PICA support vector machine plug-in
  - libSVM
  edam_operation: http://edamontology.org/operation_3695
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# genome-completeness-marker-filtering

## Summary

Screens complete bacterial genomes for comparative-genomics phenotypic trait training by requiring near-full presence of a panel of marker COGs (39/40), so that only adequately complete genomes enter the PICA SVM training set.

## When to use

Use when a COG presence/absence feature matrix (e.g. 97 genomes x 30,455 COGs) is being prepared for trait-prediction training and each genome must pass a marker-COG completeness screen before model fitting.

## When NOT to use

- Input is already a feature matrix whose genomes have all passed a 39/40 marker-COG completeness screen — re-filtering adds nothing.
- Input is raw sequencing reads or unannotated metagenomic contigs without COG annotation, so marker-COG presence cannot be tabulated.
- Goal is predicting traits on incomplete genome bins rather than building a training set — the article notes reliable prediction needs only 60-70% completeness, so a 39/40 hard filter for prediction-time bins is unnecessarily strict.

## Inputs

- COG presence/absence feature matrix for complete bacterial genomes (e.g. 97 genomes x 30,455 COGs)
- Trait labels from Additional File 1 Table S1 (obligate intracellular / facultative / free-living)
- Marker-COG completeness panel (40 marker COGs with a 39/40 presence threshold)

## Outputs

- Filtered training matrix containing only genomes passing the 39/40 marker-COG completeness screen
- Set of excluded genomes failing the completeness filter
- Downstream per-fold CV predictions and summary metrics (e.g. cv_results.csv with balanced accuracy 0.997 +/- 0.010)

## How to apply

Load the COG presence/absence feature matrix for the 97 complete bacterial genomes (43 obligate intracellular, 6 facultative, 48 free-living; 30,455 COGs) and attach trait labels from Additional File 1 Table S1. Apply the marker-COG completeness filter requiring 39 of 40 marker COGs to be present; genomes failing this screen are excluded from the training set. Pass only the filtered matrix to the PICA support vector machine plug-in (libSVM) for 5-fold cross-validation with 10 replicates. The rationale is that incomplete or misannotated genomes distort presence/absence signals that drive traits determined by gene absence in reduced genomes, so the completeness screen protects label quality before classification.

## Related tools

- **PICA** (Framework whose training pipeline consumes the completeness-filtered COG matrix for phenotype trait prediction) — https://github.com/univieCUBE/PICA
- **PICA support vector machine plug-in** (SVM plug-in that trains on the filtered feature matrix for obligate intracellular lifestyle prediction) — https://github.com/univieCUBE/PICA
- **libSVM** (Backend classifier used by the PICA SVM plug-in after completeness filtering)

## Evaluation signals

- Every genome retained in the training set has >= 39 of the 40 marker COGs present in the feature matrix.
- Filtered class composition matches the reference set: 43 obligate intracellular, 6 facultative, 48 free-living genomes.
- Downstream 5-fold CV with 10 replicates reproduces balanced accuracy 0.997 +/- 0.010, validating that filtering did not distort labels.
- The known misclassification pattern is preserved: Cand. Pelagibacter sp. IMCC9063 misclassified as obligate intracellular in 3 of 10 folds.
- Excluded genomes are logged with the count of missing marker COGs so filter decisions are auditable.

## Limitations

- Most traits can be reliably predicted only in 60-70% complete genomes; below this, trait predictions from genome bins are unreliable.
- The 39/40 marker-COG threshold was calibrated for complete genomes; applying it unchanged to draft metagenome bins may over-filter.
- Performance for further ecologically important traits is pending availability of sufficient training data.

## Evidence

- [abstract] framework-vs-completeness: "The PICA framework facilitates application and comparison of different machine learning techniques for phenotypic trait prediction."
- [discussion] completeness-threshold: "Most of the traits can be reliably predicted in only 60-70% complete genomes, which allows reasonable predictions in genome bins from metagenomes."
- [abstract] reduced-genome-traits: "Thereby we could demonstrate that also independently evolved phenotypic traits, characterized by genome reduction, can be reliably predicted based on comparative genomics."
- [readme] readme-scope: [evidence span withheld — anchor status: external_doc]

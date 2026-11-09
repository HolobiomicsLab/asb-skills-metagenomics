---
name: intracellular-lifestyle-classification
description: Use when predicting the obligate intracellular lifestyle of bacterial
  genomes from comparative-genomics COG presence/absence profiles, or reproducing
  the 0.997 ± 0.010 accuracy benchmark on complete genomes.
when_to_use_negative:
- 'Input genomes are far below the reliability threshold: most traits can be reliably
  predicted only in 60-70% complete genomes, so highly fragmentary bins may not support
  this model'
- The target trait is a presence-based phenotype not driven by genome reduction, for
  which a different phenotrex/PICA trait model should be trained instead
- Sufficient labeled training data are unavailable for further ecologically important
  traits, where performance is still pending
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: PICA
  role: Framework hosting the trait-prediction workflow; trains and evaluates phenotype
    models from COG feature matrices
  repo: github.com/univieCUBE/PICA
- name: PICA support vector machine plug-in
  role: SVM-based classifier plug-in used to train the obligate-intracellular model
    within PICA
  repo: github.com/univieCUBE/PICA
- name: libSVM
  role: SVM backend invoked by the PICA plug-in for classification
- name: phenotrex
  role: Successor package of PICA for end-to-end microbial phenotypic trait prediction,
    usable to retrain or apply trait models
  repo: univieCUBE/phenotrex
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/1471-2105-16-s14-s1
    title: Prediction of microbial phenotypes based on comparative genomics
  source_content_hash: sha256:481db5edad9ca00d7d5b48b5192c5a37f188f71c2698b039d550c9e36b182a01
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PICA
  - PICA support vector machine plug-in
  - libSVM
  - phenotrex
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# intracellular-lifestyle-classification

## Summary

Train and cross-validate an SVM classifier (PICA plug-in backed by libSVM) on a COG presence/absence feature matrix to distinguish obligate intracellular bacteria from free-living/facultative species, exploiting the fact that this trait is determined by gene absence in reduced genomes rather than gene presence.

## When to use

Use when predicting the obligate intracellular lifestyle of bacterial genomes from comparative-genomics COG presence/absence profiles, or reproducing the 0.997 ± 0.010 accuracy benchmark on complete genomes.

## When NOT to use

- Input genomes are far below the reliability threshold: most traits can be reliably predicted only in 60-70% complete genomes, so highly fragmentary bins may not support this model
- The target trait is a presence-based phenotype not driven by genome reduction, for which a different phenotrex/PICA trait model should be trained instead
- Sufficient labeled training data are unavailable for further ecologically important traits, where performance is still pending

## Inputs

- COG presence/absence feature matrix for 97 complete bacterial genomes (30,455 COGs)
- Trait labels from Additional File 1 Table S1 (43 obligate intracellular, 6 facultative, 48 free-living)

## Outputs

- cv_results.csv with per-fold predicted labels for every genome and the summary balanced accuracy metric
- Mean cross-validated balanced accuracy (expected 0.997 ± 0.010) and list of consistently misclassified genomes

## How to apply

Load the COG presence/absence feature matrix for the 97 complete bacterial genomes (43 obligate intracellular, 6 facultative, 48 free-living; 30,455 COGs) and attach trait labels. Apply the marker-COG completeness filter (39/40 marker COGs present) so only genomes passing the completeness screen enter training. Train the classifier with the PICA support vector machine plug-in backed by libSVM on the filtered matrix, then run 5-fold cross-validation with 10 replicates, recording balanced accuracy and per-genome predicted labels in each fold. Aggregate per-fold balanced accuracies into a mean ± standard deviation and tabulate consistently misclassified genomes (e.g. Cand. Pelagibacter sp. IMCC9063). The rationale is that independently evolved traits characterized by genome reduction, such as obligate intracellular lifestyle, can be reliably predicted from comparative genomics alone.

## Related tools

- **PICA** (Framework hosting the trait-prediction workflow; trains and evaluates phenotype models from COG feature matrices) — github.com/univieCUBE/PICA
- **PICA support vector machine plug-in** (SVM-based classifier plug-in used to train the obligate-intracellular model within PICA) — github.com/univieCUBE/PICA
- **libSVM** (SVM backend invoked by the PICA plug-in for classification)
- **phenotrex** (Successor package of PICA for end-to-end microbial phenotypic trait prediction, usable to retrain or apply trait models) — univieCUBE/phenotrex

## Examples

```
phenotrex predict --classifier intracellular.pkl genome.fna > predictions.tsv
```

## Evaluation signals

- Mean balanced accuracy across 10 replicates of 5-fold cross-validation equals 0.997 ± 0.010
- All positives and negatives are predicted correctly in all folds except Cand. Pelagibacter sp. IMCC9063, which is misclassified as obligate intracellular in exactly 3 of 10 folds
- All training genomes pass the marker-COG completeness screen (39/40 marker COGs present)
- cv_results.csv contains a predicted label for every genome in every fold plus the aggregated summary metric

## Limitations

- Reliable trait prediction requires at least 60-70% genome completeness, which permits reasonable predictions on metagenomic genome bins but not more fragmentary sequences
- Performance for further ecologically important traits is pending availability of sufficient training data
- Edge case: Cand. Pelagibacter sp. IMCC9063 (free-living) is misclassified as obligate intracellular in 3 of 10 folds, showing borderline genomes near the trait boundary remain error-prone

## Evidence

- [abstract] PICA SVM workflow: "We have improved and extended PICA's support vector machine plug-in and suggest its applicability to large-scale genome databases and incomplete genome sequences."
- [abstract] independently evolved reduction traits: "Thereby we could demonstrate that also independently evolved phenotypic traits, characterized by genome reduction, can be reliably predicted based on comparative genomics."
- [discussion] completeness limitation: "Most of the traits can be reliably predicted in only 60-70% complete genomes, which allows reasonable predictions in genome bins from metagenomes."
- [readme] phenotrex CLI: [evidence span withheld — anchor status: external_doc]

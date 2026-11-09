---
name: svm-feature-weight-ranking
description: Use when a trained linear SVM phenotype model (e.g. PICA intracellular
  model on 97 labelled species) must be inspected to determine whether a trait is
  predicted by gene presence or absence, and to compare the ranking against a published
  table.
when_to_use_negative:
- The model is nonlinear (non-linear kernel), so per-feature weights do not exist
  or are not interpretable as signed presence/absence predictors.
- Input is raw genome data rather than a trained classifier — this skill interprets
  an already-trained PICA/libSVM model, it does not train one or make phenotype predictions
  for new genomes.
- Genome completeness is far below the ~60-70% threshold at which traits are reliably
  predicted, since features derived from fragmentary genomes may not support stable
  rankings.
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: PICA
  role: Framework whose trained SVM phenotype model supplies the weight vector to
    be ranked; the successor package is phenotrex
  repo: https://github.com/univieCUBE/PICA
- name: PICA support vector machine plug-in
  role: Plug-in used to train the intracellular model whose weights are extracted
    and ranked
- name: libSVM
  role: Linear-kernel SVM backend used by the PICA plug-in; linear weights are required
    for signed feature ranking
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/1471-2105-16-s14-s1
    title: Prediction of microbial phenotypes based on comparative genomics
  source_content_hash: sha256:5eeb71c32cef270a44ab245ebd808d8688a51f1078802c07171aea799d9439d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/univieCUBE/PICA
  tools:
  - PICA
  - PICA support vector machine plug-in
  - libSVM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# SVM feature weight ranking for phenotype model interpretation

## Summary

Reproduce and interpret a signed COG feature ranking from a trained PICA/libSVM linear SVM phenotype model by extracting the raw SVM weight vector and ranking features by absolute weight, with the sign encoding whether gene presence or absence predicts the trait (here, the obligate intracellular phenotype).

## When to use

Use when a trained linear SVM phenotype model (e.g. PICA intracellular model on 97 labelled species) must be inspected to determine whether a trait is predicted by gene presence or absence, and to compare the ranking against a published table.

## When NOT to use

- The model is nonlinear (non-linear kernel), so per-feature weights do not exist or are not interpretable as signed presence/absence predictors.
- Input is raw genome data rather than a trained classifier — this skill interprets an already-trained PICA/libSVM model, it does not train one or make phenotype predictions for new genomes.
- Genome completeness is far below the ~60-70% threshold at which traits are reliably predicted, since features derived from fragmentary genomes may not support stable rankings.

## Inputs

- Trained PICA SVM (libSVM linear kernel) phenotype model, e.g. the intracellular model trained on all 97 labelled species
- Published reference feature ranking (Additional file 1: Table S2) for comparison

## Outputs

- Signed ranked COG feature list (features ordered by absolute SVM weight, sign encoding predictor direction)
- ranked_features.tsv — tabulated features with weights and ranks

## How to apply

Load the trained intracellular phenotype model produced by the PICA SVM plug-in (libSVM linear kernel) trained on all 97 labelled species, and extract the raw SVM weight vector. Rank COG features by their absolute weights to obtain a signed ranked feature list, where the sign of each weight encodes whether the feature's presence (positive) or absence (negative) is predictive for the trait. Compare the resulting ranked COG list against the published ranking in Additional File 1 Table S2 to confirm agreement of the top entries, since the hypothesis under test is that genome reduction traits are driven by gene absence. Tabulate the ranked features with their weights and ranks and write the table to ranked_features.tsv. Validate against the published counts: all top 50 features negative predictors, exactly 2 positive predictors among the top 100, and 14 among the top 200.

## Related tools

- **PICA** (Framework whose trained SVM phenotype model supplies the weight vector to be ranked; the successor package is phenotrex) — https://github.com/univieCUBE/PICA
- **PICA support vector machine plug-in** (Plug-in used to train the intracellular model whose weights are extracted and ranked)
- **libSVM** (Linear-kernel SVM backend used by the PICA plug-in; linear weights are required for signed feature ranking)

## Evaluation signals

- All top 50 ranked features are negative predictors, matching Additional file 1: Table S2.
- Exactly 2 positive predictors appear among the top 100 features and 14 among the top 200.
- The top entries of the reproduced ranked COG list agree with the published ranking in Additional File 1 Table S2.
- ranked_features.tsv contains both weight values and rank positions for each COG feature, with sign preserved from the SVM weight vector.

## Limitations

- The intracellular model was trained on all 97 labelled species, so ranking reproducibility depends on the same labelled training set being available.
- Reliable trait prediction requires at least 60-70% genome completeness; models built on fragmentary genome bins may yield less stable feature rankings.
- The original PICA package has been superseded by phenotrex, which builds on scikit-learn/pandas and supports newer classifiers (e.g. XGBoost) that may not expose equivalent linear weights.

## Evidence

- [results] Feature ranking of the intracellular model trained on all 97 labelled species: [evidence span withheld — anchor status: not_found]
- [abstract] The PICA framework facilitates application and comparison of different machine learning techniques for phenotypic trait prediction.: "The PICA framework facilitates application and comparison of different machine learning techniques for phenotypic trait prediction."
- [discussion] Most of the traits can be reliably predicted in only 60-70% complete genomes: "Most of the traits can be reliably predicted in only 60-70% complete genomes, which allows reasonable predictions in genome bins from metagenomes."
- [readme] phenotrex succeeds PICA with new classifiers: "Please checkout the new `phenotrex` package, which succeeds PICA"

---
name: ranked-list-comparison-to-supplementary-table
description: Use when a trained linear SVM phenotype model must be inspected or its
  feature ranking reproduced and checked against a published table, e.g. verifying
  that trait absence rather than gene presence drives prediction.
when_to_use_negative:
- The trained model uses a non-linear kernel, so no interpretable per-feature weight
  vector can be extracted and ranked
- The goal is trait prediction for new genomes rather than model inspection — use
  phenotrex predict instead
- The reference table to compare against is not the PICA-based COG ranking in Additional
  File 1 Table S2
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0769
- http://edamontology.org/topic_0622
tools:
- name: PICA
  role: Framework in which the intracellular phenotype model was trained; the trained
    model provides the weight vector to rank
  repo: github.com/univieCUBE/PICA
- name: PICA support vector machine plug-in
  role: Trains the SVM phenotype model whose raw weight vector is extracted for the
    signed feature ranking
  repo: github.com/univieCUBE/PICA
- name: libSVM
  role: Linear-kernel SVM backend whose weight vector defines the COG feature ranking
- name: phenotrex
  role: Successor package to PICA for phenotype trait prediction; alternative for
    applying or retraining the models
  repo: univieCUBE/phenotrex
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/1471-2105-16-s14-s1
    title: Prediction of microbial phenotypes based on comparative genomics
  source_content_hash: sha256:76fd8e78bfffb9e9fc9da9cc76dfc5fb61e9d7697eb5a2597667fb19fb41719f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PICA
  - PICA support vector machine plug-in
  - libSVM
  - phenotrex
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0769
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# ranked-list-comparison-to-supplementary-table

## Summary

Reproduces a signed SVM feature ranking (COG weights from a PICA/libSVM linear-kernel phenotype model) and validates it against a published supplementary ranking, confirming top-50/top-100/top-200 predictor sign counts.

## When to use

Use when a trained linear SVM phenotype model must be inspected or its feature ranking reproduced and checked against a published table, e.g. verifying that trait absence rather than gene presence drives prediction.

## When NOT to use

- The trained model uses a non-linear kernel, so no interpretable per-feature weight vector can be extracted and ranked
- The goal is trait prediction for new genomes rather than model inspection — use phenotrex predict instead
- The reference table to compare against is not the PICA-based COG ranking in Additional File 1 Table S2

## Inputs

- Trained PICA intracellular phenotype model (libSVM linear kernel) trained on all 97 labelled species
- Published reference ranking: Additional File 1 Table S2 (ranked COG features with weights)
- COG feature space used for model training

## Outputs

- Signed ranked COG feature list (rank, weight, sign indicating positive vs negative predictor)
- ranked_features.tsv (tabulated ranks and weights)
- Validation counts: negative predictors in top 50, positive predictors in top 100 and top 200

## How to apply

Load the trained intracellular phenotype model produced by the PICA SVM plug-in (libSVM linear kernel) trained on all 97 labelled species. Extract the raw SVM weight vector and rank COG features by absolute weight to obtain a signed ranked feature list, where the sign encodes positive vs negative predictor. Compare the resulting ranked COG list against the published ranking in Additional File 1 Table S2 to confirm agreement of the top entries. Tabulate the ranked features with their weights and ranks and write ranked_features.tsv. Validate by checking the sign distribution of predictors: the reproduction must show all top 50 features as negative predictors, exactly 2 positive predictors among the top 100, and 14 among the top 200.

## Related tools

- **PICA** (Framework in which the intracellular phenotype model was trained; the trained model provides the weight vector to rank) — github.com/univieCUBE/PICA
- **PICA support vector machine plug-in** (Trains the SVM phenotype model whose raw weight vector is extracted for the signed feature ranking) — github.com/univieCUBE/PICA
- **libSVM** (Linear-kernel SVM backend whose weight vector defines the COG feature ranking)
- **phenotrex** (Successor package to PICA for phenotype trait prediction; alternative for applying or retraining the models) — univieCUBE/phenotrex

## Evaluation signals

- All top 50 ranked features are negative predictors (absence predictive of obligate intracellular trait), matching Additional File 1 Table S2
- Exactly 2 positive predictors appear among the top 100 ranked features
- Exactly 14 positive predictors appear among the top 200 ranked features
- Top entries of the reproduced ranking agree with the published ranking in Additional File 1 Table S2
- ranked_features.tsv is produced with ranks, weights, and signs for each COG feature

## Limitations

- Reliable trait prediction requires at least 60-70% genome completeness, which constrains the underlying model but is relevant when retraining
- The ranking reflects only this intracellular model; sign counts and top features will differ for other traits or retrained models
- PICA is superseded by the phenotrex package, which uses the SciPy stack (scikit-learn, pandas) and different classifiers

## Evidence

- [supplementary] Ranking of COG features by SVM weight: [evidence span withheld — anchor status: not_found]
- [abstract] PICA SVM plug-in applicability: "We have improved and extended PICA's support vector machine plug-in and suggest its applicability to large-scale genome databases and incomplete genome sequences."
- [intro] libSVM included as plug-in: "It includes plug-ins for CPAR and libSVM [19] among others."
- [readme] phenotrex recommends scikit-learn stack: "Builds upon the battle-tested SciPy stack (`scikit-learn`, `pandas`, etc.)"

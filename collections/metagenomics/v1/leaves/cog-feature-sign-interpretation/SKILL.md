---
name: cog-feature-sign-interpretation
description: Use when a trained PICA/libSVM linear-kernel phenotype model must be
  explained by ranking its COG features by signed weight, e.g. to test whether gene
  absence dominates a genome-reduction trait.
when_to_use_negative:
- Input is a raw genome sequence or metagenomic bin needing trait prediction rather
  than model interpretation (use phenotrex predict instead)
- The model uses a non-linear kernel, where a single interpretable signed weight vector
  per COG feature is not available
- Training data are incomplete genomes below the 60-70% completeness level the article
  reports as required for reliable prediction
tools:
- name: PICA
  role: Framework whose trained phenotype model provides the SVM weight vector over
    COG features to be ranked and interpreted
  repo: github.com/univieCUBE/PICA
- name: PICA support vector machine plug-in
  role: Plug-in used to train the linear-kernel SVM whose weights are extracted and
    signed-ranked
  repo: github.com/univieCUBE/PICA
- name: libSVM
  role: Underlying linear-kernel SVM implementation backing the PICA SVM plug-in weight
    vector
- name: phenotrex
  role: Successor package to PICA for microbial phenotypic trait prediction; offers
    model training and explanation on user data
  repo: https://github.com/univieCUBE/phenotrex
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/1471-2105-16-s14-s1
    title: Prediction of microbial phenotypes based on comparative genomics
  source_content_hash: sha256:cfebec03b2a77c58a1cfa1c6c2551e278a506e4025a21691b3872ae8e9adb7f0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PICA
  - PICA support vector machine plug-in
  - libSVM
  - phenotrex
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# COG feature sign interpretation for SVM phenotype models

## Summary

Extracts and interprets the signed SVM weight vector of a PICA (libSVM linear kernel) phenotype model to rank COG features by absolute weight, distinguishing presence-based (positive) from absence-based (negative) predictors of a trait such as obligate intracellular lifestyle.

## When to use

Use when a trained PICA/libSVM linear-kernel phenotype model must be explained by ranking its COG features by signed weight, e.g. to test whether gene absence dominates a genome-reduction trait.

## When NOT to use

- Input is a raw genome sequence or metagenomic bin needing trait prediction rather than model interpretation (use phenotrex predict instead)
- The model uses a non-linear kernel, where a single interpretable signed weight vector per COG feature is not available
- Training data are incomplete genomes below the 60-70% completeness level the article reports as required for reliable prediction

## Inputs

- Trained PICA SVM plug-in model (libSVM linear kernel) over COG features, e.g. the intracellular phenotype model trained on all 97 labelled species
- Published reference ranking (Additional File 1 Table S2) for comparison

## Outputs

- Signed ranked COG feature list (rank, COG feature, signed SVM weight)
- ranked_features.tsv with ranked features, weights and ranks

## How to apply

Load the trained phenotype model (here: the intracellular model trained on all 97 labelled species with the PICA SVM plug-in using a libSVM linear kernel) and extract the raw SVM weight vector over COG features. Rank features by absolute weight, keeping the sign so that negative weights mark features whose absence predicts the trait. Compare the resulting ranked COG list against the published ranking in Additional File 1 Table S2 to confirm the top entries agree. Tabulate feature names, weights and ranks and write the table to ranked_features.tsv. Interpretation hinges on sign: the published intracellular model shows all top 50 features as negative predictors, exactly 2 positive predictors among the top 100 and 14 among the top 200.

## Related tools

- **PICA** (Framework whose trained phenotype model provides the SVM weight vector over COG features to be ranked and interpreted) — github.com/univieCUBE/PICA
- **PICA support vector machine plug-in** (Plug-in used to train the linear-kernel SVM whose weights are extracted and signed-ranked) — github.com/univieCUBE/PICA
- **libSVM** (Underlying linear-kernel SVM implementation backing the PICA SVM plug-in weight vector)
- **phenotrex** (Successor package to PICA for microbial phenotypic trait prediction; offers model training and explanation on user data) — https://github.com/univieCUBE/phenotrex

## Examples

```
phenotrex predict --classifier MY_TRAIT.pkl genome.fna > predictions.tsv
```

## Evaluation signals

- Reproduced ranking agrees with the published top entries in Additional File 1 Table S2
- All top 50 ranked features are negative predictors (negative signed weights)
- Exactly 2 positive predictors occur among the top 100 features
- Exactly 14 positive predictors occur among the top 200 features
- ranked_features.tsv contains rank, feature and weight columns for every COG feature in the model

## Limitations

- Reliable trait prediction requires at least 60-70% genome completeness, which affects which genomes/models are trustworthy inputs
- The PICA package is superseded by phenotrex, so the original SVM plug-in tooling may need to be accessed from the archived PICA repository
- Prediction performance for further ecologically important traits is pending availability of sufficient training data

## Evidence

- [results] Feature ranking of the intracellular model trained on all 97 labelled species shows that all top 50 features are negative predictors: [evidence span withheld — anchor status: not_found]
- [intro] Generic COG-based methods as first layer: "Generic computational methods, namely those based on large databases of COGs, will most likely represent a first layer of methods for trait prediction."
- [abstract] PICA SVM plug-in applicability: "We have improved and extended PICA's support vector machine plug-in and suggest its applicability to large-scale genome databases and incomplete genome sequences."
- [readme] phenotrex explanation usage: [evidence span withheld — anchor status: external_doc]

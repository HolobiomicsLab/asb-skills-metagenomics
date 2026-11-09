---
name: association-rule-feature-selection-cpar
description: Use when reproducing or benchmarking the CPAR2SVM pipeline of PICA —
  association-rule-based feature selection of COG profiles followed by libSVM — for
  phenotypic trait prediction on ten example traits such as aerobe, motile, or therm.
when_to_use_negative:
- 'When speed matters more than feature interpretability: SVM-only classification
  achieved comparable phenotype prediction quality with roughly 3x lower runtime on
  the ten example traits'
- 'When working with the successor workflow: phenotrex replaces PICA and offers new
  classifiers (e.g. XGboost) beyond the CPAR/libSVM plug-in pair'
- 'When genomes are less than roughly 60-70% complete: most traits can only be reliably
  predicted above that completeness threshold'
edam_operation: http://edamontology.org/operation_2995
tools:
- name: PICA
  role: Framework hosting the CPAR2SVM pipeline for phenotypic trait prediction from
    comparative-genomics features
  repo: github.com/univieCUBE/PICA
- name: PICA support vector machine plug-in
  role: Extended libSVM-based classifier plug-in used both within CPAR2SVM and as
    the SVM-only comparison
  repo: github.com/univieCUBE/PICA
- name: libSVM
  role: SVM library behind PICA's classification plug-in, applied after CPAR feature
    selection
- name: phenotrex
  role: Successor package to PICA for end-to-end microbial phenotypic trait prediction
    on the scikit-learn stack
  repo: univieCUBE/phenotrex
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/1471-2105-16-s14-s1
    title: Prediction of microbial phenotypes based on comparative genomics
  source_content_hash: sha256:f324f80ee596530cbc316bb487b2cbcd965ad7a1c322fbd93d9cf0012ffa8d55
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
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# association-rule-feature-selection-cpar

## Summary

Feature-selection/classification workflow from the PICA framework in which association rule mining (CPAR) selects informative eggNOG 2.0 COG presence/absence features from microbial genomes before a libSVM classifier predicts microbial phenotypic traits. Serves as the methodological baseline against which PICA's purely SVM-based classification was compared.

## When to use

Use when reproducing or benchmarking the CPAR2SVM pipeline of PICA — association-rule-based feature selection of COG profiles followed by libSVM — for phenotypic trait prediction on ten example traits such as aerobe, motile, or therm.

## When NOT to use

- When speed matters more than feature interpretability: SVM-only classification achieved comparable phenotype prediction quality with roughly 3x lower runtime on the ten example traits
- When working with the successor workflow: phenotrex replaces PICA and offers new classifiers (e.g. XGboost) beyond the CPAR/libSVM plug-in pair
- When genomes are less than roughly 60-70% complete: most traits can only be reliably predicted above that completeness threshold

## Inputs

- eggNOG 2.0 COG presence/absence profiles for microbial genomes
- Phenotypic trait labels for the ten PICA example traits (aerobe, anaerobe, facult, gramneg, halo, motile, photo, psychro, spore, therm)

## Outputs

- Per-trait mean balanced accuracy and mean runtime across 50 fold-fits for CPAR2SVM
- svm_vs_cpar2svm_comparison.csv tabulating balanced accuracy and runtime for SVM-only vs CPAR2SVM side by side per trait

## How to apply

Load the PICA example dataset (github.com/univieCUBE/PICA) containing eggNOG 2.0 COG presence/absence profiles and labels for the ten example traits (aerobe, anaerobe, facult, gramneg, halo, motile, photo, psychro, spore, therm). For each trait, run the CPAR2SVM pipeline via PICA's CPAR and libSVM plug-ins: first mine association rules to select the subset of COGs associated with the trait, then train the libSVM classifier on the reduced feature set. Evaluate with 5-fold cross-validation repeated over 10 replicates (50 fold-fits per trait), recording balanced accuracy and wall-clock runtime per fit. Aggregate to per-trait means of balanced accuracy and runtime, and compare side by side with the SVM-only results — the paper reports no significant accuracy difference and an approximately 3x runtime advantage for SVM-only, so CPAR2SVM is the heavier baseline. Note that the successor package phenotrex (pip installable) has superseded PICA and builds on the scikit-learn SciPy stack.

## Related tools

- **PICA** (Framework hosting the CPAR2SVM pipeline for phenotypic trait prediction from comparative-genomics features) — github.com/univieCUBE/PICA
- **PICA support vector machine plug-in** (Extended libSVM-based classifier plug-in used both within CPAR2SVM and as the SVM-only comparison) — github.com/univieCUBE/PICA
- **libSVM** (SVM library behind PICA's classification plug-in, applied after CPAR feature selection)
- **phenotrex** (Successor package to PICA for end-to-end microbial phenotypic trait prediction on the scikit-learn stack) — univieCUBE/phenotrex

## Evaluation signals

- Comparison table contains rows for all ten traits (aerobe through therm) with both mean balanced accuracy and mean runtime populated for SVM-only and CPAR2SVM
- CPAR2SVM balanced accuracy shows no significant difference from SVM-only per trait (statistical comparison over the 50 fold-fits)
- SVM-only mean runtime is approximately 3x lower than CPAR2SVM averaged over the ten traits
- Evaluation uses 10 replicates of 5-fold cross-validation (50 fold-fits per trait per method), with balanced accuracy as the metric

## Limitations

- Reliable trait prediction requires at least 60-70% genome completeness, limiting applicability to very incomplete bins
- Performance for further ecologically important traits is pending availability of sufficient training data (ongoing work)
- PICA is superseded by phenotrex, which adds classifiers such as XGboost and is available from PyPI
- Generic COG-based methods are positioned as a first layer of trait prediction; additional specific models are needed for deeper interpretation

## Evidence

- [intro] PICA plug-in ecosystem: "It includes plug-ins for CPAR and libSVM [19] among others."
- [abstract] SVM plug-in extension: "We have improved and extended PICA's support vector machine plug-in and suggest its applicability to large-scale genome databases and incomplete genome sequences."
- [intro] Motivation for SVM-only comparison: "In this study we investigate the application of purely SVM-based classification for phenotypic trait prediction in microbiology and microbial ecology, with an emphasis on metagenomic sequences and"
- [readme] PICA superseded by phenotrex: "Please checkout the new `phenotrex` package, which succeeds PICA"
- [intro] Scope of COG-based methods: "Generic computational methods, namely those based on large databases of COGs, will most likely represent a first layer of methods for trait prediction."

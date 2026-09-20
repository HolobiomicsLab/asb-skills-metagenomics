---
name: svm-classifier-training-on-cog-profiles
description: Use when training phenotype prediction models from comparative-genomics
  feature tables (e.g. eggNOG COG presence/absence profiles) and you need fast, accurate
  SVM-only classification without an explicit feature-selection step.
when_to_use_negative:
- The input is already a reduced/selected feature set or the user specifically needs
  interpretable association rules — then run the CPAR2SVM pipeline instead of SVM-only.
- Training labels for the trait of interest are insufficient; the article notes further
  ecologically important traits await sufficient training data.
- Genomes are far below the 60-70% completeness threshold at which most traits remain
  reliably predictable.
edam_operation: http://edamontology.org/operation_3094
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: PICA
  role: Framework hosting the SVM trait-classification plug-in and the comparison
    experiment on COG profiles
  repo: https://github.com/univieCUBE/PICA
- name: PICA support vector machine plug-in
  role: Purely SVM-based classifier applied to COG presence/absence profiles for trait
    prediction
  repo: https://github.com/univieCUBE/PICA
- name: libSVM
  role: SVM implementation used by the PICA plug-in and by the CPAR2SVM comparison
    arm
- name: phenotrex
  role: Successor package of PICA for end-to-end microbial phenotypic trait prediction
    (train/predict/explain)
  repo: https://github.com/univieCUBE/phenotrex
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/1471-2105-16-s14-s1
    title: Prediction of microbial phenotypes based on comparative genomics
  source_content_hash: sha256:1b5f61fc3381ee1a761a2f0f8d0f00d9858ca537f774780f266d26ce19a96609
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/univieCUBE/PICA
  tools:
  - PICA
  - PICA support vector machine plug-in
  - libSVM
  - phenotrex
  edam_operation: http://edamontology.org/operation_3094
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# svm-classifier-training-on-cog-profiles

## Summary

Train purely SVM-based phenotypic trait classifiers (PICA libSVM plug-in) on eggNOG COG presence/absence profiles, benchmarked against CPAR2SVM (association-rule feature selection + SVM) to show SVM-only achieves comparable prediction quality at roughly one-third the runtime.

## When to use

Use when training phenotype prediction models from comparative-genomics feature tables (e.g. eggNOG COG presence/absence profiles) and you need fast, accurate SVM-only classification without an explicit feature-selection step.

## When NOT to use

- The input is already a reduced/selected feature set or the user specifically needs interpretable association rules — then run the CPAR2SVM pipeline instead of SVM-only.
- Training labels for the trait of interest are insufficient; the article notes further ecologically important traits await sufficient training data.
- Genomes are far below the 60-70% completeness threshold at which most traits remain reliably predictable.

## Inputs

- eggNOG 2.0 COG presence/absence profiles for training genomes (from github.com/univieCUBE/PICA)
- Ten phenotypic trait label sets: aerobe, anaerobe, facult, gramneg, halo, motile, photo, psychro, spore, therm

## Outputs

- Per-trait mean balanced accuracy and mean runtime for SVM-only and CPAR2SVM over 10 replicates x 5 folds
- Side-by-side comparison table (e.g. svm_vs_cpar2svm_comparison.csv) with rows for all ten traits
- Trained PICA SVM trait classifiers applicable to new genomes

## How to apply

Load COG presence/absence profiles and the ten example trait labels (aerobe, anaerobe, facult, gramneg, halo, motile, photo, psychro, spore, therm) from the PICA repository. For each trait, train the PICA SVM-only classifier (libSVM plug-in) under 5-fold cross-validation repeated over 10 replicates, recording per-fold balanced accuracy and wall-clock runtime. As a decision point, run the CPAR2SVM pipeline — CPAR association-rule mining for feature selection followed by libSVM — with identical cross-validation settings to check that feature selection is unnecessary. Aggregate mean balanced accuracy and mean runtime over the 50 fold-fits per trait and per method, tabulate side by side, and confirm SVM-only shows no significant accuracy loss while running roughly 3x faster. The article's rationale is that purely SVM-based classification suits the rapidly growing genome databases and metagenomic sequences, motivating dropping the costly CPAR step.

## Related tools

- **PICA** (Framework hosting the SVM trait-classification plug-in and the comparison experiment on COG profiles) — https://github.com/univieCUBE/PICA
- **PICA support vector machine plug-in** (Purely SVM-based classifier applied to COG presence/absence profiles for trait prediction) — https://github.com/univieCUBE/PICA
- **libSVM** (SVM implementation used by the PICA plug-in and by the CPAR2SVM comparison arm)
- **phenotrex** (Successor package of PICA for end-to-end microbial phenotypic trait prediction (train/predict/explain)) — https://github.com/univieCUBE/phenotrex

## Examples

```
git clone https://github.com/univieCUBE/PICA; for trait in aerobe anaerobe facult gramneg halo motile photo psychro spore therm; do PICA --classifier libSVM --cv 5 --replicates 10 examples/$trait; done
```

## Evaluation signals

- The comparison table contains rows for all ten traits (aerobe … therm) with both balanced-accuracy and runtime metrics populated for SVM-only and CPAR2SVM.
- Mean balanced accuracy of SVM-only is not significantly lower than CPAR2SVM for any of the ten traits.
- Averaged runtime of SVM-only is approximately three times lower than CPAR2SVM across the ten traits.
- Both arms use identical cross-validation (5-fold, 10 replicates, 50 fold-fits per trait) so the runtime comparison is apples-to-apples.
- Trait models generalize to held-out genomes with reasonable prediction quality, consistent with reported 60-70% completeness robustness.

## Limitations

- Most traits can only be reliably predicted in 60-70% complete genomes, so very fragmentary genome bins may fail.
- Performance for further ecologically important traits is pending availability of sufficient training data (ongoing work).
- The PICA repository now directs users to the phenotrex package, which supersedes PICA with SciPy-stack-based classifiers (e.g. scikit-learn, XGBoost) — the PICA/libSVM path is legacy.

## Evidence

- [abstract] The PICA framework facilitates application and comparison of different machine learning techniques for phenotypic trait prediction.: "The PICA framework facilitates application and comparison of different machine learning techniques for phenotypic trait prediction."
- [intro] libSVM plug-in listing among PICA plug-ins: "It includes plug-ins for CPAR and libSVM [19] among others."
- [intro] Motivation for purely SVM-based classification at scale: "In this study we investigate the application of purely SVM-based classification for phenotypic trait prediction in microbiology and microbial ecology, with an emphasis on metagenomic sequences and"
- [readme] PICA succeeded by phenotrex with improved stack and new classifiers: "Please checkout the new `phenotrex` package, which succeeds PICA: https://github.com/univieCUBE/phenotrex"
- [readme] phenotrex prediction output format: [evidence span withheld — anchor status: external_doc]

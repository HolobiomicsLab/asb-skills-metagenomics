---
name: comparative-genomics-cog-profile-classification
description: Use when predicting biological phenotypes or ecological roles of uncharacterized
  microbial genomes or metagenomic bins from COG presence/absence profiles, or when
  benchmarking SVM-only vs CPAR2SVM phenotype classifiers.
when_to_use_negative:
- Genome completeness is well below 60-70% — the article reports most traits can be
  reliably predicted only in 60-70% complete genomes
- Sufficient training data are unavailable for the ecologically important trait of
  interest — the article notes performance for further traits is pending availability
  of training data
- Input is already a raw phenotype annotation requiring no COG-profile-based inference
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: PICA
  role: Framework for application and comparison of different machine learning techniques
    for phenotypic trait prediction from COG profiles
  repo: github.com/univieCUBE/PICA
- name: PICA support vector machine plug-in
  role: Purely SVM-based classifier applied to large-scale genome databases and incomplete
    genome sequences
  repo: github.com/univieCUBE/PICA
- name: libSVM
  role: SVM classifier plug-in used both standalone and downstream of CPAR feature
    selection
- name: phenotrex
  role: Successor package to PICA for end-to-end microbial phenotypic trait prediction
    (train/predict/explain models)
  repo: https://github.com/univieCUBE/phenotrex
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/1471-2105-16-s14-s1
    title: Prediction of microbial phenotypes based on comparative genomics
  source_content_hash: sha256:119b7a1d1aa066c75be94c18016d2a74bbbb5bddd10b79c1211452cd397f4137
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PICA
  - PICA support vector machine plug-in
  - libSVM
  - phenotrex
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# comparative-genomics-cog-profile-classification

## Summary

Predicts microbial phenotypic traits (e.g. aerobe, motile, thermophile) from eggNOG COG presence/absence profiles using the PICA framework's machine-learning plug-ins, comparing pure libSVM classification against CPAR association-rule feature selection followed by SVM. Purely SVM-based classification achieves equal predictive quality at roughly three times lower runtime than the CPAR2SVM hybrid.

## When to use

Use when predicting biological phenotypes or ecological roles of uncharacterized microbial genomes or metagenomic bins from COG presence/absence profiles, or when benchmarking SVM-only vs CPAR2SVM phenotype classifiers.

## When NOT to use

- Genome completeness is well below 60-70% — the article reports most traits can be reliably predicted only in 60-70% complete genomes
- Sufficient training data are unavailable for the ecologically important trait of interest — the article notes performance for further traits is pending availability of training data
- Input is already a raw phenotype annotation requiring no COG-profile-based inference

## Inputs

- eggNOG 2.0 COG presence/absence profiles per genome
- Phenotypic trait labels for ten traits (aerobe, anaerobe, facult, gramneg, halo, motile, photo, psychro, spore, therm)
- Genome FASTA files (for prediction with successor tool phenotrex)

## Outputs

- Per-trait phenotype predictions with confidence scores ranging from 0.5 to 1.0 (e.g. predictions.tsv with YES/NO and confidence)
- Cross-validated balanced-accuracy and runtime comparison per trait (svm_vs_cpar2svm_comparison.csv)
- Trained phenotype classifiers (e.g. MY_TRAIT.pkl model files)

## How to apply

Load eggNOG 2.0 COG presence/absence profiles together with training labels for phenotypic traits such as aerobe, anaerobe, facult, gramneg, halo, motile, photo, psychro, spore, and therm. For each trait, train the PICA SVM-only classifier (libSVM plug-in) with 5-fold cross-validation repeated over 10 replicates, recording per-fold balanced accuracy and wall-clock runtime. In parallel, run the CPAR2SVM pipeline — CPAR association rule mining for feature selection followed by libSVM — under identical cross-validation settings. Aggregate per trait into a comparison table (e.g. svm_vs_cpar2svm_comparison.csv) with mean balanced accuracy and mean runtime per method, then confirm that SVM-only shows no significant accuracy loss while running roughly 3x faster. Prefer the SVM-only plug-in for large-scale genome databases and incomplete genome sequences, as PICA's extended SVM plug-in was explicitly suggested for this setting.

## Related tools

- **PICA** (Framework for application and comparison of different machine learning techniques for phenotypic trait prediction from COG profiles) — github.com/univieCUBE/PICA
- **PICA support vector machine plug-in** (Purely SVM-based classifier applied to large-scale genome databases and incomplete genome sequences) — github.com/univieCUBE/PICA
- **libSVM** (SVM classifier plug-in used both standalone and downstream of CPAR feature selection)
- **phenotrex** (Successor package to PICA for end-to-end microbial phenotypic trait prediction (train/predict/explain models)) — https://github.com/univieCUBE/phenotrex

## Examples

```
pip install phenotrex[fasta]; phenotrex predict --classifier MY_TRAIT.pkl genome.fna > predictions.tsv
```

## Evaluation signals

- Comparison table contains rows for all ten traits with both mean balanced accuracy and mean runtime populated for both methods (50 fold-fits from 10 replicates of 5-fold CV each)
- SVM-only shows no significant balanced-accuracy loss versus CPAR2SVM averaged over the ten traits
- SVM-only mean runtime is approximately three times lower than CPAR2SVM
- Predictions carry confidence values within the documented 0.5 to 1.0 range
- Predictive power remains stable and is not perturbed by the rapid growth of genome databases

## Limitations

- Most traits can be reliably predicted only in 60-70% complete genomes, limiting applicability to partial genome bins
- Performance for further ecologically important traits awaits availability of sufficient training data (ongoing work)
- PICA is superseded by phenotrex, which adds new classifiers (e.g. XGBoost), test-driven development, and Python 3.7+ support
- The CPAR2SVM hybrid incurs roughly three times the runtime of SVM-only classification without an accuracy benefit

## Evidence

- [abstract] The PICA framework facilitates application and comparison of different machine learning techniques for phenotypic trait prediction.: "The PICA framework facilitates application and comparison of different machine learning techniques for phenotypic trait prediction."
- [intro] In this study we investigate the application of purely SVM-based classification for phenotypic trait prediction in microbiology and microbial ecology: "In this study we investigate the application of purely SVM-based classification for phenotypic trait prediction in microbiology and microbial ecology"
- [abstract] We have improved and extended PICA's support vector machine plug-in and suggest its applicability to large-scale genome databases and incomplete genome sequences.: "We have improved and extended PICA's support vector machine plug-in and suggest its applicability to large-scale genome databases and incomplete genome sequences."
- [discussion] Most of the traits can be reliably predicted in only 60-70% complete genomes, which allows reasonable predictions in genome bins from metagenomes.: "Most of the traits can be reliably predicted in only 60-70% complete genomes, which allows reasonable predictions in genome bins from metagenomes."
- [readme] This yields a tabular file containing a prediction regarding the presence of the trait (YES or NO), as well as a confidence value the model ascribes to this prediction, ranging from 0.5 to 1.0.: [evidence span withheld — anchor status: external_doc]

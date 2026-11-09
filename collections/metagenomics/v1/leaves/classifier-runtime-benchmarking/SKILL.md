---
name: classifier-runtime-benchmarking
description: Use when comparing prediction quality and computational cost of a simplified
  classifier (e.g. SVM-only) against a feature-selection-then-classifier pipeline
  (e.g. CPAR2SVM) over repeated cross-validation runs.
when_to_use_negative:
- The goal is to train a deployable phenotype model rather than to compare classifier
  quality/runtime — use standard phenotrex/PICA training and prediction instead
- Input genomes are incomplete (~60-70%) and the goal is new trait prediction; this
  skill benchmarks classifiers on complete-genome training data rather than predicting
  on genome bins
- The classifier pair being compared does not share an evaluation protocol (folds,
  replicates, metric) — runtime/accuracy parity conclusions require identical cross-validation
  settings
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: PICA
  role: Framework hosting the phenotype prediction classifiers being benchmarked on
    the example COG presence/absence dataset
  repo: github.com/univieCUBE/PICA
- name: PICA support vector machine plug-in
  role: Provides the improved SVM-only classification arm of the comparison, applied
    to large-scale genome data
  repo: github.com/univieCUBE/PICA
- name: libSVM
  role: Underlying SVM classifier used in both arms (SVM-only and after CPAR feature
    selection)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/1471-2105-16-s14-s1
    title: Prediction of microbial phenotypes based on comparative genomics
  source_content_hash: sha256:f8d22e9611059ad29368839570058180be446c748a175058af1b6f15db2e7824
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PICA
  - PICA support vector machine plug-in
  - libSVM
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# classifier-runtime-benchmarking

## Summary

Benchmark two classifier pipelines (PICA's SVM-only plug-in versus CPAR association-rule feature selection followed by SVM) on the ten example phenotypic traits, comparing balanced accuracy and wall-clock runtime to establish whether the simpler SVM-only route loses predictive quality while running ~3x faster.

## When to use

Use when comparing prediction quality and computational cost of a simplified classifier (e.g. SVM-only) against a feature-selection-then-classifier pipeline (e.g. CPAR2SVM) over repeated cross-validation runs.

## When NOT to use

- The goal is to train a deployable phenotype model rather than to compare classifier quality/runtime — use standard phenotrex/PICA training and prediction instead
- Input genomes are incomplete (~60-70%) and the goal is new trait prediction; this skill benchmarks classifiers on complete-genome training data rather than predicting on genome bins
- The classifier pair being compared does not share an evaluation protocol (folds, replicates, metric) — runtime/accuracy parity conclusions require identical cross-validation settings

## Inputs

- PICA example dataset: eggNOG 2.0 COG presence/absence profiles per genome (from github.com/univieCUBE/PICA)
- Phenotypic trait labels for ten traits: aerobe, anaerobe, facult, gramneg, halo, motile, photo, psychro, spore, therm

## Outputs

- svm_vs_cpar2svm_comparison.csv with one row per trait and columns for mean balanced accuracy and mean runtime of SVM-only and CPAR2SVM
- Per-fold balanced accuracy and wall-clock runtime measurements for each of the 50 fold-fits per method per trait

## How to apply

Load the PICA example dataset of eggNOG 2.0 COG presence/absence profiles with the ten trait labels (aerobe, anaerobe, facult, gramneg, halo, motile, photo, psychro, spore, therm). For each trait, train the PICA libSVM plug-in with 5-fold cross-validation repeated over 10 replicates, recording per-fold balanced accuracy and wall-clock runtime; repeat identically for the CPAR2SVM pipeline (CPAR association-rule feature selection followed by libSVM). Aggregate to mean balanced accuracy and mean runtime per trait across the 50 fold-fits, then tabulate side by side per trait. The rationale is that purely SVM-based classification is attractive for large genome databases and metagenomic sequences only if it does not sacrifice prediction quality relative to CPAR pre-selection. Judge correctness by confirming all ten trait rows are populated for both metrics and that SVM-only shows no significant balanced-accuracy loss while running roughly three times faster.

## Related tools

- **PICA** (Framework hosting the phenotype prediction classifiers being benchmarked on the example COG presence/absence dataset) — github.com/univieCUBE/PICA
- **PICA support vector machine plug-in** (Provides the improved SVM-only classification arm of the comparison, applied to large-scale genome data) — github.com/univieCUBE/PICA
- **libSVM** (Underlying SVM classifier used in both arms (SVM-only and after CPAR feature selection))

## Evaluation signals

- svm_vs_cpar2svm_comparison.csv contains rows for all ten traits (aerobe through therm) with both mean balanced accuracy and mean runtime populated for both methods
- No significant difference in mean balanced accuracy between SVM-only and CPAR2SVM averaged over the ten traits
- Mean SVM-only runtime is approximately three times lower than CPAR2SVM when averaged over the ten phenotypic traits
- Both methods were run with the identical protocol: 5-fold cross-validation repeated over 10 replicates (50 fold-fits per trait per method)

## Limitations

- Reliable trait prediction requires at least 60-70% genome completeness, so benchmark conclusions apply to near-complete genomes rather than very partial sequences
- Performance for further ecologically important traits is pending availability of sufficient training data (ongoing work)
- The comparison is grounded in the PICA example dataset and its ten traits; runtime and accuracy ratios may not transfer to other datasets or feature representations

## Evidence

- [abstract] The PICA framework facilitates application and comparison of different machine learning techniques for phenotypic trait prediction.: "The PICA framework facilitates application and comparison of different machine learning techniques for phenotypic trait prediction."
- [abstract] We have improved and extended PICA's support vector machine plug-in and suggest its applicability to large-scale genome databases and incomplete genome sequences.: "We have improved and extended PICA's support vector machine plug-in and suggest its applicability to large-scale genome databases and incomplete genome sequences."
- [intro] In this study we investigate the application of purely SVM-based classification for phenotypic trait prediction in microbiology and microbial ecology: "In this study we investigate the application of purely SVM-based classification for phenotypic trait prediction in microbiology and microbial ecology"
- [intro] It includes plug-ins for CPAR and libSVM [19] among others.: "It includes plug-ins for CPAR and libSVM [19] among others."
- [readme] Please checkout the new `phenotrex` package, which succeeds PICA: "Please checkout the new `phenotrex` package, which succeeds PICA"

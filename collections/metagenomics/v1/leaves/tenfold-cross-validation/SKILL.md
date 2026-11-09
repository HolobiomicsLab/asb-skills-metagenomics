---
name: tenfold-cross-validation
description: Use when a machine learning model (e.g. Random Forest) has been trained
  in MegaR on taxonomic profiles (16S or WGS) and you need a variance-minimized, unbiased
  estimate of predictive accuracy before reporting or deployment.
when_to_use_negative:
- You still have raw reads or an unpreprocessed abundance table — preprocess via MegaR's
  getGoodfeature (threshold 0.003, 5% sample percentage) first, since CV on unfiltered
  sparse features yields poor models
- The model is intended only for quick exploratory training-error inspection, not
  for reporting defensible accuracy — the confusion-matrix test error on the held-out
  split alone may suffice
- There are more than two classes and you have not used the 'Select level to classify'
  tab to remove control/irrelevant classes, which the article notes increases model
  accuracy
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3174
tools:
- name: MegaR
  role: Interactive R Shiny package whose Validation panel runs N independent runs
    of 10-fold cross validation on the trained model
  repo: https://github.com/BioHPC/MegaR
- name: R
  role: Runtime environment in which MegaR and its CV routines execute (R > 3.6.0)
- name: randomForest
  role: Package underlying the Random Forest model whose accuracy is cross-validated
- name: caret
  role: Package supporting MegaR's model fitting and cross-validation machinery
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-020-03933-4
    title: 'MegaR: an interactive R package for rapid sample classification and phenotype
      prediction using metagenome profiles and machine learning'
  source_content_hash: sha256:5f1fe73da0d11ed3802493106637310afd8c27682ff42cd2ff4a3001e2d1158f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioHPC/MegaR
  tools:
  - MegaR
  - R
  - caret
  - randomForest
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# tenfold-cross-validation

## Summary

Validate a MegaR-trained Random Forest (or other) classification model on metagenomic profiles using N independent runs of 10-fold cross validation, as prescribed by the MegaR pipeline for DIABIMMUNE T1D data, to estimate robust cross-validated accuracy and guard against overfitting and bias.

## When to use

Use when a machine learning model (e.g. Random Forest) has been trained in MegaR on taxonomic profiles (16S or WGS) and you need a variance-minimized, unbiased estimate of predictive accuracy before reporting or deployment.

## When NOT to use

- You still have raw reads or an unpreprocessed abundance table — preprocess via MegaR's getGoodfeature (threshold 0.003, 5% sample percentage) first, since CV on unfiltered sparse features yields poor models
- The model is intended only for quick exploratory training-error inspection, not for reporting defensible accuracy — the confusion-matrix test error on the held-out split alone may suffice
- There are more than two classes and you have not used the 'Select level to classify' tab to remove control/irrelevant classes, which the article notes increases model accuracy

## Inputs

- Preprocessed species/genus-level metagenomic abundance table (merged MetaPhlAn table, e.g. merged_metaphlan_t1d.txt, or QIIME OTU/BIOM file)
- Tab-separated metadata file (e.g. metadata.tsv) with sample IDs matching the profile table and a class-label column (e.g. T1D_Diagnosed)
- Trained MegaR model (optionally a downloaded RDS model file)
- Number of independent runs N (e.g. 3)

## Outputs

- Cross-validated accuracy estimate averaged over N runs of 10-fold CV (e.g. 80–90% for DIABIMMUNE cohorts; ~72% for antibiotic-cohort WGS)
- Validation report/plot from the MegaR Validation panel
- Evidence that the model is neither overfitted nor biased

## How to apply

After building the model in MegaR with metadata-derived class labels (e.g. column 7 'T1D_Diagnosed' with t/f labels and column 2 'Gid_shotgun' as sample IDs) and a training/test split (e.g. 80%), go to the Validation panel and run k-fold cross validation with k=10. Because variance is high when models are fit to small datasets such as the DIABIMMUNE T1D cohorts, configure N independent runs of the 10-fold CV (N=3 for the T1D experiment) and report the resulting cross-validated accuracy alongside the test-set confusion-matrix accuracy. This validation step follows preprocessing via getGoodfeature (species-level features, threshold 0.003, sample-percentage 5%, no normalization) and Random Forest training with selected numbers of variables at split (e.g. 41 and 81). Accept the result if cross-validation accuracy falls in the expected range — e.g. 80–90% for DIABIMMUNE cohorts in Table 2, with the antibiotic-cohort WGS model near 72%.

## Related tools

- **MegaR** (Interactive R Shiny package whose Validation panel runs N independent runs of 10-fold cross validation on the trained model) — https://github.com/BioHPC/MegaR
- **R** (Runtime environment in which MegaR and its CV routines execute (R > 3.6.0))
- **randomForest** (Package underlying the Random Forest model whose accuracy is cross-validated)
- **caret** (Package supporting MegaR's model fitting and cross-validation machinery)

## Examples

```
library(MegaR); MegaR()  # launch the Shiny app, upload merged_metaphlan_t1d.txt + metadata.tsv, then set Validation: N=3 independent runs of 10-fold CV
```

## Evaluation signals

- Cross-validation accuracy falls within the 80–90% range reported in Table 2 for all six DIABIMMUNE cohort/data-type models, except antibiotic-cohort WGS at ~72%
- The number of independent runs is set to N=3 (T1D experiment) over 10 folds, so the reported accuracy is an average over 30 fit/validation cycles, not a single split
- CV accuracy is consistent with (does not wildly exceed) the held-out test-set accuracy from the confusion matrix — a large gap signals overfitting or leakage
- Sample IDs in metadata.tsv exactly match sample IDs in the merged metaphlan/qiime file (mismatches invalidate the label assignment during CV folds)
- The model achieves highest accuracy among the tested split-parameter settings (e.g. 41 vs 81 variables at split), confirming MegaR's parameter selection was respected

## Limitations

- k-fold CV on small datasets has high variance; MegaR mitigates this with N independent runs of 10-fold CV but the variance remains nonzero
- The article reports 'various split criteria' tested with 'various accuracies' and no quantitative accuracy values in the discussion, so exact reproduction depends on matching the paper's settings
- Only GLM, Random Forest, and SVM classifiers are available in MegaR; other ML and deep-learning classifiers were not tested (planned future work), limiting CV-based comparisons

## Evidence

- [discussion] Cross-validation robustness: "Our cross validation of the improved model shows that the models are robust"
- [readme] N independent 10-fold runs to reduce variance: "MegaR provides users to select N independent runs of the 10-fold cross validation to minimize such a high variance."
- [readme] T1D validation setting: "For the T1D experiment, provide **3** for this field."
- [readme] Purpose of cross validation: "cross validation is essential to validate the model accuracy and model bias. This implies that the developed model should not be overfitted and not having bias."
- [methods] Preprocessing call feeding the model: "- `getGoodfeature(alltable2, threshold, samplePercent, normval)`"

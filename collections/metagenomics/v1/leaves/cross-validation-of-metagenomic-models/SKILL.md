---
name: cross-validation-of-metagenomic-models
description: Use when a trained metagenomic classification model (e.g. random forest
  on DIABIMMUNE T1D profiles) needs robustness verification beyond a single train/test
  split, or when overfitting/bias must be ruled out before reporting accuracies.
when_to_use_negative:
- You only need a quick single-split accuracy estimate and have no concern about overfitting
  on small data — a plain confusion matrix on the held-out 20% suffices
- The input is a raw, unfiltered taxonomic table — preprocess (species-level selection,
  threshold filtering, normalization choice) before validating
- The task is phenotype prediction of unknown samples with a pre-trained RDS model
  rather than model performance assessment
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0622
tools:
- name: MegaR
  role: R Shiny package providing cross-validation options (N independent runs of
    10-fold CV) to assess model accuracy and bias
  repo: https://github.com/BioHPC/MegaR
- name: caret
  role: R dependency used by MegaR for model training and resampling machinery
  repo: https://github.com/BioHPC/MegaR
- name: randomForest
  role: R dependency backing the RF classifier whose robustness is validated
  repo: https://github.com/BioHPC/MegaR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-020-03933-4
    title: 'MegaR: an interactive R package for rapid sample classification and phenotype
      prediction using metagenome profiles and machine learning'
  source_content_hash: sha256:6dd517e22db4af8a2620befbe6aca2a481c6efee481ce2c3f349306df22fa91e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioHPC/MegaR
  tools:
  - MegaR
  - R
  - caret
  - randomForest
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# cross-validation-of-metagenomic-models

## Summary

Validates MegaR-trained metagenomic classifiers (RF, SVM, GLM on 16S/WGS taxonomic profiles) using N independent runs of 10-fold cross validation to confirm accuracy and absence of overfitting/bias before trusting held-out test accuracies.

## When to use

Use when a trained metagenomic classification model (e.g. random forest on DIABIMMUNE T1D profiles) needs robustness verification beyond a single train/test split, or when overfitting/bias must be ruled out before reporting accuracies.

## When NOT to use

- You only need a quick single-split accuracy estimate and have no concern about overfitting on small data — a plain confusion matrix on the held-out 20% suffices
- The input is a raw, unfiltered taxonomic table — preprocess (species-level selection, threshold filtering, normalization choice) before validating
- The task is phenotype prediction of unknown samples with a pre-trained RDS model rather than model performance assessment

## Inputs

- Preprocessed taxonomic profile (merged MetaPhlAn table or QIIME OTU/BIOM) with low-abundance features removed (threshold 0.003, 5% percentage-of-sample filter)
- metadata.tsv with class column (e.g. column 7, T1D_Diagnosed) and sample-ID column (e.g. column 2, Gid_shotgun)
- Trained MegaR model (RF/SVM/GLM) and number of independent cross-validation runs (e.g. 3)

## Outputs

- Cross-validation accuracy estimates across k=10 folds for N independent runs
- Comparison of cross-validated performance against test-set confusion matrix, statistics tab, and AUC-ROC values
- Evidence statement that the model is robust / not overfitted

## How to apply

After building the model in MegaR (e.g. RF with mtry candidates 41 and 81 on an 80% training split of the T1D merged MetaPhlAn table), assess held-out performance first via getconfuMat on the 20% test set, then run the validation step with k=10 folds. Because variance is high when models are fitted to small datasets, MegaR lets you select N independent runs of the 10-fold cross validation; for the T1D experiment use N=3 runs to minimize this variance. Rationale from the paper: removing sparse features first (threshold 0.003, 5% of samples) improves accuracy, and cross validation of the improved model shows the models are robust. Compare cross-validated accuracy to the test-set confusion-matrix accuracy and AUC-ROC; agreement indicates the model is not overfitted.

## Related tools

- **MegaR** (R Shiny package providing cross-validation options (N independent runs of 10-fold CV) to assess model accuracy and bias) — https://github.com/BioHPC/MegaR
- **caret** (R dependency used by MegaR for model training and resampling machinery) — https://github.com/BioHPC/MegaR
- **randomForest** (R dependency backing the RF classifier whose robustness is validated) — https://github.com/BioHPC/MegaR

## Examples

```
> library(MegaR); MegaR()  # Model building tab: RF, 80% train split; Validation tab: k=10 folds, N=3 independent runs
```

## Evaluation signals

- Cross-validated accuracy from 3 runs of 10-fold CV is consistent with the 20% held-out test-set accuracy from getconfuMat (no large gap indicating overfitting)
- AUC-ROC of the validated model agrees with reported statistics-tab accuracy (e.g. RF ~0.94 on WGS T1D, RF 0.9017 / SVM 0.9245 on 16S T1D)
- Variance across the N independent CV runs is small, consistent with the paper's rationale for multiple runs on small datasets
- RF remains the top classifier across cohorts/data types, with the documented exception of 16S T1D where SVM wins

## Limitations

- k-fold cross validation can have high variance when the dataset is small; N independent runs mitigate but do not eliminate this
- The paper tested various split criteria and obtained various accuracies depending on the data, and does not state the exact split criteria used for final results
- Only RF, SVM, and GLM were evaluated; other machine learning and deep learning classifiers were not yet tested (planned future work), and no quantitative accuracy values were reported in the discussion section

## Evidence

- [discussion] Cross validation of the improved model: "Our cross validation of the improved model shows that the models are robust"
- [readme] MegaR CV options: "MegaR provides users to select N independent runs of the 10-fold cross validation to minimize such a high variance."
- [readme] CV rationale: "The variance in fitting the model tends to be higher if it is fitted to a small dataset, therefore k-fold cross validation can have a high variance."
- [readme] T1D setting: "* For the T1D experiment, provide **3** for this field."
- [readme] Overfitting purpose: "First and foremost, cross validation is essential to validate the model accuracy and model bias. This implies that the developed model should not be overfitted and not having bias."

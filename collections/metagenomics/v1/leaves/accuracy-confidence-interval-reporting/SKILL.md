---
name: accuracy-confidence-interval-reporting
description: Use when reproducing or reporting phenotype-classification accuracy from
  metagenomic feature tables (16S or WGS MetaPhlAn/QIIME profiles) where both held-out
  test accuracy and cross-validated accuracy with a 95% CI must be quoted, e.g. against
  MegaR Table 2 values.
when_to_use_negative:
- The model has not been cross-validated — reporting only a single train/test split
  accuracy without the N-run 10-fold estimate (MegaR treats CV as essential to check
  that the model is 'not overfitted and not having bias')
- The dataset is very small or single-cohort, where k-fold CV itself can have high
  variance and the 80–90% DIABIMMUNE benchmark ranges do not apply
- The goal is benchmarking against MetAML or comparing SVM/GLM classifiers rather
  than quoting accuracy with uncertainty for a single RF model
edam_operation: http://edamontology.org/operation_0571
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0622
tools:
- name: MegaR
  role: Interactive R Shiny interface used to train the RF model, compute the confusion
    matrix, and run N independent runs of 10-fold cross validation
  repo: https://github.com/BioHPC/MegaR
- name: randomForest
  role: Random Forest implementation underlying the RF model whose accuracy and cross-validation
    performance are reported
- name: caret
  role: R package used within MegaR for model training/cross-validation support
- name: R
  role: Runtime environment; MegaR is installed via devtools::install_github and launched
    with MegaR()
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-020-03933-4
    title: 'MegaR: an interactive R package for rapid sample classification and phenotype
      prediction using metagenome profiles and machine learning'
  source_content_hash: sha256:20570232c92faac2bf41152c21f1941cda707b1cc2b6b9a464e60b34ecdef645
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioHPC/MegaR
  tools:
  - MegaR
  - R
  - caret
  - randomForest
  edam_operation: http://edamontology.org/operation_0571
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# accuracy-confidence-interval-reporting

## Summary

Report Random Forest classification performance on metagenomic profiles by combining the test-set confusion-matrix accuracy with accuracy from N independent runs of 10-fold cross validation, each quantified with a 95% confidence interval. This is how the MegaR paper validates model robustness and avoids quoting a single split-dependent number.

## When to use

Use when reproducing or reporting phenotype-classification accuracy from metagenomic feature tables (16S or WGS MetaPhlAn/QIIME profiles) where both held-out test accuracy and cross-validated accuracy with a 95% CI must be quoted, e.g. against MegaR Table 2 values.

## When NOT to use

- The model has not been cross-validated — reporting only a single train/test split accuracy without the N-run 10-fold estimate (MegaR treats CV as essential to check that the model is 'not overfitted and not having bias')
- The dataset is very small or single-cohort, where k-fold CV itself can have high variance and the 80–90% DIABIMMUNE benchmark ranges do not apply
- The goal is benchmarking against MetAML or comparing SVM/GLM classifiers rather than quoting accuracy with uncertainty for a single RF model

## Inputs

- Preprocessed metagenomic feature table (species-level MetaPhlAn merged table or QIIME OTU/BIOM file, threshold-filtered at 0.003 with 5% sample-percentage, no normalization)
- Tab-separated metadata.tsv with class-label column (e.g. column 7, T1D_Diagnosed; t/f) and sample-ID column (e.g. column 2, Gid_shotgun) matching the feature-table sample IDs
- Trained MegaR Random Forest model (optionally the downloaded RDS model file)

## Outputs

- Test-set accuracy from the confusion matrix for each cohort/data-type model
- Cross-validated accuracy from N=3 independent runs of 10-fold cross validation
- Per-model optimized accuracy with 95% confidence interval (Table 2-style report)

## How to apply

Train the Random Forest model in MegaR (gettrainingdonerf) with the tuned mtry values (41 and 81 variables at split for the T1D experiment), an 80% training split, and class labels pulled from the metadata (e.g. column 7, T1D_Diagnosed, values t/f). Record the test-set accuracy from the confusion matrix, then run validation with N=3 independent runs of 10-fold cross validation — MegaR offers this repeated k-fold option because 'the variance in fitting the model tends to be higher if it is fitted to a small dataset'. For each cohort/data-type combination, report the optimized accuracy together with its 95% CI, and judge robustness by comparing against the paper's Table 2 ranges (80–90% for most DIABIMMUNE models; ~72% for the antibiotic-cohort WGS data). The rationale is that a single 80/20 split gives a noisy estimate; sparse-feature filtering (threshold 0.003, 5% sample-percentage) should be applied first since removing sparse features increased model accuracy.

## Related tools

- **MegaR** (Interactive R Shiny interface used to train the RF model, compute the confusion matrix, and run N independent runs of 10-fold cross validation) — https://github.com/BioHPC/MegaR
- **randomForest** (Random Forest implementation underlying the RF model whose accuracy and cross-validation performance are reported)
- **caret** (R package used within MegaR for model training/cross-validation support)
- **R** (Runtime environment; MegaR is installed via devtools::install_github and launched with MegaR())

## Examples

```
devtools::install_github("BioHPC/MegaR"); library(MegaR); MegaR()
```

## Evaluation signals

- The six tenfold cross-validation accuracies fall within the 80–90% range reported in Table 2, except the antibiotic-cohort WGS model which should be near ~72%
- Validation used N=3 independent runs of 10-fold cross validation (the T1D experiment setting), not a single k-fold run
- Each reported optimized accuracy is accompanied by a 95% confidence interval
- The test-set accuracy from the confusion matrix is consistent with (not grossly divergent from) the cross-validated accuracy, indicating the model is not overfitted
- Metadata sample IDs matched the feature-table sample IDs exactly (e.g. Gid_shotgun in column 2), so accuracy is computed on correctly aligned labels

## Limitations

- Exact split criteria used for the final results are not stated in the article — only that 'various split criteria' were tested and 'depending on the data, obtained various accuracies'
- No quantitative accuracy values are reported for some comparison sections, so CI comparison may rely on the workflow's Table 2 ranges
- Other machine learning classifiers and deep learning methods had not yet been tested at publication time, so accuracy/CI behavior is only characterized for RF, SVM, and GLM

## Evidence

- [readme] MegaR provides users to select N independent runs of the 10-fold cross validation to minimize such a high variance.: "MegaR provides users to select N independent runs of the 10-fold cross validation to minimize such a high variance."
- [discussion] Our cross validation of the improved model shows that the models are robust: "Our cross validation of the improved model shows that the models are robust"
- [readme] For the T1D experiment, provide **3** for this field.: "For the T1D experiment, provide **3** for this field."
- [discussion] removing sparse features expressed in low numbers increased the machine learning model accuracy: "removing sparse features expressed in low numbers increased the machine learning model accuracy"

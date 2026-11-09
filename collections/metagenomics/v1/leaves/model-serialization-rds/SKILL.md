---
name: model-serialization-rds
description: Use when a trained MegaR classification model (RF/GLM/SVM on merged MetaPhlAn
  or QIIME/OTU/BIOM tables) must be persisted to RDS and later reused to predict classes
  for unknown samples without repeating the training pipeline.
when_to_use_negative:
- No trained model exists yet — the unknown-sample data must first go through the
  full MegaR pipeline (readmydata, getLevelData, getGoodfeature, gettrainingdonerf)
  before serialization
- The unknown-sample table uses a different taxonomic level, feature set, or profiling
  tool format than the model was trained on (merged metaphlan table only matches MetaPhlAn-trained
  models; BIOM only matches QIIME-trained models)
- The goal is to validate or tune model accuracy (confusion matrix, AUC-ROC, k-fold
  cross validation) rather than to persist and reuse a finished model
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: MegaR
  role: Interactive R Shiny package whose Download Model button and 'Use custom model'
    Prediction tab handle RDS serialization and reload for unknown-sample prediction
  repo: https://github.com/BioHPC/MegaR
- name: R
  role: Runtime language; saveRDS/readRDS persistence and getunknpredict execution
- name: randomForest
  role: Provides the Random Forest classifier whose fitted object is serialized to
    RDS for reuse
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-020-03933-4
    title: 'MegaR: an interactive R package for rapid sample classification and phenotype
      prediction using metagenome profiles and machine learning'
  source_content_hash: sha256:f37e53648ade4d7e7254e7ade4dcfe120e995a1e7e8dc2ded95470fb5822a610
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioHPC/MegaR
  tools:
  - MegaR
  - R
  - randomForest
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# model-serialization-rds

## Summary

Serializes a trained MegaR machine learning model (e.g. Random Forest on species-level MetaPhlAn T1D features) to an RDS file via the Download Model button, and reloads that file to classify unknown samples with getunknpredict without re-training.

## When to use

Use when a trained MegaR classification model (RF/GLM/SVM on merged MetaPhlAn or QIIME/OTU/BIOM tables) must be persisted to RDS and later reused to predict classes for unknown samples without repeating the training pipeline.

## When NOT to use

- No trained model exists yet — the unknown-sample data must first go through the full MegaR pipeline (readmydata, getLevelData, getGoodfeature, gettrainingdonerf) before serialization
- The unknown-sample table uses a different taxonomic level, feature set, or profiling tool format than the model was trained on (merged metaphlan table only matches MetaPhlAn-trained models; BIOM only matches QIIME-trained models)
- The goal is to validate or tune model accuracy (confusion matrix, AUC-ROC, k-fold cross validation) rather than to persist and reuse a finished model

## Inputs

- Trained MegaR model object (e.g. randomForest model fitted on species-level T1D features)
- Downloaded RDS model file produced by the Download Model button
- Unknown-sample abundance table (merged metaphlan table if model trained on MetaPhlAn data; BIOM file if trained on QIIME data)

## Outputs

- Serialized RDS model file
- Predicted class label per unknown sample, written as a list to unknown_sample_predictions.tsv

## How to apply

After training (e.g. Random Forest with gettrainingdonerf on the species-level, threshold-0.003, 5% sample-percentage, NO-normalization T1D feature table), click the Download Model button in MegaR, which serializes the model object to an RDS file (saveRDS under the hood). To reuse the model, open the Prediction tab and select 'Use custom model', then upload the downloaded RDS model together with the unknown-sample merged metaphlan table (or BIOM file if the model was trained on QIIME data) — the sample IDs and feature space must match the training data. Run getunknpredict on the reloaded model to obtain a predicted class (e.g. 'f' or 't' for the T1D_Diagnosed labels) for each unknown sample. The rationale is that the RDS serialization preserves the fitted model state, enabling fast-turnaround phenotype prediction on new cohorts without re-running preprocessing-to-training.

## Related tools

- **MegaR** (Interactive R Shiny package whose Download Model button and 'Use custom model' Prediction tab handle RDS serialization and reload for unknown-sample prediction) — https://github.com/BioHPC/MegaR
- **R** (Runtime language; saveRDS/readRDS persistence and getunknpredict execution)
- **randomForest** (Provides the Random Forest classifier whose fitted object is serialized to RDS for reuse)

## Examples

```
saveRDS(trained_rf_model, "t1d_rf_model.RDS")  # Download Model equivalent; reload via readRDS / 'Use custom model' then getunknpredict(unknown_table, model)
```

## Evaluation signals

- The exported model file loads cleanly via readRDS / the 'Use custom model' upload without errors
- Every unknown sample in the input abundance table receives exactly one predicted class label (f or t) in the output list
- Predictions from the reloaded RDS model match predictions from the in-memory trained model on the same input
- The unknown-sample table format matches the training data type (merged metaphlan table for MetaPhlAn models, BIOM for QIIME models)

## Limitations

- With 100% of data used for training there is no held-out test set to generate a confusion matrix, so accuracy of the persisted model is not directly assessable
- The RDS model is tied to the exact feature space and profiling tool of the training data; mismatched unknown-sample tables cannot be predicted
- Model quality depends on fine-tuning choices (e.g. mtry range 41–81 for the T1D RF experiment); a poorly tuned serialized model will propagate inaccurate predictions

## Evidence

- [readme] MegaR provides the option to download the trained model for later use in Prediction. If a user clicks **Download Model**, the RDS model file is generated and downloaded.: "MegaR provides the option to download the trained model for later use in Prediction. If a user clicks **Download Model**, the RDS model file is generated and downloaded."
- [readme] It is also possible to upload a previously trained model for prediction using the **Use custom model** tab under **Prediction**.: "It is also possible to upload a previously trained model for prediction using the **Use custom model** tab under **Prediction**. If a user click the **Prediction** and select **Use cumstome model**,"
- [abstract] Unknown sample prediction with a properly trained model using MegaR will enhance researchers to identify the sample property in a fast turnaround time.: "Unknown sample prediction with a properly trained model using MegaR will enhance researchers to identify the sample property in a fast turnaround time."

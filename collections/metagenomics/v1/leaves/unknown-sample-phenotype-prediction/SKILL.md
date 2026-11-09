---
name: unknown-sample-phenotype-prediction
description: Use when a trained MegaR model exists as an RDS file and an unknown-sample
  abundance table (merged MetaPhlAn table or BIOM) must be assigned class labels (e.g.
  f/t) without re-running model training.
when_to_use_negative:
- No trained model exists yet — this skill requires a previously trained model; train
  one first via MegaR's Model building tab
- The unknown-sample table format does not match the model's training data format
  (e.g. a BIOM file for a MetaPhlAn-derived model)
- The goal is benchmarking model accuracy or cross-validation rather than labeling
  new samples — use MegaR's Validation / confusion matrix workflow instead
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3474
tools:
- name: MegaR
  role: Interactive R Shiny package providing Download Model (saveRDS), Use custom
    model upload, and getunknpredict for classifying unknown samples against a trained
    model
  repo: https://github.com/BioHPC/MegaR
- name: R
  role: Runtime environment for MegaR; readRDS loads the serialized model and the
    prediction is executed as R code
  repo: https://github.com/BioHPC/MegaR
- name: randomForest
  role: Random Forest classifier backing the trained model used for unknown-sample
    prediction
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-020-03933-4
    title: 'MegaR: an interactive R package for rapid sample classification and phenotype
      prediction using metagenome profiles and machine learning'
  source_content_hash: sha256:410eb5a3a12362337b601825e2ecd3c6b695a72660348a72f6f28de1a2b1d37a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioHPC/MegaR
  tools:
  - MegaR
  - R
  - randomForest
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# unknown-sample-phenotype-prediction

## Summary

Classify unknown metagenomic samples against a previously trained MegaR model by reloading the serialized RDS file (via Download Model / saveRDS) and running the prediction module, avoiding any re-training. This enables fast phenotype prediction (e.g. T1D t/f status) from taxonomic profiles.

## When to use

Use when a trained MegaR model exists as an RDS file and an unknown-sample abundance table (merged MetaPhlAn table or BIOM) must be assigned class labels (e.g. f/t) without re-running model training.

## When NOT to use

- No trained model exists yet — this skill requires a previously trained model; train one first via MegaR's Model building tab
- The unknown-sample table format does not match the model's training data format (e.g. a BIOM file for a MetaPhlAn-derived model)
- The goal is benchmarking model accuracy or cross-validation rather than labeling new samples — use MegaR's Validation / confusion matrix workflow instead

## Inputs

- Trained MegaR model serialized as an RDS file (from Download Model / saveRDS)
- Unknown-sample abundance table as a merged MetaPhlAn table (e.g. merged_metaphlan_t1d.txt format) or BIOM file matching the model's training format
- Class labels (f/t) as defined during training in the metadata class column

## Outputs

- List of predicted class labels (f or t), one per unknown sample
- unknown_sample_predictions.tsv mapping each unknown sample to its predicted category

## How to apply

First train a Random Forest model in MegaR (e.g. on the T1D DIABIMMUNE merged_metaphlan_t1d.txt preprocessed at species level with threshold 0.003, 5% sample percentage, NO normalization, 80% training split, class labels f/t, mtry range 41-81), then click Download Model to serialize the model to an RDS file via saveRDS. For prediction, load the RDS model without retraining (in the GUI via the Use custom model tab under Prediction, or programmatically with readRDS) and supply the unknown-sample table, which must be a merged MetaPhlAn table if the model was built from MetaPhlAn data (BIOM if built from QIIME data). Run the unknown-sample prediction function (getunknpredict) on the abundance table so the model assigns each sample a category. The output is a list of predicted classes, one per unknown sample, which is written to unknown_sample_predictions.tsv. The rationale is that MegaR's Download Model mechanism makes trained models portable and reusable for fast-turnaround phenotype prediction.

## Related tools

- **MegaR** (Interactive R Shiny package providing Download Model (saveRDS), Use custom model upload, and getunknpredict for classifying unknown samples against a trained model) — https://github.com/BioHPC/MegaR
- **R** (Runtime environment for MegaR; readRDS loads the serialized model and the prediction is executed as R code) — https://github.com/BioHPC/MegaR
- **randomForest** (Random Forest classifier backing the trained model used for unknown-sample prediction)

## Examples

```
library(MegaR); model <- readRDS("trained_rf_model.rds"); preds <- getunknpredict(model, unknown_table)
```

## Evaluation signals

- The RDS model file loads cleanly (readRDS without error) and returns a model object consistent with the trained Random Forest
- Every unknown sample in the input abundance table receives exactly one predicted class label (f or t) in the output list
- unknown_sample_predictions.tsv contains one prediction per sample, and sample IDs match those in the input table
- The unknown-sample file format matches the training format: merged MetaPhlAn table for a MetaPhlAn model, BIOM for a QIIME model
- No re-training occurs — predictions are produced solely by reloading the downloaded RDS model

## Limitations

- The unknown set must be a BIOM file if the model was developed from QIIME data and a merged MetaPhlAn table if developed from MetaPhlAn data — mismatched formats are not supported
- The downloaded model depends on MegaR's training parameters (e.g. species-level features, threshold 0.003); samples must be preprocessed consistently for reliable prediction
- Reported accuracy varies with split criteria and the article does not report quantitative accuracy values for the comparisons

## Evidence

- [readme] MegaR provides the option to download the trained model for later use in Prediction.: "MegaR provides the option to download the trained model for later use in Prediction. If a user clicks **Download Model**, the RDS model file is generated and downloaded."
- [readme] A user can load this model for fast prediction without retraining.: "A user can load this model for the fast prediction of unknown samples without training the model again."
- [readme] Unknown set must be biom file if model is developed from qiime data and merged metaphlan table if model is developed from the metaphlan data.: "Unknown set must be biom file if model is developed from qiime data and merged metaphlan table if model is developed from the metaphlan data."
- [readme] upload the downloaded RDS model (trained model) along with the unknown dataset to predict the phenotype fast: "a user can upload the downloaded RDS model (trained model) along with the unknown dataset to predict the phenotype fast without training the model"
- [abstract] Unknown sample prediction with a properly trained MegaR model enables researchers to identify sample properties with fast turnaround time.: "Unknown sample prediction with a properly trained model using MegaR will enhance researchers to identify the sample property in a fast turnaround time."

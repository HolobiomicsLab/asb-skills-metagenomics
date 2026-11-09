---
name: metagenomic-sample-phenotype-classification
description: Use when you have a metagenomic taxonomic profile table (MetaPhlAn merged
  table or QIIME OTU/BIOM) plus sample metadata with class labels, and the goal is
  to train a classifier or predict the phenotype of unknown samples.
when_to_use_negative:
- The input is raw sequencing reads rather than an already-profiled taxonomic abundance
  table (MetaPhlAn/QIIME output) — MegaR does not profile reads.
- The goal is differential abundance testing or time-interval analysis rather than
  phenotype classification (tools like MetaDprof or MetaLonDa are cited for those
  tasks).
- You need >2-class models with automatic class handling — use the 'Select level to
  classify' tab only to remove control/less important classes; deep learning classifiers
  are not yet supported.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_2269
tools:
- name: MegaR
  role: 'R Shiny package providing the full classification workflow: readmydata, getLevelData,
    getGoodfeature, gettrainingdonerf/svm/glm, getconfuMat, validation, saveRDS/getunknpredict'
  repo: https://github.com/BioHPC/MegaR
- name: R
  role: Runtime environment; MegaR is installed via devtools::install_github and requires
    R >3.6.0 with randomForest, caret, shiny dependencies
  repo: https://github.com/BioHPC/MegaR
- name: randomForest
  role: Underlying Random Forest classifier used by gettrainingdonerf, which in most
    scenarios gave the highest accuracy versus SVM and GLM
- name: caret
  role: Dependency supporting model training/cross-validation machinery within MegaR
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_006
  source_papers:
  - doi: 10.1186/s12859-020-03933-4
    title: 'MegaR: an interactive R package for rapid sample classification and phenotype
      prediction using metagenome profiles and machine learning'
  source_content_hash: sha256:0fee642b3c53c541233ed6795afa01ba1bb344eab3aef50d7a87b6f38040da11
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioHPC/MegaR
  tools:
  - MegaR
  - R
  - caret
  - randomForest
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# metagenomic-sample-phenotype-classification

## Summary

Train and apply machine learning classifiers (Random Forest, SVM, GLM) in MegaR on 16S rRNA OTU/BIOM or WGS MetaPhlAn taxonomic profiles to classify samples into phenotype categories (e.g., T1D diagnosed t/f) and predict unknown samples from a saved RDS model.

## When to use

Use when you have a metagenomic taxonomic profile table (MetaPhlAn merged table or QIIME OTU/BIOM) plus sample metadata with class labels, and the goal is to train a classifier or predict the phenotype of unknown samples.

## When NOT to use

- The input is raw sequencing reads rather than an already-profiled taxonomic abundance table (MetaPhlAn/QIIME output) — MegaR does not profile reads.
- The goal is differential abundance testing or time-interval analysis rather than phenotype classification (tools like MetaDprof or MetaLonDa are cited for those tasks).
- You need >2-class models with automatic class handling — use the 'Select level to classify' tab only to remove control/less important classes; deep learning classifiers are not yet supported.

## Inputs

- MetaPhlAn merged taxonomic profile table (e.g., merged_metaphlan_t1d.txt) or QIIME OTU/BIOM file
- Tab-separated metadata file (metadata.tsv) with sample IDs and class column (T1D_Diagnosed, values t/f)
- Unknown-sample abundance table (BIOM for QIIME-based models, merged MetaPhlAn table for MetaPhlAn-based models)
- Trained model RDS file (for prediction without retraining)

## Outputs

- Trained classification model (downloadable as RDS)
- Confusion matrix and test-set statistics from the 20% held-out test set
- Accuracy plot across selected parameters (e.g., mtry values)
- AUC-ROC performance plot
- Top-ten important species/genus features with variable importance
- Predicted class label per unknown sample (e.g., written to unknown_sample_predictions.tsv)

## How to apply

Load the profile table (e.g., merged_metaphlan_t1d.txt) via readmydata and the tab-separated metadata.tsv via readmetadata, ensuring sample IDs match exactly between the two files. Preprocess with getLevelData at Species Level and getGoodfeature with the default 0.003 abundance threshold, 5% percentage-of-sample filter, and 'NO' normalization (as used for the T1D DIABIMMUNE experiment); removing sparse low-abundance features is reported to increase model accuracy. Train models with gettrainingdonerf (mtry candidates 41 and 81, 80% training split), gettrainingdonesvm, or gettrainingdoneglm, using column 7 (T1D_Diagnosed) for class and column 2 (Gid_shotgun) for sample IDs, then assess with getconfuMat confusion matrix, statistics tab, and AUC-ROC plot on the 20% held-out test set. Confirm robustness with validation() running 3 independent runs of 10-fold cross-validation. Serialize the best model via saveRDS (Download Model button) and reuse it with getunknpredict on an unknown-sample abundance table to assign each sample a predicted class label without retraining.

## Related tools

- **MegaR** (R Shiny package providing the full classification workflow: readmydata, getLevelData, getGoodfeature, gettrainingdonerf/svm/glm, getconfuMat, validation, saveRDS/getunknpredict) — https://github.com/BioHPC/MegaR
- **R** (Runtime environment; MegaR is installed via devtools::install_github and requires R >3.6.0 with randomForest, caret, shiny dependencies) — https://github.com/BioHPC/MegaR
- **randomForest** (Underlying Random Forest classifier used by gettrainingdonerf, which in most scenarios gave the highest accuracy versus SVM and GLM)
- **caret** (Dependency supporting model training/cross-validation machinery within MegaR)

## Examples

```
devtools::install_github("BioHPC/MegaR"); library(MegaR); MegaR()
```

## Evaluation signals

- Confusion matrix on the 20% held-out test set yields a plausible accuracy (reported RF best accuracies include 0.8846 WGS three-country, 0.9385 WGS T1D; 16S T1D SVM 0.9245 vs RF 0.9017).
- validation() with 3 independent runs of 10-fold cross-validation shows cross-validated accuracy consistent with the test-set accuracy, indicating the model is not overfitted.
- Species-level feature models should markedly outperform genus-level models (reported roughly tenfold accuracy difference).
- The downloaded RDS model loads cleanly and getunknpredict returns exactly one predicted class label (f or t) per unknown sample.
- Metadata sample IDs match exactly the sample IDs in the MetaPhlAn/QIIME input file; any mismatch invalidates training.

## Limitations

- Split criteria for final results vary by dataset ('various split criteria' tested, yielding various accuracies), so exact accuracy is data-dependent.
- Only RF, SVM, and GLM classifiers are implemented; other machine learning and deep learning methods are planned future work.
- Cross-validation on small datasets can have high variance, hence the N independent runs of 10-fold CV option.
- If 100% of data is used for training, no test set remains to generate a confusion matrix.
- The unknown prediction set format must match the model's source: BIOM for QIIME-derived models, merged MetaPhlAn table for MetaPhlAn-derived models.

## Evidence

- [discussion] RF model provides highest accuracy in most metagenomic classification scenarios compared to SVM and GLM: "Our results indicate that the RF model provides the highest accuracy in most metagenomic classification scenarios compared to SVM and GLM."
- [discussion] removing sparse features increases accuracy: "removing sparse features expressed in low numbers increased the machine learning model accuracy"
- [readme] download model for prediction without retraining: "If a user clicks **Download Model**, the RDS model file is generated and downloaded. A user can load this model for the fast prediction of unknown samples without training the model again."
- [methods] default preprocessing thresholds for T1D: "This field is getting a floating number to remove profiles and their abundances below the threshold value. Default value is **0.003**."
- [readme] T1D class and ID columns: "For the T1D expeirment, column 7 (T1D_Diagnosed) has the class values (t, f) where t is True and f is False."

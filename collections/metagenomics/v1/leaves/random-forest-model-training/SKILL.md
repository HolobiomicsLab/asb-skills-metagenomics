---
name: random-forest-model-training
description: Use when a two-class or multi-class phenotype classification model must
  be built from a merged MetaPhlAn table or QIIME/BIOM OTU table plus metadata (e.g.
  T1D t/f labels), and RF accuracy must be tuned and validated.
when_to_use_negative:
- The goal is predicting unknown samples with an already-trained model — use model
  reuse/prediction from the saved RDS file instead of re-training.
- The input is raw sequencing reads or is not a MetaPhlAn/QIIME taxonomic profile
  table — MegaR only accepts OTU/BIOM and merged MetaPhlAn inputs.
- A GLM or SVM classifier is specifically required — although MegaR offers these,
  this skill is scoped to RF, which was the highest-accuracy option in this study.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3473
tools:
- name: MegaR
  role: R Shiny package providing gettrainingdonerf model building, confusion-matrix
    evaluation, mtry accuracy tuning, and RDS model download
  repo: https://github.com/BioHPC/MegaR
- name: R
  role: runtime environment (R >3.6.0) in which MegaR is installed and executed
  repo: https://github.com/BioHPC/MegaR
- name: randomForest
  role: underlying RF implementation used by MegaR for model training and mtry selection
- name: caret
  role: dependency used for cross-validation support in MegaR
provenance:
  source_task_ids:
  - task_006
  - task_003
  - task_002
  source_papers:
  - doi: 10.1186/s12859-020-03933-4
    title: 'MegaR: an interactive R package for rapid sample classification and phenotype
      prediction using metagenome profiles and machine learning'
  source_content_hash: sha256:9d4fdffe18aba7aaf8988b1e448415090a88c6f3d2e72e2e338ddeaa97a680c3
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
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# random-forest-model-training

## Summary

Train a Random Forest classifier on preprocessed metagenome taxonomic profiles (MetaPhlAn/BIOM) using MegaR's Shiny workflow, selecting the best mtry from candidate values and evaluating via confusion matrix and cross-validation.

## When to use

Use when a two-class or multi-class phenotype classification model must be built from a merged MetaPhlAn table or QIIME/BIOM OTU table plus metadata (e.g. T1D t/f labels), and RF accuracy must be tuned and validated.

## When NOT to use

- The goal is predicting unknown samples with an already-trained model — use model reuse/prediction from the saved RDS file instead of re-training.
- The input is raw sequencing reads or is not a MetaPhlAn/QIIME taxonomic profile table — MegaR only accepts OTU/BIOM and merged MetaPhlAn inputs.
- A GLM or SVM classifier is specifically required — although MegaR offers these, this skill is scoped to RF, which was the highest-accuracy option in this study.

## Inputs

- merged MetaPhlAn table (e.g. merged_metaphlan_t1d.txt) or OTU/BIOM file from MetaPhlAn2/QIIME
- tab-separated metadata.tsv with sample IDs and class column (e.g. column 7 T1D_Diagnosed, column 2 Gid_shotgun)
- preprocessed feature table from getLevelData/getGoodfeature (species level, threshold 0.003, 5% sample percentage, NO normalization)

## Outputs

- trained randomForest model object (serializable to .RDS via Download Model)
- train error-rate plot and test-set confusion matrix with accuracy statistics
- accuracy plot across mtry candidates (e.g. 41 and 81)
- AUC-ROC plot and top-10 important-feature ranking

## How to apply

Load the merged MetaPhlAn table (e.g. merged_metaphlan_t1d.txt) and metadata.tsv, then preprocess to the chosen taxon level (species for T1D) with threshold 0.003, 5% sample-percentage filtering, and NO normalization before training. In MegaR's Model building tab select Random Forest, supply the metadata column numbers (7 = T1D_Diagnosed class labels t/f, 2 = Gid_shotgun sample IDs, which must match the profile table exactly), set 80% of data for training, and provide candidate numbers of variables at split (41 and 81 for T1D); MegaR plots accuracy across these mtry values and builds the final model with the best-performing setting. Rationale: RF gave the highest accuracy in most metagenomic classification scenarios versus SVM and GLM, and removing sparse low-abundance features increased model accuracy. Evaluate on the held-out test set via the confusion matrix (getconfuMat) and statistics tab, and confirm robustness with N independent runs of 10-fold cross-validation (N=3 for T1D). The trained model can be serialized as an RDS file via Download Model for later unknown-sample prediction.

## Related tools

- **MegaR** (R Shiny package providing gettrainingdonerf model building, confusion-matrix evaluation, mtry accuracy tuning, and RDS model download) — https://github.com/BioHPC/MegaR
- **R** (runtime environment (R >3.6.0) in which MegaR is installed and executed) — https://github.com/BioHPC/MegaR
- **randomForest** (underlying RF implementation used by MegaR for model training and mtry selection)
- **caret** (dependency used for cross-validation support in MegaR)

## Examples

```
install.packages("devtools"); devtools::install_github("BioHPC/MegaR"); library(MegaR); MegaR()
```

## Evaluation signals

- Test-set accuracy from the confusion matrix for DIABIMMUNE cohorts should fall in the 80–90% range reported in Table 2 (antibiotic-cohort WGS near ~72%).
- The accuracy tab plot must show results for each selected mtry value (41 and 81), and the final model must use the best-accuracy setting.
- Metadata sample IDs (column 2 Gid_shotgun) must match the profile-table sample IDs exactly, or training will fail/mislabel.
- 3 independent runs of 10-fold cross-validation should reproduce the reported CV accuracy without indicating overfitting or high bias.
- The downloaded .RDS file must reload cleanly and return class labels (f/t) for every unknown sample passed to prediction.

## Limitations

- MegaR is limited to three classifiers (GLM, RF, SVM); deep learning and other classifiers were untested and only planned future work.
- k-fold cross-validation on small datasets can have high variance, hence the option of N independent runs of 10-fold CV.
- Exact split criteria for the final reported results were not fully stated; different train/test splits yielded various accuracies.
- The README's T1D threshold guidance contains an inconsistency (defaults 0.003 in the Threshold description but says 'leave the default value (0.03)'), so verify the threshold actually used.

## Evidence

- [discussion] RF model provides highest accuracy in most metagenomic classification scenarios compared to SVM and GLM: "Our results indicate that the RF model provides the highest accuracy in most metagenomic classification scenarios compared to SVM and GLM."
- [methods] MegaR preprocessing and normalization options: "MegaR provides fours choices for data normalization: **Cumulative Sum Scaling (CSS)**, **Quantile**, **Trimmed Mean of M-values (TMM)**, and **NO**."
- [readme] Number of variable at split (mtry) definition: "This field is the number of variable that is randomly collected to be sampled at each split time. **Accuracy** results will show the results by this randomly selected predictors."
- [abstract] MegaR builds ML models from taxonomic profiles: "The MegaR employs taxonomic profiles from either whole metagenome sequencing or 16S rRNA sequencing data to develop machine learning models and classify the samples into two or more categories."
- [discussion] Removing sparse features improves accuracy: "removing sparse features expressed in low numbers increased the machine learning model accuracy"

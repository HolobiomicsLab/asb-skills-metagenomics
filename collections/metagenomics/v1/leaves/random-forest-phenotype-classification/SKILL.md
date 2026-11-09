---
name: random-forest-phenotype-classification
description: Use when classifying metagenomic samples into phenotype classes (e.g.,
  T2D, cirrhosis, T1D) from merged MetaPhlAn or QIIME/BIOM taxonomic profile tables
  with matching tab-separated metadata, and a robust RF-based accuracy estimate is
  needed.
when_to_use_negative:
- Input is raw sequencing reads rather than a MetaPhlAn/QIIME taxonomic profile table
  — profile the reads first
- The goal requires classifiers beyond GLM, RF, and SVM (e.g., deep learning), which
  MegaR has not yet tested
- The dataset is not a microbiome-phenotype classification problem, e.g., differential-abundance
  or time-interval analysis better served by MetaDprof or MetaLonDa
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3306
- http://edamontology.org/topic_3474
tools:
- name: MegaR
  role: R Shiny package providing preprocessing (getGoodfeature), RF model training
    (gettrainingdonerf), k-fold cross validation, and unknown-sample prediction
  repo: https://github.com/BioHPC/MegaR
- name: R
  role: Language in which MegaR is implemented and invoked (library(MegaR); MegaR())
- name: R-Shiny
  role: Interactive web interface exposing model building, tuning, and cross-validation
    to the user
- name: MetAML
  role: Reference classifier against which MegaR RF accuracies are benchmarked on
    the T2D and Cirrhosis datasets
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-020-03933-4
    title: 'MegaR: an interactive R package for rapid sample classification and phenotype
      prediction using metagenome profiles and machine learning'
  source_content_hash: sha256:bd105739bdb1e092766bf6683cee62123be5865ff731c9f5beb94845ddd2919c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioHPC/MegaR
  tools:
  - MegaR
  - MetAML
  - R
  - R-Shiny
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3306
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# random-forest-phenotype-classification

## Summary

Train and cross-validate a Random Forest phenotype classifier on MetaPhlAn/QIIME taxonomic profiles using MegaR, which the authors show achieves the highest accuracy in most metagenomic classification scenarios versus SVM and GLM (e.g., 0.6683 T2D and 0.8846 Cirrhosis vs MetAML's 0.6640/0.8770).

## When to use

Use when classifying metagenomic samples into phenotype classes (e.g., T2D, cirrhosis, T1D) from merged MetaPhlAn or QIIME/BIOM taxonomic profile tables with matching tab-separated metadata, and a robust RF-based accuracy estimate is needed.

## When NOT to use

- Input is raw sequencing reads rather than a MetaPhlAn/QIIME taxonomic profile table — profile the reads first
- The goal requires classifiers beyond GLM, RF, and SVM (e.g., deep learning), which MegaR has not yet tested
- The dataset is not a microbiome-phenotype classification problem, e.g., differential-abundance or time-interval analysis better served by MetaDprof or MetaLonDa

## Inputs

- Merged MetaPhlAn taxonomic profile table (e.g., merged_metaphlan_t1d.txt) or QIIME OTU/BIOM file
- Tab-separated metadata file with sample IDs matching the profile table exactly, plus a class column
- Column numbers for class info and sample ID, and class labels (e.g., t/f for T1D_Diagnosed)

## Outputs

- Trained Random Forest model (downloadable RDS file)
- Train error plot and test-set confusion matrix with accuracy statistics
- Accuracy-vs-parameter plot over the mtry range (1-101) and AUC-ROC plot
- Top-ten important features with variable importance
- Benchmark accuracy values (e.g., MegaR 0.6683 T2D, 0.8846 Cirrhosis vs MetAML 0.6640, 0.8770) for accuracy_comparison.csv
- Phenotype predictions for unknown samples

## How to apply

Preprocess the merged MetaPhlAn table with MegaR's getGoodfeature(alltable2, threshold, samplePercent, normval) after selecting a taxonomic level (Genus/Species/All Level); use the default 0.003 abundance threshold, 5% Percentage-of-Sample filter, and NO normalization unless the data warrant CSS/Quantile/TMM. Read the metadata with readmetadata, supplying the column numbers for class info and sample ID (IDs must match the profile table exactly), and train a Random Forest with gettrainingdonerf, sweeping the number of variables at split (mtry default range 1-101) and using a 90% training split for the MetAML benchmark comparison. Validate with N independent runs of 10-fold cross validation (validation(Num, modelclas, ...)), since k-fold CV on small datasets has high variance; RF is preferred because removing sparse features and RF tuning consistently improved accuracy. Record the best cross-validation accuracy and confusion-matrix statistics, and optionally download the RDS model for unknown-sample prediction.

## Related tools

- **MegaR** (R Shiny package providing preprocessing (getGoodfeature), RF model training (gettrainingdonerf), k-fold cross validation, and unknown-sample prediction) — https://github.com/BioHPC/MegaR
- **R** (Language in which MegaR is implemented and invoked (library(MegaR); MegaR()))
- **R-Shiny** (Interactive web interface exposing model building, tuning, and cross-validation to the user)
- **MetAML** (Reference classifier against which MegaR RF accuracies are benchmarked on the T2D and Cirrhosis datasets)

## Examples

```
devtools::install_github("BioHPC/MegaR"); library(MegaR); MegaR()
```

## Evaluation signals

- Reproduced MegaR 10-fold CV accuracy meets or slightly exceeds MetAML baselines: ~0.6683 vs 0.6640 (T2D) and ~0.8846 vs 0.8770 (Cirrhosis)
- Sample IDs in the metadata file match the profile-table sample IDs exactly before training
- Sparse features removed with the 0.003 threshold and 5% Percentage-of-Sample filter yield improved accuracy over unfiltered data
- Best accuracy over the mtry sweep (default 1-101) is reported with confusion-matrix statistics on a held-out test split
- Cross-validation variance is minimized by using N independent runs of 10-fold CV (e.g., N=3 in the T1D example)

## Limitations

- Exact split criteria used for the final published results are not stated; various split criteria gave various accuracies depending on the data
- Other machine learning classifiers and deep learning methods were not yet tested (planned future work)
- The article/README report no quantitative accuracy values for some comparisons, and no specific datasets/accessions are named for the RF-vs-SVM/GLM claim beyond the MetAML benchmark
- Training on 100% of the data leaves no test set for confusion-matrix generation

## Evidence

- [discussion] RF model provides highest accuracy in most metagenomic classification scenarios compared to SVM and GLM: "Our results indicate that the RF model provides the highest accuracy in most metagenomic classification scenarios compared to SVM and GLM."
- [methods] Default threshold 0.003 for removing low-abundance profiles: "This field is getting a floating number to remove profiles and their abundances below the threshold value. Default value is **0.003**."
- [methods] getGoodfeature preprocessing function signature: "- `getGoodfeature(alltable2, threshold, samplePercent, normval)`"
- [readme] N independent runs of 10-fold cross validation minimize variance: "MegaR provides users to select N independent runs of the 10-fold cross validation to minimize such a high variance."
- [readme] Tuning the number of predictors at split for Random Forest: "For the random forest model, a user can select the number of predictors to be used during each split."

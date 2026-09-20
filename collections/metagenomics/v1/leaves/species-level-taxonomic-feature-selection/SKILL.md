---
name: species-level-taxonomic-feature-selection
description: Use when training MegaR classifiers (RF, SVM, GLM) on a merged MetaPhlAn
  or QIIME OTU/BIOM table with species-resolution taxonomy and maximum prediction
  accuracy is needed, e.g. DIABIMMUNE T1D WGS/16S profiles.
when_to_use_negative:
- Input profile lacks species-resolution taxonomy (only genus or higher ranks available)
  — use the Genus Level or All Level tabs instead
- The goal is tracking unclassified higher-order taxa — the All Level tab is designed
  to track back the taxon level for unclassified higher order
- Data are not taxonomic profiles (e.g. raw reads or functional profiles), since MegaR's
  preprocessing operates on MetaPhlAn/QIIME taxon tables
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: MegaR
  role: R Shiny package providing the Preprocess tab (Species Level feature selection)
    and getGoodfeature filtering for metagenomic classification
  repo: https://github.com/BioHPC/MegaR
- name: randomForest
  role: Classifier trained on the species-level filtered feature table (e.g. via gettrainingdonerf
    with mtry candidates 41 and 81)
- name: caret
  role: Supports the train/test split and model evaluation workflow in MegaR
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1186/s12859-020-03933-4
    title: 'MegaR: an interactive R package for rapid sample classification and phenotype
      prediction using metagenome profiles and machine learning'
  source_content_hash: sha256:0ad2ee12660039c23d932a0fd10f633c7c8479b66b986e894bb716b018ce4121
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# species-level-taxonomic-feature-selection

## Summary

In MegaR's preprocessing tab, select the Species Level feature set (rather than Genus or All Level) from a MetaPhlAn/QIIME taxonomic profile, then apply abundance threshold (0.003), percentage-of-sample (5%), and normalization filters to build a feature table for classifiers. Species-level features were reported to yield roughly tenfold better classification accuracy than genus-level features on the DIABIMMUNE T1D data.

## When to use

Use when training MegaR classifiers (RF, SVM, GLM) on a merged MetaPhlAn or QIIME OTU/BIOM table with species-resolution taxonomy and maximum prediction accuracy is needed, e.g. DIABIMMUNE T1D WGS/16S profiles.

## When NOT to use

- Input profile lacks species-resolution taxonomy (only genus or higher ranks available) — use the Genus Level or All Level tabs instead
- The goal is tracking unclassified higher-order taxa — the All Level tab is designed to track back the taxon level for unclassified higher order
- Data are not taxonomic profiles (e.g. raw reads or functional profiles), since MegaR's preprocessing operates on MetaPhlAn/QIIME taxon tables

## Inputs

- Merged MetaPhlAn species-level table (e.g. merged_metaphlan_t1d.txt from DIABIMMUNE T1D WGS data)
- QIIME OTU/BIOM table with taxonomic ranks (16S rRNA data)
- metadata.tsv with sample ID column (e.g. column 2, Gid_shotgun) and class column (e.g. column 7, T1D_Diagnosed)

## Outputs

- Species-level filtered feature table (threshold 0.003, 5% of samples, chosen normalization)
- Trained classifier (e.g. random forest RDS model) whose features are species taxa
- Test-set confusion matrix / accuracy enabling genus-vs-species comparison (e.g. accuracy_comparison.tsv)

## How to apply

Load the taxonomic profile (e.g. merged_metaphlan_t1d.txt) via MegaR's readmydata, then in the Preprocess tab click the Species Level tab so genus and species level rows from the dataset become the features (Genus Level and All Level are the alternatives). Follow with getGoodfeature using the default abundance threshold of 0.003, the 5% percentage-of-sample slider, and NO normalization for the T1D experiment — removing sparse, low-abundance features is reported to increase model accuracy. Train the model (e.g. random forest via gettrainingdonerf with 80/20 split and mtry candidates 41 and 81) on this species-level table using column 7 (T1D_Diagnosed) as class and column 2 (Gid_shotgun) as sample ID. Judge correctness by comparing held-out test accuracy against a genus-level model trained identically: the species-level model should be substantially (~tenfold) more accurate, confirming the choice to use species features for all subsequent analyses.

## Related tools

- **MegaR** (R Shiny package providing the Preprocess tab (Species Level feature selection) and getGoodfeature filtering for metagenomic classification) — https://github.com/BioHPC/MegaR
- **randomForest** (Classifier trained on the species-level filtered feature table (e.g. via gettrainingdonerf with mtry candidates 41 and 81))
- **caret** (Supports the train/test split and model evaluation workflow in MegaR)

## Examples

```
getGoodfeature(alltable2, threshold = 0.003, samplePercent = 5, normval = "NO")
```

## Evaluation signals

- Held-out test accuracy from getconfuMat for the species-level model should exceed the identically trained genus-level model (reported ~tenfold better on DIABIMMUNE T1D)
- Feature table should contain species-level taxon rows (e.g. merged_metaphlan_t1d.txt features) after filtering with threshold 0.003 and the 5% percentage-of-sample slider
- 3 runs of 10-fold cross validation (validation()) should show the species-level model is not overfitted
- RF accuracy on the species-level T1D WGS model should fall in the ~0.9 range (e.g. 0.9385) when mtry candidates 41 and 81 are used with an 80/20 split
- Metadata column mapping is correct: class = column 7 (T1D_Diagnosed), sample ID = column 2 (Gid_shotgun), with sample IDs matching the MetaPhlAn table exactly

## Limitations

- The paper does not report quantitative accuracy values in the discussion; the ~tenfold species-vs-genus improvement comes from the authors' experiments, and 'depending on the data, obtained various accuracies' across split criteria
- Species-level selection increases feature dimensionality relative to genus level, so removing sparse features (low-abundance threshold, percentage-of-sample filter) is required to improve accuracy
- Exact split criteria for final reported results are not stated in the article, so accuracy may vary with the train/test split

## Evidence

- [methods] Genus/Species Level tabs: "**Genus Level** and **Species Level** tabs return genus and species level from the dataset as the feature."
- [methods] preprocess tab feature selection: "Click preprocess tab and select the appropriate taxonomic level information to use for machine learning."
- [discussion] sparse feature removal improves accuracy: "removing sparse features expressed in low numbers increased the machine learning model accuracy"
- [readme] T1D experiment species selection: "* Clicking **Species Level** for feature selection."
- [methods] getGoodfeature signature: "- `getGoodfeature(alltable2, threshold, samplePercent, normval)`"

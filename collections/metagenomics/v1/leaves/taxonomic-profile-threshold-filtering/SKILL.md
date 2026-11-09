---
name: taxonomic-profile-threshold-filtering
description: Use when a merged MetaPhlAn table (e.g. merged_metaphlan_t1d.txt) or
  QIIME OTU/BIOM table must be filtered to informative features before training GLM,
  SVM, or RF classifiers in MegaR.
when_to_use_negative:
- Input is already a filtered feature table or non-taxonomic data (e.g. raw sequencing
  reads or gene-count matrices not produced by MetaPhlAn/QIIME)
- The goal is visualization or differential-abundance analysis (e.g. MetaDprof/MetaLonDa
  tasks) rather than building classification models
- You need to keep all taxa including very sparse features, e.g. when low-abundance
  taxa are themselves the biological signal of interest — threshold filtering may
  remove them
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_2269
tools:
- name: MegaR
  role: Interactive R Shiny package whose preprocess tab implements level selection,
    Threshold, Percentage of Sample filtering, and normalization (getLevelData, getGoodfeature)
  repo: https://github.com/BioHPC/MegaR
- name: R
  role: Runtime environment in which MegaR is installed and executed (devtools::install_github)
  repo: https://github.com/BioHPC/MegaR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-020-03933-4
    title: 'MegaR: an interactive R package for rapid sample classification and phenotype
      prediction using metagenome profiles and machine learning'
  source_content_hash: sha256:fed007f6846b28c243fdc064b03220183ff00b06893d3041325dd88accd4023c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioHPC/MegaR
  tools:
  - MegaR
  - R
  - caret
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# taxonomic-profile-threshold-filtering

## Summary

Preprocesses merged MetaPhlAn or QIIME OTU/BIOM taxonomic profiles in MegaR by selecting a taxonomic level, removing low-abundance features below a floating threshold (default 0.003), applying a percentage-of-sample filter (default 5%), and optionally normalizing, to boost downstream machine learning accuracy.

## When to use

Use when a merged MetaPhlAn table (e.g. merged_metaphlan_t1d.txt) or QIIME OTU/BIOM table must be filtered to informative features before training GLM, SVM, or RF classifiers in MegaR.

## When NOT to use

- Input is already a filtered feature table or non-taxonomic data (e.g. raw sequencing reads or gene-count matrices not produced by MetaPhlAn/QIIME)
- The goal is visualization or differential-abundance analysis (e.g. MetaDprof/MetaLonDa tasks) rather than building classification models
- You need to keep all taxa including very sparse features, e.g. when low-abundance taxa are themselves the biological signal of interest — threshold filtering may remove them

## Inputs

- Merged MetaPhlAn taxonomic profile table (e.g. merged_metaphlan_t1d.txt)
- QIIME OTU table or BIOM file
- Taxonomic level choice (Species/Genus/All)
- Threshold floating value (default 0.003)
- Percentage of Sample value (default 5%)
- Normalization choice (CSS, Quantile, TMM, or NO)

## Outputs

- Filtered feature table restricted to the selected taxonomic level and passing threshold/percentage-of-sample criteria
- Normalized (or unnormalized, if NO) processed data shown under the MegaR Data tab, ready for machine learning model building

## How to apply

Load the input profile into MegaR (readmydata), then click the preprocess tab and choose the taxonomic feature level — Species Level, Genus Level, or All Level (which tracks back the taxon level for unclassified higher-order taxa). Set the Threshold floating value to remove profiles and their abundances below the cutoff (default 0.003); the README for the DIABIMMUNE T1D experiment says to leave the default value. Adjust the Percentage of Sample slider (default 5%) so only features containing the threshold abundance in that fraction of samples are kept — removing sparse features expressed in low numbers increases machine learning model accuracy. Finally choose a normalization from CSS, Quantile, TMM, or NO; for the T1D WGS experiment select NO, and the processed data ready for model building appears under the Data tab (functions getLevelData and getGoodfeature implement these steps).

## Related tools

- **MegaR** (Interactive R Shiny package whose preprocess tab implements level selection, Threshold, Percentage of Sample filtering, and normalization (getLevelData, getGoodfeature)) — https://github.com/BioHPC/MegaR
- **R** (Runtime environment in which MegaR is installed and executed (devtools::install_github)) — https://github.com/BioHPC/MegaR

## Examples

```
getGoodfeature(alltable2, threshold, samplePercent, normval)
```

## Evaluation signals

- The processed data tab shows a reduced feature set: all retained features have abundances at or above the chosen threshold (e.g. 0.003) in at least the selected percentage of samples (e.g. 5%)
- Feature set matches the requested taxonomic level (species-level features for the T1D experiment)
- Downstream model accuracy improves relative to unfiltered data — the article reports that removing sparse features expressed in low numbers increased machine learning model accuracy
- Normalization setting matches the experiment plan (NO for the T1D experiment); no unexpected scale changes in abundance values
- Sample IDs in the filtered table still match the metadata sample IDs exactly, as required for class assignment

## Limitations

- Overly aggressive threshold or percentage-of-sample settings can discard informative sparse taxa; accuracy depends on tuning these for the specific dataset
- Cross-validated accuracy varies with split criteria, so filtered-feature performance is data-dependent
- Only four normalization options are available (CSS, Quantile, TMM, NO); other normalization schemes are not supported

## Evidence

- [methods] This field is getting a floating number to remove profiles and their abundances below the threshold value. Default value is 0.003.: "This field is getting a floating number to remove profiles and their abundances below the threshold value. Default value is **0.003**."
- [methods] The Percentage of Sample slider bar can be adjusted to select the percentage of sample that should include the threshold amount of abundance present in the data.: "The **Percentage of Sample** slider bar can be adjusted to select the percentage of sample that should include the threshold amount of abundance present in the data."
- [discussion] removing sparse features expressed in low numbers increased the machine learning model accuracy: "removing sparse features expressed in low numbers increased the machine learning model accuracy"
- [readme] Clicking Species Level for feature selection... For the T1D experiment, select NO (normalization).: "* Clicking **Species Level** for feature selection."
- [methods] Genus Level and Species Level tabs return genus and species level from the dataset as the feature. All Level tab tracks back the taxon level for unclassified higher order.: "**Genus Level** and **Species Level** tabs return genus and species level from the dataset as the feature. **All Level** tab tracks back the taxon level for unclassified higher order."

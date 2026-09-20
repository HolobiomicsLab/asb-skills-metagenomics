---
name: sparse-feature-threshold-filtering
description: Use when a MetaPhlAn/QIIME OTU or BIOM table must be reduced to reliable
  features (e.g., genus or species level) prior to training GLM, random forest, or
  SVM classifiers in MegaR.
when_to_use_negative:
- The input is already a preprocessed feature table at the desired taxonomic level
  with sparse features removed
- The analysis goal is a descriptive taxonomic profile summary rather than features
  for machine learning classification
- Legitimately rare but biologically important taxa must be retained, since threshold/percentage
  filtering removes profiles below 0.003 abundance or present in fewer than ~5% of
  samples
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3474
tools:
- name: MegaR
  role: R Shiny package providing the Preprocess tab and the getGoodfeature function
    (alltable2, threshold, samplePercent, normval) that performs taxonomic-level selection,
    threshold filtering, percentage-of-sample filtering, and normalization
  repo: https://github.com/BioHPC/MegaR
- name: R
  role: Runtime environment for installing and running MegaR via devtools::install_github
  repo: https://github.com/BioHPC/MegaR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-020-03933-4
    title: 'MegaR: an interactive R package for rapid sample classification and phenotype
      prediction using metagenome profiles and machine learning'
  source_content_hash: sha256:29e729b3cb04e7b31a30fb6cbb4c05f1278d1f9342ce1ffd1ec0dbaea8ce6698
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioHPC/MegaR
  tools:
  - MegaR
  - R
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# sparse-feature-threshold-filtering

## Summary

Preprocess metagenomic taxonomic profile tables in MegaR by selecting a taxonomic level, then removing low-abundance profiles below an abundance threshold and features present in too few samples before machine learning model building. Removing sparse features expressed in low numbers was reported to increase model accuracy.

## When to use

Use when a MetaPhlAn/QIIME OTU or BIOM table must be reduced to reliable features (e.g., genus or species level) prior to training GLM, random forest, or SVM classifiers in MegaR.

## When NOT to use

- The input is already a preprocessed feature table at the desired taxonomic level with sparse features removed
- The analysis goal is a descriptive taxonomic profile summary rather than features for machine learning classification
- Legitimately rare but biologically important taxa must be retained, since threshold/percentage filtering removes profiles below 0.003 abundance or present in fewer than ~5% of samples

## Inputs

- MetaPhlAn merged taxonomic profile table (e.g., merged_metaphlan_t1d.txt) or QIIME OTU/BIOM file
- Selected taxonomic level (Genus Level, Species Level, or All Level)
- Threshold floating value (default 0.003)
- Percentage of Sample slider value (default 5%)
- Normalization choice (CSS, Quantile, TMM, or NO)

## Outputs

- Preprocessed feature table restricted to the chosen taxonomic level, with sparse/low-abundance features removed, displayed under the Data tab and ready for model building

## How to apply

In MegaR's preprocess tab (or via getGoodfeature), first select the taxonomic level — Genus Level, Species Level, or All Level (All Level tracks back the taxon level for unclassified higher order). Then set the abundance Threshold, a floating number that removes profiles with abundances below the cutoff (default 0.003; the README's T1D walkthrough notes leaving the default, written as 0.03). Next adjust the Percentage of Sample slider so that only features containing the threshold abundance in at least that percentage of samples (default 5%) are retained. Finally choose a normalization from Cumulative Sum Scaling (CSS), Quantile, Trimmed Mean of M-values (TMM), or NO — for the DIABIMMUNE T1D experiment, NO normalization is used. The processed table shown under the Data tab is then ready for model development.

## Related tools

- **MegaR** (R Shiny package providing the Preprocess tab and the getGoodfeature function (alltable2, threshold, samplePercent, normval) that performs taxonomic-level selection, threshold filtering, percentage-of-sample filtering, and normalization) — https://github.com/BioHPC/MegaR
- **R** (Runtime environment for installing and running MegaR via devtools::install_github) — https://github.com/BioHPC/MegaR

## Examples

```
getGoodfeature(alltable2, threshold, samplePercent, normval)
```

## Evaluation signals

- The processed table under the Data tab contains only features above the threshold abundance in at least the selected percentage of samples (e.g., 5%)
- All retained features are at the selected taxonomic level (genus or species) as returned by the level tabs
- The chosen normalization (CSS, Quantile, TMM, or NO) is reflected in the transformed abundances of the processed table
- Downstream model accuracy improves relative to an unfiltered model, consistent with the article's claim that removing sparse features increased accuracy

## Limitations

- The README's T1D example is internally inconsistent about the default threshold value (stated default is 0.003 in the article, but the walkthrough text says 'leave the the deafult value (0.03)')
- Only four normalization options are offered (CSS, Quantile, TMM, NO); other schemes are not supported
- Excessive filtering can remove informative low-abundance taxa, and the article notes that various preprocessing choices yield various accuracies depending on the data

## Evidence

- [discussion] removing sparse features expressed in low numbers increased the machine learning model accuracy: "removing sparse features expressed in low numbers increased the machine learning model accuracy"
- [methods] threshold removes profiles below cutoff: "This field is getting a floating number to remove profiles and their abundances below the threshold value. Default value is 0.003."
- [methods] percentage of sample slider: "The **Percentage of Sample** slider bar can be adjusted to select the percentage of sample that should include the threshold amount of abundance present in the data."
- [readme] normalization choices: "MegaR provides fours choices for data normalization: **Cumulative Sum Scaling (CSS)**, **Quantile**, **Trimmed Mean of M-values (TMM)**, and **NO**."
- [methods] getGoodfeature signature: "- `getGoodfeature(alltable2, threshold, samplePercent, normval)`"

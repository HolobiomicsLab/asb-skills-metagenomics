---
name: low-abundance-feature-threshold-filtering
description: Use when a merged MetaPhlAn (or QIIME/BIOM) taxonomic profile table must
  be preprocessed for ML classification and sparse low-abundance taxa need removal
  prior to RF/SVM/GLM model building.
when_to_use_negative:
- The input is already a preprocessed feature table with sparse taxa removed (MegaR
  expects the raw merged MetaPhlAn/QIIME/BIOM table).
- The goal is differential abundance testing or temporal trend detection (tools like
  MetaDprof or MetaLonDa), not classification-oriented feature filtering.
- The analysis requires retaining rare taxa as features, e.g. when low-abundance biomarkers
  are biologically relevant rather than noise.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_2269
tools:
- name: MegaR
  role: R Shiny package providing the preprocess tab and getGoodfeature() to apply
    taxonomic-level selection, threshold, percentage-of-sample, and normalization
    filtering
  repo: https://github.com/BioHPC/MegaR
- name: R
  role: runtime environment in which MegaR's filtering functions (getLevelData, getGoodfeature)
    are executed
  repo: https://github.com/BioHPC/MegaR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-020-03933-4
    title: 'MegaR: an interactive R package for rapid sample classification and phenotype
      prediction using metagenome profiles and machine learning'
  source_content_hash: sha256:0e7b70870e62cfe77f6fca51e7d0fa633ae6848de16c65dedc6e38c0c83f69cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioHPC/MegaR
  tools:
  - MegaR
  - MetAML
  - R
  - R-Shiny
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# low-abundance-feature-threshold-filtering

## Summary

Removes sparse, low-abundance taxonomic features from MetaPhlAn/QIIME profile tables in MegaR before machine learning, using a floating abundance threshold (default 0.003) plus a percentage-of-sample filter (default 5%), which the authors report increases model accuracy.

## When to use

Use when a merged MetaPhlAn (or QIIME/BIOM) taxonomic profile table must be preprocessed for ML classification and sparse low-abundance taxa need removal prior to RF/SVM/GLM model building.

## When NOT to use

- The input is already a preprocessed feature table with sparse taxa removed (MegaR expects the raw merged MetaPhlAn/QIIME/BIOM table).
- The goal is differential abundance testing or temporal trend detection (tools like MetaDprof or MetaLonDa), not classification-oriented feature filtering.
- The analysis requires retaining rare taxa as features, e.g. when low-abundance biomarkers are biologically relevant rather than noise.

## Inputs

- merged MetaPhlAn taxonomic profile table (e.g. merged_metaphlan_t1d.txt)
- QIIME OTU table or BIOM file
- taxonomic level selection (Genus / Species / All Level)
- floating threshold value (default 0.003)
- percentage-of-sample value (default 5%)
- normalization choice: CSS, Quantile, TMM, or NO

## Outputs

- processed feature table containing only features passing the abundance threshold and sample-percentage filter, ready for ML model building

## How to apply

Load the merged MetaPhlAn table into MegaR (R Shiny) and select the taxonomic level (Genus, Species, or All Level) under the preprocess tab. Apply the Threshold filter with a floating cutoff — the default is 0.003, and the MetAML benchmark (T2D, liver cirrhosis) also used 0.003 — to remove profiles whose abundances fall below it. Adjust the Percentage of Sample slider (default 5%) so features must exceed the threshold in at least that share of samples; the benchmark used the default 5%. Choose a normalization (CSS, Quantile, TMM, or NO) — the MetAML benchmark used NO — and the processed table is returned by getGoodfeature(alltable2, threshold, samplePercent, normval), ready for getLevelData feature extraction and downstream model training. Rationale: the paper states that removing sparse features expressed in low numbers increased machine learning model accuracy.

## Related tools

- **MegaR** (R Shiny package providing the preprocess tab and getGoodfeature() to apply taxonomic-level selection, threshold, percentage-of-sample, and normalization filtering) — https://github.com/BioHPC/MegaR
- **R** (runtime environment in which MegaR's filtering functions (getLevelData, getGoodfeature) are executed) — https://github.com/BioHPC/MegaR

## Examples

```
getGoodfeature(alltable2, threshold=0.003, samplePercent=5, normval="NO")
```

## Evaluation signals

- Feature table row count decreases relative to the full merged MetaPhlAn table after applying threshold=0.003 and 5% sample filter.
- No retained feature has abundance below 0.003 in more than 95% of samples (the percentage-of-sample criterion).
- Cross-validation accuracy of the RF model on the filtered table meets or slightly exceeds the unfiltered baseline, consistent with the finding that removing sparse features increased accuracy.
- Benchmarking check: with threshold 0.003, 5% sample filter, and NO normalization, MegaR RF accuracy is approximately 0.6683 on T2D and 0.8846 on liver cirrhosis.
- Downstream model training (e.g. gettrainingdonerf) runs without errors on the filtered table, confirming a valid feature matrix.

## Limitations

- Optimal threshold and split criteria vary by dataset: 'we tested various split criteria and, depending on the data, obtained various accuracies'.
- The README's T1D walkthrough contains an inconsistency, mentioning the default 0.003 but then instructing users to 'leave the the deafult value (0.03)'.
- Only data preprocessing filters are covered; other classifiers and deep learning methods were not yet tested by the authors.
- No quantitative accuracy values are reported in the discussion for how filtering accuracy varies with thresholds.

## Evidence

- [methods] Threshold filtering of low-abundance profiles: "This field is getting a floating number to remove profiles and their abundances below the threshold value. Default value is **0.003**."
- [methods] Percentage of Sample filtering: "The **Percentage of Sample** slider bar can be adjusted to select the percentage of sample that should include the threshold amount of abundance present in the data."
- [methods] Normalization options: "MegaR provides fours choices for data normalization: **Cumulative Sum Scaling (CSS)**, **Quantile**, **Trimmed Mean of M-values (TMM)**, and **NO**."
- [discussion] Sparse feature removal improves accuracy: "removing sparse features expressed in low numbers increased the machine learning model accuracy"
- [readme] README preprocessing defaults: "This field is getting a floating number to remove profiles and their abundances below the threshold value. Default value is **0.003**."

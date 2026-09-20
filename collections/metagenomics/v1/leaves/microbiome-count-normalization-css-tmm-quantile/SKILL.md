---
name: microbiome-count-normalization-css-tmm-quantile
description: Use when a merged MetaPhlAn or QIIME OTU/BIOM taxonomy table needs rescaling
  across samples before GLM, Random Forest, or SVM model building, or when comparing
  how normalization choices affect classifier accuracy.
when_to_use_negative:
- Input is not a merged OTU/BIOM taxonomy table from MetaPhlAn or QIIME (e.g. raw
  sequencing reads or metadata files).
- The data have already been normalized upstream and re-scaling would distort the
  counts before model building.
- The goal is differential-abundance testing rather than preparing a feature matrix
  for machine learning classification.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: MegaR
  role: R Shiny package providing the preprocess tab, Threshold/Percentage-of-Sample
    filters, and the four CSS/Quantile/TMM/NO normalization choices
  repo: https://github.com/BioHPC/MegaR
- name: R
  role: Runtime environment in which MegaR (devtools::install_github("BioHPC/MegaR"))
    is installed and launched via library(MegaR); MegaR()
  repo: https://github.com/BioHPC/MegaR
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-020-03933-4
    title: 'MegaR: an interactive R package for rapid sample classification and phenotype
      prediction using metagenome profiles and machine learning'
  source_content_hash: sha256:c280481e0b6751f6f163cdb090f337dc42d2c8b2b48ae4b29f1fe15fb769ce51
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioHPC/MegaR
  tools:
  - MegaR
  - R
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# microbiome-count-normalization-css-tmm-quantile

## Summary

Normalize aggregated metagenomic count tables in MegaR using one of four scaling choices — Cumulative Sum Scaling (CSS), Quantile, Trimmed Mean of M-values (TMM), or NO normalization — after taxonomic-level extraction and low-abundance filtering, to prepare comparable feature matrices for machine learning models.

## When to use

Use when a merged MetaPhlAn or QIIME OTU/BIOM taxonomy table needs rescaling across samples before GLM, Random Forest, or SVM model building, or when comparing how normalization choices affect classifier accuracy.

## When NOT to use

- Input is not a merged OTU/BIOM taxonomy table from MetaPhlAn or QIIME (e.g. raw sequencing reads or metadata files).
- The data have already been normalized upstream and re-scaling would distort the counts before model building.
- The goal is differential-abundance testing rather than preparing a feature matrix for machine learning classification.

## Inputs

- Merged MetaPhlAn taxonomy table (e.g. merged_metaphlan_t1d.txt) or QIIME OTU/BIOM file
- Taxonomic level choice (Genus Level, Species Level, or All Level tab)
- Threshold floating number (default 0.003)
- Percentage of Sample slider value (default 5%)
- Normalization choice: CSS, Quantile, TMM, or NO

## Outputs

- Processed, normalization-ready abundance/feature table displayed under the Data tab, suitable for Random Forest, GLM, or SVM model building
- Per-normalization CSV count matrices when benchmarking all four variants (e.g. t1d_species_css.csv, t1d_species_quantile.csv, t1d_species_tmm.csv, t1d_species_none.csv)

## How to apply

After uploading the merged OTU/BIOM file (e.g. merged_metaphlan_t1d.txt) and extracting features at the desired taxonomic level via MegaR's preprocess tab, first apply the Threshold filter (default 0.003) to remove profiles with abundances below that value, then set the Percentage of Sample slider (default 5%) so only microbes exceeding the threshold in at least that fraction of samples are retained. Next, choose one of the four normalization options in the Normalization field: Cumulative Sum Scaling (CSS), Quantile, Trimmed Mean of M-values (TMM), or NO. The paper's getGoodfeature(alltable2, threshold, samplePercent, normval) function couples this filtering and normalization; to benchmark all four options, run the filtering-plus-normalization step four times with normval set successively to CSS, Quantile, TMM, and NO, writing each processed matrix to its own CSV (e.g. t1d_species_css.csv, t1d_species_quantile.csv, t1d_species_tmm.csv, t1d_species_none.csv). For the T1D experiment the README recommends selecting NO, and the discussion notes that removing sparse features expressed in low numbers increased machine learning model accuracy.

## Related tools

- **MegaR** (R Shiny package providing the preprocess tab, Threshold/Percentage-of-Sample filters, and the four CSS/Quantile/TMM/NO normalization choices) — https://github.com/BioHPC/MegaR
- **R** (Runtime environment in which MegaR (devtools::install_github("BioHPC/MegaR")) is installed and launched via library(MegaR); MegaR()) — https://github.com/BioHPC/MegaR

## Examples

```
getGoodfeature(alltable2, threshold=0.003, samplePercent=5, normval="CSS")
```

## Evaluation signals

- Each of the four output CSV files (t1d_species_css.csv, t1d_species_quantile.csv, t1d_species_tmm.csv, t1d_species_none.csv) exists and is parseable as an abundance table.
- The no-normalization variant matches applying only the 0.003 threshold and 5% sample filters without any rescaling of abundances.
- All samples below the Percentage-of-Sample cutoff have been dropped, so the retained features exceed the threshold abundance in at least the chosen percent of samples.
- The processed table appears under the Data tab after normalization, confirming MegaR registered the preprocessing step.
- Cross-validated model accuracy is compared across the CSS/Quantile/TMM/NO variants to confirm which normalization maximizes prediction accuracy for the dataset.

## Limitations

- MegaR offers exactly four normalization choices (CSS, Quantile, TMM, NO); other scaling schemes (e.g. DESeq2 size factors, rarefaction) are not available.
- Normalization interacts with the fixed Threshold (default 0.003) and Percentage of Sample (default 5%) filters, so changing one requires re-running the getGoodfeature step to keep comparisons fair.
- The paper reports no quantitative accuracy values for the normalization comparisons, so the best choice must be determined empirically per dataset via cross validation.
- Applying normalization to already sparse features removed by the threshold filters is not meaningful; filtering must precede normalization.

## Evidence

- [methods] Four normalization choices exist: "MegaR provides fours choices for data normalization: **Cumulative Sum Scaling (CSS)**, **Quantile**, **Trimmed Mean of M-values (TMM)**, and **NO**."
- [methods] Threshold filter default: "This field is getting a floating number to remove profiles and their abundances below the threshold value. Default value is **0.003**."
- [methods] Percentage of Sample default: "The **Percentage of Sample** slider bar can be adjusted to select the percentage of sample that should include the threshold amount of abundance present in the data. Default value is **5%**."
- [methods] getGoodfeature signature: "- `getGoodfeature(alltable2, threshold, samplePercent, normval)`"
- [discussion] Sparse feature removal improves accuracy: "removing sparse features expressed in low numbers increased the machine learning model accuracy"
- [readme] T1D experiment normalization choice: "For the T1D experiment, select **NO**."

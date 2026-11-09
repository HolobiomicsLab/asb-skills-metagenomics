---
name: r-function-pipeline-execution
description: Use when the analysis plan is expressed as a sequence of named R package
  functions with specific parameters (e.g. MegaR's getLevelData/getGoodfeature) and
  the deliverables are per-run output files written from an R session.
when_to_use_negative:
- The input is already a preprocessed, normalized feature table ready for model building
  — re-running getGoodfeature would re-filter already-thresholded data.
- The pipeline must run on non-R tooling or as a standalone GUI session only; this
  skill is specifically for chaining R package functions and producing per-run CSVs.
- Only a single taxonomic level or single normalization is needed — the four-way normalization
  sweep and per-run CSV writing add unnecessary work.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: MegaR
  role: R Shiny package whose exported functions (getLevelData, getGoodfeature) and
    app are chained to normalize and filter merged metagenomic count tables
  repo: https://github.com/BioHPC/MegaR
- name: R
  role: Execution environment (R >3.6.0, RStudio) in which the package is installed
    via devtools and the function pipeline is run
- name: devtools
  role: R library used to install MegaR directly from GitHub before running the pipeline
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-020-03933-4
    title: 'MegaR: an interactive R package for rapid sample classification and phenotype
      prediction using metagenome profiles and machine learning'
  source_content_hash: sha256:58b055146230c061467764fb2b13e5d86b0a9c3d4dc8fb41dabb1c86b3bc208f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioHPC/MegaR
  tools:
  - MegaR
  - R
  - devtools
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# r-function-pipeline-execution

## Summary

Executes a multi-step R function pipeline (installing an R package via devtools, loading it, and chaining its exported functions like getLevelData and getGoodfeature) to run a metagenomic preprocessing workflow inside an R/RStudio environment.

## When to use

Use when the analysis plan is expressed as a sequence of named R package functions with specific parameters (e.g. MegaR's getLevelData/getGoodfeature) and the deliverables are per-run output files written from an R session.

## When NOT to use

- The input is already a preprocessed, normalized feature table ready for model building — re-running getGoodfeature would re-filter already-thresholded data.
- The pipeline must run on non-R tooling or as a standalone GUI session only; this skill is specifically for chaining R package functions and producing per-run CSVs.
- Only a single taxonomic level or single normalization is needed — the four-way normalization sweep and per-run CSV writing add unnecessary work.

## Inputs

- Merged MetaPhlAn taxonomy table (merged_metaphlan_t1d.txt) or QIIME OTU/BIOM file
- Taxonomic level selector (species level for the T1D experiment)
- Threshold floating number (default 0.003)
- Percentage of Sample value (default 5%)
- Normalization choice normval (CSS, Quantile, TMM, or NO)

## Outputs

- t1d_species_css.csv processed abundance table
- t1d_species_quantile.csv processed abundance table
- t1d_species_tmm.csv processed abundance table
- t1d_species_none.csv processed abundance table (threshold/percentage filtering only, no rescaling)

## How to apply

Install the target package from GitHub with devtools::install_github("BioHPC/MegaR") or from the MegaR.tar.gz archive, then load it and launch the workflow in RStudio with library(MegaR); MegaR(). Feed the merged MetaPhlAn table merged_metaphlan_t1d.txt into getLevelData(alltable, leveld) with leveld set to species level, then apply getGoodfeature(alltable2, threshold, samplePercent, normval) with the default threshold 0.003 and default Percentage of Sample of 5%. Run the filtering-plus-normalization step four times, setting normval successively to CSS, Quantile, TMM, and NO normalization, writing each processed count matrix to its own CSV (t1d_species_css.csv, t1d_species_quantile.csv, t1d_species_tmm.csv, t1d_species_none.csv). The rationale is that removing sparse features expressed in low numbers increases machine learning model accuracy, so the same filtering (threshold + sample percentage) is held constant across all four normalization arms for a controlled comparison.

## Related tools

- **MegaR** (R Shiny package whose exported functions (getLevelData, getGoodfeature) and app are chained to normalize and filter merged metagenomic count tables) — https://github.com/BioHPC/MegaR
- **R** (Execution environment (R >3.6.0, RStudio) in which the package is installed via devtools and the function pipeline is run)
- **devtools** (R library used to install MegaR directly from GitHub before running the pipeline)

## Examples

```
library(MegaR); alltable2 <- getLevelData(alltable, "species"); good <- getGoodfeature(alltable2, 0.003, 5, "CSS")
```

## Evaluation signals

- Each of the four expected output CSVs (t1d_species_css.csv, t1d_species_quantile.csv, t1d_species_tmm.csv, t1d_species_none.csv) exists and parses as an abundance table.
- The no-normalization (NO) output matches applying only the 0.003 threshold and 5% sample-percentage filters without rescaling.
- Species-level extraction via getLevelData yields only species-level features from the merged MetaPhlAn table.
- The default threshold (0.003) and default Percentage of Sample (5%) parameter values are correctly propagated to getGoodfeature in every normalization run.
- MegaR installs cleanly via devtools::install_github and library(MegaR) loads without dependency errors.

## Limitations

- The README's own T1D walkthrough is internally inconsistent about the default threshold (methods text states 0.003 while the walkthrough says 'leave the the deafult value (0.03)'), so the intended default should be verified against the source code.
- The paper does not report quantitative accuracy values for the normalization comparisons, so correctness of the four-arm sweep cannot be judged against published metrics.
- Dependency installation relies on devtools pulling shiny, shinythemes, randomForest, stringr, caret, plyr, ggplot2, RColorBrewer, and DT; failures in any of these break the pipeline.

## Evidence

- [methods] MegaR provides fours choices for data normalization: CSS, Quantile, TMM, and NO.: "MegaR provides fours choices for data normalization: **Cumulative Sum Scaling (CSS)**, **Quantile**, **Trimmed Mean of M-values (TMM)**, and **NO**."
- [methods] getGoodfeature function signature with threshold and samplePercent parameters: "- `getGoodfeature(alltable2, threshold, samplePercent, normval)`"
- [discussion] Removing sparse features increases model accuracy: "removing sparse features expressed in low numbers increased the machine learning model accuracy"
- [readme] README install command: "> devtools::install_github("BioHPC/MegaR") "
- [readme] MegaR accepts OTU and BIOM files from MetaPhlAn and QIIME: "MegaR can take both OTU table and BIOM file from popular metagenomic profiling tools"

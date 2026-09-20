---
name: taxonomic-feature-extraction-by-level
description: Use when the input is a merged MetaPhlAn or QIIME OTU/BIOM table containing
  multiple taxonomic ranks and you need a feature matrix at a specific level (genus,
  species, or all) before filtering, normalization, or model building in MegaR.
when_to_use_negative:
- The input is already a single-level feature table (e.g. a plain abundance matrix
  without full taxonomic paths) — level extraction would have nothing to collapse.
- The analysis target is higher-order/unclassified clades that All Level handles poorly,
  or the profile format is neither MetaPhlAn merged nor QIIME OTU/BIOM.
- You only need normalization or filtering on an existing level table without re-extracting
  features.
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MegaR
  role: R Shiny package providing the Genus/Species/All Level tabs and the getLevelData
    function used to extract the feature level from merged profile tables
  repo: https://github.com/BioHPC/MegaR
- name: R
  role: Runtime environment in which MegaR is installed (devtools::install_github)
    and getLevelData is invoked
  repo: https://github.com/BioHPC/MegaR
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-020-03933-4
    title: 'MegaR: an interactive R package for rapid sample classification and phenotype
      prediction using metagenome profiles and machine learning'
  source_content_hash: sha256:618617b8a9534b3b68c786d4625277ee0eebdf13e717a5a8efc4019fd50438b4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioHPC/MegaR
  tools:
  - MegaR
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# taxonomic-feature-extraction-by-level

## Summary

Extracts species-, genus-, or all-level features from a merged MetaPhlAn/OTU taxonomy table using MegaR's getLevelData and the Preprocess tabs, converting a multi-rank profile into a single-level feature table for machine learning.

## When to use

Use when the input is a merged MetaPhlAn or QIIME OTU/BIOM table containing multiple taxonomic ranks and you need a feature matrix at a specific level (genus, species, or all) before filtering, normalization, or model building in MegaR.

## When NOT to use

- The input is already a single-level feature table (e.g. a plain abundance matrix without full taxonomic paths) — level extraction would have nothing to collapse.
- The analysis target is higher-order/unclassified clades that All Level handles poorly, or the profile format is neither MetaPhlAn merged nor QIIME OTU/BIOM.
- You only need normalization or filtering on an existing level table without re-extracting features.

## Inputs

- merged MetaPhlAn taxonomy table (e.g. merged_metaphlan_t1d.txt)
- QIIME OTU table or BIOM file from metagenomic profiling

## Outputs

- species-level feature abundance table (rows = species, columns = samples)
- genus-level or all-level feature table, depending on selected tab

## How to apply

Upload the merged profile (e.g. merged_metaphlan_t1d.txt for the DIABIMMUNE T1D WGS cohort) via the browse tab and inspect it under the Data tab. In the Preprocess tab choose the taxonomic level: 'Genus Level' or 'Species Level' return genus or species rows as features, while 'All Level' tracks back the taxon level for unclassified higher-order entries. Programmatically, this corresponds to calling getLevelData(alltable, leveld) with leveld set to the desired rank. For the T1D experiment MegaR's documentation selects Species Level so that species abundances become the ML feature space; downstream filtering (threshold 0.003, 5% Percentage of Sample) and one of the four normalization choices (CSS, Quantile, TMM, NO) then operate on this extracted level table.

## Related tools

- **MegaR** (R Shiny package providing the Genus/Species/All Level tabs and the getLevelData function used to extract the feature level from merged profile tables) — https://github.com/BioHPC/MegaR
- **R** (Runtime environment in which MegaR is installed (devtools::install_github) and getLevelData is invoked) — https://github.com/BioHPC/MegaR

## Examples

```
library(MegaR); alltable2 <- getLevelData(alltable, leveld='species')
```

## Evaluation signals

- Every retained feature row carries the selected rank label (species or genus) and no higher-rank clade rows remain in the output table.
- The number of feature rows changes predictably when switching tabs: species-level extraction yields a different feature count than genus-level on the same merged_metaphlan_t1d.txt input.
- The extracted table's sample columns match the sample IDs of the original uploaded MetaPhlAn/QIIME table exactly, so metadata sample IDs can join without mismatches.
- The subsequent getGoodfeature(alltable2, threshold=0.003, samplePercent=5, normval) step runs without error on the extracted level table, confirming it is a parseable abundance matrix.

## Limitations

- Only MetaPhlAn merged tables and QIIME OTU/BIOM formats are supported; other profile formats must be converted first.
- 'All Level' tracks back the taxon level only for unclassified higher-order entries and may not produce a clean single-rank feature set for heavily unclassified profiles.
- Level choice directly changes downstream model input; the README defaults the T1D example to Species Level, so other levels require their own validation.

## Evidence

- [methods] Genus and Species Level tabs return genus and species level from the dataset as the feature.: "**Genus Level** and **Species Level** tabs return genus and species level from the dataset as the feature. **All Level** tab tracks back the taxon level for unclassified higher order."
- [methods] Click preprocess tab and select the appropriate taxonomic level information to use for machine learning.: "Click preprocess tab and select the appropriate taxonomic level information to use for machine learning."
- [readme] Clicking Species Level for feature selection.: "Clicking **Species Level** for feature selection."
- [readme] For T1D experiment, upload merged_metaphlan_t1d.txt file.: "For T1D experiment, upload **merged_metaphlan_t1d.txt** file."

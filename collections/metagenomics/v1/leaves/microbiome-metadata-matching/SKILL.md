---
name: microbiome-metadata-matching
description: Use when a merged MetaPhlAn or QIIME OTU/BIOM profile table must be paired
  with a tab-separated metadata file whose class column (e.g. column 7, T1D_Diagnosed
  with t/f labels) and sample ID column (e.g. column 2, Gid_shotgun) must be identified
  before MegaR model building.
when_to_use_negative:
- Input is already a merged feature table with class labels embedded — re-matching
  metadata would be redundant
- Only exploratory visualization of abundance profiles is needed without phenotype
  prediction, since no metadata linkage is required
- Sample IDs between the profile table and metadata cannot be reconciled (formatting
  or renaming differences) — matching must be fixed upstream first
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: MegaR
  role: R/Shiny package that ingests the metadata file, maps class and sample ID columns
    by user-supplied column numbers, and passes the matched data to model building
    and cross-validation
  repo: https://github.com/BioHPC/MegaR
- name: R
  role: Underlying language in which MegaR metadata-reading and model-training functions
    (e.g. readmetadata, gettrainingdonerf) are executed
  repo: https://github.com/BioHPC/MegaR
- name: R-Shiny
  role: Interactive web layer providing the upload widgets and column-number fields
    for metadata matching
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-020-03933-4
    title: 'MegaR: an interactive R package for rapid sample classification and phenotype
      prediction using metagenome profiles and machine learning'
  source_content_hash: sha256:67369673ec58a372ce4352cf0785e188258855668d461d449c532979926e2e89
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioHPC/MegaR
  tools:
  - MegaR
  - MetAML
  - R
  - R-Shiny
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# microbiome-metadata-matching

## Summary

Prepares and aligns tab-separated phenotype metadata (class labels and sample IDs) with MetaPhlAn2/QIIME taxonomic profile tables in MegaR so that every sample in the profile table has a matching phenotype class for machine learning model training and benchmarking.

## When to use

Use when a merged MetaPhlAn or QIIME OTU/BIOM profile table must be paired with a tab-separated metadata file whose class column (e.g. column 7, T1D_Diagnosed with t/f labels) and sample ID column (e.g. column 2, Gid_shotgun) must be identified before MegaR model building.

## When NOT to use

- Input is already a merged feature table with class labels embedded — re-matching metadata would be redundant
- Only exploratory visualization of abundance profiles is needed without phenotype prediction, since no metadata linkage is required
- Sample IDs between the profile table and metadata cannot be reconciled (formatting or renaming differences) — matching must be fixed upstream first

## Inputs

- Tab-separated metadata file with sample IDs and a class label column (e.g. metadata.tsv from the T1D dataset)
- Merged MetaPhlAn taxonomic profile table (e.g. merged_metaphlan_t1d.txt) or QIIME OTU/BIOM file already loaded in MegaR
- Column number of class info (e.g. 7) and of sample ID (e.g. 2) in the metadata file
- List of classification labels selected from the class column (e.g. 'f' and 't')

## Outputs

- Metadata-linked feature table with class assignments for each sample, ready for GLM/RF/SVM model building
- Metadata object consumed by gettrainingdonerf and validation for cross-validation

## How to apply

Obtain a tab-separated metadata file whose rows contain sample IDs and one column contains class labels; sample IDs in the metadata MUST match exactly the sample IDs in the initial MetaPhlAn/QIIME file. In MegaR (R-Shiny), upload the metadata under the Model building tab and supply the column number for class info (7 for the T1D dataset) and for sample IDs (2 for Gid_shotgun). MegaR automatically pulls the classification labels (e.g. 'f' and 't') from the specified class column, which the user confirms in the Select classification labels field. The matched table/metadata pair then feeds model building functions such as gettrainingdonerf(mytable3, classid, sampleid, ruleout, psd, metadat, mrange) and validation(Num, modelclas, ...) in the T2D/Cirrhosis benchmark. Rationale: exact ID matching is required for correct class assignment during training and cross-validation; mismatched IDs silently corrupt the confusion matrix.

## Related tools

- **MegaR** (R/Shiny package that ingests the metadata file, maps class and sample ID columns by user-supplied column numbers, and passes the matched data to model building and cross-validation) — https://github.com/BioHPC/MegaR
- **R** (Underlying language in which MegaR metadata-reading and model-training functions (e.g. readmetadata, gettrainingdonerf) are executed) — https://github.com/BioHPC/MegaR
- **R-Shiny** (Interactive web layer providing the upload widgets and column-number fields for metadata matching)

## Examples

```
devtools::install_github("BioHPC/MegaR"); library(MegaR); MegaR()  # then upload metadata.tsv, set class column 7 and sample ID column 2 in the Model building tab
```

## Evaluation signals

- Sample IDs in the metadata file match exactly (string-for-string) the sample IDs in the merged MetaPhlAn/QIIME table; any mismatch should flag a failure before training
- The class labels pulled by MegaR from the specified column (e.g. 't'/'f' from T1D_Diagnosed) cover all samples after filtering
- Model building proceeds without 'cannot find sample' errors, and the confusion matrix dimensions equal the number of selected labels
- For the MegaR-vs-MetAML benchmark, matched T2D/Cirrhosis metadata with 90% training split yields CV accuracies around 0.6683 (T2D) and 0.8846 (Cirrhosis), meeting or slightly exceeding the MetAML baselines 0.6640/0.8770
- Train error and test confusion matrix are both produced, confirming the class column was correctly parsed

## Limitations

- Metadata sample IDs must match the profile-table sample IDs exactly; formatting differences break the linkage
- Only two column numbers (class info and sample ID) are specified — metadata layouts with repeated or ambiguous columns require manual inspection
- Exact split criteria for the final benchmark accuracies were not fully stated ('we tested various split criteria and, depending on the data, obtained various accuracies')
- No quantitative metadata-consistency checks are built into MegaR; mismatches may only surface as odd model results

## Evidence

- [readme] The sample ID in the metadata **must match** exactly to sample ID in the initial metaphlan/qiime file.: "The sample ID in the metadata **must match** exactly to sample ID in the initial metaphlan/qiime file."
- [readme] Provide a column number for class info. For the T1D expeirment, column 7 (T1D_Diagnosed) has the class values (t, f): "For the T1D expeirment, column 7 (T1D_Diagnosed) has the class values (t, f) where t is True and f is False."
- [readme] For the T1D expeirment, column 2 (Gid_shotgun) has the sample IDs.: "For the T1D expeirment, column 2 (Gid_shotgun) has the sample IDs."
- [readme] MegaR automatically pulls the class labels from the provided column number for the class info.: "MegaR automatically pulls the class labels from the provided column number for the class info."
- [other] train a Random Forest model with gettrainingdonerf(mytable3, classid, sampleid, ruleout, psd, metadat, mrange): [evidence span withheld — anchor status: not_found]

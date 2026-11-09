---
name: metadata-column-mapping-for-classification
description: Use when a tab-separated metadata file accompanies a merged metaphlan/qiime
  abundance table for supervised model training in MegaR, and the columns holding
  sample IDs and class labels must be identified by column number.
when_to_use_negative:
- The input has no accompanying metadata file, so no class labels exist and only unsupervised
  analysis is possible
- The task is purely prediction on unknown samples using a previously saved RDS model
  that already encodes the label mapping
- The metadata sample IDs do not exactly match the sample IDs in the metaphlan/qiime
  file, which must be reconciled before mapping
tools:
- name: MegaR
  role: Provides the Model building interface where column numbers for class info
    and sample ID are supplied to map metadata to feature-table samples
  repo: https://github.com/BioHPC/MegaR
- name: R
  role: Runtime environment in which MegaR (an R Shiny package) runs to apply the
    metadata mapping
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-020-03933-4
    title: 'MegaR: an interactive R package for rapid sample classification and phenotype
      prediction using metagenome profiles and machine learning'
  source_content_hash: sha256:d5189124f756113cbaaaa9bca787e4df001510e88c91af090aad7c9aa3a96f9e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioHPC/MegaR
  tools:
  - MegaR
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# metadata-column-mapping-for-classification

## Summary

Maps MegaR metadata.tsv columns to sample IDs and class labels so supervised classifiers can be trained on merged MetaPhlAn/QIIME feature tables; correct column indices (e.g. column 7 T1D_Diagnosed, column 2 Gid_shotgun for the T1D DIABIMMUNE dataset) are required for label assignment.

## When to use

Use when a tab-separated metadata file accompanies a merged metaphlan/qiime abundance table for supervised model training in MegaR, and the columns holding sample IDs and class labels must be identified by column number.

## When NOT to use

- The input has no accompanying metadata file, so no class labels exist and only unsupervised analysis is possible
- The task is purely prediction on unknown samples using a previously saved RDS model that already encodes the label mapping
- The metadata sample IDs do not exactly match the sample IDs in the metaphlan/qiime file, which must be reconciled before mapping

## Inputs

- Tab-separated metadata file (metadata.tsv) with sample ID and class columns
- Merged MetaPhlAn or QIIME OTU/BIOM feature table whose sample IDs match the metadata (e.g. merged_metaphlan_t1d.txt)

## Outputs

- Class label assignment (f/t) per sample used for model training and confusion matrix generation
- Sample-ID-to-phenotype mapping feeding the trained model and unknown sample prediction

## How to apply

Upload the metadata.tsv alongside the feature table (merged_metaphlan_t1d.txt for the T1D experiment) in the MegaR Model building tab, then supply the column number that contains class information and the column number that contains sample IDs. Sample IDs in the metadata must match exactly the sample IDs in the initial metaphlan/qiime file, and MegaR automatically pulls the class labels (e.g. t and f, where t is True and f is False) from the specified class column. For the T1D dataset, set class info column 7 (T1D_Diagnosed) and sample ID column 2 (Gid_shotgun). Correct mapping is what allows subsequent steps — an 80% training split with Random Forest (gettrainingdonerf), mtry range 41-81, and getunknpredict — to attach the right phenotype to each sample.

## Related tools

- **MegaR** (Provides the Model building interface where column numbers for class info and sample ID are supplied to map metadata to feature-table samples) — https://github.com/BioHPC/MegaR
- **R** (Runtime environment in which MegaR (an R Shiny package) runs to apply the metadata mapping)

## Examples

```
library(MegaR); MegaR()  # upload metadata.tsv, set class info column = 7 (T1D_Diagnosed), sample ID column = 2 (Gid_shotgun)
```

## Evaluation signals

- MegaR successfully pulls the expected class labels (t and f for T1D) from the specified class-info column without errors
- Every sample ID in the metadata matches exactly a sample ID in the metaphlan/qiime feature table
- The training split (80%) produces both class labels in the training and test sets, and the confusion matrix populates for both classes
- Downstream accuracy plot over mtry range 41-81 and AUC-ROC are generated, confirming labels were attached to samples correctly

## Limitations

- Sample IDs in the metadata must match exactly the sample IDs in the initial metaphlan/qiime file; any mismatch breaks the mapping
- Using 100% of data for training leaves no test set, so no confusion matrix can be generated to verify label assignment
- With more than two classes, only selected classes should be kept via the 'Select level to classify' tab or control classes will degrade the model

## Evidence

- [readme] Sample IDs must match exactly: "The sample ID in the metadata **must match** exactly to sample ID in the initial metaphlan/qiime file."
- [readme] Class column for T1D: "For the T1D experiment, column 7 (T1D_Diagnosed) has the class values (t, f) where t is True and f is False."
- [readme] Automatic label extraction: "MegaR automatically pulls the class labels from the provided column number for the class info."

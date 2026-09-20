---
name: model-performance-metric-tabulation
description: Use when you have held-out test predictions from multiple classifiers
  trained on identical cross-validation splits of microbiome abundance data and need
  a model-by-metric comparison table mirroring Figs. 4-5.
when_to_use_negative:
- Models were trained on different data splits or different feature orderings, making
  metrics non-comparable
- Only a single model was evaluated and no cross-model comparison is intended
- The task is binary-only evaluation when the underlying datasets are multi-status
  (3-label oral, 5-label gut)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3474
tools:
- name: PM-CNN
  role: Phylogeny-based multi-path CNN whose test Kappa and F1 metrics are the comparison
    anchor in the benchmark table
  repo: https://github.com/qdu-bioinfo/PM_CNN
- name: scikit-learn
  role: Provides the conventional ML baselines (KNN, DT, SVM, XGBoost, RF) trained
    on the same splits whose metrics are tabulated
- name: torch
  role: Backend for PM-CNN and the DL baselines (CNN, MDeep) whose metrics enter the
    table
  repo: https://github.com/qdu-bioinfo/PM_CNN
- name: matplotlib
  role: Renders comparison figures from the aggregated metrics mirroring Figs. 4-5
- name: MDeep
  role: Single-branch convolution deep learning baseline; output layer widened to
    the number of classes for the multi-status comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae013
    title: 'PM-CNN: microbiome status recognition and disease detection model based
      on phylogeny and multi-path neural network'
  source_content_hash: sha256:d1c1325b9eb4b60fae20c4c06b1a4fc4b5d24c6be8509a0b1581d092f7e70635
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qdu-bioinfo/PM_CNN
  tools:
  - MDeep
  - PM-CNN
  - ape (R package)
  - matplotlib
  - numpy
  - scikit-learn
  - scipy
  - torch
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# model-performance-metric-tabulation

## Summary

Aggregate test metrics (e.g. Kappa, F1) from PM-CNN and its baselines (RF, SVM, DT, KNN, XGBoost, CNN, MDeep) into a single benchmark table so multi-status microbiome classification performance can be compared like-for-like across models.

## When to use

Use when you have held-out test predictions from multiple classifiers trained on identical cross-validation splits of microbiome abundance data and need a model-by-metric comparison table mirroring Figs. 4-5.

## When NOT to use

- Models were trained on different data splits or different feature orderings, making metrics non-comparable
- Only a single model was evaluated and no cross-model comparison is intended
- The task is binary-only evaluation when the underlying datasets are multi-status (3-label oral, 5-label gut)

## Inputs

- Test-set predictions or recorded test metrics (Kappa, F1) per model per fold
- Trained model results from PM-CNN (PyTorch), 5 scikit-learn ML baselines, and 2 deep learning baselines (CNN, MDeep)
- Cross-validation split definitions shared across all models
- Dataset labels: e.g. oral microbiome Control(653)/Periodontitis(274)/Gingivitis(660); gut microbiome Control(1418)/IBD(993)/HIV(360)/EDD(222)/CRC(120)

## Outputs

- benchmark_table.csv with one row per model (PM-CNN, 5 ML baselines, 2 DL baselines) and columns of reported metrics
- Optional matplotlib comparison figures mirroring the paper's Figs. 4-5

## How to apply

Train PM-CNN on phylogenetically ordered OTU features (ordered via cophenetic distance matrix, correlation transformation, and hierarchical clustering from Step 3), then train the five conventional ML baselines (KNN, DT, SVM, XGBoost, RF via scikit-learn) and two DL baselines (regular CNN, MDeep adapted to multi-status output width) using the same cross-validation splits. For each model on each dataset (human oral microbiome: 1587 samples, 1554 OTUs, 3 labels; human gut microbiome: 3113 samples, 5597 OTUs, 5 labels), compute test metrics such as Kappa and F1. Aggregate one row per model with all reported metrics into benchmark_table.csv using pandas, and optionally render comparison figures with matplotlib mirroring the reported Figs. 4-5. The tabulation is only valid if all models saw identical splits and identical feature inputs, so record split indices alongside metrics.

## Related tools

- **PM-CNN** (Phylogeny-based multi-path CNN whose test Kappa and F1 metrics are the comparison anchor in the benchmark table) — https://github.com/qdu-bioinfo/PM_CNN
- **scikit-learn** (Provides the conventional ML baselines (KNN, DT, SVM, XGBoost, RF) trained on the same splits whose metrics are tabulated)
- **torch** (Backend for PM-CNN and the DL baselines (CNN, MDeep) whose metrics enter the table) — https://github.com/qdu-bioinfo/PM_CNN
- **matplotlib** (Renders comparison figures from the aggregated metrics mirroring Figs. 4-5)
- **MDeep** (Single-branch convolution deep learning baseline; output layer widened to the number of classes for the multi-status comparison)

## Examples

```
python PMCNN.py --train --train_otu_table ../example/train_data/example_abundance_table.csv --meta ../example/train_data/meta.csv
```

## Evaluation signals

- benchmark_table.csv exists and contains exactly one row per model: PM-CNN, KNN, DT, SVM, XGBoost, RF, CNN, MDeep
- Every row carries all reported metrics (e.g. Kappa and F1) with no missing cells
- Metric values on the oral microbiome dataset reproduce the paper's direction: PM-CNN's Kappa and F1 exceed both MDeep and regular CNN
- All models were scored on identical cross-validation splits; verify recorded split indices match across rows
- Row labels use the paper's model names and class labels (0/1/2 oral; 0-4 gut) consistent with the meta information format

## Limitations

- Comparability depends on all models sharing identical cross-validation splits and inputs; divergent splits invalidate the table
- Adapting MDeep to multi-status requires widening its output layer to the number of classes, which is a modification of the published baseline
- README's Data availability section swaps the dataset descriptions (Dataset 1 labeled gut but described as oral), so verify label counts before tabulating

## Evidence

- [abstract] PM-CNN outperformed both deep learning baselines: "We applied PM-CNN to human microbiome data for status and disease detection, demonstrating its significant superiority over existing ML models."
- [methods] training and testing steps: "Step 4: Model training and testing"
- [readme] usage of PMCNN.py: "usage: code/PMCNN.py [--train] //train PM-CNN model [--test] //test PM-CNN model"
- [supplementary] dataset labels: "The human oral microbiome data contains 1587 samples with 1554 OTUs. Contains three label types: Control(653), Periodontitis(274), and Gingivitis(660)"

---
name: deep-learning-baseline-output-layer-adaptation
description: Use when benchmarking a multi-status microbiome classifier (e.g. 3 oral
  labels Control/Periodontitis/Gingivitis or 5 gut labels) and needing convolution-based
  deep learning baselines that were originally designed for a different output task.
when_to_use_negative:
- The baseline model is already configured for the target number of classes — no output-layer
  adaptation is needed
- The task is binary or the label schema matches the baseline's original design, so
  retraining with identical splits without layer modification is sufficient
- Only conventional ML baselines (KNN, DT, SVM, XGBoost, RF) are being benchmarked
  and no convolution deep baseline is required
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: torch
  role: Implements and retrains the adapted single-branch convolution baselines (MDeep,
    regular CNN) with the widened output layer
- name: scikit-learn
  role: Trains the conventional ML baselines (KNN, DT, SVM, XGBoost, RF) on the same
    cross-validation splits as the adapted deep baselines
- name: numpy
  role: Loads and reshapes the OTU abundance table and labels for baseline training
- name: matplotlib
  role: Renders comparison figures mirroring the paper's reported Figs. 4-5
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae013
    title: 'PM-CNN: microbiome status recognition and disease detection model based
      on phylogeny and multi-path neural network'
  source_content_hash: sha256:4e16747b53ffd3da2d0c5e0c3e071638d76ea53b5a254f663b20b27b8c026f61
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PM-CNN
  - ape (R package)
  - matplotlib
  - numpy
  - scikit-learn
  - scipy
  - torch
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# deep-learning-baseline-output-layer-adaptation

## Summary

Adapts single-branch convolution deep learning baselines (regular CNN, MDeep) to multi-status microbiome classification by widening their output layers to the number of status classes, so they can be benchmarked fairly against PM-CNN using identical cross-validation splits.

## When to use

Use when benchmarking a multi-status microbiome classifier (e.g. 3 oral labels Control/Periodontitis/Gingivitis or 5 gut labels) and needing convolution-based deep learning baselines that were originally designed for a different output task.

## When NOT to use

- The baseline model is already configured for the target number of classes — no output-layer adaptation is needed
- The task is binary or the label schema matches the baseline's original design, so retraining with identical splits without layer modification is sufficient
- Only conventional ML baselines (KNN, DT, SVM, XGBoost, RF) are being benchmarked and no convolution deep baseline is required

## Inputs

- OTU abundance table (rows = samples, columns = OTUs, values = relative abundance) e.g. example_abundance_table.csv
- Sample status metadata table (SampleID, Status) with integer-coded labels (e.g. 0, 1, 2 for Control/Periodontitis/Gingivitis)
- Same cross-validation splits used for PM-CNN and ML baseline training
- Baseline model implementations (MDeep, regular CNN)

## Outputs

- Per-model, per-metric test results (Kappa, F1) for adapted MDeep and regular CNN on each cross-validation split
- benchmark_table.csv with one row per model (PM-CNN, 5 ML baselines, 2 DL baselines) and all reported metrics

## How to apply

Take the existing single-branch convolution models (regular CNN and MDeep) and modify each network's output layer so its width equals the number of status classes in the dataset (e.g. 3 classes for the human oral microbiome dataset, 5 for the gut dataset). Train each adapted model with the same cross-validation splits used for PM-CNN and the five conventional ML baselines (KNN, DT, SVM, XGBoost, RF) from scikit-learn, recording test Kappa and F1 for every split. Keeping splits identical is the rationale: the comparison must isolate the effect of the phylogenetically organized multi-path architecture, not differences in data partitioning. Aggregate model-by-metric results into benchmark_table.csv and check the direction against the paper's finding that PM-CNN achieves superior Kappa and F1 over MDeep and regular CNN on the oral dataset.

## Related tools

- **torch** (Implements and retrains the adapted single-branch convolution baselines (MDeep, regular CNN) with the widened output layer)
- **scikit-learn** (Trains the conventional ML baselines (KNN, DT, SVM, XGBoost, RF) on the same cross-validation splits as the adapted deep baselines)
- **numpy** (Loads and reshapes the OTU abundance table and labels for baseline training)
- **matplotlib** (Renders comparison figures mirroring the paper's reported Figs. 4-5)

## Examples

```
python PMCNN.py --train --train_otu_table ../example/train_data/example_abundance_table.csv --meta ../example/train_data/meta.csv
```

## Evaluation signals

- benchmark_table.csv exists and contains one row per model covering PM-CNN, the 5 ML baselines, and the 2 DL baselines with all reported metrics
- Adapted MDeep and regular CNN output layers have exactly as many units as classes in the dataset (3 for the oral dataset, 5 for the gut dataset)
- All models were evaluated on identical cross-validation splits — split indices match those used for PM-CNN
- Comparison direction reproduces the finding that PM-CNN achieves superior Kappa and F1 over MDeep and regular CNN on the human oral microbiome dataset
- Baseline metrics fall in a plausible range relative to the paper's reported values (no degenerate constant-prediction results)

## Limitations

- Adaptation of MDeep to multi-status requires widening the output layer because the original model was not designed for this setting; naive reuse would misrepresent the baseline
- The benchmark result is dataset-specific: the cited superiority of PM-CNN was observed on the human oral microbiome dataset; behavior on other datasets must be verified
- Dataset labels are integer-coded (e.g. oral: 0, 1, 2; gut: 0-4) with strong class imbalance (e.g. gut CRC n=120 vs Control n=1418), which can distort Kappa/F1 if splits are not stratified

## Evidence

- [results] Adapt MDeep to the multi-status setting by widening its output layer to the number of classes, train it with the same splits, and record test metrics.: [evidence span withheld — anchor status: not_found]
- [results] On the human oral microbiome dataset, PM-CNN outperformed both deep learning baselines, achieving superior Kappa and F1 scores compared to the single-branch convolution-based models MDeep and regular CNN.: [evidence span withheld — anchor status: not_found]
- [readme] The human oral microbiome data contains 1587 samples with 1554 OTUs. Contains three label types: Control(653), Periodontitis(274), and Gingivitis(660), which are marked with 0, 1, and 2 respectively.: "Contains three label types: Control(653), Periodontitis(274), and Gingivitis(660), which are marked with 0, 1, and 2 respectively"

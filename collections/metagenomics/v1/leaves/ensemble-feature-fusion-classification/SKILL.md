---
name: ensemble-feature-fusion-classification
description: Use when classifying microbiome samples into multiple disease/status
  labels from OTU or species abundance tables, after features have been phylogenetically
  organized into multi-layer clustering groups by PM-CNN's Step 3.
when_to_use_negative:
- Input data is not a microbial feature table (OTU table or species table) — PM-CNN's
  input format requires this structure exclusively
- Only a single-class or simple binary label exists and a lightweight conventional
  ML baseline (e.g. scikit-learn RF/SVM) is sufficient
- No phylogenetic tree or representative sequences are available to derive the clustered
  feature ordering the ensemble paths depend on
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3474
tools:
- name: PM-CNN
  role: Multi-path CNN whose per-path features are fused by ensemble learning for
    classification; trained/tested via code/PMCNN.py
  repo: https://github.com/qdu-bioinfo/PM_CNN
- name: torch
  role: Deep learning framework implementing PM-CNN's convolutional paths and ensemble
    fusion (torch >= 1.11.0)
- name: scikit-learn
  role: Trains conventional ML baselines (KNN, DT, SVM, XGBoost, RF) on the same cross-validation
    splits for comparison (>= 1.1.1)
- name: ape (R package)
  role: cophenetic function supplies the distance matrix used to build the clustered
    feature groups feeding each ensemble path
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae013
    title: 'PM-CNN: microbiome status recognition and disease detection model based
      on phylogeny and multi-path neural network'
  source_content_hash: sha256:136b6f2b7a96aec454bb9eba225dd401f034a06530416f2f6972be1bbcabdb54
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qdu-bioinfo/PM_CNN
  tools:
  - PM-CNN
  - ape (R package)
  - matplotlib
  - numpy
  - scikit-learn
  - scipy
  - torch
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# ensemble-feature-fusion-classification

## Summary

Fuses features extracted by multiple parallel convolutional paths of PM-CNN using an ensemble learning method to make multi-status microbiome classification decisions (e.g. Control/Periodontitis/Gingivitis on human oral microbiome data).

## When to use

Use when classifying microbiome samples into multiple disease/status labels from OTU or species abundance tables, after features have been phylogenetically organized into multi-layer clustering groups by PM-CNN's Step 3.

## When NOT to use

- Input data is not a microbial feature table (OTU table or species table) — PM-CNN's input format requires this structure exclusively
- Only a single-class or simple binary label exists and a lightweight conventional ML baseline (e.g. scikit-learn RF/SVM) is sufficient
- No phylogenetic tree or representative sequences are available to derive the clustered feature ordering the ensemble paths depend on

## Inputs

- Microbial abundance table (OTU or species table, .csv, samples x OTUs with relative abundances)
- Meta information file mapping SampleID to Status labels
- Multi-layer hierarchical clustering results from clustering.py (clustered_groups .csv)
- Cophenetic distance matrix (.csv) from the phylogenetic tree

## Outputs

- Trained PM-CNN model with ensemble-fused multi-path features
- Per-sample status/disease predictions and test metrics (Kappa, F1)
- Model-by-metric benchmark comparison against ML and DL baselines (e.g. benchmark_table.csv)

## How to apply

First reorder OTU features according to the multi-layer hierarchical clustering results derived from the cophenetic distance matrix (R ape cophenetic function, followed by distance transformation to a correlation matrix and hierarchical clustering). Then train PM-CNN, which extracts features via a multi-path convolutional neural network where each path processes one phylogenetically clustered feature group (parameters: --batch_size, --learning_rate, --channel, --kernel_size, --strides). An ensemble learning method then fuses the per-path features to make the final classification decision across status labels (e.g. labels 0/1/2 for Control/Periodontitis/Gingivitis in dataset 2). Evaluate with Kappa and F1 scores under cross-validation, benchmarking against ML baselines (KNN, DT, SVM, XGBoost, RF) and DL baselines (regular CNN, MDeep) on the same splits.

## Related tools

- **PM-CNN** (Multi-path CNN whose per-path features are fused by ensemble learning for classification; trained/tested via code/PMCNN.py) — https://github.com/qdu-bioinfo/PM_CNN
- **torch** (Deep learning framework implementing PM-CNN's convolutional paths and ensemble fusion (torch >= 1.11.0))
- **scikit-learn** (Trains conventional ML baselines (KNN, DT, SVM, XGBoost, RF) on the same cross-validation splits for comparison (>= 1.1.1))
- **ape (R package)** (cophenetic function supplies the distance matrix used to build the clustered feature groups feeding each ensemble path)

## Examples

```
python PMCNN.py --train --train_otu_table ../example/train_data/example_abundance_table.csv --meta ../example/train_data/meta.csv
```

## Evaluation signals

- PM-CNN achieves superior Kappa and F1 scores versus single-branch convolution baselines (regular CNN, MDeep) on the human oral microbiome dataset
- All benchmark rows (PM-CNN + 5 ML baselines + 2 DL baselines) present in benchmark_table.csv with metrics computed on identical cross-validation splits
- Test output is a class label within the dataset's label set (0/1/2 for Control/Periodontitis/Gingivitis; 0-4 for the five-label gut dataset)
- Training console output completes without error and test predictions render directly in the console as documented

## Limitations

- Phylogenetic tree construction requires external software (Mafft, FastTree) and representative sequences from the GreenGenes database, or a user-provided NEWICK file
- Performance comparison is demonstrated on the two provided human microbiome datasets (gut: 3113 samples/5597 OTUs, 5 labels; oral: 1587 samples/1554 OTUs, 3 labels); generalization to other data types is not established here
- Label imbalance exists in the datasets (e.g. only 120 CRC samples vs 1418 Control), which may affect minority-class metrics

## Evidence

- [abstract] PM-CNN organizes microbes based on their phylogenetic relationships and extracts features using a multi-path convolutional neural network. An ensemble learning method then fuses these features to make accurate classification decisions.: "PM-CNN organizes microbes based on their phylogenetic relationships and extracts features using a multi-path convolutional neural network. An ensemble learning method then fuses these features"
- [results] achieving superior Kappa and F1 scores compared to the single-branch convolution-based models MDeep and regular CNN: [evidence span withheld — anchor status: not_found]
- [readme] usage: code/PMCNN.py [--train] //train PM-CNN model [--test] //test PM-CNN model: "usage: code/PMCNN.py [--train] //train PM-CNN model [--test] //test PM-CNN model"
- [supplementary] The human oral microbiome data contains 1587 samples with 1554 OTUs. Contains three label types: Control(653), Periodontitis(274), and Gingivitis(60): "The human oral microbiome data contains 1587 samples with 1554 OTUs. Contains three label types: Control(653), Periodontitis(274), and Gingivitis(660)"
- [intro] transform the distance matrix into the correlation matrix by the designed distance transformation formula, and then carry out hierarchical clustering: "transform the distance matrix into the correlation matrix by the designed distance transformation formula, and then carry out hierarchical clustering"

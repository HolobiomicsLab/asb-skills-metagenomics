---
name: multiclass-classification-metric-computation
description: Use when a trained multiclass classifier (e.g., PM-CNN predicting Control/Periodontitis/Gingivitis
  or gut disease labels) must be scored on held-out samples with the five paper-standard
  metrics and reported to a metrics CSV.
when_to_use_negative:
- The task is regression or ranking rather than labeled multiclass classification
  with discrete status labels.
- The input abundance table has not yet been reordered by the phylogeny-based multi-layer
  hierarchical clustering order, so the model itself cannot be validly trained or
  tested.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2269
tools:
- name: PM-CNN
  role: Phylogenetic multi-path CNN that produces the per-category softmax classification
    results being evaluated
  repo: https://github.com/qdu-bioinfo/PM_CNN
- name: scikit-learn
  role: Provides the Kappa, Recall, Accuracy, Precision, and F1 metric functions applied
    to predicted vs. true labels
- name: numpy
  role: Array handling for prediction and ground-truth label vectors during metric
    computation
- name: torch
  role: Framework in which the trained PM-CNN model is run over test samples before
    scoring
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbae013
    title: 'PM-CNN: microbiome status recognition and disease detection model based
      on phylogeny and multi-path neural network'
  source_content_hash: sha256:1b51812c8428f38caa8f1ebc9933c5be44ae8b0a9f0ff801c8187b229c335943
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qdu-bioinfo/PM_CNN
  tools:
  - PM-CNN
  - numpy
  - scikit-learn
  - torch
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# multiclass-classification-metric-computation

## Summary

Computes Kappa, Recall, Accuracy, Precision, and F1 scores to quantify PM-CNN classification performance on held-out human microbiome samples, following the paper's Results 3.2 evaluation protocol.

## When to use

Use when a trained multiclass classifier (e.g., PM-CNN predicting Control/Periodontitis/Gingivitis or gut disease labels) must be scored on held-out samples with the five paper-standard metrics and reported to a metrics CSV.

## When NOT to use

- The task is regression or ranking rather than labeled multiclass classification with discrete status labels.
- The input abundance table has not yet been reordered by the phylogeny-based multi-layer hierarchical clustering order, so the model itself cannot be validly trained or tested.

## Inputs

- Held-out test OTU abundance table (.csv) in the PM-CNN microbial feature table format
- Trained PM-CNN model and its per-category softmax predictions on the test samples
- Ground-truth sample status labels (e.g., 0=Control, 1=IBD, 2=HIV, 3=EDD, 4=CRC)

## Outputs

- pm_cnn_metrics.csv containing five values: Kappa, Recall, Accuracy, Precision, F1

## How to apply

After Step 4 of the PM-CNN pipeline (model training and testing), run the trained model on the held-out OTU table to obtain per-category softmax predictions, then compute Kappa, Recall, Accuracy, Precision, and F1 with scikit-learn (>= 1.1.1) metric functions on the predicted versus true labels. Use numpy (>= 1.22.3) for array handling of the label vectors and, for multiclass problems such as the three-class oral dataset (labels 0/1/2) or five-class gut dataset (labels 0-4), average the per-class precision/recall/F1 appropriately. Write the five metric values to pm_cnn_metrics.csv so the scores can be compared against existing ML baselines. Validation follows Results 3.2: PM-CNN's five scores should demonstrate superiority over existing ML models on the human microbiome classification task.

## Related tools

- **PM-CNN** (Phylogenetic multi-path CNN that produces the per-category softmax classification results being evaluated) — https://github.com/qdu-bioinfo/PM_CNN
- **scikit-learn** (Provides the Kappa, Recall, Accuracy, Precision, and F1 metric functions applied to predicted vs. true labels)
- **numpy** (Array handling for prediction and ground-truth label vectors during metric computation)
- **torch** (Framework in which the trained PM-CNN model is run over test samples before scoring)

## Examples

```
python PMCNN.py --test --test_otu_table ../example/test_data/example_test.csv
```

## Evaluation signals

- pm_cnn_metrics.csv exists and contains all five metrics: Kappa, Recall, Accuracy, Precision, and F1.
- Each metric value lies in the valid range [0, 1] (Kappa in [-1, 1]) for the held-out test split.
- Scores are computed against true sample status labels (e.g., the three oral classes Control(653)/Periodontitis(274)/Gingivitis(660) marked 0, 1, 2).
- Following Results 3.2, PM-CNN's five scores show superiority over existing ML baselines on the human microbiome classification task.
- Re-running evaluation on the same trained model and test table reproduces identical metric values.

## Limitations

- Metric superiority claim is demonstrated on the two human microbiome datasets (Dataset 1 and Dataset 2); results may not transfer to other domains.
- The five-metric protocol covers overall classification quality only; per-class performance on rare classes (e.g., CRC with 120 samples) may be unbalanced and warrants caution.
- The comparison to baselines depends on the clustering-derived feature order; metrics are meaningless if the model was trained on unordered OTU features.

## Evidence

- [other] its performance is evaluated with Kappa, Recall, Accuracy, Precision, and F1 scores: [evidence span withheld — anchor status: not_found]
- [abstract] demonstrating its significant superiority over existing ML models: "We applied PM-CNN to human microbiome data for status and disease detection, demonstrating its significant superiority over existing ML models."
- [intro] scikit-learn >= 1.1.1: "* scikit-learn >= 1.1.1"
- [supplementary] Contains three label types: Control(653), Periodontitis(274), and Gingivitis(660): "The human oral microbiome data contains 1587 samples with 1554 OTUs. Contains three label types: Control(653), Periodontitis(274), and Gingivitis(660)"
- [readme] Step 4: Model training and testing: "Step 4: Model training and testing"

---
name: pytorch-model-training-and-testing
description: Use when a labeled microbial feature table (OTU/species abundance table
  plus sample metadata) must be classified with a phylogeny-aware deep learning model
  whose features are ordered by hierarchical clustering of phylogenetic distances.
when_to_use_negative:
- When the goal is unsupervised ordination or differential abundance testing rather
  than supervised status/disease classification from labeled samples
- When no phylogenetic tree or feature clustering order is available and phylogeny-informed
  feature adjacency cannot be constructed (the convolutional paths assume phylogenetically
  ordered features)
- When the input is raw sequence data rather than a microbial feature table (README
  requires an OTU/species abundance table as input)
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3474
tools:
- name: PM-CNN
  role: Phylogenetic multi-path CNN model with ensemble feature fusion trained and
    tested for microbiome classification
  repo: https://github.com/qdu-bioinfo/PM_CNN
- name: torch
  role: Deep learning framework (>= 1.11.0) used to implement and train/test the four-path
    CNN
- name: scikit-learn
  role: Metric functions (>= 1.1.1) to compute Kappa, Recall, Accuracy, Precision,
    and F1 on test predictions
- name: numpy
  role: Array handling (>= 1.22.3) for abundance matrices and predictions
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbae013
    title: 'PM-CNN: microbiome status recognition and disease detection model based
      on phylogeny and multi-path neural network'
  source_content_hash: sha256:c48b20adf2db8a0b1d8e6f97310b4d35efd72c335a1c002b4ede2ce28f0cc7bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qdu-bioinfo/PM_CNN
  tools:
  - PM-CNN
  - numpy
  - scikit-learn
  - torch
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# pytorch-model-training-and-testing

## Summary

Train and test PM-CNN, a phylogeny-ordered multi-path convolutional neural network with ensemble (fully-connected) feature fusion, on microbiome abundance tables to classify disease/status labels, evaluated with Kappa, Recall, Accuracy, Precision, and F1.

## When to use

Use when a labeled microbial feature table (OTU/species abundance table plus sample metadata) must be classified with a phylogeny-aware deep learning model whose features are ordered by hierarchical clustering of phylogenetic distances.

## When NOT to use

- When the goal is unsupervised ordination or differential abundance testing rather than supervised status/disease classification from labeled samples
- When no phylogenetic tree or feature clustering order is available and phylogeny-informed feature adjacency cannot be constructed (the convolutional paths assume phylogenetically ordered features)
- When the input is raw sequence data rather than a microbial feature table (README requires an OTU/species abundance table as input)

## Inputs

- Microbial abundance table (OTU or species table, CSV; samples x features, e.g. example_abundance_table.csv)
- Sample metadata CSV mapping SampleID to Status labels (e.g. meta.csv, labels marked 0/1/2 for Control/Periodontitis/Gingivitis or 0-4 for gut dataset)
- Clustered feature order file from Step 3 hierarchical clustering (output_file.csv)
- Test-set OTU table (e.g. example_test.csv)

## Outputs

- Trained PM-CNN model (torch weights)
- Per-category softmax classification predictions on the test set
- Evaluation metrics: Kappa, Recall, Accuracy, Precision, and F1 scores (e.g. written to pm_cnn_metrics.csv)

## How to apply

First ensure the OTU feature columns are ordered according to the multi-layer hierarchical clustering derived from the cophenetic distance matrix of the phylogenetic tree (Steps 1-3 of the PM-CNN pipeline), so that phylogenetically related microbes are adjacent for the convolutional paths. Train the four-path CNN in torch (>= 1.11.0) using the --train mode with the abundance table and metadata CSV; tune batch_size, learning_rate, channel, kernel_size, and strides via the CLI parameters. The per-path CNN features are flattened and fused by a fully connected ensemble layer, and per-category predictions are produced via softmax. Then run --test on the held-out OTU table and compute Kappa, Recall, Accuracy, Precision, and F1 with scikit-learn (>= 1.1.1) metrics, using numpy (>= 1.22.3) for array handling. Correctness is judged by the five metrics being reported and, per Results 3.2, PM-CNN scores showing superiority over existing ML baselines on human microbiome classification tasks.

## Related tools

- **PM-CNN** (Phylogenetic multi-path CNN model with ensemble feature fusion trained and tested for microbiome classification) — https://github.com/qdu-bioinfo/PM_CNN
- **torch** (Deep learning framework (>= 1.11.0) used to implement and train/test the four-path CNN)
- **scikit-learn** (Metric functions (>= 1.1.1) to compute Kappa, Recall, Accuracy, Precision, and F1 on test predictions)
- **numpy** (Array handling (>= 1.22.3) for abundance matrices and predictions)

## Examples

```
cd PM_CNN/code && python PMCNN.py --train --train_otu_table ../example/train_data/example_abundance_table.csv --meta ../example/train_data/meta.csv && python PMCNN.py --test --test_otu_table ../example/test_data/example_test.csv
```

## Evaluation signals

- All five metrics (Kappa, Recall, Accuracy, Precision, F1) are computed and written to the output file (pm_cnn_metrics.csv)
- Test set is strictly held out from training samples (separate --test_otu_table supplied to PMCNN.py)
- Feature columns of the input table match the clustered feature order produced by Step 3 hierarchical clustering
- PM-CNN's scores show superiority over existing ML baselines on the human microbiome classification task, following Results 3.2
- Training/test commands run without error using the README's example abundance table and metadata

## Limitations

- Phylogenetic tree construction requires representative sequences looked up from the GreenGenes database and external Mafft and FastTree installations (or a user-provided NEWICK tree)
- Model hyperparameters (batch_size, learning_rate, channel, kernel_size, strides) must be set manually via CLI flags; the README does not give default values
- Input must strictly follow the microbial feature table format; other formats are not supported by PM-CNN
- Reported datasets are imbalanced (e.g. gut dataset: Control(1418), IBD(993), HIV(360), EDD(222), CRC(120)), which may affect per-category metrics

## Evidence

- [readme] we proposed a new deep learning framework PM-CNN (Phylogenetic Multi-path Convolutional Neural Network), which combines convolutional neural networks and microbial phylogenetic structures to predict various human diseases.: "we proposed a new deep learning framework PM-CNN (Phylogenetic Multi-path Convolutional Neural Network), which combines convolutional neural networks and microbial phylogenetic structures to predict"
- [readme] Usage of PMCNN.py with --train/--test flags and parameters: "usage: code/PMCNN.py [--train] //train PM-CNN model [--test] //test PM-CNN model [--train_otu_table] //microbial abundance table [--meta] //sample disease information"
- [readme] Now, after the processing of Steps 1-3, we now have the clustered feature order. Next, we will start training our model.: "Now, after the processing of Steps 1-3, we now have the clustered feature order. Next, we will start training our model."
- [abstract] PM-CNN organizes microbes based on their phylogenetic relationships and extracts features using a multi-path convolutional neural network. An ensemble learning method then fuses these features to make accurate classification decisions.: "PM-CNN organizes microbes based on their phylogenetic relationships and extracts features using a multi-path convolutional neural network. An ensemble learning method then fuses these features"
- [readme] The input data should exclusively adhere to the microbial feature table structure, (e.g. OTU table or species table).: "The input data should exclusively adhere to the microbial feature table structure, (e.g. OTU table or species table)."

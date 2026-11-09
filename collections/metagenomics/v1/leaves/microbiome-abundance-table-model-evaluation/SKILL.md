---
name: microbiome-abundance-table-model-evaluation
description: Use when you have a microbial abundance table (OTU or species table)
  plus sample disease/status labels and need to train and benchmark a phylogeny-aware
  deep-learning classifier for microbiome status or disease detection.
when_to_use_negative:
- Input is not a microbial feature table (OTU or species table) in the README's expected
  format — the README states the input 'should exclusively adhere to the microbial
  feature table structure'
- You need a purely supervised tabular classifier without phylogenetic ordering —
  PM-CNN requires a NEWICK phylogenetic tree or clustered feature order derived from
  cophenetic distances
- Only raw sequences are available with no abundance table or status metadata — build
  the abundance table and labels first before invoking this skill
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3474
tools:
- name: PM-CNN
  role: Phylogeny-based multi-path CNN model trained on the abundance table and evaluated
    for status/disease classification
  repo: https://github.com/qdu-bioinfo/PM_CNN
- name: torch
  role: Deep-learning backend (>= 1.11.0) used to train the four-path CNN with ensemble
    fusion
- name: scikit-learn
  role: Metric functions (>= 1.1.1) to compute Kappa, Recall, Accuracy, Precision,
    and F1 on the test split
- name: numpy
  role: Array handling (>= 1.22.3) for abundance matrices and predictions during training/testing
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbae013
    title: 'PM-CNN: microbiome status recognition and disease detection model based
      on phylogeny and multi-path neural network'
  source_content_hash: sha256:43deaa6863d43253aa7b49b151e02920b8dd06b66d5c1a9bab347d361ea28eb4
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
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# microbiome-abundance-table-model-evaluation

## Summary

Train PM-CNN, a phylogeny-ordered four-path convolutional neural network with ensemble fusion, on a microbial abundance table (OTU or species table) with sample status metadata, and evaluate per-category classification performance with Kappa, Recall, Accuracy, Precision, and F1.

## When to use

Use when you have a microbial abundance table (OTU or species table) plus sample disease/status labels and need to train and benchmark a phylogeny-aware deep-learning classifier for microbiome status or disease detection.

## When NOT to use

- Input is not a microbial feature table (OTU or species table) in the README's expected format — the README states the input 'should exclusively adhere to the microbial feature table structure'
- You need a purely supervised tabular classifier without phylogenetic ordering — PM-CNN requires a NEWICK phylogenetic tree or clustered feature order derived from cophenetic distances
- Only raw sequences are available with no abundance table or status metadata — build the abundance table and labels first before invoking this skill

## Inputs

- Microbial abundance table (OTU table or species table; CSV, samples as rows, OTUs as columns, relative abundances)
- Sample metadata file (SampleID, Status label; e.g. Control, Gingivitis, Periodontitis marked 0, 1, 2)
- Hierarchical clustering results (clustered feature order) from phylogeny-based multi-layer clustering (step-3 output)
- Test-set abundance table for held-out evaluation

## Outputs

- Trained PM-CNN model (four-path CNN with fully connected ensemble fusion)
- Per-category classification predictions via softmax on the test split
- Classification metric values (Kappa, Recall, Accuracy, Precision, F1), written to pm_cnn_metrics.csv

## How to apply

First reorder OTU features so that phylogenetically related microbes are adjacent: derive the feature order by transforming the cophenetic distance matrix into a correlation matrix and running hierarchical clustering (clustering.py), which yields multi-layer clustered groups passed via --clustered_groups. Then run PMCNN.py in --train mode with --train_otu_table (microbial abundance table in the README's OTU-table format, rows=samples, columns=OTUs), --meta (SampleID/Status labels such as Control=0, Periodontitis=1, Gingivitis=2), and optionally tune --batch_size, --learning_rate, --channel, --kernel_size, and --strides. Train with torch (>= 1.11.0) using the four-path CNN whose flattened features are fused by a fully connected ensemble layer with softmax per-category output. Evaluate on held-out test data (--test --test_otu_table) using scikit-learn (>= 1.1.1) metric functions with numpy (>= 1.22.3) for array handling, computing Kappa, Recall, Accuracy, Precision, and F1. Per Results 3.2, verify superiority over existing ML baselines (e.g. on human microbiome classification) before accepting the result.

## Related tools

- **PM-CNN** (Phylogeny-based multi-path CNN model trained on the abundance table and evaluated for status/disease classification) — https://github.com/qdu-bioinfo/PM_CNN
- **torch** (Deep-learning backend (>= 1.11.0) used to train the four-path CNN with ensemble fusion)
- **scikit-learn** (Metric functions (>= 1.1.1) to compute Kappa, Recall, Accuracy, Precision, and F1 on the test split)
- **numpy** (Array handling (>= 1.22.3) for abundance matrices and predictions during training/testing)

## Examples

```
cd PM_CNN/code && python PMCNN.py --train --train_otu_table ../example/train_data/example_abundance_table.csv --meta ../example/train_data/meta.csv
```

## Evaluation signals

- All five metrics (Kappa, Recall, Accuracy, Precision, F1) are present in pm_cnn_metrics.csv with valid numeric values in [0, 1] (Kappa may be lower/negative in degenerate cases)
- Per Results 3.2, PM-CNN's scores show superiority over existing ML baselines on the human microbiome classification task
- Training run completes on the example data (e.g. human oral microbiome, 1587 samples / 1554 OTUs, three labels) and test predictions are produced for every held-out sample
- The --clustered_groups input corresponds to the multi-layer hierarchical clustering output of clustering.py applied to the cophenetic distance matrix
- Label mapping in metadata matches the README convention (e.g. Control(653), Periodontitis(274), Gingivitis(660) marked 0, 1, 2)

## Limitations

- The phylogenetic tree construction step is optional and must come from Mafft/FastTree or a user-provided NEWICK file; without a tree the feature ordering cannot be derived
- Representative sequences must be found from the GreenGenes database before Mafft alignment when building the tree
- Performance claims of superiority are grounded in human microbiome datasets (Dataset 1 gut: 3113 samples/5597 OTUs, five labels; Dataset 2 oral: 1587 samples/1554 OTUs, three labels); behavior on other data types is not established here

## Evidence

- [abstract] PM-CNN organizes microbes based on their phylogenetic relationships and extracts features using a multi-path convolutional neural network. An ensemble learning method then fuses these features to make accurate classification decisions.: "PM-CNN organizes microbes based on their phylogenetic relationships and extracts features using a multi-path convolutional neural network."
- [readme] The input data should exclusively adhere to the microbial feature table structure, (e.g. OTU table or species table).: "The input data should exclusively adhere to the microbial feature table structure, (e.g. OTU table or species table)."
- [readme] usage: code/PMCNN.py [--train] //train PM-CNN model [--test] //test PM-CNN model [--train_otu_table] //microbial abundance table [--meta] //sample disease information: "usage: code/PMCNN.py [--train] //train PM-CNN model [--test] //test PM-CNN model"
- [results] We applied PM-CNN to human microbiome data for status and disease detection, demonstrating its significant superiority over existing ML models.: "We applied PM-CNN to human microbiome data for status and disease detection, demonstrating its significant superiority over existing ML models."

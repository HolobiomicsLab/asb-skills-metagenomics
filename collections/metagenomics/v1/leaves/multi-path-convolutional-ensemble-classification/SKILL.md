---
name: multi-path-convolutional-ensemble-classification
description: Use when classifying microbiome samples (OTU/species abundance table
  + sample metadata) into disease or control status categories and you want a phylogeny-aware
  deep learning classifier benchmarked against ML baselines.
when_to_use_negative:
- When no phylogenetic tree or OTU representative sequences are available and features
  cannot be ordered by phylogeny, undermining the multi-path convolution design
- When the goal is only a simple taxonomic summary or differential abundance test
  rather than supervised status/disease classification
- When the input is not a microbial feature table (OTU/species abundance), which is
  a stated prerequisite for PM-CNN
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3474
tools:
- name: PM-CNN
  role: Phylogenetic multi-path CNN framework being trained and tested for microbiome
    status classification
  repo: https://github.com/qdu-bioinfo/PM_CNN
- name: torch
  role: Deep learning backend (>= 1.11.0) for training and running the four-path CNN
    and ensemble fusion
- name: scikit-learn
  role: Metrics library (>= 1.1.1) for computing Kappa, Recall, Accuracy, Precision,
    and F1 on test predictions
- name: numpy
  role: Array handling (>= 1.22.3) for abundance matrices and predictions
- name: ape (R package)
  role: Provides the cophenetic function to derive the cophenetic distance matrix
    from the phylogenetic tree
- name: FastTree
  role: Builds the phylogenetic tree from Mafft-aligned OTU representative sequences
- name: Mafft
  role: Multiple sequence alignment of OTU representative sequences prior to tree
    building
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbae013
    title: 'PM-CNN: microbiome status recognition and disease detection model based
      on phylogeny and multi-path neural network'
  source_content_hash: sha256:10f849db6a731c500bccef9ec0b90605fe7c8c52d8604f676ec014eaaa085e5a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qdu-bioinfo/PM_CNN
  tools:
  - FastTree
  - Mafft
  - PM-CNN
  - ape (R package)
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
n_claims: 8
---

# multi-path-convolutional-ensemble-classification

## Summary

Train and evaluate PM-CNN, a phylogeny-based four-path convolutional neural network whose per-path features are flattened and fused by a fully connected ensemble to classify microbiome disease status, scoring predictions with Kappa, Recall, Accuracy, Precision, and F1.

## When to use

Use when classifying microbiome samples (OTU/species abundance table + sample metadata) into disease or control status categories and you want a phylogeny-aware deep learning classifier benchmarked against ML baselines.

## When NOT to use

- When no phylogenetic tree or OTU representative sequences are available and features cannot be ordered by phylogeny, undermining the multi-path convolution design
- When the goal is only a simple taxonomic summary or differential abundance test rather than supervised status/disease classification
- When the input is not a microbial feature table (OTU/species abundance), which is a stated prerequisite for PM-CNN

## Inputs

- Microbial abundance table (OTU or species table, .csv) with samples as rows and features as columns
- Sample metadata table mapping SampleID to disease Status labels (e.g. Control/Periodontitis/Gingivitis marked 0/1/2)
- Phylogenetic tree in NEWICK format (from FastTree+Mafft or user-provided)
- Cophenetic distance matrix (.csv) from get_distance_matrix.R
- Hierarchical clustering result file (clustered_groups) from clustering.py

## Outputs

- Trained PM-CNN model (four-path CNN with fully connected ensemble fusion)
- Per-category classification predictions via softmax on held-out samples
- Kappa, Recall, Accuracy, Precision, and F1 metric values (e.g. written to pm_cnn_metrics.csv)

## How to apply

Reorder OTU features so phylogenetically related microbes are adjacent: obtain the phylogenetic tree (Mafft alignment plus FastTree, or a user-supplied NEWICK file), compute the cophenetic distance matrix with the cophenetic function in the R package ape, transform the distance matrix into a correlation matrix, and run hierarchical clustering to get the multi-layer clustered feature order (clustering.py). Then train the four-path CNN with torch (>= 1.11.0) using the clustered feature order, passing flattened path features into the fully connected ensemble fusion with a softmax output over status categories. Test on the held-out split with PMCNN.py --test and compute Kappa, Recall, Accuracy, Precision, and F1 with scikit-learn (>= 1.1.1) metrics, using numpy (>= 1.22.3) for array handling. Input must be a microbial feature table (OTU or species table) with samples as rows plus a metadata file mapping SampleID to Status; the clustered_groups argument comes from Step 3.

## Related tools

- **PM-CNN** (Phylogenetic multi-path CNN framework being trained and tested for microbiome status classification) — https://github.com/qdu-bioinfo/PM_CNN
- **torch** (Deep learning backend (>= 1.11.0) for training and running the four-path CNN and ensemble fusion)
- **scikit-learn** (Metrics library (>= 1.1.1) for computing Kappa, Recall, Accuracy, Precision, and F1 on test predictions)
- **numpy** (Array handling (>= 1.22.3) for abundance matrices and predictions)
- **ape (R package)** (Provides the cophenetic function to derive the cophenetic distance matrix from the phylogenetic tree)
- **FastTree** (Builds the phylogenetic tree from Mafft-aligned OTU representative sequences)
- **Mafft** (Multiple sequence alignment of OTU representative sequences prior to tree building)

## Examples

```
python PMCNN.py --train --train_otu_table ../example/train_data/example_abundance_table.csv --meta ../example/train_data/meta.csv
```

## Evaluation signals

- All five metrics (Kappa, Recall, Accuracy, Precision, F1) are present in the output file pm_cnn_metrics.csv
- Per Results 3.2, PM-CNN's scores should show superiority over existing ML baselines on the human microbiome classification task
- Prediction labels match the expected category encodings (e.g. 0/1/2 for Control/Periodontitis/Gingivitis on Dataset 1)
- Test predictions fall within the label set defined by the training metadata Status column
- Feature order supplied to the CNN matches the multi-layer hierarchical clustering output from Step 3

## Limitations

- Building the phylogenetic tree from representative sequences requires Mafft and FastTree plus GreenGenes lookups, though a NEWICK tree can be supplied instead
- Model quality depends on the clustered feature order; skipping Steps 1-3 removes the phylogenetic adjacency that the multi-path convolutions exploit
- Reported superiority is demonstrated on the human gut (3113 samples, 5597 OTUs) and human oral (1587 samples, 1554 OTUs) datasets; extreme class imbalance (e.g. CRC with only 120 samples) may affect results

## Evidence

- [abstract] PM-CNN organizes microbes based on their phylogenetic relationships and extracts features using a multi-path convolutional neural network. An ensemble learning method then fuses these features: "PM-CNN organizes microbes based on their phylogenetic relationships and extracts features using a multi-path convolutional neural network. An ensemble learning method then fuses these features"
- [methods] The cophenetice distance matrix is obtained by using the cophenetic function in the R package ape: "The cophenetice distance matrix is obtained by using the cophenetic function in the R package ape"
- [intro] transform the distance matrix into the correlation matrix by the designed distance transformation formula, and then carry out hierarchical clustering: "transform the distance matrix into the correlation matrix by the designed distance transformation formula, and then carry out hierarchical clustering"
- [readme] python PMCNN.py --train --train_otu_table ../example/train_data/example_abundance_table.csv --meta ../example/train_data/meta.csv: "python PMCNN.py --train --train_otu_table ../example/train_data/example_abundance_table.csv --meta ../example/train_data/meta.csv"
- [supplementary] The human oral microbiome data contains 1587 samples with 1554 OTUs. Contains three label types: Control(653), Periodontitis(274), and Gingivitis(660): "The human oral microbiome data contains 1587 samples with 1554 OTUs. Contains three label types: Control(653), Periodontitis(274), and Gingivitis(660)"

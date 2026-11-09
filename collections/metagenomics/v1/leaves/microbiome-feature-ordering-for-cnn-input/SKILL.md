---
name: microbiome-feature-ordering-for-cnn-input
description: Use when you have a cophenetic distance matrix from a phylogenetic tree
  (e.g., from the R ape package) of OTU features and need to reorder/group them phylogenetically
  before training a CNN-based microbiome classifier.
when_to_use_negative:
- No phylogenetic tree or cophenetic distance matrix exists for the features (e.g.,
  feature table lacks OTU representative sequences or a NEWICK tree was not built/provided)
- The feature table is already ordered by a prior clustering run and only model training
  is needed (proceed directly to PMCNN.py --train)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: numpy
  role: Array operations for applying the Gaussian-kernel distance transformation
    (Eq. 1) to the cophenetic distance matrix
  repo: https://github.com/qdu-bioinfo/PM_CNN
- name: scipy
  role: scipy.cluster.hierarchy to build the dendrogram and generate multi-layer clustering
    cuts from the correlation matrix
  repo: https://github.com/qdu-bioinfo/PM_CNN
- name: scikit-learn
  role: Silhouette coefficient metric to score and select the optimal clustering layer
  repo: https://github.com/qdu-bioinfo/PM_CNN
- name: ape (R package)
  role: Upstream provider of the cophenetic distance matrix consumed by this skill
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbae013
    title: 'PM-CNN: microbiome status recognition and disease detection model based
      on phylogeny and multi-path neural network'
  source_content_hash: sha256:919a40d35b3f1175920b7aabc7225d12df10d38562ff33934e62fae0517f4c0f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qdu-bioinfo/PM_CNN
  tools:
  - PM-CNN
  - ape (R package)
  - numpy
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# microbiome-feature-ordering-for-cnn-input

## Summary

Transforms a cophenetic distance matrix into a Gaussian-kernel-normalized correlation matrix, then performs hierarchical clustering with silhouette-based layer selection to produce phylogeny-sorted cluster feature orderings (large/medium/small layers at 1/20, 1/100, 1/500 features per cluster) for PM-CNN input.

## When to use

Use when you have a cophenetic distance matrix from a phylogenetic tree (e.g., from the R ape package) of OTU features and need to reorder/group them phylogenetically before training a CNN-based microbiome classifier.

## When NOT to use

- No phylogenetic tree or cophenetic distance matrix exists for the features (e.g., feature table lacks OTU representative sequences or a NEWICK tree was not built/provided)
- The feature table is already ordered by a prior clustering run and only model training is needed (proceed directly to PMCNN.py --train)

## Inputs

- Cophenetic distance matrix in .csv format, from Step 2 (ape cophenetic function applied to a NEWICK tree built by Mafft/FastTree)
- OTU feature list from the microbial abundance table (e.g., OTU table rows/columns)

## Outputs

- Hierarchical clustering results / ordered OTU cluster assignments written to cluster_feature_order.csv (used as --clustered_groups in PM-CNN training)
- Optimal clustering layer selected by maximum silhouette coefficient, plus large (1/20), medium (1/100), and small (1/500) granularity layers

## How to apply

Load the cophenetic distance matrix (.csv, produced upstream by the cophenetic function in the R package ape) and apply Eq. 1, the Gaussian-kernel distance transformation, using numpy/scipy to convert distances into a normalized phylogenetic correlation matrix. Run hierarchical clustering (scipy.cluster.hierarchy) on the correlation matrix to obtain candidate multi-layer cuts, including large, medium, and small granularity layers sized at 1/20, 1/100, and 1/500 of features per cluster. Score each candidate layer with the silhouette coefficient (scikit-learn) and select the layer with the highest score as the optimal partition, alongside the three granularity layers. Emit the ordered OTU cluster assignments to cluster_feature_order.csv, which is passed as the --clustered_groups argument to PMCNN.py in Step 4 of the PM-CNN pipeline.

## Related tools

- **numpy** (Array operations for applying the Gaussian-kernel distance transformation (Eq. 1) to the cophenetic distance matrix) — https://github.com/qdu-bioinfo/PM_CNN
- **scipy** (scipy.cluster.hierarchy to build the dendrogram and generate multi-layer clustering cuts from the correlation matrix) — https://github.com/qdu-bioinfo/PM_CNN
- **scikit-learn** (Silhouette coefficient metric to score and select the optimal clustering layer) — https://github.com/qdu-bioinfo/PM_CNN
- **ape (R package)** (Upstream provider of the cophenetic distance matrix consumed by this skill)

## Examples

```
python clustering.py --input /root/user/PM_CNN/example/example_distance_matrix.csv --output /root/user/PM_CNN/example/output_file.csv
```

## Evaluation signals

- Every OTU feature receives a non-empty cluster assignment in cluster_feature_order.csv
- The selected layer corresponds to the maximum silhouette coefficient among all evaluated candidate layers
- Three granularity layers are emitted at approximately 1/20, 1/100, and 1/500 of features per cluster (large/medium/small)
- Input file is a valid .csv cophenetic distance matrix from Step 2; output is consumable as the --clustered_groups argument of PMCNN.py --train

## Limitations

- Requires a phylogenetic tree (NEWICK) for the OTUs; representative sequences must be matched from the GreenGenes database when building the tree via Mafft/FastTree
- The paper/README do not publish the exact Eq. 1 Gaussian-kernel parameters, so sigma must be recovered from the repository code
- Silhouette-based layer selection may fail (or be ill-defined) if all candidate cuts are trivial (e.g., single cluster or singleton clusters)

## Evidence

- [methods] Step 3: Distance transformation and hierarchical clustering: "Step 3: Distance transformation and hierarchical clustering"
- [intro] distance transformation then hierarchical clustering: "transform the distance matrix into the correlation matrix by the designed distance transformation formula, and then carry out hierarchical clustering"
- [readme] cophenetic matrix source: "[--input] //Input cophenetic distance matrix .csv format file (This comes from the results generated by step 2)"
- [readme] clustered order feeds training: "Now, after the processing of Steps 1-3, we now have the clustered feature order. Next, we will start training our model."

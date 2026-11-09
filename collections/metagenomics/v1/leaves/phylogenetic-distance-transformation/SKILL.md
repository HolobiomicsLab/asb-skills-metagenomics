---
name: phylogenetic-distance-transformation
description: Use when you have a cophenetic distance matrix (from ape's cophenetic
  function) over microbial features and need phylogeny-aware feature ordering/clustering
  before model training.
when_to_use_negative:
- No phylogenetic tree or cophenetic distance matrix is available for the microbial
  features (the transformation requires tree-derived distances)
- The input is already a phylogenetically ordered or clustered feature table produced
  by Step 3
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: numpy
  role: Array computation for applying the Gaussian-kernel distance transformation
    formula to the distance matrix
- name: scipy
  role: Hierarchical clustering over the transformed correlation matrix to obtain
    multi-layer clustering results
- name: scikit-learn
  role: Silhouette coefficient computation to select the optimal clustering layer
- name: ape (R package)
  role: Upstream provider of the cophenetic distance matrix via the cophenetic function
- name: PM-CNN
  role: Downstream consumer of the clustered feature order for model training
  repo: https://github.com/qdu-bioinfo/PM_CNN
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbae013
    title: 'PM-CNN: microbiome status recognition and disease detection model based
      on phylogeny and multi-path neural network'
  source_content_hash: sha256:7717c701f0610946ae3465b57c80f7e84dc5469b43eadb8429f7398e530435ef
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
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# phylogenetic-distance-transformation

## Summary

Transforms a cophenetic distance matrix from a phylogenetic tree into a Gaussian-kernel-normalized correlation matrix, then applies hierarchical clustering to produce multi-layer phylogeny-sorted clusterings of OTU features for PM-CNN.

## When to use

Use when you have a cophenetic distance matrix (from ape's cophenetic function) over microbial features and need phylogeny-aware feature ordering/clustering before model training.

## When NOT to use

- No phylogenetic tree or cophenetic distance matrix is available for the microbial features (the transformation requires tree-derived distances)
- The input is already a phylogenetically ordered or clustered feature table produced by Step 3

## Inputs

- Cophenetic distance matrix in .csv format (output of get_distance_matrix.R from a NEWICK phylogenetic tree)
- Microbial feature table (OTU table or species table) whose features are being ordered

## Outputs

- Hierarchical clustering results file (e.g. output_file.csv / cluster_feature_order.csv) with ordered OTU assignments for the optimal layer plus large/medium/small granularity layers
- Clustered feature order consumed by PMCNN.py via the --clustered_groups argument

## How to apply

Load the cophenetic distance matrix (.csv) computed upstream from a NEWICK phylogenetic tree (built via Mafft and FastTree, or user-provided) using the cophenetic function in the R package ape. Apply the paper's designed distance transformation formula (Eq. 1, Gaussian kernel) with numpy/scipy to convert the distance matrix into a normalized correlation matrix measuring phylogenetic relatedness between OTUs. Run hierarchical clustering (scipy.cluster.hierarchy) on the correlation matrix to obtain candidate multi-layer cuts, then evaluate each layer with the silhouette coefficient (scikit-learn) and select the layer with the highest score as the optimal partition. Also emit large, medium, and small granularity layers sized at 1/20, 1/100, and 1/500 of features per cluster, yielding four phylogeny-sorted forms of the abundance table for PM-CNN training.

## Related tools

- **numpy** (Array computation for applying the Gaussian-kernel distance transformation formula to the distance matrix)
- **scipy** (Hierarchical clustering over the transformed correlation matrix to obtain multi-layer clustering results)
- **scikit-learn** (Silhouette coefficient computation to select the optimal clustering layer)
- **ape (R package)** (Upstream provider of the cophenetic distance matrix via the cophenetic function)
- **PM-CNN** (Downstream consumer of the clustered feature order for model training) — https://github.com/qdu-bioinfo/PM_CNN

## Examples

```
python clustering.py --input /root/user/PM_CNN/example/example_distance_matrix.csv --output /root/user/PM_CNN/example/output_file.csv
```

## Evaluation signals

- Every OTU feature receives a non-empty cluster assignment in the emitted ordering
- The selected clustering layer corresponds to the maximum silhouette coefficient among all evaluated layers
- The output is a .csv hierarchical clustering result consumable as --clustered_groups input to PMCNN.py
- The large/medium/small granularity layers contain 1/20, 1/100, and 1/500 of features per cluster respectively

## Limitations

- Depends on a valid phylogenetic tree (NEWICK) and cophenetic distance matrix from Step 2; cannot run on raw abundance tables alone
- Optimal layer selection is driven solely by the silhouette coefficient, which may not always align with downstream classification performance

## Evidence

- [intro] transform the distance matrix into the correlation matrix by the designed distance transformation formula, and then carry out hierarchical clustering: "transform the distance matrix into the correlation matrix by the designed distance transformation formula, and then carry out hierarchical clustering"
- [methods] Step 3: Distance transformation and hierarchical clustering: "Step 3: Distance transformation and hierarchical clustering"
- [intro] The cophenetice distance matrix is obtained by using the cophenetic function in the R package ape: "The cophenetice distance matrix is obtained by using the cophenetic function in the R package ape"
- [readme] [--input] //Input cophenetic distance matrix .csv format file (This comes from the results generated by step 2): "[--input] //Input cophenetic distance matrix .csv format file (This comes from the results generated by step 2)"

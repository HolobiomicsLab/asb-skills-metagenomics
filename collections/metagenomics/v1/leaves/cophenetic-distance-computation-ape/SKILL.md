---
name: cophenetic-distance-computation-ape
description: Use when you have a phylogenetic tree in NEWICK format (from FastTree/Mafft
  or user-supplied) covering all OTUs/species of a microbial feature table and need
  a pairwise phylogenetic distance matrix before correlation transformation and hierarchical
  clustering.
when_to_use_negative:
- The input is already a cophenetic distance matrix or correlation matrix rather than
  a NEWICK tree (proceed directly to distance transformation/clustering, Step 3).
- No phylogenetic structure is relevant — e.g. taxa lack a trustworthy tree — since
  PM-CNN's ordering depends on the phylogenetic correlation derived from this matrix.
- The tree tips do not match the OTU IDs in the feature table, so cophenetic distances
  cannot be aligned to the abundance table.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_0637
tools:
- name: ape (R package)
  role: Provides the cophenetic function used to compute the OTU-by-OTU distance matrix
    from the NEWICK tree; called by the PM-CNN script get_distance_matrix.R.
- name: FastTree
  role: Optionally builds the NEWICK phylogenetic tree (Step 1) from Mafft-aligned
    representative sequences, which this skill then converts to a distance matrix.
- name: Mafft
  role: Performs multiple sequence alignment of GreenGenes representative sequences
    that FastTree uses to build the input tree.
- name: PM-CNN
  role: Downstream framework (Step 3-4) that consumes the cophenetic distance matrix
    for correlation transformation, hierarchical clustering, and multi-path CNN classification.
  repo: https://github.com/qdu-bioinfo/PM_CNN
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbae013
    title: 'PM-CNN: microbiome status recognition and disease detection model based
      on phylogeny and multi-path neural network'
  source_content_hash: sha256:62164c1e36bd0ec220d9d150f02c36b3be85810e29787cc94719e4f69a0d0df1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qdu-bioinfo/PM_CNN
  tools:
  - FastTree
  - Mafft
  - PM-CNN
  - R >=
  - ape (R package)
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# cophenetic-distance-computation-ape

## Summary

Computes the pairwise microbe-microbe cophenetic distance matrix from a NEWICK phylogenetic tree using the cophenetic function in the R package ape, as Step 2 of the PM-CNN pipeline (DOI 10.1093/bioadv/vbae013). Each entry sums the branch distances from two tip nodes to their most recent common ancestor (Eq. 2).

## When to use

Use when you have a phylogenetic tree in NEWICK format (from FastTree/Mafft or user-supplied) covering all OTUs/species of a microbial feature table and need a pairwise phylogenetic distance matrix before correlation transformation and hierarchical clustering.

## When NOT to use

- The input is already a cophenetic distance matrix or correlation matrix rather than a NEWICK tree (proceed directly to distance transformation/clustering, Step 3).
- No phylogenetic structure is relevant — e.g. taxa lack a trustworthy tree — since PM-CNN's ordering depends on the phylogenetic correlation derived from this matrix.
- The tree tips do not match the OTU IDs in the feature table, so cophenetic distances cannot be aligned to the abundance table.

## Inputs

- Phylogenetic tree file in NEWICK format (e.g. example.tree from FastTree, or user-provided for shotgun data)
- Microbial feature table (OTU table or species table) whose OTUs must all appear as tree tips

## Outputs

- Pairwise OTU-by-OTU cophenetic distance matrix in .csv format (e.g. example_distance_matrix.csv), symmetric with zero diagonal

## How to apply

Read the NEWICK tree into R (R >= 4.2.1) with the ape package's read.tree, then confirm the tip labels cover every OTU in the abundance table. Apply cophenetic(tree) to obtain the pairwise distance matrix, where each entry is the sum of branch lengths from two tips to their MRCA. Verify the result is square and symmetric with a zero diagonal and dimensions equal to the number of tree tips, then write it to a .csv file (e.g. example_distance_matrix.csv). In the PM-CNN workflow this matrix is the required input for the next step, which transforms it into a correlation matrix via the designed distance transformation formula before hierarchical clustering produces multi-layer clustering groups.

## Related tools

- **ape (R package)** (Provides the cophenetic function used to compute the OTU-by-OTU distance matrix from the NEWICK tree; called by the PM-CNN script get_distance_matrix.R.)
- **FastTree** (Optionally builds the NEWICK phylogenetic tree (Step 1) from Mafft-aligned representative sequences, which this skill then converts to a distance matrix.)
- **Mafft** (Performs multiple sequence alignment of GreenGenes representative sequences that FastTree uses to build the input tree.)
- **PM-CNN** (Downstream framework (Step 3-4) that consumes the cophenetic distance matrix for correlation transformation, hierarchical clustering, and multi-path CNN classification.) — https://github.com/qdu-bioinfo/PM_CNN

## Examples

```
Rscript get_distance_matrix.R "/root/user/PM_CNN/example" "example.tree" "example_distance_matrix.csv"
```

## Evaluation signals

- Output matrix is square with dimensions equal to the number of tips in the input NEWICK tree.
- Matrix is symmetric (D[i,j] == D[j,i]) and the diagonal is all zeros.
- All OTU IDs from the feature table appear exactly once as row/column names of the distance matrix.
- Every off-diagonal entry is a non-negative branch-length sum (distance from two tips to their MRCA), consistent with Eq. 2 of the paper.
- The .csv loads cleanly into clustering.py as the [--input] cophenetic distance matrix without missing values.

## Limitations

- This step is optional only in the sense that the tree can be user-supplied; a NEWICK-format phylogeny covering the dataset's OTUs is still required, typically built via Mafft + FastTree from GreenGenes representative sequences for amplicon data.
- For shotgun data the tree must be provided by the user in NEWICK format; the FastTree/Mafft route assumes GreenGenes-referenced representative sequences.
- The skill only produces the distance matrix — correlation transformation and hierarchical clustering are handled separately by clustering.py in Step 3.

## Evidence

- [methods] The cophenetic distance between two microbes is computed as the sum of the distances from each tip node in the phylogenetic tree to their most recent common ancestor (MRCA): [evidence span withheld — anchor status: not_found]
- [intro] The cophenetice distance matrix is obtained by using the cophenetic function in the R package ape: "The cophenetice distance matrix is obtained by using the cophenetic function in the R package ape"
- [readme] Rscript get_distance_matrix.R "/root/user/PM_CNN/example" "example.tree" "example_distance_matrix.csv": "Rscript get_distance_matrix.R "/root/user/PM_CNN/example" "example.tree" "example_distance_matrix.csv""
- [intro] transform the distance matrix into the correlation matrix by the designed distance transformation formula, and then carry out hierarchical clustering: "transform the distance matrix into the correlation matrix by the designed distance transformation formula, and then carry out hierarchical clustering"

---
name: phylogenetic-tree-parsing-newick
description: Use when a phylogenetic tree exists in NEWICK format (from FastTree/Mafft
  or user-provided) and a pairwise tip-to-tip distance matrix is needed to encode
  phylogenetic relationships between OTUs/species.
when_to_use_negative:
- The phylogenetic relationships are already encoded as a distance/correlation matrix
  and the tree need not be re-parsed
- The input is a shotgun metagenomic feature table with no NEWICK tree available and
  no reference sequences from which to build one via Mafft/FastTree
edam_operation: http://edamontology.org/operation_0325
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_0637
tools:
- name: ape (R package)
  role: Provides the cophenetic function used to compute the pairwise microbe-microbe
    cophenetic distance matrix from the NEWICK tree
- name: PM-CNN
  role: Downstream framework that consumes the cophenetic distance matrix for distance
    transformation, hierarchical clustering, and multi-path CNN feature ordering
  repo: https://github.com/qdu-bioinfo/PM_CNN
- name: FastTree
  role: Optionally builds the NEWICK phylogenetic tree from the Mafft multiple-sequence
    alignment prior to cophenetic distance computation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbae013
    title: 'PM-CNN: microbiome status recognition and disease detection model based
      on phylogeny and multi-path neural network'
  source_content_hash: sha256:34f5d2eef1f4458abc66d8129462fc560c4c8b0ce877ccb2a62514c7833473d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qdu-bioinfo/PM_CNN
  tools:
  - FastTree
  - PM-CNN
  - R >=
  - ape (R package)
  edam_operation: http://edamontology.org/operation_0325
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# phylogenetic-tree-parsing-newick

## Summary

Parse a NEWICK-format phylogenetic tree and compute the pairwise microbe-microbe cophenetic distance matrix using the cophenetic function in the R package ape, as done in PM-CNN to convert evolutionary tree structure into a numerical distance matrix for downstream clustering.

## When to use

Use when a phylogenetic tree exists in NEWICK format (from FastTree/Mafft or user-provided) and a pairwise tip-to-tip distance matrix is needed to encode phylogenetic relationships between OTUs/species.

## When NOT to use

- The phylogenetic relationships are already encoded as a distance/correlation matrix and the tree need not be re-parsed
- The input is a shotgun metagenomic feature table with no NEWICK tree available and no reference sequences from which to build one via Mafft/FastTree

## Inputs

- Phylogenetic tree file in NEWICK format (e.g. example.tree from FastTree)
- Optional: representative OTU sequences (e.g. marker gene or amplicon) aligned with Mafft and tree-built with FastTree if the tree is not already provided

## Outputs

- Cophenetic distance matrix in .csv format (square, symmetric, zero diagonal, dimensions equal to number of tree tips)

## How to apply

Read the NEWICK tree into R with the ape package (read.tree), which may be the example.tree produced by FastTree from the Mafft multiple-sequence alignment of GreenGenes-derived representative sequences. Apply the ape cophenetic function (cophenetic(tree)), which computes, for each pair of tips, the sum of branch distances from each tip to their most recent common ancestor (MRCA), yielding a square pairwise matrix (Eq. 2 of PM-CNN). Verify the matrix is symmetric with a zero diagonal and dimensions equal to the number of tree tips. Write the result to a .csv file (e.g. via get_distance_matrix.R with parameters: working directory, input tree file, output csv), which then feeds the distance transformation and hierarchical clustering step of PM-CNN.

## Related tools

- **ape (R package)** (Provides the cophenetic function used to compute the pairwise microbe-microbe cophenetic distance matrix from the NEWICK tree)
- **PM-CNN** (Downstream framework that consumes the cophenetic distance matrix for distance transformation, hierarchical clustering, and multi-path CNN feature ordering) — https://github.com/qdu-bioinfo/PM_CNN
- **FastTree** (Optionally builds the NEWICK phylogenetic tree from the Mafft multiple-sequence alignment prior to cophenetic distance computation)

## Examples

```
Rscript get_distance_matrix.R "/root/user/PM_CNN/example" "example.tree" "example_distance_matrix.csv"
```

## Evaluation signals

- Output matrix is square with row and column counts equal to the number of tips in the input NEWICK tree
- Matrix is symmetric (entry [i,j] equals entry [j,i])
- Matrix diagonal is all zeros (each tip's distance to itself is zero)
- Tip labels extracted via read.tree cover all OTUs/microbes present in the feature table
- Matrix values are non-negative branch distances suitable for the subsequent distance transformation formula

## Limitations

- This step depends on a valid NEWICK tree; for shotgun data the tree may be missing and must be user-provided, and building one requires representative sequences findable in the GreenGenes database
- Tree construction (Mafft + FastTree) is marked as optional in the PM-CNN README, so the pipeline cannot run this step without either a pre-built tree or the reference sequences to build one
- No changelog found for the repository, so version-specific behavior of get_distance_matrix.R is undocumented

## Evidence

- [methods] The cophenetic distance between two microbes is computed as the sum of the distances from each tip node in the phylogenetic tree to their most recent common ancestor (MRCA): [evidence span withheld — anchor status: not_found]
- [intro] The cophenetice distance matrix is obtained by using the cophenetic function in the R package ape: "The cophenetice distance matrix is obtained by using the cophenetic function in the R package ape"
- [readme] usage: PM_CNN/code/get_distance_matrix.R parameter1 parameter2 parameter3: "usage: PM_CNN/code/get_distance_matrix.R parameter1 parameter2 parameter3"
- [readme] parameter2: Input a phylogenetic tree file in NEWICK format (This comes from the results generated by FastTree in step 1): "parameter2: Input a phylogenetic tree file in NEWICK format (This comes from the results generated by FastTree in step 1)"

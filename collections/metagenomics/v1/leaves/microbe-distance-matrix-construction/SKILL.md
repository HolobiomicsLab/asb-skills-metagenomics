---
name: microbe-distance-matrix-construction
description: Use when you have a NEWICK-format phylogenetic tree (e.g. built by Mafft/FastTree
  from GreenGenes-matched OTU representative sequences) and need a pairwise cophenetic
  distance matrix between OTUs for downstream phylogeny-aware feature ordering.
when_to_use_negative:
- The phylogenetic tree has not yet been built and representative sequences are not
  available from the GreenGenes database for Mafft alignment and FastTree tree construction
- You need an abundance-level beta-diversity (sample-by-sample) distance rather than
  a microbe-by-microbe phylogenetic distance; this skill operates on tree tips (OTUs),
  not samples
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: ape (R package)
  role: Provides the cophenetic function used to obtain the cophenetic distance matrix
    from the phylogenetic tree
- name: PM-CNN
  role: Downstream pipeline that consumes the cophenetic distance matrix for distance
    transformation, hierarchical clustering, and multi-path CNN training
  repo: https://github.com/qdu-bioinfo/PM_CNN
- name: FastTree
  role: Optionally builds the input NEWICK phylogenetic tree from Mafft-aligned OTU
    representative sequences
- name: Mafft
  role: Performs multiple sequence alignment of OTU representative sequences prior
    to tree building
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbae013
    title: 'PM-CNN: microbiome status recognition and disease detection model based
      on phylogeny and multi-path neural network'
  source_content_hash: sha256:d596f431f36ddb427b2ec3ae01c9163f5c6750f22448eb487863479477ecc753
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# microbe-distance-matrix-construction

## Summary

Constructs the pairwise microbe-microbe cophenetic distance matrix from a NEWICK phylogenetic tree using the cophenetic function in the R package ape, a required step in the PM-CNN pipeline before distance transformation and hierarchical clustering.

## When to use

Use when you have a NEWICK-format phylogenetic tree (e.g. built by Mafft/FastTree from GreenGenes-matched OTU representative sequences) and need a pairwise cophenetic distance matrix between OTUs for downstream phylogeny-aware feature ordering.

## When NOT to use

- The phylogenetic tree has not yet been built and representative sequences are not available from the GreenGenes database for Mafft alignment and FastTree tree construction
- You need an abundance-level beta-diversity (sample-by-sample) distance rather than a microbe-by-microbe phylogenetic distance; this skill operates on tree tips (OTUs), not samples

## Inputs

- Phylogenetic tree file in NEWICK format (e.g. example.tree, produced by FastTree in Step 1 or user-provided for shotgun data)
- Current working directory path
- Output filename for the cophenetic distance matrix (.csv)

## Outputs

- Cophenetic distance matrix in .csv format (square, symmetric, zero diagonal, dimensions equal to number of tree tips/OTUs)

## How to apply

Read the NEWICK phylogenetic tree into R (R >= 4.2.1) using ape's read.tree, then apply the ape cophenetic function, which computes for each pair of tips the sum of distances from each tip to their most recent common ancestor (MRCA), yielding Eq. 2's pairwise matrix. Verify the matrix is square, symmetric, has a zero diagonal, and has row/column count equal to the number of tree tips (matching the OTU count in the abundance table). Write the result to a .csv file for the next pipeline step (distance transformation to a correlation matrix followed by hierarchical clustering).

## Related tools

- **ape (R package)** (Provides the cophenetic function used to obtain the cophenetic distance matrix from the phylogenetic tree)
- **PM-CNN** (Downstream pipeline that consumes the cophenetic distance matrix for distance transformation, hierarchical clustering, and multi-path CNN training) — https://github.com/qdu-bioinfo/PM_CNN
- **FastTree** (Optionally builds the input NEWICK phylogenetic tree from Mafft-aligned OTU representative sequences)
- **Mafft** (Performs multiple sequence alignment of OTU representative sequences prior to tree building)

## Examples

```
Rscript get_distance_matrix.R "/root/user/PM_CNN/example" "example.tree" "example_distance_matrix.csv"
```

## Evaluation signals

- Output matrix is square with row/column count equal to the number of tips in the input NEWICK tree
- Matrix is symmetric (entry[i][j] equals entry[j][i])
- Matrix diagonal is zero (each tip's distance to itself is 0)
- All entries are non-negative distances
- Downstream Step 3 (clustering.py) can consume the .csv directly via its --input argument without schema errors

## Limitations

- Requires a NEWICK-format tree covering all OTUs in the feature table; the tree can come from FastTree (amplicon/marker gene data) or be user-provided (e.g. for shotgun), so tree availability and quality constrain this step
- Cophenetic distances derive solely from the tree topology and branch lengths, not from abundance data

## Evidence

- [intro] cophenetic function in R ape: "The cophenetice distance matrix is obtained by using the cophenetic function in the R package ape"
- [readme] README usage of get_distance_matrix.R: "usage: PM_CNN/code/get_distance_matrix.R parameter1 parameter2 parameter3"
- [readme] README example invocation: "Rscript get_distance_matrix.R "/root/user/PM_CNN/example" "example.tree" "example_distance_matrix.csv""
- [readme] Tree provenance in README: "The evolutionary tree of the representative sequence is constructed by FastTree and Mafft."
- [methods] Task finding on MRCA: [evidence span withheld — anchor status: not_found]

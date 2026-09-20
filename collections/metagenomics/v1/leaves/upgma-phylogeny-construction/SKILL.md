---
name: upgma-phylogeny-construction
description: Use when you have a pairwise APSS (average pairwise synteny score) matrix
  from SynTracker and need a hierarchical strain phylogeny, or when validating a synteny-based
  tree against a reference tree (e.g., a Mash-based phylogroup tree).
when_to_use_negative:
- When the goal is SNP-level strain differentiation only — SynTracker's synteny-based
  approach is designed to capture structural changes and is deliberately insensitive
  to SNPs
- When comparing strains of different species — the pipeline is designated for conspecific
  (same-species) strains
- When a reference genome cannot yield enough homologous regions for a pair (fewer
  than n regions), as those pairs are excluded from downstream analysis
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0194
tools:
- name: phangorn R package
  role: Builds the UPGMA phylogenetic trees from the synteny distance matrices
- name: TreeDist R package
  role: Computes Information-based generalized Robinson-Foulds distances between synteny-based
    and reference trees with label shuffling for empirical P-values
- name: SynTracker
  role: Generates the per-region and APSS synteny score tables that are converted
    to distances for tree building
  repo: https://github.com/leylabmpi/SynTracker
- name: Mash
  role: Source of the reference phylogroup tree used for tree-comparison validation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41587-024-02276-2
    title: Strain tracking in complex microbiomes using synteny analysis reveals per-species
      modes of evolution
  source_content_hash: sha256:cb9f4a2d1bbaa1be2d7785e7e970c1eccbbfd6b9bf7f39048d04b320914cc8f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leylabmpi/SynTracker
  tools:
  - Mash
  - SynTracker
  - TreeDist R package
  - phangorn R package
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0194
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# upgma-phylogeny-construction

## Summary

Constructs UPGMA phylogenetic trees from Average Pairwise Synteny Score (APSS) matrices produced by SynTracker, converting APSS to synteny distances (|APSS − 1|) so that strain relatedness across genomes or metagenomic assemblies can be recapitulated as dendrograms.

## When to use

Use when you have a pairwise APSS (average pairwise synteny score) matrix from SynTracker and need a hierarchical strain phylogeny, or when validating a synteny-based tree against a reference tree (e.g., a Mash-based phylogroup tree).

## When NOT to use

- When the goal is SNP-level strain differentiation only — SynTracker's synteny-based approach is designed to capture structural changes and is deliberately insensitive to SNPs
- When comparing strains of different species — the pipeline is designated for conspecific (same-species) strains
- When a reference genome cannot yield enough homologous regions for a pair (fewer than n regions), as those pairs are excluded from downstream analysis

## Inputs

- SynTracker APSS matrix (avg_synteny_scores_[n]_regions.csv or avg_synteny_scores_all_regions.csv)
- Number of regions to subsample per pair (n = 20, 40, 60, 80, 100, 200)
- Optional reference tree in Newick format (e.g., pruned Mash-based tree, Supplementary_Data_Fig_3c_10k_newick.nwk)

## Outputs

- UPGMA phylogenetic tree (Newick) per subsampling depth built from synteny distances
- Information-based generalized Robinson-Foulds tree distances with empirical P-values from 100,000 label-shuffling iterations (e.g., ecoli_info_rf_tree_distances.csv)

## How to apply

Take SynTracker's APSS output tables (e.g., avg_synteny_scores_[n]_regions.csv for n = 20, 40, 60, 80, 100, 200 regions per pair) and convert APSS values to synteny distances equal to the absolute value of APSS − 1. Build unweighted pair group method with arithmetic mean (UPGMA) phylogenetic trees from these distances using the 'phangorn' R package, doing this separately at each subsampling depth to assess robustness. To validate the resulting trees, compute Information-based generalized Robinson-Foulds distances against a reference tree (such as a pruned Mash-based tree) using the 'TreeDist' R package, shuffling the labels of the synteny-based tree with 100,000 iterations per comparison to obtain empirical P-values. In the E. coli benchmark, this recapitulated the published Mash phylogroup tree with P < 1 × 10−5 at every subsampling depth, even sampling < 2% of the genome. Exclude sample pairs with fewer than n available regions from downstream analysis at a given depth.

## Related tools

- **phangorn R package** (Builds the UPGMA phylogenetic trees from the synteny distance matrices)
- **TreeDist R package** (Computes Information-based generalized Robinson-Foulds distances between synteny-based and reference trees with label shuffling for empirical P-values)
- **SynTracker** (Generates the per-region and APSS synteny score tables that are converted to distances for tree building) — https://github.com/leylabmpi/SynTracker
- **Mash** (Source of the reference phylogroup tree used for tree-comparison validation)

## Examples

```
Rscript -e 'library(phangorn); d <- as.dist(read.csv("avg_synteny_scores_200_regions.csv", row.names=1)); tree <- upgma(d)'
```

## Evaluation signals

- Synteny distances equal |APSS − 1| for every pairwise value in the distance matrix
- The synteny-based tree recapitulates the reference (Mash) tree with empirical P < 1e-5 (100,000 label-shuffle iterations) at every subsampling depth, including n = 20 regions
- Phylogroup clades are recovered even when < 2% of the genome is sampled
- Pairs with fewer than n available regions are excluded at each subsampling depth before tree construction
- Per-depth distances and P-values are written to the output CSV (e.g., ecoli_info_rf_tree_distances.csv)

## Limitations

- Small numbers of regions per pairwise comparison mostly result in lower accuracy; there is a trade-off between accuracy and sample size
- By default SynTracker sets a seed=1 for subsampling; using --no_seed makes APSS values (and hence trees) vary between runs
- Strain delineation thresholds and tree interpretations are subjective and species-dependent; the authors recommend continuous genomic difference measures over same/different strain classifications

## Evidence

- [discussion] APSS to synteny distance conversion: "APSS values were converted to synteny distances, which are equal to the absolute value of APSS − 1"
- [discussion] UPGMA tree construction: "unweighted pair group method with arithmetic mean phylogenetic trees using the ‘phangorn’ R package"
- [discussion] Tree comparison and significance testing: "shuffling the labels of the synteny-based tree (100,000 iterations for each comparison)"
- [discussion] Exclusion of underpowered pairs: "Pairs of samples or genomes with fewer than n regions are excluded from downstream analysis."
- [readme] APSS output file structure: "The tables containing the APSS in different subsampling lengths are called `avg_synteny_scores_[subsampling length]_regions.csv`"

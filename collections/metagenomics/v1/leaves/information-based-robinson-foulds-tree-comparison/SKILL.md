---
name: information-based-robinson-foulds-tree-comparison
description: Use when you must quantify, with an empirical P-value, how well a tree
  built from your own distance data (e.g., UPGMA trees from APSS-based synteny distances
  at a given subsampling depth n) agrees with an independently published reference
  tree over the same taxa.
when_to_use_negative:
- The candidate and reference trees do not share a common set of taxa and cannot be
  pruned to identical external nodes — the Information-based generalized RFD comparison
  requires matching leaf sets.
- The goal is to compare continuous genetic distances (e.g., Mash distances or APSS
  values) rather than tree topologies — the article itself recommends considering
  continuous genomic difference measures instead of same/different classifications.
- Pairs have fewer than n regions available (pairs with fewer than n regions are excluded
  from downstream analysis, so no synteny-based tree can be built at that depth).
edam_operation: http://edamontology.org/operation_0554
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: TreeDist
  role: Calculates Information-based generalized Robinson-Foulds distances between
    the synteny-based trees and the pruned reference tree
- name: phangorn
  role: Builds the UPGMA phylogenetic trees from synteny distances (|APSS − 1|) at
    each subsampling depth before comparison
- name: SynTracker
  role: Produces the pairwise synteny scores per region and APSS tables that are converted
    to synteny distances and clustered into the trees being compared
  repo: https://github.com/leylabmpi/SynTracker
- name: Mash
  role: Source of the published reference phylogroup tree (>10,000 genomes) that is
    pruned and used as the comparison target
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41587-024-02276-2
    title: Strain tracking in complex microbiomes using synteny analysis reveals per-species
      modes of evolution
  source_content_hash: sha256:2ae7c492f537e1f7e0b27b68d35bac3322e93c289e1f78894f56076cf7ea6962
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leylabmpi/SynTracker
  tools:
  - Mash
  - SynTracker
  - TreeDist
  - TreeDist R package
  - phangorn
  - phangorn R package
  edam_operation: http://edamontology.org/operation_0554
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# information-based-robinson-foulds-tree-comparison

## Summary

Statistically tests whether a tree reconstructed from pairwise synteny distances (|APSS − 1|, UPGMA via phangorn) recapitulates a reference phylogeny (e.g., a pruned Mash-based phylogroup tree) by computing Information-based generalized Robinson-Foulds distances with the TreeDist R package and assigning empirical P-values by label shuffling (100,000 iterations).

## When to use

Use when you must quantify, with an empirical P-value, how well a tree built from your own distance data (e.g., UPGMA trees from APSS-based synteny distances at a given subsampling depth n) agrees with an independently published reference tree over the same taxa.

## When NOT to use

- The candidate and reference trees do not share a common set of taxa and cannot be pruned to identical external nodes — the Information-based generalized RFD comparison requires matching leaf sets.
- The goal is to compare continuous genetic distances (e.g., Mash distances or APSS values) rather than tree topologies — the article itself recommends considering continuous genomic difference measures instead of same/different classifications.
- Pairs have fewer than n regions available (pairs with fewer than n regions are excluded from downstream analysis, so no synteny-based tree can be built at that depth).

## Inputs

- Synteny distance matrices derived from APSS tables (e.g., avg_synteny_scores_[n]_regions.csv from SynTracker, distances = |APSS − 1|)
- Synteny-based phylogenetic trees (UPGMA trees built with phangorn at each subsampling depth n = 20, 40, 60, 80, 100, 200 regions)
- Reference tree in Newick format (e.g., Mash-based phylogroup tree, Supplementary_Data_Fig_3c_10k_newick.nwk), pruned to the analyzed taxa

## Outputs

- Information-based generalized Robinson-Foulds distances between each synteny-based tree and the pruned reference tree
- Empirical P-values per comparison from 100,000 label-shuffling iterations (e.g., P < 1e-5 at every subsampling depth)
- Per-depth results table (e.g., ecoli_info_rf_tree_distances.csv)

## How to apply

First build the candidate trees: convert APSS values to synteny distances equal to the absolute value of APSS − 1, then infer unweighted pair group method with arithmetic mean (UPPGMA) phylogenetic trees with the phangorn R package at each subsampling depth (e.g., n = 20, 40, 60, 80, 100, 200 regions per pair). Prune the reference tree (e.g., the Mash-based E. coli phylogroup newick, Supplementary_Data_Fig_3c_10k_newick.nwk) so only external nodes corresponding to the analyzed taxa are kept, ensuring identical leaf sets. Then, for each synteny-based tree, calculate Information-based generalized Robinson-Foulds distances (RFDs) against the pruned reference tree using the TreeDist R package, and shuffle the labels of the synteny-based tree 100,000 iterations per comparison to derive empirical P-values. The comparison succeeds when the observed RFD is more extreme than all (or nearly all) shuffled distances (P < 1e-5 in the E. coli recapitulation at every depth), and per-depth distances and P-values are written to a table such as ecoli_info_rf_tree_distances.csv.

## Related tools

- **TreeDist** (Calculates Information-based generalized Robinson-Foulds distances between the synteny-based trees and the pruned reference tree)
- **phangorn** (Builds the UPGMA phylogenetic trees from synteny distances (|APSS − 1|) at each subsampling depth before comparison)
- **SynTracker** (Produces the pairwise synteny scores per region and APSS tables that are converted to synteny distances and clustered into the trees being compared) — https://github.com/leylabmpi/SynTracker
- **Mash** (Source of the published reference phylogroup tree (>10,000 genomes) that is pruned and used as the comparison target)

## Examples

```
R: library(TreeDist); TreeDistance(synteny_tree, pruned_mash_tree)  # after building UPGMA trees from |APSS - 1| distances with phangorn and pruning Supplementary_Data_Fig_3c_10k_newick.nwk to the 140 analyzed genomes
```

## Evaluation signals

- Observed Information-based generalized RFD between each synteny-based tree and the pruned Mash tree falls outside all (or ≥ 99,999 of 100,000) label-shuffled distances, yielding P < 1 × 10−5.
- Recapitulation holds at every subsampling depth tested (n = 20, 40, 60, 80, 100, 200 regions per pair), even when less than 2% of the genome is sampled.
- Phylogroup clades of the 140 analyzed E. coli genomes are recovered as monophyletic groups in the synteny-based trees.
- Per-depth distances and P-values are complete and written to ecoli_info_rf_tree_distances.csv with one row per subsampling depth.
- Best performance expected at 200 regions per pairwise comparison, where the paper reports maximal average sensitivity and specificity of 98.6% and 97%.

## Limitations

- The empirical P-value is bounded by the number of shuffling iterations (100,000), so the minimum reportable P is ~1 × 10−5.
- Small numbers of regions per pairwise comparison mostly result in lower accuracy; there is a trade-off between accuracy and the number of regions sampled.
- Pairs of genomes with fewer than n available regions are excluded from downstream analysis, which can thin taxon coverage at shallow subsampling depths.

## Evidence

- [discussion] APSS values were converted to synteny distances, which are equal to the absolute value of APSS − 1: "APSS values were converted to synteny distances, which are equal to the absolute value of APSS − 1"
- [discussion] unweighted pair group method with arithmetic mean phylogenetic trees using the 'phangorn' R package: "unweighted pair group method with arithmetic mean phylogenetic trees using the ‘phangorn’ R package51"
- [discussion] calculating the Information-based generalized RFDs using the 'TreeDist' R package: "calculating the Information-based generalized RFDs using the ‘TreeDist’ R package52"
- [discussion] shuffling the labels of the synteny-based tree (100,000 iterations for each comparison): "shuffling the labels of the synteny-based tree (100,000 iterations for each comparison)"
- [discussion] pruning of the Mash-based tree so that only external nodes corresponding to the 140 E. coli genomes analyzed with SynTracker were kept: "pruning of the Mash-based tree so that only external nodes corresponding to the 140 E. coli genomes analyzed with SynTracker were kept"

---
name: label-shuffle-significance-testing
description: Use when you need an empirical P-value for similarity between two phylogenetic
  trees (or labeled structures) where no analytic null exists — e.g., testing if a
  de novo tree derived from synteny distances matches a reference classification tree.
when_to_use_negative:
- When an analytic null distribution or standard parametric test is available and
  appropriate for the statistic being compared.
- When the two trees/objects contain different sets of labels, since label shuffling
  requires matched external nodes (e.g., the Mash tree must first be pruned to the
  140 analyzed genomes).
- When the goal is measuring phylogroup classification accuracy rather than testing
  topological correspondence between a candidate and reference tree.
edam_operation: http://edamontology.org/operation_0558
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_3299
tools:
- name: TreeDist R package
  role: Computes Information-based generalized Robinson-Foulds distances between candidate
    and reference trees and supports label shuffling for empirical P-values.
  repo: https://github.com/leylabmpi/SynTracker
- name: phangorn R package
  role: Builds the UPGMA phylogenetic trees from synteny distances that are then tested
    against the reference tree.
  repo: https://github.com/leylabmpi/SynTracker
- name: SynTracker
  role: Produces the pairwise synteny scores and APSS values that are converted to
    distances for tree construction prior to significance testing.
  repo: https://github.com/leylabmpi/SynTracker
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41587-024-02276-2
    title: Strain tracking in complex microbiomes using synteny analysis reveals per-species
      modes of evolution
  source_content_hash: sha256:f8ef7952cc797104cd8292db05bd16d62eab3ec24a14468d40adc082b797f069
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leylabmpi/SynTracker
  tools:
  - SynTracker
  - TreeDist R package
  - phangorn R package
  edam_operation: http://edamontology.org/operation_0558
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# label-shuffle-significance-testing

## Summary

Empirical P-value estimation for tree or network comparisons by randomly permuting labels many times (100,000 iterations) and comparing the observed statistic (e.g., Information-based generalized Robinson-Foulds distance) to the shuffled null distribution. Used here to test whether APSS-based UPGMA trees recapitulate a published Mash phylogroup tree at P < 1e-5.

## When to use

Use when you need an empirical P-value for similarity between two phylogenetic trees (or labeled structures) where no analytic null exists — e.g., testing if a de novo tree derived from synteny distances matches a reference classification tree.

## When NOT to use

- When an analytic null distribution or standard parametric test is available and appropriate for the statistic being compared.
- When the two trees/objects contain different sets of labels, since label shuffling requires matched external nodes (e.g., the Mash tree must first be pruned to the 140 analyzed genomes).
- When the goal is measuring phylogroup classification accuracy rather than testing topological correspondence between a candidate and reference tree.

## Inputs

- Candidate phylogenetic tree (e.g., UPGMA tree built from synteny distances |APSS − 1| in Newick format)
- Reference phylogenetic tree (e.g., pruned Mash-based phylogroup tree from Supplementary_Data_Fig_3c_10k_newick.nwk)
- Number of label-shuffling iterations (100,000 per comparison)

## Outputs

- Empirical P-value per tree comparison (must be < 1e-5 to pass significance)
- Per-depth Information-based generalized Robinson-Foulds distances (written to ecoli_info_rf_tree_distances.csv)

## How to apply

Build the candidate tree from your distance matrix (here: APSS converted to synteny distances equal to |APSS − 1|, then UPGMA via the phangorn R package) and obtain the reference tree (the Mash-based E. coli phylogroup tree, pruned with Supplementary_Data_Fig_3c_10k_newick.nwk so only external nodes of the 140 analyzed genomes are kept). Compute the Information-based generalized Robinson-Foulds distance between the two trees using the TreeDist R package. Then shuffle the labels of the candidate tree 100,000 iterations per comparison, recomputing the distance each time to build a null distribution, and derive an empirical P-value as the fraction of shuffled distances at or below the observed distance. In the source task, P < 1 × 10−5 was required at every subsampling depth (n = 20, 40, 60, 80, 100, 200 regions per pair) to declare recapitulation of the published phylogroup classification.

## Related tools

- **TreeDist R package** (Computes Information-based generalized Robinson-Foulds distances between candidate and reference trees and supports label shuffling for empirical P-values.) — https://github.com/leylabmpi/SynTracker
- **phangorn R package** (Builds the UPGMA phylogenetic trees from synteny distances that are then tested against the reference tree.) — https://github.com/leylabmpi/SynTracker
- **SynTracker** (Produces the pairwise synteny scores and APSS values that are converted to distances for tree construction prior to significance testing.) — https://github.com/leylabmpi/SynTracker

## Examples

```
Rscript -e 'library(TreeDist); d <- TreeDistance(synteny_tree, pruned_mash_tree); p <- mean(shuffled_distances <= d)'
```

## Evaluation signals

- Empirical P-value must be < 1 × 10−5 at every subsampling depth (n = 20, 40, 60, 80, 100, 200 regions per pair).
- Per-depth distances and P-values are written to ecoli_info_rf_tree_distances.csv and should show consistent significance across depths.
- With less than 2% of the genome sampled, phylogroup clades should still be recapitulated in the synteny-based tree.
- The label-shuffling null distribution should be built from 100,000 iterations per comparison; fewer iterations would coarsen the minimum achievable P-value.

## Limitations

- The minimum achievable empirical P-value is bounded by the number of shuffling iterations (1/100,000 = 1e-5 here); no stronger significance can be claimed without more iterations.
- Shuffling tests topological correspondence only — it does not validate the biological correctness of either tree, only that the observed similarity is unlikely under random label permutation.
- Both trees must share the same set of external node labels, so the reference tree must be pruned to exactly the genomes analyzed with SynTracker before comparison.

## Evidence

- [discussion] Calculating the Information-based generalized RFDs using the ‘TreeDist’ R package: "calculating the Information-based generalized RFDs using the ‘TreeDist’ R package"
- [discussion] shuffling the labels of the synteny-based tree (100,000 iterations for each comparison): "shuffling the labels of the synteny-based tree (100,000 iterations for each comparison)"
- [discussion] unweighted pair group method with arithmetic mean phylogenetic trees using the ‘phangorn’ R package: "unweighted pair group method with arithmetic mean phylogenetic trees using the ‘phangorn’ R package"
- [discussion] pruning of the Mash-based tree so that only external nodes corresponding to the 140 E. coli genomes analyzed with SynTracker were kept: "pruning of the Mash-based tree so that only external nodes corresponding to the 140 E. coli genomes analyzed with SynTracker were kept"
- [discussion] APSS values were converted to synteny distances, which are equal to the absolute value of APSS − 1: "APSS values were converted to synteny distances, which are equal to the absolute value of APSS − 1"

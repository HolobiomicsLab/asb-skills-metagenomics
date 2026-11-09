---
name: mash-distance-tree-pruning
description: Use when you have a large published phylogenetic tree (e.g., a Mash distance
  tree over >10,000 genomes supplied as a newick file) and a smaller subset of analyzed
  genomes, and need a pruned reference tree to quantify agreement with trees built
  from your own distance matrix.
when_to_use_negative:
- The comparison tree is already restricted to the analyzed genomes and needs no tip
  pruning
- You need to compare against a tree built de novo from your own distances rather
  than a published reference topology
- The published tree lacks external nodes corresponding to the genomes being analyzed
  (tips cannot be matched)
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_3293
tools:
- name: Mash
  role: Source of the full distance-based phylogenetic tree (>10,000 genomes) imported
    from the literature, which is then pruned
- name: TreeDist R package
  role: Used to compute Information-based generalized Robinson-Foulds distances between
    the pruned Mash tree and the synteny-based trees, with label shuffling for empirical
    P-values
- name: phangorn R package
  role: Builds the UPGMA synteny-based trees from APSS-derived distances that are
    compared against the pruned Mash reference tree
- name: SynTracker
  role: Produces the pairwise APSS values from which synteny distances and the trees
    being validated against the pruned Mash tree are derived
  repo: https://github.com/leylabmpi/SynTracker
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41587-024-02276-2
    title: Strain tracking in complex microbiomes using synteny analysis reveals per-species
      modes of evolution
  source_content_hash: sha256:97037f6c4acdad0720b269ba6b23a8ae7d231c8b3432497171521bd12ab131ae
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
  - http://edamontology.org/topic_3293
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# mash-distance-tree-pruning

## Summary

Prune a published Mash distance-based phylogenetic tree (newick format) so that only the external nodes of the genomes under analysis are retained, yielding a reference tree for comparing against de novo synteny-based phylogenies.

## When to use

Use when you have a large published phylogenetic tree (e.g., a Mash distance tree over >10,000 genomes supplied as a newick file) and a smaller subset of analyzed genomes, and need a pruned reference tree to quantify agreement with trees built from your own distance matrix.

## When NOT to use

- The comparison tree is already restricted to the analyzed genomes and needs no tip pruning
- You need to compare against a tree built de novo from your own distances rather than a published reference topology
- The published tree lacks external nodes corresponding to the genomes being analyzed (tips cannot be matched)

## Inputs

- Published Mash distance-based phylogenetic tree in newick format (e.g., Supplementary_Data_Fig_3c_10k_newick.nwk covering >10,000 genomes)
- List of analyzed genome identifiers (e.g., 140 public E. coli genomes, ten per phylogroup) whose tips must be retained

## Outputs

- Pruned reference tree containing only the external nodes of the 140 analyzed E. coli genomes
- Reference topology used to compute Information-based generalized Robinson-Foulds distances and empirical P-values (written to e.g. ecoli_info_rf_tree_distances.csv)

## How to apply

Import the full published Mash distance-based phylogenetic tree — in this workflow the file 'Supplementary_Data_Fig_3c_10k_newick.nwk', containing >10,000 genomes imported from the literature — into R. Prune it so that only external (tip) nodes corresponding to the analyzed genomes are kept; in the E. coli case this means the 140 genomes (ten per phylogroup from the published Mash classification). The pruned tree then serves as the fixed reference topology for downstream tree-comparison statistics, e.g., computing Information-based generalized Robinson-Foulds distances with the TreeDist R package against UPGMA trees built from APSS-derived synteny distances. Significance is assessed empirically by shuffling the labels of the synteny-based tree 100,000 iterations per comparison, and clade recapitulation (e.g., of the 14 E. coli phylogroups at P < 1e-5) is judged against this pruned reference.

## Related tools

- **Mash** (Source of the full distance-based phylogenetic tree (>10,000 genomes) imported from the literature, which is then pruned)
- **TreeDist R package** (Used to compute Information-based generalized Robinson-Foulds distances between the pruned Mash tree and the synteny-based trees, with label shuffling for empirical P-values)
- **phangorn R package** (Builds the UPGMA synteny-based trees from APSS-derived distances that are compared against the pruned Mash reference tree)
- **SynTracker** (Produces the pairwise APSS values from which synteny distances and the trees being validated against the pruned Mash tree are derived) — https://github.com/leylabmpi/SynTracker

## Evaluation signals

- The pruned tree contains exactly the tips of the analyzed genome set (140 external nodes, one per analyzed E. coli genome) and no other tips
- Information-based generalized Robinson-Foulds distances to synteny-based UPGMA trees are significant at P < 1e-5 at every subsampling depth (n = 20, 40, 60, 80, 100, 200 regions per pair)
- Phylogroup clades of the published Mash classification are recapitulated even when < 2% of the genome is sampled
- Per-depth distances and empirical P-values (from 100,000 label-shuffling iterations) are recorded in ecoli_info_rf_tree_distances.csv
- Pruning retains only external nodes, preserving the internal topology connecting the retained tips

## Limitations

- The published Mash tree must be available as a newick file and its tip labels must match the identifiers of the analyzed genomes; the E. coli reference tree covered >10,000 genomes and was imported from the literature
- The pruned Mash tree represents a whole-genome k-mer-based classification; agreement of synteny-based trees with it is statistical (Information-based generalized RFD with empirical P-values), not a guarantee of identical topology
- Genome-level tree pruning does not address strain-delineation subjectivity; the article notes thresholds such as APSS 0.95 are subjective and species-dependent

## Evidence

- [discussion] Prune the Mash-based tree to the 140 analyzed E. coli genomes: "pruning of the Mash-based tree so that only external nodes corresponding to the 140 E. coli genomes analyzed with SynTracker were kept"
- [discussion] Full Mash tree imported from literature: "The full Mash distance-based phylogenetic tree, including >10,000 genomes, was imported from the literature24"
- [discussion] Newick file source: "imported from the literature24 (‘Supplementary_Data_Fig_3c_10k_newick.nwk’ file)"
- [discussion] Tree distance and significance method: "shuffling the labels of the synteny-based tree (100,000 iterations for each comparison)"

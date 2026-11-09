---
name: hierarchical-bubble-tree-organization
description: Use when a colored de Bruijn graph (directed, acyclic, nondegenerate,
  unique start/end nodes, node color sets per sample) must be decomposed into colored
  superbubbles and arranged into a parent-child tree that supports tri-tuple coordinates.
when_to_use_negative:
- Sequence similarity is insufficient (source number far less than variant number),
  so the variant detection model built on the cSupB tree is not applicable
- The graph is degenerate or has multiple start/end nodes — the method assumes a directed,
  acyclic, nondegenerate colored de Bruijn graph with unique start and end nodes
- The goal is detecting large indels, inversions, or reverse tandem duplications —
  the article states no effective solution exists for these complicated conditions
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: cSupB (VARI-cSupB / cosmo-cSupB)
  role: C++ program that runs the postorder-like traversal, identifies all cSupBs
    in one pass, and outputs the cSupB tree
  repo: https://github.com/eggleader/cSupB
- name: VARI-merge
  role: Constructs the succinct colored de Bruijn graph and color matrix that cSupB
    consumes as input
  repo: https://github.com/cosmo-team/cosmo/tree/VARI-merge
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-021-04149-w
    title: A tri-tuple coordinate system derived for fast and accurate analysis of
      the colored de Bruijn graph-based pangenomes
  source_content_hash: sha256:28d56d543b45ef26b90c522b6443124da2d30feec13b9b769ee60c83bc0e67e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eggleader/cSupB
  tools:
  - C++
  - VARI-merge
  - cSupB (VARI-cSupB / cosmo-cSupB)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# hierarchical-bubble-tree-organization

## Summary

Reconstructs the cSupB decomposition algorithm from the colored de Bruijn graph: a single postorder-like traversal identifies colored superbubbles (cSupBs) via color-intersection matching, then organizes them into a hierarchical tree (the basis of the paper's tri-tuple coordinate system).

## When to use

Use when a colored de Bruijn graph (directed, acyclic, nondegenerate, unique start/end nodes, node color sets per sample) must be decomposed into colored superbubbles and arranged into a parent-child tree that supports tri-tuple coordinates.

## When NOT to use

- Sequence similarity is insufficient (source number far less than variant number), so the variant detection model built on the cSupB tree is not applicable
- The graph is degenerate or has multiple start/end nodes — the method assumes a directed, acyclic, nondegenerate colored de Bruijn graph with unique start and end nodes
- The goal is detecting large indels, inversions, or reverse tandem duplications — the article states no effective solution exists for these complicated conditions

## Inputs

- Colored de Bruijn graph (.dbg file from VARI-merge/Cosmo construction)
- Color matrix mapping graph nodes to sample color sets (.colors.sd_vector sdsl bitvector)
- k value chosen as the maximum longest repeat segment length across input sequences

## Outputs

- List of identified colored superbubbles, each with source node, sink node, cSupB color set, and ordering/offset
- cSupB tree (hierarchical structure with levels; root = cSupB containing all samples), written to csupb_tree.txt
- Basis for the tri-tuple coordinate system combining offset value, topological structure and sample information

## How to apply

Load the colored de Bruijn graph (e.g., the Fig. 1 toy graph of three haplotype samples, k=3) and label every node with one of four states (unvisited, half-visited, to-be-visited, fully visited); run the postorder-like traversal only once, visiting a node when all its parents have been visited. During traversal, apply the cSupB matching principle — at least two adjacent outgoing edges' colors of a source node (outdegree > 1) intersect with the sink node's color and vice versa (sink indegree > 1) — using the to-be-visited source queue searched reversely, and compute each node's offset as the maximum offset among incoming parents plus one, recording source, sink, cSupB color, and order for every cSupB. Reorganize the cSupBs into a tree using the inclusion test (order(G1) < order(G2), C1 strictly a subset of C2 with C1 not the all-samples set, and no intervening cSupB C3 with C1 strictly inside C3); assign the root (containing all samples) level 1 and each child a deeper level, consistent with the corollary that a child cSupB cannot be obtained before its parent. Validation on the toy graph requires reproducing exactly the five reference cSupBs bub1<TAT,ACC,111>, bub2<GGG,GTA,110>, bub3<CAC,GTA,011>, bub4<TCA,GGA,...> through bub5 and confirming the result written to csupb_tree.txt is a tree rooted at the all-samples cSupB.

## Related tools

- **cSupB (VARI-cSupB / cosmo-cSupB)** (C++ program that runs the postorder-like traversal, identifies all cSupBs in one pass, and outputs the cSupB tree) — https://github.com/eggleader/cSupB
- **VARI-merge** (Constructs the succinct colored de Bruijn graph and color matrix that cSupB consumes as input) — https://github.com/cosmo-team/cosmo/tree/VARI-merge

## Examples

```
cosmo-cSupB primates_mtDNAs_kmc_list.dbg primates_mtDNAs_kmc_list.colors.sd_vector >log.txt
```

## Evaluation signals

- Toy-graph validation: output must match the reference decomposition of exactly five cSupBs (bub1<TAT,ACC,111>, bub2<GGG,GTA,110>, bub3<CAC,GTA,011>, bub4<TCA,GGA,...>, bub5)
- Structural invariant: the identified cSupB relations must form a tree, not a DAG or forest (per Additional file 1: Corollary 7)
- The root cSupB must contain the color set of all samples and be assigned hierarchical level 1
- Every node's offset must equal the maximum offset among its incoming parents plus one
- Ordering invariant: no child cSupB is identified before its parent cSupB (Corollary 6)

## Limitations

- If the traversal finishes with unvisited nodes remaining, cycles exist in the graph and a cycle identification/restart procedure based on half-visited nodes and offset values is required
- Degeneracy cannot be handled efficiently, and gene elements shorter than k may map to multiple nodes, making analysis complicated and ambiguous
- The choice of k value impacts the accuracy of the algorithm

## Evidence

- [methods] At least two adjacent outgoing edges’ colors of the source node intersect with the color of the sink node: "At least two adjacent outgoing edges’ colors of the source node intersect with the color of the sink node"
- [methods] Hierarchical tree assignment: "we adopt a hierarchical tree structure by exploiting the relations of cSupBs."
- [methods] Child-parent ordering corollary: "a child cSupB cannot be obtained before its parent cSupB"
- [abstract] Tri-tuple coordinate system: "we propose a tri-tuple coordinate system that combines an offset value, topological structure and sample information"
- [readme] CLI invocation: "cosmo-cSupB primates_mtDNAs_kmc_list.dbg primates_mtDNAs_kmc_list.colors.sd_vector >log.txt"

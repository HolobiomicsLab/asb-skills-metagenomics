---
name: de-bruijn-graph-postorder-traversal
description: Use when you must decompose a colored (multi-sample) de Bruijn graph
  into sample-specific superbubbles, assign offset values per node, and build the
  cSupB tree used for graph coordinates and variant detection.
when_to_use_negative:
- Sequences with insufficient similarity (source number far less than variant number)
  — the variant detection model built on the traversal is not applicable
- Highly repetitive sequences, especially tandem repeats — the cutting method fragments
  the sequence, increasing computational difficulty and error
- Cases requiring large indels, inversions, or reverse tandem duplications — the method
  only addresses small indels (< 50 bp) and cannot handle degeneracy efficiently
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0080
tools:
- name: cSupB (VARI-cSupB)
  role: C++ program that performs the postorder-like traversal, cSupB matching, offset
    computation, and tree reorganization
  repo: https://github.com/eggleader/cSupB
- name: VARI-merge (Cosmo)
  role: Builds the succinct colored de Bruijn graph and color matrix that the traversal
    operates on; installation is shared (add Makefile and cosmo-cSupB.cpp to VARI-merge's
    main directory)
  repo: https://github.com/cosmo-team/cosmo/tree/VARI-merge
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-021-04149-w
    title: A tri-tuple coordinate system derived for fast and accurate analysis of
      the colored de Bruijn graph-based pangenomes
  source_content_hash: sha256:e2495f4fa7ea28c0adb19910d81fac7c05e07242a5de57597b7c9efdbab9ff72
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eggleader/cSupB
  tools:
  - C++
  - VARI-merge (Cosmo)
  - cSupB (VARI-cSupB)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# de-bruijn-graph-postorder-traversal

## Summary

A single postorder-like traversal of a colored de Bruijn graph (as implemented in VARI-cSupB/C++) that uses four visiting states and a color-intersection matching principle to identify all colored superbubbles (cSupBs) in one pass, then reorganizes them into the hierarchical cSupB tree underlying the tri-tuple coordinate system.

## When to use

Use when you must decompose a colored (multi-sample) de Bruijn graph into sample-specific superbubbles, assign offset values per node, and build the cSupB tree used for graph coordinates and variant detection.

## When NOT to use

- Sequences with insufficient similarity (source number far less than variant number) — the variant detection model built on the traversal is not applicable
- Highly repetitive sequences, especially tandem repeats — the cutting method fragments the sequence, increasing computational difficulty and error
- Cases requiring large indels, inversions, or reverse tandem duplications — the method only addresses small indels (< 50 bp) and cannot handle degeneracy efficiently

## Inputs

- Colored de Bruijn graph file (.dbg) constructed by VARI-merge (e.g. primates_mtDNAs_kmc_list.dbg)
- Color matrix / sample-colors vector (.colors.sd_vector, e.g. primates_mtDNAs_kmc_list.colors.sd_vector)
- k-mer size k (chosen as the maximum longest repeat segment length across the input sequences; e.g. k=3 in the Fig. 1 toy graph, k=22 for VARI-merge on acyclic test graphs)
- Sample sequences with ACGT anchor segments of length k added to front and end

## Outputs

- List of colored superbubbles (cSupBs), each recording source node, sink node, cSupB color, and ordering (e.g. bub1<TAT,ACC,111> ... bub5 on the Fig. 1 toy graph)
- Offset value for every node (max parent offset + 1)
- Hierarchical cSupB tree with assigned levels (root level 1; written to csupb_tree.txt)
- Log output of the traversal and matching process (>log.txt)

## How to apply

Load the colored de Bruijn graph built by VARI-merge (directed, acyclic, nondegenerate, with unique start and end nodes) and label every node with one of four visiting states: unvisited, half-visited, to-be-visited, fully visited; a node is visited only when all its parent nodes have been visited. During this single postorder-like traversal, apply the cSupB matching principle — at least two adjacent outgoing edges' colors of a source node intersect with the sink node's color, and vice versa — pairing source nodes (outdegree > 1) with sink nodes (indegree > 1) using the to-be-visited source queue searched in reverse. For each matched cSupB record source node, sink node, cSupB color, and ordering, and compute each node's offset value as the maximum offset among its incoming parents plus one. If traversal finishes with unvisited nodes remaining, cycles must exist: identify the half-visited node via the offset-based cycle method, mark it fully visited, and restart the traversal. Finally, reorganize the cSupBs into a tree with the inclusion test (order(G1) < order(G2), C1 a strict subset of C2 with C1 not the all-samples set, and no intervening cSupB C3 strictly containing C1), assign the root (containing all samples) level 1 and each child a deeper level; a child cSupB cannot be obtained before its parent, and the cSupB structure must be a tree.

## Related tools

- **cSupB (VARI-cSupB)** (C++ program that performs the postorder-like traversal, cSupB matching, offset computation, and tree reorganization) — https://github.com/eggleader/cSupB
- **VARI-merge (Cosmo)** (Builds the succinct colored de Bruijn graph and color matrix that the traversal operates on; installation is shared (add Makefile and cosmo-cSupB.cpp to VARI-merge's main directory)) — https://github.com/cosmo-team/cosmo/tree/VARI-merge

## Examples

```
cosmo-cSupB primates_mtDNAs_kmc_list.dbg primates_mtDNAs_kmc_list.colors.sd_vector >log.txt
```

## Evaluation signals

- On the Fig. 1 toy graph (three haplotype samples, k=3), the module output must reproduce the reference decomposition of exactly five cSupBs (bub1<TAT,ACC,111>, bub2<GGG,GTA,110>, bub3<CAC,GTA,011>, bub4<TCA,GGA,..., bub5) written to csupb_tree.txt
- The reported cSupB structure must be a tree whose root cSupB contains all samples
- No child cSupB may be reported before its parent cSupB in the traversal output
- Every cSupB pairing must satisfy the matching principle: at least two adjacent outgoing edges' colors of the source intersect with the sink node's color and vice versa
- Traversal termination invariants: each node visited only after all parents visited, with offset equal to max(incoming parents' offsets) + 1; leftover unvisited nodes indicate unhandled cycles

## Limitations

- If the traversal finishes but unvisited nodes remain, the graph contains cycles; handling requires marking the half-visited node fully visited, restarting the traversal, and, via the cutting point selection model, segmenting the sequence and reconstructing the graph
- Variant detection precision decreases as sample similarity decreases and variant types become complicated; the model is limited to highly similar samples with simple variant types and only addresses small indels (< 50 bp)
- The choice of k value impacts accuracy, and gene elements shorter than k may map to multiple nodes, making analysis complicated and ambiguous; degeneracy cannot be handled efficiently
- Cosmo/VARI-merge only supports DSK files with k <= 64, and its cosmo-assemble step is not implemented; graph construction tools like VARI-merge lack mapping functionality, limiting data selection

## Evidence

- [methods] At least two adjacent outgoing edges' colors of the source node intersect with the color of the sink node: [evidence span withheld — anchor status: not_found]
- [methods] if the traversal is finished but nodes still exist that are unvisited, we call the traversal finished in advance and there must be cycles in graph: "if the traversal is finished but nodes still exist that are unvisited, we call the traversal finished in advance and there must be cycles in graph"
- [methods] we need to identify this node, mark it as fully visited, and restart the graph traversal: "we need to identify this node, mark it as fully visited, and restart the graph traversal"
- [methods] a child cSupB cannot be obtained before its parent cSupB (Additional file 1: Corollary 6): "a child cSupB cannot be obtained before its parent cSupB (Additional file 1: Corollary 6)"
- [methods] the obtained colored de Bruijn graph has the following characteristics: directed, acyclic, nondegeneracy, and unique start and end nodes: "the obtained colored de Bruijn graph has the following characteristics: directed, acyclic, nondegeneracy, and unique start and end nodes"
- [readme] VARI-cSupB is an application of the succinct colored de Bruijn graph constructed by  [VARI-merge]: "VARI-cSupB is an application of the succinct colored de Bruijn graph constructed by  [VARI-merge]"

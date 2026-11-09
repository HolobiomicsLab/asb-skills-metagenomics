---
name: colored-superbubble-matching
description: Use when a VARI-merge constructed colored de Bruijn graph (.dbg + .colors.sd_vector)
  must be decomposed into sample-specific colored superbubbles and organized into
  a tree to support a tri-tuple coordinate system or nested variant detection.
when_to_use_negative:
- The input is not a colored directed graph usable as directed, acyclic, nondegenerate
  with unique start and end nodes (e.g. degeneracy must be resolved elsewhere; the
  paper states 'we cannot handle degeneracy efficiently')
- Sample similarity is insufficient (source number far less than variant number) —
  the variant detection model built on the cSupB tree is not applicable
- You need to detect large indels, inversions, or reverse tandem duplications — the
  downstream variant detection only addresses small indels (< 50 bp) in highly similar
  samples
edam_topics:
- http://edamontology.org/topic_3678
tools:
- name: cSupB
  role: C++ implementation of the cSupB traversal, matching, and tree reorganization
    (cosmo-cSupB binary)
  repo: https://github.com/eggleader/cSupB
- name: VARI-merge
  role: Builds the succinct colored de Bruijn graph and color matrix that are the
    input to the cSupB matching traversal
  repo: https://github.com/cosmo-team/cosmo/tree/VARI-merge
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-021-04149-w
    title: A tri-tuple coordinate system derived for fast and accurate analysis of
      the colored de Bruijn graph-based pangenomes
  source_content_hash: sha256:1f37fe0671404d01700cf7cf56986f0491480e3155f269f8743b04a5a024c18e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eggleader/cSupB
  tools:
  - C++
  - VARI-merge
  - cSupB
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3678
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# colored-superbubble-matching

## Summary

Implements the cSupB decomposition of a colored de Bruijn graph: a single postorder-like traversal pairs source/sink nodes by the colored-edge intersection matching principle, then reorganizes the resulting colored superbubbles into the hierarchical cSupB tree that underlies the tri-tuple coordinate system (offset value, topological structure, sample information).

## When to use

Use when a VARI-merge constructed colored de Bruijn graph (.dbg + .colors.sd_vector) must be decomposed into sample-specific colored superbubbles and organized into a tree to support a tri-tuple coordinate system or nested variant detection.

## When NOT to use

- The input is not a colored directed graph usable as directed, acyclic, nondegenerate with unique start and end nodes (e.g. degeneracy must be resolved elsewhere; the paper states 'we cannot handle degeneracy efficiently')
- Sample similarity is insufficient (source number far less than variant number) — the variant detection model built on the cSupB tree is not applicable
- You need to detect large indels, inversions, or reverse tandem duplications — the downstream variant detection only addresses small indels (< 50 bp) in highly similar samples

## Inputs

- Colored de Bruijn graph constructed by VARI-merge (e.g. a .dbg succinct graph file)
- Color matrix from VARI-merge (.colors.sd_vector) mapping graph elements to sample colors
- Toy/test colored de Bruijn graph with explicit node colors and a defined k value (e.g. Fig. 1 graph, k=3, three haplotype samples)

## Outputs

- List of identified cSupBs, each with source node, sink node, cSupB color set and ordering (offset values computed as max incoming parent offset + 1)
- Hierarchical cSupB tree with assigned levels (root contains all samples), written to csupb_tree.txt

## How to apply

Run a single postorder-like traversal of the colored de Bruijn graph (directed, acyclic, nondegenerate, with unique start and end nodes), labeling each node with one of four visiting states (unvisited, half-visited, to-be-visited, fully visited) and visiting a node only when all its parents have been visited. During traversal apply the cSupB matching principle — at least two adjacent outgoing edges' colors of a source node intersect with the sink node's color and vice versa — pairing source nodes with outdegree > 1 and sink nodes with indegree > 1, searching the to-be-visited source queue in reverse. Record for every cSupB the source node, sink node, cSupB color and order, and compute each node's offset value as the maximum offset among its incoming parents plus one; if traversal finishes with unvisited nodes remaining, cycles exist and must be handled by marking the half-visited node fully visited and restarting. Reorganize cSupBs into a tree via the inclusion test (order(G1) < order(G2), C1 a strict subset of C2 with C1 not the all-samples set, and no intervening cSupB whose color set strictly contains C1), assigning the root — the cSupB containing all samples — level 1 and each child a deeper level. On the Fig. 1 toy graph (three haplotype samples, k=3), the implementation is validated by exactly reproducing the five reference cSupBs bub1<TAT,ACC,111>, bub2<GGG,GTA,110>, bub3<CAC,GTA,011>, bub4<TCA,GGA,... through bub5.

## Related tools

- **cSupB** (C++ implementation of the cSupB traversal, matching, and tree reorganization (cosmo-cSupB binary)) — https://github.com/eggleader/cSupB
- **VARI-merge** (Builds the succinct colored de Bruijn graph and color matrix that are the input to the cSupB matching traversal) — https://github.com/cosmo-team/cosmo/tree/VARI-merge

## Examples

```
cosmo-cSupB primates_mtDNAs_kmc_list.dbg primates_mtDNAs_kmc_list.colors.sd_vector >log.txt
```

## Evaluation signals

- On the Fig. 1 toy graph (k=3, three samples), the module must output exactly the five reference cSupBs: bub1<TAT,ACC,111>, bub2<GGG,GTA,110>, bub3<CAC,GTA,011>, bub4<TCA,GGA,... through bub5
- The reorganized cSupB structure must be a tree whose root cSupB contains all samples (Corollary 7), with each child at a deeper hierarchical level
- Every child cSupB must be obtained after its parent cSupB (Corollary 6) — parent identification must precede child in traversal order
- Every node's offset value must equal the maximum offset among its incoming parents plus one
- If the traversal finishes with unvisited nodes remaining, this indicates cycles in the graph — the cycle-identification and restart path must trigger rather than silently finishing

## Limitations

- If the traversal is finished but nodes remain unvisited, cycles exist and traversal must be restarted after marking the half-visited node fully visited (cycle handling adds complexity)
- The graph must be directed, acyclic, nondegenerate, with unique start and end nodes; degeneracy cannot be handled efficiently
- Downstream variant detection built on the cSupB tree only handles small indels (< 50 bp) and SNPs in highly similar samples; large indels, inversions, and reverse tandem duplications are unsupported
- The choice of k value impacts variant detection accuracy, and gene elements shorter than k may map to multiple nodes

## Evidence

- [methods] At least two adjacent outgoing edges' colors of the source node intersect with the color of the sink node: "At least two adjacent outgoing edges’ colors of the source node intersect with the color of the sink node"
- [methods] we adopt a hierarchical tree structure by exploiting the relations of cSupBs. Furthermore, after confirming the root cSupB (containing all samples), we assigned each cSupB a hierarchical level: "we adopt a hierarchical tree structure by exploiting the relations of cSupBs. Furthermore, after confirming the root cSupB (containing all samples), we assigned each cSupB a hierarchical level"
- [abstract] the motif cSupB and the cSupB method can be extended to any colored directed acyclic graph: "the motif cSupB and the cSupB method can be extended to any colored directed acyclic graph"
- [readme] VARI-cSupB is an application of the succinct colored de Bruijn graph constructed by [VARI-merge]: "VARI-cSupB is an application of the succinct colored de Bruijn graph constructed by  [VARI-merge]"

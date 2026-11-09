---
name: toy-graph-validation-testing
description: Use when implementing or porting the cSupB traversal/tree algorithm and
  you need a small, fully known colored DAG (the Fig. 1 toy graph, k=3) to verify
  correctness of superbubble detection, offsets, and tree levels.
when_to_use_negative:
- The input contains highly repetitive sequences (e.g. tandem repeats) — the cutting
  method fragments the sequence and the toy-graph validation does not cover this regime
- You need variant calling on low-similarity samples or complex variant types (large
  indels, inversions, reverse tandem duplications) — the cSupB variant detection this
  validates only addresses small indels (<50 bp) for highly similar samples
- The graph is degenerate or lacks the assumed properties (directed, acyclic, nondegenerate,
  unique start and end nodes) produced by VARI-merge graph construction
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0196
tools:
- name: C++ (cSupB program)
  role: C++ program implementing the cSupB postorder-like traversal, cSupB matching,
    offset computation and tree reorganization that is validated on the toy graph
  repo: https://github.com/eggleader/cSupB
- name: VARI-merge
  role: Constructs the succinct colored de Bruijn graph (.dbg) and color matrix (.colors.sd_vector)
    that the validated cSupB module consumes
  repo: https://github.com/cosmo-team/cosmo/tree/VARI-merge
- name: VARI-cSupB
  role: Application of the VARI-merge succinct colored de Bruijn graph; its cosmo-cSupB
    binary is the executable under validation
  repo: https://github.com/eggleader/cSupB
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-021-04149-w
    title: A tri-tuple coordinate system derived for fast and accurate analysis of
      the colored de Bruijn graph-based pangenomes
  source_content_hash: sha256:bf9e541c15b7a53cd084b91bb5279b057697c4556c18c6d49f9da73492c56b2f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eggleader/cSupB
  tools:
  - C++
  - C++ (cSupB program)
  - VARI-cSupB
  - VARI-merge
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# Toy-graph validation testing for cSupB tree reconstruction

## Summary

A validation procedure that reconstructs the cSupB colored-superbubble identification and tree-building algorithm on the Fig. 1 toy colored de Bruijn graph (three haplotype samples, k=3) and checks the output against the paper's reference decomposition (bub1–bub5) before trusting it on real pangenome data.

## When to use

Use when implementing or porting the cSupB traversal/tree algorithm and you need a small, fully known colored DAG (the Fig. 1 toy graph, k=3) to verify correctness of superbubble detection, offsets, and tree levels.

## When NOT to use

- The input contains highly repetitive sequences (e.g. tandem repeats) — the cutting method fragments the sequence and the toy-graph validation does not cover this regime
- You need variant calling on low-similarity samples or complex variant types (large indels, inversions, reverse tandem duplications) — the cSupB variant detection this validates only addresses small indels (<50 bp) for highly similar samples
- The graph is degenerate or lacks the assumed properties (directed, acyclic, nondegenerate, unique start and end nodes) produced by VARI-merge graph construction

## Inputs

- Fig. 1 toy colored de Bruijn graph (three haplotype samples, k=3) with node color sets
- C++ implementation of the cSupB traversal (cosmo-cSupB.cpp added to VARI-merge's main directory)
- VARI-merge .dbg graph file and .colors.sd_vector color matrix (for the full pipeline this validation gates)

## Outputs

- csupb_tree.txt containing the reference cSupB decomposition (bub1–bub5 with source, sink, color triple such as bub1<TAT,ACC,111>) and hierarchical tree levels
- Per-node offset values and cSupB orderings emitted during the single postorder-like traversal
- log.txt from a full cosmo-cSupB run when the validated module is applied to real data

## How to apply

Load the Fig. 1 toy colored de Bruijn graph (three haplotype samples, k=3) with its node colors, then label each node with one of four visiting states (unvisited, half-visited, to-be-visited, fully visited) and run the postorder-like traversal exactly once, visiting a node only when all its parent nodes have been visited. During traversal, apply the cSupB matching principle — at least two adjacent outgoing edges' colors of a source node intersect with the sink node's color and vice versa — pairing source/sink nodes with outdegree/indegree greater than one using the to-be-visited source queue searched reversely, and compute each node's offset value as the maximum offset among its incoming parents plus one. Record the source node, sink node, cSupB color and ordering for every cSupB, then reorganize the cSupBs into a hierarchical tree using the inclusion test (order(G1) < order(G2), C1 a strict subset of C2 with C1 not the all-samples set, and no intervening cSupB C3 strictly containing C1), assigning the root level 1 and each child a deeper level. Validation passes only if the output matches the reference decomposition — the five cSupBs bub1<TAT,ACC,111>, bub2<GGG,GTA,110>, bub3<CAC,GTA,011>, bub4<TCA,GGA,...> through bub5 — and the structure is confirmed to be a tree whose root contains all samples; write the result to csupb_tree.txt. Rationale: a child cSupB cannot be obtained before its parent, so a correct single-pass traversal on the toy graph implies correct ordering and subordination handling on larger graphs.

## Related tools

- **C++ (cSupB program)** (C++ program implementing the cSupB postorder-like traversal, cSupB matching, offset computation and tree reorganization that is validated on the toy graph) — https://github.com/eggleader/cSupB
- **VARI-merge** (Constructs the succinct colored de Bruijn graph (.dbg) and color matrix (.colors.sd_vector) that the validated cSupB module consumes) — https://github.com/cosmo-team/cosmo/tree/VARI-merge
- **VARI-cSupB** (Application of the VARI-merge succinct colored de Bruijn graph; its cosmo-cSupB binary is the executable under validation) — https://github.com/eggleader/cSupB

## Examples

```
cosmo-cSupB primates_mtDNAs_kmc_list.dbg primates_mtDNAs_kmc_list.colors.sd_vector >log.txt
```

## Evaluation signals

- Module output on the toy graph matches the reference decomposition exactly: five cSupBs bub1<TAT,ACC,111>, bub2<GGG,GTA,110>, bub3<CAC,GTA,011>, bub4<TCA,GGA,...> through bub5
- The reported cSupB structure is a tree (Corollary 7), with the root cSupB containing all samples and each child assigned a strictly deeper hierarchical level
- Every node's offset value equals the maximum offset among its incoming parents plus one
- Traversal finishes without unvisited leftover nodes; if it finishes in advance with unvisited nodes remaining, cycles must exist in the graph and the test graph is invalid
- csupb_tree.txt is produced and its ordering satisfies the inclusion test conditions (order(G1) < order(G2), C1 ⊂ C2 with C1 not the all-samples set, no intervening C3 strictly containing C1)

## Limitations

- The graph must be directed, acyclic, nondegenerate, with unique start and end nodes; degeneracy cannot be handled efficiently
- If the single traversal finishes with unvisited nodes remaining, cycles exist and a cycle-identification/restart procedure (mark the half-visited node fully visited and restart) is required
- Choice of the k value impacts accuracy, and gene elements shorter than k may map to multiple nodes, making analysis ambiguous

## Evidence

- [methods] cSupB matching principle: "At least two adjacent outgoing edges’ colors of the source node intersect with the color of the sink node"
- [methods] cycle signal on the toy graph: "if the traversal is finished but nodes still exist that are unvisited, we call the traversal finished in advance and there must be cycles in graph"
- [methods] tree invariant: "the cSupB structure must be a tree (Additional file 1: Corollary 7)"
- [abstract] implementation language: "We have developed a C + + program for implementing our method"
- [readme] example invocation: "cosmo-cSupB primates_mtDNAs_kmc_list.dbg primates_mtDNAs_kmc_list.colors.sd_vector >log.txt"

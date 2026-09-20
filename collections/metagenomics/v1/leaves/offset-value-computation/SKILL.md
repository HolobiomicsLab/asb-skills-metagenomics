---
name: offset-value-computation
description: Use when traversing a colored de Bruijn graph built from multiple sample
  haplotypes (e.g., 5000–20,000 simulated mitochondrial genomes) to assign positional
  coordinates to nodes and colored superbubbles (cSupBs), or to detect cycles during
  the traversal.
when_to_use_negative:
- Sequences with insufficient similarity (source number far less than variant number),
  where the variant detection model built on this coordinate system is not applicable
- Graphs with degeneracy or gene elements shorter than k, which map to multiple nodes
  and complicate offset/coordinate analysis
- Detection of large indels, inversions, or reverse tandem duplications, which the
  offset-based variant method does not address
tools:
- name: C++ (cosmo-cSupB)
  role: C++ program implementing the cSupB traversal and offset-value computation
    on the VARI-merge graph
  repo: https://github.com/eggleader/cSupB
- name: VARI-merge
  role: Constructs the succinct colored de Bruijn graph and color matrix on which
    offset values are computed (k=22 used for VARI-merge to obtain an acyclic graph)
  repo: https://github.com/cosmo-team/cosmo/tree/VARI-merge
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-021-04149-w
    title: A tri-tuple coordinate system derived for fast and accurate analysis of
      the colored de Bruijn graph-based pangenomes
  source_content_hash: sha256:369a7a021b7648264937bc96f693865fe67e51f07a47555709b28f8f8eb1b147
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eggleader/cSupB
  tools:
  - C++
  - C++ (cosmo-cSupB)
  - VARI-merge
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# offset-value-computation

## Summary

Computation of offset values for each node of a colored de Bruijn graph (a VARI-merge-constructed graph) during the postorder-like cSupB traversal, where a node's offset is the maximum offset among its incoming parents plus one. The offset value is one component of the paper's tri-tuple coordinate system (offset value, topological structure, sample information) and also powers the cycle-identification step.

## When to use

Use when traversing a colored de Bruijn graph built from multiple sample haplotypes (e.g., 5000–20,000 simulated mitochondrial genomes) to assign positional coordinates to nodes and colored superbubbles (cSupBs), or to detect cycles during the traversal.

## When NOT to use

- Sequences with insufficient similarity (source number far less than variant number), where the variant detection model built on this coordinate system is not applicable
- Graphs with degeneracy or gene elements shorter than k, which map to multiple nodes and complicate offset/coordinate analysis
- Detection of large indels, inversions, or reverse tandem duplications, which the offset-based variant method does not address

## Inputs

- Colored de Bruijn graph built by VARI-merge (e.g., a .dbg file from cosmo-build with variable-order mode)
- Color matrix from VARI-merge (e.g., a .colors.sd_vector succinct sample-color file)
- Sequence collection with k chosen as the maximum longest repeat segment length across input sequences

## Outputs

- Offset value per graph node (max incoming-parent offset + 1)
- Per-cSupB records: source node, sink node, cSupB color, ordering and offset components of the tri-tuple coordinate
- csupb_tree.txt containing the hierarchical cSupB tree with assigned hierarchical levels
- Cycle detection flags / cutting points when traversal finishes in advance

## How to apply

During the single postorder-like traversal of the colored de Bruijn graph — in which each node carries one of four visiting states (unvisited, half-visited, to-be-visited, fully visited) and is visited only when all its parents have been visited — compute the offset value of every node as the maximum offset among its incoming parents plus one. Record, alongside the offset, the source node, sink node, cSupB color and ordering for every identified cSupB so the offsets feed the tri-tuple coordinate system and the subsequent hierarchical cSupB tree. If the traversal finishes but unvisited nodes remain, the offset-based cycle identification flags cycles: locate the half-visited node, mark it fully visited, restart the traversal, and select a cutting point in the identified cycle interval to split the sequence and reconstruct the graph. Validate the output by checking that cSupBs on a toy graph (Fig. 1, three haplotypes, k=3) match the reference decomposition bub1–bub5 and write the tree to csupb_tree.txt.

## Related tools

- **C++ (cosmo-cSupB)** (C++ program implementing the cSupB traversal and offset-value computation on the VARI-merge graph) — https://github.com/eggleader/cSupB
- **VARI-merge** (Constructs the succinct colored de Bruijn graph and color matrix on which offset values are computed (k=22 used for VARI-merge to obtain an acyclic graph)) — https://github.com/cosmo-team/cosmo/tree/VARI-merge

## Examples

```
cosmo-cSupB primates_mtDNAs_kmc_list.dbg primates_mtDNAs_kmc_list.colors.sd_vector >log.txt
```

## Evaluation signals

- On the Fig. 1 toy graph (three haplotypes, k=3), the module reproduces exactly the five reference cSupBs bub1<TAT,ACC,111>, bub2<GGG,GTA,110>, bub3<CAC,GTA,011>, bub4<TCA,GGA,... through bub5, written to csupb_tree.txt
- Every node's offset equals (maximum offset among its incoming parents) + 1; verify by manual recomputation on the toy graph
- The resulting cSupB structure must form a tree whose root cSupB contains all samples; each child sits at a deeper hierarchical level than its parent (inclusion test: order(G1) < order(G2), C1 ⊂ C2)
- If the traversal ends with unvisited nodes remaining, the run must report cycles rather than silently emit coordinates
- A child cSupB is never emitted before its parent cSupB during traversal

## Limitations

- If the traversal is finished but unvisited nodes still exist, cycles must exist and the graph must be cut and reconstructed (mark the half-visited node fully visited and restart traversal)
- Degeneracy cannot be handled efficiently; k-mer/segment length choice impacts accuracy, and elements shorter than k may map to multiple nodes
- The offset/coordinate-based variant detection only handles small indels (< 50 bp) for highly similar samples; precision decreases as sample similarity drops and variant types become complicated
- Requires the graph to be directed, acyclic, nondegenerate, with unique start and end nodes

## Evidence

- [methods] we propose a cycle identification method based on the offset value: "we propose a cycle identification method based on the offset value"
- [abstract] tri-tuple coordinate system combining offset value, topological structure and sample information: "we propose a tri-tuple coordinate system that combines an offset value, topological structure and sample information"
- [methods] postorder-like traversal with four visiting states: "At least two adjacent outgoing edges’ colors of the source node intersect with the color of the sink node"
- [methods] cycle handling: mark half-visited node fully visited and restart: "we need to identify this node, mark it as fully visited, and restart the graph traversal"
- [readme] README CLI invocation of cosmo-cSupB: "cosmo-cSupB primates_mtDNAs_kmc_list.dbg primates_mtDNAs_kmc_list.colors.sd_vector >log.txt"

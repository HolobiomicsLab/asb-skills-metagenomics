---
name: colored-superbubble-decomposition
description: Use when a colored de Bruijn graph built from highly similar sequences
  (e.g., mitochondrial pangenomes) must be decomposed into nested, population-specific
  superbubble structures to enable coordinate assignment or variant detection.
when_to_use_negative:
- The input samples are not highly similar (source number far less than variant number)
  — the variant detection model built on cSupB is not applicable when sequence similarity
  is insufficient
- Sequences are highly repetitive, especially tandem repeats — the cutting-based reconstruction
  fragments sequences excessively, increasing computational difficulty and error
- The graph contains degeneracy or the gene elements are shorter than k — degeneracy
  cannot be handled efficiently and short elements map to multiple ambiguous nodes
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_0622
tools:
- name: cSupB
  role: C++ program (cosmo-cSupB) that performs the postorder-like traversal, cSupB
    matching, cycle handling, and hierarchical tree reorganization on the VARI-merge
    graph
  repo: https://github.com/eggleader/cSupB
- name: VARI-merge
  role: Constructs the succinct colored de Bruijn graph (.dbg and .colors.sd_vector)
    that is decomposed into cSupBs
  repo: https://github.com/cosmo-team/cosmo/tree/VARI-merge
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_004
  source_papers:
  - doi: 10.1186/s12859-021-04149-w
    title: A tri-tuple coordinate system derived for fast and accurate analysis of
      the colored de Bruijn graph-based pangenomes
  source_content_hash: sha256:e934f600eda979f297ac3e53bac23c3886ec8def439c8f52edb2e2405b2beb71
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eggleader/cSupB
  tools:
  - C++
  - Clustal Omega
  - MAFFT-GINSI
  - MUSCLE
  - Mugsy
  - VARI-merge
  - cSupB
  edam_operation: http://edamontology.org/operation_0524
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# colored-superbubble-decomposition

## Summary

Decomposition of a colored de Bruijn graph into colored superbubbles (cSupBs) via a postorder-like traversal, reorganized into a hierarchical cSupB tree. This overcomes graph complexity and organizes species- or population-specific haplotype sequences into a tri-tuple coordinate system (position, bubid, basecolor).

## When to use

Use when a colored de Bruijn graph built from highly similar sequences (e.g., mitochondrial pangenomes) must be decomposed into nested, population-specific superbubble structures to enable coordinate assignment or variant detection.

## When NOT to use

- The input samples are not highly similar (source number far less than variant number) — the variant detection model built on cSupB is not applicable when sequence similarity is insufficient
- Sequences are highly repetitive, especially tandem repeats — the cutting-based reconstruction fragments sequences excessively, increasing computational difficulty and error
- The graph contains degeneracy or the gene elements are shorter than k — degeneracy cannot be handled efficiently and short elements map to multiple ambiguous nodes

## Inputs

- Colored de Bruijn graph (.dbg) and color matrix (.colors.sd_vector) produced by VARI-merge from preprocessed haplotype FASTA sequences (degenerate bases replaced, random ACGT k-length anchors appended, k=22)
- DSK k-mer count files with k <= 64 consumed by Cosmo/VARI-merge graph construction

## Outputs

- Hierarchical cSupB tree with each colored superbubble assigned a hierarchical level (root cSupB contains all samples)
- Tri-tuple coordinates (offset value, topological structure/bubid, sample color) for graph positions
- cSupB decomposition files (CST/CSDI/NNI/OVI) emitted by the cSupB C++ program, plus log output

## How to apply

First construct the colored de Bruijn graph with VARI-merge (k=22 chosen to exceed the maximum longest repeat segment length K so the graph is acyclic) after replacing degenerate bases with the most frequent base of other sequences and appending two random ACGT anchor segments of length k to the front and end of each sequence to guarantee unique start and end nodes. Then perform a postorder-like traversal applying the cSupB matching principle: a source and sink pair qualifies when at least two adjacent outgoing edges' colors of the source node intersect with the color of the sink node. If the traversal finishes with unvisited nodes remaining, cycles must exist; identify them via offset values, locate the cycle interval by marking a half-visited node fully visited and restarting traversal, then cut the sequence at a selected cutting point and reconstruct the graph. Finally, reorganize all cSupBs into a hierarchical tree, assigning each cSupB a hierarchical level with the root cSupB containing all samples; the child-before-parent corollary guarantees the structure is a tree. The decomposition is implemented in the C++ program cosmo-cSupB, which consumes the VARI-merge .dbg graph and .colors.sd_vector color matrix.

## Related tools

- **cSupB** (C++ program (cosmo-cSupB) that performs the postorder-like traversal, cSupB matching, cycle handling, and hierarchical tree reorganization on the VARI-merge graph) — https://github.com/eggleader/cSupB
- **VARI-merge** (Constructs the succinct colored de Bruijn graph (.dbg and .colors.sd_vector) that is decomposed into cSupBs) — https://github.com/cosmo-team/cosmo/tree/VARI-merge

## Examples

```
cosmo-cSupB primates_mtDNAs_kmc_list.dbg primates_mtDNAs_kmc_list.colors.sd_vector >log.txt
```

## Evaluation signals

- Every cSupB satisfies the matching condition: the color intersection between the sink node and at least two adjacent outgoing edges of the source node is non-empty
- The cSupB structure forms a valid tree — a child cSupB is never obtained before its parent, and each cSupB carries a unique hierarchical level under the root cSupB containing all samples
- No unvisited nodes remain after traversal, or cycles are detected and resolved via offset-value identification, interval location, and sequence cutting with graph reconstruction
- On the 12 human mitogenomes at k=22 the resulting graph is directed, acyclic, nondegenerate, with unique start and end nodes
- cSupB analysis runtime scales as reported: ~7.5 h + 3 min for 5,000 samples and ~30.5 h + 8 min for 20,000 samples (roughly 4x growth on a fourfold sample increase)

## Limitations

- Colored de Bruijn graph construction tools, including VARI-merge, lack mapping functionality, which limits data selection and model promotion
- For highly repetitive sequences, especially tandem repeats, the cutting method fragments the sequence too much, increasing computational difficulty and error
- Variant detection precision decreases when sample similarity decreases and variant types become complicated; applicable only to high-similarity samples with simple variant types
- The choice of k value impacts the accuracy of the algorithm, and no VARI-merge version is stated
- The cSupB motif assumes a colored directed acyclic graph; cycles require the offset-value identification and cutting-point reconstruction workaround

## Evidence

- [abstract] We developed a new method, a colored superbubble (cSupB), that can overcome the complexity of graphs and organize a set of species- or population-specific haplotype sequences of interest: "We developed a new method, a colored superbubble (cSupB), that can overcome the complexity of graphs and organize a set of species- or population-specific haplotype sequences of interest"
- [methods] At least two adjacent outgoing edges’ colors of the source node intersect with the color of the sink node: "At least two adjacent outgoing edges’ colors of the source node intersect with the color of the sink node"
- [methods] if the traversal is finished but nodes still exist that are unvisited, we call the traversal finished in advance and there must be cycles in graph: "if the traversal is finished but nodes still exist that are unvisited, we call the traversal finished in advance and there must be cycles in graph"
- [methods] a child cSupB cannot be obtained before its parent cSupB: "a child cSupB cannot be obtained before its parent cSupB (Additional file 1: Corollary 6)"
- [results] Here, we set k = 22 for VARI-merge to construct an acyclic graph.: "Here, we set k = 22 for VARI-merge to construct an acyclic graph."
- [readme] To obtain the coordinate system or variants, simply use the command line as follows: cosmo-cSupB primates_mtDNAs_kmc_list.dbg primates_mtDNAs_kmc_list.colors.sd_vector >log.txt: "cosmo-cSupB primates_mtDNAs_kmc_list.dbg primates_mtDNAs_kmc_list.colors.sd_vector >log.txt"

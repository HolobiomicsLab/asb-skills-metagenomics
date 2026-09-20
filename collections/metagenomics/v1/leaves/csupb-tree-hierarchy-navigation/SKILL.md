---
name: csupb-tree-hierarchy-navigation
description: Use when a pangenome graph (e.g., from 12 human mitogenomes built with
  VARI-merge, k=22) must be navigated structurally — ordering/nesting colored superbubbles,
  resolving cycles, or mapping linear-reference coordinates and annotations onto graph
  nodes.
when_to_use_negative:
- 'Sequences are highly repetitive (e.g., tandem repeats): the cutting method fragments
  sequences too much, increasing computational difficulty and error'
- 'Sample similarity is insufficient (source number far below variant number): the
  variant detection model built on the tree is not applicable'
- 'Gene elements are shorter than k: they may map to multiple graph nodes, making
  the analysis complicated and ambiguous'
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0622
tools:
- name: cSupB (VARI-cSupB / cosmo-cSupB)
  role: C++ program that decomposes the colored de Bruijn graph into cSupBs, reorganizes
    them into a hierarchical tree, and emits CST/CSDI/NNI/OVI files plus tri-tuple
    coordinates and variants
  repo: https://github.com/eggleader/cSupB
- name: VARI-merge
  role: Constructs the succinct colored de Bruijn graph (set k=22 for an acyclic graph)
    whose decomposition is navigated by this skill; installation is identical to VARI-merge
  repo: https://github.com/cosmo-team/cosmo/tree/VARI-merge
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-021-04149-w
    title: A tri-tuple coordinate system derived for fast and accurate analysis of
      the colored de Bruijn graph-based pangenomes
  source_content_hash: sha256:6b9f9eff665cdc6b56d7404da04f4589104e3aeb75f4f8365ed4fc5c12fcab28
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eggleader/cSupB
  tools:
  - C++
  - VARI-merge
  - cSupB
  - cSupB (VARI-cSupB / cosmo-cSupB)
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# csupb-tree-hierarchy-navigation

## Summary

Navigate the colored superbubble (cSupB) hierarchical tree decomposed from a VARI-merge colored de Bruijn graph and assign each base a tri-tuple coordinate (position, bubid, basecolor), where bubid is the smallest containing cSupB. This organizes species- or population-specific haplotype sequences and provides reference-compatible coordinates (e.g., for GTF gene mapping).

## When to use

Use when a pangenome graph (e.g., from 12 human mitogenomes built with VARI-merge, k=22) must be navigated structurally — ordering/nesting colored superbubbles, resolving cycles, or mapping linear-reference coordinates and annotations onto graph nodes.

## When NOT to use

- Sequences are highly repetitive (e.g., tandem repeats): the cutting method fragments sequences too much, increasing computational difficulty and error
- Sample similarity is insufficient (source number far below variant number): the variant detection model built on the tree is not applicable
- Gene elements are shorter than k: they may map to multiple graph nodes, making the analysis complicated and ambiguous

## Inputs

- Colored de Bruijn graph built by VARI-merge (e.g., k=22) from multiple genomes (e.g., 12 human mitochondrial genomes such as AP008459, NC_012920)
- Color matrix / dbg files from VARI-merge (e.g., primates_mtDNAs_kmc_list.dbg, primates_mtDNAs_kmc_list.colors.sd_vector)
- cSupB decomposition files: CST (topology), CSDI (detailed information), NNI (node nearby information), OVI (offset value information)
- Optional linear-reference annotation (e.g., mitochondrial GTF gene coordinates) to project onto the graph

## Outputs

- Hierarchical tree of nested colored superbubbles with assigned hierarchical levels
- Tri-tuple coordinates (position, bubid, basecolor) for every node and base
- Reference-position mapping (nodepos to refpos) via RPD
- Graph-annotated records (e.g., gtf2graph.txt / mitochondrial_gene_regions.gtf) with reference start/end coordinates

## How to apply

Load the cSupB decomposition files produced by the cosmo-cSupB C++ program: CST (cSupB topology), CSDI (cSupB detailed information), NNI (node nearby information) and OVI (offset value information). Traverse the cSupB hierarchical tree postorder-like from the root bubble (the cSupB containing all samples) to obtain the ordering and nesting relations of all colored superbubbles; a child cSupB cannot be obtained before its parent, and the cSupB structure must be a tree. During traversal, detect cycles via the offset value, and if traversal finishes in advance with unvisited nodes, mark a half-visited node fully visited and restart. For each node and base compute the offset value from source/sink node positions and record the tri-tuple (position, bubid, basecolor) where basecolor is the 0-1 sample string. Downstream, apply Reference Position Determination (RPD) — mapping nodepos to refpos via the nearest parent source node containing the reference genome — so linear annotations (e.g., a mitochondrial GTF) can be projected onto the graph.

## Related tools

- **cSupB (VARI-cSupB / cosmo-cSupB)** (C++ program that decomposes the colored de Bruijn graph into cSupBs, reorganizes them into a hierarchical tree, and emits CST/CSDI/NNI/OVI files plus tri-tuple coordinates and variants) — https://github.com/eggleader/cSupB
- **VARI-merge** (Constructs the succinct colored de Bruijn graph (set k=22 for an acyclic graph) whose decomposition is navigated by this skill; installation is identical to VARI-merge) — https://github.com/cosmo-team/cosmo/tree/VARI-merge

## Examples

```
cosmo-cSupB primates_mtDNAs_kmc_list.dbg primates_mtDNAs_kmc_list.colors.sd_vector >log.txt
```

## Evaluation signals

- The cSupB structure resolves to a tree (each child obtained after its parent; traversal from the root bubble covering all samples succeeds)
- Every gene region resolves to a unique (position, bubid, basecolor) triple
- Emitted GTF (mitochondrial_gene_regions.gtf) matches the deposited reference GTF (Additional file 4 of 10.1186/s12859-021-04149-w) in gene feature counts (37 genes) and coordinate intervals (output_matches_reference)
- No traversal finishes 'in advance' with unvisited nodes remaining unless cycles were identified and handled by marking a half-visited node fully visited and restarting
- Mapped annotation shows expected overlap pattern (all 37 GTF genes located on bridges or spanning at least one cSupB)

## Limitations

- Variant detection handles only small indels (<50 bp) for highly similar samples; no effective solution for large indels, inversions, or reverse tandem duplications
- Degeneracy in the graph cannot be handled efficiently; colored de Bruijn graph tools like VARI-merge lack built-in mapping functionality
- Choice of k value impacts accuracy; genes shorter than k may map ambiguously to multiple nodes

## Evidence

- [abstract] we propose a tri-tuple coordinate system that combines an offset value, topological structure and sample information: "we propose a tri-tuple coordinate system that combines an offset value, topological structure and sample information"
- [methods] a child cSupB cannot be obtained before its parent cSupB: "a child cSupB cannot be obtained before its parent cSupB (Additional file 1: Corollary 6)"
- [methods] we need to identify this node, mark it as fully visited, and restart the graph traversal: "we need to identify this node, mark it as fully visited, and restart the graph traversal"
- [results] 37 genes in the gtf file are located on the bridges or span at least one cSupB: [evidence span withheld — anchor status: not_found]
- [readme] We only need to add the files (Makefile and cosmo-cSupB.cpp) to VARI-merge's main directory.: "We only need to add the files (Makefile and cosmo-cSupB.cpp) to VARI-merge's main directory."

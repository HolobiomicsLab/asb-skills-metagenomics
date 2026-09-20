---
name: offset-value-coordinate-assignment
description: Use when you need per-node or per-base coordinates on a VARI-merge colored
  de Bruijn graph pangenome — e.g., for offset-based cycle detection, reference position
  determination (RPD), or projecting GTF gene annotations onto the graph as (position,
  bubid, basecolor) triples.
when_to_use_negative:
- 'Gene or feature elements shorter than k (e.g., < 22 bp): they may correspond to
  multiple graph nodes, making the assigned coordinates complicated and ambiguous'
- Graphs containing degenerate positions — the method cannot handle degeneracy efficiently
- Highly repetitive sequences such as tandem repeats, where sequence cutting fragments
  the sequence too much and increases computational difficulty and error
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0654
tools:
- name: VARI-merge
  role: Constructs the succinct colored de Bruijn graph (k = 22) whose cSupB decomposition
    outputs (CST/CSDI/NNI/OVI) feed the offset-value computation
  repo: https://github.com/cosmo-team/cosmo/tree/VARI-merge
- name: cSupB
  role: C++ program (cosmo-cSupB) that computes the tri-tuple coordinate system and
    offset values from the VARI-merge graph and color matrix
  repo: https://github.com/eggleader/cSupB
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-021-04149-w
    title: A tri-tuple coordinate system derived for fast and accurate analysis of
      the colored de Bruijn graph-based pangenomes
  source_content_hash: sha256:6ae366544a12c25976e2aba89290d1c9909c1072708f73d61367c853f7e09f64
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - C++
  - VARI-merge
  - cSupB
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# offset-value-coordinate-assignment

## Summary

Assigns each node and base of a VARI-merge colored de Bruijn graph an offset value computed from source/sink node positions, completing the tri-tuple (position, bubid, basecolor) that uniquely locates any graph element. This coordinate layer is what allows existing linear references (e.g., GTF gene coordinates on the 12 human mitogenomes) to be projected onto the graph pangenome.

## When to use

Use when you need per-node or per-base coordinates on a VARI-merge colored de Bruijn graph pangenome — e.g., for offset-based cycle detection, reference position determination (RPD), or projecting GTF gene annotations onto the graph as (position, bubid, basecolor) triples.

## When NOT to use

- Gene or feature elements shorter than k (e.g., < 22 bp): they may correspond to multiple graph nodes, making the assigned coordinates complicated and ambiguous
- Graphs containing degenerate positions — the method cannot handle degeneracy efficiently
- Highly repetitive sequences such as tandem repeats, where sequence cutting fragments the sequence too much and increases computational difficulty and error

## Inputs

- cSupB decomposition files of the VARI-merge graph: cSupB topology (CST), cSupB detailed information (CSDI), node nearby information (NNI), offset value information (OVI)
- VARI-merge colored de Bruijn graph and color matrix (e.g., .dbg and .colors.sd_vector built from a kmc_list)
- Sample sequence set used to build the graph (e.g., 12 human mitochondrial genomes: AP008459, AP010675, EF153784, EF153791, EF153794, EF153814, EF397559, EU007868, FJ493500, GQ895144, GU377085, NC_012920)
- Reference genome and annotation file (GTF) for Reference Position Determination (RPD) mapping

## Outputs

- Tri-tuples (position, bubid, basecolor) for every node and base, with bubid the smallest containing cSupB and basecolor the 0-1 sample string
- Offset values for graph nodes relative to source/sink node positions
- nodepos -> refpos mappings and projected gene-region records (e.g., gtf2graph.txt / mitochondrial_gene_regions.gtf)

## How to apply

Construct the graph with VARI-merge at k = 22 (k chosen as the maximum longest repeat segment length across sequences) and add random ACGT anchor segments of length k to the front and end of each sequence so the graph is directed, acyclic, nondegenerate, with unique start and end nodes. Load the cSupB decomposition files — cSupB topology (CST), cSupB detailed information (CSDI), node nearby information (NNI) and offset value information (OVI) — then traverse the cSupB hierarchical tree from the root bubble (containing all samples) to recover the ordering and nesting of all colored superbubbles. For each node and base, compute the offset value from the source/sink node positions and record the tri-tuple (position, bubid, basecolor), where bubid is the smallest containing cSupB and basecolor is the 0-1 sample string. During the postorder-like traversal, use offset values for cycle identification: if traversal finishes with unvisited nodes remaining, a cycle exists — mark the half-visited node fully visited and restart the traversal. Apply Reference Position Determination (RPD) to map nodepos onto refpos via the nearest parent source node containing the reference genome, which is what allows mitochondrial GTF gene elements to be projected onto the graph.

## Related tools

- **VARI-merge** (Constructs the succinct colored de Bruijn graph (k = 22) whose cSupB decomposition outputs (CST/CSDI/NNI/OVI) feed the offset-value computation) — https://github.com/cosmo-team/cosmo/tree/VARI-merge
- **cSupB** (C++ program (cosmo-cSupB) that computes the tri-tuple coordinate system and offset values from the VARI-merge graph and color matrix) — https://github.com/eggleader/cSupB

## Examples

```
cosmo-cSupB primates_mtDNAs_kmc_list.dbg primates_mtDNAs_kmc_list.colors.sd_vector >log.txt
```

## Evaluation signals

- Every gene region resolves to a unique (position, bubid, basecolor) triple
- The emitted GTF (e.g., mitochondrial_gene_regions.gtf) matches the deposited reference GTF (Additional file 4 of 10.1186/s12859-021-04149-w) in gene feature counts and coordinate intervals
- The cSupB hierarchy is a tree: each child cSupB has exactly one parent, and traversal from the root bubble orders all cSupBs consistently
- No unresolved nodes remain: any cycle flagged by the offset-value method is located, its half-visited node marked fully visited, and traversal restarted to completion
- RPD mappings agree with reference-genome intervals derived via the nearest parent source node containing the reference genome

## Limitations

- Gene elements shorter than k may map to multiple nodes, making the analysis complicated and ambiguous
- Degeneracy cannot be handled efficiently
- The choice of the k value impacts the accuracy of the algorithm
- For highly repetitive sequences, especially tandem repeats, the cutting method fragments the sequence too much, increasing computational difficulty and error

## Evidence

- [abstract] Offset value is one of the three components of the graph coordinate system: "we propose a tri-tuple coordinate system that combines an offset value, topological structure and sample information"
- [methods] Offset values are used to identify cycles during postorder-like traversal: "we propose a cycle identification method based on the offset value"
- [methods] RPD maps graph node positions onto reference-genome positions: "Determine the relations between the nodes' position on the reference genome and graph (reference position determination (RPD)); then, transform the node variant to a locus variant"
- [results] k = 22 was used for VARI-merge graph construction: "Here, we set k = 22 for VARI-merge to construct an acyclic graph."
- [readme] The offset-based cSupB implementation tolerates cyclic graph structure: "Moreover, VARI-cSupB can adapt to a complex cycle structure."

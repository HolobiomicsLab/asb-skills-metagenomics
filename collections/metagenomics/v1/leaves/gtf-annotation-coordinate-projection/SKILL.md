---
name: gtf-annotation-coordinate-projection
description: Use when a GTF/GFF annotation built on a linear reference genome must
  be lifted onto a pangenome graph (constructed with VARI-merge, k=22) so gene regions
  can be located via cSupB bubble coordinates.
when_to_use_negative:
- 'Gene elements shorter than k (k=22): they may map to multiple nodes on the graph,
  making the analysis complicated and ambiguous'
- The graph is degenerate or contains unresolved cycles — the method requires a directed,
  acyclic, nondegenerate graph with unique start and end nodes
- The annotation references an organism/genome absent from the sample color set, so
  no parent source node containing the reference genome exists for RPD
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: cSupB (VARI-cSupB)
  role: C++ program that performs the cSupB decomposition of the VARI-merge graph
    and yields the tri-tuple coordinate system used to anchor GTF features
  repo: https://github.com/eggleader/cSupB
- name: VARI-merge
  role: Constructs the succinct colored de Bruijn graph (k=22, acyclic) from the 12
    human mitogenomes whose cSupB files are loaded for coordinate mapping
  repo: https://github.com/cosmo-team/cosmo/tree/VARI-merge
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-021-04149-w
    title: A tri-tuple coordinate system derived for fast and accurate analysis of
      the colored de Bruijn graph-based pangenomes
  source_content_hash: sha256:3404380bfe0d218b81df7cd00dc34a090770ce35cc08298bdf2f49c26ca188d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eggleader/cSupB
  tools:
  - C++
  - VARI-merge
  - cSupB (VARI-cSupB)
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# gtf-annotation-coordinate-projection

## Summary

Projects linear-reference GTF gene annotations (e.g., mitochondrial gene coordinates) onto a colored de Bruijn graph pangenome by mapping reference positions through the tri-tuple coordinate system (position, bubid, basecolor) derived from the cSupB decomposition. It demonstrates that existing linear annotation data remain usable on graph-based pangenome coordinates.

## When to use

Use when a GTF/GFF annotation built on a linear reference genome must be lifted onto a pangenome graph (constructed with VARI-merge, k=22) so gene regions can be located via cSupB bubble coordinates.

## When NOT to use

- Gene elements shorter than k (k=22): they may map to multiple nodes on the graph, making the analysis complicated and ambiguous
- The graph is degenerate or contains unresolved cycles — the method requires a directed, acyclic, nondegenerate graph with unique start and end nodes
- The annotation references an organism/genome absent from the sample color set, so no parent source node containing the reference genome exists for RPD

## Inputs

- GTF annotation file of mitochondrial gene coordinates (e.g., Additional file 4 of 10.1186/s12859-021-04149-w)
- cSupB decomposition files from VARI-merge (k=22): CST, CSDI, NNI, OVI
- Colored de Bruijn graph (.dbg) and color matrix (.colors.sd_vector) built by VARI-merge
- Reference mitogenome sequence (e.g., NC_012920)

## Outputs

- mitochondrial_gene_regions.gtf — gene regions with start/end reference coordinates projected onto the graph
- gtf2graph.txt — per-gene graph mapping report (bridges, containing cSupB bubid, tri-tuples)
- Unique (position, bubid, basecolor) triples for every gene region

## How to apply

Load the VARI-merge cSupB decomposition files (CST topology, CSDI details, NNI node-nearby information, OVI offset values) for the colored de Bruijn graph built at k=22, and traverse the cSupB hierarchical tree from the root bubble to recover the ordering and nesting of all colored superbubbles. For each base compute the offset value from source/sink node positions and record the tri-tuple (position, bubid, basecolor), where bubid is the smallest containing cSupB and basecolor is the 0-1 sample presence string. Apply Reference Position Determination (RPD) to map graph node positions (nodepos) onto reference-genome positions (refpos) via the nearest parent source node containing the reference genome. Project each GTF gene element using the refpos relation, emitting start/end reference coordinates; genes falling on bridges or spanning a cSupB are still resolvable through the smallest containing bubble. Write records to mitochondrial_gene_regions.gtf and validate against the deposited reference GTF for gene-feature counts and coordinate intervals.

## Related tools

- **cSupB (VARI-cSupB)** (C++ program that performs the cSupB decomposition of the VARI-merge graph and yields the tri-tuple coordinate system used to anchor GTF features) — https://github.com/eggleader/cSupB
- **VARI-merge** (Constructs the succinct colored de Bruijn graph (k=22, acyclic) from the 12 human mitogenomes whose cSupB files are loaded for coordinate mapping) — https://github.com/cosmo-team/cosmo/tree/VARI-merge

## Examples

```
cosmo-cSupB primates_mtDNAs_kmc_list.dbg primates_mtDNAs_kmc_list.colors.sd_vector >log.txt
```

## Evaluation signals

- The emitted mitochondrial_gene_regions.gtf matches the deposited reference GTF (Additional file 4) in gene feature counts and coordinate intervals (output_matches_reference)
- Every projected gene region resolves to a unique (position, bubid, basecolor) triple
- All 37 genes in the reference GTF are located — each falls on a bridge or spans at least one cSupB in gtf2graph.txt
- RPD mappings map every projected nodepos back to the correct refpos interval of the reference genome (e.g., NC_012920)

## Limitations

- Gene elements shorter than k may correspond to multiple nodes on the graph, making the analysis complicated and ambiguous
- Degeneracy cannot be handled efficiently; the construction assumes a directed, acyclic, nondegenerate graph with unique start and end nodes
- VARI-merge (and most colored de Bruijn graph tools) lack mapping functionality, so this projection must be layered on externally via the cSupB coordinate system

## Evidence

- [results] mapping the existing annotation file onto the graph shows 37 genes in the gtf file are located on the bridges or span at least one cSupB: [evidence span withheld — anchor status: not_found]
- [methods] RPD definition: "Determine the relations between the nodes' position on the reference genome and graph (reference position determination (RPD)); then, transform the node variant to a locus variant"
- [discussion] short gene elements ambiguity: "if the length of the gene element is less than k, there may be multiple corresponding nodes on the graph, which makes the analysis complicated and ambiguous"
- [readme] cSupB command line usage: "cosmo-cSupB primates_mtDNAs_kmc_list.dbg primates_mtDNAs_kmc_list.colors.sd_vector >log.txt"

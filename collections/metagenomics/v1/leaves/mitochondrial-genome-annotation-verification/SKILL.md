---
name: mitochondrial-genome-annotation-verification
description: Use when you need to transfer an existing linear-reference annotation
  (e.g., a GTF of mitochondrial gene coordinates) onto a graph pangenome and confirm
  each gene region resolves uniquely in graph coordinates.
when_to_use_negative:
- Gene elements shorter than k (= 22), which may map to multiple nodes on the graph,
  making the analysis complicated and ambiguous
- Graphs where sequence similarity is insufficient (source number far less than variant
  number), where the variant/coordinate model is not applicable
- Highly repetitive sequences (especially tandem repeats), where the cutting method
  fragments the sequence too much, increasing computational difficulty and error
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: cSupB
  role: Decomposes the VARI-merge colored de Bruijn graph into colored superbubbles
    and derives the tri-tuple coordinate system used to anchor gene annotations on
    the graph
  repo: https://github.com/eggleader/cSupB
- name: VARI-merge
  role: Constructs the succinct colored de Bruijn graph (k = 22) and color matrix
    from the 12 human mitogenomes that annotations are projected onto
  repo: https://github.com/cosmo-team/cosmo/tree/VARI-merge
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-021-04149-w
    title: A tri-tuple coordinate system derived for fast and accurate analysis of
      the colored de Bruijn graph-based pangenomes
  source_content_hash: sha256:d29ed195389595b9aefbbe1add21d4e620d93591f54034ca98b17f54e43601a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eggleader/cSupB
  tools:
  - C++
  - VARI-merge
  - cSupB
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# mitochondrial-genome-annotation-verification

## Summary

Project linear-reference mitochondrial gene annotations (GTF) onto a colored de Bruijn graph pangenome via the paper's tri-tuple coordinate system (position, bubid, basecolor), verifying that existing reference coordinates remain accommodated on the graph.

## When to use

Use when you need to transfer an existing linear-reference annotation (e.g., a GTF of mitochondrial gene coordinates) onto a graph pangenome and confirm each gene region resolves uniquely in graph coordinates.

## When NOT to use

- Gene elements shorter than k (= 22), which may map to multiple nodes on the graph, making the analysis complicated and ambiguous
- Graphs where sequence similarity is insufficient (source number far less than variant number), where the variant/coordinate model is not applicable
- Highly repetitive sequences (especially tandem repeats), where the cutting method fragments the sequence too much, increasing computational difficulty and error

## Inputs

- GTF annotation file of mitochondrial gene coordinates on the linear reference genome (e.g., NC_012920)
- cSupB decomposition files from VARI-merge (k = 22): CST (cSupB topology), CSDI (cSupB detailed information), NNI (node nearby information), OVI (offset value information)
- Colored de Bruijn graph and color matrix constructed by VARI-merge from 12 human mitochondrial genomes (AP008459, AP010675, EF153784, EF153791, EF153794, EF153814, EF397559, EU007868, FJ493500, GQ895144, GU377085, NC_012920)

## Outputs

- mitochondrial_gene_regions.gtf: gene records with start/end reference coordinates projected onto the graph
- gtf2graph.txt: mapping of annotation onto graph coordinates (Additional file 4 equivalent)
- Per-gene tri-tuples (position, bubid, basecolor) with unique resolution

## How to apply

Construct the colored de Bruijn graph with VARI-merge (k = 22, chosen as the maximum longest repeat segment length across the 12 human mitogenomes) and decompose it into colored superbubbles using cSupB, loading the CST, CSDI, NNI and OVI files. Traverse the cSupB hierarchical tree from the root bubble to establish ordering and nesting of all colored superbubbles, then compute for each node and base the offset value from source/sink node positions and record the tri-tuple (position, bubid, basecolor), where bubid is the smallest containing cSupB and basecolor is a 0-1 sample string. Apply Reference Position Determination (RPD) to map nodepos onto refpos via the nearest parent source node containing the reference genome. Project each gene element from the GTF onto the graph using the refpos relationship and emit gene regions with start/end reference coordinates to mitochondrial_gene_regions.gtf, recording genes located on bridges or spanning at least one cSupB.

## Related tools

- **cSupB** (Decomposes the VARI-merge colored de Bruijn graph into colored superbubbles and derives the tri-tuple coordinate system used to anchor gene annotations on the graph) — https://github.com/eggleader/cSupB
- **VARI-merge** (Constructs the succinct colored de Bruijn graph (k = 22) and color matrix from the 12 human mitogenomes that annotations are projected onto) — https://github.com/cosmo-team/cosmo/tree/VARI-merge

## Examples

```
cosmo-cSupB primates_mtDNAs_kmc_list.dbg primates_mtDNAs_kmc_list.colors.sd_vector >log.txt
```

## Evaluation signals

- The emitted mitochondrial_gene_regions.gtf matches the deposited reference GTF (Additional file 4 of 10.1186/s12859-021-04149-w) in gene feature counts (37 genes) and coordinate intervals (output_matches_reference)
- Every gene region resolves to a unique (position, bubid, basecolor) triple
- Gene elements located on bridges or spanning at least one cSupB are correctly reported (gtf2graph.txt)
- RPD maps each graph nodepos onto a refpos via the nearest parent source node containing the reference genome, with no unmapped reference positions in the annotation

## Limitations

- If the length of the gene element is less than k, there may be multiple corresponding nodes on the graph, making analysis complicated and ambiguous
- Colored de Bruijn graph construction tools, including VARI-merge, lack mapping functionality, which limits data selection and model promotion
- Degeneracy cannot be handled efficiently
- The cSupB method was demonstrated on an acyclic-ified graph (k = 22); cycles require cycle identification and possible sequence segmentation and graph reconstruction

## Evidence

- [results] mapping the existing annotation file onto the graph shows 37 genes in the gtf file are located on the bridges or span at least one cSupB: [evidence span withheld — anchor status: not_found]
- [methods] Determine the relations between the nodes' position on the reference genome and graph (RPD): "Determine the relations between the nodes' position on the reference genome and graph (reference position determination (RPD)); then, transform the node variant to a locus variant"
- [results] we set k = 22 for VARI-merge to construct an acyclic graph: "Here, we set k = 22 for VARI-merge to construct an acyclic graph."
- [discussion] if the length of the gene element is less than k, there may be multiple corresponding nodes on the graph: "if the length of the gene element is less than k, there may be multiple corresponding nodes on the graph, which makes the analysis complicated and ambiguous"
- [readme] To obtain the coordinate system or variants, simply use the command line as follows: "To obtain the coordinate system or variants, simply use the command line as follows:<tab>"

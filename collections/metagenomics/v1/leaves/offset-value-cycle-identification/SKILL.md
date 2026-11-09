---
name: offset-value-cycle-identification
description: Use when building a colored de Bruijn graph over highly similar haplotypes
  (e.g., mitochondrial genomes) and the postorder-like traversal finishes with unvisited
  nodes, signaling cycles that block cSupB decomposition.
when_to_use_negative:
- The graph is already acyclic (traversal completes with all nodes visited) — cycle
  handling is unnecessary and sequence cutting would fragment repeats unnecessarily
- Sequences are highly repetitive (especially tandem repeats), where the cutting method
  'will make the sequence too fragmented, thereby increasing the computational difficulty
  and error'
- Sequence similarity is insufficient (source number far less than variant number),
  where the variant detection model this cycle handling feeds into is not applicable
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: VARI-merge
  role: Constructs the colored de Bruijn graph whose traversal may finish in advance
    due to cycles; the reconstructed graph after sequence cutting is re-built with
    it
  repo: https://github.com/cosmo-team/cosmo/tree/VARI-merge
- name: cSupB
  role: C++ program whose postorder-like traversal triggers cycle identification via
    offset values and consumes the reconstructed acyclic graph for cSupB tree reorganization
  repo: https://github.com/eggleader/cSupB
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-021-04149-w
    title: A tri-tuple coordinate system derived for fast and accurate analysis of
      the colored de Bruijn graph-based pangenomes
  source_content_hash: sha256:bb212cfe61eab09f98da532326605ce3bceb6d756c2f62fdeabea64bd842ef52
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - C++
  - VARI-merge
  - cSupB
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# offset-value-cycle-identification

## Summary

A method from the cSupB/VARI-cSupB pipeline that detects cycles in the colored de Bruijn graph during postorder-like traversal using offset values, then localizes the cycle interval and reconstructs an acyclic graph by cutting the underlying sequences at a selected point.

## When to use

Use when building a colored de Bruijn graph over highly similar haplotypes (e.g., mitochondrial genomes) and the postorder-like traversal finishes with unvisited nodes, signaling cycles that block cSupB decomposition.

## When NOT to use

- The graph is already acyclic (traversal completes with all nodes visited) — cycle handling is unnecessary and sequence cutting would fragment repeats unnecessarily
- Sequences are highly repetitive (especially tandem repeats), where the cutting method 'will make the sequence too fragmented, thereby increasing the computational difficulty and error'
- Sequence similarity is insufficient (source number far less than variant number), where the variant detection model this cycle handling feeds into is not applicable

## Inputs

- Colored de Bruijn graph (.dbg) and color matrix (.colors.sd_vector) built by VARI-merge from haplotype sequences (e.g., the 12 human mitochondrial genomes)
- Postorder-like traversal state with per-node offset values (half-visited vs fully-visited status)
- Underlying sample sequences (FASTA-like haplotypes, e.g., AP008459, AP010675, EF397559, NC_012920)

## Outputs

- Identified cycle locations (nodes/edges participating in cycles, flagged via offset-value analysis)
- Cycle interval with a selected cutting point
- Reconstructed acyclic graph with unique start and end nodes, ready for cSupB tree reorganization

## How to apply

After constructing the colored de Bruijn graph (e.g., with VARI-merge at k=22 for the 12 human mitogenomes), run the postorder-like traversal; if the traversal finishes but nodes remain unvisited, cycles must exist ('finished in advance'). During traversal, track offset values so that a half-visited node revisited later is identified by its offset value as part of a cycle. Locate the cycle interval by marking the half-visited node as fully visited and restarting the graph traversal. Finally, apply the cutting point selection model to choose a point inside the cycle interval, divide the sequence into two parts, and reconstruct the graph, which removes the cycle and allows decomposition into the hierarchical cSupB tree.

## Related tools

- **VARI-merge** (Constructs the colored de Bruijn graph whose traversal may finish in advance due to cycles; the reconstructed graph after sequence cutting is re-built with it) — https://github.com/cosmo-team/cosmo/tree/VARI-merge
- **cSupB** (C++ program whose postorder-like traversal triggers cycle identification via offset values and consumes the reconstructed acyclic graph for cSupB tree reorganization) — https://github.com/eggleader/cSupB

## Examples

```
cosmo-cSupB primates_mtDNAs_kmc_list.dbg primates_mtDNAs_kmc_list.colors.sd_vector >log.txt
```

## Evaluation signals

- After handling, a full postorder-like traversal visits all nodes (no 'finished in advance' state remains)
- The reconstructed graph is directed, acyclic, nondegenerate, with unique start and end nodes
- No sequence content is lost: cutting divides the sequence into exactly two parts and the graph is rebuilt from both
- Downstream cSupB reorganization succeeds, forming a tree in which a child cSupB cannot be obtained before its parent
- On the 12 mitogenome dataset, variant calls remain consistent with the reference set (89 of 110 SNVs location-mapped to the 1000 Genomes mtDNA annotation)

## Limitations

- For highly repetitive sequences, especially tandem repeats, the cutting method makes sequences too fragmented, increasing computational difficulty and error
- Choice of the k value impacts accuracy of the overall detection pipeline into which cycle handling feeds
- Degeneracy cannot be handled efficiently even after cycle removal

## Evidence

- [methods] finished-in-advance signal: "if the traversal is finished but nodes still exist that are unvisited, we call the traversal finished in advance and there must be cycles in graph"
- [methods] offset-value basis: "we propose a cycle identification method based on the offset value"
- [methods] interval localization by restart: "we need to identify this node, mark it as fully visited, and restart the graph traversal"
- [methods] sequence cutting and reconstruction: "we select a cutting point in this interval to divide the sequence into two parts and construct the graph"
- [readme] cycle adaptability claim: "Moreover, VARI-cSupB can adapt to a complex cycle structure."

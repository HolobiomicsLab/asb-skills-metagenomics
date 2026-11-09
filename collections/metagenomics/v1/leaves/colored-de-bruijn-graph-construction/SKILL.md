---
name: colored-de-bruijn-graph-construction
description: Use when building a compact pangenome/colored de Bruijn graph from a
  panel of highly similar FASTA sequences (e.g., the 12 human mitogenomes, or 5k/20k
  simulated panels) as the input substrate for cSupB decomposition and variant detection.
when_to_use_negative:
- Input sequences have insufficient similarity (source number far less than variant
  number), which the article states makes the variant detection model inapplicable
- The dataset contains highly repetitive sequences (especially tandem repeats), where
  the sequence-cutting/graph reconstruction step fragments sequences too much
- The goal requires handling degeneracy or structural variants such as large indels,
  inversions, or reverse tandem duplications, which the method cannot address efficiently
edam_operation: http://edamontology.org/operation_0523
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: VARI-merge
  role: Constructs the succinct colored de Bruijn graph and color matrix from the
    preprocessed haplotype panel at the chosen k (k = 22)
  repo: https://github.com/cosmo-team/cosmo/tree/VARI-merge
- name: Cosmo
  role: Underlying succinct de Bruijn graph assembler/infrastructure on whose VARI
    branch VARI-merge is built
  repo: https://github.com/cosmo-team/cosmo
- name: cSupB
  role: cSupB program installed into the VARI-merge main directory to consume the
    constructed graph and color matrix for decomposition and coordinate derivation
  repo: https://github.com/eggleader/cSupB
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_005
  - task_002
  source_papers:
  - doi: 10.1186/s12859-021-04149-w
    title: A tri-tuple coordinate system derived for fast and accurate analysis of
      the colored de Bruijn graph-based pangenomes
  source_content_hash: sha256:5997d817048080cb3b78f45c432ce847946e128923f095e468bb7c6e90ada9f2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - C++
  - Clustal Omega
  - Cosmo
  - MAFFT-GINSI
  - MUSCLE
  - Mugsy
  - VARI-merge
  - cSupB
  edam_operation: http://edamontology.org/operation_0523
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# colored-de-bruijn-graph-construction

## Summary

Constructs a colored de Bruijn graph from multiple haplotype sequences (e.g., human mitochondrial genomes) using VARI-merge at a k-mer size chosen to exceed the maximum longest repeat segment length, yielding a directed, acyclic, nondegenerate graph with unique start and end nodes suitable for downstream cSupB decomposition.

## When to use

Use when building a compact pangenome/colored de Bruijn graph from a panel of highly similar FASTA sequences (e.g., the 12 human mitogenomes, or 5k/20k simulated panels) as the input substrate for cSupB decomposition and variant detection.

## When NOT to use

- Input sequences have insufficient similarity (source number far less than variant number), which the article states makes the variant detection model inapplicable
- The dataset contains highly repetitive sequences (especially tandem repeats), where the sequence-cutting/graph reconstruction step fragments sequences too much
- The goal requires handling degeneracy or structural variants such as large indels, inversions, or reverse tandem duplications, which the method cannot address efficiently

## Inputs

- Panel of highly similar haplotype FASTA sequences (e.g., 12 human mitochondrial genomes: AP008459, AP010675, EF153784, EF153791, EF153794, EF153814, EF397559, EU007868, FJ493500, GQ895144, GU377085, NC_012920, or simulated panels of 5,000 and 20,000 samples)
- k-mer size k chosen greater than the maximum longest repeat segment length K across sequences (k = 22 in the paper)

## Outputs

- Succinct colored de Bruijn graph (.dbg) with associated color matrix (.colors.sd_vector) from VARI-merge, directed/acyclic/nondegenerate with unique start and end nodes
- Preprocessing log recording degenerate-base replacements and added ACGT anchor segments

## How to apply

First preprocess the input haplotype sequences: replace degenerate bases with the most frequent base of the other sequences (recording each modification), then append two randomly generated ACGT anchor segments of length k to the front and end of every sequence so the graph has a single start and end node. Choose k to exceed K = max{ki}, the maximum longest repeat segment length across all sequences — the paper sets k = 22 for the mitogenome datasets to guarantee the graph is acyclic. Build the graph with the VARI-merge branch of Cosmo (DSK k-mer input, k <= 64 supported), installing cSupB by adding its Makefile and cosmo-cSupB.cpp to VARI-merge's main directory. Validate the resulting graph before proceeding: it must be directed, acyclic, nondegenerate, and possess unique start and end nodes.

## Related tools

- **VARI-merge** (Constructs the succinct colored de Bruijn graph and color matrix from the preprocessed haplotype panel at the chosen k (k = 22)) — https://github.com/cosmo-team/cosmo/tree/VARI-merge
- **Cosmo** (Underlying succinct de Bruijn graph assembler/infrastructure on whose VARI branch VARI-merge is built) — https://github.com/cosmo-team/cosmo
- **cSupB** (cSupB program installed into the VARI-merge main directory to consume the constructed graph and color matrix for decomposition and coordinate derivation) — https://github.com/eggleader/cSupB

## Examples

```
dsk input.fa 23 -o k23; pack-edges k23; cosmo-build k23.packed; cosmo-cSupB primates_mtDNAs_kmc_list.dbg primates_mtDNAs_kmc_list.colors.sd_vector >log.txt
```

## Evaluation signals

- The constructed graph is directed, acyclic, and nondegenerate with a unique start node and unique end node
- The chosen k value exceeds the maximum longest repeat segment length K across all input sequences (k = 22 verified for the 12 mitogenomes)
- No cycles appear during the downstream postorder-like traversal; any 'finished in advance' traversal with unvisited nodes indicates the graph precondition was violated
- Every input sample resolves to a unique path in the graph (as reported for the mitogenome graph)
- The .dbg and .colors.sd_vector files load successfully into cosmo-cSupB and emit the CST/CSDI/NNI/OVI decomposition files

## Limitations

- Colored de Bruijn graph construction tools, including VARI-merge, lack mapping functionality, which limits data selection and model promotion
- No specific version of VARI-merge is stated in the article
- DSK-based construction in Cosmo supports only k <= 64; Cosmo's graph is edge-based, so the DSK k-mer length must be set to k+1 if nodes are meant to be k-mers
- Degenerate bases must be replaced prior to construction and this cannot handle degeneracy efficiently in the graph itself

## Evidence

- [results] Here, we set k = 22 for VARI-merge to construct an acyclic graph.: "Here, we set k = 22 for VARI-merge to construct an acyclic graph."
- [methods] we randomly generate two segments consisting of ACGT with a length of k and add them to the front and end of each sequence: "we randomly generate two segments consisting of ACGT with a length of k and add them to the front and end of each sequence"
- [methods] the obtained colored de Bruijn graph has the following characteristics: directed, acyclic, nondegeneracy, and unique start and end nodes: "the obtained colored de Bruijn graph has the following characteristics: directed, acyclic, nondegeneracy, and unique start and end nodes"
- [readme] VARI-cSupB is based on the colored de Bruijn graph constructed by [VARI-merge](https://github.com/cosmo-team/cosmo/tree/VARI-merge), so the installation processing is the same as the VARI-merge. We only need to add the files (Makefile and cosmo-cSupB.cpp) to VARI-merge's main directory.: "VARI-cSupB is based on the colored de Bruijn graph constructed by [VARI-merge](https://github.com/cosmo-team/cosmo/tree/VARI-merge), so the installation processing is the same as the VARI-merge."
- [readme] Currently Cosmo only supports [DSK][dsk] files with k <= 64: [evidence span withheld — anchor status: external_doc]

---
name: genome-graph-scalability-benchmarking
description: Use when you need to quantify how a graph-based pangenome pipeline scales
  with panel size, e.g. testing whether graph construction and traversal tools remain
  tractable at 5,000–20,000 genome scale.
when_to_use_negative:
- The goal is variant-detection accuracy rather than runtime/memory scaling — use
  the accuracy benchmarking workflow on Datasets 1/2 instead
- Panels contain highly dissimilar sequences or complex variant types (large indels,
  inversions, reverse tandem duplications), where the cSupB variant model degrades
  and the scalability numbers would not be meaningful
- Sequences are highly repetitive (e.g. tandem repeats), since the cutting method
  fragments sequences and inflates computational difficulty, confounding the scaling
  measurement
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: VARI-merge
  role: Constructs the colored de Bruijn graph at k=22 for each simulated panel; graph-construction
    wall-clock time and peak memory are recorded as the construction half of the benchmark
  repo: https://github.com/cosmo-team/cosmo/tree/VARI-merge
- name: cSupB
  role: C++ program run on each constructed graph to perform postorder-like traversal,
    cSupB matching, cycle handling, and hierarchical-tree reorganization; its analysis
    wall-clock time and peak memory form the analysis half of the benchmark
  repo: https://github.com/eggleader/cSupB
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-021-04149-w
    title: A tri-tuple coordinate system derived for fast and accurate analysis of
      the colored de Bruijn graph-based pangenomes
  source_content_hash: sha256:f3499e55c0995ef640dddae14d40beca2fccda368add43bad1d6f4e1d1fc84ce
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - C++
  - VARI-merge
  - cSupB
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# genome-graph-scalability-benchmarking

## Summary

Benchmark the computational scalability of a colored de Bruijn graph pangenome pipeline (VARI-merge graph construction plus cSupB analysis) on simulated panels of 5,000 and 20,000 human mitochondrial genomes, measuring wall-clock time and peak memory to characterize runtime growth with sample count.

## When to use

Use when you need to quantify how a graph-based pangenome pipeline scales with panel size, e.g. testing whether graph construction and traversal tools remain tractable at 5,000–20,000 genome scale.

## When NOT to use

- The goal is variant-detection accuracy rather than runtime/memory scaling — use the accuracy benchmarking workflow on Datasets 1/2 instead
- Panels contain highly dissimilar sequences or complex variant types (large indels, inversions, reverse tandem duplications), where the cSupB variant model degrades and the scalability numbers would not be meaningful
- Sequences are highly repetitive (e.g. tandem repeats), since the cutting method fragments sequences and inflates computational difficulty, confounding the scaling measurement

## Inputs

- Simulated human mitochondrial genome FASTA panel with 5,000 samples (Dataset 3)
- Simulated human mitochondrial genome FASTA panel with 20,000 samples (Dataset 4)
- Chosen k-mer size (k=22) validated to exceed the longest repeat segment length K

## Outputs

- Graph-construction wall-clock time and peak memory per panel (VARI-merge)
- cSupB-analysis wall-clock time and peak memory per panel
- Tabulated benchmark results (e.g. scalability_benchmark.tsv) with CST/CSDI/NNI/OVI outputs from cSupB

## How to apply

Generate or load randomly simulated human mitogenome FASTA panels of two scales (5,000 and 20,000 samples; Datasets 3 and 4). Preprocess each panel: replace degenerate bases with the most frequent base of other sequences, verify the chosen k=22 exceeds the longest repeat segment length K across all sequences, and append randomly generated ACGT anchor segments of length k to the front and end of every sequence so the graph has a single start and end node. Construct the colored de Bruijn graph with VARI-merge at k=22, recording graph-construction wall-clock time and peak memory via a resource profiler. Run the cSupB C++ program on each graph to perform postorder-like traversal, cSupB matching, cycle handling, and hierarchical-tree reorganization, recording analysis wall-clock time and peak memory. Compare measured growth to the reported trend: 7.5 h + 3 min for 5,000 samples versus 30.5 h + 8 min for 20,000 samples, i.e. roughly 4x runtime growth with a fourfold increase in sample count (approximately linear scaling).

## Related tools

- **VARI-merge** (Constructs the colored de Bruijn graph at k=22 for each simulated panel; graph-construction wall-clock time and peak memory are recorded as the construction half of the benchmark) — https://github.com/cosmo-team/cosmo/tree/VARI-merge
- **cSupB** (C++ program run on each constructed graph to perform postorder-like traversal, cSupB matching, cycle handling, and hierarchical-tree reorganization; its analysis wall-clock time and peak memory form the analysis half of the benchmark) — https://github.com/eggleader/cSupB

## Examples

```
cosmo-cSupB panel20k_kmc_list.dbg panel20k_kmc_list.colors.sd_vector >log.txt 2>&1 | /usr/bin/time -v
```

## Evaluation signals

- Both scales (5,000 and 20,000 samples) complete without failure at k=22
- Measured runtime for 20,000 samples is roughly 4x that for 5,000 samples, consistent with near-linear growth
- Reproduced timings are close to the reported 7.5 h + 3 min (5,000 samples) and 30.5 h + 8 min (20,000 samples)
- Peak memory and wall-clock time are tabulated for both graph construction (VARI-merge) and analysis (cSupB) phases separately
- cSupB emits the CST/CSDI/NNI/OVI output files at both scales, confirming the analysis completed fully

## Limitations

- Reported timings are hardware- and implementation-dependent; absolute values may differ on other machines
- VARI-merge and other colored de Bruijn graph construction tools lack mapping functionality, which limits data selection and model promotion
- The benchmark is specific to high-similarity mitochondrial-scale panels; behavior on highly repetitive sequences or very diverse panels is not captured
- Degeneracy cannot be handled efficiently by the pipeline being benchmarked

## Evidence

- [results] Reported benchmark timings: "Samples500020,000cSupB7.5 h + 3 min30.5 h + 8 min"
- [results] k parameter selection: "Here, we set k = 22 for VARI-merge to construct an acyclic graph."
- [results] Simulated panel sizes: "Dataset 4: 20,000 randomly simulated human mitochondrial genomes."
- [methods] Anchor-segment preprocessing step: "we randomly generate two segments consisting of ACGT with a length of k and add them to the front and end of each sequence"
- [readme] Installation/execution mode from README: "We only need to add the files (Makefile and cosmo-cSupB.cpp) to VARI-merge's main directory."
- [discussion] Discussion limitation: "almost all colored de Bruijn graph construction tools, including the VARI-merge used in this paper, do not have functions such as mapping"

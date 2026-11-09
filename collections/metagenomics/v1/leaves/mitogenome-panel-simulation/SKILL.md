---
name: mitogenome-panel-simulation
description: Use when you need simulated human mitogenome FASTA panels with known
  SNP/indel ground truth to benchmark graph-based variant detection (precision/recall)
  or to measure wall-clock time and memory scaling of VARI-merge + cSupB pipelines.
when_to_use_negative:
- Sequences with insufficient similarity (source number far less than variant number)
  — the variant detection model the simulations benchmark is not applicable in that
  regime
- Studies targeting large indels, inversions, or reverse tandem duplications — cSupB
  variant detection only addresses small indels (< 50 bp)
- Highly repetitive sequences, especially tandem repeats — the cutting method fragments
  them, increasing computational difficulty and error
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_0622
tools:
- name: VARI-merge
  role: Constructs the colored de Bruijn graph from each simulated panel at k = 22,
    yielding a directed, acyclic, nondegenerate graph
  repo: https://github.com/cosmo-team/cosmo/tree/VARI-merge
- name: cSupB
  role: C++ program that traverses the graph, matches colored superbubbles, handles
    cycles, and emits variant calls benchmarked against simulation ground truth
  repo: https://github.com/eggleader/cSupB
- name: Cosmo
  role: Underlying succinct de Bruijn graph toolkit (VARI branch) whose k-mer convention
    means DSK must be run with k+1 = 23 for k = 22
  repo: https://github.com/cosmo-team/cosmo
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1186/s12859-021-04149-w
    title: A tri-tuple coordinate system derived for fast and accurate analysis of
      the colored de Bruijn graph-based pangenomes
  source_content_hash: sha256:4e5422987e10f32d2e1516d99cf79aca09955cb8426eb8c73960b5a011e3fee5
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# mitogenome-panel-simulation

## Summary

Simulate human mitochondrial genome panels (from the 12-genome Dataset 1 up to 5,000- and 20,000-genome panels) to generate ground-truth benchmark data for validating cSupB variant detection accuracy and computational scalability of colored de Bruijn graph-based pangenome analysis.

## When to use

Use when you need simulated human mitogenome FASTA panels with known SNP/indel ground truth to benchmark graph-based variant detection (precision/recall) or to measure wall-clock time and memory scaling of VARI-merge + cSupB pipelines.

## When NOT to use

- Sequences with insufficient similarity (source number far less than variant number) — the variant detection model the simulations benchmark is not applicable in that regime
- Studies targeting large indels, inversions, or reverse tandem duplications — cSupB variant detection only addresses small indels (< 50 bp)
- Highly repetitive sequences, especially tandem repeats — the cutting method fragments them, increasing computational difficulty and error

## Inputs

- 12 human mitochondrial genome sequences, Dataset 1 (AP008459, AP010675, EF153784, EF153791, EF153794, EF153814, EF397559, EU007868, FJ493500, GQ895144, GU377085, NC_012920)
- Additional file 1 simulation protocol specifying SNP, deletion, and insertion counts per run
- Simulated mitogenome FASTA panels: Dataset 2 (10 samples/run), Dataset 3 (5,000 genomes), Dataset 4 (20,000 genomes)

## Outputs

- Simulated mitogenome FASTA panels with known variant ground truth (SNP, INS < 50 bp, DEL < 50 bp)
- Preprocessed panels (degenerate bases replaced, k = 22, ACGT anchor segments appended) ready for VARI-merge graph construction
- indel_precision_recall.tsv with per-dataset precision and recall comparable to Additional file 1
- scalability_benchmark.tsv with peak memory and wall-clock time for 5,000- and 20,000-sample runs

## How to apply

Start from the 12 human mitochondrial genomes of Dataset 1 (accessions AP008459 through NC_012920, including reference NC_012920) or the Additional file 1 simulation protocol, generating panels of 10 samples per run (Dataset 2), 5,000 samples (Dataset 3), or 20,000 samples (Dataset 4). Preprocess each panel by replacing degenerate bases with the most frequent base of the other sequences (recording modifications), choosing k = 22 so it exceeds the maximum longest repeat segment length K across sequences, and appending randomly generated ACGT anchor segments of length k to the front and end of every sequence so the colored de Bruijn graph has a single start and end node. Build the colored de Bruijn graph with VARI-merge, which yields a directed, acyclic, nondegenerate graph with a unique path per sequence, then run the cSupB C++ program for postorder-like traversal, cSupB matching, cycle handling, and hierarchical-tree reorganization. For accuracy benchmarking, inject known SNPs, deletions, and insertions, keep only small indels (< 50 bp) after Nested Variant Detection (NVD) and Reference Position Determination (RPD), and compute precision and recall against the simulation ground truth; for scalability benchmarking, record graph-construction and cSupB-analysis wall-clock time and peak memory at both panel sizes.

## Related tools

- **VARI-merge** (Constructs the colored de Bruijn graph from each simulated panel at k = 22, yielding a directed, acyclic, nondegenerate graph) — https://github.com/cosmo-team/cosmo/tree/VARI-merge
- **cSupB** (C++ program that traverses the graph, matches colored superbubbles, handles cycles, and emits variant calls benchmarked against simulation ground truth) — https://github.com/eggleader/cSupB
- **Cosmo** (Underlying succinct de Bruijn graph toolkit (VARI branch) whose k-mer convention means DSK must be run with k+1 = 23 for k = 22) — https://github.com/cosmo-team/cosmo

## Examples

```
dsk simulated_panel.fasta 23 && cosmo-cSupB panel.dbg panel.colors.sd_vector >log.txt
```

## Evaluation signals

- Graph built from each simulated panel must be directed, acyclic, nondegenerate, with unique start and end nodes (guaranteed by the anchor segments and k > K)
- Runs must complete at both scales and reproduce roughly 4x runtime growth from 5,000 to 20,000 samples (7.5 h + 3 min vs. 30.5 h + 8 min)
- Detected small indels (< 50 bp) must align with simulation ground truth; precision and recall per dataset should be high across parameter sets with best stability
- All degenerate-base replacements are recorded so modifications can be audited
- Variant types assigned by NVD gap values must satisfy: gap1 > 0 = INS, gap2 > 0 = DEL, gap1 = gap2 = 0 = SNP

## Limitations

- No version of VARI-merge stated; colored de Bruijn graph construction tools, including VARI-merge, lack mapping functionality, limiting data selection and model promotion
- Variant detection precision decreases when sample similarity decreases and variant types become complicated; applicable only to high-similarity samples with simple variant types
- Gene elements shorter than k may map to multiple nodes, making analysis complicated and ambiguous; degeneracy cannot be handled efficiently
- Choice of k value impacts the accuracy of the algorithm

## Evidence

- [methods] we randomly generate two segments consisting of ACGT with a length of k and add them to the front and end of each sequence: "we randomly generate two segments consisting of ACGT with a length of k and add them to the front and end of each sequence"
- [results] Dataset 3: 5000 randomly simulated human mitochondrial genomes.: "Dataset 3: 5000 randomly simulated human mitochondrial genomes."
- [results] Here, we set k = 22 for VARI-merge to construct an acyclic graph.: "Here, we set k = 22 for VARI-merge to construct an acyclic graph."
- [methods] we replaced degenerate bases with the most frequent base of other sequences and recorded the modification: "we replaced degenerate bases with the most frequent base of other sequences and recorded the modification"
- [readme] cosmo-cSupB primates_mtDNAs_kmc_list.dbg primates_mtDNAs_kmc_list.colors.sd_vector >log.txt: "cosmo-cSupB primates_mtDNAs_kmc_list.dbg primates_mtDNAs_kmc_list.colors.sd_vector >log.txt"

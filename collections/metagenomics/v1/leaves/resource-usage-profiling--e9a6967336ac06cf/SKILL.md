---
name: resource-usage-profiling
description: Use when you need to quantify how a pipeline scales with input size,
  e.g. measuring wall-clock time and peak memory for graph construction and analysis
  at 5,000 vs 20,000 genomes.
when_to_use_negative:
- Input sequences have insufficient similarity or highly repetitive/tandem-repeat
  content, since the cutting method fragments sequences and invalidates or distorts
  the cSupB analysis being profiled
- The question is variant-detection accuracy rather than computational scaling — this
  skill only measures resource use, not precision/recall
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: cSupB
  role: Downstream analysis stage whose runtime and peak memory are profiled (postorder-like
    traversal, cSupB matching, cycle handling, tree reorganization)
  repo: https://github.com/eggleader/cSupB
- name: VARI-merge
  role: Colored de Bruijn graph construction stage whose wall-clock time and peak
    memory are profiled at k=22
  repo: https://github.com/cosmo-team/cosmo/tree/VARI-merge
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-021-04149-w
    title: A tri-tuple coordinate system derived for fast and accurate analysis of
      the colored de Bruijn graph-based pangenomes
  source_content_hash: sha256:a3a41eea91678b4334f619e77c88d1df99cebcf0151f21b1c95879987c3d9be6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eggleader/cSupB
  tools:
  - C++
  - VARI-merge
  - cSupB
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# resource-usage-profiling

## Summary

Benchmark wall-clock time and peak memory of a pangenome pipeline by instrumenting graph construction and downstream analysis separately, as done for cSupB on simulated human mitogenome panels. It quantifies scalability by comparing resource use at increasing sample counts.

## When to use

Use when you need to quantify how a pipeline scales with input size, e.g. measuring wall-clock time and peak memory for graph construction and analysis at 5,000 vs 20,000 genomes.

## When NOT to use

- Input sequences have insufficient similarity or highly repetitive/tandem-repeat content, since the cutting method fragments sequences and invalidates or distorts the cSupB analysis being profiled
- The question is variant-detection accuracy rather than computational scaling — this skill only measures resource use, not precision/recall

## Inputs

- Simulated human mitogenome FASTA panels (Dataset 3: 5,000 samples; Dataset 4: 20,000 samples)
- VARI-merge colored de Bruijn graph and color matrix files (e.g. .dbg and .colors.sd_vector)
- cSupB C++ program compiled in the VARI-merge directory

## Outputs

- Per-scale table of graph-construction and cSupB-analysis wall-clock time and peak memory (scalability_benchmark.tsv)
- CST/CSDI/NNI/OVI output files from the cSupB run
- Scalability trend summary (runtime growth ratio between sample counts)

## How to apply

Run the pipeline on panels of increasing scale (here, randomly simulated human mitochondrial genome FASTA panels of 5,000 and 20,000 samples, Datasets 3 and 4). Wrap each pipeline stage with a resource profiler so graph-construction wall-clock time and peak memory (VARI-merge at k=22) are recorded separately from the analysis stage (the cSupB C++ program's postorder-like traversal, cSupB matching, cycle handling, and hierarchical-tree reorganization). Tabulate results in a TSV (scalability_benchmark.tsv) and compare the growth ratio between scales against the expected trend: here ~4x runtime growth from 5,000 to 20,000 samples, consistent with the published 7.5 h + 3 min versus 30.5 h + 8 min timings.

## Related tools

- **cSupB** (Downstream analysis stage whose runtime and peak memory are profiled (postorder-like traversal, cSupB matching, cycle handling, tree reorganization)) — https://github.com/eggleader/cSupB
- **VARI-merge** (Colored de Bruijn graph construction stage whose wall-clock time and peak memory are profiled at k=22) — https://github.com/cosmo-team/cosmo/tree/VARI-merge

## Examples

```
cosmo-cSupB mito20k_kmc_list.dbg mito20k_kmc_list.colors.sd_vector >log.txt  # run under a resource profiler (e.g. /usr/bin/time -v) to capture wall-clock time and peak memory
```

## Evaluation signals

- Runs must complete without failure at both 5,000 and 20,000 sample scales
- Measured timing for 5,000 samples should be on the order of the reported 7.5 h (graph construction) + 3 min (cSupB analysis)
- Measured timing for 20,000 samples should be on the order of the reported 30.5 h + 8 min
- Runtime growth ratio from 5k to 20k samples should be roughly 4x (near-linear scaling)
- scalability_benchmark.tsv contains peak memory and wall-clock time entries for both scales and both stages

## Limitations

- VARI-merge version is not stated, so absolute timings may not be exactly reproducible; only the ~4x growth trend is the validation target
- Reported numbers derive from simulated mitochondrial genome panels; scaling behavior on larger or less similar pangenomes may differ
- Colored de Bruijn graph construction tools lack mapping functionality, which limits data selection and complicates promoting the benchmark to other datasets

## Evidence

- [results] On simulated large-scale panels, cSupB completed graph construction plus analysis in 7.5 h + 3 min for 5,000 samples and 30.5 h + 8 min for 20,000 samples, showing runtime growth roughly linear with a fourfold increase in sample count.: "Samples500020,000cSupB7.5 h + 3 min30.5 h + 8 min"
- [results] VARI-merge is used at k=22 to construct the acyclic graph whose construction time is profiled.: "Here, we set k = 22 for VARI-merge to construct an acyclic graph."
- [results] The panels are randomly simulated human mitochondrial genomes at 5,000 and 20,000 samples.: "Dataset 3: 5000 randomly simulated human mitochondrial genomes."
- [abstract] The cSupB C++ program implements the analysis stage being timed.: "We have developed a C + + program for implementing our method"

---
name: scaled-weight-normalization-and-summing
description: Use when multiple network inference methods (e.g., 8 NI association matrices
  from correlation or direct-model tools) must be merged into one consensus co-occurrence
  network and edge weighting (not just vote counting) should drive edge selection.
when_to_use_negative:
- You have only a single network inference result — consensus aggregation requires
  multiple (e.g., w=8) NI networks to normalize and sum.
- You want purely presence/absence edge selection by vote count; the simple voting
  (SV) algorithm, not scaled-sum, is the matching choice.
- Input is raw 16S sequence data upstream of network inference — the sequence processing
  (SP), OTU assignment, and correlation/direct inference steps must run first.
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MiCoNE
  role: Python package providing the Network/NetworkGroup API and the scaled_sum consensus
    configuration used to normalize, sum, and threshold edge weights across inference
    methods
  repo: https://github.com/segrelab/MiCoNE
- name: Python
  role: Implementation language for loading interaction matrices into Network objects
    and running the scaled-sum computation
- name: MIND
  role: Online database into which the exported consensus-network JSON is uploaded
    and visualized
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1128/msystems.00961-22
    title: 'Inferring microbial co-occurrence networks from amplicon data: a systematic
      evaluation'
  source_content_hash: sha256:9e6e72bd36a878523b1ef060e23d4b4c51470a243a4d087b36fdf65b61e25174
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/segrelab/MiCoNE
  tools:
  - MIND
  - MiCoNE
  - NetworkGroup
  - Python
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# scaled-weight-normalization-and-summing

## Summary

The scaled-sum (SS) consensus algorithm from the MiCoNE pipeline: max-normalize each network inference method's interaction scores, sum the scaled edge weights across the w networks, and retain an edge only when |s_j| exceeds (w-1) x theta. It produces a more robust consensus co-occurrence network than any single inference method.

## When to use

Use when multiple network inference methods (e.g., 8 NI association matrices from correlation or direct-model tools) must be merged into one consensus co-occurrence network and edge weighting (not just vote counting) should drive edge selection.

## When NOT to use

- You have only a single network inference result — consensus aggregation requires multiple (e.g., w=8) NI networks to normalize and sum.
- You want purely presence/absence edge selection by vote count; the simple voting (SV) algorithm, not scaled-sum, is the matching choice.
- Input is raw 16S sequence data upstream of network inference — the sequence processing (SP), OTU assignment, and correlation/direct inference steps must run first.

## Inputs

- Eight filtered NI interaction matrices (tsv or csv) loadable into MiCoNE Network objects
- Eight matching p-value matrices from the bootstrap/Pvalue step
- NI consensus configuration (method='scaled_sum', theta=0.333, pvalue_filter=true, interaction_filter=true)

## Outputs

- Consensus co-occurrence network containing only edges passing the SS threshold |s_j| > (w-1) x theta
- Multigraph JSON file compatible with the MIND database, with edges annotated with per-method interaction weights

## How to apply

Load the filtered network inference (NI) association matrices — interaction matrices plus their pvalue matrices — into MiCoNE Network objects and aggregate them in a NetworkGroup. For each candidate edge j, max-normalize each network's interaction scores (S_i), then sum the scaled weights s_j across the w networks; retain edges where |s_j| > (w-1) x theta. In the manuscript's NI configuration the consensus is applied with method='scaled_sum', theta=0.333, pvalue_filter=true, and interaction_filter=true. Assemble only qualifying edges into a multigraph annotated with per-method interaction weights, and export via network_group.write to a JSON file compatible with the MIND database. The rationale, established on mock and synthetic benchmark data sets, is that summing scaled weights exploits agreement in interaction strength across methods rather than mere presence/absence voting (the alternative simple voting, SV, keeps an edge when its vote count M_j >= floor(theta x w)).

## Related tools

- **MiCoNE** (Python package providing the Network/NetworkGroup API and the scaled_sum consensus configuration used to normalize, sum, and threshold edge weights across inference methods) — https://github.com/segrelab/MiCoNE
- **Python** (Implementation language for loading interaction matrices into Network objects and running the scaled-sum computation)
- **MIND** (Online database into which the exported consensus-network JSON is uploaded and visualized)

## Examples

```
micone init -w ni -o consensus_run  # configure NI consensus (method='scaled_sum', theta=0.333), then: bash run.sh
```

## Evaluation signals

- Output JSON parses and is MIND-compatible (uploadable/visualizable in the MIND database).
- Every retained edge satisfies |s_j| > (w-1) x theta, i.e., the sum of max-normalized weights over the w=8 networks exceeds the threshold for theta=0.333.
- No edge in the consensus network fails the SS (or, when co-applied, SV: M_j >= floor(theta x w)) threshold.
- Each retained edge carries interaction weights from the contributing inference methods, traceable to the input association matrices.
- network_group.write produces a file that exists and is non-empty after the pipeline run.

## Limitations

- Threshold behavior depends on the number of networks w: the (w-1) x theta cutoff assumes many contributing methods and degrades for small w.
- Different NI tools lead to vastly different co-occurrence networks, so scaled-sum results are sensitive to which inference methods are included in the NetworkGroup.
- The full MiCoNE pipeline requires installing per-tool conda environments (e.g., micone-sparcc, micone-spieceasi) and manually downloading the data directory from Zenodo; missing environments abort the run.

## Evidence

- [other] In the simple voting (SV) consensus algorithm, an edge is selected to be present in the final consensus network when the number of network inference methods (networks) in which that edge appears exceeds a threshold.: [evidence span withheld — anchor status: not_found]
- [methods] If you have an interaction matrix stored in a tsv or csv format, you can load it into a Network object.: "If you have an interaction matrix stored in a ``tsv`` or ``csv`` format, you can load it into a ``Network`` object"
- [methods] This JSON file is compatible with the MIND database and can be uploaded and visualized directly on the website.: "This ``JSON`` file is compatible with the ``MIND`` database and can be uploaded and visualized directly on the website"
- [readme] MiCoNE is a Python package for the exploration of the effects of various possible tools used during the 16S data processing workflow on the inferred co-occurrence networks.: "MiCoNE` is a Python package for the exploration of the effects of various possible tools used during the 16S data processing workflow on the inferred co-occurrence networks"
- [abstract] We further determine the tools and parameters that generate robust co-occurrence networks and develop consensus network algorithms based on benchmarks with mock and synthetic data sets.: "We further determine the tools and parameters that generate robust co-occurrence networks and develop consensus network algorithms based on benchmarks with mock and synthetic data sets"

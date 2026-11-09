---
name: cooccurrence-association-matrix-aggregation
description: Use when several network inference tools (e.g., sparcc, spieceasi, flashweave,
  harmonies, spring, mldm, propr) have each produced interaction and pvalue matrices
  and one robust consensus network is needed.
when_to_use_negative:
- Input is raw 16S reads or an abundance/feature table — this skill operates only
  on already-inferred association matrices, downstream of the network inference step
- Only one inference method's network is available — the SV/SS thresholds (floor(theta
  x w), (w-1) x theta) are defined across w networks and are meaningless for a single
  network
- The goal is comparing or validating networks against mock/synthetic ground truth
  rather than merging multiple inferred networks into a consensus
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: MiCoNE
  role: Python library providing Network objects (Network.load_data), the NetworkGroup
    aggregator, and the SV/scaled_sum consensus algorithms with threshold theta; exports
    via network_group.write
  repo: https://github.com/segrelab/MiCoNE
- name: MIND
  role: Online database to which the exported consensus-network JSON can be uploaded
    and visualized
- name: Python
  role: Implementation language for scripting the aggregation (Network/NetworkGroup
    API, threshold computations, validation)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1128/msystems.00961-22
    title: 'Inferring microbial co-occurrence networks from amplicon data: a systematic
      evaluation'
  source_content_hash: sha256:aa340cb75775b5eeff8d006192b6da213daebfef5ad9532b2cea634f53c1e514
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/segrelab/MiCoNE
  tools:
  - MIND
  - MiCoNE
  - NetworkGroup
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# cooccurrence-association-matrix-aggregation

## Summary

Aggregates interaction and pvalue matrices produced by multiple network inference methods (w=8 in the MiCoNE pipeline) into a single consensus co-occurrence network using MiCoNE's simple voting (SV) or scaled-sum (SS) algorithms with threshold theta.

## When to use

Use when several network inference tools (e.g., sparcc, spieceasi, flashweave, harmonies, spring, mldm, propr) have each produced interaction and pvalue matrices and one robust consensus network is needed.

## When NOT to use

- Input is raw 16S reads or an abundance/feature table — this skill operates only on already-inferred association matrices, downstream of the network inference step
- Only one inference method's network is available — the SV/SS thresholds (floor(theta x w), (w-1) x theta) are defined across w networks and are meaningless for a single network
- The goal is comparing or validating networks against mock/synthetic ground truth rather than merging multiple inferred networks into a consensus

## Inputs

- Filtered NI interaction matrices (tsv/csv) from each network inference method
- Matching pvalue matrices (tsv/csv) from each network inference method
- NI configuration table specifying consensus method, theta, pvalue_filter, interaction_filter

## Outputs

- Consensus co-occurrence network as a MIND-compatible JSON multigraph with edges annotated with per-method interaction weights

## How to apply

Load each of the w=8 filtered NI association matrices (interaction matrix plus matching pvalue matrix, in tsv or csv) into a MiCoNE Network object with Network.load_data and aggregate them in a NetworkGroup. For simple voting (SV), compute the sign-based voting count M_j per candidate edge across the w networks and retain edges where M_j >= floor(theta x w); for scaled-sum (SS), max-normalize each network's interaction scores S_i, sum the scaled weights s_j across networks, and retain edges where |s_j| > (w-1) x theta. Apply the MiCoNE consensus configuration from the NI config table (default method='scaled_sum', parameter theta=0.333, pvalue_filter=true, interaction_filter=true). Assemble only qualifying edges into a multigraph whose edges are annotated with the per-method interaction weights, then export with network_group.write to a JSON file compatible with the MIND database.

## Related tools

- **MiCoNE** (Python library providing Network objects (Network.load_data), the NetworkGroup aggregator, and the SV/scaled_sum consensus algorithms with threshold theta; exports via network_group.write) — https://github.com/segrelab/MiCoNE
- **MIND** (Online database to which the exported consensus-network JSON can be uploaded and visualized)
- **Python** (Implementation language for scripting the aggregation (Network/NetworkGroup API, threshold computations, validation))

## Examples

```
from micone import Network, NetworkGroup; ng = NetworkGroup([Network.load_data(i, p, format='tsv') for i, p in ni_pairs]); consensus = ng.get_consensus(method='scaled_sum', theta=0.333)
```

## Evaluation signals

- Output JSON file exists, parses, and is a MIND-compatible multigraph
- Every retained edge appears in at least floor(theta x w) of the input networks (SV check) and satisfies |s_j| > (w-1) x theta (SS check)
- Every edge in the consensus network carries interaction weights from the contributing inference methods
- Consensus parameters match the NI config table: method='scaled_sum', theta=0.333, pvalue_filter=true, interaction_filter=true
- No edge appears in the consensus that is absent from all input networks

## Limitations

- Consensus quality depends on theta; the default 0.333 was tuned on mock and synthetic benchmarks and may not transfer to other datasets
- Because different analysis methods and tools lead to vastly different co-occurrence networks, aggregating poorly filtered matrices can yield unstable consensus edges
- The MiCoNE pipeline data directory (nf_micone/data) must be manually downloaded from Zenodo record 7051556 before runs

## Evidence

- [abstract] We also develop a consensus network algorithm that helps generate more robust co-occurrence networks based on benchmark synthetic data sets: "We also develop a consensus network algorithm that helps generate more robust co-occurrence networks based on benchmark synthetic data sets"
- [methods] Network.load_data from tsv/csv: "If you have an interaction matrix stored in a ``tsv`` or ``csv`` format, you can load it into a ``Network`` object"
- [methods] MIND-compatible JSON export: "This ``JSON`` file is compatible with the ``MIND`` database and can be uploaded and visualized directly on the website"
- [intro] Tool choice variance motivates consensus: "The different methods and tools can lead to vastly different inferences of community compositions and co-occurrence networks"
- [readme] MiCoNE scope: "``MiCoNE`` is a Python package for the exploration of the effects of various possible tools used during the 16S data processing workflow on the inferred co-occurrence networks"

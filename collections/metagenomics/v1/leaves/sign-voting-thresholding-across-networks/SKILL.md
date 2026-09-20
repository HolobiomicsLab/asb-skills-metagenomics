---
name: sign-voting-thresholding-across-networks
description: Use when you have co-occurrence networks inferred by multiple network
  inference methods (e.g. w=8 NI association plus p-value matrices) and need to decide
  which edges belong in a single robust consensus network.
when_to_use_negative:
- You have only a single network inference method's output — voting/scaled-sum thresholds
  across w networks are undefined for w=1
- You are still at the sequence processing stage (demultiplexing, trimming, OTU assignment,
  chimera checking) and no association matrices exist yet
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: MiCoNE
  role: Python package providing Network, NetworkGroup, and the consensus network
    algorithm (simple voting / scaled-sum) used to merge per-method networks
  repo: https://github.com/segrelab/MiCoNE
- name: Python
  role: Language in which the interaction matrices are loaded into Network objects
    and the SV/SS thresholds are computed
  repo: https://github.com/segrelab/MiCoNE
- name: MIND
  role: Online database for which the exported consensus network JSON is formatted
    so it can be uploaded and visualized
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1128/msystems.00961-22
    title: 'Inferring microbial co-occurrence networks from amplicon data: a systematic
      evaluation'
  source_content_hash: sha256:99116f78c0f312961b0f630046cdd3ec62e3bf03dc574733adc30ac1e08a8e70
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/segrelab/MiCoNE
  tools:
  - MIND
  - MiCoNE
  - NetworkGroup
  - Python
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# sign-voting-thresholding-across-networks

## Summary

Consensus network construction in MiCoNE: an edge is retained in the final co-occurrence network only if the number of network inference methods reporting it exceeds a threshold (simple voting, SV), or if its sum of max-normalized interaction scores exceeds (w-1) x theta (scaled-sum, SS).

## When to use

Use when you have co-occurrence networks inferred by multiple network inference methods (e.g. w=8 NI association plus p-value matrices) and need to decide which edges belong in a single robust consensus network.

## When NOT to use

- You have only a single network inference method's output — voting/scaled-sum thresholds across w networks are undefined for w=1
- You are still at the sequence processing stage (demultiplexing, trimming, OTU assignment, chimera checking) and no association matrices exist yet

## Inputs

- Interaction (association) matrices from multiple network inference methods (tsv/csv, loadable into MiCoNE Network objects)
- Corresponding p-value matrices from each inference method
- Consensus configuration (method, threshold theta, pvalue_filter, interaction_filter) from the NI config table

## Outputs

- Consensus co-occurrence network as a multigraph whose edges carry per-method interaction weights
- MIND-compatible JSON file of the consensus network produced by 'network_group.write'

## How to apply

Load each method's interaction matrix (tsv/csv) and its p-value matrix into MiCoNE Network objects via 'Network.load_data' and aggregate them in a 'NetworkGroup'. For simple voting (SV), compute for each candidate edge j the sign-based voting count M_j across the w networks and retain edges where M_j >= floor(theta x w). For scaled-sum (SS), max-normalize each network's interaction scores S_i, sum the scaled weights s_j across networks, and retain edges where |s_j| > (w-1) x theta. Apply the MiCoNE consensus configuration (method='scaled_sum', theta=0.333, pvalue_filter=true, interaction_filter=true), assemble only qualifying edges into a multigraph annotated with per-method interaction weights, and export with 'network_group.write' to a JSON file compatible with the MIND database. The consensus algorithms were developed and validated on benchmarks with mock and synthetic data sets.

## Related tools

- **MiCoNE** (Python package providing Network, NetworkGroup, and the consensus network algorithm (simple voting / scaled-sum) used to merge per-method networks) — https://github.com/segrelab/MiCoNE
- **Python** (Language in which the interaction matrices are loaded into Network objects and the SV/SS thresholds are computed) — https://github.com/segrelab/MiCoNE
- **MIND** (Online database for which the exported consensus network JSON is formatted so it can be uploaded and visualized)

## Examples

```
from micone import Network, NetworkGroup; nets = [Network.load_data(f, p) for f, p in zip(interaction_files, pvalue_files)]; ng = NetworkGroup(nets); consensus = ng.consensus(method='scaled_sum', theta=0.333); consensus.write('consensus.')
```

## Evaluation signals

- Output JSON exists and parses as a MIND-compatible multigraph
- Every retained edge appears in more than floor(theta x w) networks (SV) or has |s_j| > (w-1) x theta (SS); no edge below threshold is present
- Every retained edge carries interaction weights from the contributing inference methods
- With theta=0.333 and w=8 networks, SS retains only edges with |s_j| > 7 x 0.333 (~2.33)

## Limitations

- Threshold theta controls stringency; with theta=0.333 SV requires an edge in at least 3 of 8 networks, which may exclude edges reported by only one sensitive method
- SS requires max-normalization of scores across methods, so methods with incomparable or missing score scales can bias the summed weights
- The MiCoNE pipeline requires conda environments per tool (e.g. micone-sparcc, micone-spieceasi) and the data directory must be downloaded manually from Zenodo

## Evidence

- [full_text] SV retains edges voted by majority of networks: [evidence span withheld — anchor status: not_found]
- [abstract] develop consensus network algorithms based on benchmarks: "We further determine the tools and parameters that generate robust co-occurrence networks and develop consensus network algorithms based on benchmarks with mock and synthetic data sets"
- [methods] load interaction matrix into Network object: "If you have an interaction matrix stored in a ``tsv`` or ``csv`` format, you can load it into a ``Network`` object"
- [methods] JSON compatible with MIND: "This ``JSON`` file is compatible with the ``MIND`` database and can be uploaded and visualized directly on the website"
- [readme] MiCoNE explores tool effects on networks: "``MiCoNE`` is a Python package for the exploration of the effects of various possible tools used during the 16S data processing workflow on the inferred co-occurrence networks."

---
name: multigraph-edge-weight-annotation
description: Use when multiple network inference association matrices (interaction
  + pvalue matrices) have been filtered and consensus edges must be assembled with
  per-method weights for export to the MIND database.
when_to_use_negative:
- Input is raw 16S sequencing reads or unfiltered interaction matrices — the consensus
  assembly presupposes eight already-filtered NI association matrices
- Only a single inference network is available — SV/SS consensus thresholds across
  w=8 networks cannot be computed
- The goal is the 16S processing steps upstream of network inference (demultiplexing,
  trimming, OTU assignment) rather than consensus network assembly
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: MiCoNE
  role: Python package providing Network objects, NetworkGroup aggregation, SV/SS
    consensus algorithms, and JSON export compatible with the MIND database
  repo: https://github.com/segrelab/MiCoNE
- name: Python
  role: Language in which interaction matrices are loaded into Network objects and
    consensus edges are assembled
- name: MIND
  role: Online database that the exported JSON multigraph is formatted to be uploaded
    to and visualized on
- name: NetworkGroup
  role: MiCoNE container aggregating the eight inference networks; its consensus method
    and write function produce the annotated multigraph
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1128/msystems.00961-22
    title: 'Inferring microbial co-occurrence networks from amplicon data: a systematic
      evaluation'
  source_content_hash: sha256:0b68df7044e381d8f91ff687d569fc85749e1bdb501fed60d699c6b140c8ece6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/segrelab/MiCoNE
  tools:
  - MIND
  - MiCoNE
  - NetworkGroup
  - Python
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# multigraph-edge-weight-annotation

## Summary

Assembles edges surviving the MiCoNE simple voting (SV) and scaled-sum (SS) consensus thresholds into a multigraph whose edges carry the per-network-inference-method interaction weights, exported as a MIND-compatible JSON file.

## When to use

Use when multiple network inference association matrices (interaction + pvalue matrices) have been filtered and consensus edges must be assembled with per-method weights for export to the MIND database.

## When NOT to use

- Input is raw 16S sequencing reads or unfiltered interaction matrices — the consensus assembly presupposes eight already-filtered NI association matrices
- Only a single inference network is available — SV/SS consensus thresholds across w=8 networks cannot be computed
- The goal is the 16S processing steps upstream of network inference (demultiplexing, trimming, OTU assignment) rather than consensus network assembly

## Inputs

- Eight filtered NI association matrices (interaction matrices plus pvalue matrices) in tsv or csv format
- MiCoNE NI config table specifying consensus method ('scaled_sum'), theta (0.333), pvalue_filter=true, interaction_filter=true

## Outputs

- Consensus multigraph containing only edges passing the SV and SS thresholds, with per-method interaction weights on each edge
- JSON file compatible with the MIND database, written via 'network_group.write'

## How to apply

Load the eight filtered NI association matrices (interaction matrices plus pvalue matrices) into MiCoNE Network objects via 'Network.load_data' and aggregate them in a NetworkGroup. Apply the MiCoNE consensus configuration (method='scaled_sum', parameter theta=0.333, pvalue_filter=true, interaction_filter=true as set in the NI config table): for simple voting, retain an edge when the count of networks in which it appears exceeds the threshold floor(theta x w); for scaled-sum, max-normalize each network's interaction scores, sum scaled weights s_j, and retain edges where |s_j| > (w-1) x theta. Assemble only qualifying edges into a multigraph, annotating each edge with the per-method interaction weights from the contributing inference methods, then export with 'network_group.write' to a JSON file compatible with the MIND database.

## Related tools

- **MiCoNE** (Python package providing Network objects, NetworkGroup aggregation, SV/SS consensus algorithms, and JSON export compatible with the MIND database) — https://github.com/segrelab/MiCoNE
- **Python** (Language in which interaction matrices are loaded into Network objects and consensus edges are assembled)
- **MIND** (Online database that the exported JSON multigraph is formatted to be uploaded to and visualized on)
- **NetworkGroup** (MiCoNE container aggregating the eight inference networks; its consensus method and write function produce the annotated multigraph)

## Examples

```
from micone import Network, NetworkGroup; network_group = NetworkGroup([Network.load_data(f) for f in ni_files]); network_group.get_consensus('scaled_sum', 0.333, True, True); network_group.write('consensus_network.')
```

## Evaluation signals

- Output JSON file exists and parses as a MIND-compatible multigraph
- Every edge in the output satisfies the SV threshold (voting count M_j >= floor(theta x w)) and/or the SS threshold (|s_j| > (w-1) x theta)
- Every retained edge carries interaction weights annotated from each contributing network inference method
- Edge set equals the intersection implied by the MiCoNE consensus config (method='scaled_sum', theta=0.333, pvalue_filter=true, interaction_filter=true)

## Limitations

- Threshold behavior depends on the number of networks w=8; the SV threshold uses floor(theta x w) so small changes in theta shift inclusion counts discretely
- MiCoNE requires the mamba package manager, otherwise 'micone init' will not work, and the pipeline data directory must be downloaded manually from Zenodo record 7051556
- Known conflicts: a preinstalled julia version may conflict with the one in the micone-flashweave environment

## Evidence

- [results] In the simple voting (SV) consensus algorithm, an edge is selected to be present in the final consensus network when the number of network inference methods (networks) in which that edge appears exceeds a threshold.: [evidence span withheld — anchor status: not_found]
- [methods] If you have an interaction matrix stored in a ``tsv`` or ``csv`` format, you can load it into a ``Network`` object: "If you have an interaction matrix stored in a ``tsv`` or ``csv`` format, you can load it into a ``Network`` object"
- [methods] This ``JSON`` file is compatible with the ``MIND`` database and can be uploaded and visualized directly on the website: "This ``JSON`` file is compatible with the ``MIND`` database and can be uploaded and visualized directly on the website"
- [abstract] We also develop a consensus network algorithm that helps generate more robust co-occurrence networks based on benchmark synthetic data sets: "We also develop a consensus network algorithm that helps generate more robust co-occurrence networks based on benchmark synthetic data sets"

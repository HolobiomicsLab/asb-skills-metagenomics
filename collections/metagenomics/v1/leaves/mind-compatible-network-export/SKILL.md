---
name: mind-compatible-network-export
description: Use when one or more network inference association matrices (interaction
  plus pvalue matrices) have been computed and qualifying consensus edges must be
  exported as a MIND-compatible JSON multigraph annotated with per-method interaction
  weights.
when_to_use_negative:
- The input is still raw 16S sequencing data or an unfiltered single-method network
  rather than a set of filtered NI association matrices ready for consensus aggregation
- The target database or downstream tool requires a format other than the MiCoNE JSON
  export (e.g., GraphML or edge list) rather than MIND-compatible JSON
edam_operation: http://edamontology.org/operation_3436
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: MiCoNE
  role: Python package providing Network/NetworkGroup objects, the SV and SS consensus
    algorithms, and the 'network_group.write' JSON export compatible with MIND
  repo: https://github.com/segrelab/MiCoNE
- name: Python
  role: Language in which the Network objects are loaded from tsv/csv interaction
    matrices and the consensus multigraph is assembled
- name: MIND
  role: Microbial Interaction Network Database that ingests and visualizes the exported
    JSON consensus networks
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1128/msystems.00961-22
    title: 'Inferring microbial co-occurrence networks from amplicon data: a systematic
      evaluation'
  source_content_hash: sha256:963d04a15a5cb22875851aacef8c425df4546a6d960b9bc5fb3a0d1cafe7c724
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/segrelab/MiCoNE
  tools:
  - MIND
  - MiCoNE
  - NetworkGroup
  - Python
  edam_operation: http://edamontology.org/operation_3436
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# mind-compatible-network--export

## Summary

Assemble edges retained by the MiCoNE consensus algorithms (simple voting and scaled-sum with threshold theta) into a multigraph and export it as a JSON file compatible with the MIND (Microbial Interaction Network Database), so consensus co-occurrence networks can be uploaded and visualized directly on MIND.

## When to use

Use when one or more network inference association matrices (interaction plus pvalue matrices) have been computed and qualifying consensus edges must be exported as a MIND-compatible JSON multigraph annotated with per-method interaction weights.

## When NOT to use

- The input is still raw 16S sequencing data or an unfiltered single-method network rather than a set of filtered NI association matrices ready for consensus aggregation
- The target database or downstream tool requires a format other than the MiCoNE JSON export (e.g., GraphML or edge list) rather than MIND-compatible JSON

## Inputs

- Eight filtered network-inference association matrices (interaction score matrices plus corresponding pvalue matrices, tsv/csv format)
- NI consensus configuration parameters (method='scaled_sum', theta=0.333, pvalue_filter=true, interaction_filter=true)

## Outputs

- Consensus co-occurrence network as a MIND-compatible JSON multigraph with edges annotated with per-method interaction weights

## How to apply

Load the eight filtered NI association matrices (interaction matrices plus pvalue matrices) into MiCoNE Network objects using 'Network.load_data' (which accepts tsv/csv interaction matrices or edge lists) and aggregate them in a NetworkGroup. Implement simple voting (SV): for each candidate edge j compute the sign-based vote count M_j across the w=8 inference networks and retain edges where M_j >= floor(theta x w). Implement scaled-sum (SS): max-normalize each network's interaction scores S_i, sum the scaled weights s_j across networks, and retain edges where |s_j| > (w-1) x theta; apply the MiCoNE consensus configuration (method='scaled_sum', theta=0.333, pvalue_filter=true, interaction_filter=true from the NI config). Assemble only qualifying edges into a multigraph whose edges carry the per-method interaction weights, then use 'network_group.write' to export to a JSON file that is compatible with the MIND database.

## Related tools

- **MiCoNE** (Python package providing Network/NetworkGroup objects, the SV and SS consensus algorithms, and the 'network_group.write' JSON export compatible with MIND) — https://github.com/segrelab/MiCoNE
- **Python** (Language in which the Network objects are loaded from tsv/csv interaction matrices and the consensus multigraph is assembled)
- **MIND** (Microbial Interaction Network Database that ingests and visualizes the exported JSON consensus networks)

## Examples

```
micone init -w ni -o ./consensus_pipeline && bash run.sh
```

## Evaluation signals

- The output JSON file exists and parses without errors
- The parsed JSON is a MIND-compatible multigraph that can be uploaded and visualized directly on the MIND website
- The edge set contains only edges passing the SV threshold (M_j >= floor(theta x w)) and the SS threshold (|s_j| > (w-1) x theta)
- Every retained edge carries interaction weights from each contributing inference method
- The consensus configuration used matches the NI config table (method='scaled_sum', theta=0.333, pvalue_filter=true, interaction_filter=true)

## Limitations

- The JSON export targets MIND compatibility; other consumers may require format conversion
- Consensus algorithm quality depends on the benchmarks with mock and synthetic data sets used to calibrate them; poorly filtered input matrices will propagate into the consensus network
- The data directory used by the MiCoNE pipeline must be manually downloaded from Zenodo (record 7051556), and running the full pipeline requires installing the various conda environments first

## Evidence

- [methods] Exporting Network object into JSON file: "This ``JSON`` file is compatible with the ``MIND`` database and can be uploaded and visualized directly on the website"
- [methods] Loading association matrices into Network objects: "If you have an interaction matrix stored in a ``tsv`` or ``csv`` format, you can load it into a ``Network`` object"
- [abstract] Consensus algorithms benchmarked: "We further determine the tools and parameters that generate robust co-occurrence networks and develop consensus network algorithms based on benchmarks with mock and synthetic data sets"
- [readme] MiCoNE purpose: "`MiCoNE` is a Python package for the exploration of the effects of various possible tools used during the 16S data processing workflow"

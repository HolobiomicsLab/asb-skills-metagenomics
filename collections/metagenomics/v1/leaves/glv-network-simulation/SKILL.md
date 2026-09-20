---
name: glv-network-simulation
description: Use when a ground-truth interaction network is required to score co-occurrence
  inference — e.g. simulating synthetic abundance tables with seqtime gLV or Klemm-Eguiluz
  dynamics before running MiCoNE network inference or consensus methods.
when_to_use_negative:
- The input is real amplicon data with no ground-truth interaction network — precision
  cannot be computed against a simulator reference
- The goal is 16S sequence processing (demultiplexing, chimera checking, OTU assignment)
  rather than benchmarking network inference against synthetic truth
tools:
- name: MiCoNE
  role: Runs the Network Inference (NI) step and scaled_sum consensus on the synthetic
    tables and scores precision/sensitivity against the simulator ground-truth network
  repo: https://github.com/segrelab/MiCoNE
- name: seqtime
  role: Simulates synthetic abundance tables with generalized Lotka-Volterra (gLV)
    and Klemm-Eguiluz dynamics, providing ground-truth interaction networks
- name: spieceasi
  role: Ships the amgut1.filt example data used to fit NorTA synthetic benchmark tables
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1128/msystems.00961-22
    title: 'Inferring microbial co-occurrence networks from amplicon data: a systematic
      evaluation'
  source_content_hash: sha256:246c4a19cac88258d7cc55c1ff45b580e28b6803cad35a968cfe77fd03b1bb82
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/segrelab/MiCoNE
  tools:
  - MiCoNE
  - Python
  - seqtime
  - spieceasi
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# glv-network-simulation

## Summary

Generates synthetic microbial abundance tables with known ground-truth interaction networks using the seqtime simulator (generalized Lotka-Volterra and Klemm-Eguiluz dynamics) so that co-occurrence network inference methods and consensus algorithms can be benchmarked with quantitative precision and sensitivity metrics.

## When to use

Use when a ground-truth interaction network is required to score co-occurrence inference — e.g. simulating synthetic abundance tables with seqtime gLV or Klemm-Eguiluz dynamics before running MiCoNE network inference or consensus methods.

## When NOT to use

- The input is real amplicon data with no ground-truth interaction network — precision cannot be computed against a simulator reference
- The goal is 16S sequence processing (demultiplexing, chimera checking, OTU assignment) rather than benchmarking network inference against synthetic truth

## Inputs

- seqtime simulator configuration with gLV and Klemm-Eguiluz dynamics parameters
- amgut1.filt example abundance table (SPIEC-EASI) used to fit NorTA synthetic data
- Ground-truth interaction networks exported by each simulator

## Outputs

- Synthetic abundance tables from NorTA and seqtime simulators
- Per-simulator ground-truth interaction networks
- precision_sensitivity_table.csv with precision and sensitivity per simulator and theta value

## How to apply

Use the seqtime simulator with generalized Lotka-Volterra (gLV) and Klemm-Eguiluz dynamics to generate synthetic abundance tables, recording each simulator's ground-truth interaction network as the reference standard. Fit NorTA to the amgut1.filt example data shipped with SPIEC-EASI as a complementary synthetic benchmark. Feed each synthetic table into the MiCoNE Network Inference (NI) step with default settings (1000 bootstrap resamples, interaction_threshold = 0.3, pvalue_threshold = 0.05), then build consensus networks with the scaled_sum method. Sweep the consensus parameter theta (e.g. theta = 1.000 and theta = 0.333) and compute precision and sensitivity of individual inference methods and the consensus against the simulator's ground-truth network, tabulating results per simulator per theta. The rationale is that only synthetic data with known true interactions permits exact precision/sensitivity scoring of inferred associations.

## Related tools

- **MiCoNE** (Runs the Network Inference (NI) step and scaled_sum consensus on the synthetic tables and scores precision/sensitivity against the simulator ground-truth network) — https://github.com/segrelab/MiCoNE
- **seqtime** (Simulates synthetic abundance tables with generalized Lotka-Volterra (gLV) and Klemm-Eguiluz dynamics, providing ground-truth interaction networks)
- **spieceasi** (Ships the amgut1.filt example data used to fit NorTA synthetic benchmark tables)

## Examples

```
micone init -w ni -o pipeline_dir
```

## Evaluation signals

- Consensus (scaled_sum) precision equals 1.000 at theta = 1.000 on both NorTA and seqtime synthetic data
- At the default theta = 0.333, consensus precision matches 0.956 (NorTA) and 0.688 (seqtime) as reported in Fig. 5
- No significant sensitivity loss at theta = 0.333 relative to individual inference methods
- precision_sensitivity_table.csv contains one precision/sensitivity row per simulator (NorTA, seqtime) and per theta value (1.000, 0.333)

## Limitations

- Consensus precision on seqtime synthetic data is substantially lower (0.688 at theta = 0.333) than on NorTA data (0.956), so results are simulator-dependent and do not transfer trivially across dynamical models
- Synthetic gLV/Klemm-Eguiluz dynamics are an idealized ground truth; conclusions may not hold for real communities where true interactions are unknown
- MiCoNE requires the mamba package manager, otherwise micone init will not work

## Evidence

- [abstract] Consensus benchmarks on synthetic data: "We further determine the tools and parameters that generate robust co-occurrence networks and develop consensus network algorithms based on benchmarks with mock and synthetic data sets"
- [methods] Synthetic data provenance: "Example data can be found `here <https://github.com/segrelab/MiCoNE-synthetic-data/tree/main/data>`__"
- [readme] Synthetic data generation scripts: [evidence span withheld — anchor status: external_doc]
- [readme] Pipeline scope for scoring: "It supports the conversion of raw 16S sequence data into co-occurrence networks."

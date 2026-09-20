---
name: synthetic-microbiome-abundance-simulation
description: 'Use when evaluating network inference or consensus algorithms against
  known ground truth: synthetic abundance tables with recorded interaction networks
  are needed to compute precision and sensitivity of inferred co-occurrence networks.'
when_to_use_negative:
- Input is real 16S amplicon data without a known interaction network — precision
  against ground truth cannot be computed, so benchmark-based validation is impossible
- The goal is evaluating upstream sequence processing steps (demultiplexing, trimming,
  OTU assignment) rather than network inference or consensus performance
- Mock community data with a defined species composition is the intended benchmark
  — the article uses both mock and synthetic data sets, and this skill covers only
  the synthetic simulator branch
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_3308
tools:
- name: MiCoNE
  role: Python package providing the Network Inference (NI) step run on the synthetic
    tables and the 'Create consensus' task (method 'scaled_sum') used to evaluate
    consensus recovery precision/sensitivity
  repo: https://github.com/segrelab/MiCoNE
- name: spieceasi
  role: Source of the amgut1.filt example data that NorTA is fit to when generating
    synthetic abundance tables
- name: Python
  role: Runtime for the simulation and evaluation scripts, including loading interaction
    matrices into MiCoNE Network objects from tsv/csv
- name: MiCoNE-synthetic-data
  role: Repository holding the synthetic data and generation scripts used in the MiCoNE
    manuscript
  repo: https://github.com/segrelab/MiCoNE-synthetic-data/tree/main/data
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1128/msystems.00961-22
    title: 'Inferring microbial co-occurrence networks from amplicon data: a systematic
      evaluation'
  source_content_hash: sha256:57fc910b0c2137625f5684f4a6631e9492686355c8e597e99968ac4d4373b836
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/segrelab/MiCoNE
  tools:
  - MiCoNE
  - MiCoNE-synthetic-data
  - Python
  - spieceasi
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# synthetic-microbiome-abundance-simulation

## Summary

Generate synthetic microbial abundance tables with known ground-truth interaction networks (NorTA fit to amgut1.filt SPIEC-EASI data; seqtime with generalized Lotka-Volterra and Klemm-Eguiluz dynamics) to benchmark co-occurrence network inference and consensus methods such as MiCoNE's scaled-sum.

## When to use

Use when evaluating network inference or consensus algorithms against known ground truth: synthetic abundance tables with recorded interaction networks are needed to compute precision and sensitivity of inferred co-occurrence networks.

## When NOT to use

- Input is real 16S amplicon data without a known interaction network — precision against ground truth cannot be computed, so benchmark-based validation is impossible
- The goal is evaluating upstream sequence processing steps (demultiplexing, trimming, OTU assignment) rather than network inference or consensus performance
- Mock community data with a defined species composition is the intended benchmark — the article uses both mock and synthetic data sets, and this skill covers only the synthetic simulator branch

## Inputs

- amgut1.filt example abundance data shipped with SPIEC-EASI (used as NorTA fit target)
- Simulator configuration for seqtime with gLV and Klemm-Eguiluz dynamics
- MiCoNE pipeline configuration (e.g. ni workflow template from micone init)

## Outputs

- Synthetic abundance tables per simulator (NorTA, seqtime)
- Ground-truth interaction network files per simulator
- Inferred networks from MiCoNE NI (1000 bootstraps, interaction_threshold=0.3, pvalue_threshold=0.05)
- Consensus networks (scaled_sum at swept theta values)
- precision_sensitivity_table.csv with precision/sensitivity per simulator and theta

## How to apply

Generate synthetic abundance tables using two complementary simulators: NorTA fit to the amgut1.filt example data shipped with SPIEC-EASI, and the seqtime simulator with generalized Lotka-Volterra (gLV) and Klemm-Eguiluz dynamics, recording each simulator's ground-truth interaction network. Run the MiCoNE Network Inference (NI) step on each synthetic table with the default pipeline settings: 1000 bootstrap resamples, interaction_threshold = 0.3, and pvalue_threshold = 0.05. Build the consensus network with the MiCoNE 'Create consensus' task using method 'scaled_sum', parameter (theta) 0.333, pvalue_filter = true, interaction_filter = true, and id_field 'taxid'. Sweep theta (e.g. 1.000 and 0.333) and compute precision and sensitivity of each individual inference method and of the consensus network against the simulator ground-truth networks, tabulating values per simulator and per theta into precision_sensitivity_table.csv. Rationale: synthetic data with known interactions is the only way to quantify recovery precision, since real amplicon-derived co-occurrence networks lack a verifiable truth.

## Related tools

- **MiCoNE** (Python package providing the Network Inference (NI) step run on the synthetic tables and the 'Create consensus' task (method 'scaled_sum') used to evaluate consensus recovery precision/sensitivity) — https://github.com/segrelab/MiCoNE
- **spieceasi** (Source of the amgut1.filt example data that NorTA is fit to when generating synthetic abundance tables)
- **Python** (Runtime for the simulation and evaluation scripts, including loading interaction matrices into MiCoNE Network objects from tsv/csv)
- **MiCoNE-synthetic-data** (Repository holding the synthetic data and generation scripts used in the MiCoNE manuscript) — https://github.com/segrelab/MiCoNE-synthetic-data/tree/main/data

## Examples

```
micone init -w ni -o synthetic_runs
```

## Evaluation signals

- Consensus (scaled_sum) precision matches Fig. 5 values: precision = 1.000 at theta = 1.000 on both NorTA and seqtime synthetic data
- At the default theta = 0.333, consensus precision equals 0.956 with NorTA and 0.688 with seqtime
- At theta = 0.333 there is no significant loss of sensitivity relative to individual inference methods
- precision_sensitivity_table.csv contains precision/sensitivity rows for each individual method and the consensus per simulator (NorTA, seqtime) and per theta value
- Each generated synthetic abundance table is accompanied by a recorded ground-truth interaction network from its simulator

## Limitations

- Consensus precision at the default theta = 0.333 is markedly lower on seqtime synthetic data (0.688) than on NorTA (0.956), so results are simulator-dependent
- The MiCoNE pipeline data directory (nf_micone/data) must be manually downloaded from Zenodo record 7051556; it is not bundled
- No changelog is available for the segrelab/MiCoNE repository, so version drift in simulator or inference behavior may go undocumented

## Evidence

- [abstract] We further determine the tools and parameters that generate robust co-occurrence networks and develop consensus network algorithms based on benchmarks with mock and synthetic data sets: "We further determine the tools and parameters that generate robust co-occurrence networks and develop consensus network algorithms based on benchmarks with mock and synthetic data sets"
- [readme] Synthetic data and generation scripts for the MiCoNE manuscript: [evidence span withheld — anchor status: external_doc]
- [methods] Example data can be found here (MiCoNE-synthetic-data repository data folder): "Example data can be found `here <https://github.com/segrelab/MiCoNE-synthetic-data/tree/main/data>`__"
- [readme] MiCoNE is a Python package for exploring effects of tools used during the 16S data processing workflow on inferred co-occurrence networks: "`MiCoNE` is a Python package for the exploration of the effects of various possible tools used during the 16S data processing workflow on the inferred co-occurrence networks."

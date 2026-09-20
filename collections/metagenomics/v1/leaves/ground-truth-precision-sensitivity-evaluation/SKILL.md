---
name: ground-truth-precision-sensitivity-evaluation
description: Use when you have synthetic abundance tables with known ground-truth
  interaction networks and need to quantify how precisely an inference or consensus
  method (e.g., MiCoNE scaled_sum) recovers true associations.
when_to_use_negative:
- Input is real amplicon data with no ground-truth interaction network — precision/sensitivity
  against a reference cannot be computed
- You are evaluating sequence processing steps (demultiplexing, trimming, OTU assignment)
  rather than network inference/consensus quality
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: MiCoNE
  role: Runs the Network Inference step (bootstrap, correlation/direct methods) and
    the 'Create consensus' task with scaled_sum and sweepable theta parameter; provides
    Network objects for comparison
  repo: https://github.com/segrelab/MiCoNE
- name: Python
  role: Loads interaction matrices into Network objects (tsv/csv) and computes precision/sensitivity
    against ground-truth networks
- name: spieceasi
  role: Source of the amgut1.filt example data to which NorTA is fit for synthetic
    abundance generation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1128/msystems.00961-22
    title: 'Inferring microbial co-occurrence networks from amplicon data: a systematic
      evaluation'
  source_content_hash: sha256:ada8d6abb5e52f43ba79b519b89066d37bdc6e63ff023aa90369eb3742391541
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/segrelab/MiCoNE
  tools:
  - MiCoNE
  - Python
  - spieceasi
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# ground-truth-precision-sensitivity-evaluation

## Summary

Evaluates co-occurrence network inference and consensus methods by sweeping the consensus parameter theta and computing precision and sensitivity of individual and consensus networks against synthetic simulator ground truths (NorTA and seqtime gLV/Klemm-Eguiluz), as done in the MiCoNE manuscript (Fig. 5).

## When to use

Use when you have synthetic abundance tables with known ground-truth interaction networks and need to quantify how precisely an inference or consensus method (e.g., MiCoNE scaled_sum) recovers true associations.

## When NOT to use

- Input is real amplicon data with no ground-truth interaction network — precision/sensitivity against a reference cannot be computed
- You are evaluating sequence processing steps (demultiplexing, trimming, OTU assignment) rather than network inference/consensus quality

## Inputs

- Synthetic abundance tables from NorTA (fit to amgut1.filt SPIEC-EASI example data)
- Synthetic abundance tables from seqtime with gLV and Klemm-Eguiluz dynamics
- Ground-truth interaction networks recorded from each simulator
- Inferred interaction matrices from MiCoNE bootstrap runs (1000 resamples, interaction_threshold = 0.3, pvalue_threshold = 0.05)

## Outputs

- precision_sensitivity_table.csv with precision and sensitivity per simulator (NorTA, seqtime) and per theta value
- Consensus networks built with scaled_sum at each swept theta value
- Comparison against reported Fig. 5 precision values (1.000 at theta=1.000; 0.956 NorTA / 0.688 seqtime at theta=0.333)

## How to apply

Generate synthetic abundance tables with NorTA (fit to amgut1.filt from SPIEC-EASI) and with the seqtime simulator using generalized Lotka-Volterra (gLV) and Klemm-Eguiluz dynamics, recording each simulator's ground-truth interaction network. Run the MiCoNE Network Inference (NI) step with default settings: 1000 bootstrap resamples, interaction_threshold = 0.3, pvalue_threshold = 0.05. Build the consensus network with the 'Create consensus' task using method 'scaled_sum', parameter 0.333, pvalue_filter = true, interaction_filter = true, and id_field 'taxid'. Sweep theta (e.g., 1.000 and 0.333) and compute precision and sensitivity of each method against the simulator ground truths, then tabulate results (e.g., into precision_sensitivity_table.csv). Rationale: consensus algorithms should improve robustness over individual methods; the paper found precision = 1.000 at theta = 1.000 on both simulators and 0.956 (NorTA) / 0.688 (seqtime) at the default theta = 0.333 without significant sensitivity loss.

## Related tools

- **MiCoNE** (Runs the Network Inference step (bootstrap, correlation/direct methods) and the 'Create consensus' task with scaled_sum and sweepable theta parameter; provides Network objects for comparison) — https://github.com/segrelab/MiCoNE
- **Python** (Loads interaction matrices into Network objects (tsv/csv) and computes precision/sensitivity against ground-truth networks)
- **spieceasi** (Source of the amgut1.filt example data to which NorTA is fit for synthetic abundance generation)

## Examples

```
micone init -w ni -o pipeline_dir  # then set consensus method='scaled_sum', parameter=0.333, pvalue_filter=true, interaction_filter=true, id_field='taxid'; bash run.sh
```

## Evaluation signals

- Consensus scaled_sum precision equals 1.000 at theta = 1.000 on both NorTA and seqtime synthetic data (matches Fig. 5)
- At default theta = 0.333, precision is 0.956 with NorTA and 0.688 with seqtime, with no significant sensitivity loss
- precision_sensitivity_table.csv contains rows per simulator (NorTA, seqtime) and per theta value with both precision and sensitivity columns
- Ground-truth interaction networks are successfully recorded from each simulator before inference (NorTA and seqtime gLV/Klemm-Eguiluz)
- NI step parameters match the defaults: 1000 bootstrap resamples, interaction_threshold = 0.3, pvalue_threshold = 0.05

## Limitations

- Precision of consensus varies strongly by simulator (0.956 NorTA vs 0.688 seqtime at theta=0.333), so results from one synthetic benchmark do not transfer automatically
- Evaluation is only as informative as the synthetic ground truth; the paper notes different pipeline choices lead to vastly different inferred networks, making cross-study comparison difficult
- The 'Create consensus' visualization of pipeline results in the paper repo was noted as 'coming soon' in the MiCoNE README

## Evidence

- [abstract] We further determine the tools and parameters that generate robust co-occurrence networks and develop consensus network algorithms based on benchmarks with mock and synthetic data sets: "We further determine the tools and parameters that generate robust co-occurrence networks and develop consensus network algorithms based on benchmarks with mock and synthetic data sets"
- [full_text] Generate synthetic abundance tables using NorTA fit to the amgut1.filt example data shipped with SPIEC-EASI: [evidence span withheld — anchor status: not_found]
- [intro] The different methods and tools can lead to vastly different inferences of community compositions and co-occurrence networks: "The different methods and tools can lead to vastly different inferences of community compositions and co-occurrence networks (52, 53), making it difficult to reliably compare networks across"
- [readme] MiCoNE is a Python package for exploring effects of tools used during the 16S data processing workflow: "`MiCoNE` is a Python package for the exploration of the effects of various possible tools used during the 16S data processing workflow on the inferred co-occurrence networks."
- [methods] If you have an interaction matrix stored in a tsv or csv format, you can load it into a Network object: "If you have an interaction matrix stored in a ``tsv`` or ``csv`` format, you can load it into a ``Network`` object"

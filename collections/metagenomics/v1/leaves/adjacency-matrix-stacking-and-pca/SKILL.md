---
name: adjacency-matrix-stacking-and-pca
description: Use when you have interaction/adjacency matrices from many MiCoNE tool-parameter
  combinations across pipeline steps and need to quantify which step (denoising-clustering,
  chimera-checking, taxonomy-assignment, OTU-processing, network-inference) drives
  the total variation among the inferred networks.
when_to_use_negative:
- The inputs are raw 16S reads or a single abundance table — this skill operates on
  inferred interaction matrices, not upstream sequence processing or OTU assignment
  itself
- Only one network is available — variance across tool/parameter combinations cannot
  be decomposed from a single adjacency matrix
- The goal is consensus network generation or selecting tools with optimal agreement
  with mock and synthetic benchmark data sets, rather than quantifying per-step variance
  contributions
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MiCoNE
  role: Loads interaction matrices into Network objects and exports NetworkGroup results
    (pvalue_filter=True, interaction_filter=False) that feed the flattened adjacency
    vectors
  repo: https://github.com/segrelab/MiCoNE
- name: Nextflow
  role: Executes the micone ni workflow template (bash run.sh) over all tool/parameter
    combinations defined in nextflow.config
- name: Python
  role: Hosts the scikit-learn PCA and ANOVA regression that decompose variance across
    the stacked adjacency vectors
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1128/msystems.00961-22
    title: 'Inferring microbial co-occurrence networks from amplicon data: a systematic
      evaluation'
  source_content_hash: sha256:8773de1926711b3193916f11d3533b810083b95ca63e90c046398b9161e58484
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/segrelab/MiCoNE
  tools:
  - MiCoNE
  - Nextflow
  - Python
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# Adjacency matrix stacking and PCA

## Summary

A method for attributing variation among co-occurrence networks to 16S pipeline steps: flatten each MiCoNE-inferred network's adjacency matrix into a vector, stack all tool/parameter combination vectors, run scikit-learn PCA, and regress principal components on the categorical step indicators (DC, CC, TA, OP, NI) by ANOVA. It reveals that taxonomy assignment — the choice of 16S reference database — contributes 65.4% of network variation and OTU processing 26.8%.

## When to use

Use when you have interaction/adjacency matrices from many MiCoNE tool-parameter combinations across pipeline steps and need to quantify which step (denoising-clustering, chimera-checking, taxonomy-assignment, OTU-processing, network-inference) drives the total variation among the inferred networks.

## When NOT to use

- The inputs are raw 16S reads or a single abundance table — this skill operates on inferred interaction matrices, not upstream sequence processing or OTU assignment itself
- Only one network is available — variance across tool/parameter combinations cannot be decomposed from a single adjacency matrix
- The goal is consensus network generation or selecting tools with optimal agreement with mock and synthetic benchmark data sets, rather than quantifying per-step variance contributions

## Inputs

- Inferred co-occurrence network interaction matrices (tsv/csv) from every tool/parameter combination of the MiCoNE ni workflow
- Network metadata files accepted by Network.load_data (interaction_file, meta_file, cmeta_file, obsmeta_file)
- Categorical step labels (DC, CC, TA, OP, NI) for each network combination

## Outputs

- Principal components of network variation from PCA on the stacked flattened adjacency vectors
- Per-step variance fractions from ANOVA regression (e.g., TA 65.4%, OP 26.8%)
- Step ranking identifying which pipeline stage dominates network variation

## How to apply

Run the MiCoNE network-inference workflow (micone init -w ni; bash run.sh) enumerating all tool/parameter combinations for the DC, CC, TA, OP, and NI steps, keeping the default thresholds interaction_threshold = 0.3, pvalue_threshold = 0.05, and 1000 Resample bootstraps. Load each inferred interaction matrix into a micone Network object (Network.load_data with interaction_file, meta_file, cmeta_file, obsmeta_file) and export via NetworkGroup.write('network.', pvalue_filter=True, interaction_filter=False) so only p-value-filtered interactions enter the analysis. Flatten each network's adjacency matrix into a vector and stack all combination vectors into a matrix, then run PCA (scikit-learn) to obtain principal components of network variation. Regress each principal component on the categorical step indicators (DC, CC, TA, OP, NI) and compute per-step variance contributions by ANOVA. The expected outcome mirrors the paper's ranking: taxonomy assignment — specifically the choice of 16S reference database — contributes the most (65.4%), followed by the OTU processing step (26.8%).

## Related tools

- **MiCoNE** (Loads interaction matrices into Network objects and exports NetworkGroup results (pvalue_filter=True, interaction_filter=False) that feed the flattened adjacency vectors) — https://github.com/segrelab/MiCoNE
- **Nextflow** (Executes the micone ni workflow template (bash run.sh) over all tool/parameter combinations defined in nextflow.config)
- **Python** (Hosts the scikit-learn PCA and ANOVA regression that decompose variance across the stacked adjacency vectors)

## Examples

```
from sklearn.decomposition import PCA; components = PCA().fit_transform(X)  # X = stacked flattened adjacency vectors
```

## Evaluation signals

- All flattened adjacency vectors have the same length (same taxa) before stacking into the PCA matrix
- Per-step variance fractions are non-negative and sum to approximately 100%
- The taxonomy assignment step emerges as the dominant variance contributor (~65.4%), with OTU processing second (~26.8%)
- Exported networks use NetworkGroup.write with pvalue_filter=True and interaction_filter=False, matching the default pvalue_threshold = 0.05 and interaction_threshold = 0.3
- ANOVA regression of each principal component on the DC/CC/TA/OP/NI step indicators reproduces the paper's step ranking

## Limitations

- The decomposition depends on the configured thresholds (interaction_threshold = 0.3, pvalue_threshold = 0.05, 1000 Resample bootstraps); changing them changes the per-step fractions
- The minimal micone Python library does not provide the functionality to execute pipelines — the network combinations must be generated via the nextflow templates (micone init -w ni)
- The data directory (nf_micone/data) must be manually downloaded from Zenodo record 7051556 before running the pipeline

## Evidence

- [abstract] we map how different choices of algorithms and parameters affect the co-occurrence network and identify the steps that contribute substantially to the variance: "we map how different choices of algorithms and parameters affect the co-occurrence network and identify the steps that contribute substantially to the variance"
- [methods] interaction matrix loading: "If you have an interaction matrix stored in a ``tsv`` or ``csv`` format, you can load it into a ``Network`` object"
- [readme] combinations of tools per step: "offering parallelized, fast and reproducible runs executed for different combinations of tools for each step of the data processing workflow."
- [other] flatten, stack, PCA method: [evidence span withheld — anchor status: not_found]
- [other] TA dominates variance: [evidence span withheld — anchor status: not_found]
- [other] validation invariant: [evidence span withheld — anchor status: not_found]

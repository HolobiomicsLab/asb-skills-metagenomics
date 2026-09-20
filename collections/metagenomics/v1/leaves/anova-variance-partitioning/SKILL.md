---
name: anova-variance-partitioning
description: Use when many co-occurrence networks have been generated from all tool/parameter
  combinations of a multi-step 16S pipeline and you must quantify which pipeline step
  contributes most to network-to-network variation.
when_to_use_negative:
- Input is raw 16S sequencing reads or a single abundance table — the networks must
  already be inferred; use the MiCoNE processing pipeline (micone init / run.sh) first
- The goal is to compare individual tools' agreement with mock/synthetic ground truth
  rather than to attribute network variance to pipeline steps
- Networks were produced with heterogeneous thresholds (not the paper's interaction_threshold
  = 0.3 and pvalue_threshold = 0.05), so step effects would be confounded with threshold
  effects
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MiCoNE
  role: Python package and nextflow pipeline used to enumerate tool/parameter combinations,
    infer the co-occurrence networks, and load interaction matrices into Network/NetworkGroup
    objects before variance analysis
  repo: https://github.com/segrelab/MiCoNE
- name: Nextflow
  role: Workflow engine executing the micone network-inference template across all
    pipeline-step combinations (run.sh)
  repo: https://github.com/segrelab/MiCoNE
- name: Python
  role: Hosts the PCA (scikit-learn) and regression/ANOVA analysis of the flattened
    network adjacency vectors
- name: qiime2
  role: Provides supported file formats and taxonomy-assignment tools (e.g. Naive
    Bayes classifier gg_13_8_99_515_806) whose variation drives the dominant TA variance
    component
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1128/msystems.00961-22
    title: 'Inferring microbial co-occurrence networks from amplicon data: a systematic
      evaluation'
  source_content_hash: sha256:e7e93c58afe181b8835f219305dd4632a99b51a03e5481169946533a6170974b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/segrelab/MiCoNE
  tools:
  - MiCoNE
  - Nextflow
  - Python
  - qiime2
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# anova-variance-partitioning

## Summary

Attributes the total variation among inferred microbial co-occurrence networks to the MiCoNE pipeline steps (DC, CC, TA, OP, NI) by combining PCA of flattened adjacency matrices with linear regression and ANOVA, revealing which pipeline step (e.g. taxonomy assignment, 65.4%) dominates network variance.

## When to use

Use when many co-occurrence networks have been generated from all tool/parameter combinations of a multi-step 16S pipeline and you must quantify which pipeline step contributes most to network-to-network variation.

## When NOT to use

- Input is raw 16S sequencing reads or a single abundance table — the networks must already be inferred; use the MiCoNE processing pipeline (micone init / run.sh) first
- The goal is to compare individual tools' agreement with mock/synthetic ground truth rather than to attribute network variance to pipeline steps
- Networks were produced with heterogeneous thresholds (not the paper's interaction_threshold = 0.3 and pvalue_threshold = 0.05), so step effects would be confounded with threshold effects

## Inputs

- Inferred co-occurrence network interaction matrices for every MiCoNE tool/parameter combination (pvalue-filtered, from NetworkGroup.write)
- Categorical step labels (DC, CC, TA, OP, NI) indicating which step was varied for each network

## Outputs

- Principal components of network variation (scikit-learn PCA over stacked flattened adjacency vectors)
- Per-step variance contribution fractions (paper result: taxonomy assignment 65.4%, OTU processing 26.8%)
- Ranking of pipeline steps by influence on the final co-occurrence network

## How to apply

Enumerate all tool/parameter selections for the denoising-clustering, chimera-checking, taxonomy-assignment, OTU-processing, and network-inference steps in MiCoNE's nextflow.config, then run the network-inference workflow (bash run.sh) for every combination, keeping the paper's default thresholds: interaction_threshold = 0.3, pvalue_threshold = 0.05, and 1000 bootstraps for the Resample step. Load each inferred interaction matrix into a micone Network object (Network.load_data with interaction_file, meta_file, cmeta_file, obsmeta_file) and export via NetworkGroup.write('network.', pvalue_filter=True, interaction_filter=False). Flatten each network's adjacency matrix into a vector, stack all combination vectors into a matrix, and run PCA (scikit-learn) to obtain principal components of network variation. Regress each principal component on the categorical step indicators (DC, CC, TA, OP, NI) and compute per-step variance contributions by ANOVA. The rationale, per the paper, is that different tools at a step perturb the network; the step whose indicator explains the most PC variance is the step whose algorithmic choice most affects the final co-occurrence network.

## Related tools

- **MiCoNE** (Python package and nextflow pipeline used to enumerate tool/parameter combinations, infer the co-occurrence networks, and load interaction matrices into Network/NetworkGroup objects before variance analysis) — https://github.com/segrelab/MiCoNE
- **Nextflow** (Workflow engine executing the micone network-inference template across all pipeline-step combinations (run.sh)) — https://github.com/segrelab/MiCoNE
- **Python** (Hosts the PCA (scikit-learn) and regression/ANOVA analysis of the flattened network adjacency vectors)
- **qiime2** (Provides supported file formats and taxonomy-assignment tools (e.g. Naive Bayes classifier gg_13_8_99_515_806) whose variation drives the dominant TA variance component)

## Examples

```
micone init -w ni -o pipeline_dir && bash run.sh
```

## Evaluation signals

- Per-step variance fractions are non-negative and sum to approximately 100%
- The taxonomy-assignment step emerges as the dominant variance contributor (~65.4%), matching the paper's reported step ranking
- The OTU-processing step is the second-largest contributor (~26.8%)
- Each inferred network loads into a micone Network object and exports via NetworkGroup.write with pvalue_filter=True without errors
- Number of stacked adjacency vectors equals the number of enumerated tool/parameter combinations in nextflow.config

## Limitations

- Variance attribution reflects the paper's default thresholds (interaction_threshold = 0.3, pvalue_threshold = 0.05, 1000 Resample bootstraps); other thresholds may shift step rankings
- ANOVA on categorical step indicators captures main effects of steps; interactions between step choices are not decomposed
- The data directory (nf_micone/data) must be manually downloaded from Zenodo record 7051556 before running the pipeline
- Conclusions are tied to the 16S amplicon pipeline steps (DC, CC, TA, OP, NI) evaluated in the MiCoNE framework

## Evidence

- [abstract] we map how different choices of algorithms and parameters affect the co-occurrence network and identify the steps that contribute substantially to the variance: "we map how different choices of algorithms and parameters affect the co-occurrence network and identify the steps that contribute substantially to the variance"
- [methods] micone init command: [evidence span withheld — anchor status: not_found]
- [methods] Bootstrap Resample bootstraps 1000: "| Bootstrap   | Resample                     | bootstraps            | 1000         |"
- [methods] If you have an interaction matrix stored in a tsv or csv format, you can load it into a Network object: "If you have an interaction matrix stored in a ``tsv`` or ``csv`` format, you can load it into a ``Network`` object"
- [readme] run.sh usage: "activate the ``micone`` environment and run the ``run.sh`` script that was copied to the directory"

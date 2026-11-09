---
name: co-occurrence-network-inference
description: Use when starting from a genus-level OTU/counts table (otu_table.tsv
  with obs_metadata.tsv) and the goal is to infer interaction networks and quantify
  how much inferred edges/nodes agree across different NI methods.
when_to_use_negative:
- The input is raw 16S reads and denoising/clustering has not been performed — use
  the full or ta_op_ni workflow templates instead of 'ni'
- The analysis goal is comparing effects of upstream pipeline steps (DC, CC, TA, OP)
  rather than comparing NI methods on a fixed counts table
- Only a single inference method is desired with no cross-method comparison, in which
  case the ensemble bootstrap and intersection machinery is unnecessary
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: MiCoNE
  role: Python package and Nextflow pipeline that orchestrates the NI workflow, bootstrapping,
    p-value computation, network construction, and genus-level grouping
  repo: https://github.com/segrelab/MiCoNE
- name: Nextflow
  role: Executes the micone pipeline template; templates initialized via 'micone init'
    and run via 'bash run.sh'
- name: SparCC
  role: Correlation-based NI method, run with iterations 50
- name: propr
  role: Correlation-based NI method run under defaults
- name: SpiecEasi
  role: Direct (graphical model) NI method, run with method 'mb', nreps 50, nlambda
    20, lambda_min_ratio 1e-2
- name: FlashWeave
  role: Direct NI method, run with sensitive true and fdr_correction true
- name: COZINE
  role: Direct NI method, run with lambda_min_ratio 0.1
- name: HARMONIES
  role: Direct NI method, run with iterations 10000 and sparsity_cutoff 0.5
- name: SPRING
  role: Direct NI method, run with nlambda 20 and lambda_min_ratio 0.01
- name: mLDM
  role: Direct NI method, run with Z_mean 1 and max_iteration 1500
- name: qiime2
  role: Provides supported input formats for the upstream files feeding the NI workflow
provenance:
  source_task_ids:
  - task_005
  - task_008
  source_papers:
  - doi: 10.1128/msystems.00961-22
    title: 'Inferring microbial co-occurrence networks from amplicon data: a systematic
      evaluation'
  source_content_hash: sha256:bf959c848f3d668a6e7d84dee7763d71544b8d91f26492d4ad0ed38f0972d4f5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/segrelab/MiCoNE
  tools:
  - COZINE
  - FlashWeave
  - HARMONIES
  - MiCoNE
  - Nextflow
  - Python
  - SPRING
  - SparCC
  - SpiecEasi
  - mLDM
  - propr
  - qiime2
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# co-occurrence-network-inference

## Summary

Infer microbial co-occurrence networks from genus-level abundance tables by running multiple network inference (NI) tools under their defaults within the MiCoNE Nextflow pipeline, converting bootstrapped results into filtered networks at interaction_threshold 0.3 and pvalue_threshold 0.05 for downstream cross-method comparison.

## When to use

Use when starting from a genus-level OTU/counts table (otu_table.tsv with obs_metadata.tsv) and the goal is to infer interaction networks and quantify how much inferred edges/nodes agree across different NI methods.

## When NOT to use

- The input is raw 16S reads and denoising/clustering has not been performed — use the full or ta_op_ni workflow templates instead of 'ni'
- The analysis goal is comparing effects of upstream pipeline steps (DC, CC, TA, OP) rather than comparing NI methods on a fixed counts table
- Only a single inference method is desired with no cross-method comparison, in which case the ensemble bootstrap and intersection machinery is unnecessary

## Inputs

- otu_table.tsv (genus-level counts table from FMT data, MiCoNE Zenodo record 7051556 data.zip)
- obs_metadata.tsv with Kingdom-to-Species taxonomy columns
- sample_metadata.tsv
- nextflow.config enumerating NI tool selections

## Outputs

- Eight network files (JSON/edge lists) with 'source', 'target', 'weight' edge fields, one per NI method
- Bootstrapped interaction matrices from 1000 Resample iterations with computed p-values
- Filtered genus-level networks (pvalue_filter=True, interaction_filter=False) for UpSet intersection analysis

## How to apply

Initialize the NI workflow template with 'micone init -w ni -o <pipeline_dir>' after installing conda environments via 'micone install'. Run eight inference tools under their defaults — correlation methods sparcc (iterations 50) and propr; direct methods spieceasi (method 'mb', nreps 50, nlambda 20, lambda_min_ratio 1e-2), flashweave (sensitive true, fdr_correction true), cozine (lambda_min_ratio 0.1), harmonies (iterations 10000, sparsity_cutoff 0.5), spring (nlambda 20, lambda_min_ratio 0.01), and mldm (Z_mean 1, max_iteration 1500) — each preceded by 1000 Resample bootstraps and Pvalue computation. Convert each bootstrapped result into a network via the pvalue-based Make network step with pvalue_threshold 0.05 and interaction_threshold 0.3. Group nodes at the Genus taxonomic level (tax_levels ['Phylum', 'Class', 'Order', 'Family', 'Genus', 'Species']) and write each filtered network with network_group.write('network.', pvalue_filter=True, interaction_filter=False). The rationale is that a single inference method can yield vastly different networks, so running the full tool ensemble with shared thresholds enables systematic comparison of method-driven rewiring.

## Related tools

- **MiCoNE** (Python package and Nextflow pipeline that orchestrates the NI workflow, bootstrapping, p-value computation, network construction, and genus-level grouping) — https://github.com/segrelab/MiCoNE
- **Nextflow** (Executes the micone pipeline template; templates initialized via 'micone init' and run via 'bash run.sh')
- **SparCC** (Correlation-based NI method, run with iterations 50)
- **propr** (Correlation-based NI method run under defaults)
- **SpiecEasi** (Direct (graphical model) NI method, run with method 'mb', nreps 50, nlambda 20, lambda_min_ratio 1e-2)
- **FlashWeave** (Direct NI method, run with sensitive true and fdr_correction true)
- **COZINE** (Direct NI method, run with lambda_min_ratio 0.1)
- **HARMONIES** (Direct NI method, run with iterations 10000 and sparsity_cutoff 0.5)
- **SPRING** (Direct NI method, run with nlambda 20 and lambda_min_ratio 0.01)
- **mLDM** (Direct NI method, run with Z_mean 1 and max_iteration 1500)
- **qiime2** (Provides supported input formats for the upstream files feeding the NI workflow)

## Examples

```
micone install && micone init -w ni -o pipeline_dir && bash run.sh
```

## Evaluation signals

- All eight network JSON files exist and each contains 'source', 'target', 'weight' edge fields
- Node and edge intersection counts across the eight methods qualitatively match the Fig. 4B/C overlap pattern of the paper
- In the reference run, only 8 of 202 total unique edges are shared by all methods while 33 of 68 unique nodes are shared, with no method producing a unique node — substantial rewiring should be reproduced qualitatively
- 'micone validate-results' completes without errors on the pipeline output directory
- Per-method networks were built with the same thresholds (pvalue_threshold 0.05, interaction_threshold 0.3) after 1000 bootstraps, confirmed in the run config

## Limitations

- MiCoNE requires the mamba package manager; otherwise 'micone init' will not work
- A preinstalled julia version may conflict with the version downloaded by the micone-flashweave environment
- The data directory (nf_micone/data) must be manually downloaded from Zenodo record 7051556
- Direct installation via anaconda cloud was not yet available; the conda env must be created from env.yml

## Evidence

- [methods] MiCoNE is a Python package for the exploration of the effects of various possible tools used during the 16S data processing workflow on the inferred co-occurrence networks.: "``MiCoNE`` is a Python package for the exploration of the effects of various possible tools used during the 16S data processing workflow"
- [methods] To set up the nextflow workflow template for the desired workflow, you can use the micone init command: "To set up the ``nextflow`` workflow template for the desired workflow, you can use the ``micone init`` command"
- [intro] The different methods and tools can lead to vastly different inferences of community compositions and co-occurrence networks, making it difficult to reliably compare networks across: "The different methods and tools can lead to vastly different inferences of community compositions and co-occurrence networks (52, 53), making it difficult to reliably compare networks across"
- [methods] Before execution of the MiCoNE pipeline we need to install the environments: micone install: "Before execution of the ``MiCoNE`` pipeline we need to install the environments: micone install"
- [readme] MiCoNE requires the mamba package manager, otherwise micone init will not work.: "> 1. MiCoNE requires the `mamba` package manager, otherwise `micone init` will not work."
- [readme] If you have a version of julia that is preinstalled, make sure that it does not conflict with the version downloaded by the micone-flashweave environment: "1. If you have a version of `julia` that is preinstalled, make sure that it does not conflict with the version downloaded by the `micone-flashweave` environment"

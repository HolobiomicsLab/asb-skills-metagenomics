---
name: cross-method-network-comparison
description: Use when several co-occurrence networks (e.g., SparCC, propr, SpiecEasi,
  FlashWeave, COZINE, HARMONIES, SPRING, mLDM) have been inferred from the same abundance
  table and you need to quantify shared vs. unique nodes/edges to reveal method-driven
  rewiring.
when_to_use_negative:
- You have only one inferred network per dataset — intersection analysis needs at
  least two NI methods' edge lists to compare.
- The inputs are raw 16S reads rather than filtered, genus-level networks; this skill
  assumes the bootstrap/Make-network/Group steps have already been run.
- You need to benchmark network accuracy against a ground-truth (mock/synthetic) community
  rather than assess cross-method consistency alone.
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: MiCoNE
  role: Pipeline framework and Python Network objects used to run the eight NI tools
    and compute node/edge intersections (UpSet analysis)
  repo: https://github.com/segrelab/MiCoNE
- name: SparCC
  role: Correlation-based network inference method (iterations 50) whose output edge
    list is one comparison input
- name: propr
  role: Correlation-based network inference method whose output edge list is one comparison
    input
- name: SpiecEasi
  role: Direct network inference method (method 'mb', nreps 50, nlambda 20, lambda_min_ratio
    1e-2) whose output edge list is one comparison input
- name: FlashWeave
  role: Direct network inference method (sensitive true, fdr_correction true) whose
    output edge list is one comparison input
- name: COZINE
  role: Direct network inference method (lambda_min_ratio 0.1) whose output edge list
    is one comparison input
- name: HARMONIES
  role: Direct network inference method (iterations 10000, sparsity_cutoff 0.5) whose
    output edge list is one comparison input
- name: SPRING
  role: Direct network inference method (nlambda 20, lambda_min_ratio 0.01) whose
    output edge list is one comparison input
- name: mLDM
  role: Direct network inference method (Z_mean 1, max_iteration 1500) whose output
    edge list is one comparison input
- name: Nextflow
  role: Executes the micone 'ni' workflow template that generates the eight networks
    being compared
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1128/msystems.00961-22
    title: 'Inferring microbial co-occurrence networks from amplicon data: a systematic
      evaluation'
  source_content_hash: sha256:eae24e60a7efb46afb27abf6b485447df47fa927292fa0f95d2a454094dae615
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
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6818181818181818
n_claims: 22
---

# cross-method-network-comparison

## Summary

Quantifies how consistent co-occurrence networks inferred by multiple network inference (NI) methods are, by computing UpSet-style node and edge intersections across the eight methods run in the MiCoNE pipeline on the same genus-level counts table.

## When to use

Use when several co-occurrence networks (e.g., SparCC, propr, SpiecEasi, FlashWeave, COZINE, HARMONIES, SPRING, mLDM) have been inferred from the same abundance table and you need to quantify shared vs. unique nodes/edges to reveal method-driven rewiring.

## When NOT to use

- You have only one inferred network per dataset — intersection analysis needs at least two NI methods' edge lists to compare.
- The inputs are raw 16S reads rather than filtered, genus-level networks; this skill assumes the bootstrap/Make-network/Group steps have already been run.
- You need to benchmark network accuracy against a ground-truth (mock/synthetic) community rather than assess cross-method consistency alone.

## Inputs

- otu_table.tsv (genus-level OTU/counts table from FMT runs)
- obs_metadata.tsv with Kingdom-to-Species taxonomy columns
- sample_metadata.tsv
- Eight network edge lists (network JSON files with 'source', 'target', 'weight' fields) produced by the MiCoNE NI pipeline

## Outputs

- UpSet node intersection counts across the eight NI methods (e.g., 33 of 68 unique nodes shared by all methods)
- UpSet edge intersection counts across the eight NI methods (e.g., 8 of 202 total unique edges common to all methods)
- Fig. 4B/C-style overlap report qualifying shared vs. unique nodes/edges and method-driven rewiring

## How to apply

Obtain the OTU tables and metadata from the MiCoNE Zenodo deposit (record 7051556 data.zip), then run the eight NI tools under MiCoNE defaults — sparcc (iterations 50), propr, spieceasi (method 'mb', nreps 50, nlambda 20, lambda_min_ratio 1e-2), flashweave (sensitive true, fdr_correction true), cozine (lambda_min_ratio 0.1), harmonies (iterations 10000, sparsity_cutoff 0.5), spring (nlambda 20, lambda_min_ratio 0.01), and mldm (Z_mean 1, max_iteration 1500) — each preceded by 1000 Resample bootstraps and Pvalue computation. Convert each bootstrapped interaction into a network with the pvalue-based Make network step using pvalue_threshold 0.05 and interaction_threshold 0.3, then group nodes at the genus taxonomic level (tax_levels ['Phylum','Class','Order','Family','Genus','Species']) and write each filtered network with network_group.write('network.', pvalue_filter=True, interaction_filter=False). Load the eight edge lists into MiCoNE Network objects and compute node and edge intersections across methods (UpSet analysis) to reproduce the overlap counts reported in Fig. 4B/C of the paper. The rationale is that only shared/unique element counts reveal substantial rewiring that per-method network statistics hide.

## Related tools

- **MiCoNE** (Pipeline framework and Python Network objects used to run the eight NI tools and compute node/edge intersections (UpSet analysis)) — https://github.com/segrelab/MiCoNE
- **SparCC** (Correlation-based network inference method (iterations 50) whose output edge list is one comparison input)
- **propr** (Correlation-based network inference method whose output edge list is one comparison input)
- **SpiecEasi** (Direct network inference method (method 'mb', nreps 50, nlambda 20, lambda_min_ratio 1e-2) whose output edge list is one comparison input)
- **FlashWeave** (Direct network inference method (sensitive true, fdr_correction true) whose output edge list is one comparison input)
- **COZINE** (Direct network inference method (lambda_min_ratio 0.1) whose output edge list is one comparison input)
- **HARMONIES** (Direct network inference method (iterations 10000, sparsity_cutoff 0.5) whose output edge list is one comparison input)
- **SPRING** (Direct network inference method (nlambda 20, lambda_min_ratio 0.01) whose output edge list is one comparison input)
- **mLDM** (Direct network inference method (Z_mean 1, max_iteration 1500) whose output edge list is one comparison input)
- **Nextflow** (Executes the micone 'ni' workflow template that generates the eight networks being compared)

## Examples

```
micone init -w ni -o ni_pipeline && bash run.sh
```

## Evaluation signals

- All eight network JSON files exist and each contains 'source', 'target', and 'weight' edge fields.
- Node intersection counts across the eight methods qualitatively match Fig. 4B of the paper (e.g., 33 of 68 unique nodes shared by all methods, with no method producing a unique node).
- Edge intersection counts qualitatively match Fig. 4C (e.g., only 8 of 202 total unique edges common to all methods), demonstrating substantial rewiring.
- Each network was produced with the same thresholds (pvalue_threshold 0.05, interaction_threshold 0.3, 1000 bootstraps) so intersections reflect method differences, not parameter differences.
- Running 'micone validate-results' passes for the pipeline run that produced the networks.

## Limitations

- Networks inferred by different methods from the same data show substantial rewiring, so only 8 of 202 unique edges are common across all methods — consensus networks are needed rather than trusting any single inference.
- FlashWeave requires a compatible Julia version; a preinstalled Julia can conflict with the micone-flashweave environment.
- The MiCoNE data directory (nf_micone/data) must be manually downloaded from Zenodo record 7051556 before running.

## Evidence

- [methods] Load the eight edge lists into MiCoNE Network objects and compute the intersections of nodes and edges across methods (UpSet analysis): [evidence span withheld — anchor status: not_found]
- [intro] The different methods and tools can lead to vastly different inferences of community compositions and co-occurrence networks: "The different methods and tools can lead to vastly different inferences of community compositions and co-occurrence networks (52, 53)"
- [readme] MiCoNE is a Python package for the exploration of the effects of various possible tools used during the 16S data processing workflow: "`MiCoNE` is a Python package for the exploration of the effects of various possible tools used during the 16S data processing workflow on the inferred co-occurrence networks."
- [readme] To initialize the nextflow templates for the micone workflow: [evidence span withheld — anchor status: not_found]

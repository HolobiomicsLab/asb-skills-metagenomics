---
name: compositional-correlation-network-construction
description: Use when you have a genus-level OTU/counts table and need to compare
  networks inferred by multiple compositional-aware inference methods, e.g. to quantify
  shared vs. unique nodes and edges (UpSet overlap) across methods.
when_to_use_negative:
- Input is raw 16S sequencing reads requiring demultiplexing, trimming, chimera checking,
  or OTU assignment — use the MiCoNE 'full' pipeline instead of the 'ni' template.
- Only a single inference method is required and cross-method robustness/consensus
  assessment is not a goal.
- The dataset lacks the taxonomy columns needed to group nodes at the genus level.
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_0699
- http://edamontology.org/topic_3174
tools:
- name: MiCoNE
  role: Python package/pipeline orchestrating bootstrapped network inference, pvalue-to-network
    conversion, taxonomic grouping, and Network-object intersection analysis
  repo: https://github.com/segrelab/MiCoNE
- name: Nextflow
  role: Workflow engine executing the NI pipeline template initialized by `micone
    init -w ni`
- name: SparCC
  role: Correlation-based compositional network inference (iterations 50) on resampled
    count tables
- name: propr
  role: Correlation-based compositional network inference method run under defaults
- name: SpiecEasi
  role: Direct graphical-model inference (method 'mb', nreps 50, nlambda 20, lambda_min_ratio
    1e-2)
- name: FlashWeave
  role: Direct network inference (sensitive true, fdr_correction true)
- name: COZINE
  role: Direct conditional-dependence inference (lambda_min_ratio 0.1)
- name: HARMONIES
  role: Direct network inference (iterations 10000, sparsity_cutoff 0.5)
- name: SPRING
  role: Direct semi-parametric rank-based inference (nlambda 20, lambda_min_ratio
    0.01)
- name: mLDM
  role: Latent variable direct inference (Z_mean 1, max_iteration 1500)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1128/msystems.00961-22
    title: 'Inferring microbial co-occurrence networks from amplicon data: a systematic
      evaluation'
  source_content_hash: sha256:18b4e5182341971fb19061557adb0e1298e8b042b47ab8c6a16c13f29fa6b671
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
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_0699
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# compositional-correlation-network-construction

## Summary

Construct microbial co-occurrence networks from genus-level OTU/count tables using eight network inference methods (SparCC, propr, SpiecEasi, FlashWeave, COZINE, HARMONIES, SPRING, mLDM) within the MiCoNE pipeline, then quantify node/edge agreement across methods with UpSet-style intersection analysis to expose method-driven rewiring.

## When to use

Use when you have a genus-level OTU/counts table and need to compare networks inferred by multiple compositional-aware inference methods, e.g. to quantify shared vs. unique nodes and edges (UpSet overlap) across methods.

## When NOT to use

- Input is raw 16S sequencing reads requiring demultiplexing, trimming, chimera checking, or OTU assignment — use the MiCoNE 'full' pipeline instead of the 'ni' template.
- Only a single inference method is required and cross-method robustness/consensus assessment is not a goal.
- The dataset lacks the taxonomy columns needed to group nodes at the genus level.

## Inputs

- otu_table.tsv (genus-level counts table from MiCoNE Zenodo record 7051556 data.zip)
- obs_metadata.tsv with Kingdom-to-Species taxonomy columns
- sample_metadata.tsv
- MiCoNE NI pipeline template (nextflow.config, main.nf, run.sh, samplesheet.csv)

## Outputs

- Eight network JSON files (one per NI method), each with 'source', 'target', 'weight' edge fields
- UpSet node and edge intersection counts quantifying shared vs. unique nodes/edges across the eight methods (Fig. 4B/C style overlap statistics)

## How to apply

Initialize the network inference (NI) pipeline with `micone init -w ni -o <pipeline_dir>` after installing conda environments via `micone install`. Run the eight inference tools under manuscript defaults — sparcc (iterations 50), propr, spieceasi (method 'mb', nreps 50, nlambda 20, lambda_min_ratio 1e-2), flashweave (sensitive true, fdr_correction true), cozine (lambda_min_ratio 0.1), harmonies (iterations 10000, sparsity_cutoff 0.5), spring (nlambda 20, lambda_min_ratio 0.01), and mldm (Z_mean 1, max_iteration 1500) — each preceded by 1000 Resample bootstraps and Pvalue computation. Convert each bootstrapped result into a network using the pvalue-based Make network step with pvalue_threshold 0.05 and interaction_threshold 0.3, then group nodes at the genus taxonomic level (tax_levels ['Phylum', 'Class', 'Order', 'Family', 'Genus', 'Species']) and write each filtered network with pvalue_filter=True, interaction_filter=False. Load the eight edge lists into MiCoNE Network objects and compute node and edge intersections across methods (UpSet analysis) to reproduce the Fig. 4B/C overlap counts. The rationale is that different methods and tools lead to vastly different network inferences, so systematic comparison of their shared/unique elements quantifies robustness of inferred associations.

## Related tools

- **MiCoNE** (Python package/pipeline orchestrating bootstrapped network inference, pvalue-to-network conversion, taxonomic grouping, and Network-object intersection analysis) — https://github.com/segrelab/MiCoNE
- **Nextflow** (Workflow engine executing the NI pipeline template initialized by `micone init -w ni`)
- **SparCC** (Correlation-based compositional network inference (iterations 50) on resampled count tables)
- **propr** (Correlation-based compositional network inference method run under defaults)
- **SpiecEasi** (Direct graphical-model inference (method 'mb', nreps 50, nlambda 20, lambda_min_ratio 1e-2))
- **FlashWeave** (Direct network inference (sensitive true, fdr_correction true))
- **COZINE** (Direct conditional-dependence inference (lambda_min_ratio 0.1))
- **HARMONIES** (Direct network inference (iterations 10000, sparsity_cutoff 0.5))
- **SPRING** (Direct semi-parametric rank-based inference (nlambda 20, lambda_min_ratio 0.01))
- **mLDM** (Latent variable direct inference (Z_mean 1, max_iteration 1500))

## Examples

```
micone install && micone init -w ni -o ni_pipeline && bash ni_pipeline/run.sh
```

## Evaluation signals

- All eight network JSON files exist after the pipeline run and each contains 'source', 'target', 'weight' edge fields.
- Node and edge intersection counts across the eight methods qualitatively match the overlap pattern reported in Fig. 4B/C (e.g. only 8 of 202 unique edges shared by all methods, 33 of 68 unique nodes shared, and no method producing a unique node).
- Each inference run is preceded by 1000 Resample bootstraps with Pvalue computation, and networks are built with pvalue_threshold 0.05 / interaction_threshold 0.3.
- `micone validate-results` reports no validation errors for the executed pipeline run.
- Every filtered network was written with genus-level grouping (tax_levels Phylum through Species) and pvalue_filter=True.

## Limitations

- Substantial method-driven rewiring is expected: different inference tools infer very different edge sets even on the same counts table, so single-method networks should not be treated as ground truth.
- FlashWeave requires a compatible Julia version — a preinstalled Julia may conflict with the version in the micone-flashweave environment.
- The nf_micone data directory must be manually downloaded from Zenodo record 7051556 (data.zip) before the pipeline can run.
- The `mamba` package manager is required, otherwise `micone init` will not work.

## Evidence

- [intro] The different methods and tools can lead to vastly different inferences of community compositions and co-occurrence networks (52, 53), making it difficult to reliably compare networks across: "The different methods and tools can lead to vastly different inferences of community compositions and co-occurrence networks (52, 53), making it difficult to reliably compare networks across"
- [methods] To set up the nextflow workflow template for the desired workflow, you can use the micone init command: "To set up the ``nextflow`` workflow template for the desired workflow, you can use the ``micone init`` command"
- [methods] | Bootstrap   | Resample                     | bootstraps            | 1000         |: "| Bootstrap   | Resample                     | bootstraps            | 1000         |"
- [methods] | Transform | Group              | tax_levels        | "['Phylum', 'Class', 'Order', 'Family', 'Genus', 'Species']" |: "| Transform | Group              | tax_levels        | "['Phylum', 'Class', 'Order', 'Family', 'Genus', 'Species']" |"
- [methods] If you have an interaction matrix stored in a tsv or csv format, you can load it into a Network object: "If you have an interaction matrix stored in a ``tsv`` or ``csv`` format, you can load it into a ``Network`` object"
- [readme] MiCoNE is a Python package for the exploration of the effects of various possible tools used during the 16S data processing workflow on the inferred co-occurrence networks.: "`MiCoNE` is a Python package for the exploration of the effects of various possible tools used during the 16S data processing workflow on the inferred co-occurrence networks."
- [readme] MiCoNE requires the mamba package manager, otherwise micone init will not work.: "MiCoNE requires the `mamba` package manager, otherwise `micone init` will not work."
- [readme] The data directory (nf_micone/data) needs to be manually downloaded: "The data directory (`nf_micone/data`) needs to be manually downloaded"

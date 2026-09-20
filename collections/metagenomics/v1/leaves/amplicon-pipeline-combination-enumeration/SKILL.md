---
name: amplicon-pipeline-combination-enumeration
description: Use when assessing how alternative tools/parameters at each 16S pipeline
  step (DC, CC, TA, OP, NI) affect the inferred co-occurrence network, or when quantifying
  which step contributes most to total network variation.
when_to_use_negative:
- Input is already a feature table or interaction matrix/edge list — use MiCoNE's
  Network import from tsv/csv or edge list instead of re-running sequence-processing
  combinations.
- Goal is a single fixed pipeline run rather than systematic evaluation of alternative
  tools and parameters at each step.
- Data are not 16S amplicon sequences — MiCoNE workflows are designed to convert raw
  16S sequencing data into co-occurrence networks.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3697
tools:
- name: MiCoNE
  role: Python package/nextflow pipeline that enumerates tool/parameter combinations
    per step and provides the Network/NetworkGroup objects for loading and exporting
    interaction matrices
  repo: https://github.com/segrelab/MiCoNE
- name: Nextflow
  role: Workflow engine that executes the parallelized run (bash run.sh) for every
    configured tool/parameter combination
  repo: www.nextflow.io
- name: qiime2
  role: Backs the taxonomy-assignment step tooling (Naive Bayes classifier gg_13_8_99_515_806,
    BLAST vs ncbi_refseq) inside the micone-qiime2 environment
- name: Python (scikit-learn)
  role: Flattens and stacks adjacency matrices, runs PCA on the stacked network vectors,
    and computes per-step variance via regression and ANOVA
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1128/msystems.00961-22
    title: 'Inferring microbial co-occurrence networks from amplicon data: a systematic
      evaluation'
  source_content_hash: sha256:7293cc5a6eafde77f5ea0395832dd57cd1b32731b0994dd50325af43359604e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/segrelab/MiCoNE
  tools:
  - MiCoNE
  - Nextflow
  - Python
  - Python (scikit-learn)
  - qiime2
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# amplicon-pipeline-combination-enumeration

## Summary

Runs every tool/parameter combination across the DC, CC, TA, OP, and NI steps of a 16S co-occurrence pipeline with MiCoNE/Nextflow, then decomposes total network variation by step using PCA plus regression and ANOVA — revealing which step (e.g. taxonomy assignment at 65.4%) dominates.

## When to use

Use when assessing how alternative tools/parameters at each 16S pipeline step (DC, CC, TA, OP, NI) affect the inferred co-occurrence network, or when quantifying which step contributes most to total network variation.

## When NOT to use

- Input is already a feature table or interaction matrix/edge list — use MiCoNE's Network import from tsv/csv or edge list instead of re-running sequence-processing combinations.
- Goal is a single fixed pipeline run rather than systematic evaluation of alternative tools and parameters at each step.
- Data are not 16S amplicon sequences — MiCoNE workflows are designed to convert raw 16S sequencing data into co-occurrence networks.

## Inputs

- Raw 16S sequence data listed in samplesheet.csv with metadata.json (data directory manually downloaded from Zenodo record 7051556)
- nextflow.config enumerating tool/parameter selections per step (e.g. otu_assignment selection = ['dada2', 'deblur'])
- Taxonomy-assignment reference resources: gg_13_8_99_515_806 Naive Bayes classifier, gg_97 closed-reference sequences (percent_identity 0.97), ncbi_refseq BLAST references

## Outputs

- One co-occurrence interaction matrix per tool/parameter combination, exported via NetworkGroup.write('network.', pvalue_filter=True, interaction_filter=False)
- Flattened adjacency vectors stacked into a matrix with scikit-learn PCA components
- Per-step variance contribution fractions across DC, CC, TA, OP, NI (reported: TA 65.4%, OP 26.8%)

## How to apply

Initialize the pipeline template with `micone init -w ni -o <pipeline_dir>` and edit `nextflow.config` to enumerate tool/parameter selections per step (e.g. under `denoise_cluster` set `otu_assignment { selection = ['dada2', 'deblur'] }`); then execute every combination with `bash run.sh` after installing the conda environments via `micone install` (requires mamba). Apply the manuscript's default network thresholds — interaction_threshold = 0.3, pvalue_threshold = 0.05, 1000 bootstraps for the Resample step — and load each inferred interaction matrix into a `Network` object (`Network.load_data` with interaction_file, meta_file, cmeta_file, obsmeta_file), exporting via `NetworkGroup.write('network.', pvalue_filter=True, interaction_filter=False)`. Flatten each network's adjacency matrix into a vector, stack all combination vectors into a matrix, and run PCA (scikit-learn) to obtain principal components of network variation. Regress each principal component on the categorical step indicators (DC, CC, TA, OP, NI) and compute per-step variance contributions by ANOVA; the rationale is that pipeline steps whose choices lead to 'vastly different inferences of community compositions and co-occurrence networks' will dominate this variance.

## Related tools

- **MiCoNE** (Python package/nextflow pipeline that enumerates tool/parameter combinations per step and provides the Network/NetworkGroup objects for loading and exporting interaction matrices) — https://github.com/segrelab/MiCoNE
- **Nextflow** (Workflow engine that executes the parallelized run (bash run.sh) for every configured tool/parameter combination) — www.nextflow.io
- **qiime2** (Backs the taxonomy-assignment step tooling (Naive Bayes classifier gg_13_8_99_515_806, BLAST vs ncbi_refseq) inside the micone-qiime2 environment)
- **Python (scikit-learn)** (Flattens and stacks adjacency matrices, runs PCA on the stacked network vectors, and computes per-step variance via regression and ANOVA)

## Examples

```
micone init -w ni -o pipeline_dir && bash run.sh
```

## Evaluation signals

- Per-step variance fractions are non-negative and sum to approximately 100%, with taxonomy assignment (TA) dominant at 65.4% and OTU processing (OP) at 26.8%.
- `micone validate-results` reports successful execution for every tool/parameter combination enumerated in nextflow.config.
- Each exported network used the manuscript defaults: interaction_threshold = 0.3, pvalue_threshold = 0.05, 1000 Resample bootstraps, and pvalue_filter=True on export.
- The stacked flattened adjacency matrix yields PCA components whose regression on the DC/CC/TA/OP/NI step indicators leaves no unassigned variance.
- The data directory nf_micone/data (Zenodo record 7051556) was downloaded and present before running the pipeline.

## Limitations

- The data directory (`nf_micone/data`) must be manually downloaded from Zenodo record 7051556 before pipeline execution.
- A preinstalled `julia` can conflict with the version downloaded by the `micone-flashweave` environment.
- `micone init` requires `mamba`; the minimal pip-installed Python library does not provide the functionality to execute pipelines.
- Alternate pipeline templates (full, ni, op_ni, ta_op_ni) are noted as 'work in progress' in the README.

## Evidence

- [intro] The different methods and tools can lead to vastly different inferences of community compositions and co-occurrence networks (52, 53), making it difficult to reliably compare networks across: "The different methods and tools can lead to vastly different inferences of community compositions and co-occurrence networks (52, 53), making it difficult to reliably compare networks across"
- [methods] ``MiCoNE`` is a Python package for the exploration of the effects of various possible tools used during the 16S data processing workflow: "``MiCoNE`` is a Python package for the exploration of the effects of various possible tools used during the 16S data processing workflow"
- [methods] | OTU assignment   | Closed reference | percent_identity    | 0.97        |: "| OTU assignment   | Closed reference | percent_identity    | 0.97        |"
- [readme] micone init -w <workflow> -o <path/to/folder>: "micone init -w <workflow> -o <path/to/folder>"
- [readme] To run the pipeline, update the relevant config files (see next section), activate the ``micone`` environment and run the ``run.sh`` script that was copied to the directory: "To run the pipeline, update the relevant config files (see next section), activate the ``micone`` environment and run the ``run.sh`` script that was copied to the directory"
- [readme] If you have a version of `julia` that is preinstalled, make sure that it does not conflict with the version downloaded by the `micone-flashweave` environment: "If you have a version of `julia` that is preinstalled, make sure that it does not conflict with the version downloaded by the `micone-flashweave` environment"

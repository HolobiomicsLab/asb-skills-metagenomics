---
name: nextflow-pipeline-execution
description: Use when you must run the MiCoNE nextflow pipeline over all combinations
  of tools/parameters for 16S processing steps (DC, CC, TA, OP, NI) and export per-combination
  co-occurrence networks.
when_to_use_negative:
- Input is already a processed abundance/interaction table and you only need network
  inference — use the minimal Python library (pip install micone), but note it 'does
  not provide the functionality to execute pipelines', so template workflows like
  op_ni/ta_op_ni are the wrong scope too.
- You want to re-run demultiplexing/trimming when only downstream steps change — initialize
  a partial template (ni or op_ni) instead of the full pipeline.
- Data are not 16S rRNA amplicons — MiCoNE templates and environments are built for
  converting raw 16S sequence data into co-occurrence networks.
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3697
tools:
- name: MiCoNE
  role: Provides the CLI (micone install/init/clean/validate-results) and nextflow
    workflow templates that enumerate alternate tools per pipeline step
  repo: github.com/segrelab/MiCoNE
- name: Nextflow
  role: DSL2 workflow engine executing run.sh across parameterized tool combinations,
    locally or via qsub on SGE clusters
  repo: www.nextflow.io
- name: qiime2
  role: Backend environment (micone-qiime2) executing taxonomy-assignment/OTU-processing
    processes; input files must follow qiime2 supported formats
- name: Python
  role: Runtime for the micone package and for loading exported interaction matrices
    into Network objects (Network.load_data) after execution
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1128/msystems.00961-22
    title: 'Inferring microbial co-occurrence networks from amplicon data: a systematic
      evaluation'
  source_content_hash: sha256:2a0083e795bf5fc680387ce54444fd940cbefbb603cb9895b37233a321ddf195
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MiCoNE
  - Nextflow
  - Python
  - qiime2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# nextflow-pipeline-execution

## Summary

Executes the MiCoNE nextflow pipeline templates (full, ni, op_ni, ta_op_ni) to run every tool/parameter combination across the 16S processing steps, from raw reads to co-occurrence networks. It is the concrete mechanism for the paper's systematic evaluation of how pipeline-step choices drive network variation.

## When to use

Use when you must run the MiCoNE nextflow pipeline over all combinations of tools/parameters for 16S processing steps (DC, CC, TA, OP, NI) and export per-combination co-occurrence networks.

## When NOT to use

- Input is already a processed abundance/interaction table and you only need network inference — use the minimal Python library (pip install micone), but note it 'does not provide the functionality to execute pipelines', so template workflows like op_ni/ta_op_ni are the wrong scope too.
- You want to re-run demultiplexing/trimming when only downstream steps change — initialize a partial template (ni or op_ni) instead of the full pipeline.
- Data are not 16S rRNA amplicons — MiCoNE templates and environments are built for converting raw 16S sequence data into co-occurrence networks.

## Inputs

- raw 16S amplicon sequencing data referenced by samplesheet.csv, with dataset metadata in metadata.json
- nextflow.config enumerating tool/parameter selections for each pipeline step (DC, CC, TA, OP, NI)
- MiCoNE data directory (nf_micone/data) downloaded manually from Zenodo record 7051556

## Outputs

- one inferred co-occurrence network (interaction matrix) per tool/parameter combination
- JSON files compatible with the MIND database via Network export
- pipeline logs, work directories, and validation reports

## How to apply

Install the pipeline environments first with `micone install` (or a single env such as `micone install -e "micone-qiime2"`), since MiCoNE relies on per-process conda environments (micone-sparcc, micone-spieceasi, micone-flashweave, etc.). Generate the template with `micone init -w <workflow> -o <pipeline_dir>` — choose `ni` to start at network inference, or `full` from raw reads — then edit `nextflow.config` to enumerate the tool/parameter selections per step, e.g. `denoise_cluster { otu_assignment { selection = ['dada2', 'deblur'] } }`. Execute with `bash run.sh` locally or `qsub run.sh` on an SGE cluster, applying the paper's default network thresholds (interaction_threshold = 0.3, pvalue_threshold = 0.05) with 1000 bootstraps in the Resample step. Run `micone validate-results` to check outputs before loading each interaction matrix into a `micone` Network object for downstream PCA/ANOVA variance decomposition. Example configurations used in the manuscript are in the scripts/runs folder of the MiCoNE-pipeline-paper repository.

## Related tools

- **MiCoNE** (Provides the CLI (micone install/init/clean/validate-results) and nextflow workflow templates that enumerate alternate tools per pipeline step) — github.com/segrelab/MiCoNE
- **Nextflow** (DSL2 workflow engine executing run.sh across parameterized tool combinations, locally or via qsub on SGE clusters) — www.nextflow.io
- **qiime2** (Backend environment (micone-qiime2) executing taxonomy-assignment/OTU-processing processes; input files must follow qiime2 supported formats)
- **Python** (Runtime for the micone package and for loading exported interaction matrices into Network objects (Network.load_data) after execution)

## Examples

```
micone init -w ni -o ./micone_ni; bash run.sh; micone validate-results
```

## Evaluation signals

- Run `micone validate-results`, the README-documented subcommand to 'Check the results of the pipeline execution'.
- Confirm the nextflow.config selections expand to the intended Cartesian product of tool choices across the DC, CC, TA, OP, and NI steps before launching run.sh.
- Downstream check: flatten each interaction matrix, run PCA + ANOVA on step indicators, and verify per-step variance fractions are non-negative, sum to ~100%, and that taxonomy assignment dominates (~65.4%) with OTU processing second (~26.8%).
- Verify networks used the default thresholds (interaction_threshold = 0.3, pvalue_threshold = 0.05, 1000 Resample bootstraps) and export with pvalue_filter=True.
- Confirm the data directory nf_micone/data is present (manually downloaded from Zenodo record 7051556) or the run will fail.

## Limitations

- MiCoNE requires the mamba package manager; otherwise micone init will not work.
- The standalone Python library (pip install micone) cannot execute pipelines — only the conda pipeline installation can.
- Known issue: a preinstalled Julia version may conflict with the one in the micone-flashweave environment.
- The nf_micone/data directory must be manually downloaded from Zenodo record 7051556.
- Alternate templates (op_ni, ta_op_ni) were marked 'work in progress' in the README.

## Evidence

- [methods] To set up the ``nextflow`` workflow template for the desired workflow, you can use the ``micone init`` command: micone init -w <workflow> -o <pipeline_dir>: [evidence span withheld — anchor status: not_found]
- [readme] To run the pipeline, update the relevant config files (see next section), activate the `micone` environment and run the `run.sh` script that was copied to the directory: "To run the pipeline, update the relevant config files (see next section), activate the `micone` environment and run the `run.sh` script that was copied to the directory"
- [readme] MiCoNE requires the `mamba` package manager, otherwise `micone init` will not work.: "MiCoNE requires the `mamba` package manager, otherwise `micone init` will not work."
- [readme] In order to run the pipeline various `conda` environments must first be installed on the system. Use the following comand to initialize all the environments: micone install: "In order to run the pipeline various `conda` environments must first be installed on the system. Use the following comand to initialize all the environments: micone install"
- [readme] The `Python` library does not provide the functionality to execute pipelines: [evidence span withheld — anchor status: external_doc]
- [readme] Example configuration files used for the analyses in the manuscript can be found [here](https://github.com/segrelab/MiCoNE-pipeline-paper/tree/master/scripts/runs).: "Example configuration files used for the analyses in the manuscript can be found [here](https://github.com/segrelab/MiCoNE-pipeline-paper/tree/master/scripts/runs)."

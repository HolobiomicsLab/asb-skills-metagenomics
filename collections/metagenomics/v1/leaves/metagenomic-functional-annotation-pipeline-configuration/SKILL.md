---
name: metagenomic-functional-annotation-pipeline-configuration
description: Use when installing MetaLAFFA or customizing its pipeline operation —
  choosing steps, per-step parameters, cluster submission settings, or reference databases
  (UniRef90, KEGG mappings, host decoys) — prior to annotating shotgun metagenomic
  FASTQ data.
when_to_use_negative:
- Input is already a gene/ortholog abundance table and only downstream aggregation
  (e.g., EMPANADA) or a later step is needed — start from a later step using MetaLAFFA's
  'using a different step as a starting point' option instead of reconfiguring the
  full pipeline
- Analysis requires assembly-based annotation or taxonomic profiling rather than read-based
  functional annotation of shotgun data
- Working on Windows — the README states MetaLAFFA is supported on Mac and Linux only
edam_topics:
- http://edamontology.org/topic_3941
tools:
- name: MetaLAFFA
  role: The functional annotation pipeline being installed and configured; provides
    the Python module-based configuration system controlling every step
  repo: https://github.com/borenstein-lab/MetaLAFFA
- name: Snakemake
  role: Workflow engine in which MetaLAFFA is implemented, enabling integration with
    distributed computing clusters via config/cluster.py
- name: Conda
  role: Installation and environment mechanism; MetaLAFFA requires Conda version 4.8
    or greater and is installed from the bioconda and borenstein-lab channels
- name: Python
  role: Language of the configuration modules (config/*.py, config/steps/*.py) that
    expose pipeline customization options
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-020-03815-9
    title: 'MetaLAFFA: a flexible, end-to-end, distributed computing-compatible metagenomic
      functional annotation pipeline'
  source_content_hash: sha256:33b4a80113c67c4cae7ec452fb77808e5cdb5d464d75b619af835feb7b6cb6de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/borenstein-lab/MetaLAFFA
  tools:
  - Conda
  - EMPANADA
  - MetaLAFFA
  - Python
  - Snakemake
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3941
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# metagenomic-functional-annotation-pipeline-configuration

## Summary

Configures MetaLAFFA, a Snakemake-implemented metagenomic functional annotation pipeline, via its Python module-based configuration system (config/file_organization.py, config/operation.py, config/cluster.py, config/steps/*.py, config/library_functions.py) to tailor pre-processing, annotation steps, and cluster submission before producing KO-, module-, and pathway-level functional profiles.

## When to use

Use when installing MetaLAFFA or customizing its pipeline operation — choosing steps, per-step parameters, cluster submission settings, or reference databases (UniRef90, KEGG mappings, host decoys) — prior to annotating shotgun metagenomic FASTQ data.

## When NOT to use

- Input is already a gene/ortholog abundance table and only downstream aggregation (e.g., EMPANADA) or a later step is needed — start from a later step using MetaLAFFA's 'using a different step as a starting point' option instead of reconfiguring the full pipeline
- Analysis requires assembly-based annotation or taxonomic profiling rather than read-based functional annotation of shotgun data
- Working on Windows — the README states MetaLAFFA is supported on Mac and Linux only

## Inputs

- Unfiltered shotgun metagenomic sequencing data (FASTQ files with sequencing adapters removed)
- MetaLAFFA Python configuration modules (config/operation.py, config/cluster.py, config/steps/*.py, config/file_organization.py, config/library_functions.py)
- Reference databases prepared by prepare_databases.py: UniRef90, hs37d5 human reference/decoy sequences, KEGG ko-to-module and ko-to-pathway mappings

## Outputs

- A configured MetaLAFFA project (Snakemake pipeline with user-specified steps, parameters, and cluster submission settings)
- Final functional profiles as tab-separated files in the pipeline's final_output_directory (output/final): KO-, module-, and pathway-level abundance tables

## How to apply

Install the pipeline and all dependencies into a dedicated Conda environment (Conda 4.8+) with 'conda create -n metalaffa metalaffa -c bioconda -c borenstein-lab'. Download and prepare default reference databases with prepare_databases.py, selecting options such as -hr (human reference/decoy sequences from the 1000 genomes project for host filtering), -km (KEGG ko-to-module and ko-to-pathway mappings), and -u (UniRef90 for read mapping/annotation); expect this to need several hours, ~40GB RAM, and ~150–300GB disk depending on whether -c cleans intermediates. Edit the Python configuration modules under config/ to set file organization, operation modes, per-step parameters (e.g., config/steps/map_reads_to_genes.py cluster_params for DIAMOND core counts), and cluster settings; note the default Conda install pins Snakemake 3.13.3. Then launch MetaLAFFA on input FASTQs, letting the Snakemake pipeline orchestrate quality control, read mapping, ortholog counting, MUSiCC correction, and EMPANADA aggregation, writing final TSV outputs to output/final.

## Related tools

- **MetaLAFFA** (The functional annotation pipeline being installed and configured; provides the Python module-based configuration system controlling every step) — https://github.com/borenstein-lab/MetaLAFFA
- **Snakemake** (Workflow engine in which MetaLAFFA is implemented, enabling integration with distributed computing clusters via config/cluster.py)
- **Conda** (Installation and environment mechanism; MetaLAFFA requires Conda version 4.8 or greater and is installed from the bioconda and borenstein-lab channels)
- **Python** (Language of the configuration modules (config/*.py, config/steps/*.py) that expose pipeline customization options)

## Examples

```
conda create -n metalaffa metalaffa -c bioconda -c borenstein-lab; conda activate metalaffa; prepare_databases.py -hr -km -u
```

## Evaluation signals

- The metalaffa Conda environment activates cleanly ('conda activate metalaffa') and all pipeline dependencies resolve
- prepare_databases.py completes and databases appear under $CONDA_PREFIX/MetaLAFFA with expected options used (-hr, -km, -u)
- Modified config/steps/*.py values (e.g., cluster_params core counts for DIAMOND) are honored at runtime without Snakemake configuration errors
- A completed run produces non-empty TSV tables for KO, module, and pathway abundances in output/final containing valid KEGG functional identifiers
- Per-step summary statistics generated by MetaLAFFA indicate expected read retention at each quality-control step

## Limitations

- Reference database preparation is resource intensive: several hours, ~40GB of RAM, and ~150–300GB free disk space (depending on use of the -c cleanup option); interruption without -c forces restarting the entire process
- Default Conda installation pins Snakemake 3.13.3, which may matter for cluster compatibility
- When running locally (not on a cluster), the number of cores in cluster_params of config/steps/map_reads_to_genes.py must be manually adjusted for DIAMOND alignment
- Known FAQ issue: 'undefined symbol: omp_get_num_procs' error when MetaLAFFA runs MUSiCC
- Windows is not supported

## Evidence

- [abstract] a Python module-based configuration system provides advanced users with a flexible interface for pipeline customization: "a Python module-based configuration system provides advanced users with a flexible interface for pipeline customization"
- [intro] MetaLAFFA offers a convenient interface for configuring pipeline operation, providing users with extensive customization options: "MetaLAFFA offers a convenient interface for configuring pipeline operation, providing users with extensive customization options"
- [readme] `-km`: Download KEGG ko-to-module and ko-to-pathway mappings.: "`-km`: Download KEGG ko-to-module and ko-to-pathway mappings."
- [readme] conda create -n metalaffa metalaffa -c bioconda -c borenstein-lab: "conda create -n metalaffa metalaffa -c bioconda -c borenstein-lab"
- [readme] Note: MetaLAFFA's default installation via Conda uses Snakemake version 3.13.3: "Note: MetaLAFFA's default installation via Conda uses Snakemake version 3.13.3"

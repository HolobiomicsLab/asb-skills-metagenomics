---
name: pipeline-run-report-verification
description: Use when a scalepopgen (nf-core-style Nextflow) run has finished and
  you must confirm the main script dispatched all sub-workflows successfully and that
  consolidated HTML execution/analysis reports document per-process resource usage.
when_to_use_negative:
- Input is not biallelic SNP data in VCF or PLINK binary format — scalepopgen is designed
  for biallelic SNP data only
- Only one sub-workflow (e.g., filtering alone) is needed and sub-workflows are run
  separately, so end-to-end dispatch verification does not apply
- The pipeline run errored out before completion — verify installation of Nextflow,
  Java, and the container system first rather than judging report artifacts
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3299
tools:
- name: Nextflow
  role: Workflow manager (DSL2, nf-core-style template) that resolves dependencies,
    dispatches the four sub-workflows, and emits the execution report
  repo: https://github.com/Popgen48/scalepopgen
- name: MultiQC
  role: Runs at the end of the pipeline to compile visualization of each analysis
    into a final HTML report
  repo: https://github.com/Popgen48/scalepopgen
- name: Singularity
  role: One of the container engines used to execute the pipeline modules
  repo: https://github.com/Popgen48/scalepopgen
- name: Docker
  role: Alternative container engine for executing the pipeline modules
  repo: https://github.com/Popgen48/scalepopgen
- name: Python
  role: Scripts embedded in the workflow collect results and produce interactive visualizations;
    also runs the scalepopgen CLI
  repo: https://github.com/Popgen48/scalepopgen
- name: R
  role: Scripts that collect and visualize sub-workflow results into combined plots
  repo: https://github.com/Popgen48/scalepopgen
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/molbev/msae057
    title: 'scalepopgen: Bioinformatic Workflow Resources Implemented in Nextflow
      for Comprehensive Population Genomic Analyses'
  source_content_hash: sha256:3c035c673fad85c37f871276182f2e30def957286118b44576fc503ef1496719
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Popgen48/scalepopgen
  tools:
  - Conda
  - Docker
  - MultiQC
  - Nextflow
  - Python
  - R
  - Singularity
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# pipeline-run-report-verification

## Summary

Validate that the scalepopgen Nextflow workflow executed all four sub-workflows (filtering/basic statistics, genetic structure, TreeMix phylogeny, signatures of selection) end-to-end by checking execution artifacts such as the HTML execution/analysis reports (including MultiQC output) and per-process resource usage.

## When to use

Use when a scalepopgen (nf-core-style Nextflow) run has finished and you must confirm the main script dispatched all sub-workflows successfully and that consolidated HTML execution/analysis reports document per-process resource usage.

## When NOT to use

- Input is not biallelic SNP data in VCF or PLINK binary format — scalepopgen is designed for biallelic SNP data only
- Only one sub-workflow (e.g., filtering alone) is needed and sub-workflows are run separately, so end-to-end dispatch verification does not apply
- The pipeline run errored out before completion — verify installation of Nextflow, Java, and the container system first rather than judging report artifacts

## Inputs

- VCF files split by chromosome and tabix-indexed, listed in a vcf_input.csv sheet with header 'chrom,vcf,vcf_idx'
- PLINK binary files (bed/bim/fam) listed in a .p.csv input sheet
- Tab-delimited sample map file (*.map) with individual IDs in column 1 and population IDs in column 2, no header
- YAML parameters file generated via the scalepopgen CLI (-params-file)

## Outputs

- MultiQC (v1.19) compiled final HTML report visualizing each analysis
- HTML execution report with successful completion status and per-process resource usage
- Interactive Plotly/Bokeh visualizations produced by embedded Python/R collection scripts
- Sub-workflow outputs: filtering/summary statistics, PCA/ADMIXTURE (with CV-error K plot and Q-matrices), IBS/FST clustering, TreeMix trees with OptM migration-edge summary, selection scans (Tajima's D, Pi, FST, SweepFinder2, iHS/XP-EHH)

## How to apply

Launch the pipeline with 'nextflow run popgen48/scalepopgen -profile test', which selects the bundled minimal test dataset and default YAML configuration parameters, letting Nextflow (DSL2) resolve dependencies and execute containerized modules via Singularity, Docker, Conda, or Mamba. Confirm the main script dispatches the sub-workflows in order: sample-then-site filtering and summary statistics, genetic-structure analyses (PCA, ADMIXTURE with CV-error-based K selection, IBS/FST clustering), TreeMix phylogeny with OptM summary of migration edges, and selection-scan workflows (Tajima's D, Pi, FST, SweepFinder2, iHS/XP-EHH on phased data). Check that the embedded Python/R scripts collect results into interactive Plotly/Bokeh visualizations and that MultiQC (v1.19) compiles each analysis into a final HTML report. The skill is validated when the run completes without error, the emitted HTML execution report documents successful completion, and per-process resource usage for all four sub-workflows is present; for reference, a full run finished in ~1.5 h wall time (~136 CPU hours).

## Related tools

- **Nextflow** (Workflow manager (DSL2, nf-core-style template) that resolves dependencies, dispatches the four sub-workflows, and emits the execution report) — https://github.com/Popgen48/scalepopgen
- **MultiQC** (Runs at the end of the pipeline to compile visualization of each analysis into a final HTML report) — https://github.com/Popgen48/scalepopgen
- **Singularity** (One of the container engines used to execute the pipeline modules) — https://github.com/Popgen48/scalepopgen
- **Docker** (Alternative container engine for executing the pipeline modules) — https://github.com/Popgen48/scalepopgen
- **Python** (Scripts embedded in the workflow collect results and produce interactive visualizations; also runs the scalepopgen CLI) — https://github.com/Popgen48/scalepopgen
- **R** (Scripts that collect and visualize sub-workflow results into combined plots) — https://github.com/Popgen48/scalepopgen

## Examples

```
nextflow run scalepopgen/ -profile test,<docker/singularity/conda/mamba> -qs 10
```

## Evaluation signals

- The pipeline run completes without error across all four sub-workflows (filtering/basic statistics, genetic structure, TreeMix, signatures of selection)
- The emitted HTML execution report documents successful completion and per-process resource usage for all four sub-workflows
- A MultiQC HTML report exists at the end of the run and includes visualizations of each analysis
- Interactive Plotly/Bokeh visualization outputs are present, produced by the embedded Python/R scripts
- As a calibration reference, a full pipeline run finished in ~1.5 h (~136 CPU hours) on the paper's benchmark dataset

## Limitations

- Users must pre-install only three dependencies (Nextflow, Java, and one of conda/mamba/singularity/docker); a missing container/environment system will prevent the run
- VCF inputs must be split by chromosome, tabix-indexed, and chromosome names must contain no punctuation marks, or input validation fails
- The README's changelog for v1.0dev has a placeholder date and empty Added/Fixed/Dependencies/Deprecated sections, so release-status and dependency-version documentation are incomplete
- The bundled test profile uses a minimal dataset; timings from the paper's larger dataset (442 samples, ~25 million SNPs, 30 breeds) will not transfer to the test run

## Evidence

- [results] MultiQC (v 1.19) is used at the end of the pipeline to compile visualization of the results of each analysis into a final HTML report.: [evidence span withheld — anchor status: not_found]
- [results] The entire pipeline was finished in ∼1.5 h, which was equivalent to ∼136 CPU hours.: "The entire pipeline was finished in ∼1.5 h, which was equivalent to ∼136 CPU hours."
- [readme] Broadly, the pipeline consists of the following four “sub-workflows”:: [evidence span withheld — anchor status: not_found]
- [readme] Please, make sure to test your setup with -profile test before running the workflow on actual data.: [evidence span withheld — anchor status: not_found]
- [results] The HTML report containing the details about the computational resources and time used by each process: "The HTML report containing the details about the computational resources and time used by each process can be accessed here:"

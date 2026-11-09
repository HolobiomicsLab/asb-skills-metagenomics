---
name: admixture-ancestry-estimation
description: Use when genome-wide biallelic SNP data (VCF or PLINK binary) need individual-level
  ancestry proportions and an objective choice of cluster number K via ADMIXTURE CV-error
  minimization.
when_to_use_negative:
- Input VCFs are not yet sample/site filtered or LD-pruned — run the scalepopgen filtering/basic-statistics
  sub-workflow first (site filtering after sample filtering, MAF/missingness/LD pruning)
  before ADMIXTURE
- Data are not biallelic SNPs (e.g. multiallelic, indel, or structural variant calls)
  — scalepopgen is designed for biallelic SNP data in VCF or PLINK binary format
- The goal is tree-based population relationships with migration edges rather than
  individual ancestry proportions — use the TreeMix sub-workflow instead
edam_operation: http://edamontology.org/operation_3199
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: ADMIXTURE
  role: Runs the ancestry inference over the user-defined K range on the LD-pruned
    PLINK bed file; each run's log file is parsed for the CV error
- name: PLINK
  role: Converts VCF to binary PED (BED) format and recodes non-integer chromosome
    IDs to integers; PLINK2 performs user-defined LD pruning before the analysis
- name: PONG
  role: Interactive visualization of the ADMIXTURE Q-matrices; the pipeline generates
    PONG input files
- name: pophelper
  role: Alternative tool for plotting the ADMIXTURE Q-matrices
- name: Python
  role: Bokeh/Plotly scripts collect the CV error per K and plot the optimal-K Q-matrix
    as interactive plots
- name: Nextflow
  role: Orchestrates the explore-genetic-structure sub-workflow steps (LD pruning,
    ADMIXTURE runs, CV-error collection, plotting)
  repo: https://github.com/Popgen48/scalepopgen
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/molbev/msae057
    title: 'scalepopgen: Bioinformatic Workflow Resources Implemented in Nextflow
      for Comprehensive Population Genomic Analyses'
  source_content_hash: sha256:ffe1f2e973652b75370e6aa472fd38d8ac466c86a6557cdcde477f9e7619f851
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Popgen48/scalepopgen
  tools:
  - ADMIXTURE
  - Bokeh
  - Eigensoft
  - Nextflow
  - PLINK
  - PONG
  - Python
  - pophelper
  edam_operation: http://edamontology.org/operation_3199
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# admixture-ancestry-estimation

## Summary

Runs ADMIXTURE ancestry-proportion inference on a LD-pruned PLINK binary (BED) genotype file across a user-defined range of ancestral components (K), selects the optimal K by lowest cross-validation (CV) error, and renders the resulting Q-matrices as interactive plots (Bokeh/Plotly) plus PONG-ready inputs — as implemented in the scalepopgen Nextflow 'explore genetic structure' sub-workflow.

## When to use

Use when genome-wide biallelic SNP data (VCF or PLINK binary) need individual-level ancestry proportions and an objective choice of cluster number K via ADMIXTURE CV-error minimization.

## When NOT to use

- Input VCFs are not yet sample/site filtered or LD-pruned — run the scalepopgen filtering/basic-statistics sub-workflow first (site filtering after sample filtering, MAF/missingness/LD pruning) before ADMIXTURE
- Data are not biallelic SNPs (e.g. multiallelic, indel, or structural variant calls) — scalepopgen is designed for biallelic SNP data in VCF or PLINK binary format
- The goal is tree-based population relationships with migration edges rather than individual ancestry proportions — use the TreeMix sub-workflow instead

## Inputs

- Filtered PLINK binary genotype file (BED/BIM/FAM) with integer chromosome IDs, produced by the scalepopgen filtering sub-workflow
- User-defined range of ancestral components (K values) and PLINK2 LD-pruning parameters
- Tab-delimited sample map (individual IDs and population IDs, '.map', no header) for population labeling/coloring

## Outputs

- Per-K ADMIXTURE log files containing the cross-validation (CV) error
- CV-error table/plot identifying the optimal K
- Q-matrix (ancestry proportions) for the optimal K, plotted interactively with Bokeh/Plotly with consistent color-population pairs
- Input files for interactive PONG visualization of all ADMIXTURE runs

## How to apply

Start from the filtered PLINK binary (BED) genotype file produced by the upstream filtering/LD-pruning sub-workflow; if chromosome IDs are non-integer, first recode them to integers (e.g. with PLINK) so downstream tools can read the data. LD-prune the SNPs with PLINK2 using user-defined LD parameters before supplying the file to ADMIXTURE, since LD pruning is the recommended pre-step for structure inference in scalepopgen. Run ADMIXTURE (v 7.0.2) on the bed file over the user-defined range of K values, parsing each run's log file to collect the CV error, then select the K with the lowest CV error. Plot the optimal-K Q-matrix as an interactive plot with the pipeline's Python Bokeh/Plotly scripts, keeping user-supplied or randomly allocated color-population pairs consistent, and generate the input files for interactive visualization with PONG (and pophelper as an alternative).

## Related tools

- **ADMIXTURE** (Runs the ancestry inference over the user-defined K range on the LD-pruned PLINK bed file; each run's log file is parsed for the CV error)
- **PLINK** (Converts VCF to binary PED (BED) format and recodes non-integer chromosome IDs to integers; PLINK2 performs user-defined LD pruning before the analysis)
- **PONG** (Interactive visualization of the ADMIXTURE Q-matrices; the pipeline generates PONG input files)
- **pophelper** (Alternative tool for plotting the ADMIXTURE Q-matrices)
- **Python** (Bokeh/Plotly scripts collect the CV error per K and plot the optimal-K Q-matrix as interactive plots)
- **Nextflow** (Orchestrates the explore-genetic-structure sub-workflow steps (LD pruning, ADMIXTURE runs, CV-error collection, plotting)) — https://github.com/Popgen48/scalepopgen

## Examples

```
nextflow run scalepopgen/ -profile test_full,docker -params-file parameters.yml -qs 10
```

## Evaluation signals

- The CV-error table contains a CV-error value for every tested K in the user-defined range
- The selected optimal K is the one with the lowest CV error across all runs
- The Q-matrix plot renders ancestry proportions for all samples of the optimal K, with breed/population colors kept consistent with user-supplied or randomly allocated color-population pairs
- Chromosome IDs in the input BED file were recoded to integers before running (non-integer IDs would break downstream tool parsing)
- Breed clustering of the optimal-K Q-matrix is qualitatively consistent with the cattle case-study results (269 samples from 17 breeds after filtering, 129,357 LD-pruned SNPs)

## Limitations

- Non-integer chromosome IDs must be recoded to integers (e.g. with PLINK) before running, since tools in the sub-workflow expect integer chromosome IDs
- LD pruning with user-defined PLINK2 parameters is required before ADMIXTURE; running on unpruned SNPs violates the workflow design (LD-based filtering pruned the case-study data to 129,357 SNPs)
- Optimal K identification is limited to the user-defined K range and the lowest-CV-error criterion; the CV-error plot must cover every tested K for validation

## Evidence

- [methods] user-defined pruning of the data based on LD parameters of Plink2 (Chang et al. 2015), before supplying the input file to its respective analysis: "user-defined pruning of the data based on LD parameters of Plink2 (Chang et al. 2015), before supplying the input file to its respective analysis"
- [intro] run ADMIXTURE on the bed file then plot CV error to pick K: "(iii) run ADMIXTURE tool on the bed file"
- [intro] plot cross-validation (CV) error to identify a suitable value of K: "(iv) plot cross-validation (CV) error to identify a suitable value of K"
- [intro] plot Q-matrices of ADMIXTURE outputs using tools such as pong and pophelper: "(v) plot Q-matrices of ADMIXTURE outputs using tools such as pong (Behr et al. 2016) and pophelper (Francis 2017)"
- [intro] conversion of VCF to binary PED (BED) file format using plink: [evidence span withheld — anchor status: not_found]
- [results] the data set was further pruned to 129,357 SNPs using LD-based filtering: "the data set was further pruned to 129,357 SNPs using LD-based filtering"

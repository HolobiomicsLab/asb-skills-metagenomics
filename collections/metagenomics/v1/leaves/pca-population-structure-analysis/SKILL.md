---
name: pca-population-structure-analysis
description: Use when genome-wide biallelic SNP data (VCF or PLINK BED/BIM/FAM) need
  population structure inference via PCA and ADMIXTURE, including determining the
  optimal ancestral components K by CV error.
when_to_use_negative:
- The analysis goal is phylogeny or migration edges — the pipeline routes those to
  a separate TreeMix sub-workflow, not to PCA/ADMIXTURE
- Data are not biallelic SNPs in VCF or PLINK binary format — scalepopgen is scoped
  to biallelic SNP data only
- Variants have not been LD-pruned — running ADMIXTURE/PCA on genome-wide LD-correlated
  SNPs violates the sub-workflow's prerequisite of user-defined PLINK2 LD pruning
  before analysis
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3056
tools:
- name: Nextflow
  role: Workflow manager orchestrating the explore-genetic-structure sub-workflow
    and its tool dependencies
  repo: https://github.com/Popgen48/scalepopgen
- name: Eigensoft (smartpca)
  role: Runs principal component analysis on the LD-pruned PLINK binary genotypes
    (v 8.0.0)
- name: ADMIXTURE
  role: Estimates ancestry proportions over a range of K values and reports cross-validation
    error per K (v 7.0.2)
- name: PLINK
  role: Recode non-integer chromosome IDs to integers and LD-prune SNPs with PLINK2
    user-defined parameters before structure analyses
- name: PONG
  role: Interactive visualization of ADMIXTURE Q-matrices across K values
- name: Python (Bokeh/Plotly)
  role: Parses ADMIXTURE logs for CV error, extracts per-PC variance explained, and
    renders interactive Q-matrix plots
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/molbev/msae057
    title: 'scalepopgen: Bioinformatic Workflow Resources Implemented in Nextflow
      for Comprehensive Population Genomic Analyses'
  source_content_hash: sha256:8022fa4a2e0ba4df7b517c0f7e4fc9721f6d460f200ffb7b2578ced3593a511f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Popgen48/scalepopgen
  tools:
  - ADMIXTURE
  - Bokeh
  - Eigensoft
  - Eigensoft (smartpca)
  - Nextflow
  - PLINK
  - PONG
  - Python
  - Python (Bokeh/Plotly)
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3056
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# pca-population-structure-analysis

## Summary

Implements the scalepopgen 'explore genetic structure' sub-workflow: PCA with smartpca (Eigensoft v8.0.0) and ADMIXTURE (v7.0.2) with identification of the optimal number of clusters (K) via cross-validation error, on LD-pruned PLINK binary SNP data.

## When to use

Use when genome-wide biallelic SNP data (VCF or PLINK BED/BIM/FAM) need population structure inference via PCA and ADMIXTURE, including determining the optimal ancestral components K by CV error.

## When NOT to use

- The analysis goal is phylogeny or migration edges — the pipeline routes those to a separate TreeMix sub-workflow, not to PCA/ADMIXTURE
- Data are not biallelic SNPs in VCF or PLINK binary format — scalepopgen is scoped to biallelic SNP data only
- Variants have not been LD-pruned — running ADMIXTURE/PCA on genome-wide LD-correlated SNPs violates the sub-workflow's prerequisite of user-defined PLINK2 LD pruning before analysis

## Inputs

- Filtered, LD-pruned PLINK binary genotype files (.bed/.bim/.fam) from the upstream filtering sub-workflow
- Sample map file (.map) of tab-delimited individual and population IDs
- YAML parameters file specifying the range of ancestral components (K values) and LD-pruning settings

## Outputs

- PCA coordinates and per-PC percentage of genetic variance explained (smartpca via Eigensoft v8.0.0)
- Per-K ADMIXTURE log files containing CV error values
- CV-error table/plot identifying the K with the lowest cross-validation error
- Interactive Q-matrix ancestry plot (Bokeh/Plotly) for the optimal K with consistent color-population pairs
- PONG input files for interactive multi-K Q-matrix visualization

## How to apply

Load the filtered PLINK binary (BED) genotype file from the upstream filtering/LD-pruning sub-workflow; if chromosome IDs are non-integer, recode them to integers with PLINK so downstream tools can read the data. LD-prune SNPs with PLINK2 using user-defined LD parameters before structure analyses, since LD-pruned markers (e.g. the case study's 129,357 SNPs from ~23.4 million) reduce bias in PCA and ADMIXTURE. Run smartpca from the Eigensoft package (v8.0.0) for PCA and extract the percentage of genetic variance explained by each PC with the pipeline's Python scripts. Run ADMIXTURE (v7.0.2) over the user-defined range of K values, parsing each run's log file to collect the CV error, then select the K with the lowest CV error and plot its Q-matrix as an interactive Bokeh/Plotly plot with consistent user-supplied or random color-population pairs. Finally, generate inputs for interactive visualization of ADMIXTURE results with PONG (and pophelper) so Q-matrices across K values can be explored.

## Related tools

- **Nextflow** (Workflow manager orchestrating the explore-genetic-structure sub-workflow and its tool dependencies) — https://github.com/Popgen48/scalepopgen
- **Eigensoft (smartpca)** (Runs principal component analysis on the LD-pruned PLINK binary genotypes (v 8.0.0))
- **ADMIXTURE** (Estimates ancestry proportions over a range of K values and reports cross-validation error per K (v 7.0.2))
- **PLINK** (Recode non-integer chromosome IDs to integers and LD-prune SNPs with PLINK2 user-defined parameters before structure analyses)
- **PONG** (Interactive visualization of ADMIXTURE Q-matrices across K values)
- **Python (Bokeh/Plotly)** (Parses ADMIXTURE logs for CV error, extracts per-PC variance explained, and renders interactive Q-matrix plots)

## Examples

```
nextflow run scalepopgen/ -profile <docker/singularity/conda/mamba> -params-file <path/to/parameters.yml> -qs 10
```

## Evaluation signals

- The CV-error table contains a CV-error value for every tested K value
- The Q-matrix plot renders ancestry proportions for all samples at the optimal K (lowest CV error)
- PC variance-extraction script outputs a per-PC percentage of genetic variance explained
- Breed/population clustering in PCA and Q-matrix plots is qualitatively consistent with the paper's cattle case-study results (442 samples, 30 breeds; 269 samples/17 breeds after filtering)
- The PLINK BED file has integer chromosome IDs so smartpca and ADMIXTURE can read it without format errors

## Limitations

- Only three dependencies (Nextflow, Java, and a container/environment system) are installed by the user; all other tool versions (Eigensoft 8.0.0, ADMIXTURE 7.0.2) are pinned by the pipeline and fetched automatically
- Non-integer chromosome IDs must be recoded to integers before this sub-workflow, a preprocessing burden on the user
- Optimal K selection depends entirely on CV error over the user-defined K range; a too-narrow range can miss the true optimum

## Evidence

- [intro] run ADMIXTURE on the bed file: "conversion of VCF to binary PED bed (BED) file format using plink (Purcell et al. 2007)"
- [intro] plot CV error to identify suitable K: "plot cross-validation (CV) error to identify a suitable value of K"
- [intro] Q-matrix plotting with pong and pophelper: "plot Q-matrices of ADMIXTURE outputs using tools such as pong (Behr et al. 2016) and pophelper (Francis 2017)"
- [methods] LD pruning before analysis: "user-defined pruning of the data based on LD parameters of Plink2 (Chang et al. 2015), before supplying the input file to its respective analysis"
- [readme] Four sub-workflows of scalepopgen: "The sub-workflows can be used separately or in combination with each other."

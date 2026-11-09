---
name: q-matrix-interactive-visualization
description: Use when ADMIXTURE runs over a range of K values have produced per-run
  Q-matrix and log files, and you need to select the K with the lowest cross-validation
  error and interactively visualize ancestry proportions per sample.
when_to_use_negative:
- The goal is PCA rather than admixture proportions — use the smartpca (Eigensoft)
  branch of the sub-workflow instead of Q-matrix plotting.
- Input is raw VCF rather than a filtered, LD-pruned PLINK BED file — run the filtering
  and LD-pruning sub-workflow first.
- Data are not biallelic SNP genotypes in VCF or PLINK binary format, which is the
  only input scalepopgen accepts.
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2885
tools:
- name: PONG
  role: Builds input files for interactive visualization of ADMIXTURE Q-matrix results
    across K runs
  repo: https://github.com/Popgen48/scalepopgen
- name: pophelper
  role: Alternative R tool used to plot Q-matrices of ADMIXTURE outputs
- name: ADMIXTURE
  role: Produces the Q-matrices and CV-error logs that are visualized (v 7.0.2 in
    the sub-workflow)
- name: Python (Bokeh/Plotly)
  role: Pipeline scripts that render the optimal-K Q-matrix as an interactive plot
    with consistent color-population pairs
- name: PLINK
  role: Recode non-integer chromosome IDs to integers and LD-prune SNPs before structure
    analysis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/molbev/msae057
    title: 'scalepopgen: Bioinformatic Workflow Resources Implemented in Nextflow
      for Comprehensive Population Genomic Analyses'
  source_content_hash: sha256:3dd407a78926a63669402d7b8d8e497bf9897d93a9ad693812008f0150a2f804
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Popgen48/scalepopgen
  tools:
  - ADMIXTURE
  - Bokeh
  - Eigensoft
  - PLINK
  - PONG
  - Python
  - Python (Bokeh/Plotly)
  - pophelper
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_2885
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# q-matrix-interactive-visualization

## Summary

Prepare ADMIXTURE Q-matrix output files and render them as interactive population-structure plots using PONG (with Poppins/pophelper-style plotting and Bokeh/Plotly scripts) within the scalepopgen explore-genetic-structure sub-workflow, so users can inspect ancestry proportions per individual at the optimal K.

## When to use

Use when ADMIXTURE runs over a range of K values have produced per-run Q-matrix and log files, and you need to select the K with the lowest cross-validation error and interactively visualize ancestry proportions per sample.

## When NOT to use

- The goal is PCA rather than admixture proportions — use the smartpca (Eigensoft) branch of the sub-workflow instead of Q-matrix plotting.
- Input is raw VCF rather than a filtered, LD-pruned PLINK BED file — run the filtering and LD-pruning sub-workflow first.
- Data are not biallelic SNP genotypes in VCF or PLINK binary format, which is the only input scalepopgen accepts.

## Inputs

- Per-run ADMIXTURE Q-matrix files (.Q) for each tested K
- Per-run ADMIXTURE log files containing the cross-validation error
- LD-pruned PLINK binary (BED/BIM/FAM) genotype file with integer chromosome IDs
- Sample map file of individual IDs and population IDs (tab-delimited, .map)

## Outputs

- CV-error table/plot covering every tested K
- Interactive Q-matrix plot of ancestry proportions for all samples at the optimal K
- PONG input files for interactive exploration of multiple K runs

## How to apply

After the sub-workflow runs ADMIXTURE (v 7.0.2) over the user-defined range of ancestral components (K values) on the LD-pruned PLINK BED file, parse each run's log file to collect the CV error and identify the K with the lowest CV error. Feed the set of Q-matrices to PONG, which builds the input files for interactive visualization, and plot the optimal-K Q-matrix with the pipeline's Python Bokeh/Plotly scripts, keeping user-supplied or randomly allocated color-population pairs consistent across plots. Q-matrices can also be plotted with pophelper as an alternative. The workflow assumes chromosome IDs were recoded to integers (e.g. with PLINK) and SNPs were LD-pruned with PLINK2 using user-defined parameters before structure inference.

## Related tools

- **PONG** (Builds input files for interactive visualization of ADMIXTURE Q-matrix results across K runs) — https://github.com/Popgen48/scalepopgen
- **pophelper** (Alternative R tool used to plot Q-matrices of ADMIXTURE outputs)
- **ADMIXTURE** (Produces the Q-matrices and CV-error logs that are visualized (v 7.0.2 in the sub-workflow))
- **Python (Bokeh/Plotly)** (Pipeline scripts that render the optimal-K Q-matrix as an interactive plot with consistent color-population pairs)
- **PLINK** (Recode non-integer chromosome IDs to integers and LD-prune SNPs before structure analysis)

## Examples

```
nextflow run scalepopgen/ -profile docker -params-file params.yml -qs 10
```

## Evaluation signals

- The CV-error table contains a CV-error value for every tested K.
- The optimal K is the one with the lowest cross-validation error across all parsed ADMIXTURE log files.
- The Q-matrix plot renders ancestry proportions for all samples of the optimal K (no missing individuals).
- Breed clustering in the Q-matrix plot is qualitatively consistent with the cattle case-study results (269 samples, 17 breeds).
- Color-population pairs remain consistent between the Bokeh/Plotly plot and the PONG visualization (user-supplied or randomly allocated pairs preserved).

## Limitations

- Chromosome IDs must be integers; non-integer IDs must be recoded with PLINK before downstream tools can read the data.
- Q-matrix interpretation is only as reliable as the upstream sample/site filtering and LD pruning; the case study pruned ~23.4 million SNPs to 129,357 SNPs before structure analysis.
- The optimal-K choice depends on the user-defined range of K values tested; a too-narrow range can miss the CV-error minimum.

## Evidence

- [intro] plot Q-matrices of ADMIXTURE outputs using tools such as pong (Behr et al. 2016) and pophelper (Francis 2017): "plot Q-matrices of ADMIXTURE outputs using tools such as pong (Behr et al. 2016) and pophelper (Francis 2017)"
- [intro] plot cross-validation (CV) error to identify a suitable value of K: "plot cross-validation (CV) error to identify a suitable value of K"
- [methods] Identify the K value with the lowest CV error and plot its Q-matrix as an interactive plot using the Python Bokeh/Plotly scripts, keeping the user-supplied or randomly allocated color-population pairs consistent: [evidence span withheld — anchor status: not_found]
- [results] the data set was further pruned to 129,357 SNPs using LD-based filtering: "the data set was further pruned to 129,357 SNPs using LD-based filtering"
- [readme] All VCF files need to be splitted by the chromosomes and indexed with tabix.: [evidence span withheld — anchor status: external_doc]

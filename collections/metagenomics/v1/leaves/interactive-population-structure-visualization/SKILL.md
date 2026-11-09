---
name: interactive-population-structure-visualization
description: Use when a Nextflow population-genomics pipeline has produced analysis
  outputs (PCA variance/coordinates, ADMIXTURE logs and Q-matrices, IBS and FST distance
  matrices, TreeMix tree files) and the user needs combined, interactive plots to
  interpret population structure immediately.
when_to_use_negative:
- The input data are raw VCF or PLINK binary files that have not yet been processed
  by the structure/phylogeny sub-workflows — run filtering, PCA, ADMIXTURE, IBS/FST,
  or TreeMix analyses first
- Only static publication figures are needed and no hover tooltips, hide/expose controls,
  or cross-plot color consistency are required
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_0092
tools:
- name: Python
  role: Runs the scripts that collect sub-workflow results, parse ADMIXTURE CV errors,
    and drive plotting
  repo: https://github.com/Popgen48/scalepopgen
- name: Bokeh
  role: Renders the optimal-K ADMIXTURE Q-matrix as an interactive bar plot and generates
    PONG input files (v3.2.1)
  repo: https://github.com/Popgen48/scalepopgen
- name: Plotly
  role: Renders PCA results (PC coordinates and variance-explained percentages) as
    interactive scatter plots (v5.18.0)
- name: Toytree
  role: Renders neighbor-joining trees from IBS and FST distance matrices as interactive
    trees (v2.0.5)
- name: ete3
  role: Flags polyphyletic population clusters in the trees and can draw TreeMix/consensus
    trees (v3.1.3)
- name: Biopython
  role: Builds neighbor-joining trees from the '1-ibs' and pairwise FST distance matrices
- name: PONG
  role: Visualization backend for ADMIXTURE Q-matrices, with input files generated
    by the pipeline's Python scripts
- name: R
  role: Provided plotting scripts draw the TreeMix and consensus trees
  repo: https://github.com/Popgen48/scalepopgen
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/molbev/msae057
    title: 'scalepopgen: Bioinformatic Workflow Resources Implemented in Nextflow
      for Comprehensive Population Genomic Analyses'
  source_content_hash: sha256:f77f8eda2d9298706591276f5f7a9629612eee9f7943a9b70731840d4b3c3c8b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Popgen48/scalepopgen
  tools:
  - Biopython
  - Bokeh
  - PONG
  - Plotly
  - Python
  - R
  - Toytree
  - ete3
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# interactive-population-structure-visualization

## Summary

Collects the outputs of scalepopgen's structure/phylogeny sub-workflows (PCA coordinates, ADMIXTURE Q-matrices, IBS/FST matrices, TreeMix trees) and renders them as interactive HTML visualizations with consistent population coloring so users can quickly interpret genomic patterns after pipeline execution.

## When to use

Use when a Nextflow population-genomics pipeline has produced analysis outputs (PCA variance/coordinates, ADMIXTURE logs and Q-matrices, IBS and FST distance matrices, TreeMix tree files) and the user needs combined, interactive plots to interpret population structure immediately.

## When NOT to use

- The input data are raw VCF or PLINK binary files that have not yet been processed by the structure/phylogeny sub-workflows — run filtering, PCA, ADMIXTURE, IBS/FST, or TreeMix analyses first
- Only static publication figures are needed and no hover tooltips, hide/expose controls, or cross-plot color consistency are required

## Inputs

- PCA output: PC coordinates plus percentage of genetic variance explained per PC
- ADMIXTURE log files containing cross-validation (CV) errors for each K
- ADMIXTURE Q-matrices for each user-defined K value
- '1-ibs' identity-by-state distance matrix
- Pairwise FST distance matrix
- TreeMix tree files (with or without bootstrapping and migration edges)
- Sample map file of individual IDs and population IDs (tab-delimited .map)

## Outputs

- Interactive HTML ADMIXTURE bar plot of the optimal-K Q-matrix (Bokeh)
- PONG input files for multi-K admixture visualization
- Interactive neighbor-joining trees built from IBS and FST matrices (Toytree), with polyphyletic clusters flagged (ete3)
- Interactive PCA scatter plots (Plotly) annotated with variance explained
- TreeMix and consensus tree plots via R plotting scripts or ete3
- A consistent color-to-population mapping shared across all emitted plots

## How to apply

Load the sub-workflow outputs: PCA variance-explained percentages and PC coordinates, ADMIXTURE log files and Q-matrices, IBS and FST distance matrices, and TreeMix tree files. Parse the ADMIXTURE logs with Python to extract cross-validation (CV) errors per user-defined K and pick the K with the lowest CV error, then plot that optimal K's Q-matrix as an interactive Bokeh (v3.2.1) bar plot and generate PONG input files. Build neighbor-joining trees from the '1-ibs' and pairwise FST distance matrices with Biopython, render them interactively with Toytree (v2.0.5), and flag polyphyletic population clusters with ete3 (v3.1.3). Render PCA results as interactive Plotly (v5.18.0) scatter plots and draw TreeMix/consensus trees with the provided R plotting scripts or ete3. Apply a single color-to-population allocation (user-supplied or random) across all plots so the color-population pair stays consistent; emit HTML artifacts with hover tooltips revealing sample/population information and controls to hide or expose dots or bars.

## Related tools

- **Python** (Runs the scripts that collect sub-workflow results, parse ADMIXTURE CV errors, and drive plotting) — https://github.com/Popgen48/scalepopgen
- **Bokeh** (Renders the optimal-K ADMIXTURE Q-matrix as an interactive bar plot and generates PONG input files (v3.2.1)) — https://github.com/Popgen48/scalepopgen
- **Plotly** (Renders PCA results (PC coordinates and variance-explained percentages) as interactive scatter plots (v5.18.0))
- **Toytree** (Renders neighbor-joining trees from IBS and FST distance matrices as interactive trees (v2.0.5))
- **ete3** (Flags polyphyletic population clusters in the trees and can draw TreeMix/consensus trees (v3.1.3))
- **Biopython** (Builds neighbor-joining trees from the '1-ibs' and pairwise FST distance matrices)
- **PONG** (Visualization backend for ADMIXTURE Q-matrices, with input files generated by the pipeline's Python scripts)
- **R** (Provided plotting scripts draw the TreeMix and consensus trees) — https://github.com/Popgen48/scalepopgen

## Examples

```
nextflow run scalepopgen/ -profile <docker/singularity/conda/mamba> -params-file <path/to/parameters.yml> -qs 10
```

## Evaluation signals

- Each emitted HTML artifact renders with hover-over tooltips revealing sample/population information
- Every interactive plot exposes controls to hide or expose dots or bars
- The same population-color mapping appears in every plot across all analyses
- The ADMIXTURE bar plot uses the K with the lowest CV error extracted from the log files
- Neighbor-joining trees reconstructed from the '1-ibs' and FST matrices reflect the same population clusters, with polyphyletic clusters flagged by ete3

## Limitations

- Interactive visualization depends on the upstream analyses having completed; there are no plots if PCA, ADMIXTURE, IBS/FST, or TreeMix outputs are missing
- Population colors must be supplied by the user or are chosen randomly, so reproducibility of colors across runs requires a user-supplied mapping
- Only biallelic SNP data in VCF or PLINK binary format are supported as pipeline input; multi-allelic or haplotype data are out of scope

## Evidence

- [abstract] Python and R scripts collect and visualize results: "several Python and R scripts are also provided to collect and visualize the results"
- [readme] README on combining and plotting results: "python and R scripts have been developed to combine and plot the results of analyses, which allows user to immediately get an impression about the genomic patterns"
- [full_text] Source card finding on interactive visualizations: [evidence span withheld — anchor status: not_found]
- [intro] Intro on plotting CV errors and Q-matrices: "plot cross-validation (CV) error to identify a suitable value of K"

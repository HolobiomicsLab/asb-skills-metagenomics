---
name: cross-analysis-color-consistency-assignment
description: Use when rendering multiple interactive plots from different sub-workflow
  outputs (PCA, ADMIXTURE Q-matrices, IBS/FST trees, TreeMix) of the same sample set
  and the population-to-color mapping must be identical across all figures.
when_to_use_negative:
- Only a single analysis/plot is being produced, so cross-plot color consistency has
  no effect
- The user has not provided a sample map linking individual IDs to population IDs,
  so no population-level color allocation can be defined
- The task is upstream analysis execution (e.g., running ADMIXTURE or TreeMix) rather
  than collection and visualization of existing results
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3318
tools:
- name: Plotly
  role: Renders PCA PC coordinates with variance-explained percentages as interactive
    scatter plots honoring the shared color mapping
- name: Bokeh
  role: Renders the optimal-K ADMIXTURE Q-matrix as an interactive bar plot with hoverable/controllable
    elements using the shared color mapping
- name: Toytree
  role: Renders neighbor-joining trees from IBS and FST distance matrices as interactive
    trees using the shared color mapping
- name: ete3
  role: Flags polyphyletic population clusters and can draw TreeMix/consensus trees
    with the shared color mapping
- name: Biopython
  role: Builds neighbor-joining trees from the '1-ibs' and pairwise FST distance matrices
    prior to interactive rendering
- name: PONG
  role: Visualization of ADMIXTURE Q-matrices for which scalepopgen generates input
    files using the shared color mapping
- name: Python
  role: Hosts the plotting scripts that parse analysis outputs and enforce the single
    color-to-population allocation across all plots
  repo: https://github.com/Popgen48/scalepopgen
- name: R
  role: Provides plotting scripts (e.g., for TreeMix and consensus trees) that consume
    the same population-color allocation
  repo: https://github.com/Popgen48/scalepopgen
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/molbev/msae057
    title: 'scalepopgen: Bioinformatic Workflow Resources Implemented in Nextflow
      for Comprehensive Population Genomic Analyses'
  source_content_hash: sha256:510e24ec75f288bb8e5bf452ef62f71734726aadb0a4dd63d1c0b7f3f43fc2cf
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# Cross-analysis color consistency assignment

## Summary

A visualization-harmonization step in scalepopgen that applies a single color-to-population allocation across all interactive plots (PCA scatter plots, ADMIXTURE bar plots, IBS/FST trees, TreeMix trees) so that a given population has the same color in every emitted HTML artifact.

## When to use

Use when rendering multiple interactive plots from different sub-workflow outputs (PCA, ADMIXTURE Q-matrices, IBS/FST trees, TreeMix) of the same sample set and the population-to-color mapping must be identical across all figures.

## When NOT to use

- Only a single analysis/plot is being produced, so cross-plot color consistency has no effect
- The user has not provided a sample map linking individual IDs to population IDs, so no population-level color allocation can be defined
- The task is upstream analysis execution (e.g., running ADMIXTURE or TreeMix) rather than collection and visualization of existing results

## Inputs

- PCA PC coordinates and per-PC percentage of genetic variance explained
- ADMIXTURE Q-matrices and log files with cross-validation errors per K
- '1-ibs' identity-by-state and pairwise FST distance matrices
- TreeMix tree files (with or without bootstrapping and migration edges)
- Sample map of individual IDs to population IDs (two-column tab-delimited .map file)
- Optional user-supplied color-to-population allocation

## Outputs

- Interactive HTML scatter plots of PCA results (Plotly)
- Interactive HTML bar plots of ADMIXTURE Q-matrices for the optimal K (Bokeh) and PONG visualization input files
- Interactive rendered neighbor-joining trees from IBS and FST matrices (Toytree) with polyphyletic population clusters flagged (ete3)
- A single, consistent population-to-color mapping shared by all plots

## How to apply

After collecting the structure-analysis outputs (PCA coordinates and variance-explained percentages, ADMIXTURE Q-matrices and CV-error logs, IBS and FST distance matrices, TreeMix tree files), define one color-to-population allocation that is either supplied by the user or randomly chosen, and reuse that single mapping when rendering every plot. Render PCA scatter plots with Plotly v5.18.0, ADMIXTURE Q-matrices of the optimal K (lowest cross-validation error) as interactive Bokeh v3.2.1 bar plots plus PONG input files, and neighbor-joining trees from the '1-ibs' and pairwise FST matrices with Toytree v2.0.5/Biopython, flagging polyphyletic clusters with ete3 v3.1.3. Each plot is emitted as an HTML artifact with hover tooltips showing sample/population information and controls to hide or expose dots or bars. The rationale is that consistent coloring lets the user immediately compare population clustering across analyses, giving a quick impression of genomic patterns without re-checking legend mappings figure by figure.

## Related tools

- **Plotly** (Renders PCA PC coordinates with variance-explained percentages as interactive scatter plots honoring the shared color mapping)
- **Bokeh** (Renders the optimal-K ADMIXTURE Q-matrix as an interactive bar plot with hoverable/controllable elements using the shared color mapping)
- **Toytree** (Renders neighbor-joining trees from IBS and FST distance matrices as interactive trees using the shared color mapping)
- **ete3** (Flags polyphyletic population clusters and can draw TreeMix/consensus trees with the shared color mapping)
- **Biopython** (Builds neighbor-joining trees from the '1-ibs' and pairwise FST distance matrices prior to interactive rendering)
- **PONG** (Visualization of ADMIXTURE Q-matrices for which scalepopgen generates input files using the shared color mapping)
- **Python** (Hosts the plotting scripts that parse analysis outputs and enforce the single color-to-population allocation across all plots) — https://github.com/Popgen48/scalepopgen
- **R** (Provides plotting scripts (e.g., for TreeMix and consensus trees) that consume the same population-color allocation) — https://github.com/Popgen48/scalepopgen

## Examples

```
nextflow run scalepopgen/ -profile test,docker -qs 10
```

## Evaluation signals

- Every emitted HTML artifact renders with hover-over tooltips revealing sample/population information
- Every emitted HTML artifact includes controls to hide or expose dots or bars
- The same population-color mapping appears in every plot (PCA scatter, ADMIXTURE bar plot, IBS/FST trees, TreeMix trees)
- The ADMIXTURE Q-matrix plotted corresponds to the K with the lowest cross-validation error extracted from the log files
- Neighbor-joining trees are correctly reconstructed from the '1-ibs' and pairwise FST distance matrices and polyphyletic population clusters are flagged

## Limitations

- If the user does not supply a color-to-population allocation, one is chosen randomly, so exact colors are not reproducible across runs unless the mapping is recorded
- Interactive rendering depends on specific pinned library versions (Bokeh v3.2.1, Toytree v2.0.5, ete3 v3.1.3, Plotly v5.18.0) and may break with other versions
- Applying a single mapping requires all analyzed samples to share a common population labeling in the sample map

## Evidence

- [methods] Single color allocation across plots: [evidence span withheld — anchor status: not_found]
- [abstract] Python and R visualization scripts: "several Python and R scripts are also provided to collect and visualize the results"
- [methods] PCA plotting with Plotly: [evidence span withheld — anchor status: not_found]
- [methods] Interactive HTML validation criteria: [evidence span withheld — anchor status: not_found]
- [readme] Purpose of combined plotting: "python and R scripts have been developed to combine and plot the results of analyses, which allows user to immediately get an impression about the genomic patterns"

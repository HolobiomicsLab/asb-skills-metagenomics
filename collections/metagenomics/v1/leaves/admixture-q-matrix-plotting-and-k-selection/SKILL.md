---
name: admixture-q-matrix-plotting-and-k-selection
description: Use when ADMIXTURE has been run over multiple user-defined K values on
  PLINK binary (bed/bim/fam) SNP data and the optimal K must be identified via CV
  error and the Q-matrices visualized interactively.
when_to_use_negative:
- Input is raw VCF rather than the Q-matrices/log files produced after running ADMIXTURE
  on the plink-generated binary files — run the genetic-structure sub-workflow first
  (including LD pruning and VCF-to-BED conversion).
- Only a single K was run and no CV errors are available — K selection via CV error
  requires ADMIXTURE runs over multiple user-defined K values.
- The goal is phylogenetic tree rendering or PCA scatter plots — those use TreeMix/Toytree/ete3
  and Plotly outputs, not the ADMIXTURE Q-matrix path.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2815
- http://edamontology.org/topic_0622
tools:
- name: Python
  role: Parses ADMIXTURE log files to extract CV errors per K and drives Q-matrix
    plotting
  repo: https://github.com/Popgen48/scalepopgen
- name: Bokeh
  role: v3.2.1 library used to render the optimal-K Q-matrix as an interactive HTML
    bar plot with hoverable/controllable elements
  repo: https://github.com/Popgen48/scalepopgen
- name: PONG
  role: Receives generated input files for interactive visualization/clustering comparison
    of ADMIXTURE Q-matrices
- name: PLINK
  role: Generates the binary bed/bim/fam files on which ADMIXTURE is run upstream
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/molbev/msae057
    title: 'scalepopgen: Bioinformatic Workflow Resources Implemented in Nextflow
      for Comprehensive Population Genomic Analyses'
  source_content_hash: sha256:6797bd23592f9c8c6858e3204e2941cd582af5c77eb43a9e83aa14aa0aadb865
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Popgen48/scalepopgen
  tools:
  - Biopython
  - Bokeh
  - PLINK
  - PONG
  - Plotly
  - Python
  - R
  - Toytree
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_2815
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# admixture-q-matrix-plotting-and-k-selection

## Summary

Parses ADMIXTURE log files to extract cross-validation (CV) errors per K, selects the K with the lowest CV error, and renders the corresponding Q-matrix as an interactive Bokeh bar plot plus PONG visualization inputs with a consistent population-color mapping.

## When to use

Use when ADMIXTURE has been run over multiple user-defined K values on PLINK binary (bed/bim/fam) SNP data and the optimal K must be identified via CV error and the Q-matrices visualized interactively.

## When NOT to use

- Input is raw VCF rather than the Q-matrices/log files produced after running ADMIXTURE on the plink-generated binary files — run the genetic-structure sub-workflow first (including LD pruning and VCF-to-BED conversion).
- Only a single K was run and no CV errors are available — K selection via CV error requires ADMIXTURE runs over multiple user-defined K values.
- The goal is phylogenetic tree rendering or PCA scatter plots — those use TreeMix/Toytree/ete3 and Plotly outputs, not the ADMIXTURE Q-matrix path.

## Inputs

- ADMIXTURE log files containing cross-validation (CV) errors per K value
- ADMIXTURE Q-matrices for each run K
- Sample map file of individual IDs to population IDs (two tab-delimited columns, .map)
- User-supplied or auto-generated color-to-population allocation

## Outputs

- Interactive HTML bar plot of Q-matrix values for the optimal K (Bokeh v3.2.1, hover tooltips, hide/expose bar controls)
- CV-error plot used to identify a suitable value of K
- Input files for visualization with PONG

## How to apply

Load the ADMIXTURE log files and Q-matrices produced by the genetic-structure sub-workflow on the plink-generated binary files. Parse the log files with Python to extract the cross-validation (CV) error for each user-defined K and identify the K with the lowest CV error; also plot CV error itself as a curve to justify the choice of K. For the optimal K, plot the Q-matrix values as an interactive bar plot using Bokeh v3.2.1 (hoverable/controllable elements) and generate the input files required for visualization with PONG. Apply a single color-to-population allocation (user-supplied or randomly chosen) so the population-color pair stays consistent across the ADMIXTURE bars and all other structure plots (PCA, trees). Each emitted HTML artifact must render with hover-over tooltips revealing sample/population information and controls to hide or expose bars.

## Related tools

- **Python** (Parses ADMIXTURE log files to extract CV errors per K and drives Q-matrix plotting) — https://github.com/Popgen48/scalepopgen
- **Bokeh** (v3.2.1 library used to render the optimal-K Q-matrix as an interactive HTML bar plot with hoverable/controllable elements) — https://github.com/Popgen48/scalepopgen
- **PONG** (Receives generated input files for interactive visualization/clustering comparison of ADMIXTURE Q-matrices)
- **PLINK** (Generates the binary bed/bim/fam files on which ADMIXTURE is run upstream)

## Examples

```
nextflow run scalepopgen/ -profile <docker/singularity/conda/mamba> -params-file <path/to/parameters.yml> -qs 10
```

## Evaluation signals

- The CV-error curve is plotted and the selected K corresponds to the lowest CV error across all user-defined K values.
- The emitted HTML bar plot renders with hover-over tooltips revealing sample/population information.
- The plot includes controls to hide or expose bars interactively.
- The same population-color mapping is used in the ADMIXTURE bar plot and in every other structure plot (PCA, IBS/FST trees).
- PONG input files generated from the Q-matrices load successfully in PONG.

## Limitations

- K selection is bounded by the user-defined K values supplied in the parameter file; ADMIXTURE itself must be run per K on the LD-pruned bed file.
- If no user-supplied color allocation is provided, a random one is chosen, so plot colors are only stable if the same allocation is reused.
- Interactive HTML output depends on pinned library versions (Bokeh v3.2.1) installed automatically by Nextflow via the chosen container/environment profile.

## Evidence

- [full_text] CV-error extraction from ADMIXTURE logs and optimal-K selection: [evidence span withheld — anchor status: not_found]
- [full_text] Bokeh Q-matrix bar plot and PONG input generation: [evidence span withheld — anchor status: not_found]
- [intro] Upstream workflow context in intro: "run ADMIXTURE tool on the bed file"
- [intro] K selection via CV error plotting: "plot cross-validation (CV) error to identify a suitable value of K"
- [intro] PONG/pophelper as Q-matrix plotting tools: "plot Q-matrices of ADMIXTURE outputs using tools such as pong (Behr et al. 2016) and pophelper (Francis 2017)"
- [full_text] Interactive validation criteria: [evidence span withheld — anchor status: not_found]
- [full_text] Consistent color mapping across analyses: [evidence span withheld — anchor status: not_found]
- [readme] README purpose of the plotting scripts: "Additionally, python and R scripts have been developed to combine and plot the results of analyses, which allows user to immediately get an impression about the genomic patterns of the analyzed"

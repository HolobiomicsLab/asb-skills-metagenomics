---
name: phylogenetic-tree-rendering-and-polyphyletic-cluster-detection
description: Use when a population-genomics pipeline has produced IBS or FST distance
  matrices and TreeMix tree files, and you need interactive visual trees plus detection
  of populations that split across clades (polyphyly).
when_to_use_negative:
- No distance matrices or TreeMix tree files exist yet — run the filtering/structure
  or phylogeny sub-workflows first to generate them.
- The analysis goal is admixture proportion plotting (Q-matrices/CV errors) or PCA
  scatter rendering, which use Bokeh, PONG, and Plotly rather than tree rendering.
- Input is raw VCF or PLINK binary data rather than processed distance-matrix/tree
  outputs of the scalepopgen pipeline.
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: Biopython
  role: Builds neighbor-joining trees from the '1-ibs' and pairwise FST distance matrices
- name: Toytree
  role: Renders the neighbor-joining trees as interactive trees (v2.0.5)
- name: ete3
  role: Flags polyphyletic population clusters and alternatively draws TreeMix/consensus
    trees (v3.1.3)
- name: TreeMix
  role: Upstream producer of tree files (with/without bootstrapping and migration
    edges) rendered by this skill
- name: R
  role: Provides plotting scripts for drawing TreeMix and consensus trees
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/molbev/msae057
    title: 'scalepopgen: Bioinformatic Workflow Resources Implemented in Nextflow
      for Comprehensive Population Genomic Analyses'
  source_content_hash: sha256:64dac4f7e65949e78581f1d0fa4fc4baf8d9286d9e8cdfa165494ac41bad1849
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Biopython
  - Bokeh
  - PONG
  - Plotly
  - Python
  - R
  - Toytree
  - TreeMix
  - ete3
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# phylogenetic-tree-rendering-and-polyphyletic-cluster-detection

## Summary

Builds neighbor-joining trees from scalepopgen's '1-ibs' and pairwise-FST distance matrices plus TreeMix tree files, renders them as interactive HTML trees with Toytree/ete3, and flags polyphyletic population clusters so users can immediately assess genomic patterns after pipeline execution.

## When to use

Use when a population-genomics pipeline has produced IBS or FST distance matrices and TreeMix tree files, and you need interactive visual trees plus detection of populations that split across clades (polyphyly).

## When NOT to use

- No distance matrices or TreeMix tree files exist yet — run the filtering/structure or phylogeny sub-workflows first to generate them.
- The analysis goal is admixture proportion plotting (Q-matrices/CV errors) or PCA scatter rendering, which use Bokeh, PONG, and Plotly rather than tree rendering.
- Input is raw VCF or PLINK binary data rather than processed distance-matrix/tree outputs of the scalepopgen pipeline.

## Inputs

- '1-ibs' identity-by-state distance matrix from upstream structure analysis
- pairwise population FST distance matrix
- TreeMix tree files (with or without bootstrapping and migration edges)
- sample map file with tab-delimited individual ID and population ID columns (.map)

## Outputs

- interactive neighbor-joining tree HTML plots of IBS distances (Toytree)
- interactive neighbor-joining tree HTML plots of FST distances (Toytree)
- rendered TreeMix and consensus trees (R plotting scripts or ete3)
- flags/annotations identifying polyphyletic population clusters (ete3)
- consistent color-to-population mapping shared across all report plots

## How to apply

Load the upstream sub-workflow outputs: the '1-ibs' distance matrix, the pairwise FST distance matrix, and TreeMix tree files (with or without bootstrapping and migration edges). Build neighbor-joining trees from the IBS and FST matrices using the Biopython module, then render them as interactive trees with Toytree v2.0.5 so users can quickly interpret the data. Flag polyphyletic population clusters — populations whose individuals do not group into a single clade — using ete3 v3.1.3. Draw the TreeMix and consensus trees with the provided R plotting scripts or with ete3. Apply the single color-to-population allocation (user-supplied or randomly chosen) so the same population-color pair appears in every plot of the report; each emitted HTML artifact must support hover-over tooltips revealing sample/population information and controls to hide or expose elements.

## Related tools

- **Biopython** (Builds neighbor-joining trees from the '1-ibs' and pairwise FST distance matrices)
- **Toytree** (Renders the neighbor-joining trees as interactive trees (v2.0.5))
- **ete3** (Flags polyphyletic population clusters and alternatively draws TreeMix/consensus trees (v3.1.3))
- **TreeMix** (Upstream producer of tree files (with/without bootstrapping and migration edges) rendered by this skill)
- **R** (Provides plotting scripts for drawing TreeMix and consensus trees)

## Examples

```
nextflow run scalepopgen/ -profile test,docker -qs 10
```

## Evaluation signals

- Each emitted HTML tree artifact renders with hover-over tooltips revealing sample/population information and controls to hide or expose elements.
- Neighbor-joining trees are reconstructed from both the '1-ibs' and the pairwise FST distance matrices, not just one.
- Polyphyletic population clusters are explicitly flagged by ete3 and consistent with individual membership in the sample map.
- The same color-to-population mapping appears in the trees as in every other plot of the report (PCA scatters, ADMIXTURE bars).
- TreeMix and consensus trees are drawn using the provided R plotting scripts or ete3 without visual inconsistencies across the report.

## Limitations

- Tree rendering depends on the phylogeny sub-workflow (TreeMix run with or without bootstrapping and migration edges) having completed upstream; it cannot run on raw VCF/PLINK inputs.
- Interpretability of polyphyly flags depends on prior sample and site filtering — in the paper's cattle benchmark, filtering reduced 442 samples (30 breeds, ~25M SNPs) to 269 samples (17 breeds, ~23.4M SNPs) before analyses.
- Interactive visualization requires the pinned library versions of the pipeline (e.g., Toytree v2.0.5, ete3 v3.1.3); other versions may not reproduce the hover/control behavior.

## Evidence

- [abstract] several Python and R scripts are also provided to collect and visualize the results: "several Python and R scripts are also provided to collect and visualize the results"
- [abstract] running TreeMix analysis with or without bootstrapping and migration edges: "running TreeMix analysis with or without bootstrapping and migration edges, followed by identification of an optimal number of migration edges"
- [readme] python and R scripts developed to combine and plot results: "Additionally, python and R scripts have been developed to combine and plot the results of analyses, which allows user to immediately get an impression about the genomic patterns of the analyzed"
- [readme] sample map defines individual-to-population membership: [evidence span withheld — anchor status: external_doc]

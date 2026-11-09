---
name: phylogenetic-tree-visualization-ete3
description: Use when running or reproducing the scalepopgen TreeMix phylogeny sub-workflow
  and you need to plot the inferred population trees, bootstrap-supported consensus
  trees, or migration-edge results from filtered biallelic SNP data.
when_to_use_negative:
- The analysis goal is exploring genetic structure (PCA, ADMIXTURE with optimal K-values)
  rather than inferring and visualizing population trees
- The input is not genome-wide biallelic SNP data in VCF or PLINK binary format as
  required by scalepopgen
- You need to visualize individual genealogies rather than population-level trees
  with migration edges from TreeMix
edam_operation: http://edamontology.org/operation_0325
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_0622
tools:
- name: ete3
  role: Python module (v 3.1.3) used to plot the PHYLIP CONSENSE consensus trees in
    the phylogeny sub-workflow
- name: TreeMix
  role: Infers the population trees and pairwise migration events (with or without
    bootstrapping, v 1.13); its bundled R scripts plot the resulting trees
- name: PHYLIP CONSENSE
  role: Collapses bootstrap replicate trees into the consensus trees that ete3 renders
    (v 3.697)
- name: OptM
  role: Summarizes TreeMix runs to identify the optimal number of migration edges
    via Δm (v 0.1.6), contextualizing the plotted trees
- name: Nextflow
  role: Workflow manager that orchestrates the 'phylogeny using treemix' sub-workflow,
    including the visualization steps
  repo: https://github.com/Popgen48/scalepopgen
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/molbev/msae057
    title: 'scalepopgen: Bioinformatic Workflow Resources Implemented in Nextflow
      for Comprehensive Population Genomic Analyses'
  source_content_hash: sha256:e7bff6679363e29ec2379a6dc55ab23fdd30bddc6c2f23881682045ccb3b2d18
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Popgen48/scalepopgen
  tools:
  - Nextflow
  - OptM
  - PHYLIP CONSENSE
  - Python
  - R
  - TreeMix
  - ete3
  edam_operation: http://edamontology.org/operation_0325
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# phylogenetic-tree-visualization-ete3

## Summary

Visualizes the population trees inferred by scalepopgen's 'phylogeny using treemix' sub-workflow: TreeMix-generated trees are plotted with the R scripts bundled with TreeMix, and bootstrap-collapsed consensus trees (PHYLIP CONSENSE) are rendered with the Python module ete3 (v 3.1.3).

## When to use

Use when running or reproducing the scalepopgen TreeMix phylogeny sub-workflow and you need to plot the inferred population trees, bootstrap-supported consensus trees, or migration-edge results from filtered biallelic SNP data.

## When NOT to use

- The analysis goal is exploring genetic structure (PCA, ADMIXTURE with optimal K-values) rather than inferring and visualizing population trees
- The input is not genome-wide biallelic SNP data in VCF or PLINK binary format as required by scalepopgen
- You need to visualize individual genealogies rather than population-level trees with migration edges from TreeMix

## Inputs

- Filtered biallelic SNP genotype VCF (chromosome-split, tabix-indexed, listed in a vcf_input.csv sheet)
- Tab-delimited sample map (.map) of individual IDs to population IDs
- YAML parameters file configuring bootstrapping and migration-edge counts for the TreeMix sub-workflow
- TreeMix tree output files from the TreeMix (v 1.13) runs
- Bootstrap replicate trees and their PHYLIP (v 3.697) CONSENSE consensus tree
- OptM (v 0.1.6) summary of TreeMix runs

## Outputs

- Phylogenetic tree figures for the TreeMix population trees (produced by the R scripts bundled with TreeMix)
- Consensus tree figures rendered with ete3 (v 3.1.3)
- OptM summary identifying a discrete optimal number of migration edges

## How to apply

First transform the filtered VCF genotype file to TreeMix-compatible input using the in-house Python conversion script, with bootstrapping and user-defined migration edges configured via the YAML parameters file. Run TreeMix (v 1.13) to infer population trees and estimate pairwise migration events, then summarize the runs with OptM (v 0.1.6), which identifies the optimal number of migration edges via the second-order rate of change in likelihood (Δm). Collapse the bootstrap replicate trees into consensus trees with the PHYLIP (v 3.697) CONSENSE package. Plot the resulting phylogenetic trees with the R scripts bundled with TreeMix, and plot the consensus trees with ete3 (v 3.1.3). Validation consists of confirming that the consensus tree figure is produced and that the OptM summary identifies a discrete optimal number of migration edges for the population tree.

## Related tools

- **ete3** (Python module (v 3.1.3) used to plot the PHYLIP CONSENSE consensus trees in the phylogeny sub-workflow)
- **TreeMix** (Infers the population trees and pairwise migration events (with or without bootstrapping, v 1.13); its bundled R scripts plot the resulting trees)
- **PHYLIP CONSENSE** (Collapses bootstrap replicate trees into the consensus trees that ete3 renders (v 3.697))
- **OptM** (Summarizes TreeMix runs to identify the optimal number of migration edges via Δm (v 0.1.6), contextualizing the plotted trees)
- **Nextflow** (Workflow manager that orchestrates the 'phylogeny using treemix' sub-workflow, including the visualization steps) — https://github.com/Popgen48/scalepopgen

## Examples

```
nextflow run scalepopgen/ -profile test,singularity -qs 10
```

## Evaluation signals

- The consensus tree figure is produced as an output of the sub-workflow
- The OptM summary identifies a discrete optimal number of migration edges for the population tree
- TreeMix tree plots generated by the bundled R scripts correspond to the configured number of user-defined migration edges
- ete3 (v 3.1.3) renders the PHYLIP (v 3.697) CONSENSE consensus tree without errors, reflecting the bootstrap-collapsed topology

## Limitations

- VCF inputs must be split by chromosome, tabix-indexed, and chromosome names must not contain any punctuation marks, or the upstream steps feeding the trees will fail
- A sample map file (.map, two tab-delimited columns without header) is mandatory to assign individuals to populations before population trees can be inferred
- The sub-workflow targets biallelic SNP data only; multiallelic or non-SNP variants are outside scalepopgen's scope

## Evidence

- [abstract] TreeMix analysis with or without bootstrapping and migration edges, followed by identification of an optimal number of migration edges: "running TreeMix analysis with or without bootstrapping and migration edges, followed by identification of an optimal number of migration edges"
- [abstract] python and R scripts developed to combine and plot results: "several Python and R scripts are also provided to collect and visualize the results"
- [readme] README: pipeline description of plotting scripts: "python and R scripts have been developed to combine and plot the results of analyses"
- [readme] README: sub-workflows usable independently: "The sub-workflows can be used separately or in combination with each other."

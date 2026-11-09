---
name: migration-edge-optimization-optm
description: Use when TreeMix has been run across multiple user-defined migration-edge
  values (optionally with bootstrapping) and you must choose the optimal number of
  migration edges for the inferred population tree from biallelic SNP data.
when_to_use_negative:
- The analysis goal is genome-wide population structure (PCA/admixture with optimal
  K) rather than tree-based relationships with migration — use the 'explore genetic
  structure' sub-workflow instead.
- The input is unfiltered or multi-allelic variant data — TreeMix input must come
  from the filtered biallelic SNP set produced upstream in scalepopgen.
- Only a single TreeMix run at one fixed migration-edge count was performed — OptM
  needs runs across multiple edge counts to compute Δm.
edam_operation: http://edamontology.org/operation_0558
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3299
tools:
- name: TreeMix
  role: Runs the population-tree inference with bootstrapping and user-defined migration
    edges on converted biallelic SNP input (v 1.13)
  repo: https://github.com/Popgen48/scalepopgen
- name: OptM
  role: Summarizes TreeMix runs to identify the optimal number of migration edges
    via the second-order rate of change in likelihood (Δm) (v 0.1.6)
- name: PHYLIP
  role: CONSENSE package collapses bootstrap replicate trees into consensus trees
    (v 3.697)
- name: ete3
  role: Python module used to plot the consensus phylogenetic trees (v 3.1.3)
- name: Nextflow
  role: Workflow management system orchestrating the scalepopgen phylogeny sub-workflow
  repo: https://github.com/Popgen48/scalepopgen
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/molbev/msae057
    title: 'scalepopgen: Bioinformatic Workflow Resources Implemented in Nextflow
      for Comprehensive Population Genomic Analyses'
  source_content_hash: sha256:8798a2c3d968719c942ec2c8293d44480851883965474b33de03eb080843c5ad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Popgen48/scalepopgen
  tools:
  - Nextflow
  - OptM
  - PHYLIP
  - Python
  - R
  - TreeMix
  - ete3
  edam_operation: http://edamontology.org/operation_0558
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# migration-edge-optimization-optm

## Summary

Summarize replicate TreeMix runs with the OptM package to identify the optimal number of migration edges on a population tree, using the second-order rate of change in likelihood (Δm) across models with different edge counts.

## When to use

Use when TreeMix has been run across multiple user-defined migration-edge values (optionally with bootstrapping) and you must choose the optimal number of migration edges for the inferred population tree from biallelic SNP data.

## When NOT to use

- The analysis goal is genome-wide population structure (PCA/admixture with optimal K) rather than tree-based relationships with migration — use the 'explore genetic structure' sub-workflow instead.
- The input is unfiltered or multi-allelic variant data — TreeMix input must come from the filtered biallelic SNP set produced upstream in scalepopgen.
- Only a single TreeMix run at one fixed migration-edge count was performed — OptM needs runs across multiple edge counts to compute Δm.

## Inputs

- Filtered biallelic SNP genotype VCF (chromosome-split, tabix-indexed) from the scalepopgen filtering sub-workflow
- TreeMix-compatible input converted from the filtered VCF via the in-house Python conversion script
- TreeMix run outputs across multiple migration-edge values (with bootstrapping)
- scalepopgen YAML parameters file specifying bootstrapping and user-defined migration edges

## Outputs

- OptM summary identifying a discrete optimal number of migration edges
- TreeMix population trees with inferred pairwise migration events
- Consensus trees from PHYLIP CONSENSE over bootstrap replicates
- Plotted phylogenetic trees (TreeMix R scripts for TreeMix trees; ete3 for consensus trees)

## How to apply

First run TreeMix (v 1.13) on TreeMix-compatible input converted from the filtered VCF genotype file via the in-house Python conversion script, executing multiple runs with bootstrapping and varying numbers of migration edges configured through the YAML parameters file. Then feed the TreeMix run outputs to OptM (v 0.1.6), which estimates the optimal number of migration edges via the second-order rate of change in likelihood (Δm) across the edge-count models. Collapse bootstrap replicate trees into consensus trees with the PHYLIP (v 3.697) CONSENSE package, and plot the TreeMix trees with the bundled R scripts and the consensus trees with ete3 (v 3.1.3). The rationale is that Δm penalizes both under- and over-fitting of migration events, giving a discrete optimum rather than picking the maximum-likelihood edge count directly.

## Related tools

- **TreeMix** (Runs the population-tree inference with bootstrapping and user-defined migration edges on converted biallelic SNP input (v 1.13)) — https://github.com/Popgen48/scalepopgen
- **OptM** (Summarizes TreeMix runs to identify the optimal number of migration edges via the second-order rate of change in likelihood (Δm) (v 0.1.6))
- **PHYLIP** (CONSENSE package collapses bootstrap replicate trees into consensus trees (v 3.697))
- **ete3** (Python module used to plot the consensus phylogenetic trees (v 3.1.3))
- **Nextflow** (Workflow management system orchestrating the scalepopgen phylogeny sub-workflow) — https://github.com/Popgen48/scalepopgen

## Examples

```
nextflow run scalepopgen/ -profile <docker/singularity/conda/mamba> -params-file <path/to/parameters.yml> -qs 10
```

## Evaluation signals

- The OptM summary identifies a discrete optimal number of migration edges for the population tree (an explicit validation step in the paper).
- The consensus tree figure is produced (an explicit validation step in the paper).
- TreeMix input matches the TreeMix-compatible format produced by the in-house Python conversion script from the filtered VCF.
- Bootstrap replicates were collapsed by PHYLIP CONSENSE into a single consensus tree without errors.
- The OptM-selected edge count corresponds to a maximum (or stable point) in the Δm second-order rate-of-change statistic across the tested edge values.

## Limitations

- The number of migration edges tested is user-defined via the YAML parameters file, so a poor choice of tested range can miss the true optimum.
- OptM requires TreeMix runs across multiple migration-edge values; a single run cannot be summarized.
- Chromosome names in the input VCF must not contain punctuation marks, per the README input requirements.

## Evidence

- [full_text] The phylogeny sub-workflow runs TreeMix analysis with or without bootstrapping and migration edges, and then identifies an optimal number of migration edges; OptM estimates this optimum via the second-order rate of change in likelihood (Δm).: [evidence span withheld — anchor status: not_found]
- [full_text] TreeMix/OptM/PHYLIP pipeline step: [evidence span withheld — anchor status: not_found]
- [abstract] Abstract sub-workflow description: "(iii) running TreeMix analysis with or without bootstrapping and migration edges, followed by identification of an optimal number of migration edges"
- [readme] README sub-workflow listing: "Broadly, the pipeline consists of the following four “sub-workflows”:</p> <ul> <li>filtering and basic statistics</li> <li>explore genetic structure</li> <li>phylogeny using treemix</li>"
- [full_text] Validation step: [evidence span withheld — anchor status: not_found]

---
name: bootstrap-consensus-tree-construction
description: Use when you need to infer population relationships and migration events
  from a filtered VCF of biallelic SNPs, with bootstrapped support and a consensus
  tree across bootstrap replicates.
when_to_use_negative:
- Data have not passed sample- and site-level filtering (missingness, MAF) — the sub-workflow
  expects filtered biallelic SNP genotype input
- The goal is individual-level phylogeny or species-level tree building rather than
  population-tree inference with migration edges
- You need admixture proportions or PCA rather than a treemix-style population tree
  — use the 'explore genetic structure' sub-workflow instead
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3299
tools:
- name: TreeMix
  role: Runs with bootstrapping and migration edges to infer population trees and
    estimate pairwise migration events (v 1.13)
  repo: https://github.com/Popgen48/scalepopgen
- name: OptM
  role: Summarizes TreeMix runs to identify the optimal number of migration edges
    via Δm, the second-order rate of change in likelihood (v 0.1.6)
  repo: https://github.com/Popgen48/scalepopgen
- name: PHYLIP
  role: CONSENSE package collapses bootstrap replicate trees into consensus trees
    (v 3.697)
- name: ete3
  role: Python module used to plot the consensus trees (v 3.1.3)
- name: Nextflow
  role: Workflow management system orchestrating the phylogeny sub-workflow steps
  repo: https://github.com/Popgen48/scalepopgen
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/molbev/msae057
    title: 'scalepopgen: Bioinformatic Workflow Resources Implemented in Nextflow
      for Comprehensive Population Genomic Analyses'
  source_content_hash: sha256:9df94cbdd598a9833e358eb02006ee24114bcb35e28acae68bdc8fd4921017a4
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
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# bootstrap-consensus-tree-construction

## Summary

Constructs population phylogenies from filtered biallelic SNP data by running TreeMix with bootstrapping and migration edges, collapsing bootstrap replicate trees into a consensus tree with PHYLIP CONSENSE, and identifying the optimal number of migration edges via OptM's Δm (second-order rate of change in likelihood).

## When to use

Use when you need to infer population relationships and migration events from a filtered VCF of biallelic SNPs, with bootstrapped support and a consensus tree across bootstrap replicates.

## When NOT to use

- Data have not passed sample- and site-level filtering (missingness, MAF) — the sub-workflow expects filtered biallelic SNP genotype input
- The goal is individual-level phylogeny or species-level tree building rather than population-tree inference with migration edges
- You need admixture proportions or PCA rather than a treemix-style population tree — use the 'explore genetic structure' sub-workflow instead

## Inputs

- Filtered biallelic SNP genotype data in VCF format (chromosome-split and tabix-indexed)
- Sample map file (tab-delimited, individual IDs and population IDs, '.map' extension, no header)
- YAML parameters file with bootstrapping and migration-edge settings

## Outputs

- TreeMix population trees with inferred pairwise migration events
- OptM summary identifying a discrete optimal number of migration edges (Δm)
- Consensus trees from PHYLIP CONSENSE across bootstrap replicates
- Tree figures plotted with TreeMix R scripts and ete3

## How to apply

First transform the filtered VCF genotype file to TreeMix-compatible input using the in-house Python conversion script, configuring bootstrapping and user-defined migration edges via the YAML parameters file. Run TreeMix (v 1.13) with bootstrapping and migration edges to infer population trees and estimate pairwise migration events, then summarize the runs with OptM (v 0.1.6), which estimates the optimal number of migration edges via the second-order rate of change in likelihood (Δm). Collapse the bootstrap replicate trees into consensus trees using the PHYLIP (v 3.697) CONSENSE package, and plot the phylogenetic trees with the R scripts bundled with TreeMix and the consensus trees with the Python module ete3 (v 3.1.3).

## Related tools

- **TreeMix** (Runs with bootstrapping and migration edges to infer population trees and estimate pairwise migration events (v 1.13)) — https://github.com/Popgen48/scalepopgen
- **OptM** (Summarizes TreeMix runs to identify the optimal number of migration edges via Δm, the second-order rate of change in likelihood (v 0.1.6)) — https://github.com/Popgen48/scalepopgen
- **PHYLIP** (CONSENSE package collapses bootstrap replicate trees into consensus trees (v 3.697))
- **ete3** (Python module used to plot the consensus trees (v 3.1.3))
- **Nextflow** (Workflow management system orchestrating the phylogeny sub-workflow steps) — https://github.com/Popgen48/scalepopgen

## Examples

```
nextflow run scalepopgen/ -profile <docker/singularity/conda/mamba> -params-file <path/to/parameters.yml> -qs <maximum number of processes>
```

## Evaluation signals

- The consensus tree figure is produced successfully
- The OptM summary identifies a discrete optimal number of migration edges for the population tree
- Bootstrap replicate trees were generated for each TreeMix run and successfully collapsed by CONSENSE
- The TreeMix-compatible input file was correctly generated from the filtered VCF by the conversion script
- Result plots are generated by both the TreeMix R scripts and ete3

## Limitations

- Chromosome names in the VCF input must not contain punctuation marks
- VCF files must be split by chromosome and indexed with tabix before running the pipeline
- TreeMix migration edges are user-defined per run; the optimum is estimated post hoc from the runs, so a suitable range of edge values must be supplied

## Evidence

- [abstract] The phylogeny sub-workflow runs TreeMix analysis with or without bootstrapping and migration edges, and then identifies an optimal number of migration edges; OptM estimates this optimum via the second-order rate of change in likelihood (Δm).: "running TreeMix analysis with or without bootstrapping and migration edges, followed by identification of an optimal number of migration edges"
- [methods] Workflow described in the source card for the phylogeny sub-workflow, including CONSENSE and ete3 steps.: [evidence span withheld — anchor status: not_found]
- [results] Validation criteria stated in the source card for the phylogeny sub-workflow.: [evidence span withheld — anchor status: not_found]
- [readme] README listing the phylogeny sub-workflow among the four sub-workflows.: "phylogeny using treemix"
- [readme] README describing the sample map format required as input alongside the VCF files.: [evidence span withheld — anchor status: external_doc]

---
name: population-phylogeny-inference-treemix
description: Reference for inferring population phylogenetic relationships and migration
  events from filtered genome-wide biallelic SNP genotype data (VCF or PLINK binary)
  using TreeMix within a Nextflow workflow.
when_to_use_negative:
- The input data have not yet passed the scalepopgen filtering sub-workflow (sample
  filtering, site filtering for missingness/MAF, and optional LD pruning) — run filtering
  and basic statistics first
- The analysis goal is population structure/admixture proportions rather than tree
  topology with migration edges — the explore-genetic-structure sub-workflow (PCA,
  ADMIXTURE with optimal K via CV error) is the appropriate choice
- Data are not biallelic SNPs in VCF or PLINK binary form; scalepopgen's sub-workflows
  are designed specifically for biallelic SNP data in these formats
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0294
- http://edamontology.org/topic_0102
tools:
- name: Nextflow
  role: Workflow manager orchestrating the phylogeny sub-workflow processes and tool
    dependencies
  repo: https://github.com/Popgen48/scalepopgen
- name: Python
  role: In-house conversion script transforms the filtered VCF to TreeMix-compatible
    input; ete3 (v 3.1.3) plots consensus trees
  repo: https://github.com/Popgen48/scalepopgen
- name: TreeMix
  role: 'Core inference engine (v 1.13): infers population trees and estimates pairwise
    migration events, with optional bootstrapping and migration edges'
- name: OptM
  role: Summarizes TreeMix runs and identifies the optimal number of migration edges
    via the second-order rate of change in likelihood (Δm); v 0.1.6
- name: PHYLIP
  role: CONSENSE package (v 3.697) collapses bootstrap replicate trees into consensus
    trees
- name: R
  role: R scripts bundled with TreeMix plot the resulting phylogenetic trees
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/molbev/msae057
    title: 'scalepopgen: Bioinformatic Workflow Resources Implemented in Nextflow
      for Comprehensive Population Genomic Analyses'
  source_content_hash: sha256:a68511a232aa9ffbdb5db72117511d489448cb4954b57456395259bf098177ef
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
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0294
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# population-phylogeny-inference-treemix

## Summary

Runs scalepopgen's 'phylogeny using treemix' sub-workflow: it converts filtered biallelic SNP VCF data to TreeMix input, infers population trees with TreeMix (v1.13) with optional bootstrapping and migration edges, selects the optimal number of migration edges with OptM (v0.1.6) via Δm (second-order rate of change in likelihood), collapses bootstrap replicates into consensus trees with PHYLIP CONSENSE (v3.697), and plots trees with TreeMix R scripts and ete3 (v3.1.3).

## When to use

Reference for inferring population phylogenetic relationships and migration events from filtered genome-wide biallelic SNP genotype data (VCF or PLINK binary) using TreeMix within a Nextflow workflow.

## When NOT to use

- The input data have not yet passed the scalepopgen filtering sub-workflow (sample filtering, site filtering for missingness/MAF, and optional LD pruning) — run filtering and basic statistics first
- The analysis goal is population structure/admixture proportions rather than tree topology with migration edges — the explore-genetic-structure sub-workflow (PCA, ADMIXTURE with optimal K via CV error) is the appropriate choice
- Data are not biallelic SNPs in VCF or PLINK binary form; scalepopgen's sub-workflows are designed specifically for biallelic SNP data in these formats

## Inputs

- Filtered biallelic SNP genotype data in VCF format (chromosome-split, tabix-indexed, listed in a vcf_input.csv sheet) or PLINK binary (.bed/.bim/.fam) format listed in a .p.csv sheet
- Tab-delimited sample map file (.map) mapping individual IDs to population IDs (no header)
- YAML parameters file specifying TreeMix options (bootstrapping, number of migration edges), produced via the scalepopgen CLI

## Outputs

- TreeMix population tree figures and inferred pairwise migration events
- OptM summary identifying a discrete optimal number of migration edges (Δm)
- Consensus tree built from bootstrap replicate trees (PHYLIP CONSENSE)
- Plotted phylogenetic trees: TreeMix trees via bundled R scripts and consensus trees via ete3

## How to apply

First transform the filtered VCF genotype file to TreeMix-compatible input using the in-house Python conversion script, configuring bootstrapping and user-defined migration edges through the YAML parameters file generated by the scalepopgen CLI. Run TreeMix (v 1.13) to infer population trees and estimate pairwise migration events, optionally with bootstrapping. Summarize multiple TreeMix runs with OptM (v 0.1.6), which identifies the optimal number of migration edges via the second-order rate of change in likelihood (Δm). If bootstrapping was used, collapse replicate trees into consensus trees with the PHYLIP (v 3.697) CONSENSE package, then plot TreeMix trees with the bundled R scripts and consensus trees with the Python module ete3 (v 3.1.3). The rationale is that population-level relationships and gene flow (migration edges) require nonstandard input formats and multistep orchestration that scalepopgen automates under Nextflow.

## Related tools

- **Nextflow** (Workflow manager orchestrating the phylogeny sub-workflow processes and tool dependencies) — https://github.com/Popgen48/scalepopgen
- **Python** (In-house conversion script transforms the filtered VCF to TreeMix-compatible input; ete3 (v 3.1.3) plots consensus trees) — https://github.com/Popgen48/scalepopgen
- **TreeMix** (Core inference engine (v 1.13): infers population trees and estimates pairwise migration events, with optional bootstrapping and migration edges)
- **OptM** (Summarizes TreeMix runs and identifies the optimal number of migration edges via the second-order rate of change in likelihood (Δm); v 0.1.6)
- **PHYLIP** (CONSENSE package (v 3.697) collapses bootstrap replicate trees into consensus trees)
- **R** (R scripts bundled with TreeMix plot the resulting phylogenetic trees)

## Examples

```
nextflow run scalepopgen/ -profile <docker/singularity/conda/mamba> -params-file <path/to/parameters.yml> -qs <maximum number of processes>
```

## Evaluation signals

- The consensus tree figure is produced (explicit validation check of the sub-workflow)
- The OptM summary identifies a discrete optimal number of migration edges for the population tree (explicit validation check)
- TreeMix input was generated from the filtered VCF via the in-house Python conversion script without format errors, using parameters from the YAML file
- Bootstrap replicate trees were successfully collapsed by CONSENSE into a single consensus tree when bootstrapping was enabled
- The Nextflow run completes the phylogeny sub-workflow reproducibly under the chosen profile (conda/mamba/singularity/docker), as reported in the pipeline HTML report

## Limitations

- Migration-edge count must be either user-defined in the YAML parameters file or inferred by OptM; a discrete optimum may not emerge for all datasets
- Requires input VCFs split by chromosome and tabix-indexed, with chromosome names containing no punctuation marks
- Users must supply a population map; the workflow operates on population-level relationships, so sample-to-population assignments must be accurate beforehand

## Evidence

- [abstract] running TreeMix analysis with or without bootstrapping and migration edges, followed by identification of an optimal number of migration edges: "running TreeMix analysis with or without bootstrapping and migration edges, followed by identification of an optimal number of migration edges"
- [intro] these workflows have the features to filter the data, carry out the analyses to explore the genetic structure and phylogenetic relationships between the populations: "these workflows have the features to filter the data, carry out the analyses to explore the genetic structure and phylogenetic relationships between the populations"
- [readme] The sub-workflows can be used separately or in combination with each other.: "The sub-workflows can be used separately or in combination with each other."
- [readme] All VCF files need to be splitted by the chromosomes and indexed with tabix.: [evidence span withheld — anchor status: external_doc]

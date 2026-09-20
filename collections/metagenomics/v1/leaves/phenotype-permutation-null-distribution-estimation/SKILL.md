---
name: phenotype-permutation-null-distribution-estimation
description: Use when the empirical null distribution of minimum P-values is needed
  for the phylogenetic (clade-level) MWAS after Step 4-1, to set a Psig threshold
  that controls type I error under the study's actual clade correlation structure.
when_to_use_negative:
- Input is a continuous or non-binary phenotype rather than the case-control labels
  (1 = case, 0 = control) that the sample list's Phenotype column must contain
- Only raw QCed FASTQ or abundance tables are available and the Step 4-1 phylogenetic
  association summary statistics (sumstats_*.txt) have not yet been generated
- The goal is direct adjustment of hidden confounding factors by integrating covariates
  into the case-control model — the article notes this is not currently implemented
  in OMARU
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3053
tools:
- name: OMARU
  role: Pipeline whose Snakefile reruns the clade-level association test on permuted
    phenotype labels and integrates permutation results into sumstats_*_annot.txt
  repo: https://github.com/toshi-kishikawa/OMARU
- name: Snakemake
  role: Workflow engine executing OMARU_Phyl_permutation_visualization.sm for the
    permutation and visualization step
  repo: https://snakemake.readthedocs.io
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqac019
    title: 'OMARU: a robust and multifaceted pipeline for metagenome-wide association
      study'
  source_content_hash: sha256:5ed4a1170b6855f282a0ba75ace60ad8662341923c265762ededc896725eb192
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/toshi-kishikawa/OMARU
  repo_url: https://github.com/toshi-kishikawa/OMARU
  tools:
  - OMARU
  - Snakemake
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# phenotype-permutation-null-distribution-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimate an empirical Bonferroni significance threshold (Psig) for OMARU clade-level case-control association tests by permuting binary phenotype labels 10,000 times and building the null distribution of the minimum P-value, with a Harrell-Davis bootstrap 95% CI reported for Pmin.

## When to use

Use when the empirical null distribution of minimum P-values is needed for the phylogenetic (clade-level) MWAS after Step 4-1, to set a Psig threshold that controls type I error under the study's actual clade correlation structure.

## When NOT to use

- Input is a continuous or non-binary phenotype rather than the case-control labels (1 = case, 0 = control) that the sample list's Phenotype column must contain
- Only raw QCed FASTQ or abundance tables are available and the Step 4-1 phylogenetic association summary statistics (sumstats_*.txt) have not yet been generated
- The goal is direct adjustment of hidden confounding factors by integrating covariates into the case-control model — the article notes this is not currently implemented in OMARU

## Inputs

- sumstats_*.txt phylogenetic association summary statistics from Step 4-1 (<OMARU_project_dir>/result/PHYL_QCed3/<Phylogenetic_reference>_association_test/<covariates>)
- original_sample_list.txt / QCed3_sample_list.txt with Sample, Phenotype (1 = case, 0 = control), Sex, Age columns in <OMARU_project_dir>/data
- covariates.txt (default sex and age; optional covariates<SUFFIX_COV>.txt)
- config.yaml parameters (N_PCs, N_PC_PHYL default 2, SUFFIX_COV)

## Outputs

- sumstats_*_annot.txt integrated summary statistics with permutation results and annotation in <OMARU_project_dir>/result/PHYL_QCed3/<Phylogenetic_reference>_association_test/<covariates>
- P-value distribution and QQ figures in result/PHYL_QCed3/<Phylogenetic_reference>_association_graph
- Phylogenetic (ggtree) tree annotated with association results in result/PHYL_QCed3/<Phylogenetic_reference>_ggtree/<covariates>

## How to apply

Load the phylogenetic association summary statistics (sumstats_*.txt) from Step 4-1 and the phenotype sample list (original_sample_list.txt / QCed3_sample_list.txt) from <OMARU_project_dir>/data, with covariates.txt defining sex and age as base covariates and N_PC_PHYL (default 2) principal components in config.yaml. Permute the binary phenotype labels (1 = case, 0 = control) 10,000 times, rerunning the clade-level association test on each permuted labeling and recording the minimum P-value per iteration to build the empirical null distribution. Define the empirical Bonferroni threshold Psig as the 95th percentile of this permutated minimum-P-value distribution and estimate its uncertainty with a Harrell-Davis bootstrap 95% confidence interval. Run the test with and without PC covariates to contrast P-value inflation, then execute `snakemake -s Snakefiles/OMARU_Phyl_permutation_visualization.sm`, which writes integrated summary statistics (sumstats_*_annot.txt) and P-value distribution/QQ figures. The rationale is that standard Bonferroni correction ignores correlation among clades, so the permutation-based threshold empirically calibrates multiple testing; in the SCZ analysis it fell below the standard Bonferroni threshold.

## Related tools

- **OMARU** (Pipeline whose Snakefile reruns the clade-level association test on permuted phenotype labels and integrates permutation results into sumstats_*_annot.txt) — https://github.com/toshi-kishikawa/OMARU
- **Snakemake** (Workflow engine executing OMARU_Phyl_permutation_visualization.sm for the permutation and visualization step) — https://snakemake.readthedocs.io

## Examples

```
cd OMARU_project_dir && snakemake -s Snakefiles/OMARU_Phyl_permutation_visualization.sm
```

## Evaluation signals

- The empirical Bonferroni threshold (95th percentile of the permutation minimum-P-value distribution) falls below the standard Bonferroni threshold, as reported for the SCZ analysis
- The QQ/distribution figure shows P-value inflation in the no-PC-covariate run relative to the PC-adjusted (e.g., N_PC_PHYL = 2) run
- sumstats_*_annot.txt files are present in <OMARU_project_dir>/result/PHYL_QCed3/<Phylogenetic_reference>_association_test/<covariates> after the workflow completes
- The reported 95% CI for Pmin is generated with the Harrell-Davis bootstrap estimator
- Exactly 10,000 permutation iterations of the association test are recorded in the empirical null distribution

## Limitations

- Hidden confounding factors cannot be directly adjusted by integrating covariates into the case-control model; this is explicitly noted as not implemented in OMARU
- The permutation workflow targets the phylogenetic (clade-level) association tests; separate Snakefiles govern gene/pathway association testing
- OMARU analysis is highly dependent on reference databases (e.g., UHGG for phylogenetic profiling), which affects the clades being tested and hence the threshold calibration

## Evidence

- [methods] Permutation-based empirical threshold derivation (source card workflow): [evidence span withheld — anchor status: not_found]

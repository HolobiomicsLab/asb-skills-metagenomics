---
name: harrell-davis-bootstrap-confidence-intervals
description: Use when performing multiple testing correction in metagenome-wide case-control
  association tests where an empirical null distribution of minimum P-values is built
  by permuting binary phenotype labels and the uncertainty of the resulting Psig threshold
  must be quantified.
when_to_use_negative:
- When a standard analytical multiple testing correction (e.g. standard Bonferroni
  on the number of tests) is sufficient and no permutation framework is being used
  — OMARU's empirical threshold is specifically built for clade-level MWAS P-values
- When input is raw sequencing data or pre-association profiling (Steps 1–3 of OMARU);
  the permutation threshold is applied only after association test summary statistics
  exist
- When analyzing continuous (non-case-control) phenotypes, since the permutation scheme
  permutes binary labels 1/0
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: OMARU
  role: End-to-end MWAS workflow that implements the phenotype permutation, minimum-P-value
    null distribution, and Harrell-Davis bootstrap CI on the empirical Bonferroni
    threshold
  repo: https://github.com/toshi-kishikawa/OMARU
- name: Snakemake
  role: Workflow engine executing the permutation and visualization Snakefile (OMARU_Phyl_permutation_visualization.sm)
  repo: https://github.com/toshi-kishikawa/OMARU
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqac019
    title: 'OMARU: a robust and multifaceted pipeline for metagenome-wide association
      study'
  source_content_hash: sha256:f6718b89c360829adbeda8bd0a0f6ee5f47d0d8e50137d325dd199e1b1fbdf83
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
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# harrell-davis-bootstrap-confidence-intervals

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

In OMARU, the Harrell-Davis estimator is used to construct a 95% bootstrap confidence interval around the empirical Bonferroni significance threshold (Psig), which itself is derived as the 95th percentile of the minimum P-value (Pmin) distribution from 10,000 phenotype permutations in the phylogenetic case-control association tests.

## When to use

Use when performing multiple testing correction in metagenome-wide case-control association tests where an empirical null distribution of minimum P-values is built by permuting binary phenotype labels and the uncertainty of the resulting Psig threshold must be quantified.

## When NOT to use

- When a standard analytical multiple testing correction (e.g. standard Bonferroni on the number of tests) is sufficient and no permutation framework is being used — OMARU's empirical threshold is specifically built for clade-level MWAS P-values
- When input is raw sequencing data or pre-association profiling (Steps 1–3 of OMARU); the permutation threshold is applied only after association test summary statistics exist
- When analyzing continuous (non-case-control) phenotypes, since the permutation scheme permutes binary labels 1/0

## Inputs

- Phylogenetic association summary statistics (sumstats_*.txt) from <OMARU_project_dir>/result/PHYL_QCed3/<Phylogenetic_reference>_association_test/<covariates>
- Phenotype sample list (original_sample_list.txt) with columns Sample, Phenotype (1 = case, 0 = control), Sex, Age
- Covariate list (covariates.txt; default sex and age) and config.yaml parameters N_PC_PHYL (default 2)

## Outputs

- Empirical Bonferroni threshold Psig (95th percentile of the permuted minimum-P-value distribution) with a Harrell-Davis bootstrap 95% CI
- Integrated summary statistics with permutation results and annotation (sumstats_*_annot.txt)
- P-value distribution and QQ figures in the <Phylogenetic_reference>_association_graph directory

## How to apply

First load the phylogenetic association summary statistics (sumstats_*.txt from Step 4-1, OMARU_Phyl_AS.sm) and the phenotype sample list (original_sample_list.txt, with phenotype encoded 1 = case, 0 = control) from <OMARU_project_dir>/data. Permute the binary phenotype labels 10,000 times, rerunning the clade-level association test on each permuted labeling and recording the minimum P-value per iteration to build the empirical null Pmin distribution. Define the empirical Bonferroni threshold Psig as the 95th percentile of this permuted minimum-P-value distribution, then estimate its uncertainty with a Harrell-Davis bootstrap 95% confidence interval — the Harrell-Davis estimator is a quantile estimator whose bootstrap resampling yields a CI on the percentile itself. Run the workflow with and without principal-component covariates (default N_PC_PHYL = 2; sex and age as base covariates via covariates.txt) to contrast P-value inflation. Execute via Snakemake to write integrated summary statistics (sumstats_*_annot.txt) and P-value distribution/QQ figures.

## Related tools

- **OMARU** (End-to-end MWAS workflow that implements the phenotype permutation, minimum-P-value null distribution, and Harrell-Davis bootstrap CI on the empirical Bonferroni threshold) — https://github.com/toshi-kishikawa/OMARU
- **Snakemake** (Workflow engine executing the permutation and visualization Snakefile (OMARU_Phyl_permutation_visualization.sm)) — https://github.com/toshi-kishikawa/OMARU

## Examples

```
cd OMARU_project_dir && snakemake -s Snakefiles/OMARU_Phyl_permutation_visualization.sm
```

## Evaluation signals

- The empirical Bonferroni threshold (95th percentile of permutation minimum P-values) should fall below the standard Bonferroni threshold, as reported for the SCZ analysis (90 SCZ patients and 81 healthy controls)
- The Harrell-Davis bootstrap 95% CI should bracket the point estimate of Psig from the 10,000-permutation Pmin distribution
- QQ/distribution figures should show P-value inflation in the no-PC-covariate run versus the PC-adjusted run (N_PC_PHYL = 2), confirming covariate correction
- Output file sumstats_*_annot.txt must appear in <OMARU_project_dir>/result/PHYL_QCed3/<Phylogenetic_reference>_association_test/<covariates> after the Snakefile completes
- Each permutation iteration must record exactly one minimum P-value, yielding 10,000 Pmin values for the null distribution

## Limitations

- Phenotype permutation for the empirical threshold is implemented for the phylogenetic association tests (Step 4-2) as documented; the article notes covariate integration into the case-control model for hidden confounders is not fully implemented in OMARU and relies on PC adjustment choices (N_PC_PHYL)
- 10,000 permutations make the workflow computationally intensive; running each permutation requires rerunning the clade-level association test
- Results depend on the chosen phylogenetic reference (e.g. CHOCO/UHGG) and the QCed3 sample list; changing references requires regenerating the null distribution

## Evidence

- [results] OMARU derives Psig by computing empirical null distributions of the minimum P-value via 10,000 phenotype permutations to control type I error, and reports a 95% CI for Pmin using the Harrell-Davis estimator.: [evidence span withheld — anchor status: not_found]

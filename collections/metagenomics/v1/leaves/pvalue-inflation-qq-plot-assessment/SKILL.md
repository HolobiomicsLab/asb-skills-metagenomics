---
name: pvalue-inflation-qq-plot-assessment
description: Use when case-control metagenomic association summary statistics (sumstats_*.txt)
  must be checked for P-value inflation and the correct number of PC covariates (e.g.
  N_PC_PHYL = 2) must be chosen before final reporting.
when_to_use_negative:
- Input is raw FASTQ or pre-QC abundance data rather than association test summary
  statistics from Step 4-1 — QC and profiling steps (OMARU_read_QC.sm, OMARU_QCed1/2.sm)
  must run first.
- The goal is functional/gene-level (UniRef90/KEGG) association assessment rather
  than the phylogenetic (UHGG-based clade) association test — use the corresponding
  gene/permutation workflow instead.
- Hidden confounders need direct adjustment in the case-control model — the article
  notes covariate integration for hidden confounding factors is not currently implemented
  in OMARU; PCs only partially address this.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: OMARU
  role: End-to-end MWAS workflow that runs the phylogenetic association tests, phenotype
    permutation, and QQ/distribution figure generation
  repo: https://github.com/toshi-kishikawa/OMARU
- name: Snakemake
  role: Workflow engine executing OMARU_Phyl_permutation_visualization.sm for permutation
    and visualization steps
  repo: https://snakemake.readthedocs.io/en/v5.1.4/index.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqac019
    title: 'OMARU: a robust and multifaceted pipeline for metagenome-wide association
      study'
  source_content_hash: sha256:e22f49deabac1835a968f0d63f5eb7e5840e37580112902a5b7b639e7834e9fd
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# pvalue-inflation-qq-plot-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assess P-value inflation in OMARU phylogenetic association tests by comparing QQ and P-value distribution figures between covariate-adjusted (PC) and no-PC runs, and validate significance via the empirical Bonferroni threshold derived from 10,000 phenotype permutations.

## When to use

Use when case-control metagenomic association summary statistics (sumstats_*.txt) must be checked for P-value inflation and the correct number of PC covariates (e.g. N_PC_PHYL = 2) must be chosen before final reporting.

## When NOT to use

- Input is raw FASTQ or pre-QC abundance data rather than association test summary statistics from Step 4-1 — QC and profiling steps (OMARU_read_QC.sm, OMARU_QCed1/2.sm) must run first.
- The goal is functional/gene-level (UniRef90/KEGG) association assessment rather than the phylogenetic (UHGG-based clade) association test — use the corresponding gene/permutation workflow instead.
- Hidden confounders need direct adjustment in the case-control model — the article notes covariate integration for hidden confounding factors is not currently implemented in OMARU; PCs only partially address this.

## Inputs

- Phylogenetic association summary statistics (`sumstats_*.txt`) from Step 4-1 at `<OMARU_project_dir>/result/PHYL_QCed3/<Phylogenetic_reference>_association_test/<covariates>`
- Phenotype sample list (`original_sample_list.txt` / QCed3_sample_list.txt) with binary phenotype (1 = case, 0 = control), sex, and age columns
- Covariate list (`covariates.txt`) and `config.yaml` parameters (N_PCs, N_PC_PHYL, SUFFIX_COV)

## Outputs

- Integrated summary statistics with permutation results and annotation (`sumstats_*_annot.txt`)
- P-value distribution and QQ figures in the `<Phylogenetic_reference>_association_graph` directory
- Empirical Bonferroni threshold Psig (95th percentile of permuted minimum P-values) with Harrell-Davis bootstrap 95% CI
- Phylogenetic tree of association results at `result/PHYL_QCed3/<Phylogenetic_reference>_ggtree/<covariates>`

## How to apply

After running OMARU_Phyl_AS.sm, repeat the clade-level association test with and without principal-component covariates (sex and age as base covariates via covariates.txt; default N_PC_PHYL = 2) to contrast P-value inflation between settings. Permute the binary phenotype labels (1 = case, 0 = control) 10,000 times, rerunning the test on each permuted labeling and recording the minimum P-value per iteration to build the empirical null distribution of Pmin. Define the empirical Bonferroni threshold Psig as the 95th percentile of the permuted minimum-P-value distribution and estimate its uncertainty with a Harrell-Davis bootstrap 95% CI. Execute `snakemake -s Snakefiles/OMARU_Phyl_permutation_visualization.sm` to write integrated summary statistics (sumstats_*_annot.txt) and the P-value distribution/QQ figures; inspect the figures for inflation in the no-PC run versus the PC-adjusted run.

## Related tools

- **OMARU** (End-to-end MWAS workflow that runs the phylogenetic association tests, phenotype permutation, and QQ/distribution figure generation) — https://github.com/toshi-kishikawa/OMARU
- **Snakemake** (Workflow engine executing OMARU_Phyl_permutation_visualization.sm for permutation and visualization steps) — https://snakemake.readthedocs.io/en/v5.1.4/index.html

## Examples

```
cd OMARU_project_dir && snakemake -s Snakefiles/OMARU_Phyl_permutation_visualization.sm
```

## Evaluation signals

- The empirical Bonferroni threshold (95th percentile of permutation minimum P-values) should fall below the standard Bonferroni threshold, as reported for the SCZ analysis.
- The QQ/distribution figure should show P-value inflation in the no-PC-covariate run versus the PC-adjusted run, confirming covariates suppress inflation.
- `sumstats_*_annot.txt` files appear at `<OMARU_project_dir>/result/PHYL_QCed3/<Phylogenetic_reference>_association_test/<covariates>` after the permutation Snakefile completes.
- Permutation null contains 10,000 minimum P-values (one per permuted phenotype labeling), each <= the smallest observed-case test P-value scale.
- The 95% CI for Pmin produced by the Harrell-Davis bootstrap estimator brackets the point estimate of Psig.

## Limitations

- Integration of hidden confounding factors as covariates directly in the case-control model is not currently implemented in OMARU and is a stated limitation; PC adjustment is the available mitigation.
- Results depend on the phylogenetic reference database (e.g. UHGG); metagenome analysis remains highly dependent on reference databases.
- The number of PCs adopted as covariates (N_PC_PHYL) must be selected manually from Step 4-1 output figures; mis-specification can leave residual inflation.

## Evidence

---
name: empirical-bonferroni-threshold-calculation
description: Use when performing OMARU phylogenetic or gene-level MWAS association
  testing and a multiple-testing correction threshold is needed that controls type
  I error better than the standard Bonferroni cutoff.
when_to_use_negative:
- When only a small number of tests are performed and a standard Bonferroni threshold
  is sufficient — the 10,000-permutation workflow is resource-intensive.
- When the phenotype is continuous rather than binary (1 = case, 0 = control); the
  permutation scheme as described permutes binary case-control labels.
- When association summary statistics from Step 4-1 (`sumstats_*.txt`) do not yet
  exist — the permutation step builds directly on those outputs.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3697
tools:
- name: OMARU
  role: End-to-end MWAS workflow whose Step 4-2 permutation and visualization Snakefile
    computes the empirical Bonferroni threshold; installed as a Conda/conda-pack package
  repo: https://github.com/toshi-kishikawa/OMARU
- name: Snakemake
  role: Workflow engine executing OMARU_Phyl_permutation_visualization.sm for the
    permutation, threshold estimation, and figure generation
  repo: https://snakemake.readthedocs.io/en/v5.1.4/index.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqac019
    title: 'OMARU: a robust and multifaceted pipeline for metagenome-wide association
      study'
  source_content_hash: sha256:0de42e5faecde59fd6c0f5b1326c5719b0aadc9cc9c4fc341820719a84dba5ca
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
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# empirical-bonferroni-threshold-calculation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Derives an empirical Bonferroni significance threshold (Psig) for OMARU case-control association tests by permuting binary phenotype labels 10,000 times, building an empirical null distribution of minimum P-values, and taking the 95th percentile with a Harrell-Davis bootstrap 95% CI.

## When to use

Use when performing OMARU phylogenetic or gene-level MWAS association testing and a multiple-testing correction threshold is needed that controls type I error better than the standard Bonferroni cutoff.

## When NOT to use

- When only a small number of tests are performed and a standard Bonferroni threshold is sufficient — the 10,000-permutation workflow is resource-intensive.
- When the phenotype is continuous rather than binary (1 = case, 0 = control); the permutation scheme as described permutes binary case-control labels.
- When association summary statistics from Step 4-1 (`sumstats_*.txt`) do not yet exist — the permutation step builds directly on those outputs.

## Inputs

- Phylogenetic association summary statistics (`sumstats_*.txt`) from Step 4-1 at `<OMARU_project_dir>/result/PHYL_QCed3/<Phylogenetic_reference>_association_test/<covariates>`
- Phenotype sample list (`original_sample_list.txt`) with binary phenotype column (1 = case, 0 = control) at `<OMARU_project_dir>/data`
- Covariate list (`covariates.txt`; default sex and age) at `<OMARU_project_dir>/data`
- config.yaml parameters (N_PC_PHYL, SUFFIX_COV, N_PCs)

## Outputs

- Integrated summary statistics with permutation results and annotation (`sumstats_*_annot.txt`) at `<OMARU_project_dir>/result/PHYL_QCed3/<Phylogenetic_reference>_association_test/<covariates>`
- Empirical Bonferroni threshold (Psig) as the 95th percentile of the permutation minimum-P-value distribution with Harrell-Davis bootstrap 95% CI
- P-value distribution and QQ figures at `<OMARU_project_dir>/result/PHYL_QCed3/<Phylogenetic_reference>_association_graph`
- Phylogenetic tree of association results at `result/PHYL_QCed3/<Phylogenetic_reference>_ggtree/<covariates>`

## How to apply

After running the clade-level association test (Step 4-1, OMARU_Phyl_AS.sm) which produces `sumstats_*.txt`, permute the binary phenotype labels (1 = case, 0 = control) in `original_sample_list.txt` 10,000 times, rerunning the association test each time and recording the minimum P-value per permutation to build the empirical null distribution. Define Psig as the 95th percentile of this permutated minimum-P-value distribution, and estimate its uncertainty with a Harrell-Davis bootstrap 95% confidence interval. Run the workflow with and without principal-component covariates (default N_PC_PHYL = 2; base covariates sex and age via `covariates.txt`) to contrast P-value inflation. Execute `snakemake -s Snakefiles/OMARU_Phyl_permutation_visualization.sm` (Step 4-2) to integrate permutation results into `sumstats_*_annot.txt` and generate P-value distribution/QQ figures.

## Related tools

- **OMARU** (End-to-end MWAS workflow whose Step 4-2 permutation and visualization Snakefile computes the empirical Bonferroni threshold; installed as a Conda/conda-pack package) — https://github.com/toshi-kishikawa/OMARU
- **Snakemake** (Workflow engine executing OMARU_Phyl_permutation_visualization.sm for the permutation, threshold estimation, and figure generation) — https://snakemake.readthedocs.io/en/v5.1.4/index.html

## Examples

```
cd OMARU_project_dir && snakemake -s Snakefiles/OMARU_Phyl_permutation_visualization.sm
```

## Evaluation signals

- The empirical Bonferroni threshold (95th percentile of permutation minimum P-values) falls below the standard Bonferroni threshold, as reported for the SCZ analysis.
- The QQ/distribution figure shows P-value inflation in the no-PC-covariate run versus the PC-adjusted run (default N_PC_PHYL = 2).
- `sumstats_*_annot.txt` files appear at `<OMARU_project_dir>/result/PHYL_QCed3/<Phylogenetic_reference>_association_test/<covariates>` integrating permutation results and annotation.
- 10,000 permutations completed with a minimum P-value recorded per iteration, and the Harrell-Davis bootstrap 95% CI for Pmin is reported.

## Limitations

- Hidden confounding factors would ideally be adjusted by integration of the covariates into a case-control model, but this is not currently implemented in OMARU.
- Metagenome analysis is still highly dependent on reference databases; database development is a challenge for the future.
- No specific versions of tools or reference databases are stated; the workflow is executed via conda-pack (Zenodo record 5201539) with Conda 4.8 or greater.

## Evidence

- [results] In multiple test correction, empirically estimated Bonferroni threshold was lower than the standard Bonferroni threshold: "In multiple test correction, empirically estimated Bonferroni threshold was lower than the standard Bonferroni threshold"
- [abstract] OMARU rigorously controls the statistical significance: "OMARU rigorously controls the statistical significance of the analysis results, including correction of hidden confounding factors and application of multiple testing comparisons"
- [other] 10,000 phenotype permutations: [evidence span withheld — anchor status: not_found]

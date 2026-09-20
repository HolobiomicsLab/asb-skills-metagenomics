---
name: principal-component-confounder-adjustment
description: Use when running an OMARU phylogenetic or gene case-control association
  test and you must decide how many PCs to add as covariates on top of metadata (e.g.
  sex, age, BMI) to control hidden confounders.
when_to_use_negative:
- The sample list has not passed read QC, profiling QC, and abundance QC — association
  tests must use QCed3_sample_list.txt, not original_sample_list.txt
- You expect hidden confounders to be adjusted inside the case-control model itself
  — the article states this integration 'is not currently implemented in OMARU' and
  must instead be handled via explicit covariates
- Input is a single pre-specified model with no need to compare PC counts — then just
  run the fixed-covariate association test without the N_PCs sweep
tools:
- name: OMARU
  role: End-to-end workflow whose phylogenetic/gene association tests (OMARU_Phyl_AS.sm)
    accept covariates and a configurable number of PCs as hidden-confounder adjustment
  repo: https://github.com/toshi-kishikawa/OMARU
- name: Snakemake
  role: Workflow engine used to execute the OMARU association-test Snakefile and vary
    the N_PCs parameter across runs
  repo: https://snakemake.readthedocs.io/en/v5.1.4/index.html
- name: Conda
  role: Required (version 4.8 or greater) to install and operate the OMARU conda-pack
    environment that contains the pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqac019
    title: 'OMARU: a robust and multifaceted pipeline for metagenome-wide association
      study'
  source_content_hash: sha256:de94f5d77a05c9a733dc680c22943305264dc52587ec2e829e1650deacb3e97d
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
  - Conda
  - OMARU
  - Snakemake
  - conda-pack
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# principal-component-confounder-adjustment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Selects the number of principal components (plus fixed covariates such as sex, age, and BMI) to include as covariates in OMARU case-control association tests, to suppress inflation of P-values from hidden confounding factors before multiple-testing correction.

## When to use

Use when running an OMARU phylogenetic or gene case-control association test and you must decide how many PCs to add as covariates on top of metadata (e.g. sex, age, BMI) to control hidden confounders.

## When NOT to use

- The sample list has not passed read QC, profiling QC, and abundance QC — association tests must use QCed3_sample_list.txt, not original_sample_list.txt
- You expect hidden confounders to be adjusted inside the case-control model itself — the article states this integration 'is not currently implemented in OMARU' and must instead be handled via explicit covariates
- Input is a single pre-specified model with no need to compare PC counts — then just run the fixed-covariate association test without the N_PCs sweep

## Inputs

- QCed3_sample_list.txt (samples passing all QC stages)
- covariates<SUFFIX_COV>.txt covariate list (e.g. sex, age, BMI, top 30 PCs)
- OMARU project config.yaml (SUFFIX_COV, N_PCs, N_PC_PHYL, N_PC_GENE parameters)
- Clade/gene abundance profiles from OMARU_QCed1/QCed2 profiling stages

## Outputs

- sumstats_*.txt summary statistics per tested PC count at result/PHYL_QCed3/<Phylogenetic_reference>_association_test/<covariates>
- Association figures at result/PHYL_QCed3/<Phylogenetic_reference>_association_graph
- Updated config.yaml with the adopted N_PC_PHYL/N_PC_GENE value
- Post-permutation sumstats_*_annot.txt with FDR/annotation for threshold PQ = 0.05

## How to apply

Write the covariate list (default covariates.txt is only sex and age; the SCZ analysis used sex, age, BMI and the top 30 principal components) as covariates<SUFFIX_COV>.txt in the project data directory and set SUFFIX_COV in config.yaml. Set N_PCs in config.yaml to the range of PC counts to trial (the SCZ phylogenetic test tried N_PCs = [0,1,2,3]) and run the association test via `snakemake -s Snakefiles/OMARU_Phyl_AS.sm`, which produces sumstats_*.txt under result/PHYL_QCed3/<Phylogenetic_reference>_association_test/<covariates>. Inspect each run's summary statistics for P-value inflation, then fix the adopted PC count by setting N_PC_PHYL in config.yaml (default 2; N_PC_GENE = 2 for the gene test) before the permutation/visualization step. Rationale: OMARU does not integrate hidden confounders directly into the case-control model, so PCs are used as explicit covariates, and multiple-test correction (PQ = 0.05) is applied afterwards; in the SCZ example the empirically estimated Bonferroni threshold was lower than the standard Bonferroni threshold.

## Related tools

- **OMARU** (End-to-end workflow whose phylogenetic/gene association tests (OMARU_Phyl_AS.sm) accept covariates and a configurable number of PCs as hidden-confounder adjustment) — https://github.com/toshi-kishikawa/OMARU
- **Snakemake** (Workflow engine used to execute the OMARU association-test Snakefile and vary the N_PCs parameter across runs) — https://snakemake.readthedocs.io/en/v5.1.4/index.html
- **Conda** (Required (version 4.8 or greater) to install and operate the OMARU conda-pack environment that contains the pipeline)

## Examples

```
cd OMARU_project_dir && snakemake -s Snakefiles/OMARU_Phyl_AS.sm
```

## Evaluation signals

- sumstats_*.txt exists under result/PHYL_QCed3/<Phylogenetic_reference>_association_test/<covariates> for each PC count trialed in N_PCs
- config.yaml records a single adopted value for N_PC_PHYL (default 2) and N_PC_GENE (default 2) after comparing the sweep results
- Adjusted runs show suppression of P-value inflation relative to unadjusted covariate sets, as reported when 'adjusting covariates in the same way as the phylogenetic analyses'
- Multiple-test correction uses the PQ = 0.05 threshold and, in the SCZ example, the empirically estimated Bonferroni threshold was lower than the standard Bonferroni threshold
- Downstream permutation/visualization produces sumstats_*_annot.txt integrating permutation results and annotation

## Limitations

- Integration of hidden confounding factors directly into the case-control model is not implemented in OMARU; covariates/PCs are the only adjustment mechanism
- The optimal PC count must be chosen separately per analysis type (N_PC_PHYL vs N_PC_GENE) and per covariate set (SUFFIX_COV), and defaults may not suit other datasets
- Empirical Bonferroni thresholds can be lower than standard Bonferroni, so significance calls depend on the permutation calibration step

## Evidence

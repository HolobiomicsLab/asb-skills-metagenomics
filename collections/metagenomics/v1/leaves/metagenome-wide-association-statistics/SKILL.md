---
name: metagenome-wide-association-statistics
description: Use when running OMARU phylogenetic case-control MWAS and needing a multiple-testing
  threshold that controls type I error under the study's own clade count and sample
  structure, rather than a standard Bonferroni cutoff.
when_to_use_negative:
- Gene or pathway association tests (Step 5) that need their own per-feature thresholds
  rather than the phylogenetic clade permutation scheme
- Analyses where hidden confounding cannot be handled by covariates at all — the article
  notes covariate integration into the case-control model is not implemented in OMARU
  beyond supplied covariate columns
- Datasets lacking a binary case/control phenotype column (1/0) in `original_sample_list.txt`,
  since permutation of phenotype labels is the core of the procedure
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: OMARU
  role: End-to-end MWAS pipeline whose phylogenetic association test and permutation/visualization
    step (Step 4-2) produce the empirical Bonferroni threshold and QQ/distribution
    figures
  repo: https://github.com/toshi-kishikawa/OMARU
- name: Snakemake
  role: Workflow engine executing the permutation and visualization Snakefile (`OMARU_Phyl_permutation_visualization.sm`)
  repo: https://snakemake.readthedocs.io/en/v5.1.4/index.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqac019
    title: 'OMARU: a robust and multifaceted pipeline for metagenome-wide association
      study'
  source_content_hash: sha256:0fd8c2b83a616575f3498584d7fbe7b8e26f1f9defdddbfb87f7c17620d77d6f
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
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# metagenome-wide-association-statistics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstruct OMARU's empirical Bonferroni significance threshold (Psig) by permuting binary phenotype labels 10,000 times in clade-level case-control association tests, taking the 95th percentile of the per-iteration minimum P-value distribution, and quantifying its uncertainty with a Harrell-Davis bootstrap 95% CI for Pmin.

## When to use

Use when running OMARU phylogenetic case-control MWAS and needing a multiple-testing threshold that controls type I error under the study's own clade count and sample structure, rather than a standard Bonferroni cutoff.

## When NOT to use

- Gene or pathway association tests (Step 5) that need their own per-feature thresholds rather than the phylogenetic clade permutation scheme
- Analyses where hidden confounding cannot be handled by covariates at all — the article notes covariate integration into the case-control model is not implemented in OMARU beyond supplied covariate columns
- Datasets lacking a binary case/control phenotype column (1/0) in `original_sample_list.txt`, since permutation of phenotype labels is the core of the procedure

## Inputs

- Phylogenetic association summary statistics (`sumstats_*.txt`) from Step 4-1 at `<OMARU_project_dir>/result/PHYL_QCed3/<Phylogenetic_reference>_association_test/<covariates>`
- Phenotype sample list (`original_sample_list.txt`) with Sample, Phenotype, Sex, Age columns (phenotype coded 1 = case, 0 = control)
- Covariate list (`covariates.txt`; default sex and age; extendable via `covariates<SUFFIX_COV>.txt`)
- `config.yaml` parameters (N_PC_PHYL = 2, N_PCs list such as [0,1,2,3])

## Outputs

- Integrated summary statistics with permutation results and annotation (`sumstats_*_annot.txt`) at `<OMARU_project_dir>/result/PHYL_QCed3/<Phylogenetic_reference>_association_test/<covariates>`
- Empirical Bonferroni threshold Psig (95th percentile of permutation minimum P-values) with Harrell-Davis bootstrap 95% CI for Pmin
- P-value distribution and QQ figures in the `<Phylogenetic_reference>_association_graph` directory
- Phylogenetic tree of association results at `result/PHYL_QCed3/<Phylogenetic_reference>_ggtree/<covariates>`

## How to apply

After Step 4-1 (`OMARU_Phyl_AS.sm`) produces clade-level summary statistics (`sumstats_*.txt`) using the QCed3 sample list and covariates (default sex and age via `covariates.txt`, with N_PC_PHYL = 2 principal components), permute the binary phenotype labels (1 = case, 0 = control in `original_sample_list.txt`) 10,000 times, rerunning the clade association test on each permuted labeling and recording the minimum P-value per iteration. Define Psig as the 95th percentile of this empirical null distribution of minimum P-values and attach a Harrell-Davis bootstrap 95% CI to Pmin. Rerun the association testing with and without PC covariates to contrast P-value inflation. Execute the whole workflow with `snakemake -s Snakefiles/OMARU_Phyl_permutation_visualization.sm`, which writes integrated statistics (`sumstats_*_annot.txt`) and P-value distribution/QQ figures. The rationale is that permutation-derived thresholds reflect the actual number of tests and correlation structure among clades, and in the SCZ analysis the empirical Bonferroni threshold fell below the standard Bonferroni threshold.

## Related tools

- **OMARU** (End-to-end MWAS pipeline whose phylogenetic association test and permutation/visualization step (Step 4-2) produce the empirical Bonferroni threshold and QQ/distribution figures) — https://github.com/toshi-kishikawa/OMARU
- **Snakemake** (Workflow engine executing the permutation and visualization Snakefile (`OMARU_Phyl_permutation_visualization.sm`)) — https://snakemake.readthedocs.io/en/v5.1.4/index.html

## Examples

```
cd OMARU_project_dir && snakemake -s Snakefiles/OMARU_Phyl_permutation_visualization.sm
```

## Evaluation signals

- The empirical Bonferroni threshold (95th percentile of permutation minimum P-values) should fall below the standard Bonferroni threshold, as reported for the SCZ analysis
- The QQ/distribution figure should show P-value inflation in the no-PC-covariate run versus the PC-adjusted run (N_PC_PHYL = 2)
- `sumstats_*_annot.txt` files are written to `<OMARU_project_dir>/result/PHYL_QCed3/<Phylogenetic_reference>_association_test/<covariates>` after Step 4-2
- Permutation null is built from exactly 10,000 phenotype permutations with minimum P-value recorded per iteration
- Harrell-Davis bootstrap 95% CI is reported alongside Pmin

## Limitations

- Covariate integration into the case-control model (adjusting hidden confounding factors) is not implemented in OMARU; adjustment relies on user-supplied covariate columns and PCs
- The workflow requires 10,000 reruns of the association test, so compute cost scales with clade count and sample size
- Metagenome analysis remains highly dependent on reference databases; results depend on the chosen phylogenetic reference (e.g. UHGG/CHOCO)

## Evidence

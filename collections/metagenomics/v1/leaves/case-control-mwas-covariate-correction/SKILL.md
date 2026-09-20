---
name: case-control-mwas-covariate-correction
description: Use when QCed FASTQ-derived clade/gene abundance tables (QCed3_sample_list.txt)
  from shotgun metagenomic data must be tested for case-control association with hidden
  confounding correction.
when_to_use_negative:
- Input is raw FASTQ that has not passed read QC, profiling QC and abundance QC (QCed1/QCed2/QCed3
  stages) — association tests require the QCed3-filtered abundance profiles.
- The question is diversity or functional profiling rather than case-control clade-level
  association testing.
- You need confounder adjustment by directly integrating covariates into the case-control
  model — this is not currently implemented in OMARU; correction relies on PC adjustment
  and multiple-testing control.
edam_topics:
- http://edamontology.org/topic_0659
tools:
- name: OMARU
  role: End-to-end pipeline providing the phylogenetic case-control association test
    with hidden confounding correction and multiple testing comparison
  repo: https://github.com/toshi-kishikawa/OMARU
- name: Snakemake
  role: Workflow engine executing the association-test Snakefile OMARU_Phyl_AS.sm
- name: Conda
  role: Installation/operation of OMARU via conda-pack environment (version 4.8 or
    greater)
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqac019
    title: 'OMARU: a robust and multifaceted pipeline for metagenome-wide association
      study'
  source_content_hash: sha256:5638f072d7d121827989d47950899ce8ddb964c54ffad813472497e863d09981
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
  edam_topics:
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# case-control-mwas-covariate-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run OMARU's phylogenetic (and gene) case-control MWAS association tests on QCed clade abundance profiles while correcting for hidden confounding via covariates (sex, age, BMI) and principal components, applying PQ=0.05 p-value/FDR thresholds for significance.

## When to use

Use when QCed FASTQ-derived clade/gene abundance tables (QCed3_sample_list.txt) from shotgun metagenomic data must be tested for case-control association with hidden confounding correction.

## When NOT to use

- Input is raw FASTQ that has not passed read QC, profiling QC and abundance QC (QCed1/QCed2/QCed3 stages) — association tests require the QCed3-filtered abundance profiles.
- The question is diversity or functional profiling rather than case-control clade-level association testing.
- You need confounder adjustment by directly integrating covariates into the case-control model — this is not currently implemented in OMARU; correction relies on PC adjustment and multiple-testing control.

## Inputs

- QCed FASTQ-derived clade abundance profiles filtered with QCed3_sample_list.txt
- original_sample_list.txt with Phenotype (1=case, 0=control), Sex, Age and other metadata columns
- covariates.txt (or covariates<SUFFIX_COV>.txt) listing covariate column names
- config.yaml with PHYL_THRESHOLD, SUFFIX_COV, N_PCs, N_PC_PHYL, PQ parameters
- UHGG-based phylogenetic reference prepared via Prepare_reference_UHGG.sh

## Outputs

- sumstats_*.txt per-clade association summary statistics (one row per clade with p-value and FDR fields) under result/PHYL_QCed3/<Phylogenetic_reference>_association_test/<covariates>
- sumstats_*_annot.txt integrating permutation results and annotation
- association figures under result/PHYL_QCed3/<Phylogenetic_reference>_association_graph
- phylogenetic tree (ggtree) of association results

## How to apply

After read QC and profiling Snakefiles produce clade abundance profiles and QCed3_sample_list.txt, write the covariate list in OMARU_project_dir/data/covariates.txt (default sex and age; SCZ analysis adopted sex, age, BMI and top 30 PCs) and set SUFFIX_COV and N_PCs=[0,1,2,3] in config.yaml. Run the phylogenetic association Snakefile OMARU_Phyl_AS.sm so per-clade case-control tests are fitted for each candidate number of PCs, then inspect sumstats_*.txt to select the adopted number and fix N_PC_PHYL (e.g. =2) in config.yaml; apply PHYL_THRESHOLD=5 (relative abundance cutoff 1e-5) and the PQ=0.05 threshold on p-value and FDR. Optionally run OMARU_Phyl_permutation_visualization.sm to integrate permutation results and annotations into sumstats_*_annot.txt. The rationale is that OMARU rigorously controls statistical significance through correction of hidden confounding factors and multiple testing comparisons, which is why disease-associated clade counts can be much lower than in original reference MWAS papers (e.g. 789 T2D clades with none significant, 692 SCZ clades with 3 significant including Streptococcus vestibularis).

## Related tools

- **OMARU** (End-to-end pipeline providing the phylogenetic case-control association test with hidden confounding correction and multiple testing comparison) — https://github.com/toshi-kishikawa/OMARU
- **Snakemake** (Workflow engine executing the association-test Snakefile OMARU_Phyl_AS.sm)
- **Conda** (Installation/operation of OMARU via conda-pack environment (version 4.8 or greater))

## Examples

```
cd OMARU_project_dir && snakemake -s Snakefiles/OMARU_Phyl_AS.sm
```

## Evaluation signals

- sumstats_*.txt exists under result/PHYL_QCed3/<Phylogenetic_reference>_association_test/<covariates> with one row per clade including p-value and FDR fields.
- Row count matches expected clade count at PHYL_THRESHOLD=5 (e.g. ~789 clades for the T2D dataset, ~692 for SCZ).
- Significance is decided at PQ=0.05 on both p-value and FDR; e.g. no T2D clade passes, while SCZ flags three clades including Streptococcus vestibularis.
- N_PC_PHYL in config.yaml is set to the number of PCs selected after inspecting the N_PCs=[0,1,2,3] trial outputs before final reporting.
- Phenotype permutation output sumstats_*_annot.txt integrates permutation and annotation columns without changing per-clade statistics.

## Limitations

- Integration of covariates directly into the case-control model is not currently implemented in OMARU; hidden confounders are instead handled via PC adjustment and multiple testing comparisons.
- Disease-associated clade counts may be considerably lower than in reference MWAS papers due to correction of hidden confounding factors (e.g. 789 T2D clades with none significant).
- Metagenome analysis is highly dependent on reference databases; database development is a future challenge.
- Preparing the UHGG phylogenetic reference can take several hours and almost 200GB of free disk space.

## Evidence

- [abstract] OMARU rigorously controls the statistical significance of the analysis results, including correction of hidden confounding factors and application of multiple testing comparisons: "OMARU rigorously controls the statistical significance of the analysis results, including correction of hidden confounding factors and application of multiple testing comparisons"
- [readme] README association test invocation: "$ snakemake -s Snakefiles/OMARU_Phyl_AS.sm"
- [readme] README PC covariate customization: "You can change the list of numbers of PCs used as covariates."
- [readme] README covariate list default: "The covariate list can be changed from the default of `<OMARU_project_dir>/data/covariates.txt`. (The default is only sex and age)."
- [results] SCZ covariates adopted in results: "We adopted sex, age, body mass index (BMI) and the top 30 principal components as covariates"

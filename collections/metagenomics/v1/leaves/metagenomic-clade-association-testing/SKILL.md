---
name: metagenomic-clade-association-testing
description: Use when QCed shotgun metagenomic FASTQ data from case and control cohorts
  are available and per-clade disease association statistics (p-values/FDR) are needed
  via phylogenetic profiling.
when_to_use_negative:
- Input is already a clade abundance/feature table rather than raw shotgun FASTQ reads
  — OMARU expects FASTQ in data/original_fastq and performs its own read QC and mapping
- The goal is gene-level or pathway-level (KEGG/GO, UniRef90) functional association
  testing rather than clade-level phylogenetic association
- Cohort samples have not passed the required QC chain — running OMARU_Phyl_AS.sm
  before producing QCed3_sample_list.txt is invalid
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: OMARU
  role: End-to-end MWAS workflow performing read QC, phylogenetic profiling, and clade
    case-control association testing with confounder adjustment and multiple-testing
    correction
  repo: https://github.com/toshi-kishikawa/OMARU
- name: Snakemake
  role: Workflow engine executing the OMARU_QCed and OMARU_Phyl_AS Snakefiles that
    carry samples through QC and association testing
- name: Conda
  role: Installation mechanism (conda-pack, version 4.8+) for the OMARU environment
    distributed via Zenodo
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqac019
    title: 'OMARU: a robust and multifaceted pipeline for metagenome-wide association
      study'
  source_content_hash: sha256:ca46559b24e90d262ed3d8b6499231e7d7a6b1d4c003ac4123265f0af034b5ed
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# metagenomic-clade-association-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Performs phylogenetic case-control MWAS clade association testing with OMARU, mapping shotgun reads to UHGG-based clade references and fitting case-control models that adjust for covariates and hidden confounders (PCs) with multiple-testing correction.

## When to use

Use when QCed shotgun metagenomic FASTQ data from case and control cohorts are available and per-clade disease association statistics (p-values/FDR) are needed via phylogenetic profiling.

## When NOT to use

- Input is already a clade abundance/feature table rather than raw shotgun FASTQ reads — OMARU expects FASTQ in data/original_fastq and performs its own read QC and mapping
- The goal is gene-level or pathway-level (KEGG/GO, UniRef90) functional association testing rather than clade-level phylogenetic association
- Cohort samples have not passed the required QC chain — running OMARU_Phyl_AS.sm before producing QCed3_sample_list.txt is invalid

## Inputs

- QCed shotgun metagenomic FASTQ files (<Sample_ID>_R1.fastq.gz / _R2.fastq.gz)
- original_sample_list.txt with sample ID, phenotype (1/0), sex, age, and other metadata
- covariates.txt (or covariates<SUFFIX_COV>.txt) listing covariate column names
- UHGG-based phylogenetic reference databases prepared by Prepare_reference_UHGG.sh
- config.yaml with PHYL_THRESHOLD, SUFFIX_COV, N_PCs, N_PC_PHYL, PQ settings

## Outputs

- sumstats_*.txt per-clade summary statistics with p-value and FDR fields under result/PHYL_QCed3/<Phylogenetic_reference>_association_test/<covariates>
- sumstats_*_annot.txt integrating permutation results and annotation
- Association figures in result/PHYL_QCed3/<Phylogenetic_reference>_association_graph
- Phylogenetic tree of association results in result/PHYL_QCed3/<Phylogenetic_reference>_ggtree/<covariates>

## How to apply

Install OMARU from the Zenodo conda-pack (record 5201539) and prepare the UHGG phylogenetic reference with Prepare_reference_UHGG.sh into OMARU_dir/OMARU_databases (~200 GB, several hours). Run the read-QC, profiling, and abundance-QC Snakefiles (OMARU_read_QC.sm, OMARU_QCed1.sm, OMARU_QCed2.sm, OMARU_QCed3.sm) to progressively filter samples into QCed1/QCed2/QCed3_sample_list.txt. Define the covariate list in data/covariates.txt (default sex and age; e.g. SCZ used sex, age, BMI, and top 30 principal components) and set config.yaml parameters: PHYL_THRESHOLD=5 (relative abundance cutoff 1e-5), SUFFIX_COV, and N_PCs=[0,1,2,3] for candidate PC covariates. Execute 'snakemake -s Snakefiles/OMARU_Phyl_AS.sm' to fit case-control tests for each clade across tried PC numbers, then fix the adopted number via N_PC_PHYL in config.yaml. Optionally run OMARU_Phyl_permutation_visualization.sm to integrate permutation results and annotations into sumstats_*_annot.txt, applying the PQ=0.05 p-value/FDR threshold. Rationale: OMARU rigorously controls statistical significance via correction of hidden confounding factors and multiple testing comparisons, which affects clade counts compared to original reference studies (e.g. 789 clades tested in T2D with none significant vs. 692 clades in SCZ with 3 significant including Streptococcus vestibularis).

## Related tools

- **OMARU** (End-to-end MWAS workflow performing read QC, phylogenetic profiling, and clade case-control association testing with confounder adjustment and multiple-testing correction) — https://github.com/toshi-kishikawa/OMARU
- **Snakemake** (Workflow engine executing the OMARU_QCed and OMARU_Phyl_AS Snakefiles that carry samples through QC and association testing)
- **Conda** (Installation mechanism (conda-pack, version 4.8+) for the OMARU environment distributed via Zenodo)

## Examples

```
snakemake -s Snakefiles/OMARU_Phyl_AS.sm
```

## Evaluation signals

- sumstats_*.txt exists under result/PHYL_QCed3/<Phylogenetic_reference>_association_test/<covariates> with one row per clade including p-value and FDR fields
- Row count of the summary table matches the number of tested clades (e.g. ~692 for SCZ, ~789 for T2D in the published runs)
- Significance is flagged only where FDR passes the PQ=0.05 threshold (e.g. 3 clades including Streptococcus vestibularis for SCZ; 0 clades for T2D)
- QCed3_sample_list.txt exists in OMARU_project_dir/data, confirming all three QC filtering stages completed before association testing
- Disease-associated clade counts are expected to be lower than reference papers due to correction of hidden confounding factors — a mismatch in that direction suggests covariate/PC configuration is working

## Limitations

- Integration of covariates into the case-control model (confounder adjustment) is not fully implemented in OMARU; hidden confounding is instead handled via PC correction, which may cause disease-associated clade counts to be considerably lower than in reference papers
- Preparing the UHGG phylogenetic reference is time and resource-intensive: several hours and almost 200 GB of free disk space
- Metagenome analysis is highly dependent on reference databases; users can supply custom references but must provide NCBI_species_scaffold, NCBI_lineage, and eachL_lineage files in OMARU_databases

## Evidence

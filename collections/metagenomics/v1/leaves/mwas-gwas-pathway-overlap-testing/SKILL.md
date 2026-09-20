---
name: mwas-gwas-pathway-overlap-testing
description: Use when both MWAS pathway enrichment results (e.g. sumstats_*_annot.txt
  from OMARU) and host GWAS pathway enrichment results (e.g. PASCAL output as <Phenotype>_GWAS_<Pathway_reference>.txt)
  are available and you need to assess their overlap significance.
when_to_use_negative:
- When only one side of the comparison is available (no PASCAL-processed GWAS pathway
  enrichment file, or no OMARU sumstats_*_annot.txt MWAS results) — the contingency
  table cannot be built.
- When the goal is testing overlap of individual taxa or genes rather than biological
  pathways — use OMARU's taxa-gene linking analysis instead of the MWAS-GWAS pathway
  interaction module.
- When a significance threshold other than the enrichment-level P<0.05 definition
  is required for pathway significance, since the contingency table construction here
  is defined at that threshold.
edam_topics:
- http://edamontology.org/topic_3837
- http://edamontology.org/topic_2269
tools:
- name: OMARU
  role: End-to-end workflow that produces the MWAS pathway enrichment results (sumstats_*_annot.txt)
    and integrates them with GWAS results via the Fisher's exact test overlap analysis
  repo: https://github.com/toshi-kishikawa/OMARU
- name: Snakemake
  role: Workflow engine used to run the integrated MWAS-GWAS pathway overlap step
    (Snakefiles/OMARU_Func_MWAS_GWAS.sm)
- name: PASCAL
  role: 'Used with GWAS summary statistics (SCZ: 22,778 cases/35,362 controls; T2D:
    77,418 cases/433,544 controls) to generate the GWAS pathway enrichment files compared
    against the MWAS results'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nargab/lqac019
    title: 'OMARU: a robust and multifaceted pipeline for metagenome-wide association
      study'
  source_content_hash: sha256:0dcb83869d5585d573f642cc3cf3d4e3a4ff564c118f7705e5677be78b2f4737
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
  - PASCAL
  - Snakemake
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3837
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# mwas-gwas-pathway-overlap-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tests whether biological pathways enriched in a disease metagenome (MWAS) significantly overlap with those enriched in the host germline GWAS, using OMARU's 2x2 contingency-table Fisher's exact test at P<0.05. It quantifies disease-specific pathway-level links between the human genome and metagenome (e.g. PFisher = 0.011 SCZ, 0.008 T2D).

## When to use

Use when both MWAS pathway enrichment results (e.g. sumstats_*_annot.txt from OMARU) and host GWAS pathway enrichment results (e.g. PASCAL output as <Phenotype>_GWAS_<Pathway_reference>.txt) are available and you need to assess their overlap significance.

## When NOT to use

- When only one side of the comparison is available (no PASCAL-processed GWAS pathway enrichment file, or no OMARU sumstats_*_annot.txt MWAS results) — the contingency table cannot be built.
- When the goal is testing overlap of individual taxa or genes rather than biological pathways — use OMARU's taxa-gene linking analysis instead of the MWAS-GWAS pathway interaction module.
- When a significance threshold other than the enrichment-level P<0.05 definition is required for pathway significance, since the contingency table construction here is defined at that threshold.

## Inputs

- MWAS pathway enrichment results: sumstats_*_annot.txt from Snakefiles/OMARU_Func_permutation.sm (SCZ n=344 and T2D n=171 metagenomic cohorts)
- GWAS pathway enrichment files: <Phenotype>_GWAS_<Pathway_reference>.txt (pathway ID and p-value columns) from PASCAL, placed in <OMARU_project_dir>/data
- SCZ GWAS summary statistics (22,778 cases / 35,362 controls) and T2D GWAS summary statistics (77,418 cases / 433,544 controls) processed via PASCAL

## Outputs

- 2x2 contingency tables of MWAS-significant vs GWAS-significant pathway counts per phenotype
- Fisher's exact test P-values (PFisher) per phenotype, e.g. 0.011 for SCZ and 0.008 for T2D
- Tables and figures of MWAS-GWAS links at <OMARU_project_dir>/result/FUNC_QCed3/MWAS_GWAS/<Pathway_reference>
- MWAS_GWAS_pathway_overlap_PFisher.tsv

## How to apply

Load the MWAS pathway enrichment results produced by the OMARU gene abundance association module (sumstats_*_annot.txt from Snakefiles/OMARU_Func_permutation.sm) and the GWAS pathway enrichment files (first column pathway ID, second column p-value) generated by running PASCAL on GWAS summary statistics, placing GWAS files in <OMARU_project_dir>/data. Define significantly enriched pathways in both the MWAS and GWAS results at the P<0.05 threshold for each phenotype (SCZ n=344, T2D n=171 metagenomic cohorts). Construct the 2x2 contingency table of pathway counts (MWAS-significant/GWAS-significant) for each phenotype separately and compute Fisher's exact test P-values with scipy.stats.fisher_exact. Alternatively, run the integrated workflow via 'snakemake -s Snakefiles/OMARU_Func_MWAS_GWAS.sm', which writes tables and figures of MWAS-GWAS links to <OMARU_project_dir>/result/FUNC_QCed3/MWAS_GWAS/<Pathway_reference>. Save contingency tables and PFisher scores to MWAS_GWAS_pathway_overlap_PFisher.tsv.

## Related tools

- **OMARU** (End-to-end workflow that produces the MWAS pathway enrichment results (sumstats_*_annot.txt) and integrates them with GWAS results via the Fisher's exact test overlap analysis) — https://github.com/toshi-kishikawa/OMARU
- **Snakemake** (Workflow engine used to run the integrated MWAS-GWAS pathway overlap step (Snakefiles/OMARU_Func_MWAS_GWAS.sm))
- **PASCAL** (Used with GWAS summary statistics (SCZ: 22,778 cases/35,362 controls; T2D: 77,418 cases/433,544 controls) to generate the GWAS pathway enrichment files compared against the MWAS results)

## Examples

```
snakemake -s Snakefiles/OMARU_Func_MWAS_GWAS.sm
```

## Evaluation signals

- Computed PFisher values must reproduce the reported 0.011 (SCZ) and 0.008 (T2D) within rounding tolerance.
- Significance of overlap must meet the specified threshold PFisher < 0.05.
- Contingency table dimensions must be 2x2 with counts consistent with the set of pathways significant at P<0.05 in both MWAS and GWAS results.
- Integrated run must write tables and figures to <OMARU_project_dir>/result/FUNC_QCed3/MWAS_GWAS/<Pathway_reference> and the TSV to MWAS_GWAS_pathway_overlap_PFisher.tsv.
- GWAS input files must have the expected two-column schema (pathway ID, p-value).

## Limitations

- Overlap significance depends on the enrichment P-value threshold (0.05); different thresholds will change the contingency tables and PFisher values.
- Requires matched pathway references between MWAS and GWAS analyses; results are reported per <Pathway_reference> (e.g. KEGG was selected as the reference of gene and biological pathway in the application example).
- OMARU does not currently integrate covariates directly into the case-control model (hidden confounder adjustment via covariate integration is not implemented), which may affect the upstream enrichment results feeding this test.

## Evidence

- [full_text] Task card workflow: Fisher's exact test construction and validation: [evidence span withheld — anchor status: not_found]
- [full_text] Validation of computed PFisher against reported values: [evidence span withheld — anchor status: not_found]

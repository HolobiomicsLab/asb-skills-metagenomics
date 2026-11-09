---
name: gwas-summary-statistics-pathway-enrichment
description: Use when GWAS summary statistics are available and the goal is pathway-level
  enrichment (PASCAL) followed by overlap testing against disease metagenome (MWAS)
  pathway enrichment at P<0.05.
when_to_use_negative:
- When raw genotype data (not GWAS summary statistics) are the input — PASCAL in this
  workflow consumes precomputed summary statistics only
- When the aim is metagenome-side pathway enrichment alone; use the OMARU functional
  association test (OMARU_Func_AS.sm / OMARU_Func_permutation.sm) without the GWAS
  overlap step
- When confounders should be integrated into the case-control model — the article
  notes covariate integration is not currently implemented in OMARU
edam_operation: http://edamontology.org/operation_3563
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: PASCAL
  role: Computes pathway enrichment from host GWAS summary statistics for SCZ and
    T2D
- name: OMARU
  role: Provides the MWAS pathway enrichment results (OMARU_Func_permutation.sm) and
    the integrated MWAS-GWAS overlap Snakefile (OMARU_Func_MWAS_GWAS.sm)
  repo: https://github.com/toshi-kishikawa/OMARU
- name: Snakemake
  role: Drives the integrated MWAS_GWAS workflow that writes overlap tables and figures
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nargab/lqac019
    title: 'OMARU: a robust and multifaceted pipeline for metagenome-wide association
      study'
  source_content_hash: sha256:fe6c5b56187623753bd30fe1c612ca6571104f0a5afb792d71bf8e9a6c9b9aae
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
  edam_operation: http://edamontology.org/operation_3563
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# gwas-summary-statistics-pathway-enrichment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute host germline pathway enrichment from GWAS summary statistics with PASCAL and test overlap with MWAS-enriched pathways via Fisher's exact test. In OMARU this links the metagenome and host genome at pathway level, yielding PFisher = 0.011 (SCZ) and 0.008 (T2D).

## When to use

Use when GWAS summary statistics are available and the goal is pathway-level enrichment (PASCAL) followed by overlap testing against disease metagenome (MWAS) pathway enrichment at P<0.05.

## When NOT to use

- When raw genotype data (not GWAS summary statistics) are the input — PASCAL in this workflow consumes precomputed summary statistics only
- When the aim is metagenome-side pathway enrichment alone; use the OMARU functional association test (OMARU_Func_AS.sm / OMARU_Func_permutation.sm) without the GWAS overlap step
- When confounders should be integrated into the case-control model — the article notes covariate integration is not currently implemented in OMARU

## Inputs

- SCZ GWAS summary statistics (22,778 cases / 35,362 controls)
- T2D GWAS summary statistics (77,418 cases / 433,544 controls)
- MWAS pathway enrichment results: sumstats_*_annot.txt from Snakefiles/OMARU_Func_permutation.sm (SCZ n=344; T2D n=171 metagenomic cohorts)
- KEGG/GO pathway reference annotation files (<pathway_reference>_annotatioin.txt.gz)

## Outputs

- <Phenotype>_GWAS_<Pathway_reference>.txt files (column 1: pathway ID, column 2: p-value)
- 2x2 contingency tables of MWAS/GWAS significant pathway counts
- Fisher's exact test P-values (PFisher = 0.011 SCZ, 0.008 T2D) saved to MWAS_GWAS_pathway_overlap_PFisher.tsv
- Tables and figures of MWAS-GWAS links at <OMARU_project_dir>/result/FUNC_QCed3/MWAS_GWAS/<Pathway_reference>

## How to apply

Run PASCAL on the host GWAS summary statistics (SCZ: 22,778 cases / 35,362 controls; T2D: 77,418 cases / 433,544 controls) to produce per-pathway enrichment files named <Phenotype>_GWAS_<Pathway_reference>.txt, with pathway ID in column 1 and p-value in column 2, placed in <OMARU_project_dir>/data. Define significantly enriched pathways in both the GWAS results and the MWAS pathway enrichment outputs (sumstats_*_annot.txt from Snakefiles/OMARU_Func_permutation.sm) at the P<0.05 threshold. Build a 2x2 contingency table of pathway counts (MWAS-significant vs GWAS-significant) for each phenotype, compute Fisher's exact test P-values with scipy.stats.fisher_exact, or run the integrated step via 'snakemake -s Snakefiles/OMARU_Func_MWAS_GWAS.sm', which writes tables and figures to <OMARU_project_dir>/result/FUNC_QCed3/MWAS_GWAS/<Pathway_reference>. Rationale: significant overlap indicates disease-specific links between the human genome and metagenome at the pathway level.

## Related tools

- **PASCAL** (Computes pathway enrichment from host GWAS summary statistics for SCZ and T2D)
- **OMARU** (Provides the MWAS pathway enrichment results (OMARU_Func_permutation.sm) and the integrated MWAS-GWAS overlap Snakefile (OMARU_Func_MWAS_GWAS.sm)) — https://github.com/toshi-kishikawa/OMARU
- **Snakemake** (Drives the integrated MWAS_GWAS workflow that writes overlap tables and figures)

## Examples

```
snakemake -s Snakefiles/OMARU_Func_MWAS_GWAS.sm
```

## Evaluation signals

- Computed PFisher must reproduce 0.011 (SCZ) and 0.008 (T2D) within rounding tolerance
- Overlap significance threshold is P<0.05 in both MWAS and GWAS pathway lists
- Contingency tables are 2x2 with consistent marginal counts against the number of tested pathways
- GWAS pathway files follow the two-column format (pathway ID, p-value) and reside in <OMARU_project_dir>/data
- Integrated run writes outputs to <OMARU_project_dir>/result/FUNC_QCed3/MWAS_GWAS/<Pathway_reference>

## Limitations

- Overlap is assessed only at the P<0.05 nominal threshold, not multiple-testing-corrected
- Metagenome analysis is highly dependent on reference databases; database choice affects pathway overlap
- Linking taxa/genes to host pathways depends on the pathway reference (e.g., KEGG, GO) selected in config.yaml (REF: PATH)

## Evidence

- [results] Significant overlaps were found with PFisher 0.011 SCZ / 0.008 T2D at P<0.05: [evidence span withheld — anchor status: not_found]
- [other] GWAS pathway files formatted as <Phenotype>_GWAS_<Pathway_reference>.txt placed in <OMARU_project_dir>/data: [evidence span withheld — anchor status: not_found]
- [other] Integrated snakemake step writes MWAS-GWAS tables and figures: [evidence span withheld — anchor status: not_found]

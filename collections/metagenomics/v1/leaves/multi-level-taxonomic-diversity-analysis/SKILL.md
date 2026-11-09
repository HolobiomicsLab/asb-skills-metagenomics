---
name: multi-level-taxonomic-diversity-analysis
description: Use when QCed2/QCed3 OMARU profiling tables (clade levels L2–L7 and gene
  abundance) are available for case-control metagenome cohorts and the goal is to
  test whether diversity differs between cases and controls.
when_to_use_negative:
- Input is raw FASTQ reads or read-QC outputs — profiling (OMARU_QCed1.sm / OMARU_QCed2.sm)
  must produce abundance tables first
- The goal is a case-control clade or gene association test (OMARU_Phyl_AS.sm / OMARU_Func_AS.sm),
  not a diversity comparison
- Covariates must be integrated directly into the case-control diversity model — covariate
  integration is not implemented in OMARU, so only post-hoc or confounding-factor
  correction paths apply
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: OMARU
  role: End-to-end workflow providing the QCed2 diversity assessment step, normalization/down-sampling
    of reads to common depth, and per-level abundance tables feeding the Shannon and
    Bray-Curtis computations
  repo: https://github.com/toshi-kishikawa/OMARU
- name: Snakemake
  role: Workflow engine executing the OMARU diversity Snakefile on the QCed2-selected
    samples
- name: vegan
  role: R package supplying vegan::adonis for PERMANOVA testing of Bray-Curtis beta-diversity
    between cases and controls
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqac019
    title: 'OMARU: a robust and multifaceted pipeline for metagenome-wide association
      study'
  source_content_hash: sha256:c59c4e58a98be5d59ba7568396d3a85f1635bc9b854e2134a4e144f1b65da365
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
  - vegan
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# multi-level-taxonomic-diversity-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Case-control assessment of metagenome diversity using OMARU outputs: Shannon alpha-diversity computed on clade relative abundances at taxonomic levels L2–L7 and on gene abundance, plus Bray-Curtis beta-diversity with PERMANOVA and MDS visualization. It detects disrupted diversity, as demonstrated for T2D (n=171) versus the null result in SCZ (n=344).

## When to use

Use when QCed2/QCed3 OMARU profiling tables (clade levels L2–L7 and gene abundance) are available for case-control metagenome cohorts and the goal is to test whether diversity differs between cases and controls.

## When NOT to use

- Input is raw FASTQ reads or read-QC outputs — profiling (OMARU_QCed1.sm / OMARU_QCed2.sm) must produce abundance tables first
- The goal is a case-control clade or gene association test (OMARU_Phyl_AS.sm / OMARU_Func_AS.sm), not a diversity comparison
- Covariates must be integrated directly into the case-control diversity model — covariate integration is not implemented in OMARU, so only post-hoc or confounding-factor correction paths apply

## Inputs

- OMARU phylogenetic profiling tables (relative abundance of clades at levels L2–L7) from QCed2-selected samples
- OMARU gene abundance tables from QCed2-selected samples
- Sample list with phenotype labels (QCed2_sample_list.txt; cases=1, controls=0)
- Condition-specific cohort definitions (T2D n=171: 170 patients/174 controls; SCZ n=344: 90 patients/81 controls per public dataset)

## Outputs

- Per-sample Shannon alpha-diversity values at each taxonomic level (L2–L7) and for gene abundance
- Bray-Curtis dissimilarity matrices per level
- PERMANOVA (vegan::adonis) P-values per level
- Non-metric multidimensional scaling (MDS) plots of sample grouping
- Compiled per-condition statistics table diversity_statistics.tsv

## How to apply

Run the OMARU diversity workflow downstream of profiling with Snakemake on the QCed2-selected samples (snakemake -s Snakefiles/OMARU_QCed2.sm) so phylogenetic and gene abundance tables exist. Down-sample reads from each sample to a common depth with the OMARU normalization step to remove sequencing-depth-driven diversity bias before estimating diversity. Compute Shannon alpha-diversity per sample on the relative abundance of clades at levels L2–L7 and on gene abundance, and test case-control differences at the multiple-testing-corrected threshold (P < 8.3e-3). Compute Bray-Curtis dissimilarities between samples at each level, run PERMANOVA with vegan::adonis, and visualize sample grouping by non-metric multidimensional scaling (MDS). Compile per-level and per-gene P-values per condition into diversity_statistics.tsv and compare against the published benchmarks for T2D and SCZ.

## Related tools

- **OMARU** (End-to-end workflow providing the QCed2 diversity assessment step, normalization/down-sampling of reads to common depth, and per-level abundance tables feeding the Shannon and Bray-Curtis computations) — https://github.com/toshi-kishikawa/OMARU
- **Snakemake** (Workflow engine executing the OMARU diversity Snakefile on the QCed2-selected samples)
- **vegan** (R package supplying vegan::adonis for PERMANOVA testing of Bray-Curtis beta-diversity between cases and controls)

## Examples

```
cd OMARU_project_dir && snakemake -s Snakefiles/OMARU_QCed2.sm
```

## Evaluation signals

- T2D alpha-diversity is significantly increased at taxonomic levels L3 and L4 with P < 8.3 × 10^-3 (multiple-testing-corrected threshold)
- T2D gene-abundance alpha-diversity is significant with P = 5.1 × 10^-3
- T2D beta-diversity is significant at levels L5–L7 with P < 1.0 × 10^-4
- SCZ shows no significant diversity differences (gene-level alpha-diversity P = 0.134), so a significant SCZ result indicates a pipeline deviation
- Confirm reads were down-sampled to a common depth before diversity estimation, otherwise depth-driven bias inflates alpha-diversity differences

## Limitations

- OMARU does not implement integration of covariates into the case-control model, so hidden confounding factors (e.g. sex, age, BMI) cannot be adjusted within the diversity test itself
- The SCZ metagenome showed little difference in diversities, so the diversity workflow may yield null results on cohorts without disrupted diversity
- Metagenome analysis is highly dependent on reference databases (e.g. UHGG-based phylogenetic reference); no specific tool or database versions are stated

## Evidence

- [other] In T2D, alpha-diversity was significantly increased at taxonomic levels L3 and L4 (P < 8.3 × 10–3) and in gene abundance data (P = 5.1 × 10–3), indicating disrupted diversity in T2D.: [evidence span withheld — anchor status: not_found]
- [other] Down-sample reads from each sample to a common depth with the OMARU normalization step to remove sequencing-depth-driven diversity bias before diversity estimation.: [evidence span withheld — anchor status: not_found]

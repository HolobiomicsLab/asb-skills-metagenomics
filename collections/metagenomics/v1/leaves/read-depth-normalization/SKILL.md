---
name: read-depth-normalization
description: Use when comparing case-control metagenomic diversity (alpha or beta)
  across samples whose read depths differ, e.g. T2D (n=171) or SCZ (n=344) fecal metagenomes,
  prior to Shannon/Bray-Curtis computation.
when_to_use_negative:
- Input is already a depth-normalized feature table from another pipeline (e.g. rarefied
  BIOM) — re-downsampling within OMARU is unnecessary
- The goal is differential abundance association testing (MWAS phylogenetic/gene tests
  via OMARU_Phyl_AS.sm or OMARU_Func_AS.sm), which uses covariate-adjusted models
  rather than depth down-sampling
- Sample QC is incomplete — samples should be selected via QCed2_sample_list.txt before
  diversity estimation
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: OMARU
  role: End-to-end workflow whose normalization step performs the read-depth down-sampling
    and diversity assessment
  repo: https://github.com/toshi-kishikawa/OMARU
- name: Snakemake
  role: Drives the OMARU diversity workflow (OMARU_QCed2.sm) that executes normalization
    and diversity estimation
  repo: https://snakemake.readthedocs.io/en/v5.1.4/index.html
- name: vegan
  role: R package used to run PERMANOVA (adonis) on Bray-Curtis dissimilarities after
    normalization
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqac019
    title: 'OMARU: a robust and multifaceted pipeline for metagenome-wide association
      study'
  source_content_hash: sha256:621917cfcfa17211cee5e4c2688fce243c5613981f0f1a182908e9833dc9473e
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# read-depth-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Down-samples shotgun metagenomic reads from each sample to a common sequencing depth using the OMARU normalization step, so that sequencing-depth-driven bias is removed before Shannon alpha-diversity and Bray-Curtis beta-diversity estimation on taxonomic (L2-L7) and gene abundance data.

## When to use

Use when comparing case-control metagenomic diversity (alpha or beta) across samples whose read depths differ, e.g. T2D (n=171) or SCZ (n=344) fecal metagenomes, prior to Shannon/Bray-Curtis computation.

## When NOT to use

- Input is already a depth-normalized feature table from another pipeline (e.g. rarefied BIOM) — re-downsampling within OMARU is unnecessary
- The goal is differential abundance association testing (MWAS phylogenetic/gene tests via OMARU_Phyl_AS.sm or OMARU_Func_AS.sm), which uses covariate-adjusted models rather than depth down-sampling
- Sample QC is incomplete — samples should be selected via QCed2_sample_list.txt before diversity estimation

## Inputs

- QCed2-selected sample list (QCed2_sample_list.txt) from OMARU profiling
- Phylogenetic abundance tables (clades L2-L7) from OMARU phylogenetic profiling
- Gene abundance tables from OMARU functional profiling (e.g. UniRef90/KEGG-based)
- Phenotype metadata (original_sample_list.txt with case/control labels)

## Outputs

- Depth-normalized (down-sampled) per-sample abundance tables
- Per-sample Shannon alpha-diversity values per taxonomic level and gene abundance
- Bray-Curtis dissimilarity matrices with PERMANOVA (vegan::adonis) P-values and MDS plots
- diversity_statistics.tsv compiling per-condition (T2D, SCZ) statistics

## How to apply

After QC and profiling steps complete (OMARU_read_QC.sm through OMARU_QCed2.sm producing phylogenetic and gene abundance tables), run the OMARU diversity workflow on the QCed2-selected samples. Normalize read depths by down-sampling each sample to a common depth with the OMARU normalization step, because raw depth differences otherwise inflate between-sample diversity comparisons. Compute Shannon alpha-diversity per sample on relative abundance of clades at taxonomic levels L2-L7 and on gene abundance, and compute Bray-Curtis dissimilarities for beta-diversity, testing case-control differences at the multiple-testing-corrected threshold (P < 8.3e-3). Judge success by reproducing the paper's expectations: T2D alpha-diversity significant at L3/L4 (P < 8.3e-3) and gene level (P = 5.1e-3), T2D beta-diversity significant at L5-L7 (P < 1.0e-4), and no significant SCZ diversity differences (gene-level alpha P = 0.134).

## Related tools

- **OMARU** (End-to-end workflow whose normalization step performs the read-depth down-sampling and diversity assessment) — https://github.com/toshi-kishikawa/OMARU
- **Snakemake** (Drives the OMARU diversity workflow (OMARU_QCed2.sm) that executes normalization and diversity estimation) — https://snakemake.readthedocs.io/en/v5.1.4/index.html
- **vegan** (R package used to run PERMANOVA (adonis) on Bray-Curtis dissimilarities after normalization)

## Examples

```
cd OMARU_project_dir && snakemake -s Snakefiles/OMARU_QCed2.sm
```

## Evaluation signals

- Down-sampling yields equal/common read depth across all samples, removing depth-driven diversity bias
- Shannon alpha-diversity computed at each taxonomic level L2-L7 and on gene abundance; T2D significance only at L3, L4 and gene level (P < 8.3e-3; gene P = 5.1e-3)
- T2D beta-diversity significant at L5-L7 (P < 1.0e-4); SCZ shows no significant diversity differences (gene-level alpha P = 0.134)
- PERMANOVA (vegan::adonis) and MDS visualization reproduce grouping reported in the paper
- diversity_statistics.tsv contains per-level and per-gene P-values for both T2D and SCZ

## Limitations

- Covariate/confounder adjustment is not integrated into the case-control model in OMARU itself; covariates must be handled as in the association-test steps
- Diversity results are dataset-dependent: T2D showed disrupted diversity while SCZ showed little difference, so non-significant SCZ results are expected and not a failure
- Reference database preparation is time/resource-intensive (~200 GB disk) and profiling quality depends on the chosen references (UHGG, UniRef90/KEGG)

## Evidence

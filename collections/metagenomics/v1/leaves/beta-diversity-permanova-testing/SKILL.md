---
name: beta-diversity-permanova-testing
description: Use when OMARU profiling tables for shotgun metagenomes (e.g., T2D n=171
  or SCZ n=344 case-control cohorts) are available and the goal is to test whether
  between-sample community composition (beta-diversity) differs between cases and
  controls.
when_to_use_negative:
- The aim is within-sample alpha-diversity (Shannon index per sample) rather than
  between-sample compositional dissimilarity
- Input is raw FASTQ that has not passed OMARU read QC (OMARU_read_QC.sm), profiling
  (OMARU_QCed1.sm), and abundance QC (OMARU_QCed2.sm) — diversity on unfiltered profiles
  is confounded by depth and outliers
- A single sample group without a case-control phenotype contrast is available, so
  PERMANOVA has no grouping variable to test
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3050
tools:
- name: OMARU
  role: End-to-end workflow providing phylogenetic (UHGG-based) and gene (UniRef90)
    abundance tables plus normalization/down-sampling that precede beta-diversity
    estimation
  repo: https://github.com/toshi-kishikawa/OMARU
- name: Snakemake
  role: Executes the OMARU diversity Snakefile steps on QCed2-selected samples
  repo: https://github.com/toshi-kishikawa/OMARU
- name: vegan
  role: R package whose adonis function performs the PERMANOVA test on Bray-Curtis
    dissimilarities
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqac019
    title: 'OMARU: a robust and multifaceted pipeline for metagenome-wide association
      study'
  source_content_hash: sha256:5a68e9024cc1131c7d8461afa8a3b0aff6186af2fc8e1c3f317f51632c969f7b
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
  - http://edamontology.org/topic_3050
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# beta-diversity-permanova-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assesses whether case-control metagenomes differ in community composition by computing Bray-Curtis dissimilarities on OMARU relative-abundance profiles (taxonomic levels L2–L7 and gene abundance) and testing group separation with PERMANOVA (vegan::adonis), visualized by non-metric multidimensional scaling (MDS).

## When to use

Use when OMARU profiling tables for shotgun metagenomes (e.g., T2D n=171 or SCZ n=344 case-control cohorts) are available and the goal is to test whether between-sample community composition (beta-diversity) differs between cases and controls.

## When NOT to use

- The aim is within-sample alpha-diversity (Shannon index per sample) rather than between-sample compositional dissimilarity
- Input is raw FASTQ that has not passed OMARU read QC (OMARU_read_QC.sm), profiling (OMARU_QCed1.sm), and abundance QC (OMARU_QCed2.sm) — diversity on unfiltered profiles is confounded by depth and outliers
- A single sample group without a case-control phenotype contrast is available, so PERMANOVA has no grouping variable to test

## Inputs

- QCed2-selected sample list (QCed2_sample_list.txt) and FASTQ inputs in OMARU_project_dir
- OMARU phylogenetic relative-abundance tables at taxonomic levels L2–L7 (UHGG/CHOCO-based)
- OMARU gene abundance table (UniRef90-based)
- Sample metadata with Phenotype coded 1 (case) / 0 (control) from original_sample_list.txt

## Outputs

- Per-level Bray-Curtis dissimilarity matrices and PERMANOVA (adonis) P-values for T2D and SCZ case-control comparisons
- Non-metric multidimensional scaling (MDS) plots of sample grouping per taxonomic level
- Compiled per-condition statistics in diversity_statistics.tsv

## How to apply

Run the OMARU diversity workflow downstream of profiling with Snakemake on the QCed2-selected samples (cd OMARU_project_dir; snakemake -s Snakefiles/OMARU_QCed2.sm) so that phylogenetic and gene abundance tables are available. Down-sample reads from each sample to a common depth with the OMARU normalization step to remove sequencing-depth-driven diversity bias before estimating dissimilarities. Compute Bray-Curtis dissimilarity between samples at each taxonomic level (L2–L7) and on gene abundance, then test case-control separation with vegan::adonis PERMANOVA and visualize grouping by non-metric MDS. Assess significance at the multiple-testing-corrected threshold (P < 8.3e-3) and compile per-level P-values into diversity_statistics.tsv. In the OMARU paper this reproduced T2D beta-diversity significance at L5–L7 (P < 1.0e-4), while SCZ showed no significant diversity differences.

## Related tools

- **OMARU** (End-to-end workflow providing phylogenetic (UHGG-based) and gene (UniRef90) abundance tables plus normalization/down-sampling that precede beta-diversity estimation) — https://github.com/toshi-kishikawa/OMARU
- **Snakemake** (Executes the OMARU diversity Snakefile steps on QCed2-selected samples) — https://github.com/toshi-kishikawa/OMARU
- **vegan** (R package whose adonis function performs the PERMANOVA test on Bray-Curtis dissimilarities)

## Examples

```
cd OMARU_project_dir; snakemake -s Snakefiles/OMARU_QCed2.sm
```

## Evaluation signals

- T2D beta-diversity reproduces significance at taxonomic levels L5–L7 with P < 1.0e-4 under the corrected threshold
- SCZ beta-diversity shows no significant case-control differences, matching the reported absence of diversity effects (SCZ gene-level alpha P = 0.134)
- All samples were down-sampled to a common sequencing depth before dissimilarity computation (no depth-driven bias)
- Per-level and per-gene P-values are written to diversity_statistics.tsv with each level L2–L7 represented
- MDS plots show the case-control grouping structure consistent with the adonis R2/P output

## Limitations

- Statistical significance depends on OMARU's correction of hidden confounding factors and multiple testing comparisons; covariate integration into the case-control model is not currently implemented in OMARU
- SCZ sample with singleton genes beyond four standard deviations (outlier of phylogenetic and gene abundance data) must be excluded or it will distort dissimilarity structure
- One SCZ sample exclusion and QCed2/QCed3 sample selection choices affect the tested cohort (T2D n=171, SCZ n=344)

## Evidence

- [discussion] OMARU provides end-to-end analysis from the processing of sequencing data, such as QC of reads and samples, to the three major analyses and the assessment of diversities.: "OMARU provides end-to-end analysis from the processing of sequencing data, such as QC of reads and samples, to the three major analyses and the assessment of diversities."
- [discussion] The metagenome of SCZ had little difference of diversities while T2D had significant ones compared to healthy controls.: "The metagenome of SCZ had little difference of diversities while T2D had significant ones compared to healthy controls."
- [abstract] including correction of hidden confounding factors and application of multiple testing comparisons: "OMARU rigorously controls the statistical significance of the analysis results, including correction of hidden confounding factors and application of multiple testing comparisons"
- [results] we excluded one SCZ sample that had singleton genes beyond four standard deviations and was an outlier of both phylogenetic and gene abundance data: "we excluded one SCZ sample that had singleton genes beyond four standard deviations and was an outlier of both phylogenetic and gene abundance data"
- [readme] Various options are available according to the function of Snakemake: "Various options are available d according to the function of `Snakemake` such as:"

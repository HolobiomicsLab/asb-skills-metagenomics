---
name: mds-ordination-visualization
description: Use when assessing case-control beta-diversity of metagenome abundance
  profiles (taxonomic levels L2-L7 or gene abundance) and you need ordination plots
  plus a significance test of sample grouping (e.g. T2D vs controls).
when_to_use_negative:
- The goal is alpha-diversity (per-sample Shannon) rather than between-sample beta-diversity
  ordination
- Input data have not been depth-normalized, so dissimilarities are driven by sequencing
  depth rather than composition
- No case-control phenotype labels exist, so PERMANOVA grouping cannot be tested
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_2269
tools:
- name: OMARU
  role: End-to-end workflow providing the profiling tables, depth normalization, and
    diversity assessment (including beta-diversity/ordination) consumed by this skill
  repo: https://github.com/toshi-kishikawa/OMARU
- name: Snakemake
  role: Runs the OMARU_QCed2.sm workflow that generates the abundance tables and diversity
    outputs downstream of profiling
  repo: https://snakemake.readthedocs.io/en/v5.1.4/index.html
- name: vegan
  role: R package supplying vegan::adonis for PERMANOVA testing of Bray-Curtis dissimilarities
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqac019
    title: 'OMARU: a robust and multifaceted pipeline for metagenome-wide association
      study'
  source_content_hash: sha256:a59698b954122ce7090586f805663d900dae59bc97d7c5450edfca2adb9bdfdc
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
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# mds-ordination-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes Bray-Curtis dissimilarities between metagenome samples and visualizes case-control grouping via non-metric multidimensional scaling (MDS), with PERMANOVA (vegan::adonis) to test significance of diversity differences, as used in OMARU's T2D/SCZ diversity assessment.

## When to use

Use when assessing case-control beta-diversity of metagenome abundance profiles (taxonomic levels L2-L7 or gene abundance) and you need ordination plots plus a significance test of sample grouping (e.g. T2D vs controls).

## When NOT to use

- The goal is alpha-diversity (per-sample Shannon) rather than between-sample beta-diversity ordination
- Input data have not been depth-normalized, so dissimilarities are driven by sequencing depth rather than composition
- No case-control phenotype labels exist, so PERMANOVA grouping cannot be tested

## Inputs

- QCed2-selected sample abundance tables at taxonomic levels L2-L7 (relative abundance of clades)
- Gene abundance table from OMARU profiling
- Sample list with phenotype labels (QCed2_sample_list.txt)
- Depth-normalized (down-sampled) read counts per sample

## Outputs

- Bray-Curtis dissimilarity matrices per taxonomic level and gene abundance
- PERMANOVA (vegan::adonis) P-values per level
- MDS ordination plots showing case-control sample grouping
- Compiled per-condition statistics (diversity_statistics.tsv)

## How to apply

Start from the OMARU phylogenetic and gene abundance tables produced by running the diversity workflow downstream of profiling on QCed2-selected samples (snakemake -s Snakefiles/OMARU_QCed2.sm). First down-sample reads from each sample to a common depth with the OMARU normalization step so sequencing-depth-driven bias does not inflate dissimilarities. Then compute Bray-Curtis dissimilarity matrices between samples at each taxonomic level (L2-L7) and on gene abundance, run PERMANOVA with vegan::adonis per level, and render sample grouping by non-metric multidimensional scaling (MDS). Judge correctness by checking whether the expected pattern reproduces: T2D beta-diversity significant at L5-L7 (P<1.0e-4) with no significant SCZ diversity differences (gene-level alpha P=0.134).

## Related tools

- **OMARU** (End-to-end workflow providing the profiling tables, depth normalization, and diversity assessment (including beta-diversity/ordination) consumed by this skill) — https://github.com/toshi-kishikawa/OMARU
- **Snakemake** (Runs the OMARU_QCed2.sm workflow that generates the abundance tables and diversity outputs downstream of profiling) — https://snakemake.readthedocs.io/en/v5.1.4/index.html
- **vegan** (R package supplying vegan::adonis for PERMANOVA testing of Bray-Curtis dissimilarities)

## Examples

```
cd OMARU_project_dir; snakemake -s Snakefiles/OMARU_QCed2.sm
```

## Evaluation signals

- Reproduced finding: T2D beta-diversity significant at taxonomic levels L5-L7 with P<1.0e-4
- Reproduced finding: no significant SCZ diversity differences (gene-level alpha P=0.134)
- Depth-normalization step applied before dissimilarity computation (common down-sampled read depth across samples)
- MDS plot shows separation of cases vs controls consistent with PERMANOVA significance at the same taxonomic level
- Per-level and per-gene P-values collected and written to diversity_statistics.tsv

## Limitations

- The metagenome of SCZ showed little difference of diversities, so the skill may yield null results for conditions without diversity disruption
- Depth normalization must precede dissimilarity estimation or sequencing-depth bias can drive apparent grouping
- OMARU analysis remains highly dependent on reference databases, which affects the abundance tables feeding the ordination

## Evidence

- [other] Compute Bray-Curtis dissimilarities between samples at each level and run PERMANOVA with vegan::adonis, visualizing sample grouping by non-metric multidimensional scaling (MDS).: [evidence span withheld — anchor status: not_found]
- [other] Down-sample reads from each sample to a common depth with the OMARU normalization step to remove sequencing-depth-driven diversity bias before diversity estimation.: [evidence span withheld — anchor status: not_found]

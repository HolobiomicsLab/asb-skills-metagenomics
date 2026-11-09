---
name: command-line-r-script-development
description: Use when building reproducible R analysis scripts that consume ranked
  abundance summary tables and must emit statistical result tables (p-value, log2FoldChange)
  plus graphical outputs, invoked via Rscript from a shell.
when_to_use_negative:
- Input is still per-read DIAMOND annotation output — it must first be condensed into
  ranked abundance summaries by DIAMOND_analysis_counter.py
- The goal is raw read preprocessing (adaptor trimming, rRNA removal) — that belongs
  to the Trimmomatic/SortMeRNA preprocessing stage, not the R analysis stage
- The analysis must run under Python 3 only — SAMSA2's aggregation scripts require
  Python 2, though the R stage itself is independent
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: SAMSA2
  role: Pipeline whose analysis stage packages the command-line-executable R scripts
    (run_DESeq_stats.R, diversity_stats.R, make_DESeq_PCA.R, make_DESeq_heatmap.R,
    make_combined_graphs.R)
  repo: https://github.com/transcript/samsa2
- name: DESeq
  role: Statistical framework used inside run_DESeq_stats.R to compute most significantly
    differing features between control vs. experimental samples
- name: DIAMOND
  role: Upstream aligner whose per-read annotation results are aggregated into the
    ranked abundance summaries that the R scripts consume
  repo: https://github.com/bbuchfink/diamond
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-018-2189-z
    title: 'SAMSA2: a standalone metatranscriptome analysis pipeline'
  source_content_hash: sha256:22d221d2b63dc6b5a669f6a90f7aaf5980ff7cda5e060dc3bcca251f2cb55376
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/transcript/samsa2
  tools:
  - DESeq
  - DIAMOND
  - R
  - SAMSA2
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# command-line-r-script-development

## Summary

Develop R scripts that run from the command line to perform statistical and graphical analysis of abundance count tables, as exemplified by the SAMSA2 analysis stage where DESeq-based scripts produce p-values, log2FoldChange scores, PCA plots, heatmaps, and stacked-bar graphs.

## When to use

Use when building reproducible R analysis scripts that consume ranked abundance summary tables and must emit statistical result tables (p-value, log2FoldChange) plus graphical outputs, invoked via Rscript from a shell.

## When NOT to use

- Input is still per-read DIAMOND annotation output — it must first be condensed into ranked abundance summaries by DIAMOND_analysis_counter.py
- The goal is raw read preprocessing (adaptor trimming, rRNA removal) — that belongs to the Trimmomatic/SortMeRNA preprocessing stage, not the R analysis stage
- The analysis must run under Python 3 only — SAMSA2's aggregation scripts require Python 2, though the R stage itself is independent

## Inputs

- Ranked abundance summary tables produced by DIAMOND_analysis_counter.py (count matrices per metatranscriptome sample)
- Sample grouping metadata distinguishing control vs. experimental metatranscriptomes

## Outputs

- DESeq results table with p-value and log2FoldChange columns for every feature (organisms or functions)
- PCA plot of sample-level variance from variance-stabilized count data
- Heatmap of the top significantly differing features
- Stacked-bar graphs of relative abundances
- Shannon and Simpson diversity indices per sample

## How to apply

Import the ranked abundance summaries produced by DIAMOND_analysis_counter.py into R as count matrices per metatranscriptome sample. Run the DESeq2 framework (run_DESeq_stats.R) to test control vs. experimental groups or individual metatranscriptomes, extracting a p-value and log2FoldChange for every annotated feature (organisms or specific functions). Compute Shannon and Simpson diversity indices per sample (diversity_stats.R) from the sorted abundance counts, and generate a PCA plot of sample-level variance from variance-stabilized DESeq count data (make_DESeq_PCA.R). Produce heatmaps of top significantly differing features (make_DESeq_heatmap.R) and stacked-bar graphs of relative abundances (make_combined_graphs.R). Design each script to be executable from the command line (e.g. Rscript) so it fits into the pipeline after the Python aggregation stage, since SAMSA2 emphasizes 'better, more polished R scripts that can be executed from the command line'.

## Related tools

- **SAMSA2** (Pipeline whose analysis stage packages the command-line-executable R scripts (run_DESeq_stats.R, diversity_stats.R, make_DESeq_PCA.R, make_DESeq_heatmap.R, make_combined_graphs.R)) — https://github.com/transcript/samsa2
- **DESeq** (Statistical framework used inside run_DESeq_stats.R to compute most significantly differing features between control vs. experimental samples)
- **DIAMOND** (Upstream aligner whose per-read annotation results are aggregated into the ranked abundance summaries that the R scripts consume) — https://github.com/bbuchfink/diamond

## Examples

```
Rscript run_DESeq_stats.R --input abundance_summary.tsv --groups control,experimental
```

## Evaluation signals

- Each R script executes from the command line without errors
- The DESeq output table contains p-value and log2FoldChange columns for every feature
- PCA, heatmap, and stacked-bar graphical outputs are generated successfully
- Diversity outputs include both Shannon and Simpson indices computed from the sorted abundance counts
- Differing features are reported for the correct comparison (control vs. experimental groups or individual metatranscriptomes)

## Limitations

- SAMSA2 aggregation scripts require Python 2.7 and the pipeline was only tested on Linux, even though the OS claim is broader
- Feature resolution depends on the local reference database choice (e.g. RefSeq for organisms, SEED Subsystems for functional ontologies); custom databases must be DIAMOND-indexed before annotation
- Scripts operate on abundance summaries, so errors in the upstream DIAMOND annotation or aggregation steps propagate into the statistics

## Evidence

- [methods] use "run_DESeq_stats.R" to determine the most significantly differing features between either individual metatranscriptomes, or control vs. experimental groups: "use "run_DESeq_stats.R" to determine the most significantly differing features between either individual metatranscriptomes, or control vs. experimental groups"
- [readme] Import these abundance summaries into R and use "run_DESeq_stats.R": [evidence span withheld — anchor status: external_doc]
- [readme] Better, more polished R scripts that can be executed from the command line.: "* Better, more polished R scripts that can be executed from the command line."
- [readme] R scripts use DESeq to compute most significantly different features between control vs. experimental samples.: "R scripts use DESeq to compute most significantly different features between control vs. experimental samples. These R scripts generate a tabular output with assigned p-values and log2FoldChange"
- [methods] PCA plots and other graphical outputs.: "PCA plots and other graphical outputs."

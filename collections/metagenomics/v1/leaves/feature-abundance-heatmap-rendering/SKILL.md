---
name: feature-abundance-heatmap-rendering
description: Use when abundance summary tables from DIAMOND_analysis_counter.py have
  already been tested with DESeq in the SAMSA2 pipeline and a heatmap of top significantly
  differing features is needed.
when_to_use_negative:
- Input is raw DIAMOND per-read output that has not yet been condensed by DIAMOND_analysis_counter.py
  into ranked abundance summaries
- No differential statistics have been computed yet — run run_DESeq_stats.R first
  to identify which features to display
- The goal is sample-level variance structure rather than feature-level abundance
  patterns — a PCA plot (make_DESeq_PCA.R) is the appropriate output
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SAMSA2
  role: Pipeline providing the DESeq-based R analysis scripts, including make_DESeq_heatmap.R,
    that consume aggregated abundance counts
  repo: https://github.com/transcript/samsa2
- name: DESeq
  role: Computes the significantly differing features (p-values and log2FoldChange)
    that define which rows appear in the heatmap
- name: DIAMOND
  role: Upstream aligner whose annotation results are aggregated into the abundance
    summaries visualized by the heatmap
  repo: https://github.com/bbuchfink/diamond
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-018-2189-z
    title: 'SAMSA2: a standalone metatranscriptome analysis pipeline'
  source_content_hash: sha256:81f5b55523cedc5af47c2ce290e2e6c61aac247f73d4c6a30643525b05e30016
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/transcript/samsa2
  tools:
  - DESeq
  - DIAMOND
  - R
  - SAMSA2
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# Feature abundance heatmap rendering

## Summary

Render heatmaps of the most significantly differing features (organisms or functions) from SAMSA2's DESeq differential abundance results, providing a visual matrix view of per-sample abundance for top features.

## When to use

Use when abundance summary tables from DIAMOND_analysis_counter.py have already been tested with DESeq in the SAMSA2 pipeline and a heatmap of top significantly differing features is needed.

## When NOT to use

- Input is raw DIAMOND per-read output that has not yet been condensed by DIAMOND_analysis_counter.py into ranked abundance summaries
- No differential statistics have been computed yet — run run_DESeq_stats.R first to identify which features to display
- The goal is sample-level variance structure rather than feature-level abundance patterns — a PCA plot (make_DESeq_PCA.R) is the appropriate output

## Inputs

- Ranked abundance summary tables produced by DIAMOND_analysis_counter.py (count matrices per metatranscriptome sample)
- DESeq results table with p-value and log2FoldChange columns per feature from run_DESeq_stats.R

## Outputs

- Heatmap of the top significantly differing features (organisms or specific functions) across samples

## How to apply

Import the ranked abundance summary tables produced by DIAMOND_analysis_counter.py into R as count matrices for each metatranscriptome sample. Run run_DESeq_stats.R first to obtain p-values and log2FoldChange scores for every annotated feature, identifying the most significantly differing features between control vs. experimental groups (or individual metatranscriptomes). Then use make_DESeq_heatmap.R to draw the heatmap restricted to these top significantly differing features. Variance-stabilized DESeq count data underlies the sample-level comparisons, complementing the PCA plot from make_DESeq_PCA.R and stacked-bar graphs from make_combined_graphs.R.

## Related tools

- **SAMSA2** (Pipeline providing the DESeq-based R analysis scripts, including make_DESeq_heatmap.R, that consume aggregated abundance counts) — https://github.com/transcript/samsa2
- **DESeq** (Computes the significantly differing features (p-values and log2FoldChange) that define which rows appear in the heatmap)
- **DIAMOND** (Upstream aligner whose annotation results are aggregated into the abundance summaries visualized by the heatmap) — https://github.com/bbuchfink/diamond

## Examples

```
Rscript make_DESeq_heatmap.R
```

## Evaluation signals

- The heatmap displays only the top significantly differing features, i.e. those carrying assigned p-values and log2FoldChange scores from the DESeq results table
- The script executes from the command line and produces the heatmap graphical output without errors
- Columns/samples in the heatmap correspond to the individual metatranscriptomes or control vs. experimental groups defined in the DESeq comparison
- Heatmap abundance values are consistent with the ranked abundance summary tables imported from DIAMOND_analysis_counter.py

## Limitations

- Heatmap content depends on DESeq differential testing between defined sample groups; with no control/experimental contrast the 'most significantly differing features' are undefined
- SAMSA2 aggregation scripts require Python 2.7, and the pipeline was only tested on Linux despite claiming any OS with Python 2.7
- The heatmaps show a subset of top features; full relative-abundance composition is better represented by the combined stacked-bar graphs from make_combined_graphs.R

## Evidence

- [readme] README analysis phase description: "R scripts use DESeq to compute most significantly different features between control vs. experimental samples. These R scripts generate a tabular output with assigned p-values and log2FoldChange"
- [methods] Article methods on R graphical outputs: "PCA plots and other graphical outputs."
- [other] Source card workflow on heatmap step: [evidence span withheld — anchor status: not_found]
- [methods] Article methods on DESeq feature testing: "use "run_DESeq_stats.R" to determine the most significantly differing features"

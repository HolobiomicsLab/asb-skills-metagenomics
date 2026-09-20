---
name: sample-pca-visualization
description: Use when ranked abundance summary tables (organism or function counts)
  from SAMSA2's DIAMOND_analysis_counter.py step have been aggregated and you need
  a sample-level dimensionality-reduction plot to inspect control vs. experimental
  group separation.
when_to_use_negative:
- Input is raw per-read DIAMOND output rather than aggregated ranked abundance summaries
  — aggregation must run first
- You need strain-level or activity inference from 16S rRNA data — SAMSA2's metatranscriptome
  PCA only reflects mRNA-based organism/function profiles
- You only need differential-feature statistics (p-values/log2FoldChange) with no
  sample-level visualization — run_DESeq_stats.R alone suffices
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0091
tools:
- name: SAMSA2
  role: Pipeline whose R analysis stage provides make_DESeq_PCA.R for PCA plotting
    of variance-stabilized DESeq count data
  repo: https://github.com/transcript/samsa2
- name: DIAMOND
  role: Upstream aligner whose per-read annotation results are aggregated into the
    ranked abundance summaries feeding the PCA
  repo: https://github.com/bbuchfink/diamond
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-018-2189-z
    title: 'SAMSA2: a standalone metatranscriptome analysis pipeline'
  source_content_hash: sha256:74f89032ee0cbd9fc363c25dcc3a7c1bf244f7ced9fbcea8db5361a317797ea9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/transcript/samsa2
  tools:
  - DESeq
  - DIAMOND
  - R
  - SAMSA2
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# sample-pca-visualization

## Summary

Generates PCA plots of sample-level variance from variance-stabilized DESeq count data as part of the SAMSA2 R analysis stage, letting practitioners visually assess clustering and separation of control vs. experimental metatranscriptome samples alongside other graphical outputs.

## When to use

Use when ranked abundance summary tables (organism or function counts) from SAMSA2's DIAMOND_analysis_counter.py step have been aggregated and you need a sample-level dimensionality-reduction plot to inspect control vs. experimental group separation.

## When NOT to use

- Input is raw per-read DIAMOND output rather than aggregated ranked abundance summaries — aggregation must run first
- You need strain-level or activity inference from 16S rRNA data — SAMSA2's metatranscriptome PCA only reflects mRNA-based organism/function profiles
- You only need differential-feature statistics (p-values/log2FoldChange) with no sample-level visualization — run_DESeq_stats.R alone suffices

## Inputs

- Ranked abundance summary tables from DIAMOND_analysis_counter.py (per metatranscriptome, organisms or functions)
- Variance-stabilized DESeq count data derived from run_DESeq_stats.R

## Outputs

- PCA plot of sample-level variance (graphical output)
- Optionally companion DESeq results table with p-value and log2FoldChange columns per feature, heatmaps, and stacked-bar abundance graphs

## How to apply

Import the ranked abundance summary tables produced by DIAMOND_analysis_counter.py into R as count matrices for each metatranscriptome sample. Run the DESeq2 framework (e.g. via run_DESeq_stats.R) to test control vs. experimental groups or individual metatranscriptomes, then use the variance-stabilized DESeq count data as input to make_DESeq_PCA.R to compute principal components of sample-level variance. Features in the count matrices can be either organisms or specific functions, so run PCA separately per annotation type as needed. The rationale is that variance-stabilized counts satisfy DESeq's assumptions for distance-based ordination, giving a plot that reflects biological rather than count-magnitude variance. Combine the PCA with DESeq statistics (p-values, log2FoldChange) and other graphical outputs such as heatmaps of top differing features and stacked-bar relative-abundance graphs for a complete sample-level picture.

## Related tools

- **SAMSA2** (Pipeline whose R analysis stage provides make_DESeq_PCA.R for PCA plotting of variance-stabilized DESeq count data) — https://github.com/transcript/samsa2
- **DIAMOND** (Upstream aligner whose per-read annotation results are aggregated into the ranked abundance summaries feeding the PCA) — https://github.com/bbuchfink/diamond

## Examples

```
Rscript make_DESeq_PCA.R
```

## Evaluation signals

- The PCA plot renders without errors when make_DESeq_PCA.R is executed from the command line
- The plot is built from variance-stabilized DESeq count data, not raw counts
- Samples from the same group (control vs. experimental) show coherent positioning, and group separation is consistent with DESeq log2FoldChange differences
- The companion DESeq results table contains p-value and log2FoldChange columns for every annotated feature (organism or function)
- Graphical outputs (PCA, heatmap, stacked-bar) are all produced as part of the same analysis stage

## Limitations

- R analysis stage requires Python 2.7-based aggregation output upstream; SAMSA2 was tested on Linux only despite claiming any OS supporting Python 2.7
- DIAMOND is set by default to run in fast mode, prioritizing speed over accuracy, which affects the underlying annotations; sensitive mode increases memory requirements significantly
- Datasets are described only as available in the SAMSA2 repository with no accession IDs, limiting independent reproduction of the plotted example

## Evidence

- [readme] R analysis stage produces PCA plots from variance-stabilized DESeq count data: [evidence span withheld — anchor status: external_doc]
- [readme] SAMSA2 version 2 adds PCA and other graphical outputs: "PCA plots and other graphical outputs."
- [methods] DESeq statistics define the features analyzed between control and experimental groups: "use "run_DESeq_stats.R" to determine the most significantly differing features between either individual metatranscriptomes, or control vs. experimental groups"
- [readme] R scripts produce tabular output with p-values and log2FoldChange per feature: "These R scripts generate a tabular output with assigned p-values and log2FoldChange scores for each feature."

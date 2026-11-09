---
name: differential-abundance-statistics-analysis
description: Use when ranked abundance summary tables from aggregated metatranscriptome
  annotations are available and the goal is to find features with significant abundance
  differences between control and experimental groups.
when_to_use_negative:
- Input is still raw per-read DIAMOND annotation output that has not yet been condensed
  into ranked abundance summary tables by the Python aggregation step
- The goal is annotation itself (mapping reads to protein databases with DIAMOND)
  rather than downstream statistical comparison
- A web-based or non-R differential analysis workflow is required; SAMSA2's analysis
  stage assumes R scripts executable from the command line
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: SAMSA2
  role: Pipeline supplying the aggregation output (DIAMOND_analysis_counter.py) and
    the R analysis scripts (run_DESeq_stats.R, make_DESeq_PCA.R, make_DESeq_heatmap.R,
    make_combined_graphs.R)
  repo: https://github.com/transcript/samsa2
- name: DESeq
  role: Statistical framework inside run_DESeq_stats.R computing most significantly
    differing features between control vs. experimental samples
- name: DIAMOND
  role: Upstream aligner whose per-read annotation results are aggregated into the
    abundance summaries consumed by this analysis stage
  repo: https://github.com/bbuchfink/diamond
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-018-2189-z
    title: 'SAMSA2: a standalone metatranscriptome analysis pipeline'
  source_content_hash: sha256:a10cd1f16da96b6765aec623a119c12bf66f07ec329bf55d01fc237dad00df7f
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
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# differential-abundance-statistics-analysis

## Summary

Statistical identification of significantly differing features (organisms or functions) between metatranscriptome sample groups using DESeq in R, followed by graphical summaries such as PCA plots, heatmaps, and stacked-bar abundance graphs within the SAMSA2 pipeline.

## When to use

Use when ranked abundance summary tables from aggregated metatranscriptome annotations are available and the goal is to find features with significant abundance differences between control and experimental groups.

## When NOT to use

- Input is still raw per-read DIAMOND annotation output that has not yet been condensed into ranked abundance summary tables by the Python aggregation step
- The goal is annotation itself (mapping reads to protein databases with DIAMOND) rather than downstream statistical comparison
- A web-based or non-R differential analysis workflow is required; SAMSA2's analysis stage assumes R scripts executable from the command line

## Inputs

- Ranked abundance summary tables produced by DIAMOND_analysis_counter.py from SAMSA2 DIAMOND annotation results
- Sample grouping information distinguishing control vs. experimental metatranscriptomes

## Outputs

- Tabular DESeq results with p-values and log2FoldChange scores per feature (organism or function)
- PCA plot of sample-level variance from variance-stabilized DESeq count data
- Heatmap of top significantly differing features
- Combined stacked-bar graphs of relative abundances
- Shannon and Simpson diversity indices per sample (diversity_stats.R)

## How to apply

Import the ranked abundance summaries produced by DIAMOND_analysis_counter.py into R as count matrices for each metatranscriptome sample. Run the DESeq2 framework via run_DESeq_stats.R to test either individual metatranscriptomes against each other or control vs. experimental groups, extracting a p-value and a log2FoldChange score for every annotated feature. Optionally compute Shannon and Simpson diversity indices per sample from the sorted abundance counts (diversity_stats.R). Generate a PCA plot of sample-level variance from variance-stabilized DESeq count data (make_DESeq_PCA.R), heatmaps of the top significantly differing features (make_DESeq_heatmap.R), and combined stacked-bar graphs of relative abundances (make_combined_graphs.R). The rationale is that the per-read DIAMOND annotation output is condensed by Python aggregation scripts into summary tables, which DESeq can then model statistically; features may be organisms or specific functions.

## Related tools

- **SAMSA2** (Pipeline supplying the aggregation output (DIAMOND_analysis_counter.py) and the R analysis scripts (run_DESeq_stats.R, make_DESeq_PCA.R, make_DESeq_heatmap.R, make_combined_graphs.R)) — https://github.com/transcript/samsa2
- **DESeq** (Statistical framework inside run_DESeq_stats.R computing most significantly differing features between control vs. experimental samples)
- **DIAMOND** (Upstream aligner whose per-read annotation results are aggregated into the abundance summaries consumed by this analysis stage) — https://github.com/bbuchfink/diamond

## Examples

```
Rscript run_DESeq_stats.R
```

## Evaluation signals

- Each R script executes from the command line without errors
- The DESeq results table contains a p-value column and a log2FoldChange column covering every annotated feature
- PCA, heatmap, and stacked-bar graphical outputs are produced from the DESeq analysis without errors
- Differing features are correctly reported for the intended comparison (control vs. experimental groups, or individual metatranscriptomes)
- Abundance summaries imported into R correspond one-to-one with the ranked tables emitted by DIAMOND_analysis_counter.py for each metatranscriptome file

## Limitations

- SAMSA2's aggregation scripts require Python 2.7 (tested on Linux only), which constrains the environment feeding this analysis stage
- Only the relative significance of features between the compared groups is assessed; the pipeline outputs simplified tables intended for direct examination or further analysis, not absolute quantification
- DIAMOND is run by default in --fast mode, which prioritizes speed over accuracy, so upstream annotation quality bounds the differential results

## Evidence

- [readme] R scripts use DESeq to compute most significantly different features between control vs. experimental samples.: "R scripts use DESeq to compute most significantly different features between control vs. experimental samples."
- [readme] These R scripts generate a tabular output with assigned p-values and log2FoldChange scores for each feature.: "These R scripts generate a tabular output with assigned p-values and log2FoldChange scores for each feature."
- [methods] use "run_DESeq_stats.R" to determine the most significantly differing features between either individual metatranscriptomes, or control vs. experimental groups: "use "run_DESeq_stats.R" to determine the most significantly differing features between either individual metatranscriptomes, or control vs. experimental groups"
- [methods] PCA plots and other graphical outputs.: "PCA plots and other graphical outputs."
- [methods] use "DIAMOND_analysis_counter.py" to create a ranked abundance summary of the DIAMOND results from each metatransciptome file: "use "DIAMOND_analysis_counter.py" to create a ranked abundance summary of the DIAMOND results from each metatransciptome file"

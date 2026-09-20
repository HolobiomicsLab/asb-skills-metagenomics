---
name: metatranscriptome-diversity-index-computation
description: Use when SAMSA2 abundance summary tables (ranked feature counts per metatranscriptome,
  produced by DIAMOND_analysis_counter.py) are available and per-sample diversity
  (Shannon, Simpson) or downstream DESeq statistics are needed.
when_to_use_negative:
- Input is raw FASTQ sequencing data that has not yet been through preprocessing,
  DIAMOND annotation, and Python aggregation — run the first 3 of 4 SAMSA2 pipeline
  stages first
- The goal is read-level annotation rather than sample-level statistics; use the DIAMOND
  annotation and aggregation stages instead of the R analysis stage
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SAMSA2
  role: Pipeline whose R analysis stage (diversity_stats.R, run_DESeq_stats.R, make_DESeq_PCA.R,
    make_DESeq_heatmap.R, make_combined_graphs.R) computes diversity indices, DESeq
    statistics, and graphical outputs from aggregated counts
  repo: https://github.com/transcript/samsa2
- name: DIAMOND
  role: Accelerated BLAST-like aligner that annotates reads against a local database;
    its per-read results are aggregated into the ranked abundance summaries feeding
    the diversity computation
  repo: https://github.com/bbuchfink/diamond
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-018-2189-z
    title: 'SAMSA2: a standalone metatranscriptome analysis pipeline'
  source_content_hash: sha256:35b4c3c0649fce4198b82522e735473d95f866cd2abaa642422186f718ed0f33
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/transcript/samsa2
  tools:
  - DESeq
  - DIAMOND
  - R
  - SAMSA2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# metatranscriptome-diversity-index-computation

## Summary

Computes per-sample Shannon and Simpson alpha-diversity indices from SAMSA2's ranked abundance summaries, as part of the R-based analysis stage that also runs DESeq differential testing and generates PCA/heatmap/stacked-bar plots.

## When to use

Use when SAMSA2 abundance summary tables (ranked feature counts per metatranscriptome, produced by DIAMOND_analysis_counter.py) are available and per-sample diversity (Shannon, Simpson) or downstream DESeq statistics are needed.

## When NOT to use

- Input is raw FASTQ sequencing data that has not yet been through preprocessing, DIAMOND annotation, and Python aggregation — run the first 3 of 4 SAMSA2 pipeline stages first
- The goal is read-level annotation rather than sample-level statistics; use the DIAMOND annotation and aggregation stages instead of the R analysis stage

## Inputs

- Ranked abundance summary tables per metatranscriptome produced by DIAMOND_analysis_counter.py (feature-level counts for organisms or functions)
- DIAMOND per-read annotation results against a local DIAMOND-indexed database (e.g., NCBI RefSeq bacteria, SEED Subsystems)

## Outputs

- Per-sample Shannon and Simpson diversity index values
- DESeq results table with assigned p-values and log2FoldChange scores per feature
- PCA plot of sample-level variance, heatmaps of top significantly differing features, and combined stacked-bar graphs of relative abundances

## How to apply

First run the SAMSA2 upstream stages (preprocessing with PEAR/Trimmomatic/SortMeRNA, DIAMOND annotation against a local DIAMOND-indexed database, then DIAMOND_analysis_counter.py) to obtain ranked abundance summaries for each metatranscriptome, where features may be organisms or specific functions. Import these summary tables into R as count matrices and compute Shannon and Simpson diversity indices per sample from the sorted abundance counts using diversity_stats.R. Follow with run_DESeq_stats.R to extract p-values and log2FoldChange per feature, make_DESeq_PCA.R on variance-stabilized counts, make_DESeq_heatmap.R for top differing features, and make_combined_graphs.R for relative-abundance stacked bars. All R scripts are written to be executable directly from the command line, enabling scripted pipeline validation.

## Related tools

- **SAMSA2** (Pipeline whose R analysis stage (diversity_stats.R, run_DESeq_stats.R, make_DESeq_PCA.R, make_DESeq_heatmap.R, make_combined_graphs.R) computes diversity indices, DESeq statistics, and graphical outputs from aggregated counts) — https://github.com/transcript/samsa2
- **DIAMOND** (Accelerated BLAST-like aligner that annotates reads against a local database; its per-read results are aggregated into the ranked abundance summaries feeding the diversity computation) — https://github.com/bbuchfink/diamond

## Examples

```
Rscript diversity_stats.R
```

## Evaluation signals

- Each R script executes from the command line without errors (README notes the 'better, more polished R scripts that can be executed from the command line')
- DESeq output table contains both p-value and log2FoldChange columns for every annotated feature
- Diversity indices are computed per sample from sorted abundance counts, giving both Shannon and Simpson values per metatranscriptome
- PCA plot, heatmap, and stacked-bar graphical outputs are generated without errors
- Abundance summaries were produced from DIAMOND results via DIAMOND_analysis_counter.py, confirming the correct upstream aggregation ran before R import

## Limitations

- Pipeline requires Python 2.7 for aggregation scripts (tested on Linux); Python 3 may work but errors may not have been caught
- DIAMOND runs in --fast mode by default, prioritizing speed over accuracy; switching to --sensitive may significantly increase memory requirements
- Full-gene-level accuracy is limited to what DIAMOND-based annotation provides (SAMSA2 reports 95% genus-level accuracy vs 91% for SAMSA 1.0)

## Evidence

- [readme] R analysis stage uses DESeq and diversity plotting: "Analysis: R scripts use DESeq to compute most significantly different features between control vs. experimental samples."
- [readme] Ranked abundance summaries are the R-stage input: "use "DIAMOND_analysis_counter.py" to create a ranked abundance summary of the DIAMOND results from each metatransciptome file."
- [methods] DESeq statistics over individual or grouped metatranscriptomes: "use "run_DESeq_stats.R" to determine the most significantly differing features between either individual metatranscriptomes, or control vs. experimental groups"
- [readme] Graphical outputs in SAMSA2: "Better, more polished R scripts that can be executed from the command line."
